# warmthcompetence Vignette

## Warmth and Competence

Warmth and competence are the two main dimensions of social perception
and judgment ([Cuddy, Fiske, and Glick 2008](#ref-cuddy2008)) . When
individuals introduce or describe themselves, their audiences
automatically make judgments about their warmth and competence. In the
*warmthcompetence* package, we provide tools that estimate warmth and
competence social perceptions from natural self-presentational language.
We use pre-trained enet regression models to provide numerical
representations of warmth and competence perceptions.

### Installation

To install *warmthcompetence* from CRAN, use the following code in your
R session:

``` r
install.packages("warmthcompetence")
```

To install the development version from GitHub, use the following code:

``` r
devtools::install_github("bushraguenoun/warmthcompetence")
```

First, we load the package into our environment.

``` r
library("warmthcompetence")
```

Note that some features depend on *spacyr* which must be installed
separately through Python. To install *spacyr*, follow the instructions
[here](https://www.rdocumentation.org/packages/spacyr/). Then run the
code below:

``` r
spacyr::spacy_initialize()
```

### Usage

*warmthcompetence* contains two main functions:
[`warmth()`](https://bushraguenoun.github.io/warmthcompetence/reference/warmth.md)
and
[`competence()`](https://bushraguenoun.github.io/warmthcompetence/reference/warmth.md).
These functions can be used as described below:

``` r
competence_scores <- competence(text_vector, ID_vector, metrics = "scores")

warmth_scores <- warmth(text_vector, ID_vector, metrics = "scores")
```

In the code above, `text_vector` is the vector of texts that will be
assessed for warmth or competence. `ID_vector` is a vector of IDs that
will be used to identify the warmth or competence scores. The `metrics`
argument allows users to decide what metrics to return. Users can return
the warmth or competence scores (`metrics = "scores"`), the features
that underlie the warmth or competence scores (`metrics = "features"`),
or both the warmth or competence scores and the features
(`metrics = "all"`). The default choice is to return the warmth or
competence scores.

### Example: Warmth and Competence Signaling in Self Introductions

We ran a study in which 393 participants were randomly assigned to
either the warmth or competence condition and asked to write a short
introduction of themselves that presented them as a \[warm/competent\]
person. Then, we had three independent judges blind to participant
condition rate each of the introductions for warmth and competence on a
scale of 1 to 10.

The study data can be accessed by calling `data("vignette_data")`. This
command will return a data frame with all of the data collected from the
study. The columns of this data frame are the following:

- `ResponseId`: participant identifiers
- `bio`: participant introductions
- `condition`: participant condition in the study (warmth versus
  competence).
- `RA_warm_AVG`: the average perceptions of warmth for each participant
  introduction on a scale of 1 (extremely low warmth) to 10 (extremely
  high warmth) across three independent judges ($\alpha = 0.76$). The
  individual RA ratings are in the following columns: `warm_1`,
  `warm_2`, and `warm_3`
- `RA_comp_AVG`: the average perceptions of warmth for each participant
  introduction on a scale of 1 (extremely low competence) to 10
  (extremely high competence) across three independent judges
  ($\alpha = 0.79$). The individual RA ratings are in the following
  columns: `comp_1`, `comp_2`, and `comp_3`

#### Predicting Warmth Perceptions

Can our warmth model predict the average warmth ratings of the
independent judges?

First we run the functions on the study data.

``` r
data("vignette_data")

# Generate competence scores
competence_scores <- competence(vignette_data$bio, vignette_data$ResponseId)

# Generate warmth scores
warmth_scores <- warmth(vignette_data$bio, vignette_data$ResponseId)

# Merge scores together
both_scores <- dplyr::inner_join(competence_scores, warmth_scores,
                                 by = c("ID" = "ID"))

# Merge scores into participant data
all_data <- dplyr::left_join(vignette_data, both_scores,
                             by = c("ResponseId" = "ID"))
```

Then, we explore whether the warmth model predicts the warmth ratings of
the independent judges.

``` r
warmth_model <- lm(RA_warm_AVG ~ warmth_predictions,
                   data = all_data)

summary(warmth_model)
#> 
#> Call:
#> lm(formula = RA_warm_AVG ~ warmth_predictions, data = all_data)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.6622 -0.2752  0.0374  0.3776  2.3164 
#> 
#> Coefficients:
#>                    Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)         5.75999    0.03213  179.27   <2e-16 ***
#> warmth_predictions  0.79405    0.04660   17.04   <2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 0.6369 on 391 degrees of freedom
#> Multiple R-squared:  0.4261, Adjusted R-squared:  0.4247 
#> F-statistic: 290.3 on 1 and 391 DF,  p-value: < 2.2e-16
```

As can be seen above, the predictions of the warmth model positively
predict the independent judges’ ratings.

#### Predicting Competence Perceptions

Can our competence model predict the average competence ratings of the
independent judges?

``` r
competence_model <- lm(RA_comp_AVG ~ competence_predictions,
                       data = all_data)

summary(competence_model)
#> 
#> Call:
#> lm(formula = RA_comp_AVG ~ competence_predictions, data = all_data)
#> 
#> Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -3.9872 -0.4573 -0.0296  0.5009  2.9592 
#> 
#> Coefficients:
#>                        Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)             5.48828    0.03785  144.98   <2e-16 ***
#> competence_predictions  0.99237    0.06899   14.38   <2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 0.7502 on 391 degrees of freedom
#> Multiple R-squared:  0.346,  Adjusted R-squared:  0.3444 
#> F-statistic: 206.9 on 1 and 391 DF,  p-value: < 2.2e-16
```

As can be seen above, the predictions of the warmth model positively
predict the independent judges’ ratings.

#### Predicting Participant Condition

Are model predictions of competence higher for the participants asked to
present themselves as a competent person compared to a warm person?

``` r
t.test(competence_predictions ~ condition, data = all_data)
#> 
#>  Welch Two Sample t-test
#> 
#> data:  competence_predictions by condition
#> t = 5.9885, df = 389.68, p-value = 4.815e-09
#> alternative hypothesis: true difference in means between group competent and group warm is not equal to 0
#> 95 percent confidence interval:
#>  0.2132765 0.4217659
#> sample estimates:
#> mean in group competent      mean in group warm 
#>               0.1707312              -0.1467900
```

As can be seen above, the model significantly predicted that
participants in the competence condition expressed more competence
compared to those in the warmth condition.

Now, are model predictions of warmth higher for the participants asked
to present themselves as a warm person compared to a competent person?

``` r
t.test(warmth_predictions ~ condition, data = all_data)
#> 
#>  Welch Two Sample t-test
#> 
#> data:  warmth_predictions by condition
#> t = -12.829, df = 372.43, p-value < 2.2e-16
#> alternative hypothesis: true difference in means between group competent and group warm is not equal to 0
#> 95 percent confidence interval:
#>  -0.8628778 -0.6335158
#> sample estimates:
#> mean in group competent      mean in group warm 
#>              -0.3610531               0.3871437
```

As can be seen above, the model significantly predicted that
participants in the warmth condition expressed more warmth compared to
those in the competence condition.

### Conclusion

In this example, our pre-trained enet regression models predicted the
ratings of independent human judges and participant judges. Feel free to
use these models on your own data and reach out with any comments,
questions, or concerns!

### References

Cuddy, Amy J. C., Susan T. Fiske, and Peter Glick. 2008. “Warmth and
Competence as Universal Dimensions of Social Perception: The Stereotype
Content Model and the BIAS Map.” In *Advances in Experimental Social
Psychology*, 40:61–149. Elsevier.
<https://doi.org/10.1016/S0065-2601(07)00002-0>.
