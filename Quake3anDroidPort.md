Guide to port Quake3 to an android device				                                                                 2014
Author: Rahul Kumar Dutta

Assuming that you have successfully installed the GPU drivers for your graphics card in your Linux Ubuntu desktop, start following these well furnished steps for your successful porting of the Quake3 game to your android device.
Non-developers may look into the following website link: https://code.google.com/p/kwaak3/ for instructions on how to install quake3 to your android device directly.

Cross-compiling and building game engine libraries of the application ioquake3:

Go to this link first: http://developer.android.com/sdk/index.html 
Download Eclipse ADT with the android SDK for Linux: choose 64 bit download
Unzip the package and place it in a folder named “Development” (anywhere or any name)
Click on the package and you should be able to find a folder named “eclipse” inside of which there is an executable of eclipse. Always use this executable to launch eclipse.
Next you have to go to the link below to download android NDK: http://developer.android.com/tools/sdk/ndk/index.html 
Download android NDK for Linux 64-bit (x86) platform available for 32 bit targets which is “android-ndk32-r10b-linux-x86_64.tar.bz2”. The reason we need this package only is that, for “Linux 64-bit (x86) platform-64 bit targets” you won’t get support for libraries that are needed for Android sdk API: 1-19 which is only available at present to the “Linux 64-bit (x86) platform-32 bit targets”. So, if your device has android 4.4.2, or kit-kat, then the package “android-ndk32-r10b-linux-x86_64.tar.bz2” should work for you. API for android-L is only available to “Linux 64-bit (x86) platform-64 bit targets” which I think is getting ready for the next release of android 5 distribution.
Once you have downloaded “android-ndk32-r10b-linux-x86_64.tar.bz2”, extract it to your “Development” folder as mentioned earlier.
After keeping it there, please don’t do anything with it, follow the next step.
The next thing you have to do is to install ADB or Android Debug Bridge which acts as an interface between your development environment and the device.
Please follow the instructions in the following link to set up ADB for your device:
	http://developer.android.com/tools/help/adb.html
If you want to use an emulator instead of the device itself, then create AVD’s or Android Virtual Devices for your applications. If you are curious then look here:
	http://developer.android.com/tools/devices/emulator.html
Then download the source code as zip from github source: https://github.com/courtc/kwaak3-cc
            or download directly using: 
 svn co https://github.com/courtc/kwaak3-cc kwaak3
            if the source address link has .git at the end, then use this,
            git clone <link>
put the kwaak3 code in a separate folder named “code” in “Development”
Kwaak3 has two stand-alone applications named “ioquake3” and “kwaak”. “ioquake3” is the standalone native code for this game whereas “kwaak” is the application code for launching this game.
Next step is to cross-compile the native code of “ioquake3” from the localhost to the target arm architecture which is in this scenario is our android platform. We must create a toolchain compiler for the android API where we may compile our code. So, we have to now go to the ndk package in “Development” folder where we had unzipped the ndk, named something like “android-ndk-r10b”. Go to build>tools and find the “make-standalone-toolchain.sh”. Remember the location of this file because you will need it in the next step.
Now open a Linux terminal window and go to the location as we mentioned in the previous step:   Development/android-ndk-r10b/build/tools
create a toolchain by typing the following command:
	
“bash make-standalone-toolchain.sh  --ndk-dir=/home/rahul/Development/android-ndk-r10b --toolchain=arm-linux-androideabi-4.6 --platform=android-4 --install-dir=/tmp/myDir --system=linux-x86_64”

now go to tmp/myDir , cut “myDir” directory and paste it inside your “Development” folder. Then rename “myDir” as “arm-linux.androideabi-4.6”. Then inside your “Development” directory,  create another folder named “toolchainAndroid4” and move “arm-linux.androideabi-4.6” to this folder. So it looks like your new toolchain exists here : 

“/Development/toolchainAndroid4/arm-linux.androideabi-4.6”

Now close this terminal window and open another Linux terminal window. Go to the location Development/code/ioquake3/ and open the file “Makefile.local” in an editor. 
            Now change the following things:
 	ANDROID_NDK_DIR=/home/Development/android-ndk-r10b/ndk-build

CC=/home/rahul/Development/toolchainAndroid4/arm-linux-androideabi-4.6/bin/arm-linux-androideabi-gcc-4.6

Remove the ‘#’ from:
ANDROID_CFLAGS=-march=armv5te -mtune=xscale -msoft-float -fpic -mthumb-interwork -ffunction-sections -funwind-tables -fstack-protector -fno-short-enums -D__ARM_ARCH_5__ -D__ARM_ARCH_5T__  -D__ARM_ARCH_5E__ -D__ARM_ARCH_5TE_
	
and add ‘#’ on other “ANDROID_CFLAGS” if it is not there. Putting a ‘#’ before these flags means you are disabling that flag. Our target architetcure is arm, so we enabled the “ANDROID_CFLAGS” that has “-march=armv5te” which means we are targetting to this “armv5te” architecture. Don’t panic about what are the other parameters about. Let them as it is.
Then, use these two flags only
CFLAGS=$(ANDROID_CFLAGS) -I$(ANDROID_NDK_DIR)/build/platforms/android-4/arch-arm/usr/include -DANDROID -D__linux__ -D__MATH_NEON
LDFLAGS=-nostdlib -L$(ANDROID_NDK_DIR)/build/platforms/android-4/arch-arm/usr/lib

You are done with this “Makefile.local”. Save everything that you changed and exit.
Now in the terminal window, go to the location “/Development/code/ioquake3/” and type the command “make”. After your build is complete, you should be able to find 
the game engine binaries,
         
“libquake3.so” inside this location: “/Development/code/ioquake3/build/release-linux-arm/”
  and 
 “libcgamearm.so”, “libqagamearm.so”, and “libuiarm.so” inside this location:
 “/Development/code/ioquake3/build/release-linux-arm/baseq3”

If you don’t see these files, then your build was most likely unsuccessful and therefore it could not generate those binaries. If you encounter this problem make sure you have made changes to the Makefile.local appropriately as I have mentioned, as for example, wrong location of  “ANDROID_NDK_DIR” or “CC” and selecting appropriate flags by enabling with removing ‘#’ or disabling unnecessary flags of your application by adding a ‘#’. 
Now you are actually done with the compiling of the native code of “ioquake3” and you’ve generated the necessary game engine binaries for the launching application of the project which is “kwaak”.    

Launching Kwaak in Eclipse environment and integrating game engine binaries  together

Now we have to build and compile the “kwaak” module. So now open the eclipse from the ADT that you have installed earlier in your Development folder. Now import a project by clicking on the file menubar, and select Android>Existing android code into workspace. Click next and select the kwaak project in Development/code/kwaak3-cc-master. check the “copy projects into workspace” checkbox and Finish.

Open the Android SDK manager by clicking the first android icon at the top on eclipse. There select the checkbox Android 4.4w, Android 1.6.  Also select in Tools the following packages:
Android SDK Tools, Android SDK platform- tools, Android SDK build tools
And in Extras check the following packages: Android Support Repository and Android Support Library.
Now install all the packages that you have selected.

Close eclipse and reopen again. 

Now select the application named “Launcher” and go to Project>properties where you select the android version for the SDK API which should be Android 4.4w. Similarly for the “Native Activity” application select Android 1.6 and apply and press ok to exit.

Now again select the Launcher application then go to Project>properties>Builders. On the right click New, click on Program and press ok. In the main tab, select the location of your NDK like this: “/home/rahul/Development/android-ndk-r10b/ndk-build” You must not forget to select ndk-build. Don’t select ndk-build.cmd, it is for windows platform. Select your working directory as Launcher that looks like something like this: ${workspace_loc:/Launcher}

Press apply and go to the Refresh tab. There check the unchecked box “During auto builds” and “Specify working set of relevant resources”. Then click “Specify resources” button.
For the Launcher application only select the “jni” checkbox, click finish, ok and exit. If this new builder is not selected on the builders list by now, then check it. Press ok. Your auto build now should have generated “libqwaakjni.so” under the hieararchy /Launcher/libs/armeabi. Just check it. Then in the same folder /Launcher/libs/armeabi , add the “libquake3.so”  “libcgamearm.so”, “libqagamearm.so”, and “libuiarm.so”  binaries, which were generated during the build of the application “ioquake3” by doing simple copy-paste.

Now click the Run Configurations from the drop down menu of Run. Under Android Application at left, add a new configuration. To the right select “Launcher” after clicking Browse and then click on the Target tab. Select the very first radio button which will allow the user to select manually the device from the options available at the prompt. At the bottom, to the “Additional Emulator Command Line Options” add the string “-gpu on”. This option is only applicable if you are using an emulator as a device. Click Run button. Select your device and press ok.
By this time you would not be able to start the game and you would get errors as if it could not find the location /sdcard/quake3/baseq3. Assuming that your device is connected with your development environment using adb, now create the path /sdcard/quake3/baseq3 by using the commands in the terminal:

	adb shell mkdir /sdcard
	adb shell mkdir /sdcard/quake3
	adb shell mkdir /sdcard/quake3/baseq3

Then download the files pak0.pk3-pak8.pk3 from the desktop version of the game by installing it on your desktop and find these files in the installed directory of your desktop.
Then push these files by using the command:
adb push <source directory path of the files> /sdcard/quake3/baseq3

Now, it is the time to run your application by using your previously configured  run settings. Run this configuration on your attached device, and you’d definitely see the welcome screen of the game. Then start and enjoy the game!

