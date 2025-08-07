<p align="center" style="background:#fff">
  <img width="650px" src="./assets/web-video-creator.png" />
</p>

[![NPM](https://nodei.co/npm/web-video-creator.png)](https://npmjs.org/package/web-video-creator)

![GitHub](https://img.shields.io/github/license/Vinlic/WebVideoCreator)
![GitHub package.json version](https://img.shields.io/github/package-json/v/Vinlic/WebVideoCreator)
![npm](https://img.shields.io/npm/dt/web-video-creator)
![GitHub Repo stars](https://img.shields.io/github/stars/Vinlic/WebVideoCreator)

# Introduction

🌈 WebVideoCreator (abbreviated as WVC) is a framework for rendering web animations into videos. It's implemented based on Node.js + Puppeteer + Chrome + FFmpeg. It performs deterministic rendering and captures any HTML5-playable animations (CSS3 animations/SVG animations/Lottie animations/GIF animations/APNG animations/WEBP animations) and any timeline-based animations driven by [RAF](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame). You can also mischievously use `setInterval` or `setTimeout` to control animations. WVC supports embedding or exporting MP4 and transparent WebM videos, as well as features such as transition compositing, audio synthesis, and font loading. Let's get started with the [Quick Start](#quick-start) 🍻.

WVC creates a virtual time environment🕒 for your cool animated pages, which can be imagined as something akin to "The Truman Show." Its main responsibility is to transform an [uncertain rendering environment](./docs/renderer-env.md#不确定性的渲染环境) into a [deterministic rendering environment](./docs/renderer-env.md#确定性的渲染环境).

All of this is made possible by Chrome's provided [deterministic rendering mode](https://goo.gle/chrome-headless-rendering) and support for the headless experimental API: [HeadlessExperimental.beginFrame](https://chromedevtools.github.io/devtools-protocol/tot/HeadlessExperimental/#method-beginFrame), This is an innovative experimental function 🧪.

Q&A and discussion group on QQ🐧: 752693580

<br>

# Features

- Developed using Node.js, it's very easy to use, extend, and develop further.
- Video processing is incredibly fast, rendering a video as long as 5 minutes can be completed in just 1 minute, Check [Performance Tips](#Performance-Tips) for optimal performance.
- Supports rendering and compositing of single scenes and multi-scene videos, with the ability to apply transition effects to multi-scene videos.
- Supports chunked video compositing, allowing chunks to be distributed to multiple devices for rendering, then combined into multi-scene videos, significantly reducing rendering time for long videos.
- Supports parallel rendering and compositing of multiple video tasks.
- Support embedding or exporting webm format videos that support transparent channels.
- API support for [distributed rendering](#distributed-rendering-solution), enabling the distribution of a large number of videos to multiple devices for rendering and final merging output with minimal wrapping of WVC.
- Supports GPU acceleration for rendering and compositing, reducing video rendering time significantly.
- Can be deployed and run on both Windows and Linux platforms, On MacOS, it is necessary to enable [Compatible Rendering Mode](#compatible-rendering-mode).

<br>

# What is it for?

WVC achieves perfect frame-by-frame capture of any animation on a web page. It minimizes the cost of achieving a WYSIWYG (What You See Is What You Get) experience. As a rendering backend, it currently explores the following applications:

**📊 Data Visualization Video Rendering**: Combine chart libraries such as ECharts to create animated charts on the web and capture them as videos using WVC. This is useful for creating dynamic ranking videos commonly seen on platforms like TikTok, especially when combined with web scraping for data collection.

**👩‍🏫 Digital Human Video Rendering**: With the popularity of AI-generated characters, various digital avatars appear on screens. WVC supports the use of transparent channel videos or mask videos on web pages, enhancing visual effects by incorporating digital humans into animations.

**🎨 Content Creation Video Rendering**: Design a simple frontend animation editor and previewer to meet content creation needs. Use WVC as the backend to achieve a WYSIWYG video effect.

**🎮️ Game or User Operation Replay Video Rendering**: Integrate with web-based games or applications to capture replays as videos, providing users with easy sharing and secondary editing options.

There are more application scenarios waiting for your exploration, and if you have great ideas, remember to raise an issue 🙋‍♀️...

## Advantages over Screen Recording Tools?

**💯 Perfect Capture**: The browser's frame composer defaults to a throttling strategy to reduce resource consumption. When drawing complex graphics or under increased system load, web animations may experience frame skipping, dropping frames, or delays. Using screen recording tools makes it difficult to ensure that every frame is captured correctly. In contrast, WVC takes control of the timing, determining when to draw the next frame onto the screen.

**🎞️ Parallel Rendering**: Screen recording tools typically cannot capture the animation content of multiple tab pages simultaneously. However, WVC can capture animations in parallel across multiple pages and ultimately combine these chunks into a single long video, with support for seamless transitions.

**🦾 Automation-Friendly**: Screen recording tools require manual operation. With WVC, a set of web animation templates can be combined with data scrapers and scheduled tasks to automate video production.

**🧩 Quick Integration**: Screen recording tools are challenging to integrate. WVC, developed as an NPM package based on Node.js, can be quickly integrated on the backend. Some developers use it to capture game replays as videos.

<br>

# Video Demos

We're also in need of animators! If you're passionate about open source projects, feel free to join us 😆.

Check out all the demos here: **[Rendering Example Page](https://github.com/Vinlic/WebVideoCreator/wiki/Rendering-Example)** 🤗

<img src="assets/demo.gif"/>

<br>

# Supported Animation Libraries

In theory, all web animation/graphics libraries should work smoothly in the WVC environment. Below, I've listed only the libraries that I have verified to be compatible:

[Anime.js](https://animejs.com/) / [GSAP](https://greensock.com/) / [D3.js](https://d3js.org/) / [Three.js](https://threejs.org/) / [Echart](https://echarts.apache.org/) / [Lottie-Web](http://airbnb.io/lottie/#/web) / [PixiJS](https://pixijs.download/release/docs/index.html) / [Animate.css](https://animate.style/) / [Mo.js](https://mojs.github.io/) / [Tween.js](https://tweenjs.github.io/tween.js/)

Please note that if you manually use [RAF](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame) to drive animations, ensure that you receive the `timestamp` parameter from the callback to set the animation's progress to that timestamp. Otherwise, frame rate asynchrony may occur.

<br>

# Quick Start

## Installation

```shell
# Install WebVideoCreator from NPM
npm i web-video-creator
```

If you encounter issues with the download of `ffmpeg-static`, please set the environment variable: `FFMPEG_BINARIES_URL=https://cdn.npmmirror.com/binaries/ffmpeg-static`.

## Create a Local Server

WVC needs to capture animations from web pages, and you can create a temporary web server locally to serve static pages for testing purposes. One of the simplest ways to do this is by using `live-server`. If you already have static web pages, you can skip this step.

```shell
# Install live-server globally from NPM
npm i -g live-server
# Start the web server
live-server
```

Create a test page in the root directory of the web server. The following HTML content demonstrates an automatically rotating red triangle SVG animation:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Test Page</title>
    </head>
    <body>
        <svg width="120" height="120" viewBox="0 0 120 120" xmlns="http://www.w3.org/2000/svg" version="1.1"
        xmlns:xlink="http://www.w3.org/1999/xlink">
            <polygon points="60,30 90,90 30,90" fill="red">
                <animateTransform attributeName="transform" attributeType="XML" type="rotate" from="0 60 70" to="360 60 70"
                    dur="10s" repeatCount="indefinite" />
            </polygon>
        </svg>
    </body>
</html>
```

## Rendering Single Video

<img style="width:550px" src="./assets/single-video.gif" />

```javascript
import WebVideoCreator, { VIDEO_ENCODER, logger } from "web-video-creator";

const wvc = new WebVideoCreator();

// Configure WVC
wvc.config({
    // Choose an appropriate encoder based on your hardware. Here, we are using the h264_nvenc encoder for Nvidia graphics cards.
    // Encoder selection can refer to docs/video-encoder.md
    mp4Encoder: VIDEO_ENCODER.NVIDIA.H264
});

// Create a single-scene video
const video = wvc.createSingleVideo({
    // URL of the page to be rendered
    url: "http://localhost:8080/test.html",
    // or can directly set the page content:
    // content: "<h1>Hello WebVideoCreator</h1>",
    // Video width
    width: 1280,
    // Video height
    height: 720,
    // Video frame rate
    fps: 30,
    // Video duration
    duration: 10000,
    // Output path for the video
    outputPath: "./test.mp4",
    // Display progress bar in the command line
    showProgress: true
});

// Listen for the completion event
video.once("completed", result => {
    logger.success(`Render Completed!!!\nvideo duration: ${Math.floor(result.duration / 1000)}s\ntakes: ${Math.floor(result.takes / 1000)}s\nRTF: ${result.rtf}`)
});

// Start rendering
video.start();
```

## Rendering Multi Video

<img style="width:550px" src="./assets/multi-video.gif" />

```javascript
import WebVideoCreator, { VIDEO_ENCODER, TRANSITION, logger } from "web-video-creator";

const wvc = new WebVideoCreator();

// Configure WVC
wvc.config({
    // Choose an appropriate encoder based on your hardware. Here, we are using the h264_nvenc encoder for Nvidia graphics cards.
    // Encoder selection can refer to docs/video-encoder.md
    mp4Encoder: VIDEO_ENCODER.NVIDIA.H264
});

// Create a multi-scene video
const video = wvc.createMultiVideo({
    // Video width
    width: 1280,
    // Video height
    height: 720,
    // Video frame rate
    fps: 30,
    // Video segment parameters
    chunks: [
        {
            url: "http://localhost:8080/scene-1.html",
            // or can directly set the page content:
            // content: "<h1>Hello WebVideoCreator</h1>",
            duration: 10000,
            // Insert a circular crop transition between the first and second scenes
            transition: TRANSITION.CIRCLE_CROP
        },
        {
            url: "http://localhost:8080/scene-2.html",
            duration: 10000
        }
    ],
    // Output path for the video
    outputPath: "./test.mp4",
    // Display progress bar in the command line
    showProgress: true
});

// Listen for the completion event
video.once("completed", result => {
    logger.success(`Render Completed!!!\nvideo duration: ${Math.floor(result.duration / 1000)}s\ntakes: ${Math.floor(result.takes / 1000)}s\nRTF: ${result.rtf}`)
});

// Start rendering
video.start();
```

## Rendering Chunk Video and Combining into Multi Video

<img style="width:550px" src="./assets/chunk-video.gif" />

```javascript
import WebVideoCreator, { VIDEO_ENCODER, TRANSITION, logger } from "web-video-creator";

const wvc = new WebVideoCreator();

// Configure WVC
wvc.config({
    // Choose an appropriate encoder based on your hardware. Here, we are using the h264_nvenc encoder for Nvidia graphics cards.
    // Encoder selection can refer to docs/video-encoder.md
    mp4Encoder: VIDEO_ENCODER.NVIDIA.H264
});

// Create chunk video 1
const chunk1 = wvc.createChunkVideo({
    url: "http://localhost:8080/scene-1.html",
    // or can directly set the page content:
    // content: "<h1>Hello WebVideoCreator</h1>",
    width: 1280,
    height: 720,
    fps: 30,
    duration: 10000,
    showProgress: true
});

// Create chunk video 2
const chunk2 = wvc.createChunkVideo({
    url: "http://localhost:8080/scene-2.html",
    width: 1280,
    height: 720,
    fps: 30,
    duration: 10000,
    showProgress: true
});

// Wait for the chunks to finish rendering
await Promise.all([chunk1.startAndWait(), chunk2.startAndWait()]);

// Set the transition effect between chunk1 and chunk2 to fade in and out
chunk1.setTransition({ id: TRANSITION.FADE, duration: 500 });

// Create a multi-scene video
const video = wvc.createMultiVideo({
    width: 1280,
    height: 720,
    fps: 30,
    // Video segments
    chunks: [
        chunk1,
        chunk2
    ],
    // Output path for the video
    outputPath: "./test.mp4",
    // Display progress bar in the command line
    showProgress: true
});

// Listen for the completion event
video.once("completed", result => {
    logger.success(`Render Completed!!!\nvideo duration: ${Math.floor(result.duration / 1000)}s\ntakes: ${Math.floor(result.takes / 1000)}s\nRTF: ${result.rtf}`)
});

// Start rendering
video.start();
```

## Global Configuration

You can globally configure WVC to adjust some common parameters.

```javascript
import WebVideoCreator, { VIDEO_ENCODER, AUDIO_ENCODER } from "web-video-creator";

const wvc = new WebVideoCreator();

wvc.config({
    // Enable this to output some debug logs from WVC
    debug: true,
    // Specify the Chrome browser version to use
    browserVersion: "...",
    // It is recommended to enable this. If disabled, the window will be displayed and requires compatibility rendering mode to render properly. It's only used for debugging visuals.
    browserHeadless: true,
    // Frame rate limit is enabled by default. Disabling it can improve rendering efficiency and support animations above 60fps. However, this might disable GPU vertical sync, potentially causing screen tearing or other issues.
    browserFrameRateLimit: true,
    // Enable this to output browser runtime logs
    browserDebug: true,
    // Enable this to output each executed FFmpeg command
    ffmpegDebug: true,
    // Path to the FFmpeg executable file; setting this will disable the internal ffmpeg-static; it's recommended to use the internal FFmpeg for completeness
    ffmpegExecutablePath: "...",
    // Path to the ffprobe executable file; setting this will disable the internal ffprobe-static; it's recommended to use the internal ffprobe for completeness
    ffprobeExecutablePath: "...",
    // Browser GPU acceleration switch; recommended to enable for improved rendering performance; if you don't have a GPU or encounter rendering issues, you can disable it
    browserUseGPU: true,
    // Path to the browser executable file; setting this will disable the internal browser; it's recommended to use the internal browser for completeness
    browserExecutablePath: "...",
    // Allow unsafe contexts, false by default. Once enabled, users can navigate to unsafe URLs, but due to unsafe context restrictions, dynamic images and embedded videos cannot be used on the page
    allowUnsafeContext: false,
    // Compatible Rendering Mode is necessary on MacOS but not recommended for other environments; enabling this will disable HeadlessExperimental.beginFrame API calls and use regular Page.screenshot, which can lead to decreased rendering performance and frame rate desynchronization in some animations; you can try enabling it if you encounter the error "TargetCloseError: Protocol error (HeadlessExperimental.beginFrame): Target closed"
    compatibleRenderingMode: false,
    // Minimum number of browser instances in the resource pool
    numBrowserMin: 1,
    // Maximum number of browser instances in the resource pool
    numBrowserMax: 5,
    // Minimum number of page instances per browser instance
    numPageMin: 1,
    // Maximum number of page instances per browser instance
    numPageMax: 5,
    // User-agent string to use when accessing pages
    userAgent: null,
    // Frame capture quality (0-100); only effective for jpeg
    frameQuality: 80,
    // Frame image format (jpeg/png); it's recommended to use jpeg as png capture is more time-consuming
    frameFormat: "jpeg",
    // Timeout for capturing images with BeginFrame
    beginFrameTimeout: 5000,
    // Video encoder for MP4 format; by default, it uses the libx264 software encoder; it's recommended to choose the appropriate hardware encoder for acceleration based on your hardware, Encoder selection can refer to docs/video-encoder.md.
    mp4Encoder: VIDEO_ENCODER.CPU.H264,
    // Video encoder for WEBM format; by default, it uses the libvpx software encoder; it's recommended to choose the appropriate hardware encoder for acceleration based on your hardware
    webmEncoder: VIDEO_ENCODER.CPU.VP8,
    // Audio encoder; it's recommended to use the default AAC encoder
    audioEncoder: AUDIO_ENCODER.AAC
});
```

## Inserting Audio

To add audio to your rendered HTML, simply include an `<audio>` element with the desired audio file. You can also set attributes like `loop`, and WVC will automatically include the audio track for looping in the video.

```html
<audio src="bgm.mp3" loop></audio>
```

You can also set various attributes to control the audio's behavior. These attributes do not always need to be paired, so you can customize them according to your needs.

```html
<!-- Control the audio volume to half the original level -->
<audio src="bgm.mp3" volume="0.5"></audio>
<!-- Start playing the audio after 3 seconds and stop it at 10 seconds -->
<audio src="bgm.mp3" startTime="3000" endTime="10000"></audio>
<!-- Loop a segment of the audio from the 5th second to the 15th second -->
<audio src="bgm.mp3" seekStart="5000" seekEnd="15000" loop></audio>
<!-- Apply a 300ms fade-in and 500ms fade-out to the audio -->
<audio src="bgm.mp3" fadeInDuration="300" fadeOutDuration="500"></audio>
```

You can also dynamically add and remove `<audio>` elements in your code to control audio entering and exiting the scene. WVC will detect them.

```javascript
const audio = document.createElement("audio");
audio.src = "bgm.mp3";
// Audio enters the scene at 3 seconds
setTimeout(() => document.body.appendChild(audio), 3000);
// Audio exits the scene at 8 seconds
setTimeout(() => audio.remove(), 8000);
```

Or call [captureCtx.addAudio](./docs/capture-ctx.md#capturecontextaddaudiooptions-object) on the page to add audio to the video.

```javascript
// Add a single audio track
captureCtx.addAudio({
    url: "bgm.mp3",
    startTime: 500,
    loop: true,
    // 80% volume
    volume: 80
});
// Add multiple audio tracks
captureCtx.addAudios([...]);
```

You can also directly use [addAudio](./docs/api-reference-high-level.md#singlevideoaddaudiooptions-object) in WVC to add local or remote audio to the video.

```javascript
const video = wvc.createSingleVideo({ ... });
// Add a single audio track
video.addAudio({
    // url: "http://.../bgm.mp3"
    path: "bgm.mp3",
    startTime: 500,
    loop: true,
    // 80% volume
    volume: 80
});
// Add multiple audio tracks
video.addAudios([...]);
```

This operation also applies to MultiVideo and ChunkVideo.

## Inserting Video

Currently, WVC supports `mp4` and `webm` video formats. To insert a video into your rendered HTML, include a `<video>` element with the desired video file. You can set attributes like `loop` and `muted`. If your src does not contain the `. mp4` suffix name, it may not be recognized. Please add the `capture` attribute to identify the element that needs to be captured.

```html
<video src="background.mp4" loop muted></video
```

If you wish to insert a video with a transparent channel, see: [Transparent Channel Videos](#transparent-channel-videos). If you are interested in video frame rate synchronization or transparent video rendering, you can refer to: [Technical Implementation](#technical-implementation).

Similar to audio, you can set various attributes to control the video's behavior, and these attributes do not always need to be paired.

```html
<!-- Control the audio volume to 70% of the original -->
<video src="test.mp4" volume="0.7"></video>
<!-- Start playing the video after 3 seconds and stop it at 10 seconds -->
<video src="test.mp4" startTime="3000" endTime="10000"></video>
<!-- Loop a segment of the video from the 5th second to the 15th second -->
<video src="test.mp4" seekStart="5000" seekEnd="15000" loop></video>
<!-- Apply a 300ms fade-in and 500ms fade-out to the video -->
<video src="test.mp4" fadeInDuration="300" fadeOutDuration="500"></video>
```

You can dynamically add and remove `<video>` elements in your code to control video entering and exiting the scene. WVC will detect them.

```javascript
const video = document.createElement("video");
video.src = "test.mp4";
// Video enters the scene at 3 seconds
setTimeout(() => document.body.appendChild(video), 3000);
// Video exits the scene at 8 seconds
setTimeout(() => video.remove(), 8000);
```

If you are using some front-end frameworks to implement animation content, WVC may not be able to listen to your changes to the `<video>` element (such as hiding or displaying). Please replace the element with the `<canvas video-capture>` element and prompt WVC to notice that it is a video canvas through `video-capture` attribute.

```html
<canvas src="test.mp4" video-capture></canvas>
```

### Transparent Channel Videos

Transparent videos are great for compositing digital avatars (e.g., VTubers) into video scenes. Please refer to **[Rendering Example Page](https://github.com/Vinlic/WebVideoCreator/wiki/Rendering-Example)** Last Demo

In WVC, transparent videos should be in the `webm` format. Internally, they will be re-encoded into two mp4 container videos: one for the color base video and one for the mask video. These videos will be used for blending and drawing using the `globalCompositeOperation` in the browser canvas.

For users, it's seamless. You just need to include a `<video>` element in your HTML with the `src` set to the webm video file.

```html
<video src="vtuber.webm"></video>
```

Webm encoding and decoding can be time-consuming. If you can obtain the original mp4 video and the mask mp4 video, it's a better solution. Just add the `maskSrc` attribute.

```html
<video src="vtuber.mp4" maskSrc="vtuber_mask.mp4"></video>
```

## Inserting Animated Images

Animated images refer to sequence frame animations in `gif` / `apng` / `webp` formats. They can naturally play in the browser, but their frame rate is usually uncontrollable. WVC proxies their rendering, replacing `img` elements with `canvas` elements, and uses ImageDecoder to decode and draw each frame in sync with virtual time.

The following animated images can be rendered as well, and you can style them as usual.

```html
<img src="test.gif"/>
<img src="test.apng"/>
<img src="test.webp"/>
```

If you are using some front-end frameworks to implement animation content, WVC may not be able to listen to your changes to the `<img>` element (such as hiding or displaying). Please replace the element with the `<canvas dyimage-capture>` element and prompt WVC to notice that it is a dyanmic image canvas through `dyimage-capture` attribute.

```html
<canvas src="test.gif" dyimage-capture></canvas>
```

## Inserting Lottie Animations

WVC comes with the built-in [lottie-web](http://airbnb.io/lottie/#/web) animation library. If you have your own Lottie animations in your web page, they should work seamlessly with WVC.

Simply insert a `<lottie>` element and set the `src` attribute.

```html
<lottie src="example.json"></lottie>
```

If you are using some front-end frameworks to implement animation content, WVC may not be able to listen to your changes to the `<lottie>` element (such as hiding or displaying). Please replace the element with the `<canvas lottie-capture>` element and prompt WVC to notice that it is a lottie canvas through `lottie-capture` attribute.

```html
<canvas src="example.json" lottie-capture></canvas>
```

## Applying Fonts

WVC can detect `@font-face` declarations in stylesheets and wait for the fonts to load before starting rendering.

```html
<style>
    @font-face {
        font-family: "FontTest";
        src: url("font.ttf") format("truetype");
    }
</style>
<p style='font-family: "FontTest"'>Hello World</p>
```

Alternatively, you can register local or remote fonts through code.

```javascript
const video = wvc.createSingleVideo({ ... });
// Register a single font
video.registerFont({
    // url: "http://.../font.ttf"
    path: "font.ttf",
    family: "FontTest",
    format: "truetype"
});
// Register multiple fonts
video.registerFonts([...]);
```

Make sure the fonts can be loaded; otherwise, rendering may not start.

## Inserting Transition Effects

WVC supports the use of [Xfade](https://trac.ffmpeg.org/wiki/Xfade) filters supported by FFmpeg to create transition effects. You can refer to the [list of transitions](./docs/transition.md).

Each chunk video parameter can be configured with a transition effect and its duration.

```javascript
import WebVideoCreator, { TRANSITION } from "web-video-creator";

...

const video = wvc.createMultiVideo({
    ...
    // Video segment parameters
    chunks: [
        {
            url: "http://localhost:8080/scene-1.html",
            duration: 10000,
            // Insert a fade in/out transition between the first and second scenes
            transition: {
                id: TRANSITION.FADE,
                duration: 500
            },
            // If you don't need to set the duration, you can directly set the transition ID
            // transition: TRANSITION.FADE
        },
        {
            url: "http://localhost:8080/scene-2.html",
            duration: 10000
        }
    ],
    ...
});

...
```

It's important to note that applying transitions will result in a shorter total video duration since the transition effect effectively overlaps a portion of two video segments. For example, if you insert a fade transition between two 5-second segments, the resulting video will have a duration of 9 seconds.

Lottie animations are also suitable for use as transition effects. You can play a full-screen Lottie animation for half of the duration at the end of one video segment and then play another full-screen Lottie animation for the remaining half at the beginning of the next video segment to create more dynamic transition effects.

## Export videos with transparent channels

WVC supports you to set the background opacity `backgroundOpacity` option to achieve transparent or semi transparent background video output, with a value range of **0-1**, Please ensure that the output video path suffix name or format option is **webm**.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set a completely transparent background
    backgroundOpacity: 0
});
```

## Delayed Rendering Start

By default, WVC starts rendering immediately after the page navigation is complete. If you

 want to perform some tasks before rendering, you can disable automatic rendering start in the options. In this case, remember to call [captureCtx.start()](./docs/capture-ctx.md#capturecontextstart) in your page code, or rendering will be blocked indefinitely.

```javascript
const video = wvc.createSingleVideo({
    url: "http://localhost:8080/test.html",
    width: 1280,
    height: 720,
    duration: 10000,
    // Disable automatic rendering start
    autostartRender: false
});
```

In your page code, call the start function when appropriate.

```html
<script>
    // 数据加载完成后启动渲染
    loadData()
        .then(() => captureCtx.start())
        .catch(err => console.error(err));
</script>
```

<br>

## Start Capturing at a Specified Time Point

By default, WVC begins capturing the screen from the 0 time point after rendering starts. However, it also supports initiating screen capture from other time points.

```javascript
const video = wvc.createSingleVideo({
    url: "http://localhost:8080/test.html",
    width: 1280,
    height: 720,
    // Start capturing from the 5-second mark
    startTime: 5000,
    duration: 10000
});
```

<br>

## Pre-render Page Operations

```javascript
const video = wvc.createSingleVideo({
    url: "http://localhost:8080/test.html",
    width: 1280,
    height: 720,
    duration: 10000,
    pagePrepareFn: async page => {
        // Get the puppeteer Page object
        const _page = page.target;
        // Click the button
        await _page.tap("#play-button");
    }
});
```

## Using Action Sequences

WVC allows you to set actions to be executed at specific time points within a video. This feature enables convenient manipulation of a page at arbitrary intervals within a video. The following code is used to perform scrolling at the 3rd, 6th, and 9th seconds of the video.

```javascript
const actionFn = async (page) => {
    const _page = page.target;
    await _page.evaluate(() => {
        window.scrollTo({
            top: window.scrollY + 1280,
            behavior: "smooth"
        });
    });
};
const video = wvc.createSingleVideo({
    width: 1080,
    height: 1280,
    fps: 60,
    outputPath: "./t2.mp4",
    showProgress: true,
    url: "https://www.bilibili.com/v/popular/all/",
    // Setting up the action sequence
    timeActions: {
        3000: actionFn,
        6000: actionFn,
        9000: actionFn
    },
    duration: 10000
});
```

## Page Console Output

If you want to see the page's logs, you can enable `consoleLog` in the video options. If there are embedded videos, you can also enable `videoPreprocessLog` to output video preprocessing logs.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Output logs from the page's console
    consoleLog: true,
    // Output video preprocessing logs for embedded videos
    videoPreprocessLog: true
});
```

## Capture a Cover Image

After rendering a video, you can capture a frame image and save it, which can be used as the video's cover image.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Enable cover image capture
    coverCapture: true,
    // Time in milliseconds to capture the image (default is 20% of the video's duration)
    coverCaptureTime: 1000,
    // Image format for capture (jpg/png/bmp), default is jpg
    coverCaptureFormat: "jpg"
});
```

## Insert a Cover Image

WVC supports inserting an image into the first frame of the video, which will be displayed when the video is not playing.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set the path to the additional cover image, supports jpg/png/bmp
    attachCoverPath: "./cover.jpg"
});
```

## Adjust Video Volume

You can control the overall volume of the output video.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set the video volume to 80% of the original
    volume: 80
});
```

## Control Output Video Quality

WVC allows you to control video image quality using `videoQuality` or `videoBitrate`.

`videoQuality` is used to calculate the bitrate based on the total pixel count of the image. The following is the method used by WVC to calculate the video bitrate internally:

```javascript
const pixels = width * height;
const videoBitrate = (2560 / 921600 * pixels) * (videoQuality / 100);
```

You can provide `videoQuality` (0-100) in the video options:

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set video quality to 80%
    videoQuality: 80
});
```

If you find the bitrate inappropriate, you can set `videoBitrate` separately:

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set video bitrate to 8Mbps
    videoBitrate: "8192k"
});
```

Additionally, you can adjust the frame image quality when using jpeg as the frame format by setting `frameQuality`. For details, see [Global Configuration](#global-configuration).

You can also adjust audio quality by setting the audio bitrate `audioBitrate`:

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set audio bitrate to 320Kbps
    audioBitrate: "320k"
});
```

## Change Pixel Format

WVC currently supports output in `yuv420p`, `yuv444p`, and `rgb24` pixel formats. It defaults to using the more compatible `yuv420p`. If you notice significant color differences between the output video and the original page, you can switch to `rgb24` to improve the issue.

```javascript
const video = wvc.createSingleVideo({
    ...,
    // Set the pixel format to rgb24
    pixelFormat: "rgb24"
});
```

<br>

# Video Encoder Selection

When rendering frames from the browser and feeding them into FFmpeg, it's necessary to encode the image data at the specified frame rate and store it in a specified format container using a video encoder. Video encoding is a resource-intensive operation, and choosing a hardware encoder can accelerate this process and reduce CPU load.

Please refer to the [Video Encoder Guide](./docs/video-encoder.md) for a list of supported video encoders by WVC.

<br>

# Progress Monitoring

You can use the `progress` event of the video instance to monitor the rendering and synthesis progress.

```javascript
const video = wvc.createSingleVideo({ ... });
video.on("progress", (progress, synthesizedFrameCount, totalFrameCount) => {
    // Output progress / synthesized frame count / total frame count
    console.log(progress, synthesizedFrameCount, totalFrameCount);
});
```

This also applies to `MultiVideo` / `ChunkVideo` and the synthesisers in the low-level API.

<br>

# Exception Handling

## Throwing Errors

You can actively throw errors in the page to interrupt rendering.

```html
<script>
    captureCtx.throwError(-1, "Abort");
</script>
```

## Listening for Page Crashes

If your page involves heavy computations or consumes excessive memory, it may crash, causing rendering to be interrupted.

If you're using the high-level API, you can be notified of page crashes through the `pageCrashed` event of the video instance.

```javascript
const video = wvc.createSingleVideo({ ... });
// Output crash errors when an error occurs
video.on("pageCrashed", err => console.error(err));
```

When using the low-level API, you can listen for page crashes through the `crashed` event of the Page instance.

```javascript
// Output crash errors when an error occurs
page.on("crashed", err => console.error(err));
```

## Listening for Other Errors

If you're using the high-level API, you can listen for errors through the `error` event of the video instance.

```javascript
const video = wvc.createSingleVideo({ ... });
video.on("error", err => console.error(err));
```

When using the low-level API, you can listen for errors through the `error` event of the Page instance.

```javascript
page.on("error", err => console.error(err));
```

## Compatible Rendering Mode

On MacOS, due to Chrome's lack of support for the BeginFrame API, it is necessary to switch to the compatible rendering mode for proper functionality. This mode may result in a decrease in rendering efficiency of around 40%. It is recommended to deploy on Windows or Linux devices for better performance.

```javascript
// Enable compatible rendering mode
wvc.config({ compatibleRenderingMode: true });
```

<br>

# Cache Management

To optimize task launching and rendering time, WVC generally maintains four types of caches: `Browser Cache`, `Preprocess Cache`, `Synthesize Cache`, and `Local Font Cache`.

**Browser Cache**: Contains cached files and records generated by the page, reducing page loading time.

**Preprocess Cache**: When rendering content references remote resources, the preprocessor attempts to fetch and cache them locally, reducing bandwidth consumption.

**Synthesize Cache**: When combining multiple video chunks into a whole, pre-rendered chunks serve as the synthesize cache. Typically, WVC clears this cache after synthesis is complete.

**Local Font Cache**: When using `registerFonts` to register local fonts for successful injection into the page, a copy of fonts from the source path is stored in the cache.

## Clearing Cache

```javascript
// Clear browser cache
wvc.cleanBrowserCache();
// Clear preprocess cache
wvc.cleanPreprocessCache();
// Clear synthesize cache
wvc.cleanSynthesizeCache();
// Clear local font cache
wvc.cleanLocalFontCache();
```

<br>

# API Reference

## High-Level API

In most cases, it is recommended to use the high-level API because it is simple but may be less flexible.

[API Reference High Level](./docs/api-reference-high-level.md)

## Low-Level API

[API Reference Low Level](./docs/api-reference-low-level.md)

<br>

# Distributed Rendering

If you have multiple devices available for rendering, you can deploy WVC on these devices. WVC provides `MultiVideo` and `ChunkVideo`, allowing you to divide the animation pages into many segments (e.g., 0-10 seconds, 10-20 seconds, etc.). Distribute their parameters to different WVC instances on different devices, create ChunkVideo instances on these devices, and execute parallel rendering to generate multiple video segments (`ts`). These segments are then sent back to the core node, where they are combined, and transitions, audio tracks, and output are handled. **The distribution and return process is not yet implemented in WVC, but it is not difficult, and you can wrap it according to your own scenario. Contributions to WVC are welcome through [PR](https://github.com/Vinlic/WebVideoCreator/pulls)!**

<br>

# Performance Tips

Performance is typically influenced by the complexity of animations and media. To improve performance:

- Divide long animations into multiple segments. For instance, you can include a seek parameter with each page URL, load the page, and seek to a specified time to start playing. Then, render and combine them as a multi-video to significantly reduce rendering time for long videos.

- Render more video chunks in parallel. To maximize system resources, select the number of parallel chunks based on the number of CPU threads, assuming your system has sufficient memory.

- CPU clock speed has a significant impact on baseline performance. Consumer-grade CPUs often have high clock speeds, which can lead to better performance.

- Consider using GPU acceleration for rendering and compositing. If your device has a GPU but it's not being utilized, check the configuration settings or report the issue.

- Using an SSD (Solid State Drive) can improve hard disk cache write performance during parallel rendering, reducing rendering time.

- Select the right video hardware encoder. By default, software encoders are used (libx264 for mp4 and libvpx for webm). If you have integrated or discrete graphics, configure the hardware encoders they support.

- Some time may be spent on network file transfers. It's advisable to deploy static file services on the same server or access the file server from a local network.

- Reducing the output video resolution and frame rate is the most effective way to reduce time consumption.

Here are the performance parameters for my personal computer as a reference:

Operating System: Windows 10 (Better performance on Linux)

CPU: AMD Ryzen 7 3700X (Clock speed 3.6-4.4GHz, 8 cores, 16 threads)

GPU: Nvidia GeForce GTX 1660 SUPER (6GB VRAM, NVENC support)

RAM: 16GB (DDR4 2400MHz)

Video Type: SVG animations + GIFs + Lottie animations

Video Resolution: 1280x720

Video Frame Rate: 30

Video Duration: 300s (5 minutes)

Rendering Time: 61s (1 minute)

Real-Time Rate: 4.844

Parallel Rendering Count: 16

<br>

# Limitations

- Constrained by browser [secure context restrictions](https://w3c.github.io/webappsec-secure-contexts/), WebVideoCreator can only access `localhost` / `127.0.0.1` or domains using HTTPS with valid certificates. For security reasons, it's recommended to use a local static server (e.g., `live-server` is a good choice), You can also use the content option to directly set the page content to avoid URL restrictions.

- The headless experimental API on Mac systems may cause crashes and needs to be switched to compatibility rendering mode to run. However, compatibility rendering mode has various issues, so it is not recommended for Mac systems. See [Compatibility Rendering Mode](#compatible-rendering-mode).

- WebVideoCreator is a pure ESM package and cannot be imported using CommonJS-style `require`. If you still want to use `require` to import it, refer to this [gist](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c) for guidance.

- The page jump request during the video rendering process will be intercepted because jumping to the page will result in the loss of capture context.
