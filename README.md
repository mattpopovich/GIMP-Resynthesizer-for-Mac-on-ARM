## GIMP's Resynthesizer Plugin for Mac on ARM (Apple Silicon)
[bootchk](https://github.com/bootchk)'s [resynthesizer plugin](https://github.com/bootchk/resynthesizer) is not terribly easy to install and use on a Mac. This repo hopes to make it easier.

Special thanks to:
- [aferrero2707](https://github.com/aferrero2707) for the original Mac port: [`gimp-plugins-collection`](https://github.com/aferrero2707/gimp-plugins-collection)
- pietvo, the maintainer of [MacPorts](https://ports.macports.org/)' [`gimp-resynthesizer`](https://ports.macports.org/port/gimp-resynthesizer/).
- [wuguipeng](https://github.com/wuguipeng)'s [`gimp-resynthesizer-mac-arm64`](https://github.com/wuguipeng/gimp-resynthesizer-mac-arm64) for the inspiration

## Install GIMP
The resynthesizer plug-in is for GIMP, so you won't be able to use it... without installing GIMP. Download GIMP's installer [here](https://www.gimp.org/downloads/).

## Install the Resynthesizer Plugin with Pre-Built Libraries (easiest)
Go to the [releases of this repo](https://github.com/mattpopovich/GIMP-Resynthesizer-for-Mac-on-ARM/releases) and follow the installation instructions for the version of GIMP that you have installed.

Note that these files are compiled for macOS computers running ARM (Apple Silicon = M1, M2, etc.) processors. They will not work on macOS computers with Intel processors or any Windows or Linux machines.

## How I Created This Repo
See [RepoCreation.md](/RepoCreation.md) for more detail on how the files in this repository were created.

## Building (and installing) the Resynthesizer Plugin via MacPorts (more difficult)
You can also install the resynthesizer plugin via MacPorts if you would like to build from source. It will take a little bit longer + could be more difficult (to build) and require some more hard drive space (libraries) but this is a perfectly valid solution as well. See [BuildingResynthesizerViaMacPorts.md](/BuildingResynthesizerViaMacPorts.md) for more information on the difficulties I had and their solutions.
