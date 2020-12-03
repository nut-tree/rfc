# OpenCV + Webassembly in nut.js

## Abstract

The overall goal outlined in this document is to replace [opencv4nodejs](https://github.com/nut-tree/opencv4nodejs) and [npm-opencv-build](https://github.com/nut-tree/npm-opencv-build) in nut.js.
As an alternative I propose using a WebAssembly build of [OpenCV.js](https://docs.opencv.org/master/d4/da1/tutorial_js_setup.html), officially provided by [OpenCV](https://github.com/opencv/opencv).

## Rationale

So far nut.js is relying on a custom build of [opencv4nodejs](https://github.com/nut-tree/opencv4nodejs) for everything related to image processing.
This comes with several drawbacks:

### Platform dependence

Both OpenCV libs provided via `npm-opencv-build` as well `opencv4nodejs` bindings are platform dependent.
Each build for a single platform requires knowledge about its pecularities and how to properly troubleshoot them.
This could cause increased maintenance efforts in the future.

### Node ABI dependence

`opencv4nodejs` uses [native abstractions for Node.js](https://github.com/nodejs/nan) to provide its bindings.
In contrast to [N-API](https://nodejs.org/api/n-api.html) based bindings, nan bindings have to be built for a specific [ABI version](https://nodejs.org/en/download/releases/).
So with every new node and / or Electron version, efforts have to be taken to provide bindings.
These bindings are platform dependent as well, so it's three time the effort to verify each build.
The latest release provides 41 prebuilt binaries (3 platforms * various node versions + 3 platforms * various Electron versions), which is getting out of hand.

### Overal complexity

The overall setup to provide pre-compiled OpenCV libs + bindings is pretty complex.
It required changes to the OpenCV [CMake](https://cmake.org) config, exports in `npm-opencv-build` to properly link against the provided libs as well as changes to the [node-gyp](https://github.com/nodejs/node-gyp) `binding.gyp` file.
The setup had me waddle through compiler / linker differences on each platform and comes with a custom packaging mechanism for `npm-opencv-build`.

While all of the above lead to a ready-to-use setup of nut.js without having to re-compile anything on your machine, it is a huge mess.

### Side-loading artifacts

[prebuild](https://www.npmjs.com/package/prebuild) and [prebuild-install](https://www.npmjs.com/package/prebuild-install) work by having `prebuild` upload build artifacts to a [GitHub release](https://github.com/nut-tree/opencv4nodejs/releases/tag/v5.3.0-3). On install, `prebuild-install` will download the respective artifact depending on your platform / node / Electron version.
Having a package side-load binaries on installation always comes with a smell, so I'd like to get rid of it!

## Proposed changes

In order to get rid of `opencv4nodejs` the following steps have to be considered.

### Replace usage of OpenCV for image manipulation

While OpenCV.js does allow to perform image manipulation like scaling etc., I'd love to keep it's usage to a minimum.
There are pure JS libraries for image manipulation like e.g. [jimp](https://www.npmjs.com/package/jimp) which should be evaluated.

### Use OpenCV WebAssembly build

There already exists [a package](https://www.npmjs.com/package/opencv-wasm) which ships OpenCV 4.3.0.
However, we might also consider releasing our own package following [the official guide](https://docs.opencv.org/master/d4/da1/tutorial_js_setup.html) to gain control about releases.

### Worker threads

Functions provided by OpenCV.js are blocking.
This is a major issue, since it causes a major slow down when searching through multiple scale stages.

A first prototype showed the following runtime:

```bash
> time node scratch.js
node scratch.js  13.35s user 0.33s system 197% cpu 6.935 total
```

Compared to nut.js v1.5.0:

```bash
time node scratch.js 
node scratch.js  2.25s user 0.37s system 164% cpu 1.594 total
```

Possible performance increase when shifting e.g. image matching to [worker threads](https://nodejs.org/dist/latest-v15.x/docs/api/worker_threads.html) should be benchmarked before any further steps are taken.