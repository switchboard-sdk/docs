# Switchboard Audio SDK

```
More internal functionality of Audio and Communication can be exposed depending on customer requirements.
```

## Adding SwitchboardAudio SDK to Your Project

1. Download the Switchboard iOS SDK package
2. Add `SwitchboardAudio.framework` under Embedded Binaries in your project's settings.
3. Import the SDK wherever you want to use it. (import SwitchboardAudio)

## Starting and Stopping the Audio System

`TMUAudioSystem` implements the singleton pattern, you can get the shared object by calling `TMUAudioSystem.sharedInstance()`.

```
let audioSystem = TMUAudioSystem.sharedInstance()
audioSystem.start()
...
audioSystem.stop()
```

## Integration with Communication SDK

The SwitchboardAudio SDK exposes the `TMURemotePeerAudioBus` protocol that can be used to integrate with a VoIP provider. `TMUAudioSystem.sharedInstance()` implements a simple audio bus that can send the audio output for rendering and receive the audio input for capturing.

The SwitchboardCommunication SDK has built-in integration with Agora, Tokbox and Vivox so if you use that, no steps required to connect SwitchboardAudio with SwitchboardCommunication.

The steps to send and receive audio to/from WebRTC are explained in the `WebRTC.md` documentation file.

## Configuring the Microphone Preprocessor

If you would like to change the microphone preprocessor settings you can use the following properties.

| Property  | Description  |
|:----------|:----------|
| microphoneLowpassFilterEnabled    | Enables/Disables the low-pass filter    |
| microphoneLowpassFilterFrequency    | Gets/sets the low-pass filter frequency (Hz)    |
| microphoneHighpassFilterEnabled    | Enables/Disables the high-pass filter    |
| microphoneHighpassFilterFrequency    | Gets/sets the high-pass filter frequency (Hz)    |
| microphoneCompressorThresholdDb    | Sets the compressor threshold (Db)    |


```
let audioSystem = TMUAudioSystem.sharedInstance()
audioSystem.microphoneLowpassFilterEnabled = true
audioSystem.microphoneLowpassFilterFrequency = 8000
```

## Audio Session Handling

The SwitchboardAudio SDK automatically manages the Audio Session and it creates the necessary audio units for the best audio experience. You should make sure that you don't call AVAudioSession while Switchboard audio system is running.

## InConversation and VoiceProcessing Flags

The Switchboard Audio SDK handles audio differently whenever the user is in an active VoIP call. The Switchboard Communication SDK automtically lets the Audio SDK know the call is started and ended, but if a custom integration is used (e.g. WebRTC) some flags have to be set manually. The `isInConversation` flag should be set to true when the user is publishing audio to a communication channel. The `voiceProcessingEnabled` flag controls whether the voice processing audio unit  (platform AEC) can be used as audio input and output.

```
let audioSystem = TMUAudioSystem.sharedInstance()
audioSystem.isInConversation = true
audioSystem.voiceProcessingEnabled = true
```

## Music Playback

The preferred way to play music with the Switchboard Audio SDK is to feed the audio data into the SDK using an audio bus. First, you need to subclass the `TMURemoteMusicBus` class and implement the `readRenderData` method. Then you can call `add(musicBus)` to let us know where to read your audio data from. The audio samples you send to our SDK will be rendered by the SDK.

```
class MyAudioBus: TMURemoteMusicBus {
	func isRendering() -> Bool {
		return true
	}
	func readRenderData(
		data: UnsafeMutableRawPointer!,
		numberOfSamples: UInt32) -> UInt32 {
		// Fill data with audio samples
		// Return number of audio samples written
	}
}

let musicBus = MyAudioBus()
let audioSystem = TMUAudioSystem.sharedInstance()
audioSystem.add(audioBus)
```


## Logging

You can receive log messages from the SwitchboardAudio SDK by setting the `TMULogger.logDestination` property. The object that you use for this value has to implement the `TMULoggerDestination` protocol.

```
TMULogger.logDestination = MyAudioLogDestination()
```
