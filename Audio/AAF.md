## Adaptive Audio Format v1.1
This is an audio format for OpenComputers. All numbers are little endian.

Here are all common data you will find:
```
Signature (US-ASCII): <DC3>AAF<ETX>
Wave Type: SQUARE (0) | SINE (1) | TRIANGLE (2) | SAWTOOTH (3)
```

### Capabilities
Thoses are the capabilities required on the sound card by the current file.
```
- [WAVE TYPES] - Act as flag, 1 = SQUARE, 2 = SINE, 4 = TRIANGLE and 8 = SAWTOOTH
- [FLAGS] - Same, a flag for miscelanous things, 1 = ADSR, 2 = VOLUME
- [CHANNELS] - 1 byte unsigned number
```

### Header
```
- [SIGNATURE]
- [CAPABILITIES] - see Capabilities above
- [AUDIO FRAGMENTS] - appended to each other
```

### Audio fragment
Total:
Channel data is organized starting from 0 up into channel number - 1
(so if 1 channel it's from 0 to 0, so one channel)
This also means that for 1 channel it should be the Channel Data.
```
- [PER CHANNEL DATA] x (channel number)
```
Channel Data:
```
- [FREQUENCY] - unsigned short, in Hertz
```
**[FREQUENCY] acts as action type if it is below 3Hz!**

ASDR Packet (if action type is 1): 
```
- [ATTACK] - unsigned short, duration in milliseconds
- [DELAY] - unsigned short, duration in milliseconds
- [SUSTAIN] - unsigned short, volume level (like [VOLUME])
- [RELEASE] - unsigned short, duration in milliseconds
```
Volume packet (if action type is 2):
```
- [VOLUME] - unsigned byte, decimal obtained by dividing by 255 (giving 255 volume levels)
```
Type packet (if action type is 3):
```
- [WAVE TYPE] - unsigned byte, described above. Must be only set at ONE flag
```
The packet always ends with (even if there's no action and a higher than 3Hz frequency is used):
```
- [DURATION] - unsigned short, duration in milliseconds
```

Q: Why wave type, ADSR and volume changes are appended with a wave body?  
A: Because those changes apply directly to a wave, and thus can be executed just before it, so with that, this was done to allow packing and free a few bytes. So if it was separate, for ADSR it would size 8 bytes, where with current method it would use 7 bytes, effectively saving 1 byte PER CHANGE!
