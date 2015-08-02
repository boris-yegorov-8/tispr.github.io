---
layout: post
title: "How to stop explain where application version can be found"
date: 2015-08-02 14:30:19 -0700
comments: true
categories: iOS, Xcode, App Icon
author: Andrei Pitsko
---
How ofen do you face situation when your colleagues, QAs, business or marketing reported about issues in test builds, but they forgot to mention build number, application version. And when you ask about it, you should spend a lot of your time to explain what is build and app versions and where it can be found. Probably it is not so big problem when you have one-two build in month, but when you have a lot of different builds every week, it becomes very big issue

For instance in [Tispr](http://tispr.com) we have develop build after each commit, every week build for internal testing, every week build for bussines and marketing departments
(we will describe about our process in the next articles)

Let's solve this issue and do build number and app version is maximum visible for users

Let's do it on application icon?
Do you think that I am crazy because I suggest to do new icon for each new build...
But we are engineers, let's do it like engineers

So it is our idea:

{% img left images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/app_icon.png 100 100 'image' 'images' %}
**Develop** - it is type/name of build (in [Tispr](http://tispr.com) we have different type of builds, for instance: Develop build, Business build, etc),
**1.0.1** - application version,
**1023** - build number

<br/>
*Let's do coding:*

Add new "Run Script" in xcode in "Build Phase"
![application icon with overlay](images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/build_phase_1.png)
![application icon with overlay](images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/build_phase_2.png)

Get build number and application version from info plist in script
```
version=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" "${INFOPLIST_FILE}"`
build=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_FILE}"`
```

Let's imagine that we have function with name "addOverlayInfo" that have two parameters (name of income icon, name of icon that will be generated)
```
function addOverlayInfo() {
    income_icon_name=$1
    outcome_icon_name=$2
```

So we will have something like this:
```
addOverlayInfo "iPhone-60@2x.png" "AppIcon60x60@2x.png"
addOverlayInfo "iPhone-60@3x.png" "AppIcon60x60@3x.png"
```

where iPhone-60@2x.png, iPhone-60@3x.png - names of current application icons
During application building Xcode will generate icon with names AppIcon60x60@2x.png, AppIcon60x60@3x.png if you use Images.xcassets
So these names we use as second parameter in our calls

Let's find icon path by name of icon
```
income_icon_path=`find ${SRCROOT} -name $income_icon_name`
```

Creation path to save generated icon
```
outcome_icon_path="${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/${outcome_icon_name}"
```

And now the most important moment. generation icons with overlay and build number, application version

After fast search in internet we found that we will need that next packages(imagemagick and ghostscript).Let's install them
```
brew install imagemagick
brew install ghostscript
```
Find width of incoming icone
```
width=`identify -format %w ${income_icon_path}`
```

Generation new icon with all important information
```
convert -background '#0008' -fill white -gravity center -size ${width}x60 -pointsize 20\
caption:"$TYPE_OF_BUILD $version($build)" ${income_icon_path} +swap -gravity south -composite ${outcome_icon_path}
```

$TYPE_OF_BUILD - it is name of our build. you can remove it or set name of your build. in [Tispr](http://tispr.com) we use CI(go.cd) to setup 
value of this variable. we are going to publish a few articles about our Continuous Integration and Continuous Delivery process in future

*Mission completed*

You can find full script here: [Code](https://gist.github.com/Pitsko/993d81ac76e8d04ca1bc)
