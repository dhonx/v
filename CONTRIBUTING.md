## Code Structure

I tried to make the code of the compiler and vlib as simple and readable as
possible.  One of V's goals is to be open to developers with different levels
of experience in compiler development. Compilers don't need to be black boxes
full of magic that only few people understand.

The V compiler is modular, and can be used by other applications. It is located
in `cmd/v/` and `vlib/v/`.

The most important and useful command to remember when working on the V compiler is
`v self`.
It rebuilds the V compiler.

Be careful, if you introduce a breaking change and rebuild V, you will no longer be able to use V to build
itself. So it's a good idea to make a backup copy of a working compiler executable.

But don't worry, you can always simply run `make`, it will download the C version of the compiler
and rebuild it from scratch.


The main files are:

1. `cmd/v/v.v`. The entry point.

- V figures out the build mode.
- Constructs the compiler object (`struct V`).
- Creates a list of .v files that need to be parsed.
- Creates a parser object for each file and runs `parse()` on them.
- The correct backend is called (C, JS, x64), and a binary is compiled.

2. `v/scanner` The scanner's job is to parse a list of characters and convert
them to tokens.

3. `v/token` This is simply a list of all tokens, their string values, and a
couple of helper functions.

4. `v/parser` The parser. It converts a list of tokens into an AST.
In V, objects can be used before declaration, so unknown types are marked as unresolved.
They are resolved later in the type checker.

5. `v/table` V creates one table object that is shared by all parsers. It
contains all types, consts, and functions, as well as several helpers to search
for objects by name, register new objects, modify types' fields, etc.

6. `v/checker`. Type checker and resolver. It processes the AST and makes sure
the types are correct. Unresolved types are resolved, type information is added
to the AST.

7. `v/gen` C backend. It simply walks the AST and generates C code that can be
compiled with Clang, GCC, Visual Studio, and TCC.

8. `json.v` defines the json code generation. This file will be removed once V
supports comptime code generation, and it will be possible to do this using the
language's tools.

9. `v/gen/x64` is the directory with all the machine code generation logic. It
defines a set of functions that translate assembly instructions to machine code
and build the binary from scratch byte by byte. It manually builds all headers,
segments, sections, symtable, relocations, etc. Right now it only has basic
support of the x64 platform/ELF format.

The rest of the directories are vlib modules: `builtin/` (strings, arrays,
maps), `time/`, `os/`, etc. Their documentation is pretty clear.

## Example Workflow for Contributing
(provided by [@spytheman](https://github.com/spytheman))

(If you don't already have a GitHub account, please create one. Your GitHub
username will be referred to later as 'YOUR_GITHUB_USERNAME'. Change it
accordingly in the steps below.)

1. Fork https://github.com/vlang/v using GitHub's interface to your own account.
Let's say that the forked repository is at
`https://github.com/YOUR_GITHUB_USERNAME/v` .
2. Clone the main v repository https://github.com/vlang/v to a local folder on your computer, say named nv/
(`git clone https://github.com/vlang/v nv`)
3. `cd nv`
4. `git remote add pullrequest https://github.com/YOUR_GITHUB_USERNAME/v`
NB: the remote named `pullrequest` should point to YOUR own forked repo, not the main v repository!
After this, your local cloned repository is prepared for making pullrequests,
and you can just do normal git operations such as: `git pull` `git status` and so on.

5. When finished with a feature/bugfix/change, you can:
`git checkout -b fix_alabala`
6. `git push pullrequest`  # (NOTE: the `pullrequest` remote was setup on step 4)
7. On GitHub's web interface, go to: https://github.com/vlang/v/pulls

Here the UI shows a dialog with a button to make a new pull request based on
the new pushed branch.
(Example dialog: https://url4e.com/gyazo/images/364edc04.png)
8. After making your pullrequest (aka, PR), you can continue to work on the
branch `fix_alabala` ... just do again `git push pullrequest` when you have more commits.
9. If there are merge conflicts, or a branch lags too much behind V's master,
you can do the following:
   1. `git pull --rebase origin master` # solve conflicts and do `git rebase --continue`
   2. `git push pullrequest -f` # this will overwrite your current remote branch with the updated version of your changes.

The point of doing the above steps, is to never directly push to the main V
repository, *only to your own fork*. Since your local `master` branch tracks the
main V repository's master, then `git checkout master`, as well as
`git pull --rebase origin master` will continue to work as expected
(these are actually used by `v up`) and git can always do it cleanly.

Git is very flexible, so there are other ways to accomplish the same thing.

## Using Github's hub CLI tool

You can download the `hub` tool from https://hub.github.com/ . Using
`hub`, you will not need to go through the (sometimes) slow website
to make PRs. Most remote operations can be done through the `hub` CLI
command.

NB: You still need to have a GitHub account.

### Preparation:
(steps 1..3 need to be done just *once*):

1. `hub clone vlang/v my_v`
2. `cd my_v`
3. `hub fork --remote-name pullrequest`

4. `git checkout -b my_cool_feature` # Step 4 is better done *once per each new feature/bugfix* that you make.

### Improve V by making commits:

5. `git commit -am "math: add a new function copysign"`

### Testing your commits locally:
You can test locally whether your changes have not broken something by
running: `v test-compiler`

### Publishing your commits to GitHub:

6. `git push pullrequest`

### Making a PR with `hub`:
(so that your changes can be merged to the main V repository)

7. `hub pull-request`

Optionally, you can track the status of your PR CI tests with:

8. `hub ci-status --verbose`

### Fixing failing tests:
If everything is OK, after 5-10 minutes, the CI tests should pass for
all platforms. If not, visit the URLs for the failing CI jobs, see
which tests have failed and then fix them by making more changes. Just use
`git push pullrequest` to publish your changes. The CI tests will
run with your updated code. Use `hub ci-status --verbose` to monitor
their status.
