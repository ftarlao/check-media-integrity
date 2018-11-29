# check-media-integrity
## Overview
*check-mi* is a Python 2.7 script that automatically checks the integrity of media files (pictures, video, audio).
You can check the integrity of a single file, or set of files in a folder and subfolders recursively, finally you can optionally output the list of bad files with their path and details in CSV format. 

The tool tests file integrity using common libraries (Pillow, ImageMagik, FFmpeg) and checking when they are effectively able to decode the media files.
Warning, **image, audio and video formats are very resilient to defects and damages** for this reason the tool cannot detect all the damaged files.

*check-mi* is able, with 100% confidence, to spot files that have broken header/metadata, truncated image files, and device i/o errors.

*check-mi* is, usually, not able to detect all the minor damages--e.g. small portion of media file overwritten with different values.
In detail, I have seen, with a small randomized experiment, that with a 5MB *jpeg* picture:
- Overwriting a portion (interval) of image file with **zeros**, you need interval **size = 1024KBytes** in order to get **50%** chance of detecting the damage.
- Overwriting a portion (interval) of image file with **different random values**, you obtain about **85%** detection ratio, for interval sizes ranging **from 4096bytes to 1024Kbytes**.

In the case you know ways to instruct Pillow, Wand and FFmpeg to be stricter when decoding, please tell me.

*check-mi* help:
```usage: check_mi.py [-h] [-c X] [-v] [-r] [-i] [-m] [-p] [-e] [-x E] P

Checks integrity of Media files (Images, Video, Audio).

positional arguments:
  P                     path to the file or folder

optional arguments:
  -h, --help            show this help message and exit
  -c X, --csv X         Save bad files details on csv file X
  -v, --version         show program's version number and exit
  -r, --recurse         Recurse subdirs
  -i, --disable-images  Ignore image files
  -m, --enable-media    Enable check for audio/video files
  -p, --disable-pdf     Ignore pdf files
  -e, --disable-extra   Ignore extra image extensions (psd, xcf,. and rare
                        ones)
  -x E, --err-detect E  Execute ffmpeg decoding with a specific err_detect
                        flag E, 'strict' is shortcut for
                        +crccheck+bitstream+buffer+explode

- Single file check ignores options -i,-m,-p,-e,-c

- With 'err_detect' option you can provide the 'strict' shortcut or the flags
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

Check a single file:

```check_mi.py ./test_folder/files/050807-124755t.jpg```

Check a folder:

```check_mi.py ./test_folder/files```

Check a folder, and subfolder recursiverly:

```check_mi.py -r ./test_folder/files```

Check a folder, and subfolder recursiverly and also check media (audio and video):

```check_mi.py -m -r ./test_folder/files```

Check a folder, and subfolder recursiverly and save bad files details to out.csv file:

```check_mi.py -r ./test_folder/files -c ./test_folder/output/out.csv```
## Required Modules

```ffmpeg-python==0.1.17
future==0.17.1
Pillow-SIMD==5.3.0.post0
PyPDF2==1.26.0
Wand==0.4.5
```
You can also use the standard Pillow-PIL module, but it is far slower that Pillow-SIMD.

In case you have only *libav* and not *ffmpeg* library/binaries in your Linux OS, you can fix this by creating a symbolic link *ffmpeg -> avconv* somewhere in your system search path.

## Test damage
test_damage.py is a funny experiment, it evaluates the probability of a random damage to be detected by this tool.
That is the outcome is a bit below expectations, the damage has to be in vital parts, better to be a random noise or to be a file truncation.
I think that the code is self explanatory.
