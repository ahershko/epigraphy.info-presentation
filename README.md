# Introducing CETEIcean for EpiDoc

If you've used EpiDoc much, you're probably also familiar with the way we usually turn it into something you can read with a web browser: XSLT. This process takes an XML file and outputs HTML, and in the process reformats the input as needed. [CETEIcean](https://github.com/TEIC/CETEIcean) achieves the same end goal in a different way, one that lets the web browser do the work, using JavaScript and CSS, instead of requiring that you convert the sources ahead of time. 

One way isn't necessarily better than the other. Both JavaScript and XSLT are fully-functional programming languages, and are therefore more-or-less equivalent in what they can do. But their particular strengths may work better in one situation or another. CETEIcean is particularly good in situations where you don't want to either convert all your XML ahead of time or lack the resources to do it on the fly. It also sidesteps some of the issues XSLT has: most programming languages only support the much older and less powerful XSLT 1.0; powerful as it is, it's a bit of a niche language, and you might have an easier time finding someone who can work in JavaScript than XSLT; it can be resource-intensive.

We're going to look at one scenario where CETEIcean works very well: presenting static TEI source documents using GitHub Pages. If you set it up right, you can collaboratively edit a source file on GitHub and then immediately see your changes when you push them, without having to do any extra work. I've chosen at random an example from EDH, <https://edh-www.adw.uni-heidelberg.de/edh/inschrift/HD000102>. The source is available on GitHub at <https://github.com/epigraphic-database-heidelberg/data/blob/master/inscriptions/1/1/HD000102.xml>, and the "raw" version (i.e. just the XML) at <https://raw.githubusercontent.com/epigraphic-database-heidelberg/data/master/inscriptions/1/1/HD000102.xml>.

The simplest way to use CETEIcean is to simply create an HTML file that will link to the JavaScript and CSS you need and then load (using JavaScript) the XML source you want and insert it into the document when a browser loads the HTML page. Unlike XSLT, CETEIcean works by directly converting the source file's EpiDoc XML into HTML Custom Elements, so, for example, 

```xml
<origin xmlns="http://www.tei-c.org/ns/1.0">
  <origPlace>
    <placeName type="provinceItalicRegion" ref="http://www.trismegistos.org/place/003201">Britannia</placeName>
    <placeName ref="http://www.trismegistos.org/place/003201">Vindolanda</placeName>
  </origPlace>
  <origDate notBefore-custom="0146" datingMethod="http://en.wikipedia.org/wiki/Julian_calendar">146 AD</origDate>
</origin>
```
will get turned into
```html
<tei-origin data-origname="origin" data-processed>
  <tei-origplace data-origname="origPlace" data-processed>
    <tei-placename type="provinceItalicRegion" ref="http://www.trismegistos.org/place/003201" data-origname="placeName" data-processed>Britannia</tei-placename>
    <tei-placename ref="http://www.trismegistos.org/place/003201" data-origname="placeName" data-processed="">Vindolanda</tei-placename>   </tei-origplace>
  <tei-origdate notbefore-custom="0146" datingmethod="http://en.wikipedia.org/wiki/Julian_calendar" data-origname="origDate" data-processed>146 AD</tei-origdate>
</tei-origin>
```
These are bascially the same: the element names are no longer in a namespace, but have all been lower-cased and given "tei-" prefixes, and some attributes have been added, but the structure and information content hasn't changed. The new elements are able to be registered with the web browser and to be easily styled using CSS, however.

Let's look at our example, [HD000102](https://hcayless.github.io/epigraphy.info-presentation/HD000102-1.html). It looks ok. You'll see that not all of the data is presented to us. We get nothing from the teiHeader, for example. And the information is all in document order, whereas EDH likes to put the transcription first. For the observant, there's also at least one error, which I would argue stems from an error in the source file. We'll take a look at an example that fixes some of these things, and hopefully you'll have a chance to play around with it and see what else you can do, ask questions, etc.

The source for our example is here <https://github.com/hcayless/epigraphy.info-presentation/blob/master/docs/HD000102-1.html>. The crucial bits start at [line 10](https://github.com/hcayless/epigraphy.info-presentation/blob/33ed5e1b12fdc5b4fd5b2cf2e7db20174e4d98fc/docs/HD000102-1.html#L10). First we initialize CETEIcean, and put it in a variable named `c`, then we add some behaviors. These behaviors do things like wrap `<supplied>` sections in square brackets and so on. Then we load the file from EDH's GitHub repo at [line 64](https://github.com/hcayless/epigraphy.info-presentation/blob/33ed5e1b12fdc5b4fd5b2cf2e7db20174e4d98fc/docs/HD000102-1.html#L64), insert it into the HTML file's `<body>` element, and run some code to clean up cases where, e.g. we have a `<supplied>` next to a `<gap>`, and so might get results like "sing[ulis][---]" instead of "sing[ulis---]", which is what we want. (Hint: this is where the bug resides)

I've also given you a papyrus example, [P.Fay 110](https://hcayless.github.io/epigraphy.info-presentation/p.fay.110.html) (compare <http://papyri.info/ddbdp/p.fay;;110>), the source for which is at <https://github.com/hcayless/epigraphy.info-presentation/blob/master/docs/p.fay.110.html>. In case you're wondering whether all the JavaScript is necessary, the answer is "it depends". For things as complex as the papyri.info example, where inline apparatus is getting turned into an apparatus at the bottom, for example, there's no choice. For simpler documents, you could probably get away with just CSS. Note too that adding things like brackets using CSS means they are visible, but can't be selected (and thus can't be copied, etc.). <https://hcayless.github.io/epigraphy.info-presentation/p.fay.110-1.html> is an example that does what it can with just CSS (here is the [source](https://github.com/hcayless/epigraphy.info-presentation/blob/master/docs/p.fay.110-1.html)).

Finally, [HD000102](https://hcayless.github.io/epigraphy.info-presentation/HD000102.html) ([source](https://github.com/hcayless/epigraphy.info-presentation/blob/master/docs/HD000102.html)) shows an example of re-ordering the elements in the text, and may give you ideas on how you might grab and display information in the header. You can figure out where the rendering problem is by comparing this with the original. What I'd like to invite you to do now is to create a GitHub account if you don't already have one, and to fork this repo, set up GitHub pages by going to "settings", scrolling down to "GitHub Pages", and choosing "master branch /docs folder" under "Source". That should give you a copy you can play with. (Note: You may notice a small lag between your making changes and those changes appearing on your page)

Happy hacking!

For more info and examples, see [Cayless & Viglianti 2018](https://www.balisage.net/Proceedings/vol21/html/Cayless01/BalisageVol21-Cayless01.html).
