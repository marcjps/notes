# XSL-FO

XSLFO is an XML markup language for formatting PDF files.

## A basic example 

This sample produces a page with Hello World written on it.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fo:root xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:fo="http://www.w3.org/1999/XSL/Format">
    <fo:layout-master-set>
        <fo:simple-page-master master-name="A4-portrait" page-height="297mm" page-width="210mm" margin="2cm">
            <fo:region-body/>
        </fo:simple-page-master>
    </fo:layout-master-set>
    <fo:page-sequence master-reference="A4-portrait">
        <fo:flow flow-name="xsl-region-body">
            <fo:block>Hello world</fo:block>
        </fo:flow>
    </fo:page-sequence>
</fo:root>
```

## How to convert it to PDF

Save the example as test.fo, and run it through Apache FOP.

```text
fop test.fo test.pdf
```

### Page setup (fo:simple-page-master)

A simple page master defines the setup for a page.  

The XSLFO can contain many page masters, to be used where appropriate.

The page master specifies the page size (page-height and page-width attributes), margin sizes (margin, margin-left, etc, attributes) and reference-orientation. (rotation of contents).  It also specifies regions.

#### Regions

The simple page master can specifies these regions on the page:

* region-body 
* region-before = top
* region-after = bottom 
* region-start = left 
* region-end = right

The region-body is the main region, it's usually the full size of the page, minus the margins.

The other regions, if used, occupy the top, bottom, left and right sides of the page, overlapping with the region-body by default.  If you want you can set margins on the region-body which are the size of the other regions, thereby eliminiating the overlap.

Adding the extent attribute on the regions in the page-master defines their size.  You can also set their padding, borders, background and reference orientation. (rotation of contents)

### Page layouts

We may want to use different page layouts in a single publication.

For example, a title page might have a different layout to a content page.  Recto (right) and verso (left) pages usually are usually given slightly different margins, with more space near the inner margins, near the spine of the book.


### Using different page layouts by calling them (fo:page-sequence)

This sample shows two different page setups (simple-page-master), each called explicitly using fo:page-sequence.

It produces two pages, one big, one small.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fo:root xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:fo="http://www.w3.org/1999/XSL/Format">
    <fo:layout-master-set>
        <fo:simple-page-master master-name="page-big" page-height="10cm" page-width="10cm" margin="2cm">
            <fo:region-body/>
        </fo:simple-page-master>
        <fo:simple-page-master master-name="page-small" page-height="5cm" page-width="5cm" margin="2cm">
            <fo:region-body/>
        </fo:simple-page-master>
    </fo:layout-master-set>
    <fo:page-sequence master-reference="page-big">
        <fo:flow flow-name="xsl-region-body">
            <fo:block>Big page</fo:block>
        </fo:flow>
    </fo:page-sequence>
    <fo:page-sequence master-reference="page-small">
        <fo:flow flow-name="xsl-region-body">
            <fo:block>Small page</fo:block>
        </fo:flow>
    </fo:page-sequence>
</fo:root>
```

### Using different page layouts using rules 

This sample produces a big first page followed by small subsequent pages.

In this case the page masters are chosen using the fo:conditional-page-master-reference rules.  page-big on the first page, and page-small on the rest.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fo:root xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:fo="http://www.w3.org/1999/XSL/Format">
    <fo:layout-master-set>
        <fo:simple-page-master master-name="page-big" page-height="10cm" page-width="10cm" margin="2cm">
            <fo:region-body/>
        </fo:simple-page-master>
        <fo:simple-page-master master-name="page-small" page-height="5cm" page-width="5cm" margin="2cm">
            <fo:region-body/>
        </fo:simple-page-master>
        <fo:page-sequence-master master-name="sequence-a">
            <fo:repeatable-page-master-alternatives>
                <fo:conditional-page-master-reference master-reference="page-big" page-position="first" />
                <fo:conditional-page-master-reference master-reference="page-small" page-position="rest" />
            </fo:repeatable-page-master-alternatives>
        </fo:page-sequence-master>
    </fo:layout-master-set>
    <fo:page-sequence master-reference="sequence-a">
        <fo:flow flow-name="xsl-region-body">
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
            <fo:block>Hello world</fo:block>
        </fo:flow>
    </fo:page-sequence>
</fo:root>
```

### Page-position

The values for the page-position attribute are:

* only = used when there's only one page in this sequence.
* first = first page of sequence
* last = last page of sequence
* rest = neither the first nor last page of sequence
* any = any page of sequence.  "Any" will override the others, but if you put it last it is applied with the least priority.

### Odd-or-even

We can also use the odd-or-even attribute to create different conditional-page-master-reference for odd and even pages.  

* odd = used on odd pages
* even = used on even pages
* any = used on any page

### Blank-or-not-blank

We can also use the blank-or-not-blank attribute to create different conditional-page-master-reference dependng on whether the page will be blank.  

we might have a blank page if we've forced a specific number of pages.

* blank = used when the page will be blank (no content to put on it)
* not-blank = used when the page will not be blank 
* any = used on any page

## Forcing a specific number of pages

We can force a page sequence to end on an odd or even page, inserting blank pages as required.  This is done with the force-page-count attribute.

```xml
<fo:page-sequence master-reference="sequence-a" force-page-count="even">
    <fo:flow flow-name="xsl-region-body">
        <fo:block>Hello world</fo:block>
    </fo:flow>
</fo:page-sequence>
```

The values for the force-page-count attibute are:

* auto = The sequence will be forced to end on odd if it began on even, and be forced to end on even if it began on odd.
* even = The sequence will contain an even number of pages
* odd = The sequence will contain an odd number of pages
* end-on-even = The sequence will end with an even page
* end-on-odd = The sequence will end with an odd page
* no-force = Doesn't force anything

## Filling regions with flows and static content 

We can only have one paginated flow in each page sequence.  We can put static content in the other regions.  The static content is repeated on every page.

In the example below we've also added a margin to the body, so that the other regions don't overlap the body.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fo:root xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:fo="http://www.w3.org/1999/XSL/Format">
    <fo:layout-master-set>
        <fo:simple-page-master master-name="page-big" page-height="10cm" page-width="10cm">
            <fo:region-body margin-top="2cm" margin-bottom="2cm" margin-left="2cm" margin-right="2cm" />
            <fo:region-before extent="2cm"/>
            <fo:region-after extent="2cm"/>
            <fo:region-start extent="2cm"/>
            <fo:region-end extent="2cm"/>
        </fo:simple-page-master>
    </fo:layout-master-set>
    <fo:page-sequence master-reference="page-big">
        <fo:static-content flow-name="xsl-region-before">
            <fo:block>Before before before before before before before before before before before</fo:block>
        </fo:static-content>
        <fo:static-content flow-name="xsl-region-after">
            <fo:block>After after after after after after after after after after after after</fo:block>
        </fo:static-content>
        <fo:static-content flow-name="xsl-region-start">
            <fo:block>Start start start start start start start start start start start start</fo:block>
        </fo:static-content>
        <fo:static-content flow-name="xsl-region-end">
            <fo:block>End end end end end end end end end end end end end end end end end end</fo:block>
        </fo:static-content>
        <fo:flow flow-name="xsl-region-body" >
            <fo:block>Body body body body body body body body body body body body body body body body</fo:block>
        </fo:flow>
    </fo:page-sequence>
</fo:root>
```

There's a xsl:flow-map which lets us customise the mapping of flow-names to regions, so perhaps we could rename the flows as we desire.  It's not implemented by Apache FOP though.

## Making columns

Region-body can be converted into columns by adding the column-count attribute to it.  The column-gap attribute allows you to change the gap between the columns.

```xml
<fo:simple-page-master master-name="page-big" page-height="10cm" page-width="10cm">
    <fo:region-body column-count="2" column-gap="100pt" />
</fo:simple-page-master>
```

## Blocks

Here's an example throwing in a bunch of blocks into two columns and styling them.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fo:root xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:fo="http://www.w3.org/1999/XSL/Format">
    <fo:layout-master-set>
        <fo:simple-page-master master-name="page-with-heading" page-height="20cm" page-width="21cm" margin="1cm">
            <fo:region-body column-count="2" margin-top="1.5cm"/>
            <fo:region-before extent="1.25cm" /> 
        </fo:simple-page-master>
        <fo:simple-page-master master-name="normal-page" page-height="20cm" page-width="21cm" margin="1cm">
            <fo:region-body column-count="2" />
        </fo:simple-page-master>
        <fo:page-sequence-master master-name="sequence-a">
            <fo:repeatable-page-master-alternatives>
                <fo:conditional-page-master-reference master-reference="page-with-heading" page-position="first" />
                <fo:conditional-page-master-reference master-reference="normal-page" page-position="rest" />
            </fo:repeatable-page-master-alternatives>
        </fo:page-sequence-master>
    </fo:layout-master-set>
    <fo:page-sequence master-reference="sequence-a">
        <fo:static-content flow-name="xsl-region-before" >
            <fo:block font-size="16pt" font-family="Helvetica" font-weight="bold"  keep-with-next="always">
                Page Mastering
            </fo:block>
            <fo:block font-size="12pt" font-family="Times Roman" font-style="italic" keep-with-next="always">
                Marc boldly emboldens what no-one has before
            </fo:block>
        </fo:static-content>
        <fo:flow flow-name="xsl-region-body">
            <fo:block font-size="12pt" font-family="Helvetica" text-align="justify" text-indent="0.5em">  
                It's that time of the year that we need to ask ourselves "How good is my PDF formatter?"  Am I keeping up wiht the funky funky titles, indents and layout my competitors are throwing around willy-nilly, or does my content just look run-of-the-mill?
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                If you're not satisified with the answer to this question, then give Apache FOP a try.  With the power of XSL:FO and the open source software craftmanship of Apache, we can all have splendid page layouts.
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" font-weight="bold" padding-top="12pt" keep-with-next="always">
                Getting into gear
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                To get started, fire up your favourite package manager and install FOP.  Then you can work on hacking up some of your own XSL:FO inspired goodness.
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" font-weight="bold" padding-top="12pt" keep-with-next="always">
                Turning it up to Eleven
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. 
            </fo:block>
            <fo:block>
                <fo:external-graphic src="url(millie.jpg)" width="100%" content-width="scale-to-fit" content-height="scale-to-fit" />
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. 
            </fo:block>
            <fo:block font-size="12pt" font-family="Helvetica" padding-top="6pt" text-align="justify" text-indent="0.5em"> 
                <fo:inline font-style="italic">- Marc</fo:inline>
            </fo:block>
        </fo:flow>
    </fo:page-sequence>
</fo:root>
```
## Block Container

You can wrap blocks in other blocks if you want to make a group them together.

There's also fo:block-container, which you can set to a fixed position, like this:

```xml
<fo:block-container position="fixed" top="10cm" left="10cm" width="4cm" height="4cm" >
    <fo:block>  
        This block is in a fixed position.
    </fo:block>
</fo:block-container>
```

In Apache FOP (at least), blocks in overlapping positions don't seem to flow around each other, they just overwrite on top of each other.  :-(

## Images

Images can be inserted using the fo:external-graphic element.  They're considered as inline formatting which can be placed inside blocks.

The space that image uses can be set with the height and width properties.  The image won't automatically scale to fit the specified area unless you add the content-width and content-height attributes with the value scale-to-fit.

To center an image, you must the text-align property on the block that contains it.

```xml
<fo:block padding-top="6pt" text-align="center">
    <fo:external-graphic src="url(millie.jpg)" width="50%" content-width="scale-to-fit" content-height="scale-to-fit" />
</fo:block>
```

## External Links

To add an external link, use:

```xml
<fo:basic-link external-destination="http://www.google.com">http://www.google.com</fo:basic-link>
```

## Internal links

To add an internal link, use:

```xml
<fo:block font-weight="bold" font-family="Helvetica" text-align="justify" text-indent="0.5em" id="chapter1">
    Chapter 1
</fo:block>
<fo:block>
    <fo:basic-link internal-destination="chapter1">Go to chapter 1</fo:basic-link>
</fo:block>
```

## Cross references

To output a cross reference to a page number, use:

```xml
<fo:block>
    Chapter one is on page number <fo:page-number-citation ref-id="chapter1"/>
</fo:block>
```

## Follow my Leader 

Leader is the dotted line often seen in a table of contents between the chapter name and the page number.

The leader is inserted with the fo:leader tag, and can be styled with dots or a line, or just left empty.

To make the leader fill all the available width, add text-align-last="justify" to the parent block.

```xml
<fo:block font-size="16pt" font-family="Helvetica" font-weight="bold"  keep-with-next="always" border-bottom="1px solid purple" text-align-last="justify">
    XSL-FO
    <fo:leader leader-pattern="dots"/>
    Rocks
</fo:block>
```

## Automatic running header with the chapter text

If you need, you can automatically fill text in the running header with the name of the current section.

Set up the header like this.  fo:retrieve-marker is used to retrieve the name of the current section.

```xml
<fo:static-content flow-name="xsl-region-before" >
    <fo:block font-size="16pt" font-family="Helvetica" font-weight="bold" text-align-last="justify" >
        <fo:inline>XSL-FO</fo:inline>
        <fo:leader  />
        <fo:inline font-weight="normal" font-size="12pt"><fo:retrieve-marker retrieve-class-name="chapterheading"/></fo:inline>
    </fo:block>
</fo:static-content>
```

Then in the main text, set the marker using fo:marker.  

```
<fo:marker marker-class-name="chapterheading">Chapter 1</fo:marker>
```

The fo:marker needs to be inside a block.  The marker can contain formatting elements inside it (it even allows other blocks) and they will get rendered inside the running header.  The marker seems to be invisble in the main text.

## Output the current page number

```xml
<fo:page-number />
```


## Tools

* Apache FOP (open source)
* RenderX XEP (commercial)
* Antenna House (commerical)

## Benefits 

* Easily flow XML into a PDF
* Quick and dirty
* Good for things requiring only simple formatting, e.g. reports.

## Drawbacks

* Lacks control because can't run logic at rendering time.

## Useful resources

* The XSL spec provides a reference to the XSLFO elements and attributes 
    https://www.w3.org/TR/xsl/

* W3Schools XSLFO tutorial
    https://w3schools.sinsixx.com/xslfo/default.asp.htm

* RenderX XSLFO tutorial
    http://www.renderx.com/tutorial.html
