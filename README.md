# crystal
Demo PhoneGap application instrumented with Adobe Mobile Services

This PhoneGap application provides a basic sample of instrumenting your app using [Adobe Mobile Services](https://marketing.adobe.com/developer/get-started/mobile/c-measuring-mobile-applications).

## Configure Adobe Mobile Services

Set up the report suite and the app:

1. Log in to Adobe Mobile Services at https://mobilemarketing.adobe.com/ with your normal Analytics account credentials.
1. Click on _Manage Apps_ and then _Add_
1. Select _New Report Suite_
1. Enter a name for the report suite, a report suite ID, and click _Save_.
1. Click on _Config File_ at the bottom of the page to download your ADBMobileConfig.json file.

## Create PhoneGap Application

Follow the CLI documentation. Here we create an app called ‘Crystal’ (Analytics, Analyse This, Billy Crystal …):

     cordova create crystal com.andrewsavory.crystal Crystal

Add platforms. The Mobile Services PhoneGap plugin currently supports iOS and Android, so we’ll add both.

     cd crystal
     cordova platform add ios
     cordova platform add android

Now edit the config.xml to add the Adobe Mobile Services SDK:

	<feature name="ADBMobile">
	     <param name="id" value="https://github.com/Adobe-Marketing-Cloud/mobile-services#0482f9cedf90c98a8d4b07219ece1933b2e46a60"/>
	</feature>

… and install:

     cordova restore plugins --experimental

You could also do this with:

     phonegap plugin add https://github.com/Adobe-Marketing-Cloud/mobile-services#0482f9cedf90c98a8d4b07219ece1933b2e46a60
     cordova save plugins --experimental

The id specifies a specific version of the plugin in git.

Copy your `ADBMobileConfig.json` file into the `www/` directory and then set up a script that will copy it into the correct platform location at build time. Update the `config.xml` to ensure this script is run during build by adding hooks. While we’re at it, we’ll automate plugin installation and copy any additional resources:

	<hook src="scripts/restore_plugins.js" type="after_platform_add"/>
	<hook src="scripts/copy_resource_files.js" type="after_prepare"/>
	<hook src="scripts/copy_AMS_config.js" type="after_prepare"/>

Once this is done you can build and run your app:

     cordova run ios
     cordova run android

This is all you need to do to be able to get full lifecycle metrics – launches, time in app, platform, operating system, etc. will all automatically be collected for you. Easy, huh?

## Path Analytics

To see more useful path analysis, add an extra page to your project:

     cp index.html screen2.html

Update the `<title>` tag in screen 2 to something more appropriate (e.g. "screen 2”).

Now we need to specifically call AMS functionality in order to track each screen that is shown. Documentation on all the available methods is at https://marketing.adobe.com/resources/help/en_US/mobile/ios/phonegap_methods.html

We should turn on debug logging by adding this to `onDeviceReady` in `index.js`:

     ADB.setDebugLogging(true);

And then for each of our screens, we'll need to call `ADB.trackState(“my screen name");`. Let’s do this by adding a trackScreen function:

    // track screens in Adobe Mobile Services
    trackScreen: function(id) {
        var title = document.getElementsByTagName("title")[0].innerHTML;
        ADB.trackState(title);
        console.log('Tracked screen: ' + title);
    }

We call it from `onDeviceReady`:

    app.trackScreen('title');

And update our `index.html` and `screen2.html` to link to each other.

If you run in the iOS simulator you will see something like:

	2015-04-04 07:18:19.043 Crystal[12655:928268] ADBMobile Debug: Analytics - Request Queued (ndh=1&t=00%2F00%2F0000%2000%3A00%3A00%200%20-120&ts=1428124699&ce=UTF-8&aid=2A8F470785311E33-60000120601138B4&c.&a.&Resolution=750x1334&AppID=Crystal%200.0.1%20%280.0.1%29&DeviceName=x86_64&OSVersion=iOS%208.2&CarrierName=%28null%29&.a&.c&pageName=Hello%20World)

After a short while (15 minutes or so), you should see your data show up in _View Paths_ in Adobe Mobile Services.