---
layout: project
title: ASCII Art Generator
permalink: ascii-art-generator/
---

ASCII Art Generator is a lightweight image to ASCII art converter, without reducing image resolution.
<br>

### Screenshots

**The Joker - Batman's greatest adversary**

*(Image copyright: http://antagonists.wikia.com/wiki/Joker_(The_Dark_Knight))*

Original input: <br>
[![Joker-1]({{site.url}}/resources/images/joker-dark-knight-thumb.jpg "Original image")]({{site.url}}/resources/images/joker-dark-knight.jpg)

Output: <br>
[![Joker-2]({{site.url}}/resources/images/joker-dark-knight-ascii-art-thumb.jpg "Output")]({{site.url}}/resources/images/joker-dark-knight-ascii-art.jpg)

Closeups: <br>
[![Joker-3]({{site.url}}/resources/images/joker-dark-knight-closeup-I-thumb.jpg "Output")]({{site.url}}/resources/images/joker-dark-knight-closeup-I.jpg)
[![Joker-4]({{site.url}}/resources/images/joker-dark-knight-closeup-II-thumb.jpg "Output")]({{site.url}}/resources/images/joker-dark-knight-closeup-II.jpg)
<br>

**And the eternal embodiment of gorgeousness, Scarlett Johanssen :heart:**

Original input: <br>
[![Scarlett-1]({{site.url}}/resources/images/scarlett-johansson-thumb.jpg "Output")]({{site.baseurl}}/images/scarlett-johansson.jpg)

Output: <br>
[![Scarlett-2]({{site.url}}/resources/images/scarlett-johansson-ascii-art-thumb.jpg "Output")]({{site.url}}/resources/images/scarlett-johansson-ascii-art.jpg)

Closeups: <br>
[![Scarlett-3]({{site.url}}/resources/images/scarlett-johansson-closeup-I-thumb.jpg "Output")]({{site.url}}/resources/images/scarlett-johansson-closeup-I.jpg)
[![Scarlett-3]({{site.url}}/resources/images/scarlett-johansson-closeup-II-thumb.jpg "Output")]({{site.url}}/resources/images/scarlett-johansson-closeup-II.jpg)
[![Scarlett-3]({{site.url}}/resources/images/scarlett-johansson-closeup-III-thumb.jpg "Output")]({{site.url}}/resources/images/scarlett-johansson-closeup-III.jpg)
<br>

### How Does It Work?

ASCII Art Generator works in three steps:
 * Get the 2D array of pixels of the image (done using [SFML](http://sfml-dev.org))
 * For each pixel `(i,j)`, assign a _rough_ shade by:
   * Calculating the average [RGB value](https://en.wikipedia.org/wiki/RGB_color_model) of pixel `(i,j)`
   * Assign the _brightness_ of the pixel `(i,j)`
   
In the RGB colour model, every pixel in an image is represented by combining red,
green and blue colours. The average RGB colour of the pixel is calculated as:

`int avgColor = (pixel[i][j].red + pixel[i][j].green +
pixel[i][j].blue) / 3;`

Then the brightness of the pixel is determined. A brigher pixel will be _less darker_ and
vice versa. Hence, characters like `'`, `"`, `/`, etc. represent the _brightest_ shades
while characters like `@`, `#`, `$`, etc. represent the darkest ones. The entire ASCII
pixel set this program uses is as shown (ASCII pixels are ordered as per their increasing
brightness):

`'@', '#', '9', '8', '&', '6', '5', '4', '0', '3', '7', '?', 'o', '>', '<', '2', ']', '[', '\\', '/', '1', '!', ';', ':', '+', '*', '~', '-', '\'', '"', '\'', ',', '.', ' '`

Now, the higher the value of `avgColor`, more the chances that the pixel is bright and
will be represented by a ASCII pixel nearer the end of the ASCII pixel set. So, the brightness is
determined in the next step as:

`int brightness = avgColor / (Max. brightness value / Max. brightness value available);`

In RBG color model, the maximum brightness possible is `255`, i.e., the maximum possible red,
green or blue value. And in our ASCII pixel set, we have `34` pixels. Hence, maximum
brightness value available is `34`.

Now the corresponding ASCII pixel for `pixel[i][j]` is given by `ASCII_pixel[brightness]`.

_NOTE : The above code lines just represent how the logic works. In ASCII Art Generator's
source, the actual code do do this is different._
<br>

### Source, Building & Running

You can find ASCII Art Generator's source [here](https://github.com/TheIllusionistMirage/ASCII-Art-Generator).

Build & run:
```
$ g++ main.cpp -std=c++11 -lsfml-graphics -lsfml-system -o ascii-art-gen && ./ascii-art-gen
```
