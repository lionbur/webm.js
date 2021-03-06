<p align="center">
  <a href="https://kagami.github.io/webm.js/"><img src="https://raw.githubusercontent.com/Kagami/webm.js/master/src/index/logo.png"></a>
</p>

<p align="center">
  Create WebM videos in your browser. No server-side, pure JavaScript.
</p>

<p align="center">
  <a href="https://travis-ci.org/Kagami/webm.js"><img alt="Build Status" src="https://travis-ci.org/Kagami/webm.js.svg?branch=master"></a>
</p>

---

### What's this?

webm.js is a simple one-page application that allows you to convert videos to WebM format right into your browser, without any plugins or server-side involved. It is built upon FFmpeg, libvpx and libopus which were ported to JavaScript using Emscripten. Since WebM is part of HTML5 stack, why can't we create one without leaving the browser?

### Try it

Latest build of webm.js is [available here](https://kagami.github.io/webm.js/). NOTE: built-in video player uses software decoding in order to play any video it can encode and thus experimental and slow.

Tested browsers:
* Firefox: fast enough to encode several minutes of SD/HD video within half of hour
* Chrome: a bit slower than FF but still good
* Edge: a bit slower than FF but still good (make sure to enable asm.js in `about:flags`)
* IE11: several times slower than Edge and will fail on big files but otherwise works

Take also look at [demo](https://raw.githubusercontent.com/Kagami/webm.js/assets/demo.webm) and [screenshots](https://github.com/Kagami/webm.js/wiki/Screenshots).

### Is it fast?

Well, partly. With the help of asm.js, generic code compiled by Emscripten is almost as fast as native, but JavaScript doesn't have access to advanced x86 instructions like SSE and codecs use them far and wide so we have some performance degradation here. Not that drastical though—I had numbers like 8x worse than native@corei7-avx for libvpx-vp8 and ~4 fps (single thread, SD, medium settings). Low-level multithreading is also [not available](https://github.com/kripken/emscripten/blob/master/site/source/docs/porting/pthreads.rst) in stable versions of browsers, but luckily we can hack it up by splitting video into chunks and encoding them in separate workers.

To get you some concrete numbers: it takes ~5m25s to encode [sample clip](https://github.com/Kagami/webm.js/blob/master/src/source/elephants-dream.webm) (860x484, 1m9s) with default settings (2pass, `-threads 4 -c:v libvpx -b:v 885k -speed 1 -auto-alt-ref 1 -lag-in-frames 25 -c:a libopus -b:a 64k`) at i7 3820. Same encode with native FFmpeg/libvpx takes ~40s. Hopefully we will improve that performance with WebAssembly and SIMD.js, stay tuned!

### What about quality?

Currently only VP8+Opus combination is supported. Opus is better than Vorbis ([and almost any other lossy audio codec](http://opus-codec.org/comparison/quality.png)) for the full range of bitrates and encoder is blazingly fast. Unfortunately, due to lack of SIMD, libvpx-vp9 encoder is rather impractical in browser and VP8 is significantly worse than VP9. Though [JavaScript SIMD API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SIMD) is part of ES7 proposal and already supported by Firefox Nightly so I'm actively looking into this.

Trying to compensate the use of outdated codec, two-pass encoding with `speed=1` and `lag-in-frames=25` is used. Splitting video in parts creates additional keyframes and therefore loses effeciency a bit, but  the difference should be negligible for the `g=128` currently used. It is also possible to specify `speed=0` or `quality=best` and other FFmpeg/libvpx options if you're trying to achieve the maximal quality.

### How to start hacking?

Build your own version of webm.js is as simple as clone the repo and run

```bash
npm i && npm run release
```

inside. Host the `dist` directory with your favourite HTTP server or use `npm start` to start the development server at 8080 port.

### I want to use it locally and don't have node.js

No problem! Just download [webm.js-gh-pages.zip](https://github.com/Kagami/webm.js/archive/gh-pages.zip), unpack it and open `index.html` with Firefox. Everything should work right away. Other browsers will require local HTTP server, run `python -mSimpleHTTPServer` in unpacked directory if in doubt.

It's also perfectly possible to build browser extension, user script or even embed WebM encoder into your site. I haven't done anything in that direction yet, but let me know if you're interested.

### License

webm.js own code, documentation, favicon and logo licensed under CC0, but the resulting build also includes the following libraries and assets:

* FFmpeg port [ffmpeg.js](https://github.com/Kagami/ffmpeg.js) (LGPL-2.1+ and few libraries under BSD, see [full license text](https://github.com/Kagami/ffmpeg.js/blob/master/LICENSE.WEBM))
* Remaining libraries in `dependencies` section of [package.json](https://github.com/Kagami/webm.js/blob/master/package.json) (BSD-like)
* [Roboto font](https://www.google.com/fonts/specimen/Roboto) by Christian Robertson (Apache License 2.0)
* [Liberation Sans font](https://fedorahosted.org/liberation-fonts/) (SIL Open Font License 1.1)
* Sample video is part of Elephants Dream movie ((c) copyright 2006, Blender Foundation / Netherlands Media Art Institute / www.elephantsdream.org)
* [GitHub Ribbon](https://github.com/blog/273-github-ribbons) (MIT)

---

webm.js - JavaScript WebM encoder

Written in 2015 by Kagami Hiiragi <kagami@genshiken.org>

To the extent possible under law, the author(s) have dedicated all copyright and related and neighboring rights to this software to the public domain worldwide. This software is distributed without any warranty.

You should have received a copy of the CC0 Public Domain Dedication along with this software. If not, see <http://creativecommons.org/publicdomain/zero/1.0/>.
