---
title: Receptive Field Sizes of Retinal Ganglion Cells
subtitle: using linear regression to demonstrate a physiological difference between retinal gagnlion cell classes using a classic data set
---

![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/retina_xsec.png)

<div style="text-align: right">

*cross section of the retina illumunated by AAV-Brainbow labeling*  
*- [Williams Lab](https://sites.wustl.edu/williams/gallery/) *

</div>

<br>

*“To suppose that the eye with all its inimitable contrivances (…) could
have been formed by natural selection, seems, I confess, absurd to the
highest degree (…) The difficulty of believing that a perfect and
complex eye could be formed by natural selection, though insuperable by
our imagination, should not be subversive of the theory”*

<div style="text-align: right">

*- Charles Darwin*

</div>

    Our eyes are a remarkable structures and their complexity gave
Charles Darwin cause to comment as in the above quote. Although Darwin
was marveling at the intricacy of the anterior eye structures (ex: the
lens), in this post we will bring the retina into focus.

## Retinal Circuitry Crash Course

    As mentioned in the previous post, the retina is actually an
outcropping of the Central Nervous System and as such it is composed of
interconnected neurons. Retinal neurons perform complex operations on
the visual signals that are transduced by photoreceptors. But why
wouldn’t the retina simply send the information directly from the
photoreceptors and let the brain process the signals? There are many
reasons, however, the simplest and most straight forward answer is that
there is a wiring bottleneck from eye to brain: There are \~ 6.4 million
cone and 110-125 million rod photoreceptors in the human
eye.<sup>1</sup> If they were to all send axons to project to the brain,
this would make the optic nerve prohibitively large and cause all sorts
of problems. For example, the point of exit for the optic nerve from the
eye creates a space, the optic disc, where there are no photoreceptors
and therefore is a blind spot. We typically aren’t perceptually unaware
of our blind spot. However, I you would like to convince yourself it
exists, please check out this demo from [Dr. Michael Bach’s Optical
Illusions & Visual
Phenonmena](https://michaelbach.de/ot/cog-blindSpot/index.html) page.

    Evolutionary pressures to simultaneously increase visual acuity
(more photoreceptors) while reducing the size of the optic disc (less
axons) favor retinal preprocessing of visual information. The retina
compresses the information from the photoreceptors by intermixing the
signals with both vertical and horizontal connections across the layers
of retinal circuitry. Vertical connectivity is made through layers of
the retina from the photoreceptors → bipolar cells → Retinal Ganglion
Cells (RGCs). RGCs in the Ganglion cell layer (see figure below) are the
output cells of the retina; their axons project through the optic nerve
and onwards to the brain. Horizontal connections are established in
within layers to interconnect photoreceptors by the Amacrine cells and
to interconnect bipolar cells with the Horizontal Cells. As a result,
the signals of \~116-132 million photoreceptors are reduced to the
output of approximately 1 million RGCs. That reduces the number of axons
by two orders of magnitude!

<center>
![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/retina_circuitry.png) [Image
credit](https://www.researchgate.net/publication/279237071_State-of-the-Art_Report_3D_structure_and_motion_multimodal_perception)
</center>

## Parallel Pathways of Retinal Ganglion Cells

    It’s not just about signal compression, because the retina performs
some complex transformations on the photoreceptor signals. For example,
there are multiple classes of RGCs that, through distinct anatomy,
morphology and connectivity within the retina, effectively carry
distinct visual information across parallel pathways from the retina to
the brain for further processing<sup>2</sup>. In the primate retina, the
most numerous ganglion cell classes are the Magnocellular (M) and
Parvocellular (P) RGCs. There are certainly many other RGC
classes<sup>3</sup>, however, in this post we will focus on M & P RGCs
as examples.

    There are many differences between M and P RGCs. Perhaps most the
most notable are the differences in physiological responses between the
two cells classes, where a **physiological response** refers to the
neurons’ changes in electrical excitability due to an environmental
stimulus. Parvocellular RGCs are commonly observed to have longer, or
sustained responses that are preferential to red-green chromatic stimuli
whereas Magnocellular RGCs typically have a shorter or ‘transient’
response that is preferential of achromatic (light-dark) luminance
stimuli.<sup>4, 5</sup> This difference in response arises due to
differences in patterns of connectivity and one way that this manifests
is a difference in receptive field size. The **receptive field** of a
RGC refers to the region of visual space where changes in visual stimuli
will elicit a change, or physiological response from the cell.

    In the following code, we will explore a data set adapted from
Cronin & Kaplan (1994) that demonstrates the differences in receptive
field sizes for M and P RGCs.<sup>7</sup> RGC receptive fields are
described as having a center-surround structure: vertical connections
from the bipolar layer form the center input which has an antagonistic
relationship to the presumably horizontal connections that make up the
surround. Here, Cronin and Kaplan measured the receptive field center
sizes for M and P RGCs in the primate retina:

``` r
#load the data into the R environment
url <- 'https://raw.githubusercontent.com/SmilodonCub/DATA621/master/macaqueRGCs.csv'
Macaque_RGCs <- read_csv( url )
glimpse( Macaque_RGCs )
```

    ## Rows: 91
    ## Columns: 3
    ## $ Eccentricity <dbl> 3.525, 6.234, 6.531, 7.718, 8.386, 11.503, 11.800, 12.542…
    ## $ Radius       <dbl> 0.098, 0.093, 0.122, 0.105, 0.123, 0.113, 0.189, 0.141, 0…
    ## $ Class        <chr> "M", "M", "M", "M", "M", "M", "M", "M", "M", "M", "M", "M…

Let’s visualize the receptive field center sizes for M and P RGCs as a
function of Eccentricity.  
**Eccentricity** refers to how far from the center of vision the
receptive field is located and is measure in [degrees of visual
angle](https://en.wikipedia.org/wiki/Visual_angle).

``` r
#visualize with ggplot
ggplot( data = Macaque_RGCs, aes( x = Eccentricity, y = Radius, col = Class) ) +
  geom_point(alpha = 0.5, size = 3) +
  geom_smooth( method = 'lm' ) +
  theme_classic() +
  ggtitle('Receptive Field Center Size', subtitle = 'M & P center radius as a function of retinal eccentricity') +
  ylab( 'Center Radius (deg)' ) +
  xlab( 'Eccentricity (deg)' )
```

![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-3-1.png) 

The figure above plots receptive field center radius for M and P RGCs. We
can see a clear difference between the two distributions.  
Here we visualize box plots of the Radius and Eccentricity distributions
for both cells classes

``` r
p1 <- ggplot( data = Macaque_RGCs, aes( x = Class, y = Radius ) ) +
  geom_boxplot() +
  theme_classic() +
  ggtitle( 'Radius' ) +
  ylab( 'Radius (deg)' )

p2 <- ggplot( data = Macaque_RGCs, aes( x = Class, y = Eccentricity ) ) +
  geom_boxplot() +
  theme_classic() +
  ggtitle( 'Eccentricity' ) +
  ylab( 'Eccentricity (deg)' )

grid.arrange( p2, p1, ncol = 2 )
```

![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-4-1.png)

From the box plots above we observe:

-   Eccentricity: While there is a tendency for M cells to be found
    further in the periphery, there is a lot of overlap for the two
    distributions.
    -   Welch 2 sample t-test finds no significant difference
-   Radius: There is relatively greater separation between the M & P
    receptive field radius distributions
    -   Welch 2 sample t-test finds a strong significant difference
        between M & P Radius distributions

Next we will use a linear regression to further characterize the
distinctions between the two cell classes.

``` r
Macaque_RGCs_d <- Macaque_RGCs %>%
  mutate( Mcells = Class == 'M' ) #add a dummy variable
RGC_lm<- lm( data = Macaque_RGCs_d, Radius ~ Mcells + Eccentricity )
summary( RGC_lm )
```

    ## 
    ## Call:
    ## lm(formula = Radius ~ Mcells + Eccentricity, data = Macaque_RGCs_d)
    ## 
    ## Residuals:
    ##       Min        1Q    Median        3Q       Max 
    ## -0.054479 -0.016994 -0.002438  0.014722  0.118379 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  0.0241523  0.0049620   4.867 4.93e-06 ***
    ## McellsTRUE   0.0830616  0.0072133  11.515  < 2e-16 ***
    ## Eccentricity 0.0038181  0.0003339  11.435  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.02636 on 88 degrees of freedom
    ## Multiple R-squared:  0.786,  Adjusted R-squared:  0.7811 
    ## F-statistic: 161.6 on 2 and 88 DF,  p-value: < 2.2e-16

``` r
plot( RGC_lm )
```

![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-7-1.png)![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-7-2.png)![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-7-3.png)![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/RGC_RFsize_files/figure-markdown_github/unnamed-chunk-7-4.png)

Performing a linear regression on the data yields a significant fit with
an *R*<sup>2</sup> value that explains \~78% of the variance in the
data. We interpret the model coefficients as follows:

-   With an increase in Radius of 1 deg, the eccentricity increases by
    \~0.004 deg for either cell class.
-   P cells (Intercept) have estimated average Radius of 0.024deg
-   M cells (`McellsTRUE`) have estimated average Radius of (0.024 +
    0.083) deg

This linear regression suggests that the cell `Class` is an important
and statistically significant factor in determining the radius of a
receptive field at a given eccentricity. However, we can also see from
the diagnostic plots that there are some systematic changes of residual
variance as a function of fitted values. This is likely due to some
non-linear behavior in observations that are measured further out in the
periphery of the retina.

## Conclusion

    The simple analysis above demonstrates that there are measurable
significant differences between retinal ganglion cell classes of the
retina. In future posts we will explore retinal circuitry in more depth
to come to a better understanding about how visual signals are
represented and carried form the eye to the brain.

## References

1.  [NIH Facts and Figures Concerning the Human
    Retina](https://www.ncbi.nlm.nih.gov/books/NBK11556/#:~:text=approximately%20200%2C000.,the%20central%20rod%2Dfree%20fovea.)
2.  Wässle, H. (2004). Parallel processing in the mammalian retina.
    Nature Reviews Neuroscience, 5(10), 747-757.
3.  Dacey, D. M., Peterson, B. B., Robinson, F. R., & Gamlin, P. D.
    (2003). Fireworks in the primate retina: in vitro photodynamics
    reveals diverse LGN-projecting ganglion cell types. Neuron, 37(1),
    15-27.
4.  Yeh, T., Lee, B. B., & Kremers, J. (1995). Temporal response of
    ganglion cells of the macaque retina to cone-specific modulation.
    JOSA A, 12(3), 456-464.
5.  Lee, B. B., Pokorny, J., Smith, V. C., Martin, P. R., & Valbergt, A.
    (1990). Luminance and chromatic modulation sensitivity of macaque
    ganglion cells and human observers. JOSA A, 7(12), 2223-2236.
6.  Lee, B. B. (1996). Receptive field structure in the primate retina.
    Vision research, 36(5), 631-644.
7.  Croner, L. J., & Kaplan, E. (1995). Receptive fields of P and M
    ganglion cells across the primate retina. Vision research, 35(1),
    7-24.

<br><br><br>
