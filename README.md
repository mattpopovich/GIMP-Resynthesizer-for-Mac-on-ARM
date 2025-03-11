## GIMP's Resynthesizer Plugin for Mac on ARM (Apple Silicon)
[bootchk](https://github.com/bootchk)'s [resynthesizer plugin](https://github.com/bootchk/resynthesizer) is not terribly easy to install and use on a Mac. This repo hopes to make it easier.

Special thanks to:
- [aferrero2707](https://github.com/aferrero2707) for the original Mac port: [`gimp-plugins-collection`](https://github.com/aferrero2707/gimp-plugins-collection)
- pietvo, the maintainer of MacPorts' [`gimp-resynthesizer`](https://ports.macports.org/port/gimp-resynthesizer/).
- [wuguipeng](https://github.com/wuguipeng)'s [`gimp-resynthesizer-mac-arm64`](https://github.com/wuguipeng/gimp-resynthesizer-mac-arm64) for the inspiration

## Install GIMP
The resynthesizer plug-in is for GIMP, so you won't be able to use it... without installing GIMP. Download GIMP's installer [here](https://www.gimp.org/downloads/).

## Install the Resynthesizer Plugin
Copy the following files:
- `resynthesizerLibraries` folder
- All the python files (`*.py`)
- `resynthesizer`
- `resynthesizer_gui`

From this repo to `/Applications/GIMP.app/Contents/Resources/lib/gimp/2.0/plug-ins`.

Relaunch GIMP and you should see the plugin under *Filters* --> *Enhance* --> *Heal selection...*.

Note that these files are compiled for macOS computers running ARM (Apple Silicon = M1, M2, etc.) processors. They will not work on macOS computers with Intel processors or any Windows or Linux machines.

## How I Created This Repo
See [RepoCreation.md](/RepoCreation.md) for more detail on how the files in this repository were created.

## Building the Resynthesizer Plugin via MacPorts
See [BuildingResynthesizerViaMacPorts.md](/BuildingResynthesizerViaMacPorts.md) for some difficulties that I had during building and their solutions.
