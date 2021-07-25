# mktrans

    Convert images into shaped transparent pngs by floodfilling the
    background with transparency (antialiased alpha channel).

    Top left pixel is assumed to be the background color and floodfill
    is from image edges, unless a different starting pixel is specified.

## Example

This program is meant to be quick and simple to use, so most of the
time simply specifying a filename, such as `foo.jpg`, is all you need.

    mktrans foo.jpg

(Result will be in `foo-transparent.png`.)

If too much or too little of the image is turning transparent, you can
specify a different "fuzz" factor.

    mktrans -f 10 foo.jpg

## Sample results

For a quick sample using the ImageMagick logo, run these commands:

    convert logo: logo.png
    mktrans logo.png
    display logo-transparent.png

[![After flattening on a saddle brown background](https://i.imgur.com/Exrm0tD.png)](https://i.imgur.com/PReCAca.png)

## Installation

It's just a shell script, so just download the mktrans file, mark it executable, and put it in your bin directory.

    wget https://github.com/hackerb9/mktrans/raw/master/mktrans
    chmod +x mktrans
    mv mktrans /usr/local/bin/

## All flags and complete usage

    Usage: mktrans [-f <fuzz>] [-s|-S] [-v] <files ... >

        -f <fuzz>: How loosely to match the background color (default 20%)
               -s: Use speedy antialiasing (much faster, slightly less acurate) 
               -S: Do not antialias transparency (useful for repeated runs)
       -p <x>,<y>: Floodfills from pixel at x,y instead of 0,0
               -v: Verbose

Output filenames will be the same as input, except suffixed with
`-transparent.png`. E.g., `mktrans foo.gif bar.jpg` creates
`foo-transparent.png` and `bar-transparent.png`.

### About -f *fuzz*

*Fuzz* is how far off the background color can be (in percent). You
usually won't have to change this. If fuzz is too high, parts of the
foreground image will be missing. If fuzz is too low, parts of the
background will not be removed. On certain images it may help to tweak
the fuzz level to get good antialiasing. (Not losing too much of the
edge, nor leaving any halos).

### About -s (speedily skip subpixels)

By default this script creates an antialiased (blurred) alpha channel
that is also eroded by half a pixel to avoid halos. Of course,
ImageMagick's morphological operations don't (yet?) work at the
subpixel level, so I'm blowing up the alpha channel to 200% before
eroding. Since this can be slow on large images, consider using the
'-s' option which tries to antialias without the subpixel eroding.

### About -S (Supress antialiasing completely)

Similar to -s, but does not antialias at all. Useful with with -p, below.

### About -p *x*,*y*

*X*,*y* specifies which pixel to start floodfilling from, instead of 0,0.
You probably should just _ignore_ this option and use The GIMP for
more fiddly, complex images.

Theoretically you can use -p to remove any lagoons that got missed by
a first floodfill. Note the letters 'a' and 'g' in the example below.

    convert logo: foo.png
    mktrans -p 160,100 foo.png
    mv foo-transparent.png foo.png
    mktrans -S -p 170,80 foo.png
    mv foo-transparent.png foo.png
    mktrans -S -p 195,65 foo.png
    mv foo-transparent.png foo.png
    mktrans -S -p 260,60 foo.png
    mv foo-transparent.png foo.png
    mktrans -S -p 285,55 foo.png
    mv foo-transparent.png foo.png
    mktrans foo.png
    mv foo-transparent.png logo-transparent.png
    
[![Using -p to fill lagoons](https://i.imgur.com/Hxl1a1A.png)](https://i.imgur.com/CmbUnHk.png)

You can find the correct coordinates for a pixel by using
ImageMagick's `display` command and then middle-clicking on the image.
Unfortunately, ImageMagick has no way to use that feature from a shell
script. If it did, this script could simply ask you to click on all
the start points of the image.

## Bugs

* The -p option is ugly and probably nobody wants to use it. Maybe I
  should remove it just to make the documentation shorter and clearer.
  Also, nearly every single bug listed here is due to -p existing.

* Running this script on an image that already has transparency will
  erode the image due to the antialiasing. Using -S is a workaround,
  but is not very satisfactory. Perhaps this script should remove any
  existing transparency before manipulating the image and then add it
  back in at the end. But then again, how often are people going to
  want to do that? The only use I can think of is when using -p.

* Because of the previous bug, if you do use -p to fill lots of
  lagoons, you'll probably want to use -S at the same time.
  (Should this be done automatically if the image already has
  transparency? Yuck, more complexity.)

* Finding the coordinates for -p is a pain. It'd be nice if there was
  a -P option which let the user click on a point (or multiple points)
  in the image to start the floodfill.

## See Also

This is similar to ImageMagick's
["bg_removal"](https://www.imagemagick.org/Usage/scripts/bg_removal)
script, but much higher quality. (It's also faster and simpler to use.) 

