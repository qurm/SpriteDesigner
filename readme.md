
# DESIGN Notes

DESIGN is a BBC BASIC Sprite editor used to help develop the Bird Strike game. It was written in 1983.
It works in `MODE 2`, and allows sprites, graphics to be designed and edited direct on the screen.
The data can then be saved back into a reserved memory location, say `&1900 - &1E00`.
This can then be saved to disk with a `*SAVE 1900 1E00`

It was developed to work in limited RAM space, as the memory map below shows.  This is why the code is very compressed, with no comments.

It is published for historic interest as an example of the tools available for games designers in the early 80s.  There was no GUI or mouse in use at that time.  It may still be of use for anyone working in retro games on the BBC Micro.

A BBC Micro is mnot required - this can be run on the Beebem emulator https://github.com/stardot/beebem-windows

Use with caution, as this will allow you to display and edit any part of the BBC memory, including the BASIC program space.  Make sure you understand the memory map!!

## Memory Map
Example from Bird Strike - others configurations are possible.
```
&1900 - &1E00           $.X (pigeon/number/scenery/player sprites)
&1E00 - &21DF           $.DESIGN BASIC
&21EE - &3000           $.G (cloud/enemy sprites)
&3000 - &7FFF           HIMEM, Mode 2 memory		
```
Example load on clean memory, Mode 7
```
PAGE	6400	&1900		(default)
LOMEM	8327	&2087
HIMEM	31744	&7C00
```
Example load MODE 2, with memory reserved for sprite data.
SET `PAGE = &1E00` first
```
PAGE    7680    &1E00
LOMEM	9608	&2588
HIMEM	9472    &3000
```
## Load and Run
Depends on where the sprite data is located.
Here program loads at `&1E00` to allow space for sprite data at `&1900`
```
PAGE = &1E00
*LOAD X 1900      
LOAD "DESIGN"
RUN
Q to Quit
```

## Using the program
In the main view, the source range of memory is copied to the MODE 2 screen memory to display at the top of the screen on start-up. The range is defined as `B%=&1900:E%=&1E00` 
In the main view, you can select a range, or sprite, from the source range of memory, and the address of the selection is shown in the upper centre.

Use these keys in the main view
```
  Move back/forward 128 / &80       <,    >.
  Move forward 64 / &40             N     
  Move back/forward 8 / &08         B    SPACE
  Quit, exit to BASIC               Q
  ```
The memory location display updates as you move.
These numbers were based on a large sprite of 128 bytes.  The exact sprite size does not matter, as the program edits pixel-by-pixel, and can deal with any size of sprite.

Copy/Dump the sprite range from the source range to the screen memory.
Use this after an edit has been saved, or after changing source location.
```  
Dump  D
```

Set the source range here:
```
  800 DEFPROCDUMP
  805 B%=&1900:E%=&1E00
```
Select the range/sprite for edit.
This will enlarge the range for editing.
```
  Edit / Left        E
  Review / Right     R 
  Activate    A   (edit Left)
  ```
Editing mode will show up as a flashing 1 pixel cursor in the enlarged sprite (top left initially).

There is a palette display below the zoom areas
Move the indicator below it to select current colour.
```
  Move back/forward              <,    >.
```
Use these keys in the pixel edit
```
  Move left/right 1 pixel       LEFT, RIGHT
  Move up/down 1 pixel          UP, DOWN     
  Movement will wrap around
  Set pixel to current colour   COPY  (END in Beebem)
  Quit Editing / Save           Q
    Confirm Save                Y (upper-case)
    Confirm Save with location  @   (prompts for location)
```

## BASIC code sections with comments
More to do here ....

Shows the main loop with INKEY values:
```
  100 REPEAT:PROCSHAPE
  115   IFINKEY(-104) L%=L%+128		,<
  116   IFINKEY(-86) L%=L%+64		N
  117   IFINKEY(-103) L%=L%-128		.>
  118   IFINKEY(-101) L%=L%-8		B
  120   IFINKEY(-99) L%=L%+8		SPACE
  121   IFINKEY(-51)PROCDUMP		D
  125   IFINKEY(-35)PROCENLARGE(&5320)	E
  126   IFINKEY(-52)PROCENLARGE(&5460)	R
  128   IFINKEY(-66)GOTO280		A
  150 UNTILINKEY(-17)			Q	QUIT
```

Shows the zoomed edit loop with INKEY values:
```
  290 REPEAT K=0
  300   REPEAT
  302     FORY=0TO40:NEXT
  304     IFINKEY(-26) X%=X%-1:K=-1	  LEFT
  306     IFINKEY(-58) Y%=Y%-1:K=-1	  UP
  308     IFINKEY(-122) X%=X%+1:K=-1    RIGHT
  310     IFINKEY(-42) Y%=Y%+1:K=-1     DOWN
  312     IFINKEY(-17) K=-1	        Q QUIT
  330     X%=X%AND15:Y%=Y%AND7
  335     A%=16*X%+640*(Y%DIV2)+4*(Y%AND1)
  337     PICOL%=A%?&5320
  340     PROCFILLPIX(&5320+A%,PICOL%EOR255)
  345     FORY=0TO40:NEXT
  347     PROCSELECT
  350     PROCFILLPIX(&5320+A%,PICOL%)
  380   UNTILK
  400 UNTILINKEY(-17)			Q quit
```
  save - EITHER 64, &40 @ OR 89, &59 Y

  COPY key, END ON PC WRITES VALUES.