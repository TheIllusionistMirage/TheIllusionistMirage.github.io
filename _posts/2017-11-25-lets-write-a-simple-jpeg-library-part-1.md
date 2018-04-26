---
layout: post
title:  "Let's Write a Simple JPEG Library, Part-I: The Basics"
date:   2017-11-25 07:58:59
categories: fleptic jpeg library c++11 jpeg-encoder jpeg-decoder jpeg-tutorial
comments: true
author: Koushtav Chakrabarty
---

A few weeks ago I started a small side project - implement the JPEG specification. Though naive, [my attempt](https://github.com/TheIllusionistMirage/libKPEG) was surprisingly fruitful and I managed to get a simple decoder working! But it wasn't easy; I had to read through the serious & technical specification of JPEG (really hurt my tiny brain) and spent many hours debugging stuff. So, I decided to document my process so that if someone like me ever decides to implement JPEG, but need hand holding in each and every step along the way, they have nothing to worry about! :sunglasses: (That is, imagining that someone even visits this blog). Also, this blog was kinda dead and bare, so I guess I needed more content anyway.

Before we delve into the details, let me set one thing clear: this is a tutorial for aimed at innocent n00bs like me. As a result, I tend to limit the discussion to the point of interest, while trying to explain things as nicely as I can. So, it may or may not be perfect. You're welcome to critic/suggest/discuss in the comments thread below :wink:

### What Exactly is Covered Here?

We are going to understand what JPEG is, how it works and all the gory details involved along the way. Then, in subsequent posts, we will imlpement a simple, proof of concept JPEG encoder & decoder. And for brevity's sake, we aren't going to implement **each and every feature** of JPEG. Instead, our focus will be only on sequential, baseline JPEG compression, with a channel depth of 8-bits. Since I mentioned these tuts are directed towards novice programmers, I will intentionally tend to use clearer but inefficient code from time to time. If you're reading this, I hope that you have a good learning experience!

Now, let's roll!

## What is an Image?

_A moment of time, frozen for eternity that gives us a glimpse of the scene, transcending time..._ Oh wait, just kidding, this is a lesson in programming & computer science, not poetry. :stuck_out_tongue:

In layman terms, a digital image is a 2D array whose elements are pixels. The pixels have one or more components depending upon the __color model__ the image uses. Grayscale images have just one pixel component for denoting the light intensity. RGB color model has three components per pixel, red, blue and green, which when combined yield the actual color of the pixel.
<br>

![Illustration-1]({{site.url}}/resources/images/illustration-1.jpg "An illustration of a digital image")
__Figure-1:__ _A pixel array_

<br>

## What is Image Compression?

For starters, the term _image compression_ refers to a set of steps, that an
algorithm takes in order to churn out a modified version of the input image in such a way that:
* the __size__ (size on disk, not to be consfused with resolution or image dimensions) of the output image is as small as possible.
* there is __minimum or no loss__ of visual __quality__ and image data.

<!--break-->

These are the two main factors that determine _how good_ the image compression technique is. It's important to keep in mind that like everything else, there's no such thing as the _ultimate image compression algorithm_. Which algorithm to use is totally dependent upon the need of the user. For example, JPEG, which is the focus of this post, is good for compressing raw images of natural scenes, which have smooth, gradual variations of color, and it is unsuitable for images with sharp contrasts. Also, JPEG compression is _lossy_, meaning that depending upon the compression quality, some amount of the original imagedata is thrown away, forever, to reduce the file size. On the other hand, there's PNG, which is a _lossless_ mode of image compression, meaning, it is guaranteed that all image data from the input can be restored back when decoding an image compressed using PNG. But since it has to store _all_ original image data, it results in file sizes that are typically larger than the ones done using JPEG. Both JPEG and PNG work on two dimensional images. There are some image formats like DICOM, which can store accurate 3D images. Hence, DICOM is extremely popular in the medical community for storing medical data of CT scans, MRIs, etc. Since accuracy and correctness are crucial in this case, the corresponding file sizes are much larger than JPEG or PNG.

So, it's all a matter of trade offs and necessities.


## Getting Started With JPEG

JPEG stands for __Joint Phographic Experts Group__, which develops this image compression technique and specifies the official standard, [ITU-T.81](https://www.w3.org/Graphics/JPEG/itu-t81.pdf), which is implemented by all JPEG libraries. At this point I would like to add that JPEG is the compression technique itself: it just specifies the steps for encoding and decoding. The actual file format for storing the images compressed using JPEG is specified in the JFIF and EXIF standards. They define how the compressed image data is to be stored in a file for exchanging and storing the compressed images.

## The Heart of JPEG Compression

Now we're perhaps at the most important part of this post. Now, I'm going to describe _why_ JPEG achieves such great compression. Instead of dealing with the pixels in the the RGB color space, JPEG converts them to the \\(YC_bC_r\\) color space. The \\(YC_bC_r\\) color space has three components per pixel: the luminance, or the brightness of the pixel, \\(Y\\), the amount of blue in the pixel, \\(C_b\\), and, the amount of red in the pixel, \\(C_r\\). The \\(Y\\) component is called the __luma__ component and \\(C_b\\) and \\(Cr\\) are known as the __chroma__ components.

The benefit in doing this color space conversion is because of how human eyes work. The retina of a eye has two major types of photoreceptors, _rods_ and _cones_. The rods are sensitive to the lumosity of light and the cones to the color. And the number of rods in the retina massively outnumber the number of cones. As a result, a human eye is able to tell apart light intensity better than they can discriminate between colors. So, our eyes are more sensitive of small variations of low frequency over a large area than to high frequency variations in the same area. As we will later see, that JPEG drops the high frequncy data of the image and stores only the small frequency variations with greater accuracy, resulting in reduced file size. Apart from these two things, JPEG also employs things like Huffman encoding as we shall later see, to achieve even more reduction!

So, in a nutshell, the JPEG exploits the fact that the human eye isn't perfect!

## 'Nuf, Talking! Show Me the Steps Already!

### Step #1: Color space transformation from RGB to Y-Cb-Cr

Like we discussed previously in the post, an image is nothing but a 2D array whose elements are pixels. Each pixel has different components depending upon the color model of the image. Mostly, this 2D array of pixels that is fed to the JPEG algorithm uses the RGB color space, meaning, the there are three components: Red (`R`), Green (`G`) and Blue (`B`). To convert the pixel to its corresponding representation in the Y-Cb-Cr color model, we use the following relations:

$$Y  = 0.299 * R + 0.587 * G + 0.114 * B \\
Cb = -0.1687 * R - 0.3313 * G + 0.5 *B + 128 \\
Cr = 0.5 * R - 0.4187 * G - 0.0813 * B + 128 \\$$

### Step #2: Chroma Subsampling

We already know that the human eye is better at dealing with the brightness or lumosity than the colors. So, the separation of the luma (\\(Y\\)) and chroma (\\(C_b\\) & \\(C_r\\)) components enables us to _throw away_ some of the information relating to the color, without significant loss of visual quality of the image that can be distinguished by the naked eye. So, what JPEG says is that, _"take only `x` \\(C_b\\) & \\(C_r\\) values for every `y` of them"_. Basically, we keep the luma component of each and every pixel, but only keep the chroma components once for every few pixels.

### Step #3: Level Shifting

Since JPEG allows only 8 bits per channel, the values of the \\(Y\\), \\(C_b\\) and \\(C_r\\) components have the range 0-255 for their values. Before the next step in the encoding process can be applied, these values have to be level shifted to center them at 0. Hence, 128 is subtracted from the all the component values of each pixel.

### Step #4: Discrete Cosin Transform (DCT) of Minimum Coded Units (MCU)

Now begins the fun part of the compression procedure. Instead of just processing the \\(YC_bC_r\\) pixels directly, they are converted to their spatial frequency values. First, The image is subdivided into 8x8 pixel blocks, called __minimum coded units__ or __MCUs__. In case the image dimensions aren't multiples of 8, the image edges are modified to make them the nearest multiple of 8. For example, if the image is 317x457, its size is converted to 320x464. This _stretching_ is usually done by just copying the rightmost column of pixels for extending horizontally, and the last row of pixels for extending vertically.  Each of these 8x8 blocks can be represented by a combination of horizontal and vertical frequencies. For a 8x8 block, the possible combinations of the spatial frequencies are as follows:

![Illustration-2]({{site.url}}/resources/images/illustration-2.jpg "Spatial frequency domain")
__Figure-2:__ _Frequencies of the MCU. Each MCU is a combination of these frequencies. The ones to the top-left have lower frequencies than the ones closer to bottom-right. (__Image borrowed from Wikipedia__)_

Now, let's take a look at the DCT formula that we are going to use:

$$\mathsf{FDCT}: S_{vu} = \frac{1}{4} C_u C_v \sum_{x=0}^{7} \sum_{y=0}^{7} s_{yx} \cos{\frac{(2x+1)u\pi}{16}} \cos{\frac{(2y+1)v\pi}{16}}$$

$$\mathsf{IDCT}: s_{yx} = \frac{1}{4} \sum_{u=0}^{7} \sum_{v=0}^{7} C_u C_v S_{vu} \cos{\frac{(2x+1)u\pi}{16}} \cos{\frac{(2y+1)v\pi}{16}}$$

where,<br>
\\(u\\) is the horizontal spatial frequency<br>
\\(v\\) is the vertical spatial frequency<br>
\\(C_u,C_v = 
\cases{
\frac{1}{\sqrt{2}},  & \text{if } u,v = 0\cr
0, & \text{otherwise}
}\\), are the normalizing factors<br>
\\(s_{yx}\\), is the pixel value at coordinates \\(\(x,y\)\\), and,<br>
\\(S_{vu}\\), is the DCT coefficient at coordinates \\(\(u,v\)\\)

The first equation, _Forward DCT_, is used when __encoding__ an image from an uncompressed format to the JPEG format and the second equation, _Inverse DCT_, is used when __decoding__ the byte stream in a JFIF/EXIF file, that was compressed previously using JPEG encoding.

Let's consider an example MCU for a channel (the example channel can be any one of Y, Cb or Cr):

$$
\begin{bmatrix}
64 & 56 & 56 & 57 & 70 & 84 & 84 & 59\\
66 & 64 & 35 & 36 & 87 & 45 & 21 & 58\\
66 & 66 & 66 & 59 & 35 & 87 & 26 & 104\\
35 & 75 & 76 & 45 & 81 & 37 & 34 & 35\\
45 & 96 & 125 & 107 & 31 & 15 & 107 & 90\\
88 & 89 & 88 & 78 & 64 & 57 & 85 & 81\\
62 & 59 & 68 & 113 & 144 & 104 & 66 & 73\\
107 & 121 & 89 & 21 & 35 & 64 & 65 & 65
\end{bmatrix}
$$

Before applying FDCT, we have to level shift the values (by subtracting 128 from each element), resulting in the following MCU:

$$
\begin{bmatrix}
-64 & -72 & -72 & -71 & -58 & -44 & -44 & -69\\
-62 & -64 & -93 & -92 & -41 & -83 & -107 & -70\\
-62 & -62 & -62 & -69 & -93 & -41 & -102 & -24\\
-93 & -53 & -52 & -83 & -47 & -91 & -94 & -93\\
-83 & -32 & -3 & -21 & -97 & -113 & -21 & -38\\
-40 & -39 & -40 & -50 & -64 & -71 & -43 & -47\\
-66 & -69 & -60 & -15 & 16 & -24 & -62 & -55\\
-21 & -7 & -39 & -107 & -93 & -64 & -63 & -63
\end{bmatrix}
$$

Now, using the above formula for calculating FDCT, we get:

$$
\begin{bmatrix}
-477.63 & 24.47 & 6.93 & -25.49 & -6.13 & -27.83 & -0.57 & 6.89\\
-65.84 & -22.93 & -4.66 & 15.25 & 16.3 & -12.69 & 12.2 & -7.67\\
7.72 & -5.29 & 14.03 & 74.8 & 3.88 & -15.81 & 13.35 & -1.86\\
44.54 & -25.13 & -24.48 & -14.24 & 3.35 & 47.02 & -33.93 & 13.8\\
-13.63 & 22.85 & 22.83 & -31.1 & -53.13 & 22 & -22.31 & 20.27\\
11.12 & -32.74 & -64.88 & 40.32 & 17.61 & -11.14 & 11.72 & -2.59\\
10.47 & 6.93 & 62.85 & -8.64 & -30.16 & 17.07 & 26.22 & -22.7\\
42.47 & -31.38 & -4.03 & -35.84 & 0.41 & 29.19 & 10.36 & -27.19
\end{bmatrix}
$$

_(Note that if we take the IDCT of the above MCU and then restore the level shift, we get back the original MCU.)_

### Step #5: Quantization

After performing the spatial frequency transformation, it can be seen that the magnitude of the top left coefficients are rather large, and that of the first coefficient is the largest. The top-left entries corsspond to low frequency changes, that we are good at observing, and as we move towards the bottom right, the changes are less obvious to our eyes.

And like we already discussed, this flaw is expoloited by JPEG by throwing away a majority of the high frequency variations. Also, if you look more closely, you will observe that if we somehow rounded off these high frequency coefficients to zero, it wouldnot have _much effect_ on the overall visual quality. Hence, what we do for each transformed MCU is that we __quantize__ them. A __quantization matrx__ is used for the quantization, which consists of 8x8 coefficients, denoting _"how much"_ each coefficient has an importance in the overall MCU. The higher the effect, the lower the value of constant used for quantizing it.

Now, to illustrate this principle on our example, we use the quantization table specified in the JPEG standard **_(ITU-T.81, Annex-K, Table-K.1, Pg.-143)_** for luminance with quality 50%:

$$
\begin{bmatrix}
16 & 11 & 10 & 16 & 124 & 140 & 151 & 161\\
12 & 12 & 14 & 19 & 126 & 158 & 160 & 155\\
14 & 13 & 16 & 24 & 140 & 157 & 169 & 156\\
14 & 17 & 22 & 29 & 151 & 187 & 180 & 162\\
18 & 22 & 37 & 56 & 168 & 109 & 103 & 177\\
24 & 35 & 55 & 64 & 181 & 104 & 113 & 192\\
49 & 64 & 78 & 87 & 103 & 121 & 120 & 101\\
72 & 92 & 95 & 98 & 112 & 100 & 103 & 199
\end{bmatrix}
$$

_(Notice that the constants towards the upper-left are significantly smaller than the ones to the bottom-right.)_

Quantization is done using the formula below:

$$
A^{\prime} = \sum_{i=0}^{7} \sum_{j=0}^{7} round(\frac{a_{i,j}}{Q_{i,j}})
$$

where,<br>
\\(A^{\prime}\\) is the quantized MCU<br>
\\(Q\\) is the quantization matrix<br>

Hence, the quantized DCT coefficients become:

$$
\begin{bmatrix}
-30 & 2 & 1 & -2 & 0 & 0 & 0 & 0\\
-5 & -2 & 0 & 1 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0\\
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0
\end{bmatrix}
$$

As you can see, after quantization, much of the bottom-right entries have become zero. And this is the key to the next step of entropy coding.

### Step #6: Entropy Coding

The entropy coding step consists of grouping coefficients of similar fequencies together by ordering the 64 coefficients in the 8x8 MCU in a zig-zag manner, and then performing run-length encoding on the 64 element vector so obtained. Then this run-length encoded data is encoded using Huffman encoding to furthur reduce the amount of bytes they take. This step is better explained with a practical example rather than just words.

The order of traversal of elements in the zig-zag manner is as follows:

$$
\begin{bmatrix}
1 & 2 & 6 & 7 & 15 & 16 & 28 & 29\\
3 & 5 & 8 & 14 & 17 & 27 & 30 & 43\\
4 & 9 & 13 & 18 & 26 & 31 & 42 & 44\\
10 & 12 & 19 & 25 & 32 & 41 & 45 & 54\\
11 & 20 & 24 & 33 & 40 & 46 & 53 & 55\\
21 & 23 & 34 & 39 & 47 & 52 & 56 & 61\\
22 & 35 & 38 & 48 & 51 & 57 & 60 & 62\\
36 & 37 & 49 & 50 & 58 & 59 & 63 & 64 
\end{bmatrix}
$$

Let's get back to the example MCU we were encoding. It's zig-zag traversal is:

$$
-30, 2, -5, 0, -2, 1, -2, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,\\ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
$$

It is important to state here that the first coefficient, -30, also called the DC coefficient will not be encoded using run-length coding like the remaining 63 coefficients, also called the AC coefficients. It's hard to explain _why_ immediately, but it will become clear in the subsequent steps. For now, just bear in mind that the run-length encoding step applies only to the 63 AC coefficients.

As it's obvious, there are a lot of zeros. Instead of using this 64 element stream and to save space, we just store the number of zeros preceding a non-zero value. For example, there are no zeros before the occurence of the first 2. So, it gets encoded as \\((0,2)\\). Similarly, there are two zeros before the occurrence of the second 1, so it gets encoded as \\((2,1)\\).

The remaining coefficients after the last non-zero value, i.e., 1, can be simply stored by having a convention that says _"the rest are all zeros"_. And the convention that is used for this is the pair \\((0,0)\\), or the EOB. It's also worth noting here that if the number of zeros before a non-zero value is more than 16, for instance, say, 23, we have can't just store 23 in the pair.

\\(( 23 \text{ zeros}, 58)\\) gets stored as \\((15, 0), (8, 58)\\) and not \\((23, 58)\\). The pair \\((15, 0)\\), just like \\((0, 0)\\), is a special value, used to denote sixteen consecutive zeros. But why do this? The reason behind this restriction becomes more clear in the next step of Huffman encoding these AC coefficients.

Hence, the run-length encoding of the AC coefficients that is obtained after this step is:

$$(0, 2), (0,-5), (1, -2), (0,1), (0, -2), (2, 1), (3, 1), (3, 1), (0,0)$$

#### Huffman Coding of the Run-Length Encoded Data

Huffman coding is a lossless data compression technique that can encode a series of input symbols (bytes, characters, etc.) into variable length codes where the frequently used symbols in the input are assigned shorter codes and the less used ones are assigned longer codes. The variable length codes are obtained by analyzing the frequency or probability of the occurence of each symbol in the input stream. Also, Huffman coding guarantees that no code is a prefix of a longer code (that is, you can't have two codes in a Huffman coding scheme that are 111 and 1110). The exact steps for performing the analysis of frequencies and constructing the Huffman table for encoding can be found in the JPEG specification. In practice, the entries in the Huffman table are stored in the JFIF/EXIF file, from which the Huffman tree of symbols are constructed. But we are going to use the sample tables provided in the JPEG specification _(**ITU-T.81, Annex-K, Sections K.3.1 & K.3.2, Page- 149**)_ for the remainder of these series of posts.

#### Encoding the DC Coefficient

Like I said in the last section, the DC coefficient is encoded a bit differently from the AC coefficients. The DC coefficient of an MCU has the largest magnitude and defines the overall value for that MCU. Also, the DC coefficients of consecutive MCUs are related; they only change gradually with respect to the each other. So, instead of encoding the entire magnitude, what JPEG does it stores the __difference__ between the DC coefficients of corresponding MCUs (Note that the difference between the DC coefficients is calculated for each channel).

\\(DC_{i+1} = DC_i + \text{difference}\\)

The DC coefficient for the \\({i+1}^{th}\\) block is obtained by adding the \\(\text{difference}\\) with the DC coefficient of the \\(i^{th}\\) block. The \\(\text{difference}\\) for the \\(0^{th}\\) MCU is always 0.

Now, every \\(\text{difference}\\) value has a value category, which denotes the minimum number of bits needed to store the difference value's bit representation. Don't confuse _bit representation_ of a value with it's binary representation; instead take a look at the following difference category table and consider the DC coefficient (-30) from our example:

```txt
              Range        DC Difference        Bits for the value
                              Category
--------------------------------------------------------------------

                0                0                   -

              -1,1               1                  0,1

           -3,-2,2,3             2              00,01,10,11

     -7,-6,-5,-4,4,5,6,7         3   000,001,010,011,100,101,110,111

       -15,..,-8,8,..,15         4       0000,..,0111,1000,..,1111

      -31,..,-16,16,..,31        5     00000,..,01111,10000,..,11111

      -63,..,-32,32,..,63        6 000000,..,011111,100000,..,111111

     -127,..,-64,64,..,127       7                   .

    -255,..,-128,128,..,255      8                   .

    -511,..,-256,256,..,511      9                   .

   -1023,..,-512,512,..,1023     A                   .

  -2047,..,-1024,1024,..,2047    B                   .

  -4095,..,-2048,2048,..,4095    C                   .

  -8191,..,-4096,4096,..,8191    D                   .

 -16383,..,-8192,8192,..,16383   E                   .

-32767,..,-16384,16384,..,32767  F                   .
```

From the table, we can see that __-30__ lies in the range __\\(-31 \text{ to } 31\\)__, so it belongs to category __5__, and, it's bit representation is __00001__, which makes the \\(\text{difference}\\) to be coded as \\((5, 00001)\\).

Now, assuming our example MCU is from the luminance(\\(Y\\)) channel, let's take a look at the sample Huffman codes from the JPEG specification for encoding DC luminance coefficients _(**ITU-T.81, Annex-K, Section-K.3.1, Table-K.3, Page- 149**)_:

```txt
Category   Code length   Code word
----------------------------------

    0           2            00
   
    1           3           010
   
    2           3           011
   
    3           3           100
   
    4           3           101
   
    5           3           110
   
    6           4          1110
   
    7           5         11110
   
    8           6        111110
   
    9           7       1111110
   
   10           8      11111110
   
   11           9     111111110
```

From the table, we can see that the Huffman code for category 5 is \\(110\\). Therefore, the final bit stream that will be written to the JFIF/EXIF file is __11000001__.

#### Encoding the AC Coefficients

Now that we know how the DC coefficients are encoded, it's time to take a look at the AC coefficients. The run-length encoding of the AC coefficients that was obtained previously:

$$(0, 2), (0,-5), (1, -2), (0, 1), (0, -2), (2, 1), (3, 1), (3, 1), (0,0)$$

Each of these pairs of AC coefficients is inspected for the number of zeros preceding it, it's value category, and finally the bit representation for the coefficient. The information for the number of preceding zeros, called the __zero-run__ is denoted using 4 bits (__RRRR__) and so is the value category of the AC coefficient (__SSSS__). This is precisely the reason why JPEG limits that the number of zeros in the run-length coding should not exceed 16 as 4 bits can represent a maximum value of 16.

So, each AC coefficient now looks like this: \\((RRRRSSSS, \text{Huffman code for coefficient})\\). Assuming that the AC coefficents are for the luminance channel, let's encode them one by one. The Huffman coding table that we will use can be found in the JPEG specification _(**ITU-T.81, Annex-K, Section-K.3.1, Table-K.5, Page- 150**)_ and you are encourage to consult the same as the table is too long to be displayed here.

Let's start with the first coefficient, \\((0,2)\\). The zero-run of before 2 is 0 and the value category is 2. So, \\(RRRR=0\\) and \\(SSSS=2\\). From Table-K.5 mentioned above, it can be seen that the Huffman code for \\((RRRR=0,SSSS=2)\\) is \\(01\\). And from the value category table (the first table in last subsection), we find that the bit representaion for \\(2\\) is \\(10\\). Hence, bit stream that will be written to the JFIF/EXIF file is __0110__.

Taking another example, for the second coefficient \\((0,-5)\\), \\(RRRR=0\\) and \\(SSSS=3\\), which makes the Huffman code to be \\(100\\). The bit representation of -5 is \\(010\\), which makes the bit stream for this coefficient __100010__.

Similarly, the bit streams for the other coefficients are calculated:<br> 
\\((1, -2)\\) : __11100101__<br>
\\((0, 1)\\) : __001__<br>
\\((0, -2)\\) : __0101__<br>
\\((2, 1)\\) : __110111__<br>
\\((3, 1)\\) : __1110101__<br>
\\((3, 1)\\) : __1110101__<br>
\\((0, 0)\\) : __1010__ (EOB)<br>

Combining all the AC coefficients for the this MCU, the final bit stream turns out to be __0110 100010 11100101 001 0101 110111 1110101 1110101 1010__.

## Summary

To sum up what we covered in this post:
* What are digital images are and image compression.
* Brief tour of the JPEG
* The theoretical foundations for understanding the encoding/decoding process of JPEG
* Brief overview of each step involved in JPEG encoding/decoding.

In the next part, we will get our hands dirty and shall start writing a JPEG decoder in C++ (yay! :smile:). You are free to follow along in your language of choice as understanding the steps is all that's required.
