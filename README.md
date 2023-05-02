# boe-bot-roam-with-ir
Parallax Boe-Bot roam with IR Headlights to avoid obstacles.

Hardware

    Parallax Boe-Bot
    2 x Servo Motors [pins 12 and 13]
    2 x IR Detectors [pins 0 and 9]
    2 IR LEDs [pins 2 and 8]
    2 x Red LEDs [pins 1 and 10]
    Piezo Speaker [pin 4]
    
Code
' Roam with IR

' {$STAMP BS2}
' {$PBASIC 2.5}

DEBUG "Program Start..."

speed          VAR Byte ' speed + or - difference from 750
distance       VAR Word ' distance to move (mm - ish)
direction      VAR Nib  ' direction 3 = forward : 1 = backward
turn           VAR Byte ' turn rate 120 = no turn : 240 = hard right : 0 = hard left

repeat         VAR Byte ' repeat, used for various features
counter        VAR Word ' count, used for various features

irDetectLeft   VAR Bit  ' detected IR value
irDetectRight  VAR Bit  ' detected IR value

rand           VAR Word ' random number
randturn       VAR Byte ' random turn value

' start warning sound
PAUSE 500 : repeat = 4 : GOSUB alarm

rand = 11000

DO
  FREQOUT 8, 1, 38500
  irDetectLeft = IN9
  FREQOUT 2, 1, 38500
  irDetectRight = IN0

  IF (irDetectLeft = 0) AND (irDetectRight = 0) THEN ' Both IR's detect obstacle
    HIGH 1   ' Turn on Right LED
    HIGH 10  ' Turn on Left LED
    distance = 80 : speed = 30 : direction = 1 : turn = 120 : GOSUB move ' reverse a bit
    RANDOM rand  ' randomise amount of turn
    randturn = rand.LOWBYTE */ 30
    RANDOM rand  ' randomise left or right turn
    IF (rand.BIT0 = 0) THEN
      distance = 90 + randturn : speed = 30 : direction = 3 : turn = 0 : GOSUB move ' turn right a bit
    ELSE
      distance = 90 + randturn : speed = 30 : direction = 1 : turn = 0 : GOSUB move ' turn left a bit
    ENDIF
  ELSEIF (irDetectLeft = 0) THEN ' Left IR detects
    LOW 1    ' Turn off Right LED
    HIGH 10  ' Turn on Left LED
    RANDOM rand  ' randomise amount of turn
    randturn = rand.LOWBYTE */ 30
    distance = 20 + randturn : speed = 30 : direction = 3 : turn = 160 : GOSUB move ' turn right a bit
  ELSEIF (irDetectRight = 0) THEN ' Right IR detects
    HIGH 1   ' Turn on Right LED
    LOW 10   ' Turn off Left LED
    RANDOM rand  ' randomise amount of turn
    randturn = rand.LOWBYTE */ 30
    distance = 20 + randturn : speed = 30 : direction = 3 : turn = 80 : GOSUB move ' turn left a bit
  ELSE ' neither detect
    LOW 1    ' Turn off Right LED
    LOW 10   ' Turn off Left LED
    distance = 10 : speed = 30 : direction = 3 : turn = 120 : GOSUB move ' move forward a bit
  ENDIF

  rand = rand + 1

LOOP

END

' sub programs
alarm:
  FOR counter = 1 TO repeat
    FREQOUT 4, 100, 3000
    PAUSE 200
  NEXT
  RETURN

move:
  repeat = (41 * distance) / 130
  FOR counter = 1 TO repeat
    PULSOUT 13, 750 + ((speed + (turn - 120)) * (direction - 2))
    PULSOUT 12, 750 - ((speed - (turn - 120)) * (direction - 2))
    PAUSE 20
  NEXT
  RETURN
