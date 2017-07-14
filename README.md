# mktrans

    Convert images into shaped transparent pngs by floodfilling
    the background with transparency (antialiased alpha channel).
    Top left pixel is assumed to be the background color.

[![After flattening on a saddle brown background](https://i.imgur.com/Exrm0tD.png)](https://i.imgur.com/PReCAca.png)

# Usage

    Usage: mktrans [-f <fuzz>] [-s] [-v] <files ... >

        -f <fuzz>: How loosely to match the background color (default 20%)
               -s: Use speedy antialiasing (much faster, slightly less acurate) 
               -v: Verbose

Output filenames will be the same as input, except suffixed with
`-transparent.png`. E.g., `mktrans foo.gif bar.jpg` creates
`foo-transparent.png` and `bar-transparent.png`.

This is similar to ImageMagick's ["bg_removal"](https://www.imagemagick.org/Usage/scripts/bg_removal) script, but much higher
quality. (It's also faster and simpler to use.) 

For a sample, run these commands:

    convert logo: logo.png
    mktrans logo.png
    display logo-transparent.png

## About -f <fuzz>

*Fuzz* is how far off the background color can be (in percent). You
usually won't have to change this. If fuzz is too high, parts of the
foreground image will be missing. If fuzz is too low, parts of the
background will not be removed. On certain images it may help to tweak
the fuzz level to get good antialiasing. (Not losing too much of the
edge, nor leaving any halos).

## About -s (speedy)

This script creates an antialiased (blurred) alpha channel that is
also eroded by half a pixel to avoid halos. Of course, ImageMagick's
morphological operations don't (yet?) work at the subpixel level, so
I'm blowing up the alpha channel to 200% before eroding. Since this
can be slow on large images, consider using the '-s' option which
skips the subpixel eroding.

---HackerB9, June 2017
