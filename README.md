# fatten

[fatten] takes [shellfire] applications and turns them into standalone programs by a process we call [fatten]ing. It does this by running the [shellfire] application in a special mode. It's written using [shellfire] and [fatten]ed by itself: it eats its own dog food.

## What's included?

Explicility, the following are included:-

* All functions loaded from modules, as defined at the point of [fatten]ing
* Any functions defined inside `_program()`
* Any `_program_*` settings in your [shellfire] application
* Global variables (or constants) explicitly in scope (those registered using `core_dependency_declares()` see [here](https://github.com/shellfire-dev/core#core_dependency_declares))
* Any snippets (files, text, that sort of thing: see the namespace `core_snippet` [here](#namespace-core_snippet))

## How to use it

For example, image you have the [shellfire] application 'overdrive'. You have a git repository 'overdrive' (perhaps at GitHub), containing the following structure:-

```bash
overdrive\
	.git\
	overdrive           # your shellfire application script
	etc\
		shellfire\
			paths.d\    # git submodule add https://github.com/shellfire-dev/paths.d
				…
		overdrive\
			paths.d\    # your application specifc .paths (if any)
				…
	lib\
		shellfire\
			core\       # git submodule add https://github.com/shellfire-dev/core
			…\          # other submodules, may be https://github.com/shellfire-dev/jsonreader
			overdrive\  # any code for your application broken out into namespaces
output\
```
You might run the following command from outside of this structure:-

```bash
fatten --repository-path ./overdrive --output-path ./output -- overdrive
```

This will then put a [fatten]ed file `overdrive` in `./output/overdrive`.

## Command Line

### Options

|Short Option|Long Option|Has Argument|Default|Description|
|------------|-----------|------------|-------|-----------|
|`-v`|`--verbose`|Optionally|`0`|Specify more than once for more verbosity. Use a number between `0` and `3` inclusive to set a specific level. Higher numbers are more verbose.|
|`-r`|`--repository-path`|`PATH`|Current path|Path to a git repository root `PATH` containing a `PROGRAM`|
|`-e`|`--etc-path`|`PATH`|`/etc`|Path `PATH` to embed in `PROGRAM` for etc|
|`-l`|`--lib-path`|`PATH`|`/lib`|Path `PATH` to embed in `PROGRAM` for lib|
|`-p`|`--var-path`|`PATH`|`/var`|Path `PATH` to embed in `PROGRAM` for var|

|`-o`|`--output-path`|`PATH`|Current path|Path `PATH` to folder (created if necessary) for fattened `PROGRAM`|
|`-c`|`--core-path`|`PATH`|Usually `lib/shellfire/core`|Path `PATH` to shellfire core|

### Non-Options

Everything after options (`--`) is a `PROGRAM` to fatten in `--repository-path` and output to `--output-path`.

"
	_program_commandLine_helpMessage_examples="
  ${_program_name} -r git-repo/bin -- minor-test
"
}


## Configuration

Anything you can do with a command line switch, you can do as configuration. But configuration can also be used with scripts. Indeed, the configuration syntax is simply shell script. Configuration files _should not_ be executable. This means that if you _really_ want to, you can override just about any feature or behaviour of [fatten] - although that's not explicitly supported. Configuration can be in any number of locations. Configuration may be a single file, or a folder of files; in the latter case, every file in the folder is parsed in 'shell glob-expansion order' (typically ASCII sort order of file names).

To set a key, use normal shell script syntax in a file, eg:-

```bash
# $HOME/.fatten/rc

# This sets the value for --etc-path
fatten_etcPath="/usr/local/etc"
```

### Configuration Keys

|Key|Equivalent Command Line Long Option|
|---|-----------------------------------|
|`fatten_verbose`\*|`--verbose`|
|`fatten_language`†|_N/A_|
|`fatten_etcPath`|`--etc-path`|
|`fatten_libPath`|`--lib-path`|
|`fatten_varPath`|`--var-path`|
|`fatten_repositoryPath`|`--repository-path`|
|`fatten_force`‡|`--force`|
|`fatten_outputPath`|`--output-path`|
|`fatten_corePath`|`--core-path`|

_\* Set this to an integer number between `0` to `3`, eg `fatten_verbose=1`. `0` is off (the default)._
_† This is used for `LC_*` variables so things like `sort` work reliably. Don't change it unless you have to. It defaults to `en_US.UTF-8`._
_‡ This is modelled as boolean. Set `fatten_force='yes'` to be equivalent to `--force`. You can still override this on the command line with `--force no`. _

### Configuration Locations

Locations are searched in order as follows:-

1. Global (Per-machine)
  1. The file `INSTALL_PREFIX/etc/fatten/rc` where `INSTALL_PREFIX` is where [fatten] has been installed.
  2. Any files in the folder `INSTALL_PREFIX/etc/fatten/rc.d`
2. Per User, where `HOME` is your home folder path\*
  1. The file `HOME/.fatten/rc`
  2. Any files in the folder `HOME/.fatten/rc.d`
3. Per Environment
  1. The file in the environment variable `fatten_RC` (if the environment variable is set and the path is readable)
  2. Any files in the folder in the environment variable `fatten_RC_D` (if the environment variable is set and the path is searchable)

Nothing stops any of these paths, or files in them, being symlinks.

_\* An installation as a daemon using a service account would normally set `HOME` to something like `/var/lib/fatten`._

### Blacklisting Configuration Locations

You can also blacklist the loading of configuration from paths defined by environment variables. In a machine-wide configuration file, put the directive `core_configuration_blacklist VAR`, where `VAR` is one or more of:-

* `_program_etcPath`
* `HOME`
* `shellfire_RC`
* `shellfire_RC_D`
* `fatten_RC`
* `fatten_RC_D`

## Exit Codes
[fatten] tries to follow the BSD exit code conventions. A non-zero exit code is indicative of failure. Typical codes are:-

| Code | Meaning | Common Causes |
| ---- | ------- | ------------- |
| 78   | Configuration issue | Configuration omitted, contradictory or incorrectly specified. |
| 77   | Permission Denied | Run with setuid / setgid bits set. |
| 76   | Protocol | Not used |
| 75   | Temporary Failure | Not used |
| 74   | I/O Error | Not used |
| 73   | Can't create | We couldn't create a temporary file or folder |
| 72   | Missing File | We tried very hard, but even a fallback dependency was missing |
| 71   |  | Not used |
| 70   | Internal Error | Something went wrong with [fatten]; an assumption was violated |
| 69   | Unavailable |  Not used |
| 68   | Unknown Host | Not used |
| 67   | Unknown User | Not used |
| 66   |  | Not used |
| 65   | Data Error | Not used |
| 64   | Incorrect command line | Command line switches omitted, contradictory or incorrectly specified |
| 2    |  | A shell builtin misbehaved |
| 1    |  | Something went wrong we didn't expect or couldn't intercept |
| 0    |  | Successful operation |



[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[fatten]: https://github.com/shellfire-dev/fatten "fatten homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[jsonreader]: https://github.com/shellfire-dev/jsonreader "jsonreader shellfire module homepage"
[xmlwriter]: https://github.com/shellfire-dev/xmlwriter "xmlwriter shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"

