---
layout: default
---

<img src="https://cloud.githubusercontent.com/assets/2762250/7824984/985c3e76-03bc-11e5-807b-1402bde4fe56.png" width="400">

Build status (
CircleCI): [![CircleCI](https://circleci.com/gh/Dash-Industry-Forum/dash.js/tree/development.svg?style=svg)](https://circleci.com/gh/Dash-Industry-Forum/dash.js/tree/development)

[Join #dashjs on Slack!](https://join.slack.com/t/dashif/shared_invite/zt-egme869x-JH~UPUuLoKJB26fw7wj3Gg)

# Overview

A reference client implementation for the playback of MPEG DASH via JavaScript and browsers that support the Media
Source Extensions. 

If your intent is to use the player code without contributing back to this project, then use the `master` branch which
holds the approved and stable public releases.

If your goal is to improve or extend the code and contribute back to this project, then you should make your changes in,
and submit a pull request against, the `development` branch. Read
our [CONTRIBUTION.md](https://github.com/Dash-Industry-Forum/dash.js/blob/development/CONTRIBUTING.md) file for a
walk-through of the contribution process.

# Demos and reference players
All these reference builds and minified files are available under both http and https.

## Samples
Multiple [dash.js samples](https://reference.dashif.org/dash.js/latest/samples/index.html) covering a wide set of common
use cases.

## Reference players

The released [pre-built reference players ](http://reference.dashif.org/dash.js/) if you want direct access without
writing any Javascript.

The [nightly build of the /dev branch reference player](http://reference.dashif.org/dash.js/nightly/samples/dash-if-reference-player/index.html)
, is pre-release but contains the latest fixes. It is a good place to start if you are debugging playback problems.

# CDN hosted build files

The latest minified files have been hosted on a global CDN and are free to use in production:

- [dash.all.min.js](http://cdn.dashjs.org/latest/dash.all.min.js)
- [dash.all.debug.js](http://cdn.dashjs.org/latest/dash.all.debug.js)

In addition, all the releases are available under the following urls. Replace "vx.x.x" with the release version, for
instance "v3.1.0".

- [http://cdn.dashjs.org/vx.x.x/dash.all.min.js](http://cdn.dashjs.org/v3.1.0/dash.all.min.js)
- [http://cdn.dashjs.org/vx.x.x/dash.all.debug.js](http://cdn.dashjs.org/v3.1.0/dash.all.debug.js)


# API Documentation

Full [API Documentation](http://cdn.dashjs.org/latest/jsdoc/module-MediaPlayer.html) is available describing all public
methods, interfaces, properties, and events.

For help, join our [Slack channel](https://dashif-slack.azurewebsites.net),
our [email list](https://groups.google.com/d/forum/dashjs) and read
our [wiki](https://github.com/Dash-Industry-Forum/dash.js/wiki).

## Tutorials

Detailed information on specific topics can be found in our tutorials:

* [Low latency streaming](https://github.com/Dash-Industry-Forum/dash.js/wiki/Low-Latency-streaming)
* [UTCTiming Clock synchronization](https://github.com/Dash-Industry-Forum/dash.js/wiki/UTCTiming---Clock-synchronization)
* [Digital Rights Management (DRM) and license acquisition](https://github.com/Dash-Industry-Forum/dash.js/wiki/Digital-Rights-Management-(DRM)-and-license-acquisition)
* [Buffer and scheduling logic](https://github.com/Dash-Industry-Forum/dash.js/wiki/Buffer-and-Scheduling-Logic)

## Quick Start for Developers

1. Install Core Dependencies
    * [install nodejs](http://nodejs.org/)
2. Checkout project repository (default branch: development)
    * ```git clone https://github.com/Dash-Industry-Forum/dash.js.git```
3. Install dependencies
    * ```npm install```
4. Build, watch file changes and launch samples page, which has links that point to reference player and to other
   examples (basic examples, captioning, ads, live, etc).
    * ```npm run start```

### Other Tasks to Build / Run Tests on Commandline.

* Build distribution files (minification included)
    * ```npm run build```
* Build and watch distribution files
    * ```npm run dev```
* Run linter on source files (linter is also applied when building files)
    * ```npm run lint```
* Run unit tests
    * ```npm run test```
* Generate API jsdoc
    * ```npm run doc```

### Troubleshooting

* In case the build process is failing make sure to use an up-to-date node.js version. The build process was
  successfully tested with node.js version 14.16.1.

### License

dash.js is released under [BSD license](https://github.com/Dash-Industry-Forum/dash.js/blob/development/LICENSE.md)

### Tested With

[<img src="https://cloud.githubusercontent.com/assets/7864462/12837037/452a17c6-cb73-11e5-9f39-fc96893bc9bf.png" alt="Browser Stack Logo" width="300">](https://www.browserstack.com/)

