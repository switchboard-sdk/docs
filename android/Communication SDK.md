# Switchboard Communication SDK

```
More internal functionality of Audio and Communication can be exposed depending on customer requirements.
```

The `Switchboard Communication SDK` needs `Switchboard SDK` to work. This document contains the guide on how to add and initialize both `Switchboard SDK` and `Switchboard Communication SDK`. Details of `Switchboard SDK` can be found in its own Documentation folder.

The folder `SampleApp` contains the Switchboard Communication Sample App.

## Adding Switchboard Communication SDK to Your Project

1. Create a folder `libs` in your module
2. Copy the following files from the `SwitchboardSDK` folder into the `libs` folder
 - `SwitchboardSDK.aar`
 - `PinPointSDK.aar`
3. Copy the following files from the `SwitchboardCommunicationSDK` folder into the `libs` folder
 - `SwitchboardCommunicationSDK.aar`
4. Add the following to your module's build.gradle dependencies
 - `implementation(files("libs/SwitchboardSDK.aar"))`
 - `implementation(files("libs/PinPointSDK.aar"))`
 - `implementation(files("libs/SwitchboardCommunicationSDK.aar"))`
 - `implementation 'io.agora.rtc:full-sdk:3.5.2'`
 - `implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.2'`
 - `implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.2'`
 - `implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.1'`
5. Add the following in your `AndroidManifest.xml`
  - ```<uses-sdk tools:overrideLibrary="com.synervoz.switchboard.audio"/>```
6. Add the following permissions to your `AndroidManifest.xml`

- INTERNET
- RECORD_AUDIO
- MODIFY_AUDIO_SETTINGS
- BLUETOOTH
- BLUETOOTH_ADMIN
- CAMERA

 The SDK requires minimum API level 23 or above
 

## Initializing Switchboard SDK

Before you can start using  `SwitchboardSDK` you need to create an `SwitchboardSDKConfig` object and send it as a parameter to the `initialize` method. You can call `initialize` in your Application or Activity's `onCreate()` method, or you can call it later before you start using the SDK. `start` method needs to be called before `SwitchboardSDK` can start audio input/output processing.

If you want to have intelligent noise removal applied to the micrphone's input you will need to send `PinPointSDK` to `SwitchboardSDKConfig`. 

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
SwitchboardSDK.getInstance().start()
SwitchboardSDK.getInstance().onForeground()
```

`SwitchboardSDK` implements the singleton pattern, you can get the shared object by calling `SwitchboardSDK.getInstance()`.

`onForeground` method lets the `SwitchboardSDK` know audio should stay active, if an application using the SwitchboardSDK goes in the background, `onBackground` should be called so that if there is no audio processing needed battery can be saved. We suggest calling `onForeground` and  `onBackground` when the application comes to foreground and goes in the background respectively.

```
override fun onFragmentResume() {
    SwitchboardSDK.getInstance().onForeground()
}

override fun onFragmentPause() {
    SwitchboardSDK.getInstance().onBackground()
}
```

You can learn more about the `SwitchboardSDK` in the `Switchboard.md` file found in the `SwitchboardSDK` folder.


## Initializing Switchboard Communication SDK

Before you can start using `CommunicationSDK` you need to create an `CommunicationSDKConfig` object and send it as a parameter to the `initialize` method. You can call `initialize` in your Application or Activity's `onCreate()` method, or you can call it later before you start using the SDK.

```
val communicationSDKConfig = CommunicationSDKConfig(
    context,
    "Client ID",
    "Client Secret"
    "App ID",
    "User ID",
)
CommunicationSDK.initialize(communicationSDKConfig)
```
`CommunicationSDK` implements the singleton pattern, you can get the shared object by calling `CommunicationSDK.getInstance()`.


## Joining Voice Communication Rooms

Users can communicate with each other in rooms. You can create a `Room` by calling the `createRoom` method. You need to provide a room identifier that will be used to uniquely reference a room. If you join the same room with multiple users they will be able to communicate. You can join a room by calling `join` method.

```
val room: Room = CommunicationSDK.getInstance().createRoom(roomID)
room.join()
```

`RoomInterface` can be implemented to get updates from the `Room` object.

```
room.roomInterface = this
```

Errors and state changes are reported via roomInterface methods.

```
fun didFailToJoinWithError(error: Error) 
fun didUpdateConnectionState(state: RoomConnectionState)
```


## Publishing and Subscribing to Rooms

To make users hear each other, they need to publish and subscribe to a room. When a user is publishing all the other users who are subscribing in the same room will hear that user.


| Method  | Description  |
|:----------|:----------|
| `room.subscribe()`    | Subscribes to a room. Users will start hearing other publishing members.    |
| `room.publish() `  | Starts publishing to a room. Subscribing members will start hearing the user.    |
| `room.unsubscribe()`   | Unsubscribes from a room. User will stop hearing other publishing users.    |
| `room.unpublish()`   | Unpublishes from a room. Subscribing users will stop hearing the user.    |
| `room.setSubscriberVideoEnabled(isEnabled: Boolean)` | If true, starts subscribing video from a room. If false, unsubscribes video from a room. User will see, if true, or not, if false, videos that other users are publishing |
| `room.setPublisherVideoEnabled(isEnabeld: Boolean)` | If true, starts publishing video to a room. If false, stops publishing video to a room. Other users will see, if true, or not, if false, the video captured by your device's camera. |
| `AgoraEngine.shared.onSwitchCameraClicked()`| Switches the camera between front and back. |
| `room.muteUser(uid: String)` | Mutes a user. |
| `room.unmuteUser(uid: String)` | Unmutes a user. |


`Room` object contains a `RoomConnectionState` object that you can use to get the state of the `Room`.
You can use the `isSubscribing` and `isPublishing` `RoomConnectionState` on the flags to get the current subscribe and publish states of a room. The following `RoomInterface` method will be triggered whenever these values change.


```
fun didUpdateConnectionState(state: RoomConnectionState)
```

Whenever a user, the current or any other, publishes or unpublishes video, these interface methods will be triggered.

| Method  | Description  |
|:----------|:----------|
| `fun didUpdatePublisherVideoView(view: SurfaceView?)` | Triggered when the current user publishes or unpublishes video. |
| `fun didUpdateSubscriberVideoViews(views: ArrayList<SubscriberVideoView>)` | Triggered when a user in the room publishes or unpublishes video. |


## Leaving Rooms

If you want to disconnect from a room, you can call the leave method. Leaving a room will automatically unsubscribe and unpublish.

```
room.leave()
```


## Presence information of users in a Room

`RoomInterface` provides a way to check the presence of users in a `Room`

Following methods of `RoomInterface` are called by the `SwitchboardCommunicationSDK` to update the listener of presence states of users in a `Room`.

```
fun userDidJoin(userID: String)
fun userDidLeave(userID: String)
fun userDidMute(userID: String)
fun userDidUnmute(userID: String)
fun videoDidMute(userID: String)
fun videoDidUnmute(userID: String)
fun videoPublisherState(mute: Boolean)
```

### Video State
`VideoState` in `SubscriberVideoView` object provides current state of video stream of users in a `Room` whether it's `NORMAL` or `FROZEN`
`SubscriberVideoView` is sent in the `didUpdateSubscriberVideoViews` method of `RoomInterface`

### Video Stats
`VideoStats` in `SubscriberVideoView` object provides information and statistics about the video stream from each user in a `Room`.

```
override fun didUpdateSubscriberVideoViews(views: ArrayList<SubscriberVideoView>) {
   for (view in views) {
       when (view.state) {
           VideoState.NORMAL -> Log.d("VIDEO_INFO", "Video state for user ${view.userID} is normal")
           VideoState.FROZEN -> Log.d("VIDEO_INFO", "Video state for user ${view.userID} is frozen")
       }
       Log.d("VIDEO_INFO", "Video stats for user ${view.userID}= ${view.stats}")
   }
    
```


## Using `SwitchboardSDK` for audio/music features

Please refer to `Switchboard.md` to see how to use `SwitchboardSDK` can be used for following features:
- Play music
- Implement ducking
- Control bluetooth profiles
- Noise removal


## Logging

You can enable adb log messages from the SwitchboardSDK by setting the `enableLogging` to `true` in `CommunicationSDKConfig` when initializing.
```
val communicationSDKConfig = CommunicationSDKConfig(
    context,
    "App ID",
    "Client ID",
    "User ID",
    "Client Secret",
    true
)
CommunicationSDK.initialize(communicationSDKConfig)
```


## SDK Usage and Limits

We currently have some usage limits in our SDK. We know this is unfortunate and we're constantly working to improve this.

The limits are:

 - for less than or equal to 5 publishers, up to 400 subscribers
 - for less than or equal to 10 publishers, up to 10 subscribers
