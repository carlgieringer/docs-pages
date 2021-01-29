---
hide:
  - toc
---

# Advanced installation guide, tips and tricks

<div markdown="1" class="coreconcepts-intro">
This guide assumes that you've already followed the [quick installation guide](../install/) and want something more. It describes how to make it faster to get into the Holonix shell, explains how to install more or less stable versions of Holochain, and discusses why we use nix-shell in the first place.
</div>

## Getting into Holonix faster and easier

You might find it tedious to try to remember the command that gets you back into the Holonix development shell. `nix-shell ~/.holonix/shellDrv` isn't that intuitive, and when you're trying to open multiple terminals at a time for testing purposes it could get annoying.

To save keystrokes, add an alias to your shell config:

```bash
echo 'alias holonix=\'nix-shell ~/.holonix/shellDrv\' >> ~/.bashrc
holonix
```

Close your terminal window, open it again, and you should be able to type `holonix` from now on to get into the shell.

### Using your favorite text editor or IDE

In most cases you can run your editor as normal. However, if you are using a text editor or integrated development environment (IDE) that needs to communicate with the Rust compiler for real-time syntax checks, then you should launch it from inside the nix-shell. This is because Holonix comes with its own version of Rust that might be different from what you may already have installed.

To do this, just open your editor from the command line while you are in the nix-shell (this example uses Vim):

```bash
nix-shell ~/.holonix/shellDrv
cd my_project
vim my_file.rs
```

## Upgrading

Any time you want to get the latest version of the dev tools, you can [follow the install procedure](#installing-the-holochain-dev-tools) again and it'll update you.

### The faster and easier method

Once again, this is a hard command to remember, so you might want to add an alias for it as well:

#### Linux (or Windows 10 with WSL2)

```bash
echo 'alias holonix-update=\'$(nix-build https://holochain.love --no-link -A pkgs.holonix)/bin/holonix\'' >> ~/.bashrc
holonix-update
```

#### macOS

```bash
echo 'alias holonix-update=\'HN_NOSUDO=true $(nix-build https://holochain.love --no-link -A pkgs.holonix)/bin/holonix\'' >> ~/.bashrc
holonix-update
```

## Staying up to date all the time

There's a trick you can use to keep yourself up to date with the newest release of all the Holochain tools. It takes a bit longer whenever there's something new, though, because it has to compile everything from source.

Every time you want to enter the Holonix development environment, use this command:

```bash
nix-shell https://holochain.love
```

## Uninstalling

You usually don't need to uninstall anything, because `nix-shell` leaves your familiar user environment alone and makes all of its own changes disappear once you exit the shell. But it does keep binaries on your device. If you want to free up some space, you can delete the contents of `/nix/store` --- just make sure nobody or nothing else is using Nix on your computer first!

## Using a specific version of the development tools

Sometimes you might want to target a specific version of Holochain --- for instance, when you're working on an application that was compiled with an older version of the <abbr title="Holochain development kit">HDK</abbr> that's incompatible with the version of Holochain you have installed. This is where Nix really shines: you can set up an environment containing all the tools that target the older version while leaving your current installation alone. Simply [find the release tag](https://github.com/holochain/holochain/tags) of the version of Holochain you're looking for (we don't yet have release tags, but that will change soon) and enter this command in your project's working folder:

```bash
nix-shell https://github.com/holochain/archive/<tag>.tar.gz
```

Note that Nix will have to compile all the binaries from source, so it'll take a long time. You'll also have to enter this same command every time you want that specific version (although it won't have to recompile on next run).

### Keeping everything local

If you find yourself needing to switch versions often, a faster method is to clone the Holochain repository, check out the release you want, and enter the nix shell from there:

```bash
git clone git@github.com:holochain/holochain.git
cd holochain
git checkout <tag>
nix-shell
```

Once you're in the shell, you can navigate to your project's working folder. This way you have all the versions of Holochain available to you locally and don't need to remember long URLs.

## More Info on Nix

We use the Nix/NixOS toolkit to build consistent development, testing, and deployment environments for Holochain Core and apps. It consists of two systems:

* NixOS, a tool for reliably building Linux-based systems from a set of configuration files (we use NixOS in our HoloPorts and automated testing VMs)
* Nix, a package manager that works on many OSes and uses the same configuration file format as NixOS

The main components of the tooling for Holochain development are:

* The [Rust](https://rust-lang.org) programming language
* [Node.JS](https://nodejs.org) and [npm](https://npmjs.com)
* Cryptographic libraries
* Common automations and scripts

It is important that these remain consistent across compatible apps and the Holochain Core, so you can get your work done without fighting package and compiler issues. And when it comes time to compile and distribute your application, it's **very important to have a deterministic build system** so the same DNA source code always results in the same hash.

The main Nix tool used in Holochain development workflows is `nix-shell`, a managed Bash shell that overlays a new environment and set of tools on top of your existing environment.

The full suite of Nix tooling is broad and deep. There’s even a dedicated programming language. Learn more with the [NixOS Wiki](https://nixos.wiki/wiki/Main_Page) or the [Pills](https://nixos.org/nixos/nix-pills/) Tutorial. The community IRC chat at `#nixos` on freenode is active and helpful.

### nix-shell

While working on Holochain, you will usually have an active `nix-shell` to run commands. This shell overlays Holochain-specific configuration on top of your existing shell---environment variables, Rust toolchains, binaries, libraries, and development tools---giving you a consistent development environment to build Holochain apps. All this setup will be cleaned up automatically when you close the shell.

If you want to re-enter the shell to do more work, or create multiple terminals to work in, you'll need to re-enter the `nix-shell`. The packages are built and cached locally on your machine, so they will not be rebuilt the next time you enter the shell. You do need to get the package configuration files from somewhere, though. If you use our quick install guide or the [Holochain repo cloning method](#keeping-everything-local), they're cached on your machine, but the ['staying up to date all the time'](#staying-up-to-date-all-the-time) and ['using a specific version'](#using-a-specific-version-of-the-development-tools) methods require an internet connection every time you want to enter the shell.