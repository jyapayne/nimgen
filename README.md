[![Chat on Gitter](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/nimgen/Lobby)
[![Build status](https://ci.appveyor.com/api/projects/status/05t5ja88lmv1rt3r/branch/master?svg=true)](https://ci.appveyor.com/project/genotrance/nimgen/branch/master)
[![Build Status](https://travis-ci.org/genotrance/nimgen.svg?branch=master)](https://travis-ci.org/genotrance/nimgen)

Nimgen is a helper for [c2nim](https://github.com/nim-lang/c2nim/) to simplify and automate the wrapping of C libraries.

Nimgen can be used to automate the process of manipulating C files so that c2nim can be run on them without issues. This includes adding/removing code snippets, removal of complex preprocessor definitions that c2nim doesn't yet comprehend and recursively running on #include files.

__Installation__

Nimgen can be installed via [Nimble](https://github.com/nim-lang/nimble):

```> nimble install nimgen```

This will download, build and install nimgen in the standard Nimble package location, typically ~/.nimble. Once installed, it can be run just like c2nim.

__Usage__

Nimgen is driven by a simple .cfg file that is read using the Nim [parsecfg](https://nim-lang.org/docs/parsecfg.html) module. The sections of the file are described further below.

```> nimgen package.cfg```

A Nimble package for a library that is wrapped with nimgen will have the following:-

* The .cfg file that tells nimgen what exactly to do
* Nimgen defined as a dependency defined in the .nimble file
* Steps within the .nimble file to download the source code that is being wrapped

This way, the library source code doesn't need to get checked into the Nimble package and can evolve independently.

Nimble already requires Git so those commands can be assumed to be present to download source from a repository. Mercurial is also suggested but depends on the user. Downloading arbitrary files depends on the OS. For Linux, wget/curl can be assumed. On Windows, [powershell](https://superuser.com/questions/362152/native-alternative-to-wget-in-windows-powershell) can be used.

__Capabilities & Limitations__

Nimgen supports compiling in C/C++ sources and static libraries as well as loading in dynamic libraries.

To see examples of nimgen in action check out the following wrappers:-
* Link with a dynamic library
    * [nimbass](https://github.com/genotrance/nimbass) - BASS audio wrapper: [docs](http://nimgen.genotrance.com/nimbass)
        * download ZIP with headers and binary
    * [nimlibxlsxwriter](https://github.com/KeepCoolWithCoolidge/nimlibxlsxwriter) - libxlsxwriter wrapper
        * git checkout
    * [nim-clblast](https://github.com/numforge/nim-clblast) - OpenCL BLAS wrapper
        * static

* Compile C/C++ code into binary
    * [nim7z](https://github.com/genotrance/nim7z) - 7z decoder wrapper: [docs](http://nimgen.genotrance.com/nim7z)
        * git sparse checkout
    * [nimarchive](https://github.com/genotrance/nimarchive) - libarchive wrapper: [docs](http://nimgen.genotrance.com/nimarchive)
        * git sparse checkout
    * [nimbigwig](https://github.com/genotrance/nimbigwig) - libbigWig wrapper: [docs](http://nimgen.genotrance.com/nimbigwig)
        * git checkout
    * [nimclipboard](https://github.com/genotrance/nimclipboard) - libclipboard wrapper: [docs](http://nimgen.genotrance.com/nimclipboard)
        * git checkout
    * [nimfastText](https://github.com/genotrance/nimfastText) - fastText wrapper: [docs](http://nimgen.genotrance.com/nimfastText)
        * git sparse checkout
    * [nimfuzzy](https://github.com/genotrance/nimfuzzy) - fts_fuzzy_match wrapper: [docs](http://nimgen.genotrance.com/nimfuzzy)
        * download header file
    * [nimkerberos](https://github.com/genotrance/nimkerberos) - WinKerberos wrapper: [docs](http://nimgen.genotrance.com/nimkerberos)
        * git sparse checkout
    * [nimmonocypher](https://github.com/genotrance/nimmonocypher) - monocypher wrapper: [docs](http://nimgen.genotrance.com/nimmonocypher)
        * git sparse checkout
    * [nimnuklear](https://github.com/genotrance/nimnuklear) - nuklear wrapper: [docs](https://nimgen.genotrance.com/nimnuklear)
        * git sparse checkout
    * [nimpcre](https://github.com/genotrance/nimpcre) - PCRE wrapper: [docs](http://nimgen.genotrance.com/nimpcre)
        * git checkout
    * [nimrax](https://github.com/genotrance/nimrax) - Radix tree wrapper: [docs](http://nimgen.genotrance.com/nimrax)
        * git checkout
    * [nimssl](https://github.com/genotrance/nimssl) - OpenSSL wrapper: [docs](http://nimgen.genotrance.com/nimssl)
        * git sparse checkout
    * [nimtess2](https://github.com/genotrance/nimtess2) - libtess2 wrapper: [docs](http://nimgen.genotrance.com/nimtess2)
        * git checkout
    * [duktape-nim](https://github.com/manguluka/duktape-nim) - Duktape wrapper
        * static

* Compile in as static binary
    * [nimssh2](https://github.com/genotrance/nimssh2) - libssh2 wrapper: [docs](http://nimgen.genotrance.com/nimssh2)
        * git sparse checkout

Nimgen only supports the ```gcc``` preprocessor at this time. Support for detecting and using other preprocessors will be based on interest.

__Config file__

In all sections below, environment variables are supported via Nim's string interpolation `%` symbol imported from the `strutils` module. Simply use double quotes to enclose any value and put `$` or `${}` around the environment variable name. In addition, the `output` var from the n.global section is available as ${output}. For example:

    [n.global]
    c_compiler="$CC"
    cpp_compiler="${CPP}-arm"
    output="src/path"

    [n.include]
    "${output}/library/include"
    "${MY_INCLUDE_PATH}/include"

Append -win, -lin and -mac/osx for OS specific sections and tasks. E.g. n.global-win, n.post-lin, download-win, execute-lin-mac.unique1. -unix can be used as a shortcut for -lin-mac.

_[n.global]_

```output``` = name of the Nimble project once installed, also location to place generated .nim files

```quotes``` = pick up any headers included using "" (and not <> which is typically used for standard headers) [default: true]

```filter``` = string to identify and recurse into library .h files in #include statements and exclude standard headers

```cpp_compiler``` = string to specify a CPP compiler executable. [default: g++]

```c_compiler``` = string to specify a C compiler executable. [default: gcc]

_[n.include]_

List of all directories, one per line, to include in the search path. This is used by:-
* The preprocessor for #include files
* Nimgen to find #include files that are recursively processed

Nimgen also adds {.passC.} declarations into the generated .nim files for these include paths if compiling source files directly.

_[n.exclude]_

List of all directories or files to exclude from all parsing. If an entry here matches any portion of a file, it is excluded from recursive processing.

_[n.prepare]_

The following keys can be used to prepare dependencies such as downloading ZIP files, cloning Git repositories, etc. Multiple entries are possible by appending any .string to the key. E.g. download.file1.

```download``` = url to download to the output directory. ZIP files are automatically extracted. Files are not redownloaded if already present but re-extracted

```extract``` = ZIP file to extract in case they are local and don't need to be downloaded. Path is relative to output directory.

```gitcheckout``` = branch, commit or tag of repository to checkout in following Git command, resets after each use. Use "-b name" for branches

```gitoutput``` = directory for all following Git commands relative to `n.global:output` [default: `n.global:output` directory]

```git``` = url of Git repository to clone. Full repo is pulled so gitremote + gitsparse is preferable. Resets if already present

```gitremote``` = url of Git repository to partially checkout. Use with gitsparse to pull only files and dirs of interest

```gitsparse``` = list of files and/or dirs to include in partial checkout, one per line. Resets if already present

```execute``` = command to run during preparation

```copy``` = copy a file to another location. Preferred over moving to preserve original. Comma separate for multiple entries. E.g. copy = "output/config.h.in=output/config.h"

_[n.post]_

This section is the same as the prepare section, but for performing actions after the project has been processed.

```gitoutput``` = output directory for Git reset [default: `n.global:output` directory]

```reset``` = perform a Git reset on all files after processing [default: false]

```execute``` = command to run after processing

_[n.wildcard]_

File wildcards such as *.nim, ssl*.h, etc. can be used to perform tasks across a group of files. This is useful to define common operations such as global text replacements without having to specify an explicit section for every single file. These operations will be performed on every matching file that is defined as a _sourcefile_ or recursed files. Only applies on source files following the wildcard declarations.

```wildcard``` = pattern to match against. All keys following the wildcard declaration will apply to matched files

_[n.sourcefile]_

This section allows selection of multiple sourcefiles without requiring a detailed section for each file. Each specific file can be listed one line at a time and file wildcards can be used to include multiple source files. E.g. `$output/include/*/h`. `[n.wildcard]` definitions can be used to perform common operations on these source files if required.

_[sourcefile]_

The following keys apply to library source code and help with generating the .nim files. -win, -lin and -osx can be used for OS specific tasks. E.g. dynlib-win, pragma-win.

```recurse``` = find #include files and process them [default: false]

```inline``` = include #include files into file being processed, alternative method to processing each header file separately with recurse. Multiple source files will get combined into the same .nim output files [default: false]

```preprocess``` = run preprocessor (gcc -E) on file to remove #defines, etc. [default: false] - this is especially useful when c2nim doesn't support complex preprocessor usage

```ctags``` = run ctags on file to filter out function definitions [default: false] - this requires the ctags executable and is an alternative to filter out preprocessor complexity

```defines``` = pulls out simple #defines of ints, floats and hex values for separate conversion [default: false] - works only when preprocess or ctags is used and helps include useful definitions in generated .nim file

```flags``` = flags to pass to the c2nim process in "quotes" [default: --stdcall]. --cdecl, --assumedef, --assumendef may be useful

```ppflags``` = flags to pass to the preprocessor [default: ""]. -D for gcc and others may be useful

```noprocess``` = do not process this source file with c2nim [default: false] - this is useful if a file only needs to be manipulated

```nowildcard``` = ignore any wildcard definitions for this sourcefile

```reset``` = reset the file back to original state after all processing [default: false]

Multiple entries for the all following keys are possible by appending any .string to the key. E.g. dynlib.win, compile.dir

```compile``` = file or dir of files of source code to {.compile.} into generated .nim. If directory, picks *.c if C mode and *.cxx, *.cpp, *.cc, *.c++ and *.C for cpp mode. Dir can also include wildcards. e.g. compile = """dir/A*.cxx"""

```pragma``` = pragmas to define in generated .nim file. E.g. pragma = "passL: \"-lssl\"" => {.passL: "-lssl".}

```dynlib``` = dynamic library to load at runtime for generated .nim procs

The following keys apply to library source code (before processing) and generated .nim files (after processing) and allow manipulating the files as required to enable successful wrapping. They are not propagated to #include files when ```recurse = true```.

```create``` = create a file at exact location with contents specified. File needs to be in the _[n.exclude]_ list in order to be created.

```pipe``` = execute a command on a file and store the output of the command as the new file contents. E.g. pipe = "cat $file | grep 'static inline'"

```search``` = search string providing context for following prepend/append/replace directives

```regex``` = regex search string providing context for the following replace directive. Specify using """ to avoid regex parsing issues

```prepend``` = string value to prepend into file at beginning or before search

```append``` = string value to append into file at the end or after search

```replace``` = string value to replace search string in file. Regex captures can be referred to using $1, $2, etc.

```move``` = search string providing context for location to move the results of a preceding search or regex match

```comment``` = number of lines to comment from search location

The following key only applies before processing and allows renaming the generated .nim files as required to enable successful wrapping. This may be for organizational purposes or to prevent usage of non-nim supported strings in module names (E.g. first letter is a number). Destination is relative to output directory if defined.

```rename``` = string value to rename generated filename. E.g. rename = "$replace(7=s7)"

  `/` = create a directory/module hierarchy

  `$nimout` = refer to the original filename

  `$replace(srch1=repl1, srch2=reply2)` = rename specific portions in `$nimout`

__Command Line__

A subset of capabilities are available through the command line to enable quick tests using nimgen. Command line flags only apply to source files specified on the command line and do not influence any ```cfg``` files which are expected to be self-sufficient.

```
Usage:
  nimgen [options] file.cfg|file.h ...

Params:
  -C<compile>  add compile entry       *
  -E<exclude>  add n.exclude entry     *
  -F<flags>    set c2nim flags         *
  -I<include>  add n.include dir       *
  -O<outdir>   set output directory
  -P<ppflags>  set preprocessor flags  *

Options:
  -c           set ctags = true
  -d           set defines = true
  -i           set inline = true
  -n           set noprocess = true
  -p           set preprocess = true
  -r           set recurse = true

Editing:
  -a<append>   append string           *
  -e<prepend>  prepend string          *
  -l<replace>  replace string          *
  -o#lines     comment X lines         *
  -s<search>   search string           *
  -x<regex>    regex search string     *

* supports multiple instances
```

__Feedback__

Nimgen is a work in progress and any feedback or suggestions are welcome. It is hosted on [GitHub](https://github.com/genotrance/nimgen) with an MIT license so issues, forks and PRs are most appreciated. Also join us at https://gitter.im/nimgen/Lobby to chat about nimgen and the future of Nim wrappers.

_Credits_

Thank you to the following contributors for their hard work!

https://github.com/jyapayne
