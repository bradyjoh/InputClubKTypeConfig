### K-Type Dev Notes

* Setting up Docker
  * done on a mac instead of windows so I have more useful command line utilities
  * https://docs.docker.com/docker-for-mac/install/
    * needs login to download, I'll save a copy on the NAS
    * Install it and run it and then docker is running. Check with `docker info`
  * KiiboHD Controller Docker setup https://github.com/kiibohd/controller/tree/master/Dockerfiles
    * More setup notes for non-docker setup: https://kiibohd.github.io/wiki/#/Setup
    * first download the controller repo: https://github.com/kiibohd/controller
      * NOPE there are issues later with unpacking the ZIP not having Git Tags. Clone the repo instead. I did mine into `/Users/john/GitHub/controller`
    * Then go to the Dockerfiles directory and build (i'm going to try ubuntu)
      * `docker build -f Dockerfile.ubuntu -t controller.ubuntu .` which says build using the Dockerfile.ubuntu file with the name controller.ubuntu. Then it'll go and pull/build a bunch of packages, took me a few minutes of waiting for it to finish. 
    * Then to get into the docker/build environment I can use `docker run -it --rm -v "$(pwd):/controller" controller.ubuntu` to get in and `exit` to get out
      * NOTE: I tried to run this from within the Dockerfiles directory and got an empty Keyboards folder. Instead, I had to run it from the directory above, the root of the repo
      * NOTE: You may need to call `exit` twice if you're in the pipenv environment within Docker.
  
* In the Docker build environment
  * You'll be dropped into the Keyboards folder which has different bash scripts to build for each keyboard type. Since we want the K Type we'll be using `./k-type.bash` to build the firmware 
  * Running `./k-type.bash`
    * I got the note the first time that `ERROR: pipenv is not active!` which instructs me to go to the keyboards directory first and then run `pipenv install` and `pipenv shell` to "launch a subshell in a virtual environment"
      * Will need an additional `exit` to get out of this
      * I think I need to run the install every time but I'm not sure
      * 
    * Now when I ran it, it printed out a bunch of info and also ran into an error. But it did tell me that it's building with KLL version 0.5.7.3 which is good to know.
    * I'm seeing an error when just running it by default, where it says `../../Debug/cli/cli.c:716:47: error: operator '&&' has no right operand` on ` #if CLI_RevisionNumber && CLI_VersionRevNumber`
      * SO post: https://stackoverflow.com/questions/48538243/macro-if-statement-returns-error-operator-has-no-right-operand which makes me think maybe CLI_VersionRevNumber isn't getting defined in my code
      * AHA! I found someone else that posted my issue: https://github.com/kiibohd/controller/issues/267
      * They said "Confirming that this fix works when cloning the repo, but not when running in a downloaded and unzipped copy." so it looks like I have to re-set this all up by cloning the repo rather than downloading and unpacking a ZIP. 
    * There were notes in the compilation about the secure DFU or not:
      * NOTE: kiibohd.secure.dfu.bin is required for secure Kiibohd Bootloaders. Only keyboards from late 2017 and onwards, support this. See lsusb -d 1c11: -v and look for iInterface 4 Kiibohd DFU Secure. Otherwise, use the kiibohd.dfu.bin file when it says iInterface 4 Kiibohd DFU. Use the physical reset button on the back of the keyboard to temporarily disable secure mode.
        * I got my keyboard in 11/2017 so I guess it supports both
      * NOTE2: Secure mode is disabled currently, but will be enabled when key negotiation is supported.
    * By default, the following list of layout files appears to have been used (meaning any KLL changes in any of them should be reflected) and I think I can change these in the k-type.bash file
      * /controller/Scan/Devices/ISSILed/capabilities.kll
      * /controller/Scan/Devices/MatrixARMPeriodic/capabilities.kll
      * /controller/Scan/Devices/PortSwap/capabilities.kll
      * /controller/Scan/Devices/UARTConnect/capabilities.kll
      * /controller/Macro/PartialMap/capabilities.kll
      * /controller/Macro/PixelMap/capabilities.kll
      * /controller/Output/HID-IO/capabilities.kll
      * /controller/Output/USB/capabilities.kll
      * /controller/Debug/latency/capabilities.kll
      * /controller/Debug/led/capabilities.kll
      * /controller/Scan/K-Type/scancode_map.kll
      * /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/k-type/release.1.kll
      * /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/stdFuncMap.kll
  * Running KLL checker https://github.com/kiibohd/controller/tree/master/Keyboards#self-testing-a-kll-layout
  
* KLL Customizations
  * Gamma correction for proper oranges/reds: https://github.com/kiibohd/controller/commit/a16ee2c3a6ea9dce1c189f14ce8ebd0ea845a659 https://github.com/kiibohd/controller/issues/255
    * As of KLL 0.5.7.0
    * Looks like we can call gamma(0) to disable, gamma(1) to enable, or gamma(2) to toggle, and gamma is a shortcut to call Pixel_GammaControl_capability
    * Also looks like LEDGamma is used to set the value, where the value is currently set to 1.0. I might try changing this to 2.2 (recommended gamma) to see what happens. Looks like it gets changed in the keyboard-specific scancode_map.kll OR in Macro/PixelMap/capabilities.kll. `LEDGamma = 2.2;`  I also need to add `gamma_enabled = "1"; ` to that scan code_map.kll file for my keyboard. I pulled the example from the Kira map. 
    * Gamma set to 2.2 caused the keyboard to flicker. Enabled(1) and 1.0 gamma did not flash. Disabled and 2.2 Gamma did not flash. Enabled and 1.5 flashed, as did 3.0. I added to the Github issue above
    * TODO Still haven't explored the gamma toggle functionality, which would allow me to toggle without flashing
  * Defining keys on different layers
    * Defined here: https://github.com/kiibohd/controller/tree/master/Keyboards#example-usage-with-web-configurator-layout-files
    * Different keymaps described https://kiibohd.github.io/wiki/#/Keymaps
    * It's noted that The order of the files matter, as the right-most file will overwrite any setting in the previous files.
    * Make a folder in the controller folder for your custom KLL specifically under /controller/kll/layouts/, so it could be /controller/kll/layouts/bradyjoKType
    * Add all custom KLL files into the folder
    * Edit the bash file to replace the BuildPath, DefaultMap and PartialMap if those are things you reconfigured
      * I found everything in the original bash file under `/root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/` which is different than the `controller/kll/layouts/` destination indicated in the web guide. I also found a note that the "Compiler looks in kll/layouts and the build directory for layout files (precedence on build directory)"
      * BuildPath means nothing for the input, and instead just defines the output file name so multiple builds don't overwrite each other. It can be left as default
      * DefaultMap and PartialMaps probably need full paths defined to the KLL files in my custom folder. NOPE I tried that and it didn't work. All 3 attempts in this didn't work: `DefaultMap="/controller/defaultLayerPartial controller/defaultLayerPartial k-type/release.1 stdFuncMap bradyjoKType/defaultLayerPartial"` Looks like I have to put my files under the `kll Installation Path: /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll` which can be done easily enough with a copy command
      * Copy command for editable files to buildable files: `cp /controller/kll/layouts/bradyjoKType/ /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/ -r` alongside a working DefaultMap: `DefaultMap="k-type/release.1 stdFuncMap  bradyjoKType/globalValues bradyjoKType/defaultLayerPartial"` got all my KLL changes into the firmware build, including global values for setting Gamma that I thought had to go in the BaseMap (but never worked)
      * Add the Partial maps for extra layers to the bash file: `PartialMaps[1]="bradyjoKType/layer1Partial"`
    * Making one key do a multi keypress combo looks like `U"W" : U"LGUI" + U"LALT" + U"LEFT";` or `U"S" : U"LCTRL" + U"DOWN";` to get W and S to do these things
    * Make one key do multiple sequential commands or keypresses with something like `U"ENTER" : U"H", U"E", U"L", U"L", U"L", U"L", U"L", U"L", U"O";`
      * Note that repeats of the same function will deduplicate, so the above sample would evaluate to `helo`
    * Based on the items in stdFuncMap.kll, 
      * `U"Function2"` will do a layerShift (enables as long as the key is pressed down)
      * `U"Lock1"` will do a layerLock (enables the layer until the key is pressed a second time) 
      * `U"Latch4"` will do a layerLatch (enables the layer for the next key press alone)
      * I also see in capabilities.kll that there's a `layerState(layer, state)` function
      * Finally there's `layerRotate(1)` to go forward and 0 to go back. 
  * Different animations depending on the active layer for indication of layer 2 versus default layer
    * Hints https://github.com/kiibohd/controller/issues/269
    * More Animation reference that had unique info on row and column layout, and setting individual pixels: https://kll.wiki/index.php?title=Animation_Reference
    * Can't do it based on active layer, but can switch active animations when a key is pressed by doing something like `U"LALT" : U"LGUI", A[miami_wave](start);` 
      * Doesn't work on the CAPS LOCK key, either because it's in conjunction with a layer shift thing (layer 1 only active when button is pressed) or because CAPS lock is special. I got the error `'HID(USBCode,default)"240"Function1' Invalid USB HID code, missing FuncMap layout`. That then made me think of a Github issue https://github.com/kiibohd/controller/issues/137 where function shifts didn't work if they weren't alone. The fix was to call layerShift(1) directly.  Now it does work on the CAPS LOCK key, but it doesn't go back to the other animation after the keypress
      * I found that I can activate the new animation when hitting the key, but can't return to the original when I depress. 
  
* Flashing to the keyboard manually
  * Hints can be taken from the desktop configurator flashing output
  * Hints on https://kiibohd.github.io/wiki/#/Flashing
  * Need `dfu-util` installed (already installed within docker though, so if flashing from docker there's no need to install)
  * Then just run (possibly with sudo) `dfu-util -D kiibohd.dfu.bin`
  * It looks like there is also a script at controller/Keyboards/linux-gnu.K-Type.gcc.ninja/load
  * When I called that, it said waiting for device
    * Oh that's because it's looking for `dfu-util -l | grep -q "Kiibohd DFU"`
    * I called it outside of Docker and got the following output: `Found DFU: [1c11:b007] ver=027f, devnum=59, cfg=1, intf=0, path="20-2", alt=0, name="Kiibohd DFU Secure", serial="91410000BB59001B002D000542334E45 - mk20dx256vlh7"`
  * I found that I can just run `dfu-util -D /Users/john/GitHub/controller/Keyboards/linux-gnu.K-Type.gcc.ninja/kiibohd.dfu.bin` from outside Docker in a new Terminal window
    * I can also flash the `kiibohd.secure.dfu.bin` if I want, although I'm not sure what I gain from doing that. I got my keyboard
  * During development, I basically ran these commands in this order:
    * `cp /controller/kll/layouts/bradyjoKType/ /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/ -r` to get the files into the build directory
    * `./k-type.bash ` from `controller/Keyboards` in the Docker container
    * `dfu-util -D /Users/john/GitHub/controller/Keyboards/linux-gnu.K-Type.gcc.ninja/kiibohd.dfu.bin` to flash to the keyboard after putting it in flash mode
  
* Coming back 5 months later to the already-set-up files

  * `cd /Users/john/GitHub/controller`

  * I used `git status` to see I had edited `**Keyboards/k-type.bash**`with the following lines. I have a copy of this file stored in my config repo as well: 

  * ```
    +# Don't forget to copy editable files to the KLL install directory
    +# cp /controller/kll/layouts/bradyjoKType/ /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/ -r
    +# ls -l /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/bradyjoKType/
    +
    ++DefaultMap="k-type/release.1 stdFuncMap  bradyjoKType/globalValues bradyjoKType/defaultLayerPartial"
    
    +PartialMaps[1]="bradyjoKType/layer1Partial"
    +PartialMaps[2]="bradyjoKType/layer2Partial"
    ```

  * Those lines added the KLL for my custom layouts and defined my custom animations and such. 

  * I did a `git stash` and then `git pull` and then `git stash apply` to hide the changes, pull the latest controller code from GitHub, and then reapply my changes on top of the latest code. 

  * Looks like there was a change to the default k type config files, firstly the base layer has been  renamed to` k-type/update.1`and now there are two partial layer maps `k-type/update.1.layer.1` and `k-type/update.1.layer.2`. However I couldn't find them in the repo so I think they get built/pulled when I start up docker, so that's next

  * Went to `controller/Dockerfiles` and ran `docker build -f Dockerfile.ubuntu -t controller.ubuntu .` which took only a couple seconds

  * then went up to the root controller directory and ran `docker run -it --rm -v "$(pwd):/controller" controller.ubuntu` to get in and `exit` (possibly call twice if using pipenv) to get out

  * Call `pipenv install` and then `pipenv shell` to get into pipenv

  * Ah, the new configuration files are at `/root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/k-type/update.1.kll`

    * update.1.kll is a new set of animations that includes Kira animations and is KLL version 0.5.7.11
    * update.1.layer.1.kll has customizable Nkey or 6key rollover, sleep and wakeup, different international keys, and some other stuff
    * update.1.layer.2.kll has animation control with f7 rotating through
    * release.1.kll (the one I used originally) still has the rainbow_wave animation defined within, but that's it

  * I don't really want/need the new files so I'll edit the `k-type.bash` build file to just use the files I have. However, I need to copy them into the Docker first. Copy command for editable files to buildable files: `cp /controller/kll/layouts/bradyjoKType/ /root/.local/share/virtualenvs/Keyboards-41-guJFC/lib/python3.6/site-packages/kll/layouts/ -r`

  * Then `./k-type.bash ` from `controller/Keyboards` in the Docker container

    * Lots of new notes on new KLL versions:

    * ```
       === 0.5.7.6 === 
      KLL now supports the replace:clearactive animation option.
      For example:
      A[my_animation] <= pfunc:interp, replace:clearactive, single;
      U"Q" : A[my_other_animation](replace:clearactive);
      
      The main use for clearactive is to clear any non-paused animations from the animation stack.
      While leaving any animations that are currently paused still on the stack.
      This means it's possible to save those animations to non-volatile memory (if your keyboard supports it).
      If you use replace:clear (as was the default before), when you save the current settings on your keyboard
      the keyboard won't understand the sequence of events in order to display the current LED colors.
      
       === 0.5.7.8 === 
      KLL now supports fade profile brightness settings.
      A fade profile is a selection of LEDs associated to one of 4 groups:
       - Keys
       - Underglow
       - Indicators
       - Active Layer
      Fade profiles are used to apply a configurable dimming and brightening, on top of whatever animation may be set.
      0.5.7.8 adds support for the fade_control() which (among other things), can adjust brightness per-fade profile.
      The most useful application of this would be to dim the LEDs under keys, while leaving underglow and indicator LEDs on.
      
      U"B" : fade_control(0, 3, 20); # Increment brightness on fade profile 0 (keys) by 20
      U"C" : fade_control(1, 4, 10); # Decrement brightness on fade profile 1 (underglow) by 10
      KLL_LED_FadeBrightness[2] = 150; # Set default brightness of indicators to 150
      
      For more details, review the KLL definition file:
      https://github.com/kiibohd/controller/blob/ae7bf0b711dfba5e77451cd2f269032f929a8657/Macro/PixelMap/capabilities.kll#L89
      
       === 0.5.7.11 === 
      KLL now supports inverting the Layer Fade profile.
      This means when a layer is active, only the keys in that layer will show the active animation.
      It is used by default on Gemini Dusk and Dawn.
      
      To use, add the following to your KLL file:
      KLL_LED_FadeActiveLayerInvert = 1;
      KLL_LED_FadeBrightness[0] = 255;
      KLL_LED_FadeBrightness[1] = 255;
      KLL_LED_FadeBrightness[2] = 255;
      KLL_LED_FadeBrightness[3] = 0; # Sets unused LEDs to off
      Layer[1] : fade_layer_highlight(1); # Activates layer fade when layer is triggered
      Layer[2] : fade_layer_highlight(2); # Activates layer fade when layer is triggered
      ```

    * 

  * Then `dfu-util -D /Users/john/GitHub/controller/Keyboards/linux-gnu.K-Type.gcc.ninja/kiibohd.dfu.bin` from outside docker to flash to the keyboard after putting it in flash mode

  * Animation was really wonky after flashing. Looks like it might just be the new way animations work and overwrite one another. At least all the regular bindings work, meaning it's not corruption and is just an animation change in behavior

    * Looks like the change in the animation declarations might have been `replace:all` changing to `replace:clear`

  * I had some issues with power: the keyboard would light up for a second and then go dark and not respond. Looking in the console, I saw cycling errors of `Keyboard - K-Type PixelMap USB::handlePowerDomainDidChangeTo not in power tree` and `258845.843879 AppleUSBCDCCompositeDevice@(null): AppleUSBHostCompositeDevice::ConfigureDevice: unable to set a configuration (0xe00002c0)`. Unplugging and replugging the device resulted in the same issue. This was all through an Apple USB C Multiport Adapter so I tried again with a direct USB C cable and saw the same console issues.

    * Not sure what in particular fixed it, but it seems to settle out after a while

  * Switching animations now results in the expected styles but the default animation when plugging in is still wonky. Likely due to using the default `rainbow_wave` function using `replace:all` in `release.1.kll`. That's the only thing in release.1.kll so I'm going to move rainbow_wave to my own global KLL file, fix its replace command, and then no longer reference release.1 in the k-type.bash file. Great, that fixed it! 