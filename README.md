## GIMP's Resynthesizer Plugin for Mac on ARM (Apple Silicon)
[bootchk](https://github.com/bootchk)'s [resynthesizer plugin](https://github.com/bootchk/resynthesizer) is not terribly easy to install and use on a Mac. This repo hopes to make it easier.

Special thanks to [wuguipeng](https://github.com/wuguipeng)'s [`gimp-resynthesizer-mac-arm64`](https://github.com/wuguipeng/gimp-resynthesizer-mac-arm64) for the inspiration.

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
See [BuildingResynthesizerViaMacPorts](/BuildingResynthesizerViaMacPorts.md) for some difficulties that I had doing building this and their solutions.
