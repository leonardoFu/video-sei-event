# video SEI event Explainer

# Introduction

In these years, interactions are frequently used in live activities. Such as shop tickets and Livestreaming quiz and so on. The most convenient way to keep activities sync with livestream is to use SEI (Supplemental Enhancement Information) of H.264. 

When we use MSE to play livestream, it's easy to get access to H.264 NAL units by demux live video format. but when we use iOS Safari or WebView on iOS, we can only use `<video src="{HLS_URL}">` to play live stream. So there is no chance that we can get access to the raw content of live stream (for example, SEI).

We propose a new `sei` event to solve this problem.


# Use cases

activities that use SEI to keep user sync with live stream. websites can get SEI information when specific NAL unit getting parsed in `<video />`.

calculating end-to-end delay between the host and user.

AI based subtitles with livestream, we can get realtime subtitles and put it into H.264 by SEI

AI based body or face recognation, we can use it to render plugins beside it.


# Proposed API

```Javascript
interface SEIEvent {
  type: 'sei';
  timestamp: number;
  byteLength: number;
  copyTo: (dest: Uint8Array) => void 
};
```


# Example

```Javascript
  const video = document.createElement('video');
  video.src = 'foo.hls';
  video.play();
  
  video.addEventListener('sei', (e) => {
    const seiData = new Uint8Array(e.byteLength);
    e.copyTo(seiData);
    const timestamp = e.timestamp;
    
    const interactionData = parseSEI(seiData);
    
    const currentTime = video.currentTime;
    // use timestamp to sync SEI with livestream
    setTimeout(() => {
      renderInteraction(interactionData);
    }, (timestamp - currentTime) * 1000)
  })
```

# Limitation
## Not available when using with EME
If you are using EME for encrypted media source playback, SEI data may not be accessible, because EME module only emits decoded video frame, not AVC samples

## Not suitable for high accuracy scenes
We get the sei timestamp when parsing the AVC bitstream, but when rendering, it's difficult to sync SEI with the exact frame. So if you want to render SEI information and concern about the synchronization between video frame and SEI, we suggest you to carry the SEI data only using Keyframe.


# Implementation Details

* SEI data includes the whole SEI Nal unit to ensure that user can parse SEI data with SEI Specification
