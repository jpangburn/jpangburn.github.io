---
layout: page
title: Solar Tracker Notes
permalink: /solartracker/
note: This file is unlinked on the web site as it's just for me and my dad to fix the solar tracker
---

# Notes about the solar tracker

## Physical Setup
Start time: 11:07  
End time: 16:00 (4 PM)  
Full west position: 910 ticks  
This gives about 10 ticks of padding to east and west before actuator reaches physical limits

## User Interface
The UI consists of a display screen that turns on when the device is reset or when the user pushes the Status button, and 3 buttons to control the tracker.
![UI](/assets/img/TrackerInterface.jpg)

### The Display Screen
The screen times out after a short while to save power.  If it's off, just push the Status button to turn it on.  It shows the following items (marked in the graphic):

- Time - This is the 24 hour local time.  It should be correct or the tracker position will be incorrect as well.
- Position indicator - Prefixed with a "P" the position indicates the current position in ticks for the linear actuator.  With the current actuator it should be between 0 (full east) and 910 (full west).  Note that it does not update while the unit is moving, but will update when moves are complete.
- Tracking indicator - Prefixed with a "T:", the tracking status will be either "Y" or "N".  When it shows "Y" the tracker will use the current time to position itself to point at the sun.  When it shows "N", automatic tracking is disabled.  This is the only safe way to work on the physical structure because it won't move automatically.
- Message - The message is context dependent on what you're doing with it.  When configuring, manually moving it around, disabling/enabling tracking, or if there's an error condition- the message will show you something (hopefully) relevant.

### The Buttons
The buttons marked in the graphic are the following:
- Status - If the screen is off, pushing the Status button wakes it up.  If the tracker is in tracking mode and it's moving, pushing this button will halt the tracker immediately and set tracking to disabled- useful if you forgot to disable it before doing something with the physical structure.  It also has various functions during setup described later.
- East - When the screen is active, pushing the East button will move the tracker east and disable tracking.  If you then hold the Status button and then release both buttons, the tracker will enter "auto move" mode and move to position 0 (the east limit).
- West - When the screen is active, pushing the West button will move the tracker west and disable tracking.  If you then hold the Status button and then release both buttons, the tracker will enter "auto move" mode and move to position 910 (the west limit).

## Setup After Reset
If the controller loses power (or you push the reset button on the Arduino, or plug in a laptop to reprogram or look at the Arduino IDE's Serial Monitor), it will not remember what position it was at.  If the screen is still on after reset, you can configure the position.  If the screen has already shut off, just hit the Status button to wake it up.

In this case, the display message should read "Setup Required" and the Position indicator will say "P UNK" for "unknown".  If the tracker is not at the east limit, use the East/West buttons to move it as precisely as you can to the east limit.  If you exceed the east limit by approximately 10 ticks, the actuator will start to twist instead of push and Very Bad Things<sup>TM</sup> will happen.  If you are shy of the east limit by more than approximately 10 ticks, then when the tracker goes full west it will exceed its physical limit.  This is untested, but the controller *should* notice the tracker isn't moving and enter an error state and stop moving.  I suggest avoiding an inadvertant test of this feature by being as precise as possible with the east limit.

Once positioned at the east limit, push the Status button and release it.  The display message should say "Press to zero".  Press and release the Status button.  The display message should say "Position zeroed" followed shortly by "Danger! Tracking!", the Position indicator should switch from "UNK" to "000", and the Tracking indicator should switch from "N" to "Y".  While it doesn't move quickly, this is your hint to make sure you haven't left any tools/clamps/children/etc. in a dangerous position.  If it's during daylight tracking hours, the tracker will move to the position it should be at for the current time of day.  If it's outside tracking hours, it will not move.

Since all this is a pain, try to only cut power (or cause a reset) when the tracker is sitting at the east limit.  This makes it very easy- you just hit the Status button again when the display is active, it will ask you to press again to zero, and you're done.  It will be tracking at that point with no fuss.

## Physical Structure Maintenance
Since sometimes you may need to work on the physical structure (maybe changing out a broken panel, putting wires in conduit, etc), it is helpful to have the tracker stop moving or manually move it to a different orientation.  While you can just unplug the controller and reset it once you're done, it's probably easier to put the controller into "tracking disabled" mode.  In this mode it will not move until you re-enable tracking mode.

To do this, hit the Status button to wake up the screen.  Hit the Status button again and the display message should say "Tracking disabled" and the Tracking indicator should say "T:N".  The Tracking indicator is there so when the message changes after you push other buttons, you'll still know it's not tracking and it's safe to work on the structure.  It will never go from "N" back to "Y" without you telling it to.

You can use the East/West buttons to manually move it to a different orientation if needed.  <font color="red">WARNING: you can exceed the movement limits by holding the East/West buttons!!!</font>  Use them with care.  While moving the Position indicator does not update, but it will update after you release the movement buttons.

The "auto move" feature is handy to move it full east or full west without sitting there holding the button for several minutes (see the [Buttons](#the-buttons) section for auto move instructions).  Auto move is also nice because you don't have to worry about exceeding the movement limits (unless the tracker has been improperly zeroed at somewhere other than the east limit of course).

To re-enable tracking, make sure the display is active and a position number is displayed in the Position indicator (if it says "P UNK" then the controller lost power during your maintenance and needs to be zeroed at the east limit, see [Setup After Reset](#setup-after-reset)) then hit the Status button again.  The display message should say "Press to enable" (if it says "Press to zero" instead- stop!  Follow the Setup After Reset directions instead.).  Pressing the button again will re-enable tracking.  It will say "Tracking enabled" followed shortly by "Danger! Tracking!" as the tracker is likely off of its correct position for the time of day (assuming it was disabled for more than a couple minutes).

## Error Situations
There are a few planned error situations.  The following errors will show up on the screen when you press the Status button and won't let you do anything else until they're resolved.

### RTC Lost Power
This is a pain two reasons:
- It means the backup battery (one of those flat watch batteries) in the ChronoDot (the round timekeeping device attached to the Arduino) is dead.  Look in the electronics closet- there's another ChronoDot there in one of those boxes that has a bunch of little components (wires/resistors/buttons/etc.).  In the segment with the ChronoDot is another battery.
- There is currently no way to set the clock using the buttons.  This is to discourage people setting the clock without replacing the battery.  The clock would only run with a dead backup battery until the next time the controller loses power so this would be bad.
To set the clock (after replacing the battery), load the Solar_Tracker_Positional.ino file in the Arduino IDE.  Locate the following line in the `void setup()` function:
```
//rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
```
Uncomment this line.  Connect your laptop with the USB cable to the controller, set the board/port in the Arduino IDE to "MEGA 2560" and whatever the correct USB port is (something like /dev/cu.usbserialxxx on my mac), then upload the program.  The display message on the LCD screen should show "Needs setup".  Do not unplug the computer or the next time the power fails, the clock will be set incorrectly and the tracker will likely be WAY off.  Comment that line out again, and upload the program again- this way when the battery dies years later you'll get this message again and know what's wrong with it.  Now disconnect the laptop and proceed to the [Setup After Reset](#setup-after-reset) section.

### Motor Sense Err
A motor sense error means when the tracker was moving, it didn't see any ticks from the position sensor for 3/4 of a second.  This is a clear error condition telling the controller that a sensor wire is broken/pulled/etc. and it can no longer determine what the actuator position is.  This is probably relatively easy to fix as it's likely a broken/disconnected wire.

To resolve this, disconnect power to the controller. Check the two green wires coming out of the controller- these are the sensor wires.
![UI](/assets/img/SensorWires.jpg)
Try the following steps in order to determine the problem:
1. Make sure they're still plugged in to the breadboard.  Refer to the photo above.  Make sure the capacitor is still securely plugged in and didn't get bumped too much by someone pushing buttons.
2. Make sure the brown wire going into the Arduino from above the capacitor hasn't gotten unplugged. If it has, it's pin 13 on the Arduino (between the gray and white wires in the picture).  The bare ground wire seems unlikely to have been removed on accident.
3. Unscrew the 4 screws on the actuator's access panel.  Using a tester, test for continuity on each green wire from where it connects on the breadboard to the wires coming from the sensor in the actuator.
4. If none of that solved it, here's some info to help you figure it out.  The sensor in the actuator is a reed switch.  As the motor moves, the wires from the sensor go from a closed to open circuit (just like a button).  About 3/4 of the time the circuit is closed and you should have continuity from one green wire on the breadboard to the other one.  The other 1/4 of the time, the circuit is open and there should not be continuity between the green wires- but there should be continuity from each green wire to its corresponding wire on the reed switch.  If this all seems correct, the remaining possibility is damage to the Arduino.  To determine if that's the case, you'll need to take this thing to your desk and write a test program to show when the green wires are connected and when they're not.  If this program fails, then that port is dead.  Try another port.  Or another Arduino.