# GRASP BTOP PROJECT

## Makefile

1. **BANNER** Line 3 

    *Linux Console Color*

    HELP CONTENT 
    
    [Linux Console Color](https://www.cnblogs.com/unclemac/p/12783387.html) or use `man console_color` in shell

2. **BTOP_VERSION** Line 5

    `$(shell head -n100 src/btop.cpp 2>/dev/null | grep "Version =" | cut -f2 -d"\"" || echo " unknown")`

    `2>/dev/null`           Error Message stream goes to Msg Black Hole. /dev/null will delete any bytes written into it.

    `cut -f2 -d"\""`        `-d"\""` cut the stdin into many fields by "\"", `-f2` chooses the second fields.

    `|| echo " unknown"`    If there are any errors up front, do `echo`

3. **TIMESTAMP** or **DATESTAMP** Line 6 or 7

    `$(shell date +%s 2>/dev/null || echo "0")`     `date` get current date, `+%s` get seconds from 1970.1.1 00:00:00

4. **echo $$?** Line 8

    `shell command -v gdate >/dev/null; echo $$?`   `command -p` find command in given path and run command
                                                    `command -v` Search command path but not run command

    `echo $$?`                                      Return the return value of the previous command

5. **filter** Line 25

    `$(filter unknown Darwin, $(PLATFORM))`     filter given element (first argument) from the second element, the return value could be empty

6. **tr** Line 37

    `$(shell echo $(PLATFORM) | tr '[:upper:]' '[:lower:]')`    `tr` command replace the first argument element to the second

7. **$(info text)** Line 195

    Makefile problem <tab> and <space>. If there is <tab> before statement, this statement will be treated as a shell command. However, if no target before this statement, it still will be treated as makefile statement. A <space> for ident will be nice. <tab> and <space> not same in makefile.

    Be careful not put any <space> after a value, for example:

    `A=B<space>` will make `ifeq ($(A),B)` return false cause \$(A) return A<space>, but `A<space>=<space>B` will make `ifeq ($(A),B)` return true. A front <space> will be OK

8. **.ONESHELL:** Line 229

    `.ONESHELL` will make commands after it run in the same shell. Without `.ONESHELL`, only commands in the same line will run in the same shell. Here is an example:

    ```
    all:
        cd ~/github
        pwd
    ```
    
    will return ~ if this make run in ~ dir

    ```
    .ONESHELL:
    all:
        cd ~/github
        pwd
    ```

    and

    ```
    all:
        cd /github;pwd
    ```

    will return ~/github 

9. **.PHONY:** Line 373

    Non-File Targets


## btop From main

### INIT

1. **getuid() and geteuid()** Line 818

    `getuid()` get real usrid, `geteuid()` get effective usrid. //TODO

2. **for (const auto& env : {"XDG_CONFIG_HOME", "HOME"})** Line832

    Line above create a auto type (or char *) reference env of array `{"XDG_CONFIG_HOME", "HOME"}`

3. **getenv(env)** Line 833

    `getenv()` has a argument of char * type, which is name of environment variable, and this function aim to get its content in OS. Here is an example

    ```
    #include <stdio.h>
    #include <stdlib.h>

    int main()
    {
            printf("PATH: %s\n",getenv("PATH"));
            printf("HOME: %s\n",getenv("HOME"));
            printf("ROOT: %s\n",getenv("ROOT"));
            return 0;
    }
    ```

    Run this code and we get

    ```
    PATH: /home/lee/anaconda3/bin:/mnt/c/Users/AlbertLee/.dotnet/tools // Here is a lot omittance
    HOME: /home/lee
    ROOT: (null)
    ```

4. **access()** Line 833

    This function test if usr has permission to access files whose path is its argument

5. **std::filesystem** Line 834 //TODO add more content

    To using this namespace, you must include <filesystem> at beginning of file. fs = filesystem

    ```
    fs::path(std::getenv(env)) / (((string)env == "HOME") ? ".config/btop" : "btop")
    ```

    `/` append directory.

    ```
    Theme::theme_dir = fs::canonical(Global::self_path / "../share/btop/themes", ec);
    ```
    `fs::canonical` converts path p to a canonical absolute path, i.e. an absolute path that has no dot, dot-dot elements or symbolic links in its generic format representation. //TODO Error output?

6. **/proc/self/exe** Line 857

    `/proc/self/exe` is a symlink to current program, which means you can get current program directory by calling `std::filesystem::read_symlink`

### Config init

1. **Config load** in btop_config.cpp

    1) void load(const fs::path& conf_file, vector<string>& load_warnings)

        (1) `std::ifstream cread(conf_file);` in btop_config.cpp Line 602
        ```
        cread.peek() //return the next character but not delete it in stream
        getline(cread, v_string, '\n') //getline from stream cread and write in v_string, getline until '\n'(dlim), and stream will not contain '\n'
        cread >> std::ws; //will delete <space> and '\n' in stream before valid character
        cread.ignore(SSmax, '\n'); //will keep ignoring (delete) SSmax characters until '\n'
        ```
        example: test.cpp
        ```
        #include "iostream"
        #include "filesystem"
        #include "fstream"
        #include "string"
        namespace fs = std::filesystem;

        int main()
        {
                fs::path Path("./info.log");
                std::ifstream Read(Path);
                if(Read.good())
                {
                        std::cout << "stream :";
                        std::cout << (char)(Read.peek()) << std::endl;
                        Read >> std::ws;
                        std::cout << "stream :";
                        std::cout << (char)(Read.peek()) << std::endl;
                        std::cout << "stream :";
                        std::cout << (char)(Read.get()) << std::endl;
                        std::cout << "stream :";
                        std::cout << (char)(Read.peek()) << std::endl;
                        std::string vstring;
                        getline(Read,vstring,'=');
                        std::cout << "stream :";
                        std::cout << vstring << std::endl;
                        std::cout << "stream :";
                        std::cout << (char)(Read.peek()) << std::endl;
                        return 0;
                }else{
                        return 1;
                }
        }
        ```
        info.log
        ```
        
                 #inf=o
        123
        ```
        output
        ```
        stream :

        stream :#
        stream :#
        stream :i
        stream :inf
        stream :o
        ```

        (2) `template` in btop_tools.hpp Line 223

        ```
        //* Compare <first> with all following values
        template<typename First, typename ... T>
        inline bool is_in(const First& first, const T& ... t) {
            return ((first == t) or ...);
        }
        ```
    
        (3) `bools.contains(name)` in btop_config.cpp Line 624
        ```
        if (bools.contains(name)) {
            cread >> value;
            if (not isbool(value))
                load_warnings.push_back("Got an invalid bool value for config name: " + name);
            else
                bools.at(name) = stobool(value);
		}
        ```

2. `Config::check_boxes(Config::getS("shown_boxes"))` Line 885 in btop.cpp

    how to use GPU_SUPPORT?

