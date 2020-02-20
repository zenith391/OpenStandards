# Open Graphics Format (OGF)
Note: any text between `<` and `>` is not literal text (of course) and is just are just placeholders for the actual values.

OGF can contain multiple images, if the animated flag is set, it should be interpreted as an animation, otherwise if there are
multiple images that aren't animated, they should be just treated as a list of image, if this doesn't make sense in the current usage, the first image only should be used.

**THIS IS A DRAFT AND ISN'T FINISHED!**

## Types
Any `Text` type is a UTF-8 string starting with a 2 bytes unsigned integer (0-65535) for its length, any optional `Text` is counted as not present if the length is equals to 0. 

A `Boolean` type is a byte equals to 0 for false or 1 for true. If the boolean value is any other value, the resulting boolean is false.

## Flags
If least-significant bit is equals to 1 (flags & 0b00000001 == 0b00000001), the file is an animated image. The images are in
increasing order dependending on entry ID of Image Data entries.

## Header
```
<Signature> - ASCII "OGFIMAG" followed by ASCII End Of Text (ETX, byte equals to 3)
<Version> - Unsigned byte (currently equals to 1)
<Flags> - Byte, see above
<Entries Length> - Unsigned byte (0-255)
<Entries>
```

## Entries
Any entry start with a byte for its type and a 4 bytes integer for their length.
So every entry start with `<Type> - Unsigned byte (0-255)` and `<Length> - Unsigned 4 bytes integer`.
Entries might come in any order, but if an entry (ex: image data) have a dependency over another one (ex: rgb palette), the dependency must come before.
Notice that unless marked so, any entry can be found multiple times in a single file. Allowing for multiple images in a single file.

### Entry IDs
An entry ID (and not type) is the place of the offset, so first entry is at ID 0, second at ID 1, ..., tenth at ID 9, etc, the entry id is stored on an unsigned byte thus limiting the number of entries to 256 (0-255). An entry cannot reference the entry ID 255 as it is used for saying the entry ID is unspecified.

### Metadata
This entry **MUST** be on every file!
The entry type is 0.
```
<Frame Time> - Integer, for animated images only, it is the time between each frames in milliseconds, equals to 0 if image isn't animated.
<Author> - Text, the name of the author of the picture, optional
<Extended Name> - Text, the extended name of the picture, optional
<Location> - Text, the location where the picture was made, can be in X-Y-Z format or just a name, optional
<Custom> - Text, optional, for custom/unofficial values, this must be in a key=pair format where each entry is ended by ; (ex: mycustom=thing;thisisnot=pork;abc=123)
```

### RGB Palette
The entry type is 1 and the entry is optional. The bit depth is determined from the image linking to this palette.
So if a 8-bit color depth image is linking to this palette, this should be interpreted as 8-bit palette.
12-bit and 16-bit bit depth are 8-bit color depth (as they all use 8-bit for color, but just have additional bits for alpha), which mean a 8-bit palette is suited for those bit depths (8-bit, 12-bit and 16-bit)

This also means images should never be a different bit depth than the palette was made word.
```
<Values>
```
Yep it just contain a Values field. This field corresponds to a concatenation of the 24-bit RGB values. Meaning the entry is 768 bits long for a 8-bit/12-bit/16-bit bit depth. What this means is that for example, 1-bit have 2 RGB colors, 2-bit have 4, 4-bit have 16, 8-bit 256, and 24-bit doesn't use any palette.

### Image Data
The entry type is 2. There **MUST** be atleast one image data in a file.

```
<Width> - Unsigned 2 bytes integer (0-65535)
<Height> - Unsigned 2 bytes integer (0-65535)
<Bit Depth> - Unsigned byte.
<Pixels> - A list of pixel structs (format depending on bit depth) appended in XY order from top-left.
```
It always ends with:
```
<RGB Palette> - entry ID (see above)
```

Pixel struct:
```
<Background> - format depending on bit depth
<Foreground> - format depending on bit depth
<Character> - UTF-8 character that MUST FIT in 1 byte
```
#### Bit Depths

Byte Value | Bit Depth
---------- | ---------
0 | 1-bit
1 | 2-bit
2 | 4-bit
3 | 8-bit
4 | 24-bit
5 | 32-bit (24-bit + alpha 8-bit)
6 | 16-bit (8-bit + alpha 8-bit)
7 | 12-bit (8-bit + alpha 4-bit)

The 12-bit is technically the hardest to implement due to reading being padded on 8-bit usually. So this would need to separate it into 2 4-bit values and read them independently. 

1-bit, 4-bit and 8-bit colors are changed to 24-bit RGB values via the RGB palette. If the reference is null (equals to 255) then the default [OC palette is used](https://ocdoc.cil.li/_media/api:oc-256-color.png). 24-bit RGB is interpreted as it and the alpha values too.

If the bit depth is 12-bit and the image ends not being padded on 8-bit, 4-bit padding must be added to fit. The padding can be anything as it is ignored.
