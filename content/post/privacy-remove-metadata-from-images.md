+++
title = "Privacy: remove metadata from images"
draft = false
description = "Exif data"
topics = []
tags = ["infosec", "privacy", "image", "exif"]
date = "2015-11-28T00:00:00+08:00"

+++

Digital cameras (including smartphones) and computers (screenshots) embed technical metadata into image files. It serves great purpose to record image profiles but it could exposed some private information i.e., device profiles, GPS coordinates, etc and thus caused privacy issues.

Before share images online, strip clean the image EXIF (Exchangeable Image File) metadata. The `libimage-exiftool-perl` package contains the library and program to read and write meta information in multimedia files.

```
$ sudo apt-get install libimage-exiftool-perl
```

To view the exif metadata:
```
$ exiftool /path/to/image.jpg
```

A single file:
```
$ exiftool -all= /path/to/image.jpg
```

All under the target directory:
```
$ exiftool -all= /path/to/images/*.jpg
```

To set a property e.g., Copyright:
```
$ exiftool -Copyright=your-copyrighted-text image.jpg
```
