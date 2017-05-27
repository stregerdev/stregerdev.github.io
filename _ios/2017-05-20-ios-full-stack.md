---
layout:
title: "10 - Full stack iOS"
excerpt: "Firebase Authentication, Database, and Storage. Beer Hug Part 2"
tags:
    - ios
--- 
[Source Code](https://github.com/jdstregz/BeerHug)

Hello and welcome to the last tutorial in this tutorial series (for now). I'd like to thank everyone for reading. This series has been a part of an independent study that I created for the University of Delaware. This will be the last tutorial for that series, but I hope to continue into more advanced and in depth ios topics later on. 

### Full stack?
When we talk about turning our application into full stack, we mean to combine the use of backend and frontend development. We have done a ton of front end development with User Experience and User Interaction and we have done a little bit of backend with Core Data, API Requests, and Data Modeling. We are going to turn our Beer Hug application into a full stack by incorperating it with Firebase.

### Introduction to Firebase
Firebase is a mobile and web application development platform. It's pretty great for a lot of complementary features that you would want for your application. We are going to be using the realtime database and the authentication, but it is extremely easy to add other features such as storage, crash reporting, or cloud messaging.

#### The goal
What we want to do is give users the ability to log in, via either email or facebook authentication, and then allow users to "hug" or save beers that they like to their profile so that they can view them later. 

### Firebase Set Up
First, either create a google account or use an existing one to log into [Firebase](https://firebase.google.com/). 

Click get started to go to the console.

Create a new project! Name it whatever you like. Then you can add an ios app.

You will see this screen:

![screen1]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/screen_1.png)

The bundle identifier is in the main project details:
It is in the form of a backwards .com url: com.bundlename.appname

One you type it in you can move to the next steps:
![screen2]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/screen_2.png)

Download the .plist file and include it in your project.

![screen3]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/screen_3.png)

Go into your pod file and edit it to have the following:
```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'
workspace 'YourWorkspaceName'
target 'YourAppName' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for BeerHug
  pod 'Alamofire', '~> 4.4'
  pod 'Firebase/Core'
  pod 'Firebase/Auth'
  pod 'Firebase/Database'
  pod 'Firebase/Storage'
  pod 'FacebookCore'
  pod 'FacebookLogin'
  pod 'FacebookShare'
  pod 'SwiftKeychainWrapper'

end
```

Most of these pods we aren't going to use yet, but we originally need the Firebase/Core for this step. Make sure you also put <kbd>workspace 'YourWorkspaceName'</kbd> so that when you run pod install, it adds it to your current workspace. 

Run the install
```bash
$ pod install
```

Last but not least:

![screen4]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/screen_4.png)

Toss in FIRApp.configure() along with the firebase import in your app delegate. Once you are done you should be able to use Firebase and all of it's features.

### Facebook Registration
Since we want to allow facebook registration, we need to do some set up with a facebook developer account. Head on over to the [facebook for developers](https://developers.facebook.com/) page. 

After signing in with a facebook account, in the upper right hand corner of the page you can select "My Apps" and add a new application. Put in the display name for your application and submit. You will be brought to a page with several "products" you can add. Click the "Get Started" button in the Facebook Login section.

You wil be brought to a page like this:
![fb1]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/fb_1.png)

You can go through the steps of the Facebook Login quickstart, but it may not help you much since most of the steps are default in Objective-C. We are doing swift. 

We do not need to manually download the Facebook SDK. In fact, we have already downloaded the proper dependencies by putting the following in our pod file:
```
pod 'FacebookCore'
pod 'FacebookLogin'
pod 'FacebookShare'
```

Next, go to the Settings tab of your facebook app window, add the Bundle ID of your Xcode project, similarly to how we did so with Firebase.

Right click your Info.plist file in your xcode project, and "Open as source Code." Copy and paste the following XML snippet into the body of your file (<dict>...</dict>):
Make sure to fill in your facebook app id where applicable. 
```
<key>CFBundleURLTypes</key>
<array>
  <dict>
  <key>CFBundleURLSchemes</key>
  <array>
    <string>fbYOUR_FACEBOOK_APP_ID</string>
  </array>
  </dict>
</array>
<key>FacebookAppID</key>
<string>YOUR_FACEBOOK_APP_ID</string>
<key>FacebookDisplayName</key>
<string>test</string>
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>fbapi</string>
  <string>fb-messenger-api</string>
  <string>fbauth2</string>
  <string>fbshareextension</string>
</array>
```

Next we need to connect via the app delegate.

### Facebook App Delegate Set Up
Import the following at the top of the AppDelegate file:
```swift
import FBSDKLoginKit
```

Add the following function to your AppDelegate.swift class:
```swift
func application(_ application: UIApplication, open url: URL, sourceApplication: String?, annotation: Any) -> Bool {
    return FBSDKApplicationDelegate.sharedInstance().application(application, open: url, sourceApplication: sourceApplication, annotation: annotation)
}
```

Then add the following to the first function (didFinishLaunchingWithOptions):
```swift
FBSDKApplicationDelegate.sharedInstance().application(application, didFinishLaunchingWithOptions: launchOptions)
```

What we are doing in these snippits is setting our application to contain an instance of the Facebook Login SDK that can be used throughout the app. We are also setting the app to be the source of the facebook requests so that we know where to return to. 

Once we have made these changes we should be able to compile successfully.

### Change authentication settings on Firebase
We still have just a couple more steps until we can successfully authenticate with facebook and firebase. Go into your firebase console on the web.

Click on the Authentication tab.
Go to Sign-In Method.
Enable Email/Password.
Enable Facebook. Make sure you type in your AppID that is in your Facebook app console, along with the secret (which is in the app console as well).

Once these are both enabled we can then add our authentication code to our project.

### Add authentication for facebook

First we need to create a view for signing in. As you can see on my main view controller, I have added a sign in button:
![sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/sign-in.png)

When you click on the sign in button, you are brought to a sign in page:
![sign-in-page]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/sign-in-page.png)

I'm not going to go through the steps of how to make this view, but you need two text fields for the name and password, a sign in button for those fields, and also another button that the user can click if they want to use facebook instead. 

If we wanted to we can add all kinds of authentication services, which is a addition we can add later on, such as Google, twitter, and Github. 

Create a view controller that contains all of the IBOutlets and IBActions for the objects on your sign in scene. 

Let's first add our imports:
```swift
import UIKit
import FBSDKCoreKit
import FBSDKLoginKit
import Firebase
import SwiftKeychainWrapper
```

Next we add our authentication functions:
```swift
@IBAction func facebookButtonPressed(_ sender: Any) {
    let facebookLogin = FBSDKLoginManager()
    facebookLogin.logIn(withReadPermissions: ["email"], from: self) { (result, error) in
        if error != nil {
            print("Unable to authenticate with facebook")
        } else if result?.isCancelled == true {
            print("User cancelled Facebook authentication")
        } else {
            print("JOSH: Successfully authenticated with Facebook")
            let credential = FIRFacebookAuthProvider.credential(withAccessToken: FBSDKAccessToken.current().tokenString)
            self.firebaseAuth(credential)
        }
        
    }
}
func firebaseAuth(_ credential: FIRAuthCredential) {
    FIRAuth.auth()?.signIn(with: credential, completion: {
        (user, error) in
        if error != nil {
            print("JOSH: Unable to authenticated with Firebase - \(error.debugDescription)")
            
        } else {
            print("Successfully authenticated with Firebase")
            if let user = user {
                let userData = ["provider": credential.provider]
                self.completeSignIn(id: user.uid, userData: userData)
            }
            
        }
    })
}


func completeSignIn(id: String, userData: Dictionary<String, String>) {
    navigationController?.popViewController(animated: true)
}
```
Here's what's happening:
1. We create a facebook login manager 
2. We use the manager to login. The only permissions that we are getting from the user is their "email", we can request more information later. 
3. If the request to log in ends up coming in as an error, then we are printing out that there was a problem, we also are checking to see if the user cancelled out of the window.
4. If there is no error, then we have successfully authenticated and we pass our facebook auth provider's credential to our firebaseAuth function

Within our firebase auth function:
1. We use our facebook credential to sign in via Firebase (instead of just the facebook request)
2. If there is an error then we print out what was wrong
3. Otherwise we have succesfully authenticated with firebase. We pass in the user.uid and the userData (which contains whatever user data we want to pass, in this case the credential provider). This is set up for later when we want to use auto sign in

Let's see if this works:

![facebookauth]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/facebookauth.gif)
![facebookauthdone]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/facebookauthdone.gif)

Beautiful.

### Add authentication for email
It is pretty simple to add authentication for email. Simply add this to the IB action of signing in with email and password and clicking "Sign In":
```swift
@IBAction func signInPressed(_ sender: Any) {
    if let email = emailField.text, let pwd = passwordField.text {
        FIRAuth.auth()?.signIn(withEmail: email, password: pwd, completion: { (user, error) in
            if error == nil {
                print("JOSH: Email user authenticated with Firebase")
                if let user = user {
                    let userData = ["provider": user.providerID]
                    self.completeSignIn(id: user.uid, userData:  userData)
                }
            } else {
                FIRAuth.auth()?.createUser(withEmail: email, password: pwd, completion: { (user, error) in
                    if error != nil {
                        print("JOSH: Unable to authenticated with Firebase using email - \(error.debugDescription)")
                    } else {
                        print("JOSH: Successfully authenticated with firebase")
                        if let user = user {
                            let userData = ["provider": user.providerID]
                            self.completeSignIn(id: user.uid, userData: userData)
                        }
                    }
                })
            }
        })
    }
}
```
Explanation:
1. We check to see if the email and password have been entered
2. If they have been then we attempt to sign in with the email and password 
3. If there is no error, then we complete the sign in
4. If there was an error, that means that the user does not exist and we try to create a new one
5. If there was an error from that then print the debug description
6. If there was no error then we are good to go

### Intro to FirebaseDB - saving our user in our database and auto login
We are going to dive into some FirebaseDB now. Go to your firebase console and navigate to the database. By default, firebase authentication saves the data of all users that sign up for our app. What it doesn't do automatically is create a user object in the database. We want to create a user object because we want to eventually save "hugged beers" to a profile. In order to do that we need to associate that in our database. 

Say we had this profile page actually be the first page of our app. We don't want our users to have to sign in every time. We want to save our user id in our application so that we can use it as a persistent login key. 

![db]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/db2.png)

In this picture we have the firebase db. Yours at this point would be empty, but it isn't difficult to understand. 

In this database we have a parent node called "users". Within the users node, we have several "object id's". These represent different users. Within each object ID we have a node for hugs which will represent liked beers, and a provider stating which authentication service was used to create the user. Within hugs we have another series of object id's that represent each liked beer. Each of those hug id's contains an id for the beer and the name of the beer. 

To mimic the creation of these fields, we create the following service.

Create a new file > DataService.swift
```
//
//  DataService.swift
//  BeerHug
//
//  Created by Josh Streger on 5/26/17.
//  Copyright © 2017 Josh Streger. All rights reserved.
//

import Foundation
import Firebase

let DB_BASE = FIRDatabase.database().reference()
let KEY_UID = "uid"

class DataService {
    
    static let ds = DataService()
    
    private var _REF_BASE = DB_BASE
    private var _REF_USERS = DB_BASE.child("users")
    
    var REF_BASE: FIRDatabaseReference {
        return _REF_BASE
    }
    var REF_USERS: FIRDatabaseReference {
        return _REF_USERS
    }
    
    func createFirebaseDBUser(uid: String, userData: Dictionary<String, String>) {
        REF_USERS.child(uid).updateChildValues(userData)
    }
}
```
To access the database we use firebase references. Once we have a reference, we can call child notes to extend the reference. We have created a function that creates a user using a user reference.

Edit the completeSignIn function:
```swift
func completeSignIn(id: String, userData: Dictionary<String, String>) {
    DataService.ds.createFirebaseDBUser(uid: id, userData: userData)
    KeychainWrapper.standard.set(id, forKey: KEY_UID)
    navigationController?.popViewController(animated: true)
}
```
Now everytime we log in with a new user, it should add it to our database. You can actually see this occur in real time:

![user_created]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/user_created.gif)

#### Auto sign in
If you noticed we have included a line in our completeSignIn function that saves our user id to a global key that is going to be used for auto login:
```swift
KeychainWrapper.standard.set(id, forKey: KEY_UID)
```

We can check to see if we are logged in through the following statement:
```swift
if KeychainWrapper.standard.string(forKey: KEY_UID) != nil {
    // Do something if your signed in
}
```
In this particular scenario, I have created a button on my main view called "view profile". I have set it to only appear if the user is logged in (using the conditional above in viewDidAppear:

![profile]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/profile.png)

I have also created a segue from the button action to a profile view:
![profile_view]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/profile_view.png)

This contains a uiview with a label and image at the top, and a table view with prototype cells similar to the BeerTableVC.swift. 

Our goal is to have the table load all of the beers that we have liked. But first we need to actually like beers. 

### Saving "hugged" beers to the database
Assuming that you have created a profile page for your beer that looks like the following:
![beer_pro]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/beer_pro.png)

The little beer hug icon at the top is actually a button. I want to save the hugged beer to the database when the button is clicked:
```swift
@IBAction func beerHugButtonPressed(_ sender: Any) {
    // this is where a "hug" implementation will go
    if let uid = KeychainWrapper.standard.string(forKey: KEY_UID) {
        let firebasePost = DataService.ds.REF_USERS.child(uid).child("hugs").childByAutoId()
        let beerPost: Dictionary<String, String> = [
            "name": self.beer.name,
            "id": self.beer.id
        ]
        firebasePost.setValue(beerPost)
        
    }
}
```
What this is doing:
1. Checking to see if the uid exists of the current user (the user is signed in)
2. If he is signed in, then we create a firebase database reference to the uid node. We go one node deeper to the hugs node, and then we call childByAutoID(). This creates a child node within hugs with a unique objectID. 
3. We set the beer post variable as a dictionary and pass in the name and id.
4. Finally we post to the database

See it in action:
![hugged_beer]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/hugged_beer.gif)

Next we need to retrieve all of our hugged beers. 
Create a new file FirebaseBeer.swift:
```swift
//
//  FirebaseBeer.swift
//  BeerHug
//
//  Created by Josh Streger on 5/26/17.
//  Copyright © 2017 Josh Streger. All rights reserved.
//

import Foundation

class FirebaseBeer {
    private var _name: String!
    private var _id: String!
    private var _postKey: String!
    
    var name: String {
        return _name
    }
    
    var id: String {
        return _id
    }
    
    var postKey: String {
        return _postKey
    }
    
    init(name: String, id: String, postKey: String) {
        self._name = name
        self._id = name
        self._postKey = postKey
    }
    
    init(postKey: String, postData: Dictionary<String, AnyObject>){
        self._postKey = postKey
        if let name = postData["name"] as? String {
            self._name = name
        }
        if let id = postData["id"] as? String {
            self._id = id
        }
        
        
    }
}
```
This is pretty much just setting up a class that can take in a post key and our firebase data.

Then we add some boilerplate code to our BeerCell.swift:
```swift
func configureFirebaseCell(beer: FirebaseBeer) {
    beerNameLabel.text = beer.name
    beerNameLabel.adjustsFontSizeToFitWidth = true
}
```

Lastly, we finish our profile page view controller:
```swift
//
//  ProfilePageVC.swift
//  BeerHug
//
//  Created by Josh Streger on 5/26/17.
//  Copyright © 2017 Josh Streger. All rights reserved.
//

import UIKit
import SwiftKeychainWrapper
import Firebase

class ProfilePageVC: UIViewController, UITableViewDelegate, UITableViewDataSource {
    @IBOutlet weak var tableView: UITableView!
    @IBOutlet weak var profileImage: UIImageView!
    @IBOutlet weak var profileName: UILabel!
    
    var beers = [FirebaseBeer]()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.delegate = self
        tableView.dataSource = self
        if let uid = KeychainWrapper.standard.string(forKey: KEY_UID) {
            print(uid)
            DataService.ds.REF_USERS.child(uid).child("hugs").observe(.value, with: { (snapshot) in
                
                if let snapshot = snapshot.children.allObjects as? [FIRDataSnapshot] {
                    for snap in snapshot {
                        print("SNAP: \(snap)")
                        if let fbdict = snap.value as? Dictionary<String, AnyObject> {
                            let key = snap.key
                            let fbeer = FirebaseBeer(postKey: key, postData: fbdict)
                            self.beers.append(fbeer)
                            print("Appended: \(fbeer.name), \(fbeer.id)")
                        }
                    }
                }
                self.tableView.reloadData()
            })
        }
      
    }

    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return beers.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let beer = beers[indexPath.row]
        print("JOSH:\(beer.name)")
        if let cell = tableView.dequeueReusableCell(withIdentifier: "BeerCell", for: indexPath) as? BeerCell {
            let beer = beers[indexPath.row]
            cell.configureFirebaseCell(beer: beer)
            print("configured Cell")
            return cell
        } else {
            return BeerCell()
        }
    }
    
    @IBAction func backButtonPressed(_ sender: Any) {
        navigationController?.popViewController(animated: true)
    }

}

```
1. We create a list of FirebaseBeers that are going to be displayed in the cells
2. We set the tableview delegate and datasource to ourselves 
3. We use a firebase function called observe(). What this does is pretty much add a listener to our "hugs" node. If there are nodes within hugs then those are passed on into what is called a snapshot. A snapshot is in the form of a dictionary (sort of like JSON). For each snapshot, we pass in the key and dictionary of the data to our FirebaseBeer class which will then add it to the list of FirebaseBeers. 
4. Since our cellForRowAt reads and configures the cell based on the list of firebase beers, we should see it display all of our data.

![done]({{ site.url }}{{ site.baseurl }}/assets/images/ios/full-stack/done.gif)

# Thank you
Next steps include allowing the user to view the profiles of the beers that he has hugged, comments on particular beers, ratings, etc. There are plenty of great things that can be done with this app. All of the source code is in the link at the top. 