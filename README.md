# reproducible-macports

This simple wrapper makes [MacPorts](https://www.macports.org/) reproducible and suitable for use as part of a wider git tree, so all your builds are tied to specific, versioned portfiles and distfiles.

To use, it, first add it to an existing git repository as a submodule, eg

```bash
# Create a git repository, if one does not already exist.
mkdir my-git-repository
cd my-git-repository
git init

# Install reproducible-macports
git submodule add https://github.com/raphaelcohn/reproducible-macports.git
./reproducible-macports/install-reproducible-macports
```

A folder, `reproducible-macports.conf`, is created. To use reproducible-macports, simply replace any usages of [MacPorts](https://www.macports.org/)'s `port` with `reproducible-port`. This folder is automatically added to git, and contains two port repositories, `local-ports` and `macports-ports`. The former is empty, for your own local Portfiles; the later is a symlink to a known version of the official macports ports. If you like, you can use your own copy of this.

The first time `reproducible-port` is run it compiles [MacPorts](https://www.macports.org/) from its own, versioned copy, then creates local PortIndices for a known set of all Portfiles. This is quite slow. Subsequent runs are very fast. Downloaded distfiles are put into `reproducible-macports.conf/cache` and so can be checked in - avoiding the problem of the interent losing them during a build.

Installed binaries and libraries can be found in `reproducible-macports.conf/root`.

## Caveats

The installation is tied to a specific PATH. If you move it delete `reproducible-macports/prefix`.

The software can support officially unsupported Macs running Mojave and Catalina. To add yours, fork this project, and edit `functions/macos.functions`'s function `reproducible_macports_supportOfficiallyUnsupportedMacs`. Please consider a pull request.