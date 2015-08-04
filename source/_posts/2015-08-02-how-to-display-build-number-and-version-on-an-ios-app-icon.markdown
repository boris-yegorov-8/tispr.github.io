---
layout: post
title: "How to display build number and version on an iOS app icon"
date: 2015-08-02 14:30:19 -0700
comments: true
categories: [iOS, Continuous Delivery]
author: Andrei Pitsko
---
How often do you face the situation where your colleagues, QA, business or marketing report issues in test builds, but they forget to mention a build number or application version? And when they ask you about it, you spend a lot of time explaining the build number and application version, and where it can be found? When you have one to two builds per month the issue is not very big, but when you have a lot of different builds per week, it becomes cumbersome.

For instance at [tispr](http://tispr.com) we build on each commit and every week for internal testing, and weekly demo's for business & marketing departments.

Let's solve this issue, by creating a build number and app version with maximum visibility for users!

Our solution: On the application icon.
You must be thinking: A new icon for each new build?! Our answer is yes, why not.

Let's look at the how:

{% img left ({{ root_url }} /images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/app_icon.png 100 100 'image' 'images' %}
**Develop** - it is type/name of build (if you have a different type of builds, for instance: Develop build, Business build, etc);
**1.0.1** - application version;
**1023** - build number

<br/>
*Let's code:*

Add new "Run Script" in Xcode in "Build Phase"
![application icon with overlay]({{ root_url }} /images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/build_phase_1.png)
![application icon with overlay]({{ root_url }} /images/2015-08-02-how-to-stop-explain-where-application-version-can-be-found/build_phase_2.png)

Get build number and application version from info plist in script
```
version=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" "${INFOPLIST_FILE}"`
build=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_FILE}"`
```

Let's imagine that we have a function with the name "addOverlayInfo" which has two parameters (name of incoming icon, name of icon that will be generated)
```
function addOverlayInfo() {
    income_icon_name=$1
    outcome_icon_name=$2
```

The result is something like this:
```
addOverlayInfo "iPhone-60@2x.png" "AppIcon60x60@2x.png"
addOverlayInfo "iPhone-60@3x.png" "AppIcon60x60@3x.png"
```

where `iPhone-60@2x.png`, `iPhone-60@3x.png` - the names of the current application icons
While the application builds the Xcode, the icons with the names `AppIcon60x60@2x.png`, `AppIcon60x60@3x.png` will be generated, if you use Images.xcassets.
So, these names are used as second parameters in our calls.

Let's find the icon path by name of the icon:
```
income_icon_path=`find ${SRCROOT} -name $income_icon_name`
```

Create a path to save the generated icon:
```
outcome_icon_path="${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/${outcome_icon_name}"
```

And now the most important moment: Generating icons with an overlay, including the build number and application version. 

Following a quick search on the internet we found that we will need these packages(imagemagick and ghostscript). Let's install them:
```
brew install imagemagick
brew install ghostscript
```
To find the width of the incoming icon:
```
width=`identify -format %w ${income_icon_path}`
```

To generate the new icon with all the important information:
```
convert -background '#0008' -fill white -gravity center -size ${width}x60 -pointsize 20\
caption:"$TYPE_OF_BUILD $version($build)" ${income_icon_path} +swap -gravity south -composite ${outcome_icon_path}
```

`$TYPE_OF_BUILD` - The name of our build. 
To complete the process you might want to consider automating it by making it part of your Continuous Delivery process. 

*Mission completed*

You can find the full script here: [Code](https://gist.github.com/Pitsko/993d81ac76e8d04ca1bc)
