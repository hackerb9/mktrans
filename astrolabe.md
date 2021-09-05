# Using -S to deal with multiple runs

If you run this script on a file that already has transparency, you'll
probably want to use the `-S` option to suppress antialiasing. But why
would you want multiple runs of mktrans on the same file? Because one
can clear out "lagoons", areas that should have been transparent but
the floodfill didn't reach them. You do that by using the `-p x,y`
option to specify which pixel to start floodfilling at.

## Lagoon problem

Here's an example image which has already been run through `mktrans` once. As you can see, it left a black lagoon in the middle of the loop on this astrolabe. 

<img src="README.md.d/a-transparent.png" align="center" width="50%" alt="After first run of mktrans">

## Eroded pixels problem

We can find the coordinates of the lagoon by running `display
a-transparent.png` and clicking the middle mouse button. In this case,
256,64 is one of the black pixels.

If we did the most obvious thing and just ran

```bash
mktrans -p 256,64 a-transparent.png
```

then the output file (a-transparent-transparent.png) would lose some pixels due to rerunning the antialiasing step on the old transparency. 

<img src="README.md.d/a-transparent-transparent (without -S).png" align="center" width="50%" alt="Second run, but without -S">

## Kludge: Using -S on subsequent runs

Running `mktrans -S` supresses antialiasing completely. 

```bash
mktrans -S -p 256,64 a-transparent.png
```
As you can see this looks much better:

<img src="README.md.d/a-transparent-transparent (with -S).png" align="center" width="50%" alt="Second run, with  -S">

## But wait... isn't there a better way?

Yes. There are at least two ways this could be better, but there are no plans to implement them. 

1. mktrans could automatically use `-S` when it detects a file already contains transparency.

2. More importantly, `-S` isn't even the right solution. The proper thing to do is to extract and remove the alpha channel from the original image, run mktrans, and
then composite the original alpha channel with the new one.

But, both of those improvements are only necessary if people are going to be running `mktrans` iteratively and the author (hackerb9) is not convinced that
people want to do that. The only use case is when people are using the `-p` option, which is already a pain in the butt to use as it requires firing up a graphical
program, like ImageMagick's `display`, to find the coordinates of lagoons. Wouldn't people just want to use a program like the [GNU Image Manipulation Program](https://gimp.org/)
instead of using mktrans? 

If you think these or other improvements should be made, please file a bug with an explanation of your reasoning.

