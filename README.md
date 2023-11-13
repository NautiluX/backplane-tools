# backplane-tools

backplane-tools offers an easy solution to install, remove, and upgrade a useful set of tools for interacting with OpenShift clusters, managed or otherwise.

## Table of Contents
<!-- toc -->
- [Tools](#tools)
- [FAQ - How do I...](#faq-how-do-i)
  - [Install backplane-tools for the first time](#install-backplane-tools-for-the-first-time)
    - [1. Bash one-liner](#1-bash-oneliner)
    - [2. Get, verify, and extract the release](#2-get-verify-and-extract-the-release)
    - [3. Bootstrap the application](#3-bootstrap-the-application)
    - [4. (Recommended) Add the tools to my $PATH](#4-recommended-add-the-tools-to-my-path)
    - [5. (Recommended) Cleanup](#5-recommended-cleanup)
  - [List available tools](#list-available-tools)
  - [List installed tools](#list-installed-tools)
  - [Install everything](#install-everything)
  - [Install a specific thing](#install-a-specific-thing)
  - [Upgrade everything](#upgrade-everything)
  - [Upgrade a specific thing](#upgrade-a-specific-thing)
  - [Remove everything](#remove-everything)
  - [Remove a specific thing](#remove-a-specific-thing)
- [Design](#design)
  - [Directory Structure](#directory-structure)
  - [Installing](#installing)
  - [Upgrading](#upgrading)
  - [Removing](#removing)
<!-- tocstop -->

## Tools
The tools currently managed by this application include:

* aws
* backplane-cli
* ocm
* osdctl
* rosa

## FAQ - How do I...
Quick reference guide

### Install backplane-tools for the first time

#### 1. Bash one-liner

Install backplane-tools with the following command:

```shell
go install github.com/openshift/backplane-tools; backplane-tools install all
```

Follow instructions to add tools to `PATH`.

#### 2. Get, verify, and extract the release
Download the [latest release](https://github.com/openshift/backplane-tools/releases/latest) for your architecture & operating system, as well as the release's checksum file. Verify the release integrity with:
```shell
sha256sum --check --ignore-missing backplane-tools_${RELEASE_TAG}_checksums.txt
```
where `RELEASE_TAG` is set to the latest release version (ie - `v0.1.0`). If the release file you've downloaded reports an `OK` status, then proceed with installation. Otherwise, retry the download.

Extract the `backplane-tools` asset file into a dedicated directory for easy cleanup:
```shell
mkdir -p backplane-tools_${RELEASE_TAG}/
tar -xzvf backplane-tools_${RELEASE_TAG}_${OS}_${ARCH}.tar.gz -C backplane-tools_${RELEASE_TAG}/
```
where `OS` is set to your local operating system (`linux`, `darwin`) and `ARCH` is set to your system architecture (`arm64`, `amd64`).

#### 3. Bootstrap the application
Run the following to bootstrap the application:
```shell
./backplane-tools install backplane-tools
```
If you've never installed `backplane-tools` before, you will likely see a warning recommending you add additional entries to your `$PATH`; see [the next section](#3-recommended-add-the-tools-to-my-path) for steps to address this warning.

#### 4. (Recommended) Add the tools to my $PATH
> [!NOTE]
> By default, backplane-tools and the applications it manages are not added to your `$PATH`. This avoids making assumptions on how users prefer to set their `$PATH` (ie- some users may prefer to `source` a different configuration file in their shell's .rc file), and, consequently, avoid unintentional collisions between the tools backplane-tools manages and those the user self-manages. This way, if you one day wish to take over the management of a certain utility, you can easily choose to invoke that instead by placing its location earlier on your `$PATH` than backplane-tools' entry.

Add the following line to your shell's .rc file (`.bashrc`, `.zshrc`, etc):
```shell
export PATH=${PATH}:${HOME}/.local/bin/backplane/latest
```

After updating your .rc file, restart your shell (ie - close + reopen the terminal window), then confirm installation succeeded by running:

```shell
backplane-tools list installed
```

If the command was able to execute and lists `backplane-tools` as one of the installed applications, then you can safely cleanup the release file and its contents, if you so desire.

#### 5. (Recommended) Cleanup
Once `backplane-tools` successfully installs itself, you can safely remove the release file, its checksum, and its contents:

```shell
rm -rf backplane-tools_${RELEASE_TAG}*
```

### List available tools
```shell
backplane-tools list available
```

### List installed tools
```shell
backplane-tools list installed
```

### Install everything
```shell
backplane-tools install all
```
or
```shell
backplane-tools install
```

### Install a specific thing
```shell
backplane-tools install <tool name>
```

### Upgrade everything
```shell
backplane-tools upgrade all
```

> [!WARNING]
> Using `backplane-tools upgrade all` is the same as running `backplane-tools install all`. See the [Upgrading](#upgrading) design section for more details.

### Upgrade a specific thing
```shell
backplane-tools upgrade <tool name>
```

### Remove everything
```shell
backplane-tools remove all
```

### Remove a specific thing
```shell
backplane-tools remove <tool name>
```

## Design

backplane-tools strives to be simplistic and non-invasive; it should not conflict with currently installed programs, nor should it require extensive research before operating.

### Directory structure

The following diagram summarizes the directory structure backplane-tools generates when managing tools on the local filesystem:
```
                                    $HOME/
                                      |
                                      V
                                   .local/
                                      |
                                      V
                                     bin/
                                      |
                                      V
                                  backplane/
                                      |
            -------------------------------------------------------------
           |                          |                                  |
           V                          V                                  V
        latest/                     toolA/                             toolB/ 
           |                          |                                  |
      ---------             ---------------------              ----------------//--
     |         |           |                     |            |          |         |
     V         V           V                     V            V          V         V
   linkToA linkToB     version1/             version2/    version0.1/   ...    version10.0/
                           |                     |            |                    |
                           V                     V            V                    V
                       ---------             ---------       ...              -----------
                      |         |           |         |                      |           |
                      V         V           V         V                      V           V
                  executable  README    executable* README                executable*  README

* = linked to the latest/ directory
```

When installing a new tool, backplane-tools will create `$HOME/.local/bin/backplane/` to hold the files managed by the application, if it does not already exist. **To avoid conflicts with other dependency management systems, all actions taken by backplane-tools are confined to this directory.** This means that backplane-tools can be used safely alongside your system's normal package manager, 3rd party managers like flatpak or snap, and language-specific tools like pip or `go install`.

Next, backplane-tools creates a subdirectory `$HOME/.local/bin/backplane/latest/`; within which users will find links to the latest executables for each tool installed. In order to most effectively utilize backplane-tools, it's recommended this directory is added to your environment's `$PATH`, however there is no requirement to do so in order to utilize the application.

Finally, subdirectories are added as `$HOME/.local/bin/backplane/<tool name>/` for each tool being installed, if one does not already exist. Here, backplane-tools stores the version-specific data and files needed to execute each program. How these tool-directories are organized depends on the tool itself, but generally each tool will contain one or more "versioned-directories". Each versioned-directory contains a complete installation of the tool, at the version the directory is named after. These versioned-directories are not removed during installation or upgrade, thus, if a recently upgraded tool contains incompatabilities or bugs, a previous version can still be utilized.

### Installing
When installing a new tool, backplane-tools creates a basic structure as described in [the above section](#directory-structure): a parent directory containing a `latest/` and one or more `<tool name>/` subdirectories. Within the tool directories, it downloads, unpacks, checksums, and installs the requested tool of the same name. Because the tools are downloaded from their respective sources (usually GitHub), and *not* a centralized service, installation logic must be crafted specifically for each tool. 

Despite the risks this places on maintainability, in practice, tools have been found to rarely change their distribution strategy. This means that, once in place, little upkeep has been required thus far. Conversely, the benefit of this design lies in it's lack of infrastructure requirements; there aren't any servers to administer or packages to maintain. This lends the tool to easy contribution or forking: in order to add a desired tool, one only needs to add the relevant logic to backplane-tools.

Finally, after performing the necessary steps to install a new version of the tool, the tool's executable is symlinked to the `$HOME/.local/bin/backplane/latest/` directory, so that it can be easily invoked with the latest versions of other tools being managed by the application.

### Upgrading
At present, upgrading is the exact same as installing. This means if you run `backplane-tools upgrade all` - you will find that all tools that backplane-tools manages will now be installed on your system.

### Removing
Users are able to remove individual tools or completely remove all files and data managed by backplane-tools.

`backplane-tools remove <toolA> <toolB> ...` allows users to remove a specific set of tools from their system. This is done by removing the tool-specific directory at `$HOME/.local/bin/backplane/<tool name>`, as well as the tool's linked executable in `$HOME/.local/bin/backplane/latest/`.

`backplane-tools remove all` allows users to remove everything managed by backplane-tools. This is done by completely removing `$HOME/.bin/local/backplane/`. Subsequent calls to `backplane-tools install` will cause the directory structure to be recreated from scratch.
