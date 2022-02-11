# Switchboard SDK

```
More internal functionality of Audio and Communication can be exposed depending on customer requirements.
```

## Adding Switchboard SDK to Your Project
 
1. Create a folder `libs` in your module
2. Copy the following files from the `SwitchboardSDK` folder into the `libs` folder
 - `SwitchboardSDK.aar`
 - `PinPointSDK.aar`
3. Add the following to your module's build.gradle dependencies
  - `implementation(files("libs/SwitchboardSDK.aar"))`
  - `implementation(files("libs/PinPointSDK.aar"))`
4. Add the following in your `AndroidManifest.xml`
 - ```<uses-sdk tools:overrideLibrary="com.synervoz.switchboard.audio"```
 
 The SDK requires minimum API level 23 or above
 

 ## Initializing Switchboard SDK

 Before you can start using `SwitchboardSDK` you need to create an `SwitchboardSDKConfig` object and send it as a parameter to the `initialize` method. You can call `initialize` in your Application or Activity's `onCreate()` method, or you can call it later before you start using the SDK. `SwitchboardSDK` implements the singleton pattern, you can get the shared object by calling `SwitchboardSDK.getInstance()`.

 If you want to have intelligent noise removal applied to the microphone's input you will need to send `PinPointSDK` to `SwitchboardSDKConfig`. 

 ```
 val pinPointConfig = PinPointConfig(
    "Api Key",
    "Api Secret",
    true
)

val audioConfiguration = AudioConfiguration(
    sampleRate = SampleRate.HZ_44100,
    bufferSize = BufferSize.SAMPLES_512
)

val switchboardSDKConfig = SwitchboardSDKConfig(
    context,
    audioConfiguration,
    "Superpowered Key",
    pinPointConfig
)
SwitchboardSDK.initialize(switchboardSDKConfig)
 ```
 
 
 ## Starting and Stopping the SwitchboardSDK's Audio Processing
 
 `start` method needs to be called before `SwitchboardSDK` can start audio input/output processing. Similarly `stop` method stops the audio intput/output processing.
 
 ```
 SwitchboardSDK.getInstance().start()
```
 
 `onForeground` method lets the `SwitchboardSDK` know audio should stay active, if an application using the SwitchboardSDK goes in the background, `onBackground` should be called so that if there is no audio processing needed battery can be saved. We suggest calling `onForeground` and  `onBackground` when the application comes to foreground and goes in the background respectively.

 ```
 override fun onFragmentResume() {
     SwitchboardSDK.getInstance().onForeground()
 }

 override fun onFragmentPause() {
     SwitchboardSDK.getInstance().onBackground()
 }
```


## Send audio to and getting audio out of SwitchboardSDK

You can feed audio data into the `SwitchboardSDK` and get audio out by implementing `RemotePeerAudioBus`. First, you need to subclass the `RemotePeerAudioBus` class. Then to send the audio to the `SwitchboardSDK` you implement the `readRenderData` method. To get the audio from the `SwitchboardSDK` you implement the `writeCaptureData` method. Then you can call `addRemoteMusicBus` to let us know where to read your audio data from.

```
class MyRemotePeerAudioBus : RemotePeerAudioBus {
    
    override fun isRendering() = true
    
    override fun readRenderData(
        data: ByteArray?,
        numberOfSamples: Long) -> Long {
        // Fill data with audio samples
        // Return number of audio samples written
    }
    
    override fun isCapturing() = true
    
    override fun writeCaptureData(
        data: ByteArray?,
        numberOfSamples: Long) {
        // data contains audio samples from the SwitchboardSDK
        // Return number of audio samples written
    }
}

val myAudioBus = MyRemotePeerAudioBus()
SwitchboardSDK.getInstance().addRemotePeerAudioBus(audioBus)
```


## Mixing Music with Voice Audio (Ducking)

There are two ways to set up music ducking with the Switchboard SDK.
You can either send the music audio data to our SDK and allow us to render the mixed audio, or you can do the ducking yourself based on the volume values we send out in our `MusicDuckingInterface` interface.


### Feeding Music into SwitchboardSDK

You can feed your music data into the `SwitchboardSDK` by using an `MusicAudioBus`. First, you need to subclass the `MusicAudioBus` class and implement the `readRenderData` method. Then you can call `addRemoteMusicBus` to let us know where to read your music data from.

```
class MyMusicAudioBus : MusicAudioBus {
	
	override fun isRendering() = true
    
    	override fun readRenderData(
		data: ByteArray?,
		numberOfSamples: Long) -> Long {
		// Fill data with audio samples
		// Return number of audio samples written
	}
}

val myMusicAudioBus = MyMusicAudioBus()
SwitchboardSDK.getInstance().addRemoteMusicBus(audioBusMusicPlayer)
```

The audio samples you send to our SDK will be rendered by us and we will make sure your music is ducked.


### Handling Ducking

If you prefer not sending your music audio data to our SDK you can implement ducking using the `MusicDuckingInterface` interface. This interface contains a method `duckMusicWithAmount` that is called with the volume value when ducking should happen on the music audio. The value passed in this function is always between 0 and 1. The volume value depends only on the voice audio data, so sending in your music audio data is not needed.

```
SwitchboardSDK.getInstance().duckingInterface = object : MusicDuckingInterface {
    override fun duckMusicWithAmount(value: Float) {
        Log.d(TAG, "Ducking value: $value")
    }
}
```

### Configuring Ducking Properties

You can change the ducking target amount when the local user speaks with `localDuckingAmount`

```
SwitchboardSDK.getInstance().duckingSettings.localDuckingAmount = 0.15F
```

You can change the ducking target amount when a remote user speaks with `remoteDuckingAmount`

```
SwitchboardSDK.getInstance().duckingSettings.remoteDuckingAmount = 0.05F
```

You can change the number of seconds ducking should be applied for with `numSecondsToHoldDucking`

```
SwitchboardSDK.getInstance().duckingSettings.numSecondsToHoldDucking = 2.0F
```

### Configuring Volumes

```
SwitchboardSDK.getInstance().remoteMicVolume = 1F
```
```
SwitchboardSDK.getInstance().musicVolume = 1F
```

## Configuring Bluetooth Profiles
You can configure the bluetooth profile to be used during the communication. To enable A2DP profile you can do the following. The A2DP profile uses microphone of the smartphone instead of the bluetooth headset and uses  bluetooth headset for audio output. Disabling A2DP uses HFP profile which uses connected bluetooth headset for both microphone input and audio output.

```
SwitchboardSDK.getInstance().allowBluetoothA2dp = true
```


## Logging

You can enable adb log messages from the `SwitchboardSDK` by setting the `enableLogging` to `true` in `SwitchboardSDKConfig` when initializing.
