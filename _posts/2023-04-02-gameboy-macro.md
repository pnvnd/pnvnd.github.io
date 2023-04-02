---
layout: post
title: "Gameboy Macro without Soldering"
subtitle: "Hack for broken DS Lite"
background: '/img/gameboy-macro/gbmacro_05.jpg'
---

# Introduction
There's are some videos that show the "Gameboy Micro" working without soldering.  However, these videos do not show you clear screenshots of what it should look like or the positioning, just the end result with tape on top.  Here are the example videos:
1. [https://www.youtube.com/watch?v=GbOzY7UIec4](https://www.youtube.com/watch?v=GbOzY7UIec4)
2. [https://www.youtube.com/watch?v=_BEkI1gKwUg](https://www.youtube.com/watch?v=_BEkI1gKwUg)

## Why?
Converting a DS Lite into a "Gameboy Macro" is a way to save the device after the top screen stops functioning.  When the top screen is not detected, the bottom doesn't load, just a quick white flash, and the device turns off.  The proper way to fix this is to solder a 330 Ohm resistor (surface mount) to contact points labelled `LEDA2` and `LEDC2`.  This will allow you to use just the bottom half of the DS Lite as a Gameboy Advance (GBA) without producing more electronic waste.  This hack is a last-ditch-effort to save a DS Lite device WITHOUT using a solder, and without buying anything outside of regular household items.

## Instructions

1. Open up the DS Lite and remove the top half. There are plenty of tutorials on how to do this, so we'll skip right to the important part.

2. Take the rubber pad for the buttons and slice off one of the black contacts (not from the D-Pad!).  Reminder, that the Gameboy Advance only had the `D-PAD`, `START`, `SELECT`, `L+R` triggers, and `A+B` buttons.  We're essentially removing contact with one of the useless buttons `X/Y` to make this work.  In other tutorials, this pad is cut up with scissors.  Using an X-Acto knife (or similar) works.
    ![Screenshot](/img/gameboy-macro/gbmacro_01.jpg)

3. Place the black side of the contact faced down on the contact points labelled `LEDA2` and `LEDC2`.  Once in place, put some tape on it.  Kapton tape works great here.  If you don't have this handy, open up the top half of the DS Lite and use the yellow tape there.  It should still be nice and sticky if it hasn't been removed before.

    ![Screenshot](/img/gameboy-macro/gbmacro_02.jpg)

4. Place the bottom screen back in place and make sure to push in the ribbon cable all the way.  This is a good time to put in the battery to test if it turns on.  If you just get a white sceen, re-seat the ribbon cable and try again. 
    ![Screenshot](/img/gameboy-macro/gbmacro_03.jpg)

5. Once everything is working, put the shell back together and try again.
    ![Screenshot](/img/gameboy-macro/gbmacro_04.jpg)


# Remarks
Be extra careful when removing the wifi wires.  If you remove them too fast, it might hit the `EM10` filter and crack it.  While this still allows the device to function, you lose charging capability.
   ![Screenshot](/img/gameboy-macro/gbmacro_05.jpg)