# TextExpander Snippets
A collection of TextExpander snippets I've created. Find below a category listing with the description of each snippets file/category and a few examples

## Table of Contents
* [Installing a group from this repository](#installing-a-group-from-this-repository)
* [Shell (bash) Scripting](#shell-bash-scripting)
    * [General](#general)
    * [Bash options](#bash-options)
    * [Product and script information](#product-and-script-information)
    * [Script options parsing](#script-options-parsing)
    * [Script templates](#script-templates)
    * [Example from shtpl (All options enabled)](#example-from-shtpl-all-options-enabled)

## Installing a group from this repository

1. Download one of the `.textexpander` suffixed files from the repository into a known folder (`Downloads` perhaps?)
2. Open the TextExpander window
3. Click on the `+` (Plus) button located in the bottom left corner of the screen and select the *Add Group from File* option in the appearing sub-menu
4. Select the file you downloaded earlier from the opened file browser. Once imported, a folder named similar to the downloaded file will show up in your TextExpander groups
5. Enjoy

## Shell (bash) Scripting

This contains bash specific snippets that are put together in a master snippet (shtpl). Below is a listing of the snippets it contains

### General

- **sh!** The shabang for the bash shell: `#!/bin/bash`

### Bash options

- **shaliason** Turn on alias expansion for your script: `shopt -s expand_aliases`
- **shstrict** Set the bash environment with the [Unofficial Bash Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/) plus some other options I've found useful:
    ```
    set -euo pipefail
    # If glob doesn't return anything then the result of the globbing is null
    # instead of the string with the glob expressions
    #shopt -s nullglob
    ```

### Product and script information

- **shproductinfo** Sets some `product_` prefixed variables containing the product information. Useful if you're creating a script around a product
    ```
    product_human_readable_name='%filltext:name=Product Human-Readable Name:default=<PRODUCT_HUMAN_READABLE_NAME>%'
    # To snake_case
    product_name="$( sed -re 's/\s+//g' <<< "${product_human_readable_name,,}" )"
    product_version='%filltext:name=Product Version:default=<PRODUCT_VERSION>%'
    product_release='%filltext:name=Product Release:default=%'
    ```
- **shscriptinfo** Sets some `script_` prefixed variables containing the script information. Useful for the script's help for example
    ```
    script_name='%filltext:name=Script Name:default=<SCRIPT_NAME>%'
    script_human_readable_name='%fillpart:name=Include Product Information%%filltext:name=Product Human-Readable Name:default=<PRODUCT_HUMAN_READABLE_NAME> %%fillpartend%%filltext:name=Script Long Name:default=<SCRIPT_HUMAN_READABLE_NAME>%'
    script_description='%filltext:name=Script Description:default=<SCRIPT_DESCRIPTION>%'
    script_version='%filltext:name=Script Version:default=<SCRIPT_VERSION>%'
    ```
- **shscriptdir** Sets the `script_directory` with the current script's absolute directory (If it is a symlink then it will contain the symlink's location), the `real_script_path` with the real (absolute and canonicalized) path (up to the filename) of the script, and  the `real_script_directory` with the real (absolute and canonicalized) directory containing the script. Useful when creating system tools that need to refer to a library directory or something of the like
    ```
    script_directory="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

    if real_script_path="$( readlink -e "$script_directory/$( basename "$0" )" )"; then
        real_script_directory="$( dirname "$real_script_path" )"
    else
        real_script_directory="$script_directory"
    fi
    ```

### Script options parsing

- **shscriptopts** Creates a functional options parsing script and functions to handle the default options in it (`-v` Verbose: Output progress messages, sometimes including the `set -x` output of certain parts of the script to the stdout, `-q` Quiet: Output only the script's progress messages and nothing more, `-s` Silent: Intended for automation, output a format that can be used by another tool to do something based on this script's output, e.g. If this is a build script, output a set of lines to the stdout containing the absolute paths of the files produced by this script)
   ```
   # Option Defaults
    verbose=false
    quiet=true
    silent=false

    # Options Function Definition
    function print_usage {
        local exit_code=0

        cat <<-USAGE
    %fillpart:name=Include Script Information%		$script_human_readable_name ($script_name) $script_version

        $script_description

        Usage: $script_name [OPTION]...
    %fillpartend%
        Output Mode Options
            -v    Verbose mode
            -q    Quiet mode
            -s    Silent mode

        Other Options

            -h    Prints this message and exits
      USAGE

        return "$exit_code"
    }
    function echo_silent {
        local exit_code=0

        if [[ "$silent" == true || "$quiet" == true || "$verbose" == true ]]; then
            echo "${@}"
            exit_code="$?"
        fi

        return "$exit_code"
    }
    function echo_quiet {
        local exit_code=0

        if [[ "$quiet" == true || "$verbose" == true ]]; then
            echo "${@}"
            exit_code="$?"
        fi

        return "$exit_code"
    }
    function echo_verbose {
        local exit_code=0

        if [[ "$verbose" == true ]]; then
            echo "${@}"
            exit_code="$?"
        fi

        return "$exit_code"
    }
    function redirect_command_with_mode {
        local exit_code=0

        if [[ "$silent" == true || "$quiet" == true ]]; then
            "${@}" 1>/dev/null
            exit_code="$?"
        elif [[ "$verbose" == true ]]; then
            "${@}"
            exit_code="$?"
        fi

        return "$exit_code"
    }

    number_of_mode_options=0

    # Option Parsing
    while getopts ":vqsh" option; do
        case "$option" in
            # Verbose
            v)
                number_of_mode_options=$((number_of_mode_options+1))

                verbose=true
                quiet=false
                silent=false
                ;;
            # Quiet
            q)
                number_of_mode_options=$((number_of_mode_options+1))

                verbose=false
                quiet=true
                silent=false
                ;;
            # Silent
            s)
                number_of_mode_options=$((number_of_mode_options+1))

                verbose=false
                quiet=false
                silent=true
                ;;
            # Help
            h)
                print_usage
                exit "$?"
                ;;
            \?)
                echo "WARNING: Invalid option -$OPTARG" >&2
                ;;
            :)
                echo "ERROR: Option -$OPTARG requires an argument" >&2
                exit_code=1
            ;;
        esac
    done

    if [[ "$exit_code" -eq 0 ]]; then
    %fillpart:name=Include Script Information%    echo_quiet -e "$script_human_readable_name ($script_name) $script_version\n"%fillpartend%
        if [[ "$number_of_mode_options" -gt 1 ]]; then
            echo "WARNING: Multiple mode options passed. The last one passed will prevail" >&2
        fi

        # TODO: Write code to execute after Options Parsing occurred
    else
        print_usage
    fi
    ```
    It is worth mentioning that the options parsing is made with `getopts` so no dependencies are needed
    
    You can use the `echo_(verbose|quiet|silent)` functions in this script to control the output. 
    - `echo_verbose` sends output to the stdout only if the `-v` option was set
    - `echo_quiet` outputs to stdout if `-q` or any more verbose option (`-v`) was set
    - `echo_(silent)` sends output to stdout if `-s` or any more verbose option (`-q|-v`) was set
    
    The `redirect_command_with_mode` function receives a command (You could pass a bash array expansion `${array[@]}` for example) and redirects its output based on the *mode* set with one of the verbosity options
    - If `-v` was set then the command's output is not redirected
    - If `-q` or '-s' were set then the command's stdout is redirected to `/dev/null` (aka. the output to stdout is suppressed). stderr stays unaltered

### Script templates

- **shbodytpl** The script's body template. Most of the snippets above are contained in here and are optional (Checkboxes control the template settings). It also features a single exit point and a newline at the end of the file
    ```
    %snippet:shstrict%

    exit_code=0

    %fillpart:name=Include Product Information%%snippet:shproductinfo%%fillpartend%

    %fillpart:name=Include Script Information%%snippet:shscriptinfo%%fillpartend%
    %snippet:shscriptdir%

    %fillpart:name=Include Default Script Options%%snippet:shscriptopts%%fillpartend
    %
    exit "$exit_code"


    ```
- **shtpl** The master script template. It wraps the script body template above with the shebang and the template's information (Off by default)
    ```
    %snippet:sh!%

    %fillpart:name=Bash Template Information%# Bash Script Template Version 1.1.0
    #
    # Changelog:
    #    MODIFIED    VERSION    (MM/DD/YYYY)
    #    mig8447     1.0.1      07/18/2018 - Added Product Information
    #    mig8447     1.0.0      06/27/2018 - Created%fillpartend%

    %snippet:shbodytpl%
    ```
    
## Example from shtpl (All options enabled)

Click on the below image to see a demo if the snippet in action

[![Example of the shtpl snippet](http://img.youtube.com/vi/-0p3SPxDEh0/0.jpg)](http://www.youtube.com/watch?v=-0p3SPxDEh0 "Example of the shtpl snippet")

Find the output of the snippet below

```
#!/bin/bash

# Bash Script Template Version 1.1.0
#
# Changelog:
#    MODIFIED    VERSION    (MM/DD/YYYY)
#    mig8447     1.0.1      07/18/2018 - Added Product Information
#    mig8447     1.0.0      06/27/2018 - Created

set -euo pipefail
# If glob doesn't return anything then the result of the globbing is null
# instead of the string with the glob expressions
#shopt -s nullglob

exit_code=0

product_human_readable_name='<PRODUCT_HUMAN_READABLE_NAME>'
# To snake_case
product_name="$( sed -re 's/\s+//g' <<< "${product_human_readable_name,,}" )"
product_version='<PRODUCT_VERSION>'
product_release=''

script_name='<SCRIPT_NAME>'
script_human_readable_name='<PRODUCT_HUMAN_READABLE_NAME><SCRIPT_HUMAN_READABLE_NAME>'
script_description='<SCRIPT_DESCRIPTION>'
script_version='<SCRIPT_VERSION>'
script_directory="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

if real_script_path="$( readlink -e "$script_directory/$( basename "$0" )" )"; then
    real_script_directory="$( dirname "$real_script_path" )"
else
    real_script_directory="$script_directory"
fi

# Option Defaults
verbose=false
quiet=true
silent=false

# Options Function Definition
function print_usage {
    local exit_code=0

    cat <<-USAGE
		$script_human_readable_name ($script_name) $script_version

		$script_description

		Usage: $script_name [OPTION]...

		Output Mode Options
		    -v    Verbose mode
		    -q    Quiet mode
		    -s    Silent mode

		Other Options

		    -h    Prints this message and exits
	USAGE

    return "$exit_code"
}
function echo_silent {
    local exit_code=0

    if [[ "$silent" == true || "$quiet" == true || "$verbose" == true ]]; then
        echo "${@}"
        exit_code="$?"
    fi

    return "$exit_code"
}
function echo_quiet {
    local exit_code=0

    if [[ "$quiet" == true || "$verbose" == true ]]; then
        echo "${@}"
        exit_code="$?"
    fi

    return "$exit_code"
}
function echo_verbose {
    local exit_code=0

    if [[ "$verbose" == true ]]; then
        echo "${@}"
        exit_code="$?"
    fi

    return "$exit_code"
}
function redirect_command_with_mode {
    local exit_code=0

    if [[ "$silent" == true || "$quiet" == true ]]; then
        "${@}" 1>/dev/null
        exit_code="$?"
    elif [[ "$verbose" == true ]]; then
        "${@}"
        exit_code="$?"
    fi

    return "$exit_code"
}

number_of_mode_options=0

# Option Parsing
while getopts ":vqsh" option; do
    case "$option" in
        # Verbose
        v)
            number_of_mode_options=$((number_of_mode_options+1))

            verbose=true
            quiet=false
            silent=false
            ;;
        # Quiet
        q)
            number_of_mode_options=$((number_of_mode_options+1))

            verbose=false
            quiet=true
            silent=false
            ;;
        # Silent
        s)
            number_of_mode_options=$((number_of_mode_options+1))

            verbose=false
            quiet=false
            silent=true
            ;;
        # Help
        h)
            print_usage
            exit "$?"
            ;;
        \?)
            echo "WARNING: Invalid option -$OPTARG" >&2
            ;;
        :)
            echo "ERROR: Option -$OPTARG requires an argument" >&2
            exit_code=1
        ;;
    esac
done

if [[ "$exit_code" -eq 0 ]]; then
    echo_quiet -e "$script_human_readable_name ($script_name) $script_version\n"
    if [[ "$number_of_mode_options" -gt 1 ]]; then
        echo "WARNING: Multiple mode options passed. The last one passed will prevail" >&2
    fi

    # TODO: Write code to execute after Options Parsing occurred
else
    print_usage
fi

exit "$exit_code"

```
