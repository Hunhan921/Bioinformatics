
# Estimating allele frequencies

Let’s use [Bayes](./bayes.md) to estimate allele frequencies -
quantifying our uncertainty - for a couple of important variants in
global populations. Here are the datasets:

1.  Data on the O blood group variant (rs8176719): [O blood group
    data](https://raw.githubusercontent.com/whg-training/whg-training-resources/main/docs/statistical_modelling/introduction/data/1000_genomes_o_blood_group_grouped.tsv)

rs8176719 has two alleles - the functional ‘C’ allele, and a deletion
allele that results in a
[frameshift](https://en.wikipedia.org/wiki/Frameshift_mutation).
Individuals that have two copies of the deletion have ‘O’ blood group.

2.  Data on **rs61028892**, a variant that has been associated with
    [control of fetal
    haemoglobin](https://www.medrxiv.org/content/10.1101/2023.05.16.23289851v1.full)
    in individuals with sickle cell disease: [rs61028892
    data](https://raw.githubusercontent.com/whg-training/whg-training-resources/main/docs/statistical_modelling/introduction/data/1000_genomes_rs61028892_grouped.tsv)

Both datasets above come from the [1000 Genomes Project Phase 3
dataset](https://www.internationalgenome.org/data-portal/data-collection/phase-3).

:::tip Challenge

Load one or both of these datasets into R using `read_tsv()`. Then use
`dbeta()` to plot the posterior distribution of the allele frequency
and/or the O blood group frequency across all populations and then in
individual populations.

**Hint.** `ggplot()` is one good way to do this. Some further tips on
this are below.

:::

``` r
setwd("D:/OneDrive - Queen Mary, University of London/4. Self-learning/WHG training/whg-training-resources-replicate/docs/statistical_modelling/introduction/data")
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.2     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.4     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
oblood <- read_tsv("1000_genomes_o_blood_group_grouped.tsv") 
```

    ## Rows: 29 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr (1): population
    ## dbl (3): C/C, -/C, -/-
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
rs61028892 <- read_tsv("1000_genomes_rs61028892_grouped.tsv")
```

    ## Rows: 29 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr (1): population
    ## dbl (3): rs61028892_0, rs61028892_1, rs61028892_2
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
head(oblood)
```

    ## # A tibble: 6 × 4
    ##   population          `C/C` `-/C` `-/-`
    ##   <chr>               <dbl> <dbl> <dbl>
    ## 1 African Ancestry SW     5    27    29
    ## 2 African Caribbean      12    40    44
    ## 3 Bengali                17    47    22
    ## 4 British                 6    49    34
    ## 5 CEPH                   16    43    40
    ## 6 Colombian               7    23    64

``` r
head(rs61028892)
```

    ## # A tibble: 6 × 4
    ##   population          rs61028892_0 rs61028892_1 rs61028892_2
    ##   <chr>                      <dbl>        <dbl>        <dbl>
    ## 1 African Ancestry SW           58            3            0
    ## 2 African Caribbean             85           11            0
    ## 3 Bengali                       86            0            0
    ## 4 British                       89            0            0
    ## 5 CEPH                          99            0            0
    ## 6 Colombian                     94            0            0

:::tip Challenge

Add the posterior mean and lower and upper values forming a 95% credible
set to the data frame.

Add 95% credible intervals for each population to the data frame, using
`qbeta()`, and then plot these as the estimates (points) and 95%
confidence intervals

:::

## Plotting the posterior

To get `ggplot()` to plot a posterior density for the whole dataset, or
for individual populations, is not conceptually difficult - we’re just
plotting the beta distribution after all. But does require a bit of
complexity in terms of code. In short, `ggplot()` takes a single data
frame as data, so you have to build a big dataframe that represents the
posterior density at a grid of x axis values, for each population you
want to plot.

You could find lots of ways to do this, but here’s one way that is
fairly re-useable. Let’s write a function that reads in one row of data
and generates the dataframe we need. Rather than hard-code this for the
O blood group example, we’ll make it work with generic ‘reference’ and
‘alternate’ counts (e.g. non-O blood group, and O blood group counts) so
it can be re-used for other examples:

``` r
generate_posterior = function(
    row,
    at= seq( from = 0, to = 1, by = 0.01 )
) {
    posterior_distribution = dbeta(
        at,
        shape1 = row$alternate_count + 1,
        shape2 = row$reference_count + 1
    )
    tibble(
        population = row$population,
        reference_count = row$reference_count,
        alternative_count = row$alternate_count,
        at = at, 
        value = posterior_distribution
    )
}
```

For example, for the O blood group data you could apply this to the
whole dataset like this:

``` r
data = read_tsv( "https://raw.githubusercontent.com/whg-training/whg-training-resources/main/docs/statistical_modelling/introduction/data/1000_genomes_o_blood_group_grouped.tsv" )
```

    ## Rows: 29 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr (1): population
    ## dbl (3): C/C, -/C, -/-
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
overall_counts = (
    data
    %>% summarise(
        population = 'all',
        reference_count = sum( `C/C` + `-/C` ),
        alternate_count = sum( `-/-` )
    )
)
print( overall_counts )
```

    ## # A tibble: 1 × 3
    ##   population reference_count alternate_count
    ##   <chr>                <dbl>           <dbl>
    ## 1 all                   1404            1100

``` r
overall_posterior = generate_posterior(
    overall_counts,
    at = seq( from = 0, to = 1, by = 0.01 )
)
```

If you print this, you should see a data frame with 101 rows (one for
each of those `at` values) showing the posterior distribution.

``` r
head(overall_posterior)
```

    ## # A tibble: 6 × 5
    ##   population reference_count alternative_count    at value
    ##   <chr>                <dbl>             <dbl> <dbl> <dbl>
    ## 1 all                   1404              1100  0        0
    ## 2 all                   1404              1100  0.01     0
    ## 3 all                   1404              1100  0.02     0
    ## 4 all                   1404              1100  0.03     0
    ## 5 all                   1404              1100  0.04     0
    ## 6 all                   1404              1100  0.05     0

:::tip Note

Make sure you understand what that data frame is showing. To recap, it’s
the *posterior distribution of the frequency of O blood group across all
populations*, evaluated at a grid of 101 points between zero and one.

The posterior is a [beta distribution](./some_distributions.md) so we
used `dbeta()` to compute it.

:::

Want to plot it? No problem!

``` r
p = (
    ggplot( data = overall_posterior )
    + geom_line( aes( x = at, y = value ))
)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Ok that’s not good enough. Let’s zoom in:

``` r
print( p + xlim( 0.35, 0.55 ))
```

    ## Warning: Removed 80 rows containing missing values or values outside the scale range
    ## (`geom_line()`).

![](allele-frequencies_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

:::tip Challenge

Ok that’s not good enough either. Here are some things you should do to
fix it.

1.  Does your plot look kind of jagged-y? That’s because the posterior
    distribution is concentrated around a small region (almost all the
    mass is between about 0.4 and 0.47), but we have only evaluated on a
    grid of 101 points across the interval. To fix this, *increase the
    number of grid points* (i.e. the `at` variable above) and replot.

2.  **Always give your plots meaningful x axis and y axis labels**.
    (Otherwise you’ll just waste people’s time making them ask what they
    are). The `xlab()` and `ylab()` functions can be used for this,
    e.g.:

``` r
p = (
    ggplot( data = overall_posterior )
    + geom_line( aes( x = at, y = value ))
    + xlab( "My x axis label" )
    + ylab( "My y axis label" )
)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

3.  Let’s get rid of the grey background and make the text bigger, using
    ’theme_minimal()\`:

``` r
print( p + theme_minimal(16) )
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

4.  Personally, I don’t like that the y axis label is printed at 90
    degrees to the reading direction - do you? That can be fixed too
    with a bit of ggplot magic, which I always have to [look up in the
    documentation](https://ggplot2.tidyverse.org) - it looks like this:

``` r
print(
    p
    + theme_minimal(16)
    + theme(
        axis.title.y = element_text( angle = 0, vjust = 0.5 )
    )
)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

**Note**. you have to do this call to `theme()` *after*
`theme_minimal()`, otherwise it resets this property.

**Challenge** Put all this together to make a final plot of the
posterior distribution now.

``` r
# Increase number of grid point 
generate_posterior = function(
    row,
    at= seq( from = 0, to = 1, by = 0.001 )
) {
    posterior_distribution = dbeta(
        at,
        shape1 = row$alternate_count + 1,
        shape2 = row$reference_count + 1
    )
    tibble(
        population = row$population,
        reference_count = row$reference_count,
        alternative_count = row$alternate_count,
        at = at, 
        value = posterior_distribution
    )
}

overall_counts = (
    oblood #this data was loaded to the environment before
    %>% summarise(
        population = 'all',
        reference_count = sum( `C/C` + `-/C` ),
        alternate_count = sum( `-/-` )
    )
)
print( overall_counts )
```

    ## # A tibble: 1 × 3
    ##   population reference_count alternate_count
    ##   <chr>                <dbl>           <dbl>
    ## 1 all                   1404            1100

``` r
overall_posterior = generate_posterior(
    overall_counts,
    at = seq( from = 0, to = 1, by = 0.001 )
)

ggplot( data = overall_posterior )+ 
  geom_line( aes( x = at, y = value ))+
  xlim( 0.3, 0.6 )+
  xlab( "O blood group frequency" )+
  ylab( "Posterior density" )+
  theme_minimal(16)+
  theme( axis.title.y = element_text( angle = 0, vjust = 0.5 ))
```

    ## Warning: Removed 700 rows containing missing values or values outside the scale range
    ## (`geom_line()`).

![](allele-frequencies_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

:::

Congratulations!

## Plotting multiple populations

Plotting multiple populations ought to be easy now - we just somehow
need to call `generate_posterior()` for each row of our data, instead of
for the whole set. One way to do that is simple to loop over the rows
and accumulate the results:

``` r
per_population_posterior = tibble()
for( i in 1:nrow( oblood )) {
    # summarise one row, as before
    population_data = (
        oblood[i,]  
        %>%
        summarise(
            population = population,
            reference_count = sum( `C/C` + `-/C` ),
            alternate_count = sum( `-/-` )
        )
    )
    per_population_posterior = bind_rows(
        per_population_posterior,
        generate_posterior( population_data )
    )
}
```

If you look at `per_population_data` you should see it has thousands of
rows (or tens of thousands if you increased the number of `at` values),
the same number of rows per population. You could count them like this:

``` r
per_population_posterior %>% group_by( population ) %>% summarise( number_of_rows = n() )
```

    ## # A tibble: 29 × 2
    ##    population          number_of_rows
    ##    <chr>                        <int>
    ##  1 African Ancestry SW           1001
    ##  2 African Caribbean             1001
    ##  3 Bengali                       1001
    ##  4 British                       1001
    ##  5 CEPH                          1001
    ##  6 Colombian                     1001
    ##  7 Dai Chinese                   1001
    ##  8 English                       1001
    ##  9 Esan                          1001
    ## 10 Finnish                       1001
    ## # ℹ 19 more rows

Getting ggplot to plot this is now easy - we use a **facet**:

``` r
p = (
    ggplot( data = per_population_posterior )
    + geom_line( aes( x = at, y = value ))
    + facet_grid( population ~ . )
)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

Cool!

:::tip Note

That `facet_grid()` call works like this. You give it two variables to
facet over rows and columns of the result, and write
`variable1 ~ variable2`. In our case, we just want to facet over one
variable rows, so we do `population ~ .`.

`ggplot()` then does all the work of splitting up the data up into each
of the facets and arranges the plot into rows and columns. It’s a very
powerful feature for quickly exploring datasets.

:::

As usual, this initial plot isn’t quite good enough to start with. We
should do several things:

1.  Those y axis scales are useless - too hard to see. We should get rid
    of them.
2.  The facet labels (on the right) are useless as well! We can’t see
    the population names.
3.  A more subtle bug is that the posteriors all have slightly different
    heights (depending on how spread-out the distribution is). But at
    the moment they all have the same scale.

All this can be fixed with suitable calls to ggplot - see the comments
below:

``` r
p = (
    ggplot( data = per_population_posterior )
    + geom_line( aes( x = at, y = value ))
    + facet_grid(
        population ~ .,
        # Make y axis facets have their own scales, learnt from the data
        scales = "free_y"
    )
    + theme_minimal(16)
    + xlab( "O blood group frequency" )
    + ylab( "Posterior density" )
    + theme(
        # remove Y axis tick labels
        axis.text.y = element_blank(),
        # rotate facet labels on right of plot
        strip.text.y.right = element_text(angle = 0, hjust = 0),
        # rotate overall y axis label 90
        axis.title.y = element_text(angle = 0, vjust = 0.5)
    )

)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

## Ordering populations

That’s all very well, currently the populations are sorted in
alphabetical order. Wouldn’t it be nicer to order the populations by
allele frequency?

The way to do this in ggplot is not obvious at first glance - you have
to re-order the data itself. In R, there is a specific way to do this
known as a **factor**. A factor is a set of string values that take one
of a set of levels. You specify the order of the levels, and voila, the
data is ordered.

Let’s do that now. First let’s compute the frequency in the original
data:

``` r
oblood$O_bld_grp_frequency = oblood[['-/-']] / ( oblood[['C/C']] + oblood[['-/C']] + oblood[['-/-']])
```

Now let’s use that to get an ordered list of populations. An easy way is
to use the [dplyr `arrange()`
function](https://dplyr.tidyverse.org/reference/arrange.html) to order
the dataframe by the frequency, then get the populations:

``` r
ordered_populations = (
    oblood
    %>% arrange( O_bld_grp_frequency )
)$population
```

Finally, we’ll convert the `population` column of
`per_population_posterior` to a factor with these levels:

``` r
per_population_posterior$population = factor(
    per_population_posterior$population,
    levels = ordered_populations 
)
```

That’s it! (If you try `str(data)` you’ll see that the `population`
column is now a factor.)

``` r
str(oblood)
```

    ## spc_tbl_ [29 × 5] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ population         : chr [1:29] "African Ancestry SW" "African Caribbean" "Bengali" "British" ...
    ##  $ C/C                : num [1:29] 5 12 17 6 16 7 7 0 8 26 ...
    ##  $ -/C                : num [1:29] 27 40 47 49 43 23 39 1 32 44 ...
    ##  $ -/-                : num [1:29] 29 44 22 34 40 64 47 1 59 29 ...
    ##  $ O_bld_grp_frequency: num [1:29] 0.475 0.458 0.256 0.382 0.404 ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   population = col_character(),
    ##   ..   `C/C` = col_double(),
    ##   ..   `-/C` = col_double(),
    ##   ..   `-/-` = col_double()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

``` r
str(per_population_posterior)
```

    ## tibble [29,029 × 5] (S3: tbl_df/tbl/data.frame)
    ##  $ population       : Factor w/ 29 levels "Spanish","Bengali",..: 18 18 18 18 18 18 18 18 18 18 ...
    ##  $ reference_count  : num [1:29029] 32 32 32 32 32 32 32 32 32 32 ...
    ##  $ alternative_count: num [1:29029] 29 29 29 29 29 29 29 29 29 29 ...
    ##  $ at               : num [1:29029] 0 0.001 0.002 0.003 0.004 0.005 0.006 0.007 0.008 0.009 ...
    ##  $ value            : num [1:29029] 0.00 1.31e-68 6.81e-60 8.43e-55 3.43e-51 ...

Now if you regenerate the above plot, things should be in order.

``` r
p = (
    ggplot( data = per_population_posterior )
    + geom_line( aes( x = at, y = value ))
    + facet_grid(
        population ~ .,
        # Make y axis facets have their own scales, learnt from the data
        scales = "free_y"
    )
    + theme_minimal(16)
    + xlab( "O blood group frequency" )
    + ylab( "Posterior density" )
    + theme(
        # remove Y axis tick labels
        axis.text.y = element_blank(),
        # rotate facet labels on right of plot
        strip.text.y.right = element_text(angle = 0, hjust = 0, size = 10),
        # rotate overall y axis label 90
        axis.title.y = element_text(angle = 90, vjust = 0.5)
    )

)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

:::tip Question

In which populations is O blood group at lowest frequency? In which
populations at highest frequency? A:

``` r
oblood$population[which.min(oblood$O_bld_grp_frequency)]
```

    ## [1] "Spanish"

``` r
oblood$population[which.max(oblood$O_bld_grp_frequency)]
```

    ## [1] "Peruvian"

:::

:::tip Challenge

Add the posterior mean and lower and upper values forming a 95% credible
set to the data frame.

Add 95% credible intervals for each population to the data frame, using
`qbeta()`, and then plot these as the estimates (points) and 95%
confidence intervals

:::

``` r
str(rs61028892)
```

    ## spc_tbl_ [29 × 4] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ population  : chr [1:29] "African Ancestry SW" "African Caribbean" "Bengali" "British" ...
    ##  $ rs61028892_0: num [1:29] 58 85 86 89 99 94 93 2 95 99 ...
    ##  $ rs61028892_1: num [1:29] 3 11 0 0 0 0 0 0 4 0 ...
    ##  $ rs61028892_2: num [1:29] 0 0 0 0 0 0 0 0 0 0 ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   population = col_character(),
    ##   ..   rs61028892_0 = col_double(),
    ##   ..   rs61028892_1 = col_double(),
    ##   ..   rs61028892_2 = col_double()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

This *rs61028892* dataset, without prior knowledge in bioinformatic, I
assumed (with the help from AI), that *rs61028892_0* column counts
individuals with 0 copies of the alternate allele. They are homozygous
for the reference allele. *rs61028892_1* column counts individuals with
1 copy of the alternate allele. They are heterozygous for the reference
allele.*rs61028892_2* column counts individuals with 2 copies of the
alternate allele. They are homozygous for the alternate allele.

In the previous practice, our aim was to find the O blood frequency in
the population. It should be equivalent to the frequency of having 2
copies of the alternate allele. Because it is said that in the
beginning, rs8176719 has two alleles - the functional ‘C’ allele, and a
deletion allele that results in a
[frameshift](https://en.wikipedia.org/wiki/Frameshift_mutation).
Individuals that have two copies of the deletion have ‘O’ blood group.

``` r
#Generate posterior with 95% CI
generate_posterior = function(
    row,
    at= seq( from = 0, to = 1, by = 0.0001 )
) {
    posterior_distribution = dbeta(
        at,
        shape1 = row$alternate_count + 1,
        shape2 = row$reference_count + 1,
    )
    
    # Add summary statistics
    posterior_mean = (row$alternate_count + 1)/((row$alternate_count + 1) + (row$reference_count + 1))
    
    lower = qbeta(
      0.025,
        shape1 = row$alternate_count + 1,
        shape2 = row$reference_count + 1
    )
    upper = qbeta(
      0.975,
        shape1 = row$alternate_count + 1,
        shape2 = row$reference_count + 1
    )
    tibble(
        population = row$population,
        reference_count = row$reference_count,
        alternative_count = row$alternate_count,
        at = at, 
        value = posterior_distribution,
        posterior_mean = posterior_mean,
        lower = lower,
        upper = upper
    )
}
```

``` r
per_population_posterior= tibble()
for( i in 1:nrow( rs61028892)) {
    # summarise one row, as before
    population_data = (
        rs61028892[i,]   
        %>%
        summarise(
            population = population,
            reference_count = sum( rs61028892_0 + rs61028892_1 ),
            alternate_count = sum( rs61028892_2 )
        )
    )
    per_population_posterior = bind_rows(
        per_population_posterior,
        generate_posterior( population_data )
    )
}
```

``` r
rs61028892$deletion_grp_frequency = rs61028892[["rs61028892_2"]]/(rs61028892[["rs61028892_0"]] + rs61028892[["rs61028892_1"]] + rs61028892[["rs61028892_2"]])

# or rs61028892$deletion_grp_frequency <- rs61028892$rs61028892_2 / (
#  rs61028892$rs61028892_0 + rs61028892$rs61028892_1 + rs61028892$rs61028892_2
#)

ordered_populations = (
    rs61028892
    %>% arrange( deletion_grp_frequency )
)$population


per_population_posterior$population = factor(
    per_population_posterior$population,
    levels = ordered_populations 
)
```

``` r
p = (
    ggplot( data = per_population_posterior )
    + geom_line( aes( x = at, y = value ))
    + facet_grid(
        population ~ .,
        # Make y axis facets have their own scales, learnt from the data
        scales = "free_y"
    )
    + theme_minimal(16)
    + xlab( "Deletion allele group frequency" )
    + ylab( "Posterior density" )
    + theme(
        # remove Y axis tick labels
        axis.text.y = element_blank(),
        # rotate facet labels on right of plot
        strip.text.y.right = element_text(angle = 0, hjust = 0, size = 10),
        # rotate overall y axis label 90
        axis.title.y = element_text(angle = 90, vjust = 0.5)
    )

)
print(p)
```

![](allele-frequencies_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->
