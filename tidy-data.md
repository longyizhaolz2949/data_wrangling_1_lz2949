tidy data
================

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

## PULSE data

``` r
pulse_df = 
  haven::read_sas("data/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  pivot_longer(bdi_score_bl:bdi_score_12m,
               names_to = "visit",
               values_to = "bdi_scores", 
               names_prefix = "bdi_score_"
               ) |>
      mutate(
        visit = replace(visit, visit == "bl", "00m")
      )
```

## learning assessment

In the litters data, the variables gd0_weight and gd18_weight give the
weight of the mother mouse on gestational days 0 and 18. Write a data
cleaning chain that retains only litter_number and these columns;
produces new variables gd and weight; and makes gd a numeric variable
taking values 0 and 18 (for the last part, you might want to use recode
…).

import . kentgthen the litters dataset

``` r
litters_df = 
  read_csv("./data/FAS_litters.csv") |>
  janitor::clean_names() |>
  select(litter_number, gd0_weight, gd18_weight) |>
  pivot_longer(
    gd0_weight:gd18_weight,
    names_to = "gd",
    values_to = "weight",
    names_prefix = "gd"
  ) |> 
  mutate (
    gd= case_match (
      gd, 
      "0_weight" ~ 0, 
      "18_weight" ~ 18 # any time see 18 weight, replace it with 18
    )
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## LoTR

import LoTR wirds data

``` r
fellowship_df = 
  readxl::read_excel('data/LotR_Words.xlsx', range = "B3:D6") |>
  mutate(movie = "fellowship")


two_towers_df = 
  readxl::read_excel('data/LotR_Words.xlsx', range = "F3:H6") |>
  mutate(movie = "two towers")

return_of_the_king_df = 
  readxl::read_excel('data/LotR_Words.xlsx', range = "J3:L6") |>
  mutate(movie = "return of the king")


lotr_df = 
  bind_rows (fellowship_df, two_towers_df, return_of_the_king_df) |>
  janitor::clean_names() |>
  pivot_longer (
    male:female, 
    names_to = "gender",
    values_to = "word" ) |>
  relocate(movie)
```

## Revist FAS

``` r
litters_df = 
  read_csv("data/FAS_litters.csv") |>
  janitor::clean_names() |>
  mutate(wt_gain = gd18_weight - gd0_weight)|>
  select(litter_number, group, wt_gain) |>
  separate(group, into = c("does", "day_of_tx"), 3)
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df = 
  read_csv("data/FAS_pups.csv") |>
  janitor::clean_names() |>
  mutate (
    sex = case_match(
      sex,
      1 ~ "male", 
      2 ~ "female"
    )
  )
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number")
```
