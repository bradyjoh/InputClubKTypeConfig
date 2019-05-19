# InputClubKTypeConfig


## Helpful Links
* Current ["manual"](https://github.com/kiibohd/controller/blob/master/Documentation/Keyboards/K-Type.md) with information like entering flash mode, configuring layout, installing the flash utilities on MacOS or Windows, and adding animations
  * Wants:
    * Default keyboard layout graphic
    * List of official firmware revisions and release notes
    * Method to reset to factory default
* Current client-side (downloadable) [configuration utility](https://github.com/kiibohd/configurator/releases) 
  * Note: When downloading the utility, the ZIP file contains a bunch of extra files but does not get a certificate popup when trying to execute (on Windows, at least).
  * Note: More feature support in the client-side utility than the web utility. For example, no lighting customization support in the web utility. 
  * Configurator Wants:
    * Change speed of animations using keyboard shortcuts (slower, faster)
    * Cycle through all enabled animations using keyboard shortcuts (forward, back, demo mode?)
    * Auto-start enables multiple animations at once?
    * Resume from sleep with the previous animation customizations (brightness, speed)



## Configuration Utility Modifications

There are a few things I want out of the keyboard, and the below list will outline the changes that I made to ensure the best possible experience from my K-Type. To pull in old changes, go to an old firmware build and copy and paste the JSON file into the configurator

### Modifications as of Configurator v0.3.1

* Change the CAPSLK key to be a temporary function modifier
  * This allows CAPSLK to function the same as the Fn key, by providing quick access to the f1 function layer. This makes left-handed key combos more easy to press. 
* Switch the LGUI, LALT and LCTRL keys to follow the Apple keyboard convention
  * This means that Command (LGUI) and Option (LALT) are switched around, so LGUI is directly next to the spacebar. 
  * Don't forget to switch the keycaps around as well!
* Add arrow keys to the f1 function layer on I, J, K and L
  * This provides a layout similar to 60% keyboards like the Vortex Pok3r, and allows for easy access to arrow keys without moving your hands
* Add VOL+, VOL-, and MUTE keys to the 3, 2, and 1 keys on the f1 function layer (this displaces the default LED controls)
* *TBD* Add key combo macros to the S and D keys to move between Google Chrome tabs
* *TBD* Add key combo macros to the W and E keys to move between fullscreen macOS windows (LCTRL + arrow)
* *TBD* Add shortcut keys in the F1-F8 area for LED control (on/off, bright/dim, play/pause, switch between animations following the YouTube guide [here](https://www.youtube.com/watch?v=Gv5QvRUT9gc))

## Other Notes

* FLASH is bound to ESC on the f1 function layer, so when you are ready to flash you can push f1 + ESC. The LEDs should go off and the board should go into FLASH mode for you to flash using the configuration utility
* LED Brightness can be controlled with the =+ and -_ keys on the f1 function layer
* When using the CAPSLK function modifier, the keys are easiest to press in decreasing ability: S, D, A, W, E, R, Q, 2, 3, 1, X, C, Z
* In the configurator under the Special section, CLEAR isn't actually a binding but rather clears the current assignment on the current layer. NONE will make a key binding but won't make it do anything
* To flash from the configuration utility, you just need to point to the downloaded dfu-util.exe file. The configuration utility should point to the proper file to flash. Put your keyboard into flash mode and wait for the "no keyboard is in flash mode" warning to disappear, and then click Flash. It should take 5-10 seconds to flash your keyboard and come back online.
* To continue editing after flashing, you can click the Home button and either look through the Layouts dropdown to find your firmware, or alternatively import the JSON again from the last firmware you flashed. On Windows, the generated folder should be in a path similar to `C:\Users\Username\AppData\Roaming\Kiibohd Configurator\firmware-cache\KType-Standard-5651a86aee45d2f6d7e52fa0a82661af` 
* I tried to change the animation and re-flash it and the board stayed dark after flashing and would not respond. I tried to reflash but it didn't recognize the keyboard in flash mode, so I had to use a SIM card removal tool to push the Flash button on the underside of the keyboard. 
* Default Animation notes as of Configuration Utility v0.3.1
  * Miami Wave is my favorite from a color scheme and style perspective, with a slanted line fading through the keyboard
  * Rainbow Wave is my second favorite, and while it looks nice it's a bit overwhelming in color. 
  * Fingerprint is almost worthless, with no fading or anything else. The key simply turns on in a color of your choosing when pressing down, and off immediately after. 
  * Fingerprint two tone is interesting, as it offers the same behavior as Fingerprint but leaves the key light on in a secondary color of your choosing indefinitely after pressing the key.