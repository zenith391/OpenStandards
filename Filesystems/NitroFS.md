# NitroFS
NitroFS is a little, very simple and non-efficient filesystem. Originally made for OpenComputers Minecraft mod.  
Numbers MUST be Big Endian. The bytes and sectors are assumed to be 1-based.
If OSDI is present, the partition type must be `Nitro_FS`

## Sizes and Computations
- Default Logical Sector Size (SS): 512
- Default Logical Sector Offset (SO): 512
- Content Address (CA) -> Physical Address: CA * SS + SO

## Head structure
The head structure contains the signature and some informations for booting, reading/writing attributes, etc. The next entry written will be at Default Logical Sector Offset, this is made for the filesystem to be synced with sectors and allow very easy and fast reads/writes.

| Field          |                  Value                    |
|----------------|-------------------------------------------|
| Signature      | `NTRFS1`                                  |
| Attribute Type | FourCC: `FCH1` (Fuchas1) or `UNIX`        |
| Boot File      | CA pointing to a Lua file to boot         |
| Root Directory | CA pointing to root directory             |
- Total header size: 14 bytes

## Filesystem Structure

### Directory Entry
- D = Directory - 1 byte character encoded in US-ASCII
- `Unused` - 2 bytes - used for easier integration with file entry
- `Parent` - 2 bytes (logical sector number of parent, from 0 to 65535)
- `Name` - 32 bytes, the string is terminated with \0
- `Attributes` - 2 byte - System dependent
- `Number of childrens` - unsigned short
- `Childres` (contains a Content Address) - 2 bytes per children. If the CA is 0 then this child doesn't exists.
- Childrens entry max length: 473 (or 236 childrens per directory)
Used space of empty directory: 132

An equivalent as C struct would be:
```c
struct ChildrenEntry {
	unsigned short address;
};

struct DirectoryEntry {
	unsigned char type = 'D';
	unsigned short padding;
	unsigned short parent;
	unsigned char* name[32];
	unsigned char attributes;
	unsigned short childrens;
	ChildrenEntry* entries;
};
```

### File Entry
- F = File - 1 byte character encoded in US-ASCII
- Size: 2 bytes - size (in bytes) of this file
- Parent: 2 bytes - parent of this file (must be a directory)
- Name: 32 bytes
- Attributes - 2 bytes - Dependent on header's [OS] field.
- Fragment - 2 bytes - CA of the first fragment of the file, equals to 0 if file is empty

And as a C struct it would again give:
```c
struct FileEntry {
	unsigned char type = 'F';
	unsigned short size;
	unsigned short parent;
	unsigned char name[32];
	unsigned char attributes;
	unsigned short fragment;
};
```

### Fragment Entry
- R = fRagment - 1 byte character encoded in US-ASCII
- Next - 2 bytes - CA of the next fragment of the file, equals to 0 if it is last fragment
- Data - 509 bytes
This makes only a 3 byte loss for every 512 bytes on a file.

Finally, it's C structure would be:
```c
struct FragmentEntry {
	unsigned char type = 'R';
	unsigned short next;
	char data[509];
};
```

## Attributes
Attributes, as seen in the Head structure can be of different types.

`FCH1` (Fuchas1) is the 1st attribute system of Fuchas, currently effective for versions higher or equals to `0.5`.
It is based on Fuchas 0.5's attribute system which has 5 attributes that all can be combined: System, Read Only, Protected, Hidden and No Execute. They are respectively represented with the 1st (`00001`), 2nd (`00010`), 3rd (`00100`), 4th (`01000`) and 5th (`10000`) bit. Effectively only using 5 bits out of the 16 available bits.
