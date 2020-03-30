# Open Audio Format

Open Audio Format is a simple 8-channel audio format that plays on Computronics beep/noise/sound cards.

## The Format

Files should be saved with the `.oaf` extension, and are made up of 16-byte segments. Each byte-pair in a segment determines a frequency to play. A value of 0 is silent, and any other value below 20 will be rounded to 20.

Segments determine notes to play for 0.10 seconds by default, though this can be changed by the user.

For example, the following byte array (assuming each number is 2 bytes) would play a C4 chord:
`262 330 393 00 00 00 00 00`
