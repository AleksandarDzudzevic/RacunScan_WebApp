# RacunScan_WebApp

## 1) IOS SWIFT PART
### 1.1 Firebase Google Sign-up
Firebase Google signup is a better option, might be necessary to start from the new device but it can be easily done following this tutorial:[firebase authentication](https://firebase.google.com/docs/auth)
```.swift
import UIKit
import GoogleSignIn
import CoreData
import Firebase
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    let signInConfig = GIDConfiguration.init(clientID: K.clientID)

    func application(
      _ application: UIApplication,
      didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
      GIDSignIn.sharedInstance.restorePreviousSignIn { user, error in
        if error != nil || user == nil {
        } else {
            print("Previous sign in restored!")
        }
      }
    FirebaseApp.configure()
    return true
    }

    func application(_ application: UIApplication,
                     open url: URL, sourceApplication: String?, annotation: Any) -> Bool {
        var handled: Bool

        handled = GIDSignIn.sharedInstance.handle(url)
        if handled {
          return true
        }
        
        return false
    }

    @available(iOS 9.0, *)
    func application(_ app: UIApplication, open url: URL,
                     options: [UIApplication.OpenURLOptionsKey : Any]) -> Bool {
        var handled: Bool
        handled = GIDSignIn.sharedInstance.handle(url)
        
        if handled {
          return true
        }
        // Handle other custom URL types.
        // If not handled by this app, return false.
        return false
    }

    // MARK: UISceneSession Lifecycle

    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        // Called when a new scene session is being created.
        // Use this method to select a configuration to create the new scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
        // Called when the user discards a scene session.
        // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
        // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
    }


}
```
### 1.2 RacunScanner Swift App
Content view code (Firebase not integrated but can easily be done)
```.swift
//
//  ContentView.swift
//  RacunScanner
//
//  Created by Aleksandar Dzudzevic on 5.6.23..
//

import SwiftUI
import CodeScanner
import SafariServices
import UIKit
import GoogleSignIn
import GoogleSignInSwift

struct ContentView: View {
    @State private var isPresentingScanner = false
    @State private var scannedCode: String = ""
    @State private var tipKartice: String = ""
    @State private var showFullCode = false
    @State private var isSignedIn = false
    
    var body: some View {
        VStack(spacing: 40) {
            if isSignedIn {
                Text("Izaberite opciju i skenirajte račun")
                    .font(.title)
                    .underline()
                if !scannedCode.isEmpty {
                    Text("Račun je uspešno skeniran")
                        .font(.title)
                        .bold()
                    Text(tipKartice)
                    
                    if showFullCode {
                        Text(scannedCode)
                            .font(.footnote)
                            .lineLimit(nil)
                        Button("Idi na Link") {
                            openScannedCodeURL()
                        }
                        .foregroundColor(.blue)
                        
                        Button("Go Back") {
                            scannedCode = ""
                            showFullCode = false
                        }
                    } else {
                        Button("Vidi Link") {
                            showFullCode = true
                        }
                    }
                } else {
                    Text("Izaberite opciju i skenirajte račun")
                        .font(.title)
                        .underline()
                    
                    Button("FIRMINA KARTICA Skeniraj QR Kod") {
                        self.isPresentingScanner = true
                        self.scannedCode = ""
                        self.tipKartice = "Kupovina preko firmine kartice"
                    }
                    .sheet(isPresented: $isPresentingScanner) {
                        self.scannerSheet
                    }
                    
                    Button("PRIVATNA KARTICA Skeniraj QR Kod") {
                        self.isPresentingScanner = true
                        self.scannedCode = ""
                        self.tipKartice = "Kupovina preko privatne kartice"
                    }
                    .sheet(isPresented: $isPresentingScanner) {
                        self.scannerSheet
                    }
                }
            } else {
                VStack(spacing: 20) {
                    // Add the Google Sign-In button
                    GoogleSignInButton(action: handleSignInButton)
                    
                    
                    // Add the other buttons for scanning options here
                }
            }
        }
        .onAppear {
            GIDSignIn.sharedInstance.restorePreviousSignIn { user, error in
                if user != nil {
                    isSignedIn = true
                }
            }
        }
    }
    
    // Handle the Google Sign-In button action
    func handleSignInButton() {
        if let presentingViewController = UIApplication.shared.windows.first?.rootViewController {
            GIDSignIn.sharedInstance.signIn(withPresenting: presentingViewController) { signInResult, error in
                if let error = error {
                    // Handle sign-in error
                } else {
                    // User signed in successfully
                    isSignedIn = true
                }
            }
        } else {
            // Handle error case where presentingViewController is not available
        }
    }
    
    var scannerSheet: some View {
        CodeScannerView(
            codeTypes: [.qr],
            completion: { result in
                if case let .success(code) = result {
                    self.scannedCode = code.string
                    self.isPresentingScanner = false
                }
            }
        )
    }
    
    private func openScannedCodeURL() {
        if let url = URL(string: scannedCode) {
            let safariViewController = SFSafariViewController(url: url)
            UIApplication.shared.windows.first?.rootViewController?.present(safariViewController, animated: true, completion: nil)
        }
    }
    
    struct ContentView_Previews: PreviewProvider {
        static var previews: some View {
            ContentView()
        }
    }
    
}
```
App code 
```.swift
import SwiftUI
import GoogleSignIn

@main
struct RacunScannerApp: App {
    // ...

    var body: some Scene {
        WindowGroup {
            ContentView()
        
                .onOpenURL { url in
                    // Handle the authentication redirect URL here
                    if GIDSignIn.sharedInstance.handle(url) {
                        return
                    }
                    
                    // Handle other custom URL types
                }
        }
    }
    func application(
      _ app: UIApplication,
      open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]
    ) -> Bool {
      var handled: Bool

      handled = GIDSignIn.sharedInstance.handle(url)
      if handled {
        return true
      }

      // Handle other custom URL types.

      // If not handled by this app, return false.
      return false
    }
    // ...
}
```
### 1.3 Vapor API (only started)

For the Vapor API connection was successfully set up and GET & POST worked well when it comes to server interactions.
For the whole configuration and setting up process I followed this [Vapor tutorial for Swift](https://youtu.be/2tACpHQeHfI)
To start with Vapor-> [Vapor website](https://vapor.codes/)
```.swift
 
let app = try Application(.detect())
defer { app.shutdown() }

app.get("hello") { req in
    return "Hello, world."
}

try app.run()
```
## 2) ASP.NET PART (NEW VSC version)
### 2.2 Google Sign-up for ASP
Google sign-up successfully configured on google cloud [Secret client and secret ID stored in this json](https://github.com/AleksandarDzudzevic/RacunScan_WebApp/blob/main/Asp.net_WebApp/client_secret_677994717306-mclgtmlgc1qfbu0o4po4qvjl5h7f7c5s.apps.googleusercontent.com%20(1).json)
The same Google console project was used for the Firebase IOS Google sign-in because the Firseship change was decided recently it is not entirely done but possible account manipulation between platforms is possible through google console.

All source code for this as well as for the web app is linked to an [open drive](https://drive.google.com/drive/folders/1sHeQUceworzDEgDDury9AKn-GZy-xYtA?usp=sharing).
### 2.2 Web App part
The code for everything, including the WEB_APP is on this [Gdrive](https://drive.google.com/drive/folders/1sHeQUceworzDEgDDury9AKn-GZy-xYtA?usp=sharing)
## 3) Future steps
1) Set up Vapor_API and Firebase part of the Swift code for Google signing on IOS (ideally link it to same google cloud project as web app.)
2) Through Vapor API in the json post :(signed in user, link of the receipt, date from datetime library, type of the card used, which is stored in the app as a variable)
3) Add GET request in the web app and store in the racuni.sql data received. (just connect the table tp index.html in the views folder so that receipts are displayed once user gets registered)
4) That should be all but of course connecting all the parts might ask for something extra, if I can help somehow lmk.
