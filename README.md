## GIMP's Resynthesizer Plugin for Mac on ARM (Apple Silicon)
[bootchk](https://github.com/bootchk)'s [resynthesizer plugin](https://github.com/bootchk/resynthesizer) is not terribly easy to install and use on a Mac. This repo hopes to make it easier.

### Install GIMP
The resynthesizer plug-in is for GIMP, so you won't be able to use it... without installing GIMP. Download GIMP's installer [here](https://www.gimp.org/downloads/).

### Install the Resynthesizer Plugin
Copy `resynthesizerLibraries`, `*.py`, `resynthesizer`, and `resynthesizer_gui` files from this repo to `/Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins`. Relaunch GIMP and you should see the plugin under *Filters* --> *Enhance* --> *Heal selection...*.

### How I Created This Repo

I built [gimp-resynthesizer via MacPorts](https://ports.macports.org/port/gimp-resynthesizer/). Then, copied the resulting `*.py`, `resynthesizer`, and `resynthesizer_gui` from `/opt/local/lib/gimp/2.0/plug-ins/` to `/Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins`. Done!

However, I didn't want to force people to install MacPorts and build `gimp-resynthesizer`. I had some decent difficulties doing that (details in the section below) and I feel like the process may not be very repeatable. Ideally, I wanted something that was "drag and drop". Unfortunately, just copying over the   `*.py`, `resynthesizer`, and `resynthesizer_gui` files results in:

```console
user@Mac /Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins $ ./resynthesizer
dyld[8592]: Library not loaded: /opt/local/lib/libgimpui-2.0.0.dylib
  Referenced from: <B97A09A8-B52F-3238-8754-CAEB415B80FF> /Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins/resynthesizer
  Reason: tried: '/opt/local/lib/libgimpui-2.0.0.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/opt/local/lib/libgimpui-2.0.0.dylib' (no such file), '/opt/local/lib/libgimpui-2.0.0.dylib' (no such file)
```

So, we have some missing libraries (`libgimpui-2.0.0.dylib` above + many more). I figured I would just include all the libraries that `resynthesizer` uses. MacPorts builds its libraries to `/opt/local/lib` (Confirm via `port contents gimp-resynthesizer`) and I can copy them from there. However, there were a lot of missing libraries and those libraries have missing libraries as well. Plus, since I want to put the libraries in a folder other than `/opt/local/lib`, I need to change the path of all the library references... It was a bit of a rabbit hole, but I think I made it work. 

It was a cycle of trying to run the resynthesizer via command line (`./Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins/resynthesizer`) then copying over the missing library from `/opt/local/lib`, then fixing the library's path. I was effectively making a bunch of "find and replace" changes from `/opt/local/lib` to `@executable_path/resynthesizerLibraries/`, where `@executable_path` is something along the lines of `/Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins`. 

Some helpful commands were:
```console
$ # Confirm this is an ARM executable
$ file resynthesizer
resynthesizer: Mach-O 64-bit executable arm64
$
$ # Show where this executable expects libraries to be
$ otool -L resynthesizer
resynthesizer:
	/opt/local/lib/libgimpui-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
	/opt/local/lib/libgimpwidgets-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
	/opt/local/lib/libgimpmodule-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
    [...]
$
$ # Add a directory to the runtime library search path
$ install_name_tool -add_rpath @executable_path/../../../../ resynthesizer 
$
$ # Change the library's path
$ install_name_tool -change /opt/local/lib/libgimpui-2.0.0.dylib @rpath/lib/libgimpui-2.0.0.dylib resynthesizer
$
$ # Confirm the library's path was changed
$ otool -L resynthesizer
resynthesizer:
	@rpath/lib/libgimpui-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
	/opt/local/lib/libgimpwidgets-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
	/opt/local/lib/libgimpmodule-2.0.0.dylib (compatibility version 1001.0.0, current version 1001.38.0)
    [...]
```

I did this **many** of times until all the libraries were happy, which made the resynthesizer executable happy:

```console
user@Mac /Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins $ ./resynthesizer
./resynthesizer is a GIMP plug-in and must be run by GIMP to be used
```

Yay! No more library complaints.

### Building the Resynthesizer Plugin via MacPorts
This was a little tricky and required a few tweaks for `sudo port install gimp-resynthesizer` to finish successfully.

When I first tried it, I was getting errors installing `graphviz`:
```console
$ sudo port install gimp-resynthesizer
[...]
Error: Failed to configure graphviz: consult /opt/local/var/macports/build/_opt_local_var_macports_sources_rsync.macports.org_macports_release_tarballs_ports_graphics_graphviz/graphviz/work/graphviz-12.2.1/config.log
Error: Failed to configure graphviz: configure failure: command execution failed
Error: See /opt/local/var/macports/logs/_opt_local_var_macports_sources_rsync.macports.org_macports_release_tarballs_ports_graphics_graphviz/graphviz/main.log for details.
Error: Follow https://guide.macports.org/#project.tickets if you believe there is a bug.
Error: Processing of port gimp-resynthesizer failed
```

I then saw the following in `graphviz/main.log`: 

```
:info:configure configure: error: *** A compiler with support for C++17 language features is required.
:info:configure Command failed:  cd "/opt/local/var/macports/build/_opt_local_var_macports_sources_rsync.macports.org_macports_release_tarballs_ports_graphics_graphviz/graphviz/work/graphviz-12.2.1" && ./configure --prefix=/opt/local ac_cv_prog_AWK=/usr/bin/awk --disable-guile --disable-io --disable-java --disable-lua --disable-man-pdfs --disable-ocaml --disable-perl --disable-php --disable-python --disable-python23 --disable-python24 --disable-python25 --disable-r --disable-ruby --disable-sharp --disable-silent-rules --disable-swig --disable-tcl --with-codegens --with-digcola --with-fontconfig --with-freetype2 --with-gts --with-ipsepcola --with-webp --without-ann --without-devil --without-gdk --without-gdk-pixbuf --without-ghostscript --without-glade --without-glitz --without-gnomeui --without-gtk --without-gtkgl --without-gtkglext --with-lasi --without-ming --with-pangocairo --without-poppler --without-qt --with-quartz --without-rsvg --without-smyrna 
:info:configure Exit code: 1
:error:configure Failed to configure graphviz: consult /opt/local/var/macports/build/_opt_local_var_macports_sources_rsync.macports.org_macports_release_tarballs_ports_graphics_graphviz/graphviz/work/graphviz-12.2.1/config.log
:error:configure Failed to configure graphviz: configure failure: command execution failed
```

I found [this thread]( https://trac.macports.org/ticket/71577) on fixing the C++17 error:
```console
$ sudo port install clang-18
$ sudo port install graphviz configure.cxx=clang++-mp-18
```

Pro: `graphviz` was now installed. Con: Now `gexiv2` wouldn't install.

[This ticket](https://trac.macports.org/ticket/71179) led me to [this post](https://trac.macports.org/wiki/ProblemHotlist#clts16) which resulted in the following fix:
```console
$ sudo rm -rf /Library/Developer/CommandLineTools/usr/include/c++
$ sudo port clean gexiv2
$ sudo port install gexiv2
```

This allowd `gexiv2` to be installed. 

Finally, `gimp-resynthesizer` built successfully:

```console
$ sudo port install gimp-resynthesizer
```

All that's left is to copy the `*.py`, `resynthesizer`, and `gimp-resynthesizer` files from their installation location (`/opt/local/lib`) to GIMP's installation location (`/Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins`).

You can find their installation location with the following command:
```console
 % port contents gimp-resynthesizer
Port gimp-resynthesizer @20190428-adfa25ab_0 contains:
  /opt/local/lib/gimp/2.0/plug-ins/plugin-heal-selection.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-heal-transparency.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-map-style.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-render-texture.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-resynth-enlarge.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-resynth-fill-pattern.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-resynth-sharpen.py
  /opt/local/lib/gimp/2.0/plug-ins/plugin-uncrop.py
  /opt/local/lib/gimp/2.0/plug-ins/resynthesizer
  /opt/local/lib/gimp/2.0/plug-ins/resynthesizer_gui
  [...]
  ```
