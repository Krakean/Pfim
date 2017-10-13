[![Pfim-viewer](img/pfim-viewer.png)](img/pfim-viewer.png)

*A sample WPF interface that uses the Pfim library to load Targa and DDS images. Images are not my own*

## Motivation

I needed a C# [Targa](https://en.wikipedia.org/wiki/Truevision_TGA) and
[Direct Draw Surface (DDS)](https://en.wikipedia.org/wiki/DirectDraw_Surface)
decoder and the ones out there didn't satisfy my needs, so I wrote my own.

Pfim fills this role by being entirely written in .NET (eases cross platform
worries) with an easy to learn API. Pfim also clocks in as one of the fastest
(if not the fastest) implementation. See the benchmarks for more information.

## Installation

[Install from NuGet](http://www.nuget.org/packages/Pfim/)

## Usage

Pfim emphasizes on being frontend and backend agnostic. This means that you
can add Pfim to your server, Windows Form, or WPF app! The downside is that
you may have to put in a little work to get into a useable form, but as you'll
see it is not that hard!

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

## Benchmarks

The following contestants:

- Pfim
- [DevIL](http://openil.sourceforge.net/)
- [FreeImage](http://freeimage.sourceforge.net/)
- [ImageMagick](https://www.imagemagick.org/script/index.php)
- [TargaImage](https://www.codeproject.com/Articles/31702/NET-Targa-Image-Reader)

**Caveat**: These libraries do much more than decode images, and may not have the time or want the sacrifice that may come with optimizing the decoding process.

These libraries competed in decoding the following images:

- A large (1200x1200) 24bit (no alpha component) targa image
- A small (64x64) 24bit (no alpha component) targa image
- A large (1200x1200) 32bit run length encoded (RLE) targa image
- A small (64x64) 32bit run length encoded (RLE) targa image
- A small (64x64) uncompressed DDS image
- A small (64x64) DXT1 encoded DDS image
- A small (64x64) DXT3 encoded DDS image
- A small (64x64) DXT5 encoded DDS image

[![Pfim-benchmark1](img/benchmark.png)](img/benchmark.png)

The light blue is Pfim. Notice that it sets the record for highest throughput for each category relative to all the other decoders benchmarked.

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
