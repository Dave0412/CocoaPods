## Advanced User Authentication

In order for the iOS SDK to control the Spotify app, they will need to authorize your app. The functionality to do this is built in and can be implemented directly inside of your `AppDelegate.swift`:

#### Implement Session Delegate

In order to handle auth, we need to add a `SPTSessionManagerDelegate` inside of your `AppDelegate.swift`:

```swift
class AppDelegate: UIResponder, UIApplicationDelegate, SPTSessionManagerDelegate {
  ...
```

This will require us to implement the following three methods:
```swift
func sessionManager(manager: SPTSessionManager, didInitiate session: SPTSession) {
  print("success", session)
}
func sessionManager(manager: SPTSessionManager, didFailWith error: Error) {
  print("fail", error)
}
func sessionManager(manager: SPTSessionManager, didRenew session: SPTSession) {
  print("renewed", session)
}
```

#### Instantiate `SPTConfiguration`

At a class-level, we can define our Client ID, Redirect URI and instantiate the SDK:

```swift
let SpotifyClientID = "[your spotify client id here]"
let SpotifyRedirectURL = URL(string: "spotify-ios-quick-start://spotify-login-callback")!

lazy var configuration = SPTConfiguration(
  clientID: SpotifyClientID,
  redirectURL: SpotifyRedirectURL
)
```

#### Setup Token Swap

The authentication process provides a `refresh_token`, which can be stored locally inside of your app. This can be used, along with your Client ID, Client Secret and Redirect URL, to obtain an `access_token` that is valid for 60 minutes.

However, as we strongly discourage the use of Client Secrets in your iOS app code, we have written two well-documented web server examples that can do this for you:
- [Glitch](https://glitch.com/~spotify-token-swap)
- [One-click deploy with Heroku](https://github.com/bih/spotify-token-swap-service#one-click-with-heroku)

Once you have set them up, and have the `tokenSwapURL` and `tokenRefreshURL` we can set this up in our `AppDelegate.swift` in a class-level closure:

```swift
lazy var sessionManager: SPTSessionManager = {
  if let tokenSwapURL = URL(string: "https://[my token swap app domain]/api/token"),
     let tokenRefreshURL = URL(string: "https://[my token swap app domain]/api/refresh_token") {
    self.configuration.tokenSwapURL = tokenSwapURL
    self.configuration.tokenRefreshURL = tokenRefreshURL
    self.configuration.playURI = ""
  }
  let manager = SPTSessionManager(configuration: self.configuration, delegate: self)
  return manager
}()
```

#### Configure Initial Music

iOS requires us to define a `playURI` (as shown in the last step) in order to play music to wake up the Spotify main application. This is an iOS-specific requirement. There's two values `self.configuration.playURI` accepts:

**An empty value:** If empty, it will resume playback of user's last track. Example:

```swift
self.configuration.playURI = ""
```

**A valid Spotify URI:** Otherwise, provide a Spotify URI. Example:

```swift
self.configuration.playURI = "spotify:track:20I6sIOMTCkB6w7ryavxtO"
```

#### Invoke Auth Modal

With `SPTConfiguration` and `SPTSessionManager` both configured, we can invoke the authorization screen:

```swift
let requestedScopes: SPTScope = [.appRemoteControl]
self.sessionManager.initiateSession(with: requestedScopes, options: .default)
```

#### Configure Auth Callback

Once a user successfully returns to your application, we'll need to notify `sessionManager` about it by implementing the following method:

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
  self.sessionManager.application(app, open: url, options: options)
  return true
}
```

Now, when a user authorizes they should return to your application with the `sessionManager(manager: SPTSessionManager, didInitiate session: SPTSession)` method being successfully invoked.