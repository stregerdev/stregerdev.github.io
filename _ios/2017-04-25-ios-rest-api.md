---
layout:
title: "9 - REST API and iOS"
excerpt: "CocoaPods, Alamofire, and a Beer Hug Part 1"
tags:
    - ios
---
[Source Code](https://github.com/jdstregz/BeerHug)

Welcome, welcome. It's time to dive into some more advanced topics in ios. In the next couple lessons we are going to be building a pretty awesome application called Beer Hug. Beer Hug will be an app where you can look up any information for any brewery or beer and it will tell you virtually anything about it. Let's get started.

Note: From this point forward, I'm not going to spend time in the tutorials going over how to create certain portions of the UI. I will show pictures and gifs on how my app looks and you can create it to your likeness. UI/UX development is a creative process, and you should spend time embarking on your own creativity. That being said, I will include source code for all functional parts of the application with descriptions. 

## Create Beer Hug
Create a single page application in Xcode. No need for Core Data. Feel free to turn off landscape modes. Exit out of this project once you have created it.

## What is CocoaPods?
[CocoaPods](https://cocoapods.org/) is a dependency manager for Swift and Objective-C Cocoa projects. It has over 30 thousand libraries and is used in over 1.9 million apps. CocoaPods can help you scale your projects elegantly.

We can easily search for cool and useful libraries to implement in our own projects. You can even share your own pods!

## What is Alamofire?
[Alamofire](https://github.com/Alamofire/Alamofire)is a Swift-based HTTP networking library for iOS and Mac OS X. It is going to be the basis of our RESTful mobile application. It is also the reason why we need CocoaPods, since Alamofire exists as a pod.

## Downloading and initializing CocoaPods for your project.
In your terminal, install the gem.
```bash
$ sudo gem install cocoapods
```
Navigate to your project folder within the terminal. We are going to initialize a pod file.
```bash
$ pod init
```

Within the pod file, change the contents to contain the following:
```ruby
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'BeerHug' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for WeatherMe
  pod 'Alamofire', '~> 4.4'

end
```
This allows us to download Alamofire as a pod (dependency). Time to install the pod we just inserted:
```bash
$ pod install
```
After a successfull pod installation, we should see a message like:
```
[!] Please close any current Xcode sessions and use `BeerHug.xcworkspace` for this project from now on.
```
This means exactly what it says. If we want access to our pods then we need to use BeerHug.xcworkspace instead of our xcodeproj file.

## Creating the Home Page
We want our homepage to be pretty simple. We basically want a text box where we can type in a name of our beer, and then a button that will initiate the search. This is what mine currently looks like. It has a lot more than necessary:

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/mainvc.gif)

Make sure to create IBOutlets for all objects in our UI as well as IBActions for all of our buttons. We can also implement the same scroll view fix for the keyboard covering our fields like we have in previous examples. Eventually what we want to do is implement different types of searches, such as filtering our results by the ABV or IBU fields. Also with breweries we want to also find breweries near us via location services. For now, a single text field will suffice.



## Creating the Beer Table View
Our second view is going to be a table view. This will be very similar to how we have created table views in the past. 

Create a table view with a prototype cell. Include a label to represent the name of the beer (some beers have long names so we need to conform it to the width of the cell). The height of my cell is 60. I also made the entire table a beer-ish color. 

Make sure to incorperate IBOutlet for the table view and set the data source and delegate to the self while including UITableViewDelegate and UITableViewDataSource in the class declaration. There will be the functions that we need to implement to make the errors dissapear. 

Create the seque from our first page to our new table view page and add the performSegue function to our button pressed. We don't need to pass any data in at the moment. This is all setting up for what we want to do. 

BeerCell.swift:
```swift
class BeerCell: UITableViewCell {

    @IBOutlet weak var beerNameLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }
    
    func configureCell(beer: Beer) {
        beerNameLabel.text = beer.name
        beerNameLabel.adjustsFontSizeToFitWidth = true
    }
}
```

BeerTableVC.swift:
```swift
//
//  BeerTableVC.swift
//  BeerHug
//
//  Created by Josh Streger on 5/15/17.
//  Copyright © 2017 Josh Streger. All rights reserved.
//

import UIKit
import Alamofire

class BeerTableVC: UIViewController, UITableViewDelegate, UITableViewDataSource {
    
    @IBOutlet weak var tableView: UITableView!
    
    private var _search: String!
    
    var search: String {
        get {
            return _search
            
        } set {
            _search = newValue
        }
    }
    
    var beers = [Beer]()

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        tableView.delegate = self
        tableView.dataSource = self
        
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
    
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return beers.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        if let cell = tableView.dequeueReusableCell(withIdentifier: "BeerCell", for: indexPath) as? BeerCell {
            let beer = beers[indexPath.row]
            cell.configureCell(beer: beer)
            return cell
        } else {
            return BeerCell()
        }
        
    }
    
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60
    }
    
    

    @IBAction func backButtonPressed(_ sender: Any) {
        _ = navigationController?.popViewController(animated: true)
        
    }
}

```
We are going to have a couple of errors at this point because we haven't defined our class "Beer". We also shouldn't create our beer model until we know what it needs to contain. Our goal is to pass in the value of the text field to our table view scene, which will then access a large database of beers and return a list of beers that possibly match that search. If only we had a large database of beers...

## Registering for BreweryDB
Luckily, we totally do have access to a large database of beers. BreweryDB is an awesome API that has a ton of beer and brewery information. 
[Register for BreweryDB Here!](http://www.brewerydb.com/developers). 
The way that API's work is similar to when you request a url. Web browsers send requests to sites that return HTML in the form of web pages. This API (when requests are sent to it) will return data in the form of JSON (Javascript Object Notation). After you register your app you will be able to access parts of the database for free (up to 400 requests per day) which is perfect for development. 

Download [Postman](https://www.getpostman.com/). This little application is amazing for sending requests to endpoints of your choosing, and can return pretty data. Once you've downloaded postman. Check out what a search requests for the beer "Bud Light" returns:

For the purposes of this tutorial, I cannot show you my whole url. The reason is because with every registraton for the BreweryDB, a special api key is given for access to the DB. You want to keep this key secret to your own application. 

The request url looks like this:
http://api.brewerydb.com/v2/search/?key=YOUR-API-KEY&q=bud light&type=beer

And our output:
![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/postman-output.png)

You may not see it yet, but this JSON output is awesome. In swift there is a great way to decompose the data within our JSON into a format we can use in our classes.

## Beer Data Model
We can see in our output of our search that there are many fields to a beer. The ones that we are going to label as important are the following:
- id
- name
- description
- abv
- ibu
- glass["name"]
- style["category"]["name"]
- style["name"]
- servingTemperature
- foodPairings
- availability

You can also view the BreweryDB documentation to see if there are any other return fields that you are interested in. I would say out of these fields, the last three only appear sometimes, so I'll explain special cases for that later.

We need to actually create the beer class:

```swift
//
//  Beer.swift
//  BeerHug
//
//  Created by Josh Streger on 5/15/17.
//  Copyright © 2017 Josh Streger. All rights reserved.
//

import Foundation
import Alamofire

class Beer {
    var _id: String!
    var _name: String!
    var _nameDisplay: String!
    var _description: String!
    var _abv: String!
    var _ibu: String!
    var _iconURL: String!
    var _mediumImageURL: String!
    var _largeImageURL: String!
    var _styleCategory: String!
    var _styleID: Int!
    var _styleName: String!
    var _isOrganic: String!
    var _servingTemperature: String!
    var _availability: String!
    var _glass: String!
    var _foodPairings: String!
    
    var foodPairings: String {
        if _foodPairings == nil {
            _foodPairings = "N/A"
        }
        return _foodPairings
    }
    
    var glass: String {
        if _glass == nil {
            _glass = "N/A"
        }
        return _glass
    }
    
    var availability: String {
        if _availability == nil {
            _availability = "N/A"
        }
        return _availability
    }
    var id: String {
        if _id == nil {
            _id = ""
        }
        return _id
    }
    
    var name: String {
        if _name == nil {
            _name = "N/A"
        }
        return _name
    }
    
    var nameDisplay: String {
        if _nameDisplay == nil {
            _nameDisplay = "N/A"
        }
        return _nameDisplay
    }
    var description: String {
        if _description == nil {
            _description = "N/A"
        }
        return _description
    }
    var abv: String {
        if _abv == nil {
            _abv = "N/A"
        }
        return _abv
    }
    var ibu: String {
        if _ibu == nil {
            _ibu = "N/A"
        }
        return _ibu
    }
    var iconURL: String {
        if _iconURL == nil {
            _iconURL = "N/A"
        }
        return _iconURL
    }
    var mediumImageURL: String {
        if _mediumImageURL == nil {
            _mediumImageURL = "N/A"
        }
        return _mediumImageURL
    }
    var largeImageURL: String {
        if _largeImageURL == nil {
            _largeImageURL = "N/A"
        }
        return _largeImageURL
    }
    /*
     var _styleCategory: Int!
     var _styleID: Int!
     var _styleName: String!
     var _isOrganic: String!
     var _servingTemperature: String!
 
 */
    var styleCategory: String {
        if _styleCategory == nil {
            _styleCategory = "N/A"
        }
        return _styleCategory
    }
    var styleID: Int {
        if _styleID == nil {
            _styleID = 0
        }
        return _styleID
    }
    var styleName: String {
        if _styleName == nil {
            _styleName = "N/A"
        }
        return _styleName
    }
    var isOrganic: String {
        if _isOrganic == nil {
            _isOrganic = "N/A"
        }
        return _isOrganic
    }
    var servingTemperature: String {
        if _servingTemperature == nil {
            _servingTemperature = "N/A"
        }
        return _servingTemperature
    }
    
    
}
```
All that we are doing here is setting up our variables for our class and also making it so that our variables are never nil. 

## Filling our Beers

Before we reach the part where we implement Alamofire, we need to do a couple things first. 

1. We need to add a privacy setting to our application's Info.plist to allow Arbitrary Loads of data from non-secure links. This means that we are allowing our application to load http requests, not just https. 

2. We also need to make a Constants.swift file (New > File > Swift). This is where we are going to store all of our constants for our requests

This is what mine looks like. Keep in mind that I'm censoring my own API Key:

```swift
import Foundation

typealias  DownloadComplete = () -> ()
// base urls
let BASE_URL = "http://api.brewerydb.com/v2/search"
let API_KEY = "&?key=THIS_WILL_BE_WHERE_YOUR_API_KEY_GOES"
let Q = "&q="
let TYPE_BEER = "&type=beer"
```

These are pretty much just a bunch of constants so that we don't have to write out url's everywhere. The typealias Download Complete is a little special. It's pretty much a way that we are going to declare when our api requests have completed, in order to execute particular functions after. 

Now that we have done those things, we can actually make our request.

This will be in your MainVC:
```swift
@IBAction func pourBeerPressed(_ sender: Any) {
      let search = beerNameField.text!
      performSegue(withIdentifier: "SearchBeer", sender: search)
      
}

// Make sure to set the identifier for your segue to "SearchBeer"
// this is very similar to our segue's in previous tutorials

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if segue.identifier == "SearchBeer" {
        if let destination = segue.destination as? BeerTableVC {
            destination.search = sender as? String
        }
    }
```
This will be added to our Beer.swift class:
```swift
init(beerDict: Dictionary<String, AnyObject>) {
    if let name = beerDict["name"] as? String {
        self._name = name.capitalized
    }
    if let abv = beerDict["abv"] as? String {
        self._abv = abv
    }
    if let ibu = beerDict["ibu"] as? String {
        self._ibu = ibu
    }
    if let details = beerDict["description"] as? String {
        self._description = details
    }
    if let labels = beerDict["labels"] as? Dictionary<String, AnyObject> {
        if let icon = labels["icon"] as? String {
            self._iconURL = icon
        }
        if let medium = labels["medium"] as? String {
            self._mediumImageURL = medium
        }
        if let large = labels["large"] as? String {
            self._largeImageURL = large
        }
    }
    if let servingTemp = beerDict["servingTemperatureDisplay"] as? String {
        self._servingTemperature = servingTemp
    }
    if let organic = beerDict["isOrganic"] as? String {
        self._isOrganic = organic
    }
    if let available = beerDict["available"] as? Dictionary<String, AnyObject> {
        if let availableName = available["name"] as? String {
            self._availability = availableName
        }
    }
    if let style = beerDict["style"] as? Dictionary<String, AnyObject> {
        if let category = style["category"] as? Dictionary<String, AnyObject> {
            if let categoryName = category["name"] as? String {
                self._styleCategory = categoryName
            }
        }
        if let styleTitle = style["name"] as? String {
            self._styleName = styleTitle
        }
    }
    
    if let glassSection = beerDict["glass"] as? Dictionary<String, AnyObject> {
        if let glassName = glassSection["name"] as? String {
            self._glass = glassName
        }
    }
    
    if let foodPairing = beerDict["foodPairings"] as? String {
        self._foodPairings = foodPairing
    }
    
    if let idText = beerDict["id"] as? String {
        self._id = idText
    }
}
```

init is a function that goes along with classes. It is a way to create an instance of Beer using a Dictionary. A Dictionary is pretty much like a mapping of data. In this case we are using Dictionary<String, AnyObject> to describe the format of the JSON data. In this sense if we want to access the name of our beer, we only need to access beerDictionary["name"], since the json data uses the name keyword to show the name of the beer. There are sections such as the glass, style, and image urls that will require another nesting of Dictionary<String, Any Object>.

Now that we have that in place we can call our Alamofire module to actually get this data:

Then we have the following function in our BeerTableVC:
```swift
override func viewDidAppear(_ animated: Bool) {
      print("view did appear")
      if self.beers.isEmpty {
          self.downloadBeerData {
              print("beer data downloaded")
              self.tableView.reloadData()
          }
      }
  }


func downloadBeerData(completed: @escaping DownloadComplete) {
      
      let searchString = BASE_URL+API_KEY+Q+this.search+TYPE_BEER

      var searching = searchString.addingPercentEncoding(withAllowedCharacters: NSCharacterSet.urlQueryAllowed)!

      let url = URL(string: searching)!
      Alamofire.request(url).responseJSON { response in
          let result = response.result
          print(response)
          if let dict = result.value as? Dictionary<String, AnyObject> {
              if let list = dict["data"] as? [Dictionary<String, AnyObject>] {
                  for obj in list {
                      let beer = Beer(beerDict: obj)
                      
                      self.beers.append(beer)
                  }
                  
                  self.tableView.reloadData()
              }
          }
          completed()
      }
      
  }
```
Not too complicated:
1. We take our search name and put it in the full string with all of our constants to make the actual request url
2. We then need to pass this string into a function that will make our string safe to transform into a url. 
3. We create the url and pass it into Alamofire
4. We take the response's result and we cast it as a Dictionary<String,AnyObject>
5. We want to get the list of all of the returned beers that exists in the "data" part of the dictionary, so we create an array of Dictionaries to represent the returned beers.
6. For each dictionary in that list, we create a beer that we can then append in our beer list
7. We reload the table view, and based on our cellForRow, it should add the beers

Cheers:

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/cheers.gif)

There are a ton of features you can add to this simple way to send requests to the beer api. All of the following features I have added in the source code that I have linked at the top of this tutorial. I will not go into detail on how to do each addition, but get creative with the BreweryDB API:

#### Beer Profile
Create a beer profile to pass the beer data into upon clicking on a cell. You can either request and save the brewery data with the beer data or send a different request:

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/beer_profile.gif)

#### Brewery Search
Create a different button to search for breweries:

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/brewery_search.gif)

#### Brewery Profile
Open up a brewery profile

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/brewery_profile.gif)


#### Brewery Beers
Have a button that gets all of the beers that belong to the designated brewery

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/brewery_beers.gif)

#### Location Search
Add in a way to use a location picker to search via geo locations (I used a cocoapod for the location picker):

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/location_picker.gif)

![mainvc]({{ site.url }}{{ site.baseurl }}/assets/images/ios/rest-api/location_search_profile.gif)

Stay tuned for the next tutorial where we will be making our application full stack with firebase!


