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

# Add MacPorts to reproducible-macports/local-ports
# eg
cd reproducible-macports/local-ports
mkdir my-port-category  # eg 'ASIC'.
cd my-port-category
vi Portfile  # See MacPorts guide at https://guide.macports.org .
```

A folder, `reproducible-macports.conf`, is created. To use reproducible-macports, simply replace any usages of [MacPorts](https://www.macports.org/)'s `port` with `reproducible-port`. This folder is automatically added to git, and contains two port repositories, `local-ports` and `macports-ports`. The former is empty, for your own local Portfiles; the later is a symlink to a known version of the official macports ports. If you like, you can use your own copy of this.

The first time `reproducible-port` is run it compiles [MacPorts](https://www.macports.org/) from its own, versioned copy, then creates local `PortIndex` files for a known set of all `Portfile`s. This is quite slow. Subsequent runs are very fast. Currently, on every run, the `PortIndex` files are reindexed. This can be suppressed by setting the environment variable `REPRODUCIBLE_PORT_NO_INDEX` to any value, including an empty string.

Installed binaries and libraries can be found in `reproducible-macports.conf/root`.

A cache of downloaded distfiles (downloaded archives for versions) can be found in `reproducible-macports.conf/distfiles`; by default, this is set up to be committed to git, so allowing one to capture internet originating files for source code that might disappear or be unavailable during a build.


## Caveats

The installation is tied to a specific path (it has to use absolute paths, a limitation of the software packages it compiles). If you move it delete `reproducible-macports/prefix`.

The software can support officially unsupported Macs running Mojave and Catalina. To add yours, fork this project, and edit `functions/macos.functions`'s function `reproducible_macports_supportOfficiallyUnsupportedMacs`. Please consider a pull request. Currently, it supports a 2009 MacBook Pro 5,2 running Mojave.

If a local port file is invalid, indexing does not fail but `reproducible-port` will produce a cryptic error on install such as `reproducible-macports/reproducible-port install bad_port` for `Error: Port bad_port not found`. Sadly, this also can occur is `bad_port` exists but `REPRODUCIBLE_PORT_NO_INDEX` has become set and it has never been indexed.


## License

The license for this project is MIT.
