#!/bin/bash 
# B9 June 2017

# mktrans 
# This is similar to ImageMagick's bg_removal script, but much higher
# quality. (It's also faster and simpler to use.) 
#
# For a sample, run these commands:
#    convert logo: logo.png
#    mktrans logo.png
#    display logo-transparent.png


# Fuzz is how far off the background color can be (in percent).
# This is important for getting good antialiasing.
defaultfuzz=20

usage() {
    cat <<EOF

mktrans: Convert images into shaped transparent pngs by floodfilling
	 the background with transparency (antialiased alpha channel).

	 Top left pixel is assumed to be the background color and floodfill
	 is from image edges, unless a different starting pixel is specified.

Example: mktrans foo.jpg

Usage: mktrans [-f <fuzz>] [-s] [-p <x>,<y>] [-A] [-v] <files ... >

	-f <fuzz>: How loosely to match the background color (default $defaultfuzz%)
	       -s: Use speedy antialiasing (much faster, slightly less acurate) 
	       -p <x>,<y>: Floodfills from pixel at x,y instead of 0,0
	       -A: Do not antialias transparency
	       -v: Verbose (see how 'convert' is being called)

Output filenames will be the same as input, except suffixed with
"-transparent.png". E.g., 'mktrans foo.gif bar.jpg' creates
foo-transparent.png and bar-transparent.png.

Side note: This creates an antialiased (blurred) alpha channel that is
also eroded by half a pixel to avoid halos. Of course, ImageMagick's
morphological operations don't (yet?) work at the subpixel level, so
I'm blowing up the alpha channel to 200% before eroding. Since this
can be slow on large images, consider using the '-s' option which
uses a faster, lower quality antialiasing.
EOF
    exit 0
}

fuzz=$defaultfuzz
pixelcomma="0,0"
pixelplus="+0+0"

while getopts f:sAhp:v name; do
    case $name in
        f) fuzz=$OPTARG
           ;;
        s) sflag=True
           ;;
        A) noantialias=True
           ;;
        v) vflag=True
           ;;
        h) usage
           ;;
        p) pixelcomma=$OPTARG
           pixelplus=+${OPTARG%,*}+${OPTARG#*,}
           pflag=True
           ;;
        *) usage
           ;;
    esac
done

shift $((OPTIND-1))
[[ "$#" != 0 ]] || usage


for filename; do
    # Get color of 0,0 (top left) pixel
    color=$(convert "$filename" -format "%[pixel:p{$pixelcomma}]" info:-)
    if [[ "$color" == *rgba*",0)" ]]; then
	color="${color%,0)},1)"	# Floodfill only works with opaque colors.
    fi
    if [[ "$color" == "none" ]]; then
	echo "Error: $filename: pixel at $pixelcomma is completely transparent. Cannot floodfill." >&2
	continue
    fi

    options=""
    if [ -z "$pflag" ]; then
	# Add a 1 pixel border so we'll fill from the bottom and sides as well.
	options+=" -bordercolor $color -border 1 "
    fi
    # In a new stack, make a copy of the image
    options+=" ( +clone "
    # [copy] floodfill with transparency ("none") starting at top-left
    options+="		-fuzz $fuzz% -fill none -floodfill $pixelplus $color"
    # [copy] extract just the transparency (alpha channel)
    options+="		-alpha extract"

    if [ -z "$noantialias" ]; then
	if [ -z "$sflag" ]; then
            # [copy] blow up the alpha channel so we can do sub-pixel morphology
            options+="	-geometry 200%"
            # [copy] blur the alpha channel to make it antialiased
            options+="	-blur 0x0.5"
            # [copy] shrink the region that is opaque by half a pixel.
            options+="	-morphology erode square:1"
            # [copy] scale the alpha channel back to normal size.
            options+="	-geometry 50%"
	else	# sflag: speedy antialias
            # [copy] blur the alpha channel to make it antialiased
            options+="	-blur 0x1"
            # [copy] only antialias inside the figure (<50% opacity becomes 0%)
            options+="	-level 50%,100%"
	fi
    fi
    # [copy] end the stack.
    options+=" ) "
    # Compose the original image and the copy's alpha channel.
    options+=" -compose CopyOpacity -composite"
    if [ -z "$pflag" ]; then
	# Remove the 1 pixel border we added
	options+=" -shave 1"
    fi
    
    [ "$vflag" ] && echo convert "$filename" $options "${filename%.*}-transparent.png"

    convert "$filename" $options "${filename%.*}-transparent.png"
done
