![](https://raw.githubusercontent.com/SmilodonCub/basicVisualBlog/main/assets/img/cajal.png)

<div style="text-align: right">

*sketch of retinal circuitry by Ramon y Cajal*

</div>

<br>

## Dark Adaptation

    Imagine that you are at a movie theater watching Lord of the Rings.
LotR is a very long film, so you have to step out briefly for a break.
When you return to the theater to find your seat again, you take a
moment to pause before you step any further. The theater is so dark
compared to the lobby! It takes you some time to be able to find your
way. It’s a very common subjective experience: when you step into a
relatively darker room, it takes a while for your eyes to ‘adjust’.

    In this post we will explore one of the physiological reasons why it
takes some time to become fully adjusted, or adapted, to the dark. <br>

## The Time Course of Retinal Dark Adaptation

    To begin, let’s look at some data that quantifies the dark adaption
process. The following code will visualize data from a classic
psychophysics experiment published in 1937 by Hecht, Haig &
Chase<sup>1</sup> to observe to time course of dark adaptation. They
took several human observers and placed them in a room to get adapted to
the light before switching the lights off to place them in a dark
environment. Then, they measuring the observers thresholds to find out
how bright a stimulus needs to be for it to be detected at various time
points after the lights were turned off. They performed this experiment
multiple times and varied the light intensity that the subjects were
first adapted to.

The following code will load and visualize the data:

``` r
library( ggplot2 )
library( dplyr )
library( tidyverse )
library( broom )
library( segmented )
```

``` r
url <- 'https://raw.githubusercontent.com/SmilodonCub/DATA621/master/Hecht_Haig_Chase_1937.csv'
darkAdapt <- read.csv( url ) %>%
  mutate( Observer = factor( Observer ),
          adaptation_I = factor( adaptation_I ) )
glimpse( darkAdapt )
```

    ## Rows: 359
    ## Columns: 4
    ## $ Observer     <fct> S.H., S.H., S.H., S.H., S.H., S.H., S.H., S.H., S.H., S.H…
    ## $ adaptation_I <fct> 400k photons, 400k photons, 400k photons, 400k photons, 4…
    ## $ Time         <dbl> 0.19, 0.52, 1.10, 1.50, 2.20, 2.70, 3.40, 4.40, 6.40, 7.7…
    ## $ Log_I        <dbl> 8.26, 7.58, 7.07, 6.80, 6.37, 6.19, 6.00, 5.92, 5.80, 5.7…

Visualizing the results with `ggplot`:
![](Dark_Adaptation_files/figure-markdown_github/unnamed-chunk-3-1.png)

    The figure above shows the results for 3 participants for Hecht,
Haig & Chase’s dark adaptation experiment. If we look at the data series
where the observers were light adapted to a bright room (400k, 38900 and
19500 photons), we can readily see that for all observers, there is a
very distinct and abrupt corner to the envelop of the data and it looks
to be a composite of two different curves. For example, if we follow the
curve for subject C.H.’s adaptation from 400,000 photons, we see a sharp
corner after approximately 18 minutes of adaptation to the dark
environment. Roughly the same trend is followed for subjects A.M.C. and
S.H. As the initial light adaptation level is lowered
(400k\>38900\>19500\>3800 photons), the effect becomes less apparent and
is basically absent with the 263 photon data.

    The cause for these two distinct modes in the data is directly
linked to the physiology of our retina. The retina is a thin laminar
structure that lines the back of our eyes. It is derived from the neural
ectoderm, that is to say that during development, the retina grows as an
outcropping from the same tissue that our central nervous system
develops from. As such, the retina has layers of interconnected neurons
(see the sketch by [Ramon y
Cajal](https://en.wikipedia.org/wiki/Santiago_Ram%C3%B3n_y_Cajal)
above). These neurons send signals about the visual light in our
environment back to the brain by way of the optic nerve. The retinal
cells that directly interact with light to change, or transduce, the
light to a biological representation are called photoreceptors.

    The are two main classes of photoreceptor in the vertebrate retina:
rods and cones<sup>2</sup>. Rods are very sensitive and in the right
conditions can detect even single photon of light<sup>3</sup>. Rods are
relied on for vision in the dark, or **scotopic vision** where their
sensitivity comes in handy. However, at higher light levels, rods
saturate and do not contribute much to vision perception. On the other
hand, cones are relied upon for vision in brighter environments, or
**photopic vision**. Many vertebrates have multiple classes of cones
with slightly different spectral sensitivities. The difference in
activity between classes of cones is the basis for color vision. (more
on this in future posts) Because rods and cones have such different
characteristics and operating ranges, the vertebrate retina is often
referred to as a ‘duplex retina’.

## Piecewise Regression of Dark Adaptation Curves

    The dual curvature of the dark adaptation curves is a consequence of
a duplex retina having both rods and cones. The following code
illustrates this further by modelling the 400k photon data series for
observer C.H. as a piecewise regression. Rod<sup>4</sup> and
Cone<sup>5</sup> adaptation has been described to follow an exponential
decay, so we will use this form to fit the data.

``` r
#separately fit the cone regime and the rod regime of the dark adaptation data
CH_400k <- darkAdapt %>% #filter the 400k photon data from Observer CH
  filter( Observer == 'C.H.',
          adaptation_I == '400k photons' )
#Cone regime: an exponential nonlinear fit using self-starting (guesses it's own start parameters) asymptotic regression function
fit_cones <- nls( Log_I ~ SSasymp( Time, yf, y0, log_alpha ), data = CH_400k[1:14,])
cones_predict <- data.frame( prediction = predict( fit_cones, newdata = CH_400k[14:25,]), Time = CH_400k[14:25,]$Time )
fit_conesA <- augment( fit_cones )
#Rod regime: ""
fit_rods <- nls( Log_I ~ SSasymp( Time, yf, y0, log_alpha ), data = CH_400k[15:25,])
rods_predict <- data.frame( prediction = predict( fit_rods, newdata = CH_400k[1:15,]), Time = CH_400k[1:15,]$Time )
fit_rodsA <- augment( fit_rods )
```

Visualizing the results with `ggplot`:
![](Dark_Adaptation_files/figure-markdown_github/unnamed-chunk-5-1.png)

    The fitted curves above shows rather convincingly how the retina
trades off reliance onto rods over cones during dark adaptation.

    Before Time = 0 of the experiment, the observers had be fully light
adapted, so the rod photoreceptors were saturated from the high light
intensity. At Time = 0, the light went off. The rods need time to
recover from being in a bright environment, so, at first, the observers
could only use their cone photoreceptors to detect the visual stimuli in
the dark, therefore, the data follows the curve for the cone
sensitivity. However, at the Cone/Rod break, the rod photoreceptors have
had enough time to recover and take over detection of stimuli in the
dark adapted state.

    The length of time to reach a fully dark adapted state is dependent
on how bright the previous environment was: the 263 photon adaptation
curves reach the threshold for rod sensitivity much faster than the
other conditions for all subjects. And this is to be expected from our
every day experiences. Going back to our movie theater example: If we
step back into the movie theater to find our seat, we will adapt to the
theater more quickly if we only venture out into the hallway. However,
if we go to a brightly lit lobby it will be much more problematic to
find out seat again.

## References

1.  Hecht, S., Haig, C., & Chase, A. M. (1937). The influence of light
    adaptation on subsequent dark adaptation of the eye. The Journal of
    general physiology, 20(6), 831-850.
2.  Warrant, E. J. (2015). Photoreceptor evolution: ancient ‘cones’ turn
    out to be rods. Current Biology, 25(4), R148-R151.
3.  Rieke, F., & Baylor, D. A. (1998). Single-photon detection by rod
    cells of the retina. Reviews of Modern Physics, 70(3), 1027.
4.  Baylor, D. A., Nunn, B. J., & Schnapf, J. L. (1984). The
    photocurrent, noise and spectral sensitivity of rods of the monkey
    Macaca fascicularis. The Journal of physiology, 357(1), 575-607.
5.  Baylor, D. A., Nunn, B. J., & Schnapf, J. L. (1987). Spectral
    sensitivity of cones of the monkey Macaca fascicularis. The Journal
    of Physiology, 390(1), 145-160.
6.  Reuter, T. (2011). Fifty years of dark adaptation 1961–2011. Vision
    research, 51(21-22), 2243-2262.
7.  [Exponential Curve
    Fitting](https://douglas-watson.github.io/post/2018-09_exponential_curve_fitting/)

<br><br><br>
