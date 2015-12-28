Compressing pdf
===============

Using Ghostscript we can do:

::

    $ gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/screen\
    -dNOPAUSE -dQUIET -dBATCH -sOutputFile=out.pdf in.pdf

To get a better resolution ``-dPDFSETTINGS`` can be ``/screen``,
``/ebook``, ``/printer`` or ``/prepress``. They imply an image
resolution of 72dpi, 150dpi or 300dpi see the ``ColorImageResolution``
parameter in `ps2pdf
options <http://ghostscript.com/doc/current/Ps2pdf.htm#Options>`__ Look
also at the ``DownSamplexxxImage`` options.

If you want to do color conversion use ``ColorConversionStrategy``
switch with one of ``/LeaveColorUnchanged``, ``/Gray``, ``/RGB``,
``/CMYK`` or ``/UseDeviceIndependentColor``. ( ``/RGB`` is not allowed
with [w:PDF/X] but correct in [w:PDF/A])

We may try also to use ``DownsampleType`` with the the value
``/Bicubic`` to gain a better compression than the default method.

In any case it is a *lossy* compression.

We can use ``qpdf`` to do a content-preserving changes.

::

    $ qpdf --linearize input.pdf output.pdf

Or ``pdftk``:

::

    $ pdftk file1.pdf output file2.pdf compress

If you have a pdf with only scanned images, you can use ImageMagick
``convert`` to create a pdf with jpeg compression (you will loose all
text informations).

::

    $ convert input.pdf -density 200x200 -quality 90 -compress jpeg \
    output.pdf

Imagemagik allow to choose the density quality and `compress
type <http://www.imagemagick.org/script/command-line-options.php#compress>`__

choosing the pixel density for images
-------------------------------------

Usually we choose the pixel [w:pixel density] in dependence with the
device capabilities.

Computer screen use between 72dpi and 100 dpi, but if we keep this low
density for images we will not get a good result when zooming in. The
better is to use 72\*max zoom rate.

For printing we use 300dpi, 300dpi or 400dpi is also a good density for
OCR.

For ebook reader we have a usual density close ranging from 200dpi to
300dpi, E-Ink Pearl, E Ink Carta HD, are 1430 x 1080, 265 dpi, 6.8'.

But usualy the book is zoomed out from a larger format.

6.8'=17.27cm diagonal is 4.1'=10.41cm wide and and 5.42'=13.78cm heigh.

If you have an A5 book, 148x210 mm you have to zoom out by 104/148 =
0.70 to make it full width, or 138 / 210 = 0.65 to get it full height.

But there is usually a margin on printed matters that you don't want to
keep on the ebook reader. So you can often keep a 100% zoom factor, and
crop the page.

If you have an A4 book, you should at least reduce by 0.70 so your 300
dpi on the tablet need only 210 dpi on the original.

Extracting from pdf
===================

using mutool
------------

*mutool* extract objects in the current directory, so you better act in
a dedicated subdirectory:

::

    $ mkdir extracted; cd extracted

To see the list of objects

::

    $ mutool extract ../document.pdf
    Info object (897 0 R):
    <</CreationDate(D:20110929010358Z)/ModDate(D:20111031125854+11'00')/Producer(ABBYY FineReader 8.0 Professional Edition; modified using iTextSharp 5.0.6 \(c\) 1T3XT BVBA)>>

    Pages: 274

    Retrieving info from pages 1-274...
    Mediaboxes (28):
        1   (901 0 R):  [ 0 0 485.28 702.36 ]
        2   (1 0 R):    [ 0 0 485.16 723.84 ]
    ....
    Fonts (10):
        3   (4 0 R):    TrueType 'TimesNewRomanPSMT' (855 0 R)
        ....
    Images (274):
        1   (901 0 R):  [ DCT ] 1348x1951 8bpc DevRGB (905 0 R)
        2   (1 0 R):    [ JBIG2 ] 4043x6032 1bpc DevGray (3 0 R)
    ...
        274 (820 0 R):  [ DCT ] 1348x1939 8bpc DevRGB (823 0 R)

The second image is a jbig2 compressed png 4043x6032 monochrome and is
the object number 3

To extract an image:

::

    $ mutool extract ../document.pdf 3
    extracting image img-003.png

Using pdfimages
---------------

To list images:

::

    $ pdfimages -list document.pdf
    page   num  type   width height color comp bpc  enc interp  object ID x-ppi y-ppi size ratio
    --------------------------------------------------------------------------------------------
       1     0 image    1348  1951  rgb     3   8  jpeg   no       905  0   201   200  963K  12%
       2     1 image    4043  6032  gray    1   1  jbig2  no         3  0   600   601   30B 0.0%
       3     2 image    4046  6034  gray    1   1  jbig2  no         6  0   600   600 3742B 0.1%
       4     3 image    4043  6032  gray    1   1  jbig2  no         9  0   600   601   30B 0.0%

The object number is important for extracting the images. The listing is
more detailled than the one you get with ``mutool info`` or
``qpdf --show-pages``.

Then you can extract the images either in the native format with:

::

    $ pdfimages  -all -f 3 -l 3 document.pdf document-images

That generate a ``document-images-000.jb2e`` in the original [w:jbig2]
format.

The [w:jbig2] is a patent protected format from IBM and Mitsubishi.
JBIG2 is designed for lossy or lossless encoding of 'bilevel' (1-bit
monochrome) images at moderately high resolution, and in particular
scanned paper documents. In this domain it can be very efficient,
offering compression ratios on the order of 100:1. JBIG2 images can be
included in PDF from version 1.4. It is very similar to the JB2
compression scheme used in the [w:DjVu] file format, but JB2 is open
source.

To manipulate jbig2 file you can use the open source encoder
`jbig2enc <https://github.com/agl/jbig2enc>`__ or decoder `jbig2dec from
ghostscript <http://www.ghostscript.com/jbig2dec.html>`__ (man
[man:jbig2dec]), which can decode jbig2 to png or pbm.

If you need to use the image out of pdf, you may prefer a more usual
format tha ``jbig2`` and do:

::

    $ pdfimages  -png -f 3 -l 3 document.pdf document-images

to get a ``document-images-000.png``. Note that you get images in
``jbig2``, ``jpeg``, ``jpeg2000`` if they are yet in this format in the
pdf stream, the only available conversions are to ``pbm``, ``tiff`` and
``png``.
