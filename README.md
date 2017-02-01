Download all of Donald Trump's tweets using R
---------------------------------------------

``` r
## install rtweet package
install.packages("rtweet")

## load rtweet
library(rtweet)

## function to scrape IDs
.trumpids <- function(year) {
    ## build url
    url <- paste0("http://trumptwitterarchive.com/",
                  "data/realdonaldtrump/", year, ".json")
    ## return ids
    jsonlite::fromJSON(url)[["id_str"]]
}
## function to download status ids
trumpids <- function(trumptwitterarchive = FALSE) {
    if (trumptwitterarchive) {
        ids <- c(2009:2017) %>%
            lapply(.trumpids) %>%
            unlist(use.names = FALSE)
    } else {
        ids <- "data/realdonaldtrump-ids-2009-2017.csv" %>%
            read.csv(stringsAsFactors = FALSE) %>%
            unlist(use.names = FALSE)
    }
   ids
}
## function to download twitter data
trumptweets <- function() {
    ## get archive of status ids
    ids <- trumpids()
    ## get newest trump tweets
    rt1 <- get_timeline(
        "realdonaldtrump", since_id = ids[length(ids)])
    ## download archive
    message("    Downloading ", length(ids), " tweets...")
    rt2 <- lookup_statuses(ids[1:16000])
    message("    You're halfway there...")
    rt3 <- lookup_statuses(ids[16001:(length(ids))])
    message("    Huzzah!!!")
    rbind(rt1, rt2, rt3)
}

## run function to download Trump's twitter archive
djt <- trumptweets()

## save as excel file
install.packages("openxlsx")
openxlsx::write.xlsx(djt, "realdonaltrump-fullarchive.xlsx")

## save as csv file
write.csv(djt, "data/realdonaltrump-fullarchive.csv",
          row.names = FALSE)
```

Inspecting the data
-------------------

``` r
## check out 100 most popular hashtags
djt$hashtags %>%
    strsplit(" ") %>%
    unlist(use.names = FALSE) %>%
    tolower %>%
    table() %>%
    sort(decreasing = TRUE) %>%
    head(100)
## 100 more popular mentions
djt$mentions_screen_name %>%
    strsplit(" ") %>%
    unlist(use.names = FALSE) %>%
    tolower %>%
    table() %>%
    sort(decreasing = TRUE) %>%
    head(100)
## view test of 10 most recent tweets
djt$text[1:10]
```

Plotting the data
-----------------

``` r
## plot four groups of hashtags
p <- ts_filter(djt, "2 days", txt = "hashtags",
               filter = c("makeamericagreatagain|maga",
                          "trump",
                          "debate",
                          "draintheswamp|americafirst"),
               key = c("MakeAmericaGreatAgain",
                       "Trump",
                       "Debates",
                       "DrainTheSwamp/AmericaFirst"))

## install and load ggplot2
install.packages("ggplot2")
library(ggplot2)

## uncomment following line to save image
## png("trumptweets.png", 7, 5, "in", res = 127.5)
p %>%
    ggplot(aes(x = time, y = freq, color = filter)) +
    theme_bw() +
    geom_line() +
    facet_wrap( ~ filter, ncol = 2) +
    labs(x = "", y = "",
         title = "Hashtags used by Donald Trump",
         subtitle = "Used entire archive of @realDonaldTrumpTweets") +
    theme(legend.position = "none",
          text = element_text(size = 12,
                              family = "Avenir Next Condensed"),
          plot.title = element_text(
              family = "Avenir Next Condensed Medium", size = 20))
## dev.off()
## uncomment previous line to save image
```

<p align="center">
<img src="trumptweets.png" alt="trumphashtags">
</p>
