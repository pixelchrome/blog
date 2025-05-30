---
title: Converting image files with macOS command line tool sips

date:
  created: 2024-12-30

linktitle: Converting image files with macOS command line tool sips
slug: converting-images-with-sips

tags:
  - macOS
  - CLI

authors:
- harry
---
## Simple Image Conversion from the Command Line

```sh
sips -s format [image type] [file name] --out [output file]
```

<!-- more -->

### Example

```sh
sips -s format png test.jpg --out test.png
```

## Batch Image Conversion with sips

```sh
for i in *.jpeg; do sips -s format png $i --out Converted/$i.png;done
```

## Links

* [Converting Image File Formats with the Command Line & sips](https://osxdaily.com/2013/01/11/converting-image-file-formats-with-the-command-line-sips/)
