# rossum
v0.1.4

This is `rossum`, a CMake-like build file generator for Fanuc Robotics (Karel)
projects.


## Overview

This tool introduces a package based workflow for Karel development: packages
are directories that contain a marker file (a *manifest*) that contains some
metadata describing the dependencies of that package and any translatable
targets. Detection of packages and generation of the build file is done once
at configuration time, relying on the build tool's dependency resolution to
build all targets in the correct order.


## Requirements

`rossum` is written in Python 2, so naturally it needs a Python 2 install.

The generated Ninja build files require a recent (> 1.7.1) version of [Ninja][]
to be present.

For translating Karel sources, `rossum` expects [ktransw][] version 0.2.2 or
newer. Refer to the `ktransw` documentation for any additional requirements
that `ktransw` may have.

Finally, `rossum` uses [EmPy][] for transforming the build file template to the
actual build files.


## Installation

The 0.1.4 release is a convenience release that includes all the necessary
tools to quickly setup a working installation of rossum. Download it from the
[releases][] page.

Alternatively, clone this repository to your machine and add the directory
containing `rossum.py` and `rossum.cmd` to your `PATH`. Command sessions
opened after setting up the `PATH` should be able to successfully run `rossum`
from anywhere.

For installation of `ktransw`, see [ktransw][].

For maximum convenience, make sure that `ninja.exe` and `ktransw.cmd` are also
on the `PATH`. An alternative would be to copy `ninja.exe` to the build
directory of the workspace (the one containing the generated build file) and
to specify the path to `ktransw.cmd` as a command line argument to `rossum`.
Note that this would have to be repeated each time a new build directory is
created.

See [Installing Python on Windows][] in the HHGtP for information on how to
install Python on Windows.

[EmPy][] can be installed with `pip` with `pip install empy` in a Windows
command shell session. Usage and installation of `pip` is covered in
[Installing Python on Windows][].


## Usage

```
usage: rossum [-h] [-v] [-V] [-q] [-c ID] [--support PATH] [-d]
              [--ktrans PATH] [--ktransw PATH] [-n] [-p PATH] [-r INI] [-w]
              SRC [BUILD]

Version 0.1.4

A cmake-like Makefile generator for Fanuc Robotics (Karel) projects
that supports out-of-source builds.

positional arguments:
  SRC                   Main directory with packages to build
  BUILD                 Directory for out-of-source builds (default: 'cwd')

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Be verbose
  -V, --version         show program's version number and exit
  -q, --quiet           Be quiet (only warnings and errors will be shown)
  -c, --core            Version of the core files used when translating
                        (default: V7.70-1)
  --support             Location of KAREL support directory (default: auto-
                        detect based on selected core version and FANUC
                        registry keys)
  -d, --dry-run         Do everything except writing to Makefile
  --ktrans              Location of ktrans (default: auto-detect)
  --ktransw             Location of ktransw (default: assume it's on the
                        Windows PATH)
  -n, --no-env          Do not search the KPKG_PATH, even if it is set
  -p, --pkg-dir         Additional paths to search for packages (multiple
                        allowed)
  -r, --robot-ini       Location of robot.ini (default: source dir)
  -w, --overwrite       Overwrite any Makefile that may exist in the build dir

Usage example:

  mkdir C:\foo\bar\build
  cd C:\foo\bar\build
  rossum C:\foo\bar\src
```


## Examples

Please see [rossum_example_ws][] for an example workspace with some packages
that show how to use `rossum`.


## Glossary

Terminology used by `rossum`.

### package
A directory containing any number of source and header files, organised into
`src` and `include` sub directories, along with a JSON package manifest. The
manifest declares translationable binary targets and additional meta-data such
as build dependencies (other `rossum` packages), author, version and test
binaries.

### workspace
A directory containing a set of `rossum` compatible packages. Note that while
possible, packages are typically not directly stored in the top-level workspace
directory, but in a *source space* sub directory.

### source space
The sub directory of the *workspace* that contains a set of of `rossum`
compatible packages. Usually named `src`.

### build space
The sub directory of the *workspace* that will store all the build output
(p-code files).

### build file
The ninja build file generated by `rossum` at the end of the configuration
phase.


## FAQ

#### Does this run on Windows?
Yes, it only runs on Windows, actually.

#### Is Roboguide (still) needed?
`rossum` only generates build files, it does not replace `ktrans` or Roboguide,
so depending on your project's requirements (is it Karel only? Do you need to
translate TP programs, etc), yes, you still need Roboguide.

#### Does this work with just WinOLPC or OlpcPRO?
I haven't tested it explicitly, but it should work. Auto-detection of the
location of `ktrans` will probably need some work. If you are willing to assist,
please contact me.

#### I pointed rossum to my Roboguide workcell directory, but it doesn't work
`rossum` only recognises directories that contain a so called *manifest*: a
JSON file that describes what `rossum` should do with the files in the package.
It doesn't currently understand Roboguide workcells (`.frw` and others), but
that may change in future versions.

#### Copying a build file to another build directory doesn't work
`rossum` generates build files with absolute paths. Moving the build file to
another directory is possible, but compilation artefacts (`.pc`) will still
be placed in the build directory that was used when generating the file.

This may change in future versions (see `TODO.md`).

#### How do I translate my sources for a different core version?
`rossum` by default will configure the build file to use the `V7.70-1` version
of the system core files. This can be changed at generation time using the 
`--core` command line option for setting a default that will persist as long
as the build file remains unchanged.

See the output of `ktrans /?` for a list of supported core versions.

#### I don't want to open-source my Karel projects, can I still use this?
Of course: you are not required to open-source anything. The licenses used
in the [rossum_example_ws][] workspace are just examples. `rossum` does not
use the information in the `license` key at this time, so there are no
restrictions on the values allowed there.

#### Why use JSON for the manifests, and not YAML/xml/X?
I wanted to keep `rossum` as easy to install as possible, and also not burden
users (developers) with having to create/maintain too much metadata. Of the
file formats supported by the Python Standard Library, JSON seemed like a
good fit with the requirements for the manifests: hierarchical, support for
key-value pairs, lists and easy to edit. PyYAML needs to be installed on
Windows, and xml is a bit too verbose (and easy to get wrong when editing).

Depending on the needs, `rossum` might move to YAML in the future.

#### Your code is a mess
Yes. As is often the case, `rossum` was basically needed *yesterday*, and as
such hasn't received the attention and thought it deserve(s)(d). Refactoring
is on the `TODO.md` list though.


## Disclaimer

WinOLPC, OlpcPRO and Roboguide are products of Fanuc America Corporation. The
author of `rossum` is not affiliated with Fanuc in any way.



[ninja]: https://ninja-build.org
[ktransw]: https://github.com/gavanderhoorn/ktransw_py
[EmPy]: https://pypi.python.org/pypi/EmPy
[releases]: https://github.com/gavanderhoorn/rossum/releases
[rossum_example_ws]: https://github.com/gavanderhoorn/rossum_example_ws
[Installing Python on Windows]: http://docs.python-guide.org/en/latest/starting/install/win/
