# Tex to SVG with MathJax In Node.js

## What is working so far
Right now the piece of code that gives me **something** is:
```JavaScript
require('mathjax').init({
    loader: {load: ['input/tex', 'output/svg']}
  }).then((MathJax) => {
    const svg = MathJax.tex2svg('\\frac{1}{x^2-1}', {display: true});
    console.log(MathJax.startup.adaptor.outerHTML(svg));
  }).catch((err) => console.log(err.message));
```

But the more generic form is:
```JavaScript
require('mathjax').init({ ... }).then((MathJax) => { ... });
```
 I've taken this snippets from https://npm.io/package/mathjax.
 
 Currently the first piece of code on top only results in SVG code printed in the terminal and has to be copied into a text document with .svg extension.
 And this is what the website says about the second snippet:
 > where the first `{ ... }` is a MathJax configuration, and the second `{ ... }` is the code to run after MathJax has been loaded.



This code fro a different website also works by outputting SVG code into the terminal not an SVG file:
```JavaScript
var mjAPI = require("mathjax-node");

mjAPI.config({MathJax: {SVG: {font: "TeX"}}});
mjAPI.start();

mjAPI.typeset({
  math: "\sin^2+\cos^2=1",
  format: ("TeX"),
  svg:true,
  useFontCache: false,
  ex: 6, width: 100,
}, function (data) {
  if (!data.errors) {console.log(data.svg)}
});
```

The code is from https://www.debugcn.com/en/article/29137786.html.



After modifying the first snippet, the following code does produce a file but with a small unnecessary part at the beginning:

```JavaScript
require('mathjax').init({
    loader: {load: ['input/tex', 'output/svg']}
  }).then((MathJax) => {
    // Enter LaTex code as a string of the form var LatexCode = some code in this line
    var LatexCode = 'x^2+y^2=z^2';
    const svg = MathJax.tex2svg(LatexCode, {display: false});

    fs = require('fs');
    // Enter a string in the form var uniqueName = some name in this line
    var uniqueName = 'test';
    var fileName = uniqueName + ".svg";
    fs.writeFile(fileName, MathJax.startup.adaptor.outerHTML(svg), function (err) {
    if (err) return console.log(err);

});
  }).catch((err) => console.log(err.message));
```

<div style="page-break-after: always; visibility: hidden">
\pagebreak
</div>

Okay, now the code does exactly what I want:
```JavaScript
require('mathjax').init({
  loader: { load: ['input/tex', 'output/svg'] }
}).then((MathJax) => {
  // Enter LaTex code as a string of the form var LatexCode = some code in this line
  var LatexCode = 'i \\hbar \\frac{\\partial}{\\partial t}\\Psi(\\mathbf{r},t) = \\hat H \\Psi(\\mathbf{r},t)'; // This is an example and has to be removed.
  const svg = MathJax.tex2svg(LatexCode, { display: false });
  // Saving svg code to a file.
  fs = require('fs');
  // Enter a string in the form var uniqueName = some name in this line
  var uniqueName = 'test'; // This is an example and has to be removed.
  var fileName = uniqueName + ".svg";
  // Converting the output svg to a string.
  var svgAsString = MathJax.startup.adaptor.outerHTML(svg);
  // Removing code that is unnecessary at beginning and end of string.
  // The number of characters can change depending on setting used so 
  // if something goes wrong it may be the number of chars being removed.
  var finalString = svgAsString.substring(41, svgAsString.length - 16);
  fs.writeFile(fileName, finalString, function (err) {
    if (err) return console.log(err);
  });
}).catch((err) => console.log(err.message));
```
***Remember to  escape the forward slash in LaTex formulas!***

## Does not work with Blender sadly
So yeah, the svg produced by MathJax is not compatible with Blender. Blender
does not recognize the paths and shows nothing. Seems like blender requires that the svg be purely paths. And I don't know exactly what MathJax produces.
On a good note, I found out that Dvisvgm comes together with MikTex. I don't know since when this is the case but this means I only have to install MikTex.
 
 ### Paths for Blender
 In the Docs it says that Blender can only import SVGs with paths and no rasterized elements. My next step is to see if I can use a DVI intermediate instead of a PDF intermediate. I have tried it before but for some reason it did not work. I wonder if it is because that conversion does not use paths only.
 
 
I made a small change but it still doesn't work in Blender.

   ```JavaScript
require('mathjax').init({
  loader: { load: ['input/tex', 'output/svg'] },
  SVG: { useGlobalCache: false, useFontCache: false }
}).then((MathJax) => {
  // Enter LaTex code as a string of the form var LatexCode = some code in this line
  var LatexCode = 'i \\hbar \\frac{\\partial}{\\partial t}\\Psi(\\mathbf{r},t) = \\hat H \\Psi(\\mathbf{r},t)'; // This is an example and has to be removed.
  const svg = MathJax.tex2svg(LatexCode, { display: false });
  // Saving svg code to a file.
  fs = require('fs');
  // Enter a string in the form var uniqueName = some name in this line
  var uniqueName = 'test'; // This is an example and has to be removed.
  var fileName = uniqueName + ".svg";
  // Converting the output svg to a string.
  var svgAsString = MathJax.startup.adaptor.outerHTML(svg);
  // Removing code that is unnecessary at beginning and end of string.
  // The number of characters can change depending on setting used so 
  // if something goes wrong it may be the number of chars being removed.
  var finalString = svgAsString.substring(41, svgAsString.length - 16);
  fs.writeFile(fileName, finalString, function (err) {
    if (err) return console.log(err);
  });
}).catch((err) => console.log(err.message));
   ```
   