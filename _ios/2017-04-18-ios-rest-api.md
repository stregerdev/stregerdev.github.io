//---
layout:
title: "6 - REST API and iOS"
excerpt: "CocoaPods, Alamofire, and a Simple Weather App"
tags:
    - ios
//---
## Create WeatherMe
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

target 'WeatherMe' do
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
[!] Please close any current Xcode sessions and use `WeatherMe.xcworkspace` for this project from now on.
```
This means exactly what it says. If we want access to our pods then we need to use WeatherMe.xcworkspace instead of our xcodeproj file.