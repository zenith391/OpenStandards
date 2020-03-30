# Open Audio Format

Open Audio Format is a simple 8-channel audio format that plays on Computronics beep/noise/sound cards.

## The Format

Files should be saved with the `.oaf` extension, and are made up of 8-byte segments. Each byte in a segment determines a frequency to play. A value of 0 is silent, and any other value below 20 will be rounded to 20.

Segments determine notes to play for 0.10 seconds by default, though this can be changed by the user.

For example, the following byte array would play a C4 chord:
`262 330 393 00 00 00 00 00`
