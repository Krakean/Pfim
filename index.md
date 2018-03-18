[![Pfim-viewer](img/pfim-viewer.png)](img/pfim-viewer.png)

*A sample WPF interface that uses the Pfim library to load Targa and DDS images. Images are not my own*

Pfim is a .NET Standard 1.0 compatible Targa (tga) and Direct Draw Surface
(dds) decoding library with an emphasis on speed and ease of use. Pfim can be
used on your linux server, Windows Form, or WPF app!

## Motivation

I needed a C# [Targa](https://en.wikipedia.org/wiki/Truevision_TGA) and [Direct
Draw Surface (DDS)](https://en.wikipedia.org/wiki/DirectDraw_Surface) decoder
and the ones out there didn't satisfy my performance and portability needs, so
I wrote my own. As consequence, Pfim does not parse every Targa and DDS image,
but every image that I've seen in the wild is supported. If your image is not
supported, please create an [issue](https://github.com/nickbabcock/Pfim/issues)
with the image attached.

## Installation

[Install from NuGet](http://www.nuget.org/packages/Pfim/)

## Usage

Pfim emphasizes on being frontend and backend agnostic. While it can be used
anywhere, the downside is that you may need some code to display the image in
whatever form desired, but as you'll see it is not that hard!

{% highlight csharp %}
// Load image from file path
IImage image = Pfim.FromFile(@"C:\image.tga");
{% endhighlight %}

If one already has the data stream and knows the format, each format can be directly called:

{% highlight csharp %}
// Obtain a stream of data somehow
var stream = new MemoryStream();

// Creates a direct draw surface image
IImage image = Dds.Create(stream);

// Creates a targa image
IImage image2 = Targa.Create(stream);
{% endhighlight %}

Every image is decoded into a 24bit RGB or 32bit RGBA format, as denoted by `image.Format`.

## Benchmarks

The contestants:

- Pfim 0.4.4
- [DevIL](http://openil.sourceforge.net/) 0.0.13
- [FreeImage](http://freeimage.sourceforge.net/) 4.3.6
- [ImageMagick](https://www.imagemagick.org/script/index.php) 7.4.3
- [TargaImage](https://www.codeproject.com/Articles/31702/NET-Targa-Image-Reader) 1.0
- [ImageFormats](https://github.com/dbrant/imageformats) 1.0
- [StbSharp](https://github.com/rds1983/StbSharp) 0.6.8.33
- [TgaSharpLib](https://github.com/ALEXGREENALEX/TGASharpLib) 1.0

[![Pfim-benchmark1](img/benchmark.png)](img/benchmark.png)

Notice Pfim is one of the fastest if not the fastest in each decoding each image.

**Caveat**: Some libraries do much more than decode images, and may not have the time or want the sacrifice that may come with optimizing the decoding process.

Image definitions:

- A large (1200x1200) 24bit (no alpha component) targa image
- A small (64x64) 24bit (no alpha component) targa image
- A large (1200x1200) 32bit run length encoded (RLE) targa image
- A small (64x64) 32bit run length encoded (RLE) targa image
- A small (64x64) uncompressed DDS image
- A small (64x64) DXT1 encoded DDS image
- A small (64x64) DXT3 encoded DDS image
- A small (64x64) DXT5 encoded DDS image

The benchmarking was done through [Benchmarkdotnet](https://github.com/dotnet/BenchmarkDotNet) and the benchmark code can
be found in the repo.

## Integrations

Since Pfim is backend and frontend agnostic, one has to write the appropriate translations. Below is the WPF specific code, which can be seen in the Pfim.Viewer sample project in the source code.

{% highlight csharp %}
private static BitmapSource LoadImage(IImage image)
{
    PixelFormat format;
    switch (image.Format)
    {
        case ImageFormat.Rgb24:
            format = PixelFormats.Bgr24;
            break;

        case ImageFormat.Rgba32:
            format = PixelFormats.Bgr32;
            break;

        default:
            throw new Exception("Format not recognized");
    }

    // Create a WPF ImageSource and then set an Image to our variable.
    // Make sure you notify property changes as appropriate ;)
    return BitmapSource.Create(image.Width, image.Height,
        96.0, 96.0, format, null, image.Data, image.Stride);
}
{% endhighlight %}

Below is the code used in the Windows Forms sample, Pfim.Viewer.Forms (also found in the source code).

{% highlight csharp %}
var image = Pfim.FromFile(dialog.FileName);

PixelFormat format;
switch (image.Format)
{
    case ImageFormat.Rgb24:
        format = PixelFormat.Format24bppRgb;
        break;

    case ImageFormat.Rgba32:
        format = PixelFormat.Format32bppArgb;
        break;

    default:
        throw new Exception("Format not recognized");
}

unsafe
{
    fixed (byte* p = image.Data)
    {
        var bitmap = new Bitmap(image.Width, image.Height, image.Stride, format, (IntPtr) p);
        pictureBox.Image = bitmap;
    }
}
{% endhighlight %}

## Release Notes

### 0.4.4 - October 31st 2017
* Fix red and blue color swap for TopLeft encoded targa images
* 20x performance improvement for TopLeft encoded targa images

### 0.4.3 - October 31st 2017
* Fix infinite loop on certain large targa and dds images

### 0.4.2 - October 10th 2017
* Release .NET Standard 1.0 version that doesn't contain File IO

### 0.4.1 - October 9th 2017
* Fix decoding of non-square uncompressed targa images
* Fix edge case decoding for compressed targa images

### 0.4.0 - September 17th 2017
* Released for netstandard 1.3
* 25% performance improvement on compressed dds images
* Bugfix in compressed targa decoder

### 0.3.1 - August 18th 2015
* Fix pixel depth calculations for compressed dds

### 0.3 - April 30 2015
* Internalized a lot of API to simplify usage
* Publish benchmarking

### 0.2 - April 29 2015
* All decoded images now derive from `IImage`

### 0.1 - April 26 2015
* Initial release
