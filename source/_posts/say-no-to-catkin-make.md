---
title: Say no to catkin_make
date: 2017-01-19 17:03:20
tags: ros
author: alex
---
`catkin` is the wrapper around cmake that ROS uses. Majority of the people I
know are still using plain `catkin_make` to build their workspaces but that is
pretty basic. Please use `catkin`, it is much more powerful and convienient.

To initialize a workspace (that already has a `src` folder)

```sh
catkin init
catkin build
```

Then you can easily set cmake parameters

```sh
catkin config --install --cmake-args -DCMAKE_FIND_FRAMEWORK=LAST \
  -DCMAKE_BUILD_TYPE=Release
```

or 

```sh
catkin config --install --cmake-args -DCMAKE_FIND_FRAMEWORK=LAST \
  -DCMAKE_BUILD_TYPE=Release \
  -DPYTHON_LIBRARY=$(python -c "import sys; print sys.prefix")/lib/libpython2.7.dylib \
  -DPYTHON_INCLUDE_DIR=$(python -c "import sys; print sys.prefix")/include/python2.7 \
  -DQT_USE_FRAMEWORKS=ON -DCATKIN_ENABLE_TESTING=ON
```

If you're using homebrewed libraries on MacOS.

To build a particular package

```sh
catkin build pkg
```

Or blacklist/whitelist with 

```sh
catkin config (--blacklist|--whitelist) pkg
```

catkin will build your workspace in parallel and resolve dependencies
automatically. If you are building an individual package you can force it to
build only that package by 

```sh
catkin build pkg --no-deps
```

Verbosity and job control

```sh
catkin build -v -jN
```

