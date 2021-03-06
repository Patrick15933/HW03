Introduction
============

Hi! So as the README.md says, I don’t really have any personal data to
practice graphing with, so I decided to go with looking at some Covid
data! Totally not stressful at all.

Where did the data come from?
-----------------------------

All of the data from this Excel sheet comes from several different
sources:

-   The population data comes from the U.S. Census’ prediction of Texas
    counties, which can be found
    [here](https://www2.census.gov/programs-surveys/popest/tables/2010-2019/counties/totals/co-est2019-annres-48.xlsx)

-   Texas county land areas (in square miles) was taken from the
    Wikipedia page
    [here](https://en.wikipedia.org/wiki/List_of_counties_in_Texas)

-   Most importantly, the Covid-19 data comes from The New York Times’
    [GitHub page](https://github.com/nytimes/covid-19-data)

Graphing
--------

Just need to do some tidying up here…

    column_names_tx <- c("date", "text", "skip", "skip", "numeric", "numeric", "skip", "numeric", "numeric")
    texas_covid <- read_xlsx("us-counties covid 7-18-2020.xlsx", sheet = "texas_density_covid", col_types = column_names_tx)
    texas_covid$county <- as.factor(texas_covid$county) #this coerces the "county" column to be made up of 20 factors
    texas_covid$date <- as.Date(texas_covid$date) #coerces "date" column to be read by R as a date, not a character string
    names(texas_covid)[names(texas_covid) == "cases/100,000"] <- "cases_per_ht" #this changes the name of the cases/100,000 column, as its previous name was causing difficulties
    names(texas_covid)[names(texas_covid) == "deaths/100,000"] <- "deaths_per_ht" #same with this one

Okay! Here is the code for the graph:

    gal_nue <- c(1169:1294, 1923:2037) #This subset selects for Galveston and Nueces counties - when I used a character vector to select for the two, it would only select for every other row of that data. I have no idea why.
    other_counties <- c(1:1168, 1295:1922, 2038: 2526) #This selects for all other counties

    ggplot(texas_covid, aes(date, cases_per_ht)) +
      geom_point(alpha = 0.2, color = "darkblue") +
      geom_point(data = texas_covid[gal_nue, ], alpha = 0.5, color = "red4") +
      stat_smooth(data = texas_covid[other_counties, ], se = F, color = "darkblue") +
        stat_smooth( #curve for outliers
          data = texas_covid[1169:1294, ], 
          color = "red4",
          se = F) + 
        stat_smooth(
          data = texas_covid[1923:2037, ],
          color = "red4",
          se = F) +
      scale_x_date(limits = as.Date(c("2020-05-10", NA))) + #I decided to limit the scope of the graph because the early dates lacked important information
      scale_y_continuous("Cases / 100,000", expand = c(0, 0), limits = c(0, 2500)) +
      labs(
        title = "Covid-19 cases in the most densely populated Texas counties",
        x = "Date"
      ) +
      coord_cartesian(xlim = as.Date(c("2020-05-15", "2020-07-16"))) +
        annotate("text", x = as.Date("2020-07-01"), y = 2250, label = "Nueces County") +
        annotate("curve", x = as.Date("2020-07-07"), y = 2200, xend = as.Date("2020-07-14"), yend = 2050,
            arrow = arrow(length = unit(0.2, "cm"), type = "closed")) + #Nueces text and curve
        annotate("text", x = as.Date("2020-06-26"), y = 1650, label = "Galveston County") +
        annotate("curve", x = as.Date("2020-06-28"), y = 1550, xend = as.Date("2020-07-05"), yend = 1300,
            arrow = arrow(length = unit(0.2, "cm"), type = "closed")) + #Galveston text and curve
        annotate("text", x = as.Date("2020-07-01"), y = 100, label = "All other counties") +
        annotate("curve", x = as.Date("2020-07-08"), y = 150, xend = as.Date("2020-07-10"), yend = 700,
            arrow = arrow(length = unit(0.2, "cm"), type = "closed")) + #Galveston text and curve
      geom_vline(xintercept = as.Date("2020-06-03"), linetype = "longdash") +
      geom_vline(xintercept = as.Date("2020-06-17"), linetype = "longdash") +
        annotate("text", x = as.Date("2020-05-30"), y = 1800, label = "Phase III") +
        annotate("text", x = as.Date("2020-06-11"), y = 1150, label = "Two Weeks\npost-Phase III") +
      theme_classic()

![](Covid-Data-Import-and-Graph_files/figure-markdown_strict/Graph%201-1.png)

-   I tried to put labels for the trend lines to the right of the plot,
    but I couldn’t figure out how :(

-   I found out that there is a lot of trial-and-error involved with
    positioning labels - maybe it would be better to wait until the plot
    is practically complete before adding in the theme (that gets rid of
    the grid lines) so that I can better measure where to place the
    coordinates

    -   It also doesn’t help that this graph uses dates as the x-axis,
        which makes changing those values more tedious

When making this graph, I noticed that there were clear outliers. By
looking more closely at the data, it was clear that two counties (Nueces
and Galveston) had much higher rates of infection than other counties.

-   A quick look at both counties’ wiki pages show that they are not
    bordering each other (which rules out the two counties being part of
    single case of increased infection). Both counties actually have
    quite dissimilar population densities (Nueces = 433 ppl/sq mi,
    Galveston = 857 ppl/ sq mi), and they certainly are not nearly as
    dense as Dallas and Harris counties, which are nearly 3,000
    ppl/sq mi.

-   However, what they do have in common is that they are coastal
    counties. That might have something to do with this?
