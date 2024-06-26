# mexIPOPT work by apjanke

This is apjanke's work on the mexIPOPT / Octave.app problem that rdzman is having.

See GH issue: <https://github.com/octave-app/octave-app/issues/280>

## Terminology

* ipopt-rdz - ipopt, built with coinbrew, and using rdz's custom build process (and using system Homebrew)
*

## What's in here

* `ipopts/`         - ipopt builds and sources
  * `coinbrew`      - coinbrew binary for building ipopt
  * `fetch_*`       - downloader programs for ipopt source and coinbrew
  * `build_ipopt`   - our custom ipopt build script
  * `opt/`          - where the built ipopt libraries go
* `repos/`          - where you put your cloned repos
* `build_mexipopt`  - builds the mexIPOPT MEX file, once you have ipopt built
* `run_mex_test`    - runs a test on the built mexIPOPT MEX file, once it is built

Much of these dirs are gitignored, and are auto-created by the `fetch_*` and `build_*` programs.

The ipopts/ directory holds various builds and sources for ipopt (the main library, not the mexIPOPT Matlab bindings). This needs a custom build instead of just using the brewed ipopt. For one, the MUMPS that the Homebrew ipopt formula includes causes it to be MPI-enabled, which is making it throw MPI-related errors when used inside mexIPOPT.

To get the `coinbrew` binary, do: `wget https://raw.githubusercontent.com/coin-or/coinbrew/v2.0/coinbrew; chmod a+x coinbrew` in the directory where it should live.

To fetch and build Ipopt:

```bash
./coinbrew fetch Ipopt --no-prompt
./coinbrew build Ipopt --prefix ~/build/ipopt/install --tests all --no-prompt
```

That'll fetch repos into a `Ipopt` and `ThirdParty` repos in that directory. It'll grab the latest version. As of this testing, that's ipopt 3.14.16. And it looks like it includes custom coinbrew patches.

## rdzman's original instructions

The above is my adaptation of rdzman's build steps. rdzman [says in the GH issue](https://github.com/octave-app/octave-app/issues/280#issuecomment-2163724808):

Build IPOPT from source

Looks like this depends on prior HomeBrew installs of metis and gfortran.

```
cd ~/build/ipopt
export METIS_VER=`ls /opt/homebrew/Cellar/metis`
export with_metis_cflags=-I/opt/homebrew/Cellar/metis/${METIS_VER}/include
export with_metis_lflags="-L/opt/homebrew/Cellar/metis/${METIS_VER}/lib -lmetis"
export FC=/opt/homebrew/bin/gfortran
wget https://raw.githubusercontent.com/coin-or/coinbrew/v2.0/coinbrew
chmod a+x coinbrew
./coinbrew fetch Ipopt --no-prompt
./coinbrew build Ipopt --prefix ~/build/ipopt/install --tests all --no-prompt
```

Build native Octave IPOPT interface

This uses my own fork of the mexIPOPT repo, based on an older version that I was able to get working, and with all the binary builds stripped out for faster cloning.

```
cd ~/build/ipopt-octave
git clone -b no-bin --depth=1 https://github.com/rdzman/mexIPOPT.git mexIPOPT
export PKG_CONFIG_PATH=~/build/ipopt/install/lib/pkgconfig
cd mexIPOPT/toolbox
mkdir -p bin
octave --no-gui --eval "CompileIpoptMexLib"
mkdir ~/ipopt-octave
cp -p ~/build/ipopt-octave/mexIPOPT/toolbox/ipopt_auxdata.m ~/ipopt-octave
cp -p bin/ipopt_oct.mex ~/ipopt-octave/ipopt.mex
```
