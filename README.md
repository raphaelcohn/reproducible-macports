# reproducible-macports

This simple wrapper makes [MacPorts](https://www.macports.org/) reproducible and suitable for use as part of a wider git tree, so all your builds are tied to specific, versioned portfiles and distfiles. This project arose because it was too hard to do this with homebrew (especially because homebrew ties recipe syntax too tightly to the version of Ruby and homebrew).

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

A cache of downloaded distfiles (downloaded archives for versions) can be found in `reproducible-macports.conf/distfiles`; by default, this is set up to be committed to git, so allowing one to capture internet originating files for source code that might disappear or be unavailable during a build. An empty `.gitignore` file is in this folder. Simply edit to contain `/*/` to not commit any distfiles to git.


## Quick Uses


### Getting out of a mess

If you get into a mess, just delete `reproducible-macports/prefix`, eg `rm -rf reproducible-macports/prefix`


### Sharing built binaries with others

reproducible-port always builds from source; for some ports, this can take a very long time indeed. It is possible to share built ports without setting up a webserver. If all of your colleagues are using recent (ie not unsupported) Macs, and have checked out the parent repository (eg `my-git-repository` ) containing `reproducible-macports` and `reproducible-macports.conf` to ***exactly*** the same location (eg `/Users/must-be-the-same/Documents/my-git-repository`), then edit `reproducible-macports.conf/software/.gitignore` and remove the line `/*/*.tbz2`. The built ports in `reproducible-macports.conf/software` can then be checked into git and will be used by other checkouts.


### Removing unwanted distfiles for ports no longer used

Delete folders in `reproducible-macports.conf/distfiles`.


### Adding binaries to the path as part of a build

At the beginning of a build script, say `build` in `my-git-repository`, add:-

```bash
#!/usr/bin/env sh

set -e
set -f
set -u

use_reproducible_ports()
{
	local reproducible_port=./reproducible-port.conf/reproducible-port
	
	set +e
		"$reproducible_port" -q -N uninstall inactive 2>/dev/null
		local exitCode=1
		while [ $exitCode -eq 1 ]
		do
			"$reproducible_port" -q -N uninstall --follow-dependencies --follow-dependents leaves 2>/dev/null
			exitCode=$?
		done
	set -e
	
	local portName
	for portName in "$@"
	do
		"$reproducible_port" -q -N clean --logs --work "$portName"
		"$reproducible_port" -q -N install --no-rev-upgrade "$portName"
	done
	
	# Run in case local-ports or macports-ports have changed.
	"$reproducible_port" -q -N upgrade outdated
	
	export PATH="$(pwd)"/reproducible-macports.conf/root/bin:"$(pwd)"/reproducible-macports.conf/root/sbin:/usr/bin:/usr/bin:/bin:/sbin
}

#¹

use_reproducible_ports wget bzip2

# Now saying wget or bunzip will use the binary installed as a reproducible port; all other binaries will be those shipped with macos.
```

¹As an aside, if you want a location independent build script, copy the function `_program_path_find` from `reproducible-port` and then do `cd "$(_program_path_find)" 1>/dev/null 2>/dev/null`; this will set the current working directory (`CWD`) to the absolute folder path containing your build script.



## Advanced Uses


### Automatically maintaining compiled and installed ports

A frequent use case for reproducible-port is to install a build-local set of known, good binaries and libraries which are rebuilt or reinstalled when the list as a whole changes. This can be simply done using `reproducible-port-list`. Firstly, edit the file, `reproducible-macports.conf/PortList`. Add a list of ports, one per line. For example:-

```
wget
bzip2
```

(Make sure the final line ends with a line feed; comments starting a line with # and empty lines are allowed but are treated as changes if they change).

Now, as part of your build script (such as `my-git-repository/build`), add lines equivalent to:-

```
"$(pwd)"/reproducible-macports/reproducible-port-list
export PATH="$(pwd)"/reproducible-macports.conf/root/bin:"$(pwd)"/reproducible-macports.conf/root/sbin:/usr/bin:/usr/bin:/bin:/sbin
```

From now on, whenever the file `reproducible-macports.conf/PortList` changes, ports will be added or removed as necessary; if a port changes version, it will be upgraded. Additionally, when ports are no longer in the `PortList`, their files in `reproducible-macports.conf/distfiles` and `reproducible-macports.conf/software` will be removed.


### Changing which ports require root permissions to install

A hard-coded list of ports requiring root permissions to install them is kept in `reproducible-macports.conf/ports-requiring-root-to-install`. It is only used by `reproducible-port-list` currently. By default, this file is a symlink; if you need to change it, break the symlink and copy the file's contents to `reproducible-macports.conf/ports-requiring-root-to-install` and then edit them; it consists of a line-delimited list of ports (one port per line). Comments beginning `#` and blank lines are permitted, but not whitespace.


## Caveats

The installation is tied to a specific path (it has to use absolute paths, a limitation of the software packages it compiles). If you move it delete `reproducible-macports/prefix`.

The software can support officially unsupported Macs running Mojave and Catalina. To add yours, fork this project, and edit `functions/macos.functions`'s function `reproducible_macports_supportOfficiallyUnsupportedMacs`. Please consider a pull request. Currently, it supports a 2009 MacBook Pro 5,2 running Mojave.

If a local port file is invalid, indexing does not fail but `reproducible-port` will produce a cryptic error on install such as `reproducible-macports/reproducible-port install bad_port` for `Error: Port bad_port not found`. Sadly, this also can occur is `bad_port` exists but `REPRODUCIBLE_PORT_NO_INDEX` has become set and it has never been indexed.


## License

The license for this project is MIT.
