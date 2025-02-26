# FTE Tools README

# Preface

This file covers the tools include with the FTEQW source code.

FTEQW features a set of tools for compiling and editing QuackeC, textures, models, packages, and other formats associated with idTech.

### Contents

- [FTEQCC](#fteqcc)
- [Package Management](#package-management)
- [IQM Tool](#iqm-tool)
- [Image Tool](#image-tool)
- [Contact](#contact)
- [License](#license)

# FTEQCC

FTEQCC is used to compile 

Full FTEQCC documentation can be found here: https://fte.triptohell.info/moodles/fteqcc/README.html

### Package Management

The commandline version of fteqcc doubles up as a pak/pk3 creator/extractor:

	`fteqcc -l PACKAGENAME`

Lists the contents of a file on the stdout.

	`fteqcc -x PACKAGENAME`

Extracts all files from the named package to the working directory. You should normally use ../foo.pak in order to avoid overwriting files unintentionally.

	`fteqcc -x PACKAGENAME SUBDIR`

Extracts specific files from the named package.

	`fteqcc -0 SUBDIR`

Generates a `subdir.pak` file with the contents of that subdir. Unlike most pak generators, the pak will also be openable with any zip tool for users to easily view files (they shouldn't edit them though).

	`fteqcc -9 SUBDIR`

Generates a `subdir.pk3` file with the contents of that subdir. Contents will be compressed to reduce filesize, but won't work in vanilla quake.

	`fteqcc -z SUBDIR`

Generate a `subdir.pk3` dictionary file, and an (additional) `subdir.pXX` file (name generated sequentially). The dictionary file will be versioned with different servers potentially using different versions. This prevents re-downloading redundant copies of the same files for each different version. The spanned data files are not versioned, so these should only be generated by authoritive developers (ie: not same-filename forks).

Many engines support interpretting `foo.pk3dir` subdirs as proto-packages, which makes it easier to test packages without constant re-compression.

# IQM Tool

### About:

This is a commandline tool for creating iqm files from intermediate files exported from modelling programs.

### Possible Inputs:

- SMD
- GLTF
- GLB
- IQE
- MD%Mesh
- MD5Anim
- FBX
- OBJ

### Outputs:

- IQM / VVM

The default format. IQM supports skeletal animations and other features used by modern model formats.

**VVM** is a moniker for the backwards-compatible extensions FTE supports which are not supported by other engines/tools. Think of it as a "fork" of the IQM model format.

The **VVM** variant was created by Vera Visions hence the name, "Vera Visions Model".

- MDL

Quake's model format (idTech 2). It is vertex animation based and has many limitations, therefore it's recommend to use IQM / VVM unless you are targeting stock Quake.

**Note:** While Half-Life 1 & 2 also use the `.mdl` extension, it is a different format altogether _not_ supported by IQM Tool.

### Usage:

IQM Tool Can be used two ways:

	`iqm foo.cmd`

Uses a command script to decide which files to read, modifiers for each sequence or mesh, etc.

	`iqm foo.iqm foo.gltf`

Converts the gltf file to an iqm file. Animation rate will be guessed.

	`iqm foo.mdl foo.gltf`

Writes out a Quake MDL file. Framegroups will be used for each animation sequence.

### Command script:

```
	#COMMENT
	//COMMENT
```

Just a comment, ignored.

	`exec filename`

Invokes an external script (useful when you have multiple models with the same animations or so).

	`modelflags BITS`

Sets the output model's flags. Queryable in gamecode.

	`mesh MESHNAME ATTRIBUTELIST`

Overrides attributes of an imported mesh to use for the visible part of the model. Use the `import` command to actually import the geometry.

Attributes are:

	`contents BITSORNAMES`

Controls which traces may impact the imported surfaces. You should normally specify 0 if you are using hitboxes. Default is CONTENTS_BODY.

	`surfaceflags BITSORNAMES`

Misc info eg reported by tracelines.

	`body NUM`

The 'body' number which can be queried in gamecode (typically used to report eg headshots).

	`geomset SET ID`

Sets this mesh to only draw when geomset SET has been configured as ID.

	`lodrange MIN MAX`

This mesh will only be drawn when the screen coverage is within the specified range. Exact interpretation can vary according to engine cvars.

	`hitbox BODYNUM BONENAME MINS MAXS`

Creates a cuboid mesh around the named bone (as a box in the base pose). The mesh will be given CONTENTS_BODY such that it will be hit by hitmesh traces.
The bodynum arg can be queried via gamecode.

	`bone NAME ATTRIBUTELIST`

	`rename NEWNAME`

The bone read from input files will be written as NEWNAME in the output file (so gamecode can manipulate things consistently).

	`group GROUPNUM`

Bones can be reordered in the output according to their group numbers. Only the root bone of each group needs its group specified (children will inherit).
This simplies bone range operations in gamecode

```
	import FILENAME ATTRIBUTELIST
	model FILENAME ATTRIBUTELIST
	scene FILENAME ATTRIBUTELIST
	animation FILENAME ATTRIBUTELIST
```

Loads model AND animation data from an imported file.

File formats must match the given filename extensions.

Attributes are:

```
	Any attributes supported by the mesh command (such attributes here define the default values for imported meshes).
	name NEWNAME
```

The animation will be queryable to gamecode as the NEWNAME.

	`fps RATE`

Overrides the frame rate of the animation.

	`loop`

Specifies that the animation must loop.

	`clamp`

Specifies that the animation must NOT loop (animation will stop once it reaches the last pose).

	`unpack`

Generates single-pose animations, consistent with the way vanilla QC animates.

	`pack`

Generates multiple-pose animations, gamecode will be able to specify animation and time separately.

	`nomesh`

Do NOT generate meshes from this file.

	`noanim`

Do NOT generate animations from this file.

	`materialprefix PATH`

Prefixes the imported texture with an additional path (to avoid conflicts with other files).

```
	start FRAME
	end	FRAME
```

Limits inputed data to the specified poses.

	`scale SCALE`

Resizes the input data.

	`rotate PITCH ROLL YAW`

Rotates the imported data (quake is +x=forward, +y=left, +z=up).

	`translate X Y Z`

Moves the mesh+bones around a bit.

	`event [SEQUENCE:]POSE EVCODE EVSTRING`

Defines a model event at the specified timestamp.

```
	output FILENAME
	output_iqm FILENAME
	output_vvm FILENAME
	output_qmdl FILENAME
	output_md16 FILENAME
```

Specifies the file to be written. You may have one per supported output model format, each will contain roughly equivelent data (where supported).

# Image Tool

### About:

This is a tool for converting various image file types to more favourable ones.
This includes generating wad files, cubemap images, etc.

### Wad Examples:

	`imgtool --ext mip [--PIXFMT] [--resize W H] *.EXT`

Converts the input files to quake's miptex format. Input files will not be changed.

If compression was specified, the resulting miptex will contain an additional high-colour alternative.

If --resize is used, the paletted data will be resized without affecting the high-colour alternative.

Supported pixel formats:

	`rgba8,rgb8, rgb565,rgba5551,rgba4444, l8, bc1-7, etc1,etc2,etcp,etca, astc*x*, e5bgr9`

**NOTE:*** use of alternative formats requires a qbsp which does NOT strip this information. 
The vanilla qbsp will work fine for this purpose, but more advanced qbsp utils have a tendancy to strip the info (including ericw's, sadly).

	`imgtool -w WADFILE *.mip`

Packs the specified miptex files into the named wad file.

	`imgtool -x --ext EXT WADORBSP`

Extracts the textures from the specified wad or bsp file, saving them as EXT files.

### General Examples:
	`imgtool --help`

Shows a list of compressed pixel formats, and supported file formats.

	`imgtool -i FILE`

Shows info about the named file(s).

	`imgtool --ext ktx [--PIXFMT] *.EXT`
	`imgtool --ext dds [--PIXFMT] *.EXT`

Converts the input files to a hardware-friendly file format (with the specified pixel format). Input files will not be changed.

ktx supports all recognised hardware formats. dds is more limited (and eg doesn't support etc or astc).

hud artwork should generally be astc4x4 or bc7 for best quality.

models and walls can be of lower quality, astc6x6, etc2, or bc1 are good choices.

for hdr pixel formats, try astc4x4_hdr or bc6.
	
	`imgtool --ext png *.EXT`

Converts the input files to png format for viewing/editing. Input files will not be changed.

	`imgtool --cube [--PIXFMT] -o mysky.dds mysky_*.tga`

Converts 6 input files to a single cubemap file.

Inputs will be reordered according to quake's typical cubemap postfixes (and flipped as appropriate), otherwise be careful with wildcards.

	`imgtool --2darray [--PIXFMT] -o foo.ktx IDX0 IDX1 IDXN`

Converts the input textures into a 2d texture array. Input file order matters.
	
	`imgtool --3d [--PIXFMT] -o foo.ktx LAYER0 LAYER1 LAYERN`

Converts the input textures into a 3d texture. Input file order matters.

### Which pixel format to use:

### ASTC:

- The ldr-only profile is part of the gles3.2 spec (but likely to be emulated on nvidia).
- ASTC has a range of block sizes with each block being 16 bytes, thus larger block sizes yield greater compression.
- This format supports multiple planes, which reduces issues with multiple gradients in a block, which means it can cope with pixel art better than eg s3tc with larger block sizes.
- Bits are distributed according to usage per block, this means it can use more bits for the rgb channels where alpha is constant, giving it more useful bits than eg bc3.
- ASTC is able to use a second set of weights for any single channel (not just alpha), which makes it suitable for encoding normalmaps for instance.
- To encode ASTC pixel formats, you will need to install astcenc - https://github.com/ARM-software/astc-encoder/releases (or use astc-encoded source files).

### ETC2:

- Part of the gles3.0/gl4.3 spec
- ETC2 is a superset of ETC1, and has been somewhat obsoleted by ASTC.

### BC1/2/3:

- AKA s3tc, AKA dxt, available only via optional extensions, but is mandated by d3d 9_1 feature level.
- These formats all encode two 565 colours per `4*4` block with 2-bits per pixel for interpolation, which can result in discolouration.
- bc2 uses an additional 64bit block to encode 4bit alpha.
- bc3 also uses an additional block for alpha, but does so simiarly to its rgb values (two 8bit alpha values, with 3-way interpolation).
- To encode s3tc pixel formats, you will need to install libnvtt-bin - https://github.com/castano/nvidia-texture-tools (or export from image editors as dds or ktx files).
### BC4/5:
- AKA rgtc, part of the gl3.0 spec, or d3d10.
- These formats reuse bc3's 'alpha' compression to encode one or two channels respectively. BC1 can be handy for heightmaps/greyscale/etc, while BC5 can be useful for 2-channel normalmaps. They are not generally useful as eg wall textures.

### BC6/7:

- AKA bptc, part of the gl4.2, or d3d11.
- These formats support one or two planes per block, which means they can cope with pixel art quite a bit better than bc1/3.
- While they're the same size as bc3 (twice that of bc1), they have multiple modes per block that allow them to eg avoid wasting bits on unused alpha data.
- The difference is that BC7 encodes RGBA ldr data, while BC6 encodes RGB hdr data.
- To encode s3tc pixel formats, you will need to install (debian experimental) libnvtt-bin - https://github.com/castano/nvidia-texture-tools (or export from image editors as dds or ktx files).

### Too Long didn't read:

- Use ASTC if you're targetting modern mobile devices (astc6x6 for walls, astc4x4 for huds, or something).
- Use ETC2 if you're targetting older mobile users.
- Use BC1 for desktop model/wall textures, and bc3 for things with non-binary alpha channels.
- Use BC7 for desktop hud textures or other textures where BC1 does a terrible job.
- Use jpegs if you just want to get filesize down without caring about performance.
- If the user's hardware doesn't support the used formats then FTE can software-decode, so if you're expecting both mobile+desktop users with a single set of textures then favour mobile (desktop GPUs have better memory bandwidth).
	
# Contact

### Matrix

Join our Matrix Space here:

https://matrix.to/#/#fte:matrix.org

### IRC

We have a channel on [QuakeNet](https://www.quakenet.org):

**Server:** irc.quakenet.org

**Channel:** #fte

### Forums

**[Spike](https://forums.insideqc.com/memberlist.php?mode=viewprofile&u=26)** and **[eukara](https://forums.insideqc.com/memberlist.php?mode=viewprofile&u=949)** can be found on [insideqc.com](https://forums.insideqc.com/)

### Discord

There is a semi-official Discord server that **Spike** is in, however, we really recommend you use open, secure, privacy-respecting communication platforms like the ones listed above.

# License

Copyright (c) 2004-2024 FTE's team and its contributors
Quake source (c) 1999 id Software

FTEQW is supplied to you under the terms of the same license as the
original Quake sources, the GNU General Public License Version 2.
Please read the `LICENSE` file for details.

The latest source & binaries are always available at:

    [fteqw.org](https://fteqw.org)

    [fteqcc.org](https://fteqcc.org)