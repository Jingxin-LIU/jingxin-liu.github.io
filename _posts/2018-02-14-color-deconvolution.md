---

layout: post

title:  "Color Deconvolution"

date:   2018-02-09

excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."

tags:

- IHC
- DAB
- Color Deconvolution
- Color
---

**Color Deconvolution** is perhaps the most widely used method for stain color unmixing in digital HISTOPATHOLOGY images.  It is proposed by Ruifrok et al. in 2001[^1].

## Theory

For digital histopathology image, we assume it is captured by a video microscopy system with a RGB camera, assuming that gray levels in each of the RGB channels are linear with brightness or transmission $T$, with $T$ being $\frac{I}{I_0}$. This assumption is reasonably accurate for CCD cameras with a gamma of 1.0. 

The color deconvolution approach is proposed following the Lambert-Beer's Law [^2], which relates the attenuation of light to the properties of the material through which the light is travelling. By the definition the **transmittance** of material sample is related to its **optical depth $\tau$** and to its **absorbance $A$**  as

$$T = \frac{I}{I_0} = e^{-\tau} = 10^{-A}$$

Where $I_0$ is the intensity of light entering the material sample, and $I$ is the intensity of light detected after passing the material sample. 

In color deconvolution related works, $I \in R^{c \times n}$ is the matrix of RGB intensites, where $c=3$ for RGB channels, and $n = $ number of pixels. Let $M\in R^{c \times r}$ be the stain color appearance matrix whose columns represent color basis of each stain such that $r$ is the number of stains ($r=3$ for color deconvolution), and $S\in R^{r \times n}$ be the stain density maps, where the rows of which represent the concentration of each stain, then $I$ can be written as follows [^3]:

$$I = I_0 e^{-MS}$$

$I$ should be first converted to **Optical Density (OD)** values:

$$OD = -log_{e} (\frac{I}{I_0})$$

So, **optical density**  is equal to **optical depth**. In all color deconvolution related works, we would use the above equation to calculate the OD value. However, it is note that some literatures gives different definition that 

> *the optical density is often said to be identical with the absorbance*[^4] [^5].

$I_0 = [255, 255, 255]$ for a typical 8 bit RGB camera. Both $I$ and $I_0$ should be normalized to [0,1] before transform RGB to OD. So the above equation becomes,

$$OD = -log_{e} (I).$$

Hence, 

$$OD = MS.$$

Consequently, color deconvolution is given an observation matrix $OD$, and empirically estimated $M$, to find stain intensity map matrix $S$:

$$S = M^{-1} OD.$$

How to get the stain color apperance matrix $M$?

![ink img](http://www.mecourse.com/landinig/software/cdeconv/hd.png)
<center>Fig.0: Example DAB-H image.</center>
The above image is a typical DAB-H stained digital IHC image, which is also used in [Colour Deconvolution - eCourse](http://www.mecourse.com/landinig/software/cdeconv/cdeconv.html). So, there are two different stains (colors): *He* (blue) and *DAB* (brown). The RGB vectors for *He* and *DAB* could be [89, 75, 182] and [187,109,57] respectively as shown below



So the RGB stain matrix is $M_{RGB} = [89, 75, 182;  187,109,57;  255,255,255]$ since there is no the third stain. Then $M$ is calculated as:

$$M = 1 - M_{RGB}/255$$

$M$ should be normalized to achieve correct balancing of the absorbtion factor for each separate stain, by divide each stain vector by its total length:

$$\hat{M}_{R1} = M_{R1} / \sqrt{ M_{R1}^2 + M_{G1}^2 +M_{B1}^2}$$

$$\hat{M}_{G1} = M_{G1} / \sqrt{ M_{R1}^2 + M_{G1}^2 +M_{B1}^2}$$

$$\hat{M}_{R2} = M_{R2} / \sqrt{ M_{R2}^2 + M_{G2}^2 +M_{B2}^2}$$
, etc.

If there is no the third stain, the third component RGB values should be assigned as:

$$M_{R3} = \begin{cases} 0 & {if (M_{R1}^2 + M_{R2}^2)>1} \\ \sqrt{1 - (M_{R1}^2 - M_{R2}^2)}  &{else}\end{cases}$$

and use same equation to calculate $M_{B3}$ and $M_{G3}$. The new assigned values should also be normalized.

So, 

$$ S = \hat{M}^{-1} OD. $$

Finally, $S$ should be transformed back to RGB space by:

$$ S_{RGB} = e^{-S}$$

The $S_{RGB}$ is finally image, and each channel represents the corresponding stain.

In summary, the whole process of color deconvolution is

### Color deconvolution algorithm

**Input: $I$, $M$**

1. normalize $M$, and calculate $M^{-1}$
2. Convert $I$ to $OD$
3. Calcuate $S$
4. Convert $S$ to RGB space as $S_{RGB}$



## Implementation

There are many implementations:

1. The offical implement is a **color deconvolution plugin** in *ImageJ* and *Fiji* using JAVA ([code](http://www.mecourse.com/landinig/software/colour_deconvolution.zip)).

2. **Stain Normalization Toolbox** by Warwick University Tissue Image Analytics Group using MATLAB ([website](https://warwick.ac.uk/fac/sci/dcs/research/tia/software/sntoolbox/)).

3. MATLAB implementation by Jakob Nikolas Kather on GitHub([website](https://github.com/jnkather/ColorDeconvolutionMatlab)).

etc.



## Drawbacks

There are two main drawbacks on color deconvolution.

#### 1. The results highly depend on the predefined stain color apperance matrix $M$  

Color deconvolution requires very accurate stain color apperance matrix $M$. Although the offical implementation provide a number of "build in" stain vectors, those vectors are not able to deal with the inconsistency with same stains. The figure shows below has two histopathology slides of melanomas, both stained with hematoxylin and eosin, but with drastically different appearances. Both images were obtained by scanning the slides at 20X using an Aperio Scanscope [^6].


​			
![inconsistency img](https://raw.githubusercontent.com/JingxinLIU415/JingxinLIU415.github.io/master/assets/img/blog_img/colorDeconv_stainInconsist.jpg )
<center>Fig.1: Example image of inconsistency with same stains.</center>
​			

#### 2. Color deconvolution is not able to perfectly separate DAB stain and quantify DAB intensity

DAB does not follow Beer-Lambert law. See [CM van der Loos paper](http://www.pubmedcentral.nih.gov/articlerender.fcgi?artid=2326109) :

>"The brown DAB reaction product is not a true absorber of light, but a scatterer of light, and has a very broad, featureless spectrum. This means that DAB does not follow the Beer-Lambert law, which describes the linear relationship between the concentration of a compound and its absorbance, or optical density. As a consequence,darkly stained DAB has a different spectral shape than lightly stained DAB."

The Fig.2 below shows the top view of HSI color space plot of DAB-H image pixels. It is obvious that the strongly stained DAB pixels board spectrum on Hue. 

![hsv top view of img](https://raw.githubusercontent.com/JingxinLIU415/JingxinLIU415.github.io/master/assets/img/blog_img/colorDeonv_dab-HSI.jpg)

<center>Fig.2: Top view of HSI color space plot of DAB-H image pixels.</center>

Fig.3 illustrates an example of the resulted DAB channel image of a heavily stained tissues using color deconvolution. It is seen that the very dark pixels gives higher values on DAB channel image. 

![cd dark img](https://raw.githubusercontent.com/JingxinLIU415/JingxinLIU415.github.io/master/assets/img/blog_img/colorDeconv_cd-dark.jpg)

<center> Fig.3: The DAB channel image of strong stained tissues using color deconvolution.</center>

![DAB-L img](https://raw.githubusercontent.com/JingxinLIU415/JingxinLIU415.github.io/master/assets/img/blog_img/colorDeconv_DAB-L.jpg)

<center> Fig.4: Visualizations of pixel colours of the DAB-H stained image along the luminance axis and the colour deconvolution DAB channel axis.</center>

As illustrated in Fig.4, the same DAB channel value can correspond to different pixel colours. Further, as illustrated in Fig. 4, it is clear that using simple thresholding cannot completely separate the stains especially if the stained tissues is dark. Consequently, **color deconvolution with one single threshold  is not able to accurately separate strongly stained DAB tissues, and the DAB channel value is not able to used for quantifying DAB intensity**.



In order to separate the positive stain (brown colour) from the negative stain (blue colour), one possible solution is set different different DAB channel thresholds based on the luminance values (see Luminance Adaptive Multiple Threshold [^7]).

To quantify the immunostain intensity, one solution is firstly use Luminance Adaptive Multiple Threshold seperating DAB stained pixel. Then using each pixel's corresponding luminance value to discribing DAB stain intensity [^8].




*If you find any essential mistake in the translation, you are welcomed to point them out with corrections through posting comments or via email to the me (jingxin.liu@outlook.com).*
{: .notice}



*****

#### References

[^1]: Ruifrok, Arnout C., and Dennis A. Johnston. "Quantification of histochemical staining by color deconvolution." *Analytical and quantitative cytology and histology* 23.4 (2001): 291-299.

[^2]: https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law

[^3]: Vahadane, Abhishek, et al. "Structure-preserving color normalization and sparse stain separation for histological images." *IEEE transactions on medical imaging* 35.8 (2016): 1962-1971.

[^4]: https://www.rp-photonics.com/optical_density.html

[^5]: https://en.wikipedia.org/wiki/Absorbance

[^6]: Macenko, Marc, et al. "A method for normalizing histology slides for quantitative analysis." *Biomedical Imaging: From Nano to Macro, 2009. ISBI'09. IEEE International Symposium on*. IEEE, 2009.

[^7]: Liu, Jingxin, Guoping Qiu, and Linlin Shen. "Luminance adaptive biomarker detection in digital pathology images." *Procedia Computer Science* 90 (2016): 113-118.

[^8]: Liu, Jingxin, et al. "An End-to-End Deep Learning Histochemical Scoring System for Breast Cancer Tissue Microarray." *arXiv preprint arXiv:1801.06288* (2018).




​	

