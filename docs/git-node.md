# git-node

A custom Git command for managing pull requests. You can run it as
`git-node` or `git node`. To see the help text, run `git node`.

- [`git node land`](#git-node-land)
  - [Prerequistes](#git-node-land-prerequistes)
  - [Git bash for Windows](#git-bash-for-windows)
  - [Demo & Usage](#demo--usage)
- [`git node backport`](#git-node-backport)
  - [Example](#example)
- [`git node sync`](#git-node-sync)
- [`git node metadata`](#git-node-metadata)
- [`git node v8`](#git-node-v8)
  - [Prerequistes](#git-node-v8-prerequistes)
  - [`git node v8 major`](#git-node-v8-major)
  - [`git node v8 minor`](#git-node-v8-minor)
  - [`git node v8 backport <sha..>`](#git-node-v8-backport-sha)
  - [General options](#general-options)
- [`git node wpt`](#git-node-wpt)
  - [Example](#example-1)

## `git node land`

```
git-node land [prid|options]

Manage the current landing session or start a new one for a pull request

Positionals:
  prid, options  ID of the Pull Request                                 [number]

Options:
  --version       Show version number                                  [boolean]
  --help          Show help                                            [boolean]
  --apply         Apply a patch with the given PR id                    [number]
  --amend         Amend the current commit                             [boolean]
  --continue, -c  Continue the landing session                         [boolean]
  --final         Verify the landed PR and clean up                    [boolean]
  --abort         Abort the current landing session                    [boolean]

Examples:
  git node land 12344       Land https://github.com/nodejs/node/pull/12344 in
                            the current directory
  git node land --abort     Abort the current session
  git node land --amend     Append metadata to the current commit message
  git node land --final     Verify the landed PR and clean up
  git node land --continue  Continue the current landing session
```

<a id="git-node-land-prerequistes"></a>

### Prerequistes

1. See the readme on how to
   [set up credentials](../README.md#setting-up-credentials).
1. It's a Git command, so make sure you have Git installed, of course.
1. Configure your upstream remote and branch name.

   ```
   $ cd path/to/node/project
   $ ncu-config set upstream your-remote-name
   $ ncu-config set branch your-branch-name
   ```

   For example

   ```
   # Add a remote called "upstream"
   $ git remote add upstream git@github.com:nodejs/node.git
   # See your remote names
   $ git remote -v

   upstream	git@github.com:nodejs/node.git (fetch)
   upstream	git@github.com:nodejs/node.git (push)

   # Tell ncu that your upstream remote is named "upstream"
   $ ncu-config set upstream upstream

   # Tell ncu that you are landing patches to "master" branch
   $ ncu-config set branch master
   ```

### Git bash for Windows

If you are using `git bash` and having trouble with output use
`winpty git-node.cmd metadata $PRID`.

Current known issues with git bash:

- git bash Lacks colors.
- git bash output duplicates metadata.

### Demo & Usage

1. Landing multiple commits: https://asciinema.org/a/148627
2. Landing one commit: https://asciinema.org/a/157445

```
Steps to land a pull request:
==============================================================================
$ cd path/to/node/project

# If you have not configured it before
$ ncu-config set upstream <name-of-remote-to-nodejs/node>
$ ncu-config set branch master   # Assuming you are landing commits on master

$ git checkout master
$ git node land --abort          # Abort a landing session, just in case
$ git node land $PRID            # Start a new landing session
$ git node land $URL             # Start a new landing session using the PR URL

# Follow instructions provided.

$ git node land --final          # Verify all the commit messages
==============================================================================
```

Note that for all of these commands, you can run either
`git node <cmd>` or `git-node <cmd>` - they are just aliases.

```
git-node <command>

Commands:
  git-node land [prid|options]    Manage the current landing session or start a
                                  new one for a pull request
  git-node metadata <identifier>  Retrieves metadata for a PR and validates them
                                  against nodejs/node PR rules
  git-node v8 [major|minor|backport]  Update or patch the V8 engine

Options:
  --version  Show version number                                       [boolean]
  --help     Show help                                                 [boolean]
```

## `git node backport`

Demo: https://asciinema.org/a/221244

```
git node backport <identifier>

Backport a PR to a release staging branch.

Positionals:
  identifier  ID or URL of the pull request                            [string] [required]

Options:
  --version  Show version number                                                 [boolean]
  --help     Show help                                                           [boolean]
  --to       release to backport the commits to                        [number] [required]
```

### Example

```
Backporting https://github.com/nodejs/node/pull/12344 to v10.x

# Sync master with upstream for the commits, if they are not yet there
$ git checkout master
$ git node sync

# Backport existing commits from master to v10.x-staging
$ git checkout v10.x-staging
$ git node sync
$ git node backport 12344 --to 10
```

## `git node sync`

Demo: https://asciinema.org/a/221230

```
git node sync

Sync the branch specified by ncu-config.

Options:
  --version  Show version number                                                 [boolean]
  --help     Show help                                                           [boolean]
```

## `git node metadata`

This tool is inspired by Evan Lucas's [node-review](https://github.com/evanlucas/node-review),
although it is a CLI implemented with the GitHub GraphQL API.

```
git-node metadata <identifier>

Retrieves metadata for a PR and validates them against nodejs/node PR rules

Positionals:
  identifier  ID or URL of the pull request                            [string] [required]

Options:
  --version         Show version number                                          [boolean]
  --help            Show help                                                    [boolean]
  --owner, -o       GitHub owner of the PR repository         [string] [default: "nodejs"]
  --repo, -r        GitHub repository of the PR                 [string] [default: "node"]
  --file, -f        File to write the metadata in                                 [string]
  --readme          Path to file that contains collaborator contacts              [string]
  --check-comments  Check for 'LGTM' in comments                                 [boolean]
  --max-commits     Number of commits to warn                        [number] [default: 3]
```

Examples:

```bash
PRID=12345

# fetch metadata and run checks on nodejs/node/pull/$PRID
$ git node metadata $PRID
# is equivalent to
$ git node metadata https://github.com/nodejs/node/pull/$PRID
# is equivalent to
$ git node metadata $PRID -o nodejs -r node

# Or, redirect the metadata to a file while see the checks in stderr
$ git node metadata $PRID > msg.txt

# Using it to amend commit messages:
$ git node metadata $PRID -f msg.txt
$ echo -e "$(git show -s --format=%B)\n\n$(cat msg.txt)" > msg.txt
$ git commit --amend -F msg.txt

# fetch metadata and run checks on https://github.com/nodejs/llnode/pull/167
# using the contact in ../node/README.md
git node metadata 167 --repo llnode --readme ../node/README.md
```

## `git node v8`

Update or patch the V8 engine.  
This tool will maintain a clone of the V8 repository in `~/.update-v8/v8`
if it's used without `--v8-dir`.

<a id="git-node-v8-prerequistes"></a>

### Prerequistes

If you are on macOS, the version of `patch` command bundled in the system may
be too old for `git node v8` to work. Try installing a newer version of patch
before using this tool. For instance, with homebrew:

```
$ brew install gpatch
```

And make sure `which patch` points to `/usr/local/bin/patch` installed by
homebrew instead of `/usr/bin/patch` that comes with the system (e.g. by
modifying yoru `PATH` environment variable).

### `git node v8 major`

- Replaces `deps/v8` with a newer major version.
- Resets the embedder version number to `-node.0`.
- Bumps `NODE_MODULE_VERSION` according to the [Node.js ABI version registry][].

Options:

- `--branch=branchName`: Branch of the V8 repository to use for the upgrade.
  Defaults to `lkgr`.

- `--no-version-bump`: Disable automatic bump of the `NODE_MODULE_VERSION`
  constant.

### `git node v8 minor`

Compare current V8 version with latest upstream of the same major. Applies a
patch if necessary.  
If the `git apply` command fails, a patch file will be written in the Node.js
clone directory.

### `git node v8 backport <sha..>`

Fetches and applies the patch corresponding to `sha`. Multiple commit SHAs can
be provided to this command. Increments the V8 embedder version number or patch
version and commits the changes for each commit (unless the command is
called with `--squash`). If a patch fails to be applied, the command will pause
and let you fix the conflicts in another terminal.

Options:

- `--no-bump`: Set this flag to skip bumping the V8 embedder version number or
  patch version.
- `--squash`: Set this flag to squash multiple commits into one. This should
  only be done if individual commits would break the build.

### General options

- `--node-dir=/path/to/node`: Specify the path to the Node.js git repository.
  Defaults to current working directory.
- `--base-dir=/path/to/base/dir`: Specify the path where V8 the clone will
  be maintained. Defaults to `~/.update-v8`.
- `--v8-dir=/path/to/v8/`: Specify the path of an existing V8 clone. This
  will be used instead of cloning V8 to `baseDir`.
- `--verbose`: Enable verbose output.

## `git node wpt`

Update or patch the Web Platform Tests in core.
The updated files are placed under `./test/fixtures/wpt` by default. In addition
to the assets, this also updates:

- `./test/fixtures/wpt/versions.json`
- `./test/fixtures/wpt/README.md`
- `./test/fixtures/wpt/LICENSE.md`

### Example

```
$ cd /path/to/node/project
$ git node wpt url  # Will update test/fixtures/wpt/url and related files
```

[node.js abi version registry]: https://github.com/nodejs/node/blob/master/doc/abi_version_registry.json
