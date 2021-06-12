---
title: sfcr package in action
author: 'João Macalós'
date: '2021-06-12'
slug: []
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-06-12T11:07:45+02:00'
featured: no
math: true
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: [sfcr]
references:
- id: godley2007monetary
  title: >
       Monetary Economics: An Integrated Approach To Credit, Money, Income, Production and Wealth
  author:
  - family: Godley
    given: Wynne
  - family: Lavoie
    given: Marc
  publisher: Palgrave Macmillan
  type: book
  issued:
    year: 2007
---

<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/d3/d3.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/sankey/sankey.js"></script>
<script src="{{< blogdown/postref >}}index_files/sankeyNetwork-binding/sankeyNetwork.js"></script>

The objective of this are notebook is to show the *sfcr* package in action. In this example, I’m going to work with the Portfolio Choice model from Godley and Lavoie (2007).

As usual, we start by loading the required packages:

``` r
library(tidyverse)
#> -- Attaching packages --------------------------------------- tidyverse 1.3.0 --
#> v ggplot2 3.3.2     v purrr   0.3.4
#> v tibble  3.0.4     v dplyr   1.0.2
#> v tidyr   1.1.2     v stringr 1.4.0
#> v readr   1.4.0     v forcats 0.5.0
#> -- Conflicts ------------------------------------------ tidyverse_conflicts() --
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
library(sfcr)
```

# The matrices of model PC

We start the analysis by writing down the balance-sheet and transactions-flow matrices of the model with the help of the `sfcr_matrix()` function:

``` r
bs_pc <- sfcr_matrix(
  columns = c("Households", "Firms", "Government", "Central bank", "sum"),
  codes = c("h", "f", "g", "cb", "s"),
  r1 = c("Money", h = "+Hh", cb = "-Hs"),
  r2 = c("Bills", h = "+Bh", g = "-Bs", cb = "+Bcb"),
  r3 = c("Balance", h = "-V", g = "+V")
)

sfcr_matrix_display(bs_pc, "bs")
```

<table style="color: black; width: auto !important; margin-left: auto; margin-right: auto;" class="table table-striped table-hover">
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:left;">
Households
</th>
<th style="text-align:left;">
Firms
</th>
<th style="text-align:left;">
Government
</th>
<th style="text-align:left;">
Central bank
</th>
<th style="text-align:left;">
`\(\sum\)`
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Money
</td>
<td style="text-align:left;">
`\(+Hh\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(-Hs\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Bills
</td>
<td style="text-align:left;">
`\(+Bh\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(-Bs\)`
</td>
<td style="text-align:left;">
`\(+Bcb\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Balance
</td>
<td style="text-align:left;">
`\(-V\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+V\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
`\(\sum\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
</tbody>
</table>

``` r
tfm_pc <- sfcr_matrix(
  columns = c("Households", "Firms", "Government", "CB current", "CB capital"),
  codes = c("h", "f", "g", "cbc", "cbk"),
  c("Consumption", h = "-C", f = "+C"),
  c("Govt. Expenditures", f = "+G", g = "-G"),
  c("Income", h = "+Y", f = "-Y"),
  c("Int. payments", h = "+r[-1] * Bh[-1]", g = "-r[-1] * Bs[-1]", cbc = "+r[-1] * Bcb[-1]"),
  c("CB profits", g = "+r[-1] * Bcb[-1]", cbc = "-r[-1] * Bcb[-1]"),
  c("Taxes", h = "-TX", g = "+TX"),
  c("Ch. Money", h = "-(Hh - Hh[-1])", cbk = "+(Hs - Hs[-1])"),
  c("Ch. Bills", h = "-(Bh - Bh[-1])", g = "+(Bs - Bs[-1])", cbk = "-(Bcb - Bcb[-1])")
)

sfcr_matrix_display(tfm_pc)
```

<table style="color: black; width: auto !important; margin-left: auto; margin-right: auto;" class="table table-striped table-hover">
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:left;">
Households
</th>
<th style="text-align:left;">
Firms
</th>
<th style="text-align:left;">
Government
</th>
<th style="text-align:left;">
CB current
</th>
<th style="text-align:left;">
CB capital
</th>
<th style="text-align:left;">
`\(\sum\)`
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Consumption
</td>
<td style="text-align:left;">
`\(-C\)`
</td>
<td style="text-align:left;">
`\(+C\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Govt. Expenditures
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+G\)`
</td>
<td style="text-align:left;">
`\(-G\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Income
</td>
<td style="text-align:left;">
`\(+Y\)`
</td>
<td style="text-align:left;">
`\(-Y\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Int. payments
</td>
<td style="text-align:left;">
`\(+r_{-1}\cdot Bh_{-1}\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(-r_{-1}\cdot Bs_{-1}\)`
</td>
<td style="text-align:left;">
`\(+r_{-1}\cdot Bcb_{-1}\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
CB profits
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+r_{-1}\cdot Bcb_{-1}\)`
</td>
<td style="text-align:left;">
`\(-r_{-1}\cdot Bcb_{-1}\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Taxes
</td>
<td style="text-align:left;">
`\(-TX\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+TX\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Ch. Money
</td>
<td style="text-align:left;">
`\(-\Delta Hh\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+\Delta Hs\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
Ch. Bills
</td>
<td style="text-align:left;">
`\(-\Delta Bh\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(+\Delta Bs\)`
</td>
<td style="text-align:left;">
</td>
<td style="text-align:left;">
`\(-\Delta Bcb\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
<tr>
<td style="text-align:left;">
`\(\sum\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
<td style="text-align:left;">
`\(0\)`
</td>
</tr>
</tbody>
</table>

## The model

To simulate the model, we first write down the equations and the external values with the `sfcr_set()` function:

``` r
pc_eqs <- sfcr_set(
  Y ~ C + G,
  YD ~ Y - TX + r[-1] * Bh[-1],
  TX ~ theta * (Y + r[-1] * Bh[-1]),
  V ~ V[-1] + (YD - C),
  C ~ alpha1 * YD + alpha2 * V[-1],
  Hh ~ V - Bh,
  Hh1 ~ V * ((1 - lambda0) - lambda1 * r + lambda2 * ( YD/V )), # EQ 4.6A
  Bh ~ V * (lambda0 + lambda1 * r - lambda2 * ( YD/V )),
  Bs ~ Bs[-1] + (G + r[-1] * Bs[-1]) - (TX + r[-1] * Bcb[-1]),
  Hs ~ Hs[-1] + Bcb - Bcb[-1],
  Bcb ~ Bs - Bh
)

pc_ext <- sfcr_set(
  # Exogenous
  
  r ~ 0.025,
  G ~ 20,
  
  # Parameters
  
  alpha1 ~ 0.6,
  alpha2 ~ 0.4,
  theta ~ 0.2,
  lambda0 ~ 0.635,
  lambda1 ~ 0.05,
  lambda2 ~ 0.01
)
```

We then simulate the model to get the steady state values:

``` r
pc <- sfcr_baseline(
  equations = pc_eqs, 
  external = pc_ext, 
  periods = 70, 
  hidden = c("Hh" = "Hs")
)
```

The fact that the hidden equation is fulfilled already indicates that the accounting part of the model is water tight.

However, the `sfcr` package proposes a validation workflow where the user uses the simulated model and the balance-sheet and transactions-flow matrices to further validate that the model is consistent.

``` r
sfcr_validate(bs_pc, pc, "bs")
#> Water tight! The balance-sheet matrix is consistent with the simulated model.
```

``` r
sfcr_validate(tfm_pc, pc, "tfm")
#> Water tight! The transactions-flow matrix is consistent with the simulated model.
```

Given the portfolio choice equations, the model is well specified if equation 4.6A is equal to 4.6. We can use the base `all.equal()` function to check it:

``` r
all.equal(pc$Hh, pc$Hh1)
#> [1] TRUE
```

We can safely conclude that the baseline model is well specified. The `sfcr` package also allows the user to plot the DAG representation of the model and a Sankey’s diagram of the transactions-flow matrix. Let’s check it out:

## DAG

Let’s use the `sfcr_dag_cycles_plot()` function to see the structure of the model:

``` r
sfcr_dag_cycles_plot(pc_eqs)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

As can be seen, there’s a cycle in this model. The `sfcr` package relies on the `igraph` package to identify the best ordering of the equations, reducing the the its computational burden. The blocks of independent equations and the iterations needed at each block to simulate the model can be retrieved by calling the `sfcr_get_matrix()` and `sfcr_get_blocks()` equations.

Alternatively, it is also possible to use the `sfcr_dag_blocks_plot()` function to visualize this structure:

``` r
sfcr_dag_blocks_plot(pc_eqs)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

## Sankey’s diagram

Another functionality offered by the `sfcr` package is that it can generate a Sankey’s diagram representation of the transactions-flow matrix.

Here, it is crucial to have the matrix validated in order to assure that the diagram is correctly specified. Luckily, we already saw that it was the case earlier in this notebook:

``` r
sn <- sfcr_sankey(tfm_pc, pc)
htmlwidgets::onRender(sn, 'function(el) { el.querySelector("svg").removeAttribute("viewBox") }')
```

<div id="htmlwidget-1" style="width:672px;height:480px;" class="sankeyNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"links":{"source":[8,10,9,10,11,8,8,8,12,0,1,2,3,3,4,5,6,7],"target":[0,1,2,3,4,5,6,7,7,14,14,13,13,16,15,15,17,15],"value":[43.418278,20,63.418278,0.796236,0.300973,12.782708,2.863079,4.849477,2.863079,43.418278,20,63.418278,0.495263,0.300973,0.300973,12.782708,2.863079,7.712555]},"nodes":{"name":["Consumption","Govt. Expenditures","Income","Int. payments","CB profits","Taxes","Ch. Money","Ch. Bills","Households","Firms","Government","CB current","CB capital","Households","Firms","Government","CB current","CB capital"],"group":["Consumption","Govt. Expenditures","Income","Int. payments","CB profits","Taxes","Ch. Money","Ch. Bills","Households","Firms","Government","CB current","CB capital","Households","Firms","Government","CB current","CB capital"]},"options":{"NodeID":"name","NodeGroup":"name","LinkGroup":null,"colourScale":"d3.scaleOrdinal(d3.schemeCategory20);","fontSize":14,"fontFamily":null,"nodeWidth":15,"nodePadding":10,"units":"dollars","margin":{"top":null,"right":null,"bottom":null,"left":null},"iterations":32,"sinksRight":true}},"evals":[],"jsHooks":{"render":[{"code":"function(el) { el.querySelector(\"svg\").removeAttribute(\"viewBox\") }","data":null}]}}</script>

We can now move to simulation of different scenarios.

# Scenario 1: Increase in the rate of interest on bills

Let’s start by increasing the rate of interest on bills by 100 points:

``` r
shock1 <- sfcr_shock(
  variables = sfcr_set(
    r ~ 0.035
  ),
  start = 5,
  end = 60
)

pc2 <- sfcr_scenario(pc, scenario = shock1, periods = 60)
```

What happen in this scenario with the share of bills/money in household portfolios?

Since these ratios are not calculated when we simulate the scenario, we must calculate them now. We use the `dplyr` package from the `tidyverse` to manipulate the model:

``` r
pc2 <- pc2 %>%
  mutate(BV = Bh / V,
         MV = Hh / V)
```

We reshape the model into the long format, and then we plot.

``` r
pc2_long <- pc2 %>%
  pivot_longer(cols = -period)

pc2_long %>%
  filter(name %in% c("BV", "MV")) %>%
  ggplot(aes(x = period, y = value)) +
  geom_line() +
  facet_wrap(~ name, scales = 'free_y') +
  labs(title = "Wealth alocation")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />

``` r
pc2_long %>%
  filter(name %in% c("YD", "C")) %>%
  ggplot(aes(x = period, y = value)) +
  geom_line(aes(linetype = name)) +
  labs(title = "Evolution of disposable income and household consumption",
       subtitle = "Following an increase of 100 points in the rate of interest")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-15-1.png" width="672" />

## Visualizing the sensitivity to parameters’ values

Let’s check the sensitivity of the model with the `sfcr_expand()` and `sfcr_multis()` functions:

``` r
values <- seq(0.61, 0.8, 0.01)
alpha1Exp <- sfcr_expand(x = pc_ext, variable = alpha1, values = values)
#
```

We then use the `sfcr_multis()` function to generate multiple baseline models:

``` r
pc_multi_bl <- sfcr_multis(alpha1Exp, pc_eqs, periods = 50)
```

Finally, apply the `shock1` shock to all the models in `pc_multi_bl`:

``` r
shock1 <- sfcr_shock(
  variables = sfcr_set(
    r ~ 0.035
  ),
  start = 5,
  end = 50
)

pc_multi_scn <- sfcr_multis(pc_multi_bl, shock1, periods = 50)
```

``` r
# We also need to expand the color palettes
colourCount <- 20 # number of levels
getPalette <- grDevices::colorRampPalette(RColorBrewer::brewer.pal(6, "GnBu")[c(3:6)])

bind_rows(pc_multi_scn) %>%
  mutate(simulation = as_factor(simulation)) %>%
  pivot_longer(cols = -c(period, simulation)) %>%
  filter(name %in% c("Y", "YD", "C")) %>%
  ggplot(aes(x = period, y = value, color = simulation)) +
  geom_line() +
  theme(legend.position = "bottom") +
  scale_color_manual("alpha10",
    values = getPalette(20),
    labels = as.character(values)) +
  facet_wrap(~ name, scales = 'free_y')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-19-1.png" width="864" />

We can see that the changing the `alpha1` parameter affects the level of the variables but do not change its underlying dynamics.

# References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-godley2007monetary" class="csl-entry">

Godley, Wynne, and Marc Lavoie. 2007. *Monetary Economics: An Integrated Approach To Credit, Money, Income, Production and Wealth*. Palgrave Macmillan.

</div>

</div>
