Working with PDF/DJVU
=====================


PDF compression
---------------


Compressing pdf
~~~~~~~~~~~~~~~

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
with :wikipedia:`PDF/X` but correct in :wikipedia:`PDF/A`)

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

Choosing the pixel density for images.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usually we choose the pixel :wikipedia:`pixel density` in dependence with the
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


PDF compression of Images.
~~~~~~~~~~~~~~~~~~~~~~~~~~
There are many compression algorithm the can be applied to an image
before including it in pdf. Images are stored as binary strings.

You can list the compression methods by:
::

    $strings document.pdf| \
      sed -n '/\/Filter \/[a-zA-Z]*Decode/s/^.*\/Filter \(\/[a-zA-Z]*Decode\).*/\1/'p| \
      sort -u

Imagemagick allow many options for decoding images listed by:
::

    $ convert -list compress

but only some are usable, for pdf output, they are listed in the
`ImageMagick manual: pdf options
<http://www.imagemagick.org/Usage/formats/#pdf_options>`_
:

+------------------------------+------------------------------+
|Compression                   |image '/Filter [... ]' setting|
|                              |                              |
+==============================+==============================+
|"-compress none"              |'/ASCII85Decode'              |
+------------------------------+------------------------------+
|"-compress zip"               |'/FlateDecode'                |
+------------------------------+------------------------------+
|"-compress jpeg"              |'/DCTDecode'                  |
+------------------------------+------------------------------+
|"-compress lzw"               |'/LZWDecode'                  |
+------------------------------+------------------------------+
|"-alpha off -monochrome       |'/CCITTFaxDecode'             |
|-compress fax"                |                              |
+------------------------------+------------------------------+
|"+compress" "-compress rle"   |  '/RunLengthDecode'          |
| any thing else               |                              |
+------------------------------+------------------------------+

When you use *ps2pdf* Ghostscript by default examine the image to
decide between JPEG and LZW or Flate compression:


`Scanning Lecture Notes – Compression
<https://www.guyrutenberg.com/2012/10/08/scanning-lecture-notes-compression/>`_
compare the size of some compressions method for a 38 pages handwritten document.
It shows the following resut.

| compression     | size MB |
| Deflate         |      10 |
| LWZ             |     9.7 |
| Group 4         |     2.7 |
| JBig2 (lossy)   |     2.1 |
| djvu (lossless) |     2.2 |
| djvu (lossy)    |     1.8 |

djvu files are generated by minidjvu.

An other test in `PDFs vs PNG vs JBIG2
<http://ssdigit.nothingisreal.com/2010/03/pdfs-jpeg-vs-png-vs-jbig.html>`_
show the following results:

| compression    | size MB |
| jpeg           |    43.8 |
| png            |     6.9 |
| jbig2          |     0.9 |
| jbig2 upscaled |     1.4 |




Extracting from pdf
-------------------

using mutool
~~~~~~~~~~~~

*mutool* extract objects in the current directory, so you better act in
a dedicated subdirectory:

::

    $ mkdir extracted; cd extracted

To see the list of objects

::

    $ mutool info ../document.pdf
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
the object number is found in the last column which read here ``(3 0
R)`` so the object number is ``3``.

To extract an image:

::

    $ mutool extract ../document.pdf 3
    extracting image img-003.png

The same command can also extract fonts, but here the given object
nomber is to the one of the font. To extract the ``TimesNewRomanPSMT``
you do
::

    $ mutool extract ../document.pdf 857
    extracting font TimesNewRomanPSMT-0857.ttf



Using pdfimages
~~~~~~~~~~~~~~~

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

That generate a ``document-images-000.jb2e`` in the original :wikipedia:`jbig2`
format.

The :wikipedia:`jbig2` format is patent protected from IBM and Mitsubishi.
JBIG2 is designed for lossy or lossless encoding of 'bilevel' (1-bit
monochrome) images at moderately high resolution, and in particular
scanned paper documents. In this domain it can be very efficient,
offering compression ratios on the order of 100:1. JBIG2 images can be
included in PDF from version 1.4. It is very similar to the JB2
compression scheme used in the :wikipedia:`DjVu` file format, but JB2 is open
source.

To manipulate jbig2 file you can use the open source encoder
`jbig2enc <https://github.com/agl/jbig2enc>`__ or decoder `jbig2dec from
ghostscript <http://www.ghostscript.com/jbig2dec.html>`__ (man
[man:jbig2dec]), which can decode jbig2 to png or pbm.

If you need to use the image out of pdf, you may prefer a more usual
format than ``jbig2`` and do:

::

    $ pdfimages  -png -f 3 -l 3 document.pdf document-images

to get a ``document-images-000.png``. Note that you get images in
``jbig2``, ``jpeg``, ``jpeg2000`` if they are yet in this format in the
pdf stream, the only available conversions are to ``pbm``, ``tiff`` and
``png``.


Using pdf-parser.py
~~~~~~~~~~~~~~~~~~~

To look at the description of object containing images in a document:

::

    $ ./pdf-parser.py -s '/Subtype /Image' document.pdf
    obj 6 0
     Type: /XObject
     Referencing:
     Contains stream

      <<
        /Type /XObject
        /Subtype /Image
        /BitsPerComponent 8
        /Width 773
        /Height 279
        /ColorSpace /DeviceRGB
        /Filter /DCTDecode
        /Length 18841
      >>

    obj 7 0
     Type: /XObject
     Referencing:
     Contains stream

      <<
        /Type /XObject
        /Subtype /Image
        /BitsPerComponent 8
        /Width 587
        /Height 480
        /ColorSpace /DeviceRGB
        /Filter /DCTDecode
        /Length 40962
      >>
      ...



Encoding pdf with jbig2.
~~~~~~~~~~~~~~~~~~~~~~~~

The jbig2 options are:

-  ``-d | --duplicate-line-removal``: When encoding generic regions each
   scan line can be tagged to indicate that it's the same as the last
   scanline This is an option
   because some versions of ``jbig2dec`` cannot handle this.
-  ``-p | --pdf``: Encode with PDF format for
   JBIG2 streams. In symbol mode the output is to a series of files:
   ``symboltable`` and ``page-``\ *n* (numbered from 0)
-  ``-s | --symbol-mode``: use symbol encoding. Turn on for scanned text
   pages it implies symbol recognition and encode each recognized
   symbol only once.
-  ``-t <threshold>``: sets the fraction of pixels which have to match
   in order for two symbols to be classed the same increasing this will
   increase the number of symbol classes.
-  ``-T <threshold>``: sets the black threshold (0-255). Any gray value
   darker than this is considered black. Anything lighter is considered
   white.
-  ``-r | --refine <tolerance>``: (requires ``-s``) turn on refinement
   for symbols with more than ``tolerance`` incorrect pixels. (10 is a
   good value for 300dpi, try 40 for 600dpi). Note: this is known to
   crash Adobe products.
-  ``-O <outfile>``: dump a PNG of the 1 bpp image before encoding. Can
   be used to test loss.
-  ``-2`` or ``-4``: upscale either two or four times before converting
   to black and white.
-  ``-S`` Segment an image into text and non-text regions. This isn't
   perfect, but running text through the symbol compressor is terrible
   so it's worth doing if your input has images in it (like a magazine
   page). You can also give the ``--image-output`` option to set a
   filename to which the parts which were removed are written (PNG
   format).



::

    $ .jbig2 -s --pdf *.pbm
    $ python pdf.py output > jbig2_pbm.pdf
    $ rm output.*

::

    $ jbig2 -d -p -s *.jpg; pdf.py J > JBIG2.pdf

    $ jbig2 -s -p -d -v *.jpg; pdf.py > jbig2_doc.pdf


`Jbig2enc manual
<https://raw.githubusercontent.com/agl/jbig2enc/master/doc/jbig2enc.html>`_


Bundling pdf pages
~~~~~~~~~~~~~~~~~~

To create a blank page:
::

    $ echo "" | ps2pdf -sPAPERSIZE=a4  - /tmp/blank.pdf
    $ convert -size 1024x1448 xc:white /tmp/blank.pdf


To insert the page at some position:
::

    $ pdftk A=document.pdf B=/tmp/blank.pdf cat A1-3 B1-1 A4-230 B1-1 A231-250 output combined.pdf

Creating djvu document
----------------------

Bitonal document
~~~~~~~~~~~~~~~~
You can create a bitonal djvu page with:
::

    $ cjb2 -clean image.pbm page.djvu


``-clean`` removes small marks caused by noise and dust during the
scanning  process.

To get a smaller file you can try ``-lossy`` and check that the visual
quality of the page is unchanged.

The default resolution is 300 for *pbm* files and unchanged fot *tiff*
file, but you can choose a specific resolution with ``-dpi``.


You can also use *minidjvu* to encode multiple bitonal pages:
::

    $ minidjvu --clean image_*.pbm document.djvu

*minidjvu* has options similar to the previous *cjb2* ones:
``-clean``, ``--lossy``, ``--dpi``, you can also fine tune the lossy
encoding with ``--aggression``, ``--erosion``, ``--smooth``,
the ``--lossy`` option is a shortcut for
``--clean  --erosion --aggression 100 --smooth``

If there i a loss of quality with ``--lossy`` you can try to drop
``--clean`` and  ``--erosion`` with ``--aggression 100 --smooth``
wich can also be written ``--match --smooth``.

Refs:
    `cjb2(1)
    <http://djvu.sourceforge.net/doc/man/cjb2.htmlhttp://djvu.sourceforge.net/doc/man/cjb2.html>`_,
    :man:`minidjvu(1)`

Colour document.
~~~~~~~~~~~~~~~~

Refs:
    :man:`didjvu(1)`,
    `c44(1)
    <http://www.djvuzone.org/open/doc/c44.html>`_,
    `Créer un fichier DjVu/Linux
    <https://fr.wikisource.org/wiki/Aide:Cr%C3%A9er_un_fichier_DjVu/Linux>`_


Working with djvu.
------------------


Bundling djvu pages.
~~~~~~~~~~~~~~~~~~~~

To bundle individual pages in a document:
::

    $ djvm -c page_*.djvu document.djvu


To list the components of a document:
::

    $ djvm -l document.djvu

To insert a new page as 18th page in a document:
::

    $ djvm -i document.djvu page.djvu 18

To remove the 18th page:
::

    $ djvm -d document.djvu 18

To insert an empty blank page first create a new blank page with the
same size, and the same resolution than your book pages:
::

    $ convert -size 2583x3354 xc:white /tmp/blank.tiff
    $ cbj2 -dpi 600 /tmp/blank.tiff blank.djvu

then  insert it as previously:
::

    $ djvm -i document.djvu blank.djvu 2

Refs:
    `djvm(1)
    <http://djvu.sourceforge.net/doc/man/djvm.html>`_



Managing djvu outline.
~~~~~~~~~~~~~~~~~~~~~~
The outline is represented by :wikipedia:`S-expressions` in a textual
file.


To print the current outline:
::

    $ djvused -e "print-outline" document.djvu

To replace the outline by a new one create a text file
with the textual representation of the outline:

Your file looks like:
::

    $ cat outline.el
    (bookmarks
     ("Contents"
      "#5" )
     ("Preface"
      "#20" )
     ("chapter 1"
      "#31"
      ("Section 1.1"
       "#31"
       ("The first subject"
        "#32" )
       ("The second subject"
        "#36" )
       ("The third subject"
        "#41" )
       ("The fourth subject"
        "#46" ) ) ) )


Then insert your outline with:
::

    $ djvused -s -e "set-outline "outline.el" document.djvu

Refs:
    :man:`djvused(1)`

Setting the page numbers.
~~~~~~~~~~~~~~~~~~~~~~~~~

To renumber some page:
::

    $ djvused -s -e "select 1; set-page-title C0;" document.djvu

If you have many pages to renumber, create a script to generate your
*djvused* program.

::

    $ cat repaginate.sh
    #!/bin/sh
    echo <<EOF
    select 1; set-page-title C0;
    select 262; set-page-title C4;
    EOF
    for i in $(seq 2 261)
    do
      echo "select $i; set-page-title $(($i+1));"
    done

    $ sh repaginate.sh > /tmp/repaginate.djvused
    $ djvused -s -f /tmp/repaginate.djvused document.djvu


Refs:
    :man:`djvused(1)`



PDF and DJVU bookmarks
----------------------

..  _bmconverter:

Bookmark conversion
~~~~~~~~~~~~~~~~~~~

`bmconverter.py <http://goerz.github.io/bmconverter.py/>`_
converts between the bookmark description formats used
by different pdf and djvu bookmarking tools such as pdftk, the iText
toolbox, pdfLaTeX pdfWriteBookmarks, jpdftweak, djvused, and the DJVU
Bookmark Tool.

*bmconverter.py* is `available in GitHub
<https://github.com/goerz/bmconverter.py>`_.


pdf bookmarks with pdftk.
~~~~~~~~~~~~~~~~~~~~~~~~~
`pdftk - The PDF Toolkit
<https://www.pdflabs.com/tools/pdftk-server/>`__ (GPL) is a java
*compiled (with gcj)* application which uses the
`iText library <http://en.wikipedia.org/wiki/IText>`__ (LGPL).

It can merge, split, rotate, encryt, decrypt, attach files, unpack,
repair pdf documents. It allows also to fill PDF Forms with FDF data
or XFDF data and flatten Forms.

*pdftk* allows also to edit bookmarks, by dumping bookmarks to a text
file, and importing bookmarks from a text file. The bookmark format is
specific to *pdftk*; but the :ref:`bmconverter <bmconverter>` program
allow to convert it from and to other formats.

To dump the bookmarks do

::

    $ pdftk document.pdf dump_data output bookmarks.txt

You can the work with the text file *bookmark.txt* then

::

    $ pdftk document.pdf update_info bookmarks.txt output document_new.pdf


The python3 script `booky <https://github.com/SiddharthPant/booky>`_
is a *pdftk* wrapper that uses a simpler bookmark format.
