Chipotle
================
Waveley Qiu
2021-11-22

## A. Data Import and Cleaning

First, let’s scrape all Chipotle locations in NY from the company
website.

``` r
## chipotle url -- NY locations
chipotle_url <- "https://locations.chipotle.com/ny/new-york"

## read in url
chipotle_html <-
  read_html(chipotle_url)

## helper function to parse CSS key from website
get_chipotle_data <- function(key){
  node <- 
    chipotle_html %>%
    html_elements(key) %>%
    html_text()
  return(node)
}
```

We want to extract store name, street address, and store hours from the
website. Store hours are not contained in a tidy way on the website, so
we will first include just store name (`name`) and street address
(`location`) in our working dataframe.

``` r
# location (street address), branch name (store name)
chipotle_loc <- get_chipotle_data(".c-address-street-1")
chipotle_branchname <- get_chipotle_data(".LocationName")

chipotle_df <-
  tibble(
    name = chipotle_branchname,
    location = chipotle_loc
  )
```

Now, we will work on extracting store hours. First, we will read in the
entirety of the main body of the webpage to see if the structure of the
data might allow us to extract hours.

``` r
first_hours <-
  tibble(
    raw = get_chipotle_data("#main span")
  )
```

In this single-column dataset, it seems that all store information is
presented in predictable chunks, with store hours being listed in rows
immediately after location name. We will use the lag function to
associate location names to the store hours. After this, we will perform
a left join to our working dataset (using store location as the common
identifier) to produce our desired dataset.

``` r
temp_chipotle <-
  first_hours %>% 
  mutate(
    id = lag(raw, 2)
  )

# nice and clean
chipotle_df <-
  chipotle_df %>% 
    left_join(
    temp_chipotle,
    by = c("name" = "id")
  ) %>%
  mutate(
    hours = trimws(raw, which = "both")
  ) %>%
  select(-raw)
```
