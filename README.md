<p align="center">
  <h1 align="center">sv4git</h1>
  <p align="center">semantic version for git</p>
  <p align="center">
    <a href="https://github.com/bvieira/sv4git/releases/latest"><img alt="Release" src="https://img.shields.io/github/release/bvieira/sv4git.svg?style=for-the-badge"></a>
    <a href="https://pkg.go.dev/github.com/bvieira/sv4git/v2"><img alt="Go Reference" src="https://img.shields.io/badge/-Reference-blue?style=for-the-badge&logo=go&labelColor=gray"></a>
    <a href="https://github.com/bvieira/sv4git/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/bvieira/sv4git?style=for-the-badge"></a>
    <a href="/LICENSE"><img alt="Software License" src="https://img.shields.io/badge/license-MIT-informational.svg?style=for-the-badge"></a>
    <a href="https://github.com/bvieira/sv4git/actions?workflow=ci"><img alt="GitHub Actions Status" src="https://img.shields.io/github/workflow/status/bvieira/sv4git/ci?style=for-the-badge"></a>
    <a href="https://goreportcard.com/report/github.com/bvieira/sv4git"><img alt="Go Report Card" src="https://goreportcard.com/badge/github.com/bvieira/sv4git?style=for-the-badge"></a>
    <a href="https://conventionalcommits.org"><img alt="Conventional Commits" src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-informational.svg?style=for-the-badge"></a>
  </p>
</p>

## Getting Started

### Pre Requirements

- Git 2.17+

### Installing

- Download the latest release and add the binary to your path.
- Optional: Set `SV4GIT_HOME` to define user configs. Check the [Config](#config) topic for more information.

If you want to install from source using `go install`, just run:

```bash
go install github.com/bvieira/sv4git/v2/cmd/git-sv@latest
```

### Config

There are 3 config levels when using sv4git: [default](#default), [user](#user), [repository](#repository). All of them are merged considering the follow priority: **repository > user > default**.

To see the current config, run:

```bash
git sv cfg show
```

#### Configuration Types

##### Default

To check the default configuration, run:

```bash
git sv cfg default
```

##### User

For user config, it is necessary to define the `SV4GIT_HOME` environment variable, eg.:

```bash
SV4GIT_HOME=/home/myuser/.sv4git # myuser is just an example.
```

And create a `config.yml` file inside it, eg.:

```bash
.sv4git
└── config.yml
```

##### Repository

Create a `.sv4git.yml` file on the root of your repository, eg.: [.sv4git.yml](.sv4git.yml).

#### Configuration format

```yml
version: "1.0" #config version

versioning: # versioning bump
    update-major: [] # Commit types used to bump major.
    update-minor: [feat] # Commit types used to bump minor.
    update-patch: [build, ci, chore, fix, perf, refactor, test] # Commit types used to bump patch.
    # When type is not present on update rules and is unknown (not mapped on commit message types);
    # if ignore-unknown=false bump patch, if ignore-unknown=true do not bump version
    ignore-unknown: false

tag:
    pattern: '%d.%d.%d' # Pattern used to create git tag.

release-notes:
    # Headers names for release notes markdown. To disable a section just remove the header line.
    # It's possible to add other commit types, the release note will be created respecting the following order:
    # feat, fix, refactor, perf, test, build, ci, chore, docs, style, breaking-change
    headers: 
        breaking-change: Breaking Changes
        feat: Features
        fix: Bug Fixes

branches: # Git branches config.
    prefix: ([a-z]+\/)? # Prefix used on branch name, it should be a regex group.
    suffix: (-.*)? # Suffix used on branch name, it should be a regex group.
    disable-issue: false # Set true if there is no need to recover issue id from branch name.
    skip: [master, main, developer] # List of branch names ignored on commit message validation.
    skip-detached: false # Set true if a detached branch should be ignored on commit message validation.

commit-message:
    types: [build, ci, chore, docs, feat, fix, perf, refactor, revert, style, test] # Supported commit types.
    scope:
        # Define supported scopes, if blank, scope will not be validated, if not, only scope listed will be valid.
        # Don't forget to add "" on your list if you need to define scopes and keep it optional.
        values: []
    footer:
        issue: # Use "issue: {}" if you wish to disable issue footer.
            key: jira # Name used to define an issue on footer metadata.
            key-synonyms: [Jira, JIRA] # Supported variations for footer metadata.
            use-hash: false # If false, use :<space> separator. If true, use <space># separator.
            add-value-prefix: '' # Add a prefix to issue value.
    issue:
        regex: '[A-Z]+-[0-9]+' # Regex for issue id.
```

### Running

Run `git-sv` to get the list of available parameters:

```bash
git-sv
```

#### Run as git command

If `git-sv` is configured on your path, you can use it like a git command:

```bash
git sv
git sv current-version
git sv next-version
```

#### Usage

Use `--help` or `-h` to get usage information, don't forget that some commands have unique options too:

```bash
# sv help
git-sv -h

# sv release-notes command help
git-sv rn -h
```

##### Available commands

| Variable                     | description                                                    | has options or subcommands |
| ---------------------------- | -------------------------------------------------------------- | :------------------------: |
| config, cfg                  | Show config information.                                       |     :heavy_check_mark:     |
| current-version, cv          | Get last released version from git.                            |            :x:             |
| next-version, nv             | Generate the next version based on git commit messages.        |            :x:             |
| commit-log, cl               | List all commit logs according to range as jsons.              |     :heavy_check_mark:     |
| commit-notes, cn             | Generate a commit notes according to range.                    |     :heavy_check_mark:     |
| release-notes, rn            | Generate release notes.                                        |     :heavy_check_mark:     |
| changelog, cgl               | Generate changelog.                                            |     :heavy_check_mark:     |
| tag, tg                      | Generate tag with version based on git commit messages.        |            :x:             |
| commit, cmt                  | Execute git commit with convetional commit message helper.     |     :heavy_check_mark:     |
| validate-commit-message, vcm | Use as prepare-commit-message hook to validate commit message. |     :heavy_check_mark:     |
| help, h                      | Shows a list of commands or help for one command.              |            :x:             |

##### Use range

Commands like `commit-log` and `commit-notes` has a range option. Supported range types are: `tag`, `date` and `hash`.

By default, it's used [--date=short](https://git-scm.com/docs/git-log#Documentation/git-log.txt---dateltformatgt) at `git log`, all dates returned from it will be in `YYYY-MM-DD` format.

Range `tag` will use `git for-each-ref refs/tags` to get the last tag available if `start` is empty, the others types won't use the existing tags. It's recommended to always use a start limit in a old repository with a lot of commits. This behavior was maintained to not break the retrocompatibility.

Range `date` use git log `--since` and `--until`. It's possible to use all supported formats from [git log](https://git-scm.com/docs/git-log#Documentation/git-log.txt---sinceltdategt). If `end` is in `YYYY-MM-DD` format, `sv` will add a day on git log command to make the end date inclusive.

Range `tag` and `hash` are used on git log [revision range](https://git-scm.com/docs/git-log#Documentation/git-log.txt-ltrevisionrangegt). If `end` is empty, `HEAD` will be used instead.

```bash
# get commit log as json using a inclusive range
git-sv commit-log --range hash --start 7ea9306~1 --end c444318

# return all commits after last tag
git-sv commit-log --range tag
```

##### Use validate-commit-message as prepare-commit-msg hook

Configure your `.git/hooks/prepare-commit-msg`:

```bash
#!/bin/sh

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

git sv vcm --path "$(pwd)" --file "$COMMIT_MSG_FILE" --source "$COMMIT_SOURCE"
```

**Tip**: you can configure a directory as your global git templates using the command below:

```bash
git config --global init.templatedir '<YOUR TEMPLATE DIR>'
```

Check [git config docs](https://git-scm.com/docs/git-config#Documentation/git-config.txt-inittemplateDir) for more information!

## Development

### Makefile

Run `make` to get the list of available actions:

```bash
make
```

#### Make configs

| Variable   | description             |
| ---------- | ----------------------- |
| BUILDOS    | Build OS.               |
| BUILDARCH  | Build arch.             |
| ECHOFLAGS  | Flags used on echo.     |
| BUILDENVS  | Var envs used on build. |
| BUILDFLAGS | Flags used on build.    |

| Parameters | description                          |
| ---------- | ------------------------------------ |
| args       | Parameters that will be used on run. |

```bash
#variables
BUILDOS="linux" BUILDARCH="amd64" make build

#parameters
make run args="-h"
```

### Build

```bash
make build
```

The binary will be created on `bin/$BUILDOS_$BUILDARCH/git-sv`.

### Tests

```bash
make test
```

### Run

```bash
#without args
make run

#with args
make run args="-h"
```
