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

4. **`echo $$?`** Line 8

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