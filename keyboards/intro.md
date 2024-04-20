# I like QMK

[QMK](https://qmk.fm/) is an opensource firmware that allows anyone the ability to customize the functionality of a given keyboard that supports it.

A brief list of keyboards I have used with QMK:

* [ErgoDoxEZ](https://www.ergodox.io/)
* [crkbd](https://github.com/foostan/crkbd)
* [reviung41](https://github.com/gtips/reviung)
* [Moonlander](https://github.com/gtips/reviung)
* [Mini42](https://docs.controller.works/mini42/)


There are *literally* a shit ton of keyboards that can use this. 

Another firmware that is actively being developed is [ZMK](https://zmk.dev/) which allows one to create wireless keyboard firmware. Adds another layer of complexity, but will be cool to see that come along. 

## Resources

I spent a long time tinkering with this stuff. I spent a while obsessing over small things, looking at how to make comfortable layouts, making cool features that I never use, flashing my keyboard more than actually doing work. A normal experience I imagine.

To help speed one up I am gonna quickly dump my resources and notes that I have gathered while doing this.

#### Layouts:

* [Miryoku Layout](https://github.com/manna-harbour/miryoku)
	* literally the most talked about layout 
* [KeymapDB](https://keymapdb.com/)
	* glorious fancy layouts

### Home Row Mods (HRM)

Ever feel your pinky burn after a long day of copy pasting code to get your program to work? How about your hand is aching cause your keyboard is so damn small? Or maybe it is the opposite, the keyboard is too big and you are leaning a bit too much on the shift key as the day goes on.

Well Home Row Mods can help by having a normal key serve a dual role. Think of a QWERTY keyboard, the home row is as follows (left to right): **asdfjkl;**. Those are the *proper* resting positions for your hands. What if those keys could be shift or even **control**! Well they can, with HRM mods anything is possible. That **f** key is looking mighty fine as a shift key. So lets add the functionality with QMK. It would look like this:

```c
LSFT_T(KC_F)
```

So now when you tap the F key you still get F, but when you hold the F key you now get SHIFT instead. Amazing.

**Who HRM is for**: If you program, use the cli, use a window manager, or do anything that is more development oriented you will like this.

**Who HRM is not for**: If you game more than you actually use a keyboard for technical work, you will have a bad time. NOTE: You can just create a layer that doesn't have them active. So not a huge deal, but so many people complain about this on forums that I feel the disclaimer is warranted.

#### Useful links

* [Preconditions Blog](https://precondition.github.io/)
	* Gives you the rundown of home row mods

##### Alternatives to Home Row Mods
* [Callum Mods](https://github.com/callum-oakley/qmk_firmware/tree/master/users/callum)
	* Store mods on another layer. Requires one to switch to layer, hold key, and relese layer
	* Can use One Shot Mods to make this a bit nicer. Switch to layer -> Press One Shot Mod -> Release Layer
	* Callum has a different implementation for modifiers than base QMK. Has some issues
 * [Combo Mods](https://jasoncarloscox.com/blog/combo-mods/)
	 * Uses Combos to activate modifiers
	 * Can be finnicky due to timing, keyboard layout, and how easy it is to press multiple keys at once

#### My thoughts

I like Home Row Mods. They make things convenient to access. I have also gone down the rabbit hole of space optimization and I have learned that you just gotta say fuck it. Many users will mention that they don't want to duplicate keys, they want to be a purist. 

The reality of the situation is that duplication and redundancy help. Especially once you go smaller. Having modifiers in one location will inevitable lead you to twisting your hands in strange ways. A quick example can be described as follows:

> You design your traditional QWERTY keyboard with Home Row Mods on ASDF keys where A serves as the Windows/Gui/Mac key and A.

To hit GUI+A you would now need to duplicate the GUI key regardless. Most people put it on the other side, requiring the second hand a la Miryoku's layout. I do this, but what about those days where you just feel tired? You know what I mean, that mid day slump roles around, your hand is mighty heavy on that mouse, and you know, god dammit, I don't want to stop leaning on my chair!

That is why I just duplicate it. If you know keys are needed for frequent combinations, shortcuts, etc. Then duplication of the key is not a waste if it saves you time, effort, and comfort.

Really the biggest benefit with home row mods is that I don't have to hold keys at awkward angles when I am feeling lazy. I had a professor who swore by emacs and that shit fucked up my hand so hard. Partially my fault, partially the fact that my thinkpad fought me every. single. keypress. 


Anyways.... Home row mods are great. Here is a quick rundown of things you will add to your config to mess with and use them:

* config.h

```c
// Setup tapping term
// This is the time (ms) for the processor to distinguish between press or hold
#define TAPPING_TERM 200
#define TAPPING_TERM_PER_KEY

// Interrupt is the ability for the processor to distinguish between keypresses
// Without this you might get more triggers than not due to Interrupts not being caught
#define IGNORE_MOD_TAP_INTERRUPT

// Use Permissive Hold but allow me to control it per Key
// Permissive hold is another option which makes registering of mod taps a bit more aggressive
// Specifically if I hold a key to make Shift, press another key, and release the Shift key what happens to the pressed key?
// If you don't have Permissive Hold for Shift then the pressed key will not be shifted
// With this I get the shifted key without having to hold down keys for the entire action
#define PERMISSIVE_HOLD_PER_KEY

```

* keymap.c

```c
#ifdef TAPPING_TERM_PER_KEY
uint16_t get_tapping_term(uint16_t keycode, keyrecord_t *record) {
    switch (keycode) {
        // tap dance
#    ifdef TAP_DANCE_ENABLE
        // set tap dance keys to use a shorter tap term
        case QK_TAP_DANCE ... QK_TAP_DANCE_MAX:
            return TAPPING_TERM - 10;
#    endif
        case BSPC:
        case ENT:
            return TAPPING_TERM - 10;
        // mods on my pinky keys need to be faster since I exclude them from permissive hold
        case MOD_A:
        case MOD_O:
            return TAPPING_TERM - 10;
        // Increase time for one shot mods to prevent them from being seen as Hold actions
        case OSMG:
        case OSMA:
        case OSMC:
        case OSMS:
            return TAPPING_TERM + 30;

        default:
            return TAPPING_TERM;
    }
}
#endif

#ifdef PERMISSIVE_HOLD_PER_KEY
// makes it a little difficult to manage with layer taps
bool get_permissive_hold(uint16_t keycode, keyrecord_t *record) {
    switch (keycode) {

        case BSPC: // don't want hold to popup when I try to delete stuff
        // home row
        case MOD_A:
        case MOD_O:
        // bottom row
        case MOD_C:
        case MOD_COMM:
			// don't apply permissive hold for those above
            return false;
        default:
            return true;
    }
}
#endif
```

### Forget User Spaces

I know you can make a userspace. I am just lazy and refuse to abide by their rules. 

To make modifications to your current keymap which enables the use of source files you can do the following:

- Create a lib folder
	- Store functionality in the lib folder that is easily made independant
	- layers for instance
	- layer_lock which I grabbed from another user
	- keycodes

To then include them in the compilation you make only a few edits.

1. In the keymap.c file you can add them like so:

```c 
#include "quantum.h"
#include QMK_KEYBOARD_H
#include "lib/keycodes.h"

#include <stdio.h>
// #include "lib/oneshot.h"
#ifdef LAYER_LOCK_ENABLED
#    include "lib/layer_lock.h"
#endif
#ifdef TAP_DANCE_ENABLE
#    include "lib/tapdance.h"
#endif
#ifdef LEADER_ENABLE
LEADER_EXTERNS();
#endif
#ifdef COMBO_ENABLE
#    include "g/keymap_combo.h"
#endif

```

2. In rules.mk you add any c files (not header files) to the source list. This is important as the headers only serve the references to functions

```c
TAP_DANCE_ENABLE = yes 
SRC += lib/tapdance.c
```


Anyways, that isn't super helpful or anything. But saves me time and prevents me from dealing with user space.