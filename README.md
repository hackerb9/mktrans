# mktrans
Make background transparent using antialiased alpha channel

mktrans: Convert images into shaped transparent pngs by floodfilling
	 the background with transparency (antialiased alpha channel).
         Top left pixel is assumed to be the background color.

Usage: mktrans <files ... >

Output filenames will be the same as input, except suffixed with
"-transparent.png". E.g., 'mktrans foo.gif bar.jpg' creates
foo-transparent.png and bar-transparent.png.

This is similar to ImageMagick's bg_removal script, but much higher
quality. (It's also faster and simpler to use.) 

For a sample, run these commands:

    convert logo: logo.png
    mktrans logo.png
    display logo-transparent.png

Side note: This creates an antialiased (blurred) alpha channel that is
also eroded by half a pixel to avoid halos. Of course, ImageMagick's
morphological operations don't (yet?) work at the subpixel level, so
I'm blowing up the alpha channel to 200% before eroding.
