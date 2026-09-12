---
layout: post
title: Chicago Crime and Census Data
date: 12/09/2026
categories: EDA
tags: Data, EDA
---

# Introduction

I had been working my way through the IBM Data Engineering [programme](https://www.edx.org/certificates/professional-certificate/ibm-data-engineering) on EdX, when I was introduced to the Chicago [census hardship data](https://data.cityofchicago.org/Health-Human-Services/Selected-socioeconomic-indicators-by-neighborhood/i9hv-en6g/about_data) as part of a course. The plot that introduced the data was as the one found below, showing the hardship index as a function of per-capita income.

![Per Capita Income vs Hardship Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/01-IncomeVsHardship.png)

I thought this an interesting graph, and wondered what other data was available. I found both [population data](https://data.cityofchicago.org/Community-Economic-Development/ACS-5-Year-Data-by-Community-Area/t68z-cikk/data_preview) for 2023 and [crime data](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2/about_data) for 2001 to 2026.

One thing I want to attempt this year is to imporove my ability to work with Polars, Matplotlib, and Seaborn. I chose Polars over Pandas as it seems much more intuitive to use, with Polars commands mirroring SQL more than Pandas. Plus from what I have read Polars is better with larger datasets on constrained hardware.

The full [EDA session](https://github.com/ogladr-kjarr/learning-EDA-with-Python/blob/main/ChicagoCrimeSocioEconomicData.ipynb) is found on GitHub, it is meandering in nature, I didn't sit down with a clear goal of what to look for, rather I just went where I thought seemed more interesting. When I merged the census data to the crime data I did think of some questions to ask of the data. However, I do not have the Data Science know-how, let alone domain knowledge to answer those questions except very superficially and I suspect wrongly.

# Crime Data

The crime data starts in 2001 and ends in 2026, however full years of reports span 2003 to 2025. Some crimes do not have a community area to link to the census data, these were removed. Some crimes were renamed part way through the dataset, these were joined together.

The below plot is simply a count of all the number of crimes per category committed over the full breadth of the timespan of the dataset. They y-axis is on a log scale so as to better show the full distribution of counts. Before using a log scale the higher counts such as theft and battery made much of the rest of the categories non visible.

![Crime By Category Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/02-CrimeByCategory.png)

This plot shows that there is a large variation in the number of crimes committed per category. It is no surprise then when in the plot below it is shown the amount of times each crime was the most-recorded crime in a census area.

![Top Crime Categories Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/03-TopCrimeCategories.png)

As shown the top crimes in each area coincide with the top four in the previous plot showing crimes by category.

These plots are static though, just giving an overall count of crimes. What's more interesting is categories over time. The plot below shows the recorded number of crimes per year for the top four crimes in the dataset. It shows that there is a very large reduction in crime as the years progress. This was so stark a change I double checked to make sure I hadn't introduced an error somewhere. It looks like theft also had a large reduction during Covid restrictions.

![Crime By Year Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/04-CrimeByYear.png)

If we focus on the earliest year and latest year for which there is full data, we get the plot below. It shows much of what the previous plot showed, that there is a large reduction in many crimes, especialy theft. Shown is only a subset of all crimes though, specifically those categories that ended up with over 1000 reports each in 2025.

![Crime By Two Years Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/05-CrimeBySelectedYears.png)

I wondered if arrests would be up in 2025 over those in 2003 to account for the drop in crime, that the fear of arrest would deter crime, however in the plot below it is shown that the arrest ratio is actually down for most crimes in 2025 when compared to 2003.

![Arrest Rate By Two Years Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/06-ArrestRateByYears.png)

# Crime and Census Data

Joining the census data to the crime data allows for comparison between areas. The plot below shows the total crimes per area, while also showing the hardship and census indicators in plots below the crime plot. It shows that these census hardship indicators are not in themselves a good indicator of whether an area will have high crime or now.

![Crimes By Census Area Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/07-CrimesByAreaCensusData.png)

The next plot shows that population amount does go some way to explaining different areas crime rates. The issue I have is that I know nothing about these areas. I don't know if, for example, the highest popuation areas arise from housing density, which may be an indicator of increased crime. 

![Crimes By Population Plot](/assets/img/2026-09-12-Chicago-Crime-EDA/08-CrimeByPopulationArea.png)

Finally the plot below looks at census data and crime counts, per year, for the four highest recorded crimes. First it shows that theft is the only one of the top four crimes to increase as an area has more income. Second it shows what other plots have shown, a large decrease in these crimes between 2003 and 2025.

![Crimes By Income Quarted](/assets/img/2026-09-12-Chicago-Crime-EDA/09-CrimeByIncomeQuartet.png)


There are a number of other plots, and questions that I attempted to ask and answer from the data, all in the same [Jupyter notebook on GitHub](https://github.com/ogladr-kjarr/learning-EDA-with-Python/blob/main/ChicagoCrimeSocioEconomicData.ipynb).