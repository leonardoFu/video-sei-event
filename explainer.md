# video SEI event Explainer

## Problem and Motivation

In these years, interactions are frequently used in live activities. Such as shop tickets and Livestreaming quiz and so on. The most convenient way to keep activities sync with livestream is to use SEI (Supplemental Enhancement Information) of H.264. 

When we use MSE to play livestream, it's easy to get access to H.264 NAL units by demux live video format. but when we use iOS Safari or WebView on iOS, we can only use `<video src="{HLS_URL}">` to play live stream. So there is no chance that we can get access to the raw content of live stream (for example, SEI).

We propose a new `sei` event to solve this problem.

## Key-use cases

### Render element synchronously with video frame in `<video>` elements
Sometimes during the live activity, we'd have some interaction with audiences, for example: subtitles, Face recognition based stickers, question forms or goods for sale. Those information and SEI can be both produced with a NTP timestamp, so that we can know when to render it at the Webapp side.

### Calculate end to end delay from broadcaster to client
End to end delay is important to measure the live experience, especially for outside live and e-commerce livestreaming, in which scenario broadcasters concern about the feedback from The audience. so end to end dalay is an key indicator to measure the CDN quality.

### Get speaker information when watching live stream from video conference
In video conference senario, there would be more than 1 speakers, when the conference is recorded as an video file, we want to replay it to get some information, and to know who is talking, and other user information of the speaker, we want use the SEI to get the information, and render it synchronously with the video track.

### Body or face information generated by media server
Some times the realtime body or face recongnization is not efficent for Web browsers, in particular for those old devices or mobile phones. So we need the server side to run the Algorithm and put the information into the SEI.


## Proposed Solution

Get SEI information from web video, with loose accurate timestamp information so we can use it to sync with `video.currentTime`.

Can be used with [Media Source Extensions??? (w3.org)](https://www.w3.org/TR/media-source-2/),  parse the livestream by a JavaScript demuxer and a remuxer, to get the fmp4 stream with SEI data. And SEI event would be triggered when Web Video parsing the H.264 NALs.

Can be used with both WebCodecs API and [Media Source Extensions??? (w3.org)](https://www.w3.org/TR/media-source-2/) , demux live stream and generate `EncodedVideoChunks`, and pass it to `SourceBuffer` directly.




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
