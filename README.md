# check-media-integrity

- Converted from Python2.7 to Python 3 (3.8).
- Quick fix for the error:
```
********WARNING*******************************************************
You are using Python Pillow PIL module and not the Pillow-SIMD module.
Pillow-SIMD is a 4x faster drop-in replacement of the base PIL module.
Uninstalling Pillow PIL and installing Pillow-SIMD is a good idea.
**********************************************************************
Files integrity check for: c:\LiClipseWorkspace\check-media-integrity\src\test_folder\
Parallel worker got unexpected error 'NoneType' object has no attribute 'error_detect'
```

## Overview
*check-mi* is a Python 2.7 script that automatically checks the integrity of media files (pictures, video, audio).
You can check the integrity of a single file, or set of files in a folder and subfolders recursively, finally you can optionally output the list of bad files with their path and details in CSV format. 

The tool tests file integrity using common libraries (Pillow, ImageMagik, FFmpeg) and checking when they are effectively able to decode the media files.
Warning, **image, audio and video formats are very resilient to defects and damages** for this reason the tool cannot detect all the damaged files.

*check-mi* is able, with 100% confidence, to spot files that have broken header/metadata, truncated image files (with *strict_level* >0), and device i/o errors.

*check-mi* is, usually, not able to detect all the minor damages--e.g. small portion of media file overwritten with different values.
In detail, I have tested *strict_level* 1 with a small randomized experiment, executed on a single 5MB *jpeg* picture:
- Overwriting a portion (interval) of image file with **zeros**, you need interval **size = 1024KBytes** in order to get **50%** chance of detecting the damage.
- Overwriting a portion (interval) of image file with **different random values**, you obtain about **85%** detection ratio, for interval sizes ranging **from 4096bytes to 1024Kbytes**.

In the case you know ways to instruct Pillow, Wand and FFmpeg to be stricter when decoding, please tell me.

*check-mi* help:
```
usage: check_mi.py [-h] [-c X] [-v] [-r] [-z Z] [-i] [-m] [-p] [-e] [-x E]
                   [-l L] [-t T] [-T K]
                   P

Checks integrity of Media files (Images, Video, Audio).

positional arguments:
  P                     path to the file or folder

optional arguments:
  -h, --help            show this help message and exit
  -c X, --csv X         Save bad files details on csv file X
  -v, --version         show program's version number and exit
  -r, --recurse         Recurse subdirs
  -z Z, --enable_zero_detect Z
                        Detects when files contains a byte sequence of at
                        least Z equal bytes. This case is quite common, for
                        jpeg format too, you need to set high Z values
  -i, --disable-images  Ignore image files
  -m, --enable-media    Enable check for audio/video files
  -p, --disable-pdf     Ignore pdf files
  -e, --disable-extra   Ignore extra image extensions (psd, xcf,. and rare
                        ones)
  -x E, --err-detect E  Execute ffmpeg decoding with a specific err_detect
                        flag E, 'strict' is shortcut for
                        +crccheck+bitstream+buffer+explode
  -l L, --strict_level L
                        Uses different apporach for checking images depending
                        on L integer value. Accepted values 0,1 (default),2: 0
                        ImageMagick idenitfy, 1 Pillow library+ImageMagick, 2
                        applies both 0+1 checks
  -t T, --threads T     number of parallel threads used for speedup, default
                        is one. Single file execution does not take advantage
                        of the thread option
  -T K, --timeout K     Number of seconds to wait for new performed checks in
                        queue, default is 120 sec, you need to raise the
                        default when working with video files (usually) bigger
                        than few GBytes

- Single file check ignores options -i,-m,-p,-e,-c,-t

- strict_level: level 0 execution may be faster than level 1 and level 2 is
the slowest one. 0 have low recall and high precision, 1 has higher recall, 2
has the highest recall but could have more false positives

- With 'err_detect' option you can provide the strict shortcut or the flags
supported by ffmpeg, e.g.: crccheck, bitstream, buffer, explode, or their
combination, e.g., +buffer+bitstream

- Supported image formats/extensions: ['jpg', 'jpeg', 'jpe', 'png', 'bmp',
'gif', 'pcd', 'tif', 'tiff', 'j2k', 'j2p', 'j2x', 'webp']

- Supported image EXTRA formats/extensions:['eps', 'ico', 'im', 'pcx', 'ppm',
'sgi', 'spider', 'xbm', 'tga', 'psd', 'xcf']

- Supported audio/video extensions: ['avi', 'mp4', 'mov', 'mpeg', 'mpg',
'm2p', 'mkv', '3gp', 'ogg', 'flv', 'f4v', 'f4p', 'f4a', 'f4b', 'mp3', 'mp2']

- Output CSV file, has the header raw, and one line for each bad file,
providing: file name, error message, file size
```
## Examples

Check a single file (remind, strict_level default is 1 and number of threads default is 1):

```check_mi.py ./test_folder/files/050807-124755t.jpg```

Check a folder (folder ```files```, contains the media files):

```check_mi.py ./test_folder/files```

Check a folder, and subfolder recursiverly:

```check_mi.py -r ./test_folder/files```

Check a folder, and subfolder recursiverly and also check media (audio and video):

```check_mi.py -m -r ./test_folder/files```

Check a folder, and subfolder recursiverly and save bad files details to out.csv file:

```check_mi.py -r ./test_folder/files -c ./test_folder/output/out.csv```

Check a folder, and subfolder recursiverly using 4 processes/threads, and save bad files details to out.csv file:

```check_mi.py -r -t 4 ./test_folder/files -c ./test_folder/output/out.csv```
## Required Modules

```ffmpeg-python==0.1.17
future==0.17.1
Pillow-SIMD==5.3.0.post0
PyPDF2==1.26.0
Wand==0.4.5
```
You can also use the standard Pillow-PIL module, but it is far slower than Pillow-SIMD.

In case you have only *libav* and not *ffmpeg* library/binaries in your Linux OS, you can fix this by creating a symbolic link *ffmpeg -> full/path/to/avconv* somewhere in your system search path.

## Test damage
test_damage.py is a funny experiment, it evaluates the probability of a random damage to be detected by this tool.
That is the outcome is a bit below expectations, the damage has to be in vital parts, better to be a random noise or to be a file truncation.
I think that the code is self explanatory.
