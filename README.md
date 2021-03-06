# SwiftLint

A tool to enforce Swift style and conventions, loosely based on
[GitHub's Swift Style Guide](https://github.com/github/swift-style-guide).

SwiftLint hooks into [Clang](http://clang.llvm.org) and
[SourceKit](http://www.jpsim.com/uncovering-sourcekit) to use the
[AST](http://clang.llvm.org/docs/IntroductionToTheClangAST.html) representation
of your source files for more accurate results.

![Test Status](https://travis-ci.org/realm/SwiftLint.svg?branch=master)
[![codecov.io](https://codecov.io/github/realm/SwiftLint/coverage.svg?branch=master)](https://codecov.io/github/realm/SwiftLint?branch=master)

![](assets/screenshot.png)

## Installation

Using [Homebrew](http://brew.sh/)

```
brew install swiftlint
```

You can also install SwiftLint by downloading `SwiftLint.pkg` from the
[latest GitHub release](https://github.com/realm/SwiftLint/releases/latest) and
running it.

You can also build from source by cloning this project and running
`git submodule update --init --recursive; make install` (Xcode 7.1).

## Usage

### Xcode

Integrate SwiftLint into an Xcode scheme to get warnings and errors displayed
in the IDE. Just add a new "Run Script Phase" with:

```bash
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```

![](assets/runscript.png)

#### Format on Save Xcode Plugin

To run `swift autocorrect` on save in Xcode, install the
[SwiftLintXcode](https://github.com/ypresto/SwiftLintXcode) plugin from Alcatraz.

### Atom

To integrate SwiftLint with [Atom](https://atom.io/) install the
[`linter-swiftlint`](https://atom.io/packages/linter-swiftlint) package from
APM.

### Command Line

```
$ swiftlint help
Available commands:

   autocorrect  Automatically correct warnings and errors
   help         Display general or command-specific help
   lint         Print lint warnings and errors for the Swift files in the current directory (default command)
   rules        Display the list of rules and their identifiers
   version      Display the current version of SwiftLint
```

Run `swiftlint` in the directory containing the Swift files to lint. Directories
will be searched recursively.

To specify a list of files when using `lint` or `autocorrect` (like the list of
files modified by Xcode specified by the
[`ExtraBuildPhase`](https://github.com/norio-nomura/ExtraBuildPhase) Xcode
plugin, or modified files in the working tree based on `git ls-files -m`) you
can do so by passing the option `--use-script-input-files` and setting the
following instance variables: `SCRIPT_INPUT_FILE_COUNT` and
`SCRIPT_INPUT_FILE_0`, `SCRIPT_INPUT_FILE_1`... `SCRIPT_INPUT_FILE_{SCRIPT_INPUT_FILE_COUNT}`.

These are same environment variables set for input files to
[custom Xcode script phases](http://indiestack.com/2014/12/speeding-up-custom-script-phases/).

## Rules

There are only a small number of rules currently implemented, but we hope the
Swift community (that's you!) will contribute more over time. Pull requests are
encouraged.

The rules that *are* currently implemented are mostly there as a starting point
and are subject to change.

See the [Source/SwiftLintFramework/Rules](Source/SwiftLintFramework/Rules)
directory to see the currently implemented rules.

### Disable a rule in code

Rules can be disabled with a comment inside a source file with the following
format:

`// swiftlint:disable <rule>`

The rule will be disabled until the end of the file or until the linter sees a
matching enable comment:

`// swiftlint:enable <rule>`

For example:

```swift
// swiftlint:disable colon
let noWarning :String = "" // No warning about colons immediately after variable names!
// swiftlint:enable colon
let hasWarning :String = "" // Warning generated about colons immediately after variable names
```

It's also possible to modify a disable or enable command by appending
`:previous`, `:this` or `:next` for only applying the command to the previous,
this (current) or next line respectively.

For example:

```swift
// swiftlint:disable:next force_cast
let noWarning = NSNumber() as! Int
let hasWarning = NSNumber() as! Int
let noWarning2 = NSNumber() as! Int // swiftlint:disable:this force_cast
let noWarning3 = NSNumber() as! Int
// swiftlint:disable:previous force_cast
```

Run `swiftlint rules` to print a list of all available rules and their
identifiers.

### Configuration

Configure SwiftLint by adding a `.swiftlint.yml` file from the directory you'll
run SwiftLint from. The following parameters can be configured:

Rule inclusion:

* `disabled_rules`: Disable rules from the default enabled set.
* `opt_in_rules`: Some rules are opt-in.
* `whitelist_rules`: Can not be specified alongside `disabled_rules` or
  `opt_in_rules`. Acts as a whitelist, only the rules specified in this list
  will be enabled.

```yaml
disabled_rules: # rule identifiers to exclude from running
  - colon
  - comma
  - control_statement
opt_in_rules: # some rules are only opt-in
  - empty_count
  - missing_docs
  # Find all the available rules by running:
  # swiftlint rules
included: # paths to include during linting. `--path` is ignored if present.
  - Source
excluded: # paths to ignore during linting. Takes precedence over `included`.
  - Carthage
  - Pods
  - Source/ExcludedFolder
  - Source/ExcludedFile.swift

# configurable rules can be customized from this configuration file
# binary rules can set their severity level
force_cast: warning # implicitly
force_try:
  severity: warning # explicitly
# rules that have both warning and error levels, can set just the warning level
# implicitly
line_length: 110
# they can set both implicitly with an array
type_body_length:
  - 300 # warning
  - 400 # error
# or they can set both explicitly
file_length:
  warning: 500
  error: 1200
# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
type_name:
  min_length: 4 # only warning
  max_length: # warning and error
    warning: 40
    error: 50
  excluded: iPhone # excluded via string
variable_name:
  min_length: # only min_length
    error: 4 # only error
  excluded: # excluded via string array
    - id
    - URL
    - GlobalAPIKey
reporter: "xcode" # reporter type (xcode, json, csv, checkstyle)
```

#### Defining Custom Rules

You can define custom regex-based rules in you configuration file using the
following syntax:

```yaml
custom_rules:
  pirates_beat_ninjas: # rule identifier
    name: "Pirates Beat Ninjas" # rule name. optional.
    regex: "([n,N]inja)" # matching pattern
    match_kinds: # SyntaxKinds to match. optional.
      - comment
      - identifier
    message: "Pirates are better than ninjas." # violation message. optional.
    severity: error # violation severity. optional.
  no_hiding_in_strings:
    regex: "([n,N]inja)"
    match_kinds: string
```

This is what the output would look like:

![](assets/custom-rule.png)

You can filter the matches by providing one or more `match_kinds`, which will
reject matches that include syntax kinds that are not present in this list. Here
are all the possible syntax kinds:

* argument
* attribute.builtin
* attribute.id
* buildconfig.id
* buildconfig.keyword
* comment
* comment.mark
* comment.url
* doccomment
* doccomment.field
* identifier
* keyword
* number
* objectliteral
* parameter
* placeholder
* string
* string_interpolation_anchor
* typeidentifier

#### Nested Configurations

SwiftLint supports nesting configuration files for more granular control over
the linting process.

* Set the `use_nested_configs: true` value in your root `.swiftlint.yml` file
* Include additional `.swiftlint.yml` files where necessary in your directory
  structure.
* Each file will be linted using the configuration file that is in its
  directory or at the deepest level of its parent directories. Otherwise the
  root configuration will be used.
* `excluded`, `included`, and `use_nested_configs` are ignored for nested
  configurations.

### Auto-correct

SwiftLint can automatically correct certain violations. Files on disk are
overwritten with a corrected version.

Please make sure to have backups of these files before running
`swiftlint autocorrect`, otherwise important data may be lost.

Standard linting is disabled while correcting because of the high likelihood of
violations (or their offsets) being incorrect after modifying a file while
applying corrections.

## License

MIT licensed.
