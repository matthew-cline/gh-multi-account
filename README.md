`gh-ma` is a Bash wrapper script for the [GitHub CLI executable](https://cli.github.com/) which makes it easier to simultaneously use multiple accounts on the same host via the use of [personal access tokens](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).  When [CLI issue #326](https://github.com/cli/cli/issues/326) is finally implemented the `gh-ma` script will be retired.

Pairs of token ids and tokens are stored in a configuration file, and one of the tokens is chosen at runtime.  A token is chosen as follows (in order of precedence):

1. If any of the [GitHub CLI environmental variables](https://cli.github.com/manual/gh_help_environment) related to tokens are set then those are used.
1. If `--token-id <ID>` is present at the start of the command line arguments then token id `<ID>` is used and looked up in `tokens.conf`.
1. If `gh-ma` is invoked in a git repository with a remote origin then the id in `tokens.conf` with the most specific match to the remote origin URL is chosen. (If no match is found an error occurs)
1. If the environmental variable `GH_DEFAULT_TOKEN_ID` is set then that is used.
1. If the file `default-token-id.sh` is present in the `gh-ma` config directory then it is sourced.  If it sets the global variable `token_id` then that is used as the token id.  As an example, you could use different token ids depending upon the current working directory.
1. If the file `default-token-id` is present in the `gh-ma` config directory then its content is used as the id.

# Installation and configuration
1. Rename `gh-ma` to `gh` and place it in a directory in your `PATH` which comes in `PATH` before the directory containing the GitHub CLI executable.  The wrapper script should be able to automatically detect and use the CLI executable.  If it can't, then set the environmental variable `GH_EXE` to the path of the CLI executable.
1. Create the config directory `$HOME/.conf/gh/multi-account`, or `$GH_CONFIG_DIR/multi-account` if using the `GH_CONFIG_DIR` environmental variable.  Alternatively you can make any arbitrary directory the config directory by using the environmental variable `GH_MA_CONFIG_DIR`.
1. In the config directory create the file `tokens.conf` to specify which accounts use which token.  If you wish, you can even use different tokens for different repositories under the same account.
1. Optionally create the files `default-token-id` and/or `default-token-id.sh` in the config directory.

**NOTE**: It is possible to use `gh-ma` via aliases instead of renaming it to `gh` and installing it in `PATH`, but this is not recommended.  Specifically, the GitHub CLI executable can rewrite the [git config file](https://git-scm.com/docs/git-config) so that `gh` is used as a [custom credential helper](https://git-scm.com/docs/gitcredentials#_custom_helpers) so that plain `git` commands will use GitHub CLI authentication.  If the wrapper script is only invoked via shell aliases then plain `git` commands will authenticate differently than wrapped invocations of `gh`.

# tokens.conf and token ids
Lines in the `tokens.conf` file take the format of `token_id: token`.  Multiple spaces and tabs are allowed between the colon and the [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) string.  The colon can be omitted if there is at least one whitespace character between the id and the token.  The token id must start at the beginning of the line, with no whitespace in front of it.

A token id has three different formats.  From most specific to least specific, the formats are:
1. `host/account/repo`
1. `host/account`
1. `host`

The most specific id which matches the current repository's remote origin URL will be used.

Example token ids:
```
github.com/google/guava
github.com/google
github.com
```

If the token value is either `none` or blank then the wrapper script will pass no token to the GitHub CLI executable, allowing the normal authentication flow to happen.

The order of the lines is unimportant, except when the exact same token id is used multiple times, in which case the last line using that token id is the one which will count.

Empty lines and lines starting with a "#" are ignored.

# Command line options
Any command line option meant for `gh-ma` must come at the very start of the command line arguments.

* `--token-id <ID>`: Specify the id of the [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) to use.  This overrides everything except for [GitHub CLI environmental variables](https://cli.github.com/manual/gh_help_environment) related to tokens.
* `--version-ma`: Prints the version of `gh-ma`.
* `--ma-info`: Prints various information about the configuration/setup of `gh-ma`.  Useful for troubleshooting.
* `--is-ma`: Exits with `0` if passed to the wrapper script, exits with `1` (and prints an error message) if passed directly to the CLI executable.

# Compatibility
`gh-ma` should run on any [POSIX](https://en.wikipedia.org/wiki/POSIX) compliant system on which is installed the executables for `bash`, `readlink`, `ln` and `which`.  However, it has only been tested on [Fedora](https://en.wikipedia.org/wiki/Fedora_(operating_system)) 33.

`gh-ma` should work with [GitHub Enterprise](https://github.com/enterprise), but this has not been tested.
