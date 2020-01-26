# Vidiot Tutorial
RULE #1: If you get something cool to happen, take a picture of the Vidiot patch so you can remember how you got there!!!

RULE #2: For all of the larger, stacked knobs, the top knob on the stack determines a base value, while the bottom knob determines the impact of modulations on the base value. Turn the bottom knob all the way to the left, and incoming modulations will not change the base value of the effect. Turn a bottom knob all the way to the right and modulations will highly impact the base value set by the top knob.

RULE #3: Signal typically flows according to the lines on the Vidiot, **except when you patch an output to an input**. Whatever output you patch in overrides the default modulation shown in the UI of the Vidiot. Outputs are indicated with a filled oval surrounding the patch point, while inputs are surrounded by a hollow oval.

0. Check to see if everything is turned on!  
	• Turn the monitor (tv) on. This will show you the Luma (brightness) output of the Vidiot. Most of this tutorial will focus on this output.  
	• Turn the projector on. The big power button should be blue when it's on.  
	• Turn the Vidiot on. If you're facing the Vidiot, the power is on the back right side of the unit.  
	• Turn on the video camera. Open the shutter at front of the camera, and unfold the preview screen from the left side of the camera if it's not already open.  

1. Check to see if the Vidiot is “zeroed”.
	• All knobs are turned completely counter clockwise (note that most knobs are stacked)  
	• All horizontal toggles are in their leftmost position  
	• All vertical toggles are in their topmost position  
	• The two white buttons near the top of the interface are “down” (pressed). This ensures proper vertical and horizontal synchronization.  
	• If you’re using the video monitor, make sure input “A” is selected; this is the button all the way to the right of the front monitor panel.  

2. OK, our first task is to get vertical lines on the screen. You’ll want to reference the Luma Processor description for, found on page 9.
	• We’ll be using the “Luma” output of the video synth, which only controls the brightness / contrast of the output… no color here! So we’e looking for black / white lines.  
	• The Luma processor is on the left side of the synth, you can find a guide to the interface on page 9 of the manual.
	• We want to modulate our signal using a waveform to create our horizontal lines. Inside of the Luma processor is an eight-position switch which enables us to select the source (waveform) for our modulation (widget #37 in the manual on page 9). Choose the seventh option, the vertical triangle (this is one before the furthest clockwise option. Make sure you don’t turn it all the way clockwise!!!  
	• OK, now we need to select the amount of modulation applied to both the brightness and the contact. We do these with the stacked knobs on the left of the Vidiot (40,41,54, and 55 in the manual on page 9). Turn the Brightness CV Level (40) all the way to the right. This means we will have as much control as possible over the Brightness modulation. Now turn the Brightness Control (41, the upper part of the stacked knob) to about the 9 o’clock position.   
	• Next turn both contrast knobs (54,55) completely clockwise. You should now have a fuzzy line on the screen! If you don’t make sure that the modulation source knob (37) is not completely clockwise, but one away (the vertical triangle) and check the positions of all the knobs.  

3. That was a lot of work to get a stripe on the screen, but now we can start having fun. We can control the frequency of our modulation using the Horizontal Frequency knob (25 on page 7) in the upper left corner of the Vidiot. Lots of lines now!  

4. Try making small tweaks to the Brightness (41 on page 9) and Contrast (55 on page knobs) to see how they change the image.  

5. Let’s add horizontal lines! We can use the other modulator (27 & 28 on page 7) to do this.  
	• Turn both (27 / 28) knobs to 9 o’clock.  
	• Our first patch! In the Vidiot, outputs (jacks with filled ovals around them) go to inputs (jacks with ovals that do **not** have a fill around them).  Let’s start by patching the Vertical Square Out (20 on page 7) to the brightness CV input (47 on page 9).  
	• Turn Vertical Frequency up (28).  
	• Try pulling the the cable end plugged into 20 into the other vertical shapes (Vertical Triangle Out, 32, Vertical Sine Out, 36, 3). What changes?  
	
6. Let’s animate. We can do this by telling our vertical (or horizontal) sync not to lock. The vertical sync button is 23 on page 7. Make sure it’s in the “up” position now. You might get some crazy animations happening immediately…. try adjusting the Vertical Frequency (28) to tame this.  

7. OK, let’s try one of the more complex shapes to modulate our brightness. Try taking the existing patch and use the circle or diamond (49 or 50 on page 7) to patch into the Brightness CV input (47). You can now use the Symmetry Knob (26 on page 7, the top center knob)  to change the relationship between the horizontal and vertical syncs.  

8. Let’s play with some toggles.  
	• We can invert the brightness and contrast modulations by flipping toggles 46 and 56 on page 9.  
	• We can invert the Luma signal altogether by changing the Luma Negative Mode to “On”.  
	• Make sure to keep playing with the various frequency / symmetry knobs while you experiment with these toggles, as well as the brightness and contrast control knobs.  

9. OK, time to bring the camera input in! Plug the camera into the input on the back of the Vidiot (remember, inputs have **hollow** ovals surrounding them). This input is two jacks to the right of the power jack. Make sure that the camera is turned on, the shutter at the front of the camera is open, and that the camera is plugged into power. DO NOT PLAY WITH THE SETTINGS ON THE CAMERA. We want to make sure it’s always set to feed its live input directly to its output, which is plugged into the Vidiot. Once the camera is plugged in, all modulations that you create are now being applied to the camera signal! For example, if you re-zero the board, and then turn the brightness, contrast, and detail controls to 12 o’clock, you will just see the “normal” video signal. Now you can start patching in modulations (like we did in steps 2–8) and see them applied to the video.  

10. We can also use the video signal as a “key”, basically a mask to apply another effect.  There are two effects to apply a key to in the Luma Processor. The first inverts the video signal, while the second “Solarizes” it, mimicking what happens if you re-expose photographic film to light during development of the film. You can also use the key to both invert and solarize the video signal. Once you turn on the solarize / invert toggles (68 and 69 on page 9)  you can set a luminance threshold for where the effects kick in using the Key 1 Threshold knob (77 on page 9), right next to the key source in jack (76 on page 9).  

11. You can also an audio source to drive modulations. Simply plug it into the appropriate spot (labeled with an ear) on the back of the Vidiot, and then patch it wherever you want to apply the modulation. In addition to the patching output, both Pattern generators can be set to use the audio input as their signal to feed brightness_contrast_detail or the RGB modifiers.  

12. Last but not least, play with the color controls! The same modulations that we applied to Luma can be applied to the RGB output of the Vidiot. You can also patch the Luma output from the back of the Vidiot into any input to use it as a modulation source. The Luma output can also be selected as the main modulation source for the colorizer by using control #39 on page 11, and selecting the eyeball icon. Read the (brief) section on the Colorizer in the manual to learn more, and have fun experimenting.
