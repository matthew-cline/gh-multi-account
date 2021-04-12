`gh-ma` is a Bash wrapper script for the [GitHub CLI executable](https://cli.github.com/) which makes it easier to simultaneously use multiple accounts on the same host via the use of [personal access tokens](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

# Installation
1. Rename `gh-ma` to `gh` and place it in a directory in your `PATH` which comes before in `PATH` before the directory containing the GitHub CLI executable.  The wrapper script should be able to automatically detect and use the CLI executable.  If it can't, then set the environmental variable `GH_EXE` to the path of the CLI executable.
1. Create the config directory `$HOME/.conf/gh/multi-account`, or `$GH_CONFIG_DIR/multi-account` if using the `GH_CONFIG_DIR` environmental variable.  Alternatively you can make any arbitrary directory the config directory by using the environmental variable `GH_MA_CONFIG_DIR`.
1. In the config directory create the file `tokens.conf` to specify which accounts use which token.

**NOTE**: It is possible to instead us `gh-ma` via aliases, but this is not recommended.

# tokens.conf file format
Lines in the config file take the format of `token_id: token`.  Multiple spaces and tabs are allowed between the colon and the.  The colon can be omitted if there is at least one whitespace character between the id and the token.  The token id must start at the beginning of the line, with no whitespace in front of it.

A token id has three different formats.  From most specific to least specific, the formats are:
1. `host/account/repo`
1. `host/account`
1. `host`

The most specific id which matches the current repo will be used.

Example token ids:
```
github.com/google/guava
github.com/google
github.com
```

If the token value is either `none` or blank then the wrapper script will pass no token to the GitHub CLI executable, allowing the normal authentication flow to happen.

The order of the lines is unimportant, except when the exact same token id is used multiple times, in which case the last line using that token id is the one which will count.

Empty lines and lines starting with a "#" are ignored.

# Compatibility
`gh-ma` should run on any [POSIX](https://en.wikipedia.org/wiki/POSIX) compliant system on which is installed the executables for `bash`, `readlink`, `ln` and `which`.  However, it has only been tested on [Fedora](https://en.wikipedia.org/wiki/Fedora_(operating_system)) 33.
