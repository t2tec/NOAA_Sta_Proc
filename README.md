# NOAA Observation Station Processing
# Creating Master Observation Station Lists

## Summary

The **2016 Weather Data Exploratory Analysis** project was started to review the raw data from **NOAA** and identify areas of uncertainty and their potential impact on reaching a greater than 95% scientific certainty.

This is **Part 1** of the **2016 Weather Data Exploratory Analysis**.

## Goal

The goal of the project is to create a set of master observation station lists from the NOAA stations files which will be used during the **2016 Weather Data Exploratory Analysis**.

The master lists will consist of observation stations that collect temperature data, their respective geographic coordinates, country, state and location information, and years of operation. Three master lists will be created for use in future exploratory analysis and saved in `.Rds` format:

+ `master_stations_ww.rds`
+ `master_stations_us50.rds`
+ `master_stations_us48.rds`