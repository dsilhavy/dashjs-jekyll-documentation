---
layout: default
title: Low Latency Streaming
parent: Advanced Features
nav_order: 1
---

# Low Latency Streaming

One of the major challenges in OTT streaming is reducing the live streaming latency. This can be crucial for live events
like sport games or for an optimal streamer-user interaction in eSports games.

## Use case: On Par with Other Distribution Means

A live event is distributed over DASH as well over regular TV distribution. The event should play-out approximately at
the same time on both devices in order to avoid different perceptions of the same service when received over different
distribution means. The objective should be to get to a range of delay for the DASH based service that is equivalent to
cable and IPTV services [1].

## Use case: Sports Bar

Sports bars are commonly in close proximity to each other and may all show the same live sporting event. Some bars may
be using a provider which distributes the content using DVB-T or DVB-S services whilst others may be using DASH ABR.
Viewers in a bar with a high latency will have their viewing spoiled as they will hear cheers for the goal before it
occurs on their local screen.

This creates a commercial incentive for the bar operator to switch to the provider with the lowest latency. The
objective should be to get the latency range to not be perceptibly different to that of a DVB broadcast solution for
those users who have a sufficient quality (high and consistent speed) connection [1].

### Use case: Professional streamer with interactive chat

Professional streamers interacting with a live audience on social media, often via a directly coupled chat function in
the viewing app/environment. They can generate direct revenue in several ways including:

* In stream advertising
* Micropayments (for example Twitch “bits”)

A high degree of interactivity between the performer and the audience is required to enable engagement. Lower latencies
increases the engagement and consequently the incentive for the audience members to reward the performer with likes,
shares, subscribes, micropayments, etc.

Typical use cases include gamers, musicians and other performers where in some part the direction of the performance can
be guided by the audience response [1].

### Use case: Sports betting

A provider wants to offer a live stream that will be used for wagering within an event. The content must be delivered
with low latency and more importantly within a well-defined sync across endpoints so customers trust the game is fair.
There are in some cases legal considerations, for example the content cannot be shown if it is more than X seconds
behind live.

Visual and aural quality are secondary in priority in these scenarios to sync and latency. The lower the latency the
more opportunities for “in play betting” within the game/event. This in turn increases revenue potential from a
game/event [1].

## CMAF low latency streaming

The Common Media Application Format introduces the concept of "chunks". A CMAF chunk has multiple "moof" and "mdat"
boxes, allowing the client to access the media data before the segment is completely finished. The benefits of the
chunked mode become more obvious when looking at a concrete example:

![Low Latency streaming]({{site.baseurl}}/assets/images/llstreaming.png)

So let’s assume we have 8 second segments and we are currently 3 seconds into segment number four. For classic media
segments, this leaves us with two options:

* Option 1: since segment four is not completed, we start with segment three. That way, we end up 11 seconds behind the
  live edge – 8 seconds coming from segment three, and 3 seconds coming from segment four.
* Option 2: we wait for segment four to finish and immediately start downloading and playing it. We end up with 8
  seconds of latency and a waiting time of 5 seconds.

With CMAF chunks, on the other hand, we are able to play segment four before it is completely available. In the example
above, we have CMAF chunks with a 1 second duration, which leads to eight chunks per segment. Let’s assume that only the
first chunk contains an IDR frame and therefore we always need to start the playback from the beginning of a segment.
Being three seconds into segment four leaves us with 3 seconds of latency. That’s much better than what we achieved with
classic segments. We can also fast decode the first chunks and play even closer to the live edge [2].

## CMAF low latency streaming with dash.js

dash.js supports CMAF low latency streaming since version 2.6.8. For that reason, a dedicated sample page is available:

* [Low latency sample page](https://reference.dashif.org/dash.js/nightly/samples/low-latency/testplayer/testplayer.html)

### dash.js configuration

The following Sections below will give a detailed explanation on L2ALL and LoL+. Some parameters are valid for all low
latency algorithms:

| Parameter        | Description                                                                                                                                                                                                                                                                                                                  |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| liveDelay        | Lowering this value will lower latency but may decrease the player's ability to build a stable buffer.                                                                                                                                                                                                                       |
| maxDrift         | Maximum latency deviation allowed before dash.js to do a seeking to live position                                                                                                                                                                                                                                            |
| playbackRate     | Maximum catch-up rate, as a percentage, for low latency live streams.                                                                                                                                                                                                                                                        |
| latencyThreshold | The maximum threshold for which live catch up is applied. For instance, if this value is set to 8 seconds, then live catchup is only applied if the current live latency is equal or below 8 seconds. The reason behind this parameter is to avoid an increase of the playback rate if the user seeks within the DVR window. |

The corresponding API call looks the following:

```javascript
player.updateSettings({
    streaming: {
        delay: {
            liveDelay: 4
        },
        liveCatchup: {
            maxDrift: 0,
            playbackRate: 0.5,
            latencyThreshold: 60
        }
    }
});
```

Please check the [API documentation](http://cdn.dashjs.org/latest/jsdoc/module-Settings.html) for additional
information.

### MPD specific low latency parameters

It is also possible to configure specific low latency settings via MPD. The required information is encapsulated in
a `<ServiceDescription>` element:

```xml

<ServiceDescription id="0">
    <Latency max="6000" min="2000" referenceId="0" target="4000"/>
    <PlaybackRate max="1.04" min="0.96"/>
</ServiceDescription>
```

For more details please refer to the DASH-IF IOP guidelines [4].

### dash.js requirements

In order to use dash.js in low latency mode the following requirements have to be fullfilled:

#### Client requirements

> * The [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) must be supported on the client browser.
> * The server must support [HTTP 1.1 Chunked transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)

#### Server and content requirements

The content and the manifest must be conditioned to support CMAF low latency chunks

The manifest must contain two additional attributes

* @availabilityTimeComplete: specifies if all segments of all associated representations are complete at the adjusted
  availability start time. If the value is set to false, then it may be inferred by the client that the segment is
  available at its announced location prior to completion.
* @availabilityTimeOffset (ATO): provides the time in how much earlier segments are available compared to their computed
  availability start time (AST).

The segments must contain multiple CMAF chunks. This will result in multiple "moof" and "mdat" boxes per segment.
Example

````
[styp] size=8+16
[prft] size=8+24
[moof] size=8+96
  [mfhd] size=12+4
    sequence number = 827234
  [traf] size=8+72
    [tfhd] size=12+16, flags=20038
      track ID = 1
      default sample duration = 1001
      default sample size = 15704
      default sample flags = 1010000
    [tfdt] size=12+8, version=1
      base media decode time = 828060233
    [trun] size=12+12, flags=5
      sample count = 1
      data offset = 112
      first sample flags = 2000000
[mdat] size=8+15704
[prft] size=8+24
[moof] size=8+92
  [mfhd] size=12+4
    sequence number = 827235
  [traf] size=8+68
    [tfhd] size=12+16, flags=20038
      track ID = 1
      default sample duration = 1001
      default sample size = 897
      default sample flags = 1010000
    [tfdt] size=12+8, version=1
      base media decode time = 828061234
    [trun] size=12+8, flags=1
      sample count = 1
      data offset = 108
[mdat] size=8+897
[prft] size=8+24
[moof] size=8+92
  [mfhd] size=12+4
    sequence number = 827236
  [traf] size=8+68
    [tfhd] size=12+16, flags=20038
      track ID = 1
      default sample duration = 1001
      default sample size = 7426
      default sample flags = 1010000
    [tfdt] size=12+8, version=1
      base media decode time = 828062235
    [trun] size=12+8, flags=1
      sample count = 1
      data offset = 108
[mdat] size=8+7426
````

# Challenges in low latency streaming

Compared to ABR algorithms for "classic" live streaming an ABR algorithm for low latency streaming has to overcome
additional challenges.

## Challenge 1: Throughput estimation

Common throughput based ABR algorithms calculate the available bandwidth on the client side using the download time for
a segment:

````
Calculated Throughput = (Segment@Bitrate*Segment@duration) / DownloadTime

Example: 

Calculated Throughput = (6Mbit/s * 6s) / 3s = 12 Mbit/s
````

The concept described above is a problem for clients operating in low latency mode. Since segments are transferred via
HTTP 1.1 Chunked transfer encoding the download time of a segment is often times similar to its duration. The download
of a segment is started prior to its completion. Therefore, the data is still generated on the server side and arrives
in small chunks at the client side .

For instance, the download time for a segment with six second duration will be approximately six seconds. There will be
idle times in which no data is transferred from the server to the client. However, the connection remains open while the
client waits for new data. The total download time includes these idle times. Consequently, the total download time is
not a good indicator for the available bandwidth on the client side.

### Low latency throughput estimation in dash.js

dash.js offers two different modes for low latency throughput estimation

#### Default throughput estimation

For every segment that is downloaded the default algorithm saves the timestamp and the length of bytes received
throughout the download process. The data packets do not arrive at moof boundaries. For instance a single "data burst"
might contain multiple moof/mdat pairs. For every data point an entry in the corresponding array is created:

```javascript
 downloadedData.push({
    ts: Date.now(), // timestamp when the data arrived
    bytes: value.length // length of the data
});
```

After the download of a segment is completed, the array above is cleared and the throughput is calculated in the
following way:

```javascript
function _calculateDownloadedTimeByBytesReceived(downloadedData, bytesReceived) {
    downloadedData = downloadedData.filter(data => data.bytes > ((bytesReceived / 4) / downloadedData.length));
    if (downloadedData.length > 1) {
        let time = 0;
        const avgTimeDistance = (downloadedData[downloadedData.length - 1].ts - downloadedData[0].ts) / downloadedData.length;
        downloadedData.forEach((data, index) => {
            // To be counted the data has to be over a threshold
            const next = downloadedData[index + 1];
            if (next) {
                const distance = next.ts - data.ts;
                time += distance < avgTimeDistance ? distance : 0;
            }
        });
        return time;
    }
}
```

1. In the first step the downloadedData array is filtered and all entries that do not have a certain size are removed.
2. In the next step the average time distance between two consecutive data points is calculated
3. If time distance between two consecutive data points is smaller than the average time distance the time distance is
   added to the total download time
4. The total download time is used to calculate the throughput as described before. Using this approach the download
   time is no longer equal to the duration of the segment.

#### Moof based throughput estimation

In contrast to the default throughput algorithm, the moof based throughput estimation is based on saving the download
time for each CMAF chunk. For that reason, the start and the endtime of each chunk, starting with a moof box and ending
with an mdat box are saved:

```javascript
// Store the start time of each chunk download                             
const flag1 = boxParser.parsePayload(['moof'], remaining, offset);
if (flag1.found) {
    // Store the beginning time of each chunk download 
    startTimeData.push({
        ts: performance.now(),
        bytes: value.length
    });
}

const boxesInfo = boxParser.findLastTopIsoBoxCompleted(['moov', 'mdat'], remaining, offset);
if (boxesInfo.found) {
    const end = boxesInfo.lastCompletedOffset + boxesInfo.size;

    // Store the end time of each chunk download 
    endTimeData.push({
        ts: performance.now(),
        bytes: remaining.length
    });
}
```

The download time of the segment is calculated the following way:

```javascript
function _calculateDownloadedTimeByMoofParsing(startTimeData, endTimeData) {
    let datum, datumE;
    // Filter the first and last chunks in a segment in both arrays [StartTimeData and EndTimeData]
    datum = startTimeData.filter((data, i) => i > 0 && i < startTimeData.length - 1);
    datumE = endTimeData.filter((dataE, i) => i > 0 && i < endTimeData.length - 1);
    // Compute the download time of a segment based on the filtered data [last chunk end time - first chunk beginning time]
    let segDownloadTime = 0;
    if (datum.length > 1) {
        for (let i = 0; i < datum.length; i++) {
            if (datum[i] && datumE[i]) {
                let chunkDownladTime = datumE[i].ts - datum[i].ts;
                segDownloadTime += chunkDownladTime;
            }
        }

        return segDownloadTime;
    }
    return null;
}
```

#### dash.js configuration

The desired throughput calculation mode can be selected by changing the respective settings parameter:

Value | Mode
 --- | ---
ABR_FETCH_THROUGHPUT_CALCULATION_DOWNLOADED_DATA | Default throughput estimation
ABR_FETCH_THROUGHPUT_CALCULATION_MOOF_PARSING | Moof based throughput estimation

```javascript
player.updateSettings({
    streaming: {
        abr: {
            fetchThroughputCalculationMode: Constants.ABR_FETCH_THROUGHPUT_CALCULATION_DOWNLOADED_DATA
        }
    }
})
```

## Challenge 2: Maintaining a consistent live edge

When playing in low latency mode the client needs to maintain a consistent live edge allowing only small deviations
compared to the target latency.

### Maintaining a consistent live edge in dash.js

In order to maintain a consistent live edge dash.js either adjusts the playback rate of the video (catchup mechanism),
or performs a seek back to the live edge. The catchup behavior of dash.js based on the deviation compared to the target
latency is depicted below:

<img width="1076" alt="Bildschirmfoto 2022-05-03 um 10 35 34" src="https://user-images.githubusercontent.com/2427039/166425465-7734787e-8726-47b8-8f72-fe30572d91fe.png">

#### Default catchup mechanism

In order to determine whether the catchup mechanism should be enabled the following logic is applied:

```javascript
function _defaultNeedToCatchUp(currentLiveLatency, liveDelay, liveCatchupLatencyThreshold, minDrift) {
    try {
        const latencyDrift = Math.abs(_getLatencyDrift());

        return latencyDrift > 0;
    } catch (e) {
        return false;
    }
}
```

The latency drift is compared against the minimum allowed drift `minDrift`. In addition, the catchup mechanism is only
applied if the current live latency is smaller than the defined threshold in `latencyThreshold` (see dash.js
configuration above).

In case the catchup mechanism is applied the new playback rate is calculated the following way:

```javascript
function _calculateNewPlaybackRateDefault(liveCatchUpPlaybackRate, currentLiveLatency, liveDelay, bufferLevel, currentPlaybackRate) {
    const cpr = liveCatchUpPlaybackRate;
    const deltaLatency = currentLiveLatency - liveDelay;
    const d = deltaLatency * 5;

    // Playback rate must be between (1 - cpr) - (1 + cpr)
    // ex: if cpr is 0.5, it can have values between 0.5 - 1.5
    const s = (cpr * 2) / (1 + Math.pow(Math.E, -d));
    let newRate = (1 - cpr) + s;
    // take into account situations in which there are buffer stalls,
    // in which increasing playbackRate to reach target latency will
    // just cause more and more stall situations
    if (playbackStalled) {
        // const bufferLevel = getBufferLevel();
        if (bufferLevel > liveDelay / 2) {
            // playbackStalled = false;
            playbackStalled = false;
        } else if (deltaLatency > 0) {
            newRate = 1.0;
        }
    }

    // don't change playbackrate for small variations (don't overload element with playbackrate changes)
    if (Math.abs(currentPlaybackRate - newRate) <= minPlaybackRateChange) {
        newRate = null;
    }

    return {
        newRate: newRate
    };
}
```

Note that the new playback rate must differ from the current playback rate by a hardcoded threshold:

```
minPlaybackRateChange = isSafari ? 0.25 : 0.02;
```

#### LoL+ based catchup mechanism

The LoL+ based catchup mechanism follows the same principles as the default catchup mechanism. In the first step dash.js
checks if the catchup mechanism should be applied:

```javascript
function _lolpNeedToCatchUpCustom(currentLiveLatency, liveDelay, minDrift, currentBuffer, playbackBufferMin, liveCatchupLatencyThreshold) {
    try {
        const latencyDrift = Math.abs(_getLatencyDrift());

        return latencyDrift > 0 || currentBuffer < playbackBufferMin;
    } catch (e) {
        return false;
    }
}
```

Compared to the default catchup mechanism, the LoL+ based catchup check uses `playbackBufferMin`. If either the latency
drift is larger than the minimum allowed drift `minDrift` or the current buffer length is smaller than the minimum
buffer `playbackBufferMin` the catchup mode is activated.

> Note that a change of playback rate can also mean that the playback rate is **decreased**. This can be useful to avoid buffer underruns.

The new playback rate is calculated in the following way:

```javascript
function _calculateNewPlaybackRateLolP(liveCatchUpPlaybackRate, currentLiveLatency, liveDelay, minDrift, playbackBufferMin, bufferLevel, currentPlaybackRate) {
    const cpr = liveCatchUpPlaybackRate;
    let newRate;

    // Hybrid: Buffer-based
    if (bufferLevel < playbackBufferMin) {
        // Buffer in danger, slow down
        const deltaBuffer = bufferLevel - playbackBufferMin;  // -ve value
        const d = deltaBuffer * 5;

        // Playback rate must be between (1 - cpr) - (1 + cpr)
        // ex: if cpr is 0.5, it can have values between 0.5 - 1.5
        const s = (cpr * 2) / (1 + Math.pow(Math.E, -d));
        newRate = (1 - cpr) + s;

        logger.debug('[LoL+ playback control_buffer-based] bufferLevel: ' + bufferLevel + ', newRate: ' + newRate);
    } else {
        // Hybrid: Latency-based
        // Buffer is safe, vary playback rate based on latency

        // Check if latency is within range of target latency
        const minDifference = 0.02;
        if (Math.abs(currentLiveLatency - liveDelay) <= (minDifference * liveDelay)) {
            newRate = 1;
        } else {
            const deltaLatency = currentLiveLatency - liveDelay;
            const d = deltaLatency * 5;

            // Playback rate must be between (1 - cpr) - (1 + cpr)
            // ex: if cpr is 0.5, it can have values between 0.5 - 1.5
            const s = (cpr * 2) / (1 + Math.pow(Math.E, -d));
            newRate = (1 - cpr) + s;
        }

        logger.debug('[LoL+ playback control_latency-based] latency: ' + currentLiveLatency + ', newRate: ' + newRate);
    }

    if (playbackStalled) {
        if (bufferLevel > liveDelay / 2) {
            playbackStalled = false;
        }
    }

    // don't change playbackrate for small variations (don't overload element with playbackrate changes)
    if (Math.abs(currentPlaybackRate - newRate) <= minPlaybackRateChange) {
        newRate = null;
    }

    return {
        newRate: newRate
    };
}

``` 

If the buffer level is smaller than the buffer level defined in `playbackBufferMin` the playback rate is decreased. If
the buffer is "safe", the playback rate is adjusted depending on the latency.

#### Calculating the new playback rate

Both catchup mechanisms share a common method to determine the calculation of the new playback rate:

```javascript
const s = (cpr * 2) / (1 + Math.pow(Math.E, -d));
newRate = (1 - cpr) + s;
```

* `cpr` is defined as the catchup `playbackRate`
* `d` is defined as a multiple of the delta latency or the delta buffer
* `Math.E` represents the base of natural logarithms, e, approximately 2.718.

If the current live latency is greater larger than the target latency `d` is positive, otherwise `d` is negative.
Consequently, if the playback rate needs to be incremented to reach the target latency the
equation `Math.pow(Math.E, -d)` will result in values smaller than 1. For negative `d` values, situations in which the
playback rate should be decreased, the equation `Math.pow(Math.E, -d)` will result in values greater than 1.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/Exp.svg/2880px-Exp.svg.png" height="400" />

As an example consider a situation in which the target latency is set to 2 seconds and the current latency equals 5
seconds:

```formula
cpr = 0.5
delta latency = current latency - target latency = 5 - 2 = 3
d = delta latency * 5 = 3 * 5 = 15 
s = (cpr * 2) / (1 + Math.pow(Math.E, -d)) = (0.5 * 2) / (1 + 3.0590232050182605e-7) = 0.999999694097773
new rate = (1 - cpr) + s = (1 - 0.5) + 0.999999694097773 = 1.499999694097773
```

The new rate will always stay within the target boundaries `1 +/- 0.5`

#### dash.js configuration

The desired catchup mechanism can be selected by changing the respective settings parameter:

Value | Mode
 --- | ---
LIVE_CATCHUP_MODE_DEFAULT | Default catchup mechanism
LIVE_CATCHUP_MODE_LOLP | LoL+ based catchup mechanism

```javascript
player.updateSettings({
    streaming: {
        liveCatchup: {
            mode: Constants.LIVE_CATCHUP_MODE_DEFAULT
        }
    }
})
```

#### Seeking to the live edge

In addition, both catchup algorithms share a common logic to seek back to the live edge. If the latency delta exceeds
the threshold defined in `maxDrift` the seek is performed:

```javascript
// we reached the maxDrift. Do a seek
const maxDrift = mediaPlayerModel.getCatchupMaxDrift();
if (!isNaN(maxDrift) && maxDrift > 0 &&
    deltaLatency > maxDrift) {
    logger.info('[CatchupController]: Low Latency catchup mechanism. Latency too high, doing a seek to live point');
    isCatchupSeekInProgress = true;
    _seekToLive();
}
```

# Low latency ABR algorithms in dash.js

With dash.js version 3.2.0 two low latency specific algorithms were implemented. A description of both algorithms LoL+
and L2A can found in the following.

## LoL+

LoL+ is designed as a series of sophisticated yet robust player improvements for low latency live (LLL) streaming. LoL+
consists of five essential modules:

1) The **bitrate selection module** implements a learning-based ABR algorithm to choose a suitable bitrate at each
   segment download. The ABR algorithm is based on an SOM model that considers multiple QoE metrics as well as bandwidth
   variability.
2) The **playback speed control module** implements a hybrid algorithm that considers both the current latency and
   buffer level to control the playback speed.
3) The **throughput measurement module** accurately calculates the throughout by removing the idle times between the
   chunks of a segment through a three-step algorithm.
4) The **QoE evaluation module** computes the QoE considering five key metrics: selected bitrate, number of bitrate
   switches, rebuffering duration, latency and playback speed.
5) Lastly, the **weight selection module** implements a two-step dynamic weight assignment for the SOM model features.
   We also added manual (equal value of 0.4 each) and random (based on Xavier formula) weight assignment for the SOM
   model features for comparison.

The modules can be found in the following files (as of dash.js v3.2):

1. Bitrate selection module (i.e., ABR algorithm):
   `dash.js/src/streaming/rules/abr/lolp/LoLpRule.js` and
   `LearningAbrController.js`
2. Playback speed control module:
   `dash.js/src/streaming/controllers/CatchupController.js`
3. Throughput measurement module:
   `dash.js/src/streaming/net/FetchLoader.js`
4. QoE evaluation module:
   `dash.js/src/streaming/rules/abr/lolp/LoLpQoeInfo.js` and `dash.js/src/streaming/rules/abr/lolp/LoLpQoEEvaluator.js`
5. Weight selection module:
   `dash.js/src/streaming/rules/abr/lolp/LoLpWeightSelector.js`

### Basic dash.js configuration

How to enable each of the LoL+ modules:

```javascript
// Code snippet from the samples Section `lolp_index.html`
player.updateSettings({
    streaming: {
        buffer: {
            stallThreshold: 0.05    // used in BufferController for more accurate stall tracking in low-latency live streaming
        }
        abr: {
            useDefaultABRRules: true,
            ABRStrategy: 'abrLoLP',    // to enable LoLp bitrate selection module
            fetchThroughputCalculationMode: 'abrFetchThroughputCalculationMoofParsing'    // to enable LoLp throughput measurement module
        },
        liveCatchup: {    // to enable LoLp playback speed control module
            playbackBufferMin: playbackBufferMin,
            mode: 'liveCatchupModeLoLP'
        }
    }
});
```

Note: The weight selection module is used in the bitrate selection module and does not need to be enabled separately.

### LoL+ example

An example for low latency streaming with LoL+ can be
found [here](http://reference.dashif.org/dash.js/nightly/samples/low-latency/lolp_index.html)

### Tuning parameters

Parameter | Description | Recommended value
--- | --- | ---
liveDelay | Lowering this value will lower latency but may decrease the player's ability to build a stable buffer. Note that this value affects several dash.js components (in addition to the calculation of playback rate), such as deriving which segment to request based on deviation from the live edge. | 1.5s-2s
playbackRate | Maximum catch-up rate, as a percentage, for low latency live streams. | 0.3 (i.e., 0.7x-1.3x).
playbackBufferMin | Minimum buffer allowed before activating buffer-based playback speed control (in place of latency-based). | 0.5s

The corresponding API call looks the following:

```javascript
player.updateSettings({
    streaming: {
        delay: {
            liveDelay: 2
        },
        liveCatchup: {
            playbackRate: 0.3,
            playbackBufferMin: 0.5
        }
    }
});
```

### Advanced tuning parameters

The parameters below are available but not advised to be changed. For advanced users, please refer to the paper [3] for
further details on these parameters. The following parameters are not exposed in the Settings object and have to be
changed in the respective classes.

Module | Parameter(s) | Remarks
--- | --- | ---
Bitrate selection module | targetLatency = 0 | SOM target latency
Bitrate selection module | targetRebufferLevel = 0 | SOM target rebuffering duration.
Bitrate selection module | targetSwitch = 0 | SOM target number of switches.
Bitrate selection module | throughputDelta = 10000 | SOM variation in throughput.
Weight selection module | DWS_TARGET_LATENCY = 1.5 | Latency constraint used in weight selection module (set in `LolpRule.js`).
Weight selection module | DWS_BUFFER_MIN = 0.3 | Minimum buffer value constraint used in weight selection module (set in `LolpRule.js`).
Weight selection module | weightObj.throughput, weightObj.latency, weightObj.buffer, weightObj.switch | The weight (value ranges between 0 and 1) of the current throughput, latency, rebuffering duration, and number of switches features of the SOM model (Bitrate selection module) will be assigned dynamically by this module based on the defined optimization function.

### Detailed code description

The ABR algorithm used in LoL+ bitrate selection module is explained below in pseudocode and code snippets:

<img src="http://reference.dashif.org/dash.js/markdown_images/lolp_pseudocode.png" height="600" />

At each segment download, the bitrate selection module is triggered and works as follows:

##### (1) Obtain input parameters (pseudocode lines 4-10):

* Current player state: throughput, latency, rebuffer duration, bitrate variation (normalized)
* Weight vector from weight selection module: w

##### (2) Update previously selected neuron with current player state values (pseudocode line 11):

```javascript
// Code snippet from `LearningAbrController.js`
_updateNeurons(currentNeuron, somElements, [throughputNormalized, latency, rebuffer, bitrateSwitch]);
```

##### (3) Iterate all neurons to find the winner neuron that is closest (i.e., shortest distance) to the target state, while not violating special condition (pseudocode lines 12-27):

```javascript
// Code snippet from `LearningAbrController.js`
// special condition downshift immediately
if (somNeuron.bitrate > throughput - throughputDelta || isBufferLow) {
    if (somNeuron.bitrate !== minBitrate) {
        // encourage to pick smaller bitrates throughputWeight=100
        distanceWeights[0] = 100;
    }
}

// calculate the distance with the target
let distance = _getDistance(somData, [throughputNormalized, targetLatency, targetRebufferLevel, targetSwitch], distanceWeights);
if (minDistance === null || distance < minDistance) {
    minDistance = distance;
    minIndex = somNeuron.qualityIndex;
    winnerNeuron = somNeuron;
    winnerWeights = distanceWeights;
}
```

##### (4) Update the winner neuron with target state values. (pseudocode line 28):

```javascript
// Code snippet from `LearningAbrController.js`
_updateNeurons(winnerNeuron, somElements, [throughputNormalized, targetLatency, targetRebufferLevel, bitrateSwitch]);
```

Note: The QoE evaluation module is provided but QoE score is not used as a SOM factor in the current implementation of
the ABR algorithm. Advanced users may consider using it.

## L2A

In the context of adaptive streaming, an ABR algorithm aims at seamlessly adjusting (or adapting) the rate of the media
stream, to compensate for changing network conditions. Additionally, a buffer is typically deployed to protect the
client from abrupt changes in the communication channel (throughput, jitter etc.), or temporal misestimations of the ABR
algorithm. Since long buffer queues compound delay of the media rendering process, low-latency streaming requires very
short buffers, that in turn offer less protection against channel state estimation errors. Such errors are propagated to
the ABR decisions, that in turn can have a detrimental effect on streaming experience.

Therefore the goal of  **Learn2Adapt-LowLatency (L2A-LL)**, a low-latency ABR, is to strike a favorable balance between
keeping the buffer as short as possible, while provisioning against its complete depletion. This is achieved by
selecting the highest sustainable bitrate for each video fragment, that does not completely consume the buffer budget
available at the time of request. L2A-LL is, in essence, an optimization solution with the objective of minimizing
latency, while at the same time maximizing achievable video bitrate and ensuring uninterrupted and stable streaming.

L2A-LL formulates the ABR optimization problem under an online (machine) learning framework, based on convex
optimization. First, the streaming client is modelled by a learning agent, whose objective is to minimize the average
buffer displacement of a streaming session. Second, certain requirements regarding the decision set (available bitrates)
and constraint functions are fulfilled by a) allowing the learning agent to make decisions on the video bitrate of each
fragment, according to a probability distribution and by b) deriving an appropriate constraint function associated with
the upper bound of the buffer queue, that adheres to time averaging constraints.

### Basic dash.js configuration

How to enable L2A:

```javascript
player.updateSettings({
    streaming: {
        abr: {
            useDefaultABRRules: true,
            ABRStrategy: 'abrL2A'    // to enable L2A bitrate selection module
        }
    }
});

```

### L2A example

An example for low latency streaming with L2A can be
found [here](http://reference.dashif.org/dash.js/nightly/samples/low-latency/l2all_index.html)

### Advanced Tuning parameters

The following changes are for experienced users and need to be made in `streaming/rules/abr/L2ARule.js`. Please check
the paper [5] for further details.

Line | Parameter | Description | Conclusion
--- | --- | --- | ---
306 | const horizon=4 | **Optimization
horizon**: This parameter is used to specify the amount of steps required to achieve convergence. In live streaming settings this parameter must be kept low for stable performance. The selected value has been verified experimentally and alteration is not suggested. This parameter is used in lines 307 and 308, at the calculation of the 'vl' and 'alpha' parameters respectively. The calculation of 'vl' and 'alpha' are according to the theoretical performance guarantees as specified in the MMSys Publication [5]. Higher 'vl' values make the algorithm more aggressive in the bitrate selection and 'alpha' is the step size of the gradient descent approach of the learning process. | higher 'horizon' leads to more aggressive bitrate selection (higher 'vl') and less exploration (large 'alpha'). Not advisable for live streaming scenarios with short buffers.
322 | const react = 2 | **Reactiveness to
volatility** (abrupt throughput drops). This parameter is used to recalibrate the 'l2AParameter.Q'. Higher values make the algorithm more conservative. The chosen value has been experimentally selected and alteration is not suggested. |  Higher 'react' results in a more conservative algorithm (higher l2AParameter.Q). Caution: 'react' values higher than the selected (react=2) may make the algorithm select lowest bitrate for extended periods, until recovery of the l2AParameter.Q (updated at every fragment download)

# Test streams

Description | Segment length | Chunk length | Url
--- | --- | --- | ---
Akamai Low Latency Stream (Single Rate) | 2.002s | 0.033s | https://akamaibroadcasteruseast.akamaized.net/cmaf/live/657078/akasource/out.mpd
Akamai Low Latency Stream (Multi Rate) | 2.002s | 0.033s | https://cmafref.akamaized.net/cmaf/live-ull/2006350/akambr/out.mpd
Low Latency (Single-Rate) (livesim-chunked) | 8s | 1s | https://livesim.dashif.org/livesim/chunkdur_1/ato_7/testpic4_8s/Manifest300.mpd
Low Latency (Multi-Rate) (livesim-chunked) | 8s | 1s | https://livesim.dashif.org/livesim/chunkdur_1/ato_7/testpic4_8s/Manifest.mpd

# Test results

[Spreadsheet](https://docs.google.com/spreadsheets/d/1dRt2hhX0RjSDlW2X2hRb9YEnR_PU_08sV-ESF-YSMbo/edit#gid=220322418)

# Material

## Articles

* [Daniel Silhavy - dash.js – Low Latency Streaming with CMAF](https://websites.fraunhofer.de/video-dev/dash-js-low-latency-streaming-with-cmaf/)
* [Will Law - Using LL-HLS with byte-range addressing to achieve interoperability in low latency streaming](https://blogs.akamai.com/2020/11/using-ll-hls-with-byte-range-addressing-to-achieve-interoperability-in-low-latency-streaming.html)
* [Theo Karagkioules,R. Mekuria,Dirk  Griffioen, Arjen  Wagenaar - Online learning for low-latency adaptive streaming](https://dl.acm.org/doi/pdf/10.1145/3339825.3397042)
* [May Lim, Mehmet N Akcay, Abdelhak  Bentaleb, Ali C. Begen, R. Zimmermann - When they go high, we go low: low-latency live streaming in dash.js with LoL](https://dl.acm.org/doi/abs/10.1145/3339825.3397043)

## Videos

* [Will Law - Chunky Monkey](https://www.youtube.com/watch?v=BYRjZNUgzFc&list=PLkyaYNWEKcOfARqEht42i1P4kBemzEV2V&index=11)
* [Theo Karagkioules,R. Mekuria,Dirk  Griffioen, Arjen  Wagenaar - Online learning for low-latency adaptive streaming](https://www.youtube.com/watch?v=NV7a8k2AfYg)
* [May Lim, Mehmet N Akcay, Abdelhak  Bentaleb, Ali C. Begen, R. Zimmermann - When they go high, we go low: low-latency live streaming in dash.js with LoL](https://www.youtube.com/watch?v=9xU582WomTg)

# Bibliography

* [1] [DASH-IF - Report on Low-Latency Live Service with DASH](https://docs.google.com/document/d/1WW4b586znnn8gQ9wvd1uj_7iKzAW-Wf5QargrCtVGnQ)
* [2] [Daniel Silhavy - dash.js – Low Latency Streaming with CMAF ](https://websites.fraunhofer.de/video-dev/dash-js-low-latency-streaming-with-cmaf/)
* [3] Abdelhak Bentaleb, Mehmet N. Akcay, May Lim, Ali C. Begen, R. Zimmermann - Catching the Moment with LoL + in
  Twitch-Like Low-Latency Live Streaming Platforms (to appear in IEEE Trans. Multimedia
    - [pdf](http://dx.doi.org/10.1109/TMM.2021.3079288))
* [4] [DASH-IF DASH Live Services](https://dashif.org/docs/CR-Low-Latency-Live-r8.pdf)
* [5] [Theo Karagkioules,R. Mekuria,Dirk  Griffioen, Arjen  Wagenaar - Online learning for low-latency adaptive streaming](https://dl.acm.org/doi/pdf/10.1145/3339825.3397042)

