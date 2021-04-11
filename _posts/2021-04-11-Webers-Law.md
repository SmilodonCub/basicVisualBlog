<img src="descartes.png" width="100%" height="100%" />

## An Introduction to Weber’s Law

    Visual environments and the minds that perceive them are complex.
Psychophysics is an branch off the psychology tree which aims to
quantify the relationships between physical stimuli in the environment
and their internal representation, or perception, experienced by
observers.<sup>2</sup> Here we will look at a psychophysical phenomenon,
**Weber’s Law**, that describes the simple linear relationship between
changes in intensity of a stimulus and the way the sensation of the
change is perceived by human subjects.

    In 1846, Ernst Weber described a relationship where “the minimum
increase of stimulus which will produce a perceptible increase of
sensation is proportional to the pre-existent stimulus”. In other words,
as a stimulus intensity increases, so to does the increment of change
such that the observer can perceive a difference in intensity. The
minimum increment of stimulus needed to be detected by a subject is also
called the **just noticeable difference (jnd)**.<sup>1</sup> Another way
to state Weber’s Law is that the jnd increases at a constant rate as
stimulus intensity increases, which Gustav Fetchner (the godfather of
psychophysics) summarized in this simple equation:
$$\\frac{jnd}{I} = k$$
where *I* is the intensity of the stimulus and *k* is the value of the
constant that describes the relationship.  
    Remarkably, Weber’s Law has been demonstrated for just about every
mode of sensation available to study. The proportional relationship
between jnd and intensity remains the same whether describing visual,
auditory or tactile increments while the constant, *k* changes to
describe each process. In this post, we will walk through an
illustration of Weber’s Law for a visual psychophysics experiment and
find the constant, *k*, by way of linear regression in `R`.

First, import the `R` libraries that will be used for this analysis:

``` r
library( ggplot2 )
library( dplyr )
library( tidyverse )
library( broom )
```

## Demonstrating Weber’s Law

    Interestingly, Weber’s Law had actually been described for visual
stimuli long before it was declared. In the late 1700s a French
scientist, Pierre Bouger, made the observation that his ability to
detect his shadow cast by a candle onto a screen was proportional to the
screen’s illumination by another candle placed behind it. Since Bouger
and following Weber, many investigators have looked at jnds of visual
stimuli<sup>3</sup>. For example, the following code reproduces the
results from Cornsweet and Pinsker (1965) for a subject performing 3
different luminance discrimination tasks<sup>4</sup>.

-   Task 1: discriminate 2 flashed discs presented briefly on a dark
    background. Which disc was brighter?
-   Task 2: 2 equal discs are present (on constantly) and one briefly
    changes intensity. Which disc became brighter?
-   Task 3: 2 equal discs are present before a trial, however, they
    extinguish \~5 seconds before two test discs are shown. Which test
    disc is brighter?

    Let’s take a look at data that was adapted from Cornsweet & Pinsker
(1965, Fig. 4). We’ll observe how a simple linear regression can be
applied to find the constant, *k*:

``` r
url <- 'https://raw.githubusercontent.com/SmilodonCub/DATA621/master/cornsweet_AllExp.csv'
cornsweet_df <- read.csv( url )
ggplot( data = cornsweet_df, aes( x = x, y = y, col = factor( Exp ) ) ) +
  geom_point() +
  geom_line() +
  theme_classic() +
  scale_color_manual(labels = c("Task 1", "Task 2", "Task 3"), values = c("black", "azure4", "azure3") ) +
  labs( title = "Cornsweet & Pinsker (1965)", x = "Log L (ft.-lamberts)", 
        y = expression( paste( "Log ", Delta, "L (ft.-lamberts)" ) ), color = 'Increment Threshold' )
```

![](webers_law_files/figure-markdown_github/unnamed-chunk-2-1.png)

    In the figure above, the observer’s jnd ( Log *Δ*L, y-axis) is
plotted as a function of the stimulus intensity (Log L, x-axis). We see
that, for most of the stimulus’s luminance range, the data points follow
a clear linear envelope. The major points of deviation from linearity
occur for Task 1 & 2 at very low levels of stimulus intensity.

    In the following code, we will model the subjects response as a
linear regression and compare the fits between tasks:

``` r
cornsweet_lms <- cornsweet_df %>%
  nest( -Exp ) %>%
  mutate( fit = map( data, ~ lm( y ~ x, data = . ) ),
          cornsweet_lms_res = map( fit, augment ) ) %>%
  unnest( cornsweet_lms_res )
```

    Visualize the data for each task with the corresponding linear
regression fit:

``` r
cornsweet_lms %>%
  ggplot( aes( x = x, y = .fitted ) ) +
  geom_line() +
  geom_point( aes( x = x, y = y ) ) +
  facet_grid( Exp ~ . ) +
  theme_bw() +
  labs( title = "Linear Regression Fits for each Task", x = "Log L (ft.-lamberts)", 
        y = expression( paste( "Log ", Delta, "L (ft.-lamberts)" ) ) )
```

![](webers_law_files/figure-markdown_github/unnamed-chunk-4-1.png)

    We can see that Tasks 1 & 2 have greater deviations from the fitted
regression line, but let’s get a better look by plotting the residuals
directly.

    Visualize the residuals for each task’s linear model fit:

``` r
cornsweet_lms %>%
  ggplot( aes( x = x, y = .resid ) ) +
  geom_point() +
  facet_grid( Exp ~ . ) +
  theme_bw() +
  labs( title = "Residuals", x = "Log L (ft.-lamberts)", y = expression( "Residuals" ) )
```

![](webers_law_files/figure-markdown_github/unnamed-chunk-5-1.png)

    The data for Task 3 does not extend into the same lower intensity
range (\<-3) as Task 1 & 2 where their residuals are greatest. As a
result, a linear model accounts for more variance for Task 3’s data. We
can demonstrate this by calculating the *R*<sup>2</sup> for each linear
fit.

``` r
cornsweet_R2 <- cornsweet_df %>%
  nest( -Exp ) %>%
  mutate( fit = map( data, ~ lm( y ~ x, data = . ) ),
          cornsweet_lms_res = map( fit, glance ) ) %>%
  unnest( cornsweet_lms_res ) %>%
  ggplot( aes( x = factor( Exp ), y = r.squared ) ) +
  geom_bar( stat = "identity" ) +
  labs( title = expression( paste( R^{2}, ' for each Task') ), x = "Task #", y = expression( R^{2} ) )
cornsweet_R2
```

![](webers_law_files/figure-markdown_github/unnamed-chunk-6-1.png)

    The *R*<sup>2</sup> values are all very close to 1 indicating a very
good linear fit, especially considering that these measurements were
acquired from a human psychophysics subject. The slope of each of the
regression lines can be interpreted as the constant, *k*, also known as
the **Weber Fractions**.

Weber Fraction for Cornsweet & Pinsker tasks:

-   Task 1 $\\frac{jnd}{I}=$ 0.7443
-   Task 2 $\\frac{jnd}{I}=$ 0.6462
-   Task 3 $\\frac{jnd}{I}=$ 0.8781

    The slope (*k*, Weber’s Fraction) for Task 3 is greater than Tasks 1
& 2 and Task 3 has the highest *R*<sup>2</sup> value. This is due to the
residuals with relatively high magnitudes obtained for discrimination
thresholds at very low luminance intensities. But these tasks seem very
similar….Why would there be any difference in linear fits between the
three tasks at all? The experimentors manipulated the adaptation state
of the subjects before the stimulus discs were flashed on the screen;
this was done to try to extend the linear range of the Weber’s Law
relationship for Task 3. Do you think they were successful?….

Please refer to the original paper for more details about the task and
experimental design.

### the Take Home

    Weber’s Law demonstrates that despite the difficult task of
representing our environment, simple relationships that describe our
sensation of a complicated world can be found. The data and code above
walk us through a classic finding in visual psychophysics. However,
similar relationships for other sensory modalities and in many other
species have been described.<sup>5</sup> Gustav Fetchner and Ernst Weber
first described the situation of tactile weight discrimination: For a
subject to be able to discriminate the weight of two objects, the
difference in weight has to increase proportionally to the weight of the
objects. For example, it is much easier to tell the difference between
two objected weighing 5lbs and 6lbs than two objects weighing 55lbs and
56lbs. Additionally, Weber’s Law has also been applied to human factors
for business strategy where a 1$ discount for a 3$ product is definitely
perceived favorably over a 1$ discount for a 80$ product.

![](weberPrice.jpg)

## References

1.  Lu, Z. L., & Dosher, B. (2013). Visual Psychophysics: From
    laboratory to theory. MIT Press.
2.  Palmer, S. E. (1999). Vision science: Photons to phenomenology. MIT
    press.
3.  Barlow, H. B. (1957). Increment thresholds at low intensities
    considered as signal/noise discriminations. The Journal of
    Physiology, 136(3), 469-488.
4.  Cornsweet, T. N., & Pinsker, H. M. (1965). Luminance discrimination
    of brief flashes under various conditions of adaptation. The Journal
    of Physiology, 176(2), 294-310.
5.  Laming, D. (2009). Weber’s law. In Insid. Psychol. a Sci. over 50
    years. Oxford University Press.

<br><br><br>
