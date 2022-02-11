# Switchboard Communication iOS SDK

```
More internal functionality of Audio and Communication can be exposed depending on customer requirements.
```


## What's Inside

Inside the package you'll find two folders
 - Frameworks
 - Sample App

In the Frameworks folder you'll find the Switchboard framework files that needed to be added in your project.
 - `SwitchboardCommunication.framework`
 - `SwitchboardAudio.framework`

In the Sample App folder, you'll find a SampleApp project showing how to use the frameworks.

## Running the SwitchboardCommunicationSampleApp

1. Download the Agora SDK for iOS from https://download.agora.io/sdk/release/Agora_Native_SDK_for_iOS_v3_3_0_FULL.zip
2. Unzip it's contents and move the `xcframework`s into the `Frameworks` folder
4. Open up `SwitchboardCommunicationSampleApp.xcodeproj`
5. Under Frameworks, Libraries, and Embedded Content, add the Agora frameworks
6. Select the `SwitchboardCommunicationSampleApp` and your device or a simultaor
7. Press Play

## Adding Switchboard SDK to Your Project

1. Download the Switchboard iOS SDK package
2. Add `SwitchboardCommunication.framework` and `SwitchboardAudio.framework` under Embedded Binaries in your project's settings.
3. Import the SDK wherever you want to use it. (import SwitchboardSDK)


## Initializing Switchboard SDK

Before you can start using our SDK you need to initialize it. We will provide you an App ID that you need to pass to the configure method. You can call configure in your AppDelegate’s `application(_:didFinishLaunchingWithOptions:)` method, or you can call it later before you start using the SDK.

```
let config = SwitchboardSDK.Config()
config.appID = "YOUR_APP_ID"
SwitchboardSDK.configure(config)
```

## Room API

### Joining Voice Communication Rooms

Users can communicate with each other in rooms. You can create a room by calling the createRoom method. You need to provide a room identifier that will be used to uniquely reference a room. If you join the same room with multiple users they will be able to communicate.

```
let room = SwitchboardSDK.createRoom(roomID: "Room Identifier")
room.delegate = self
room.join()

```
Errors and state changes are reported via delegate calls.

```
func room(_ room: Room didFailWithError error: Error)
func room(room: Room, didUpdateConnectionState state: RoomConnectionsState)
```


### Publishing and Subscribing to Rooms

To make users hear each other, they need to publish and subscribe to a room. When a user is publishing all the other users who are subscribing in the same room will hear that user.


| Method  | Description  |
|:----------|:----------|
| `room.subscribe()`    | Subscribes to a room. Users will start hearing other publishing members.    |
| `room.publish() `  | Starts publishing to a room. Subscribing members will start hearing the user.    |
| `room.unsubscribe()`   | Unsubscribes from a room. User will stop hearing other publishing users.    |
| `room.unpublish()`   | Unpublishes from a room. Subscribing users will stop hearing the user.    |
| `room.setSubscriberVideoEnabled(isEnabled: Bool)` | If true, starts subscribing video from a room. If false, unsubscribes video from a room. User will see, if true, or not, if false, videos that other users are publishing |
| `room.setPublisherVideoEnabled(isEnabeld: Bool)` | If true, starts publishing video to a room. If false, stops publishing video to a room. Other users will see, if true, or not, if false, the video captured by your device's camera. |
| `room.setCameraPosition(_ cameraPosition: AVCaptureDevice.Position)`| Sets if the camera used is front or back camera. |
| `room.muteUser(_ userID: String)` | Mutes a user. |
| `room.unmuteUser(_ userID: String)` | Unmutes a user. |


You can use the isSubscribing and isPublishing flags to get the current subscribe and publish states of a room. The following delegate call will be triggered whenever these values change.


```
func room(room: Room, didUpdateConnectionState state: RoomConnectionsState)
```

The Switchboard SDK always tries to maintain the subscribing and publishing states and handle all network errors automatically. You can get notified about the reconnection events by listening to the following delegate calls.

```
func didStartReconnectingToRoom(_ room: Room)
func didReconnectToRoom(_ room: Room)
```

Whenever a user, the current or any other, publishes or unpublishes video, these delegate calls will be triggered.

| Method  | Description  |
|:----------|:----------|
| `func room(room: Room, didUpdatePublisherVideoView view: UIView?)` | Triggered when the current user publishes or unpublishes video. |
| `func room(room: Room, didUpdateSubscriberVideoViews views: [SubscriberVideoView])` | Triggered when a user in the room publishes or unpublishes video. |


### Leaving Rooms


If you want to disconnect from a room, you can call the leave method. Leaving a room will automatically unsubscribe and unpublish.

```
room.leave()
```

The following delegate call will be triggered when the room is left.

```
func didLeaveRoom(_ room: Room)
```

### Room Limits

We currently have some usage limits in our SDK. We know this is unfortunate and we're constantly working to improve this.

The limits are:

 - for less than or equal to 5 publishers, up to 400 subscribers
 - for less than or equal to 10 publishers, up to 10 subscribers

### Broadcast Rooms with Amazon IVS SDK

The sample app in the SampleApp-IVS folder uses the Amazon IVS Player. The IVS SDK has to be downloaded separately from [https://docs.aws.amazon.com/ivs/latest/userguide/SWPG.html](https://docs.aws.amazon.com/ivs/latest/userguide/SWPG.html). To create an room that uses Amazon IVS call the `SwitchboardSDK.createBroadcastRoom()` method. OBS can be used to start a stream to this room and you should be able to watch the stream using our sample app.

## Activating the Bose PinPoint Noise Filter

Bose PinPoint reduces unwanted background noise by using AI built on deep neural networks, trained by thousands of hours of audio.

1) Contact us to get your PinPoint apiKey and apiSecret values
2) Make sure the PinPoint.xcframework file is added to your Xcode project
3) When you call SwitchboardSDK.configure pass in your `pinPointConfig` object

## Mixing Music with Voice Audio (Ducking)

There are two ways to set up music ducking with the Switchboard SDK. You can either send the music audio data to our SDK and allow us to render the mixed audio, or you can do the ducking yourself based on the volume values we send out in our DuckingDelegate class.

### Feeding Audio into Switchboard SDK

You can feed your audio data into the Switchboard SDK by using an AudioBus. First, you need to subclass the AudioBus class and implement the readRenderData method. Then you can call setAudioIn to let us know where to read your audio data from.

```
class MyAudioBus: AudioBus {
	func sampleRate() -> Int {
		return 44100
	}
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

let audioBus = MyAudioBus()
SwitchboardSDK.setAudioIn(audioBus)
```

The audio samples you send to our SDK will be rendered by us and we will make sure your audio is ducked based on our audio.


### Using DuckingDelegate

If you prefer not sending your audio data to our SDK you can implement ducking using the ducking delegate protocol. This protocol contains a method that is called with the volume value when ducking should happen on the music audio. The value passed in this function is always between 0 and 1.

```
func duckWithAmount(value: Float)

SwitchboardSDK.duckingDelegate = self
```

## Synchronizing Music with NTP

The SwitchboardSDK has built-in capabailities for synchronizing music using the Network Time Protocol. You can use the following two methods to build a co-listening experience in your app. The `play` method is typically called by the DJ user, while `syncPlay` is called by other listeners. The `absoluteStartTime` parameter has to be transferred by the integrating application.

```
- (void)play:(NSURL *)fileURL offset:(NSTimeInterval)offset completion:(void (^)(BOOL success, NSDate *absoluteStartTime))completion
- (void)syncPlay:(NSURL *)fileURL absoluteStartTime:(NSDate *)absoluteStartTime completion:(void (^)(BOOL success))completion
```
