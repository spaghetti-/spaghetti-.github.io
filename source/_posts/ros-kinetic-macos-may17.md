---
title: ROS Kinetic Installation on MacOS (Homebrew, as of May '17)
date: 2017-05-09 15:05:20
tags: ros
---

Every now and again I nuke my ROS installation and build from scratch because of
dependencies getting updated/my usecase changing. I prefer to have two
installations of ROS on my system these days. Jade, for stability and feature
matching with my platforms and Kinetic for testing and submitting patches on. I
manage both in an `eselect`esque way (it's a tool found on Gentoo Linux for
managing versions via symlinks).

```
sudo mkdir /opt/kinetic
sudo mkdir /opt/jade
sudo chown -R `whoami`:staff /opt/kinetic /opt/jade
ln -s /opt/kinetic/install /opt/ros/kinetic # or jade
```

OpenCV3
-------

Homebrew uses an outdated brew contrib release which does not have CMake fixes
for the freetype contrib module. Install it with `--HEAD`:

```
brew install opencv3 --with-python --with-eigen --c++11 --with-contrib \
  --with-ffmpeg --with-tbb --with-qt --with-non-free --without-test --HEAD
```

Note: takes around 20 minutes to compile.

AFAIK, Jade still uses OpenCV2 while Kinetic has migrated to CV3. To build
`cv_bridge` do 

```
brew ln opencv3 --force
```

This may cause a problem with Jade stuff, so unlink and relink opencv2 if need
be. Homebrew warns about this when you try to install opencv3.

Qt
--

```
echo 'export PATH="/usr/local/opt/qt/bin:$PATH"' >> ~/.zshrc
```

Other than that, there were absolutely no errors involved in compiling the
desktop version of kinetic on MacOS. Wonderful. Easiest it's ever been really.

