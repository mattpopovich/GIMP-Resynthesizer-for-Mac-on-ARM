### Building the Resynthesizer Plugin via MacPorts
This was extremely easy on my M1 Mac with Sequoia 15.3.2 as the command `sudo port install gimp-resynthesizer` finished the first time successfully. Hopefully this is how it goes for you! However, it required a few tweaks to finish successfully on a different M1 Mac with Sequoia 15.3.1. Not sure what the difference was between the two, but the rest of this will be from the perspective of the difficult installation.

When I first tried `sudo port install gimp-resynthesizer`, I was getting errors installing `graphviz`:
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

**Pro**: `graphviz` was now installed. **Con**: Now `gexiv2` wouldn't install.

```console
$ sudo port install gimp-resynthesizer
[...]
:info:build /opt/local/include/glib-2.0/glib/glib-typeof.h:43:10: fatal error: 'type_traits' file not found
350 :info:build    43 | #include <type_traits>
351 :info:build       |          ^~~~~~~~~~~~~
352 :info:build 1 error generated.
[...]
```

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
