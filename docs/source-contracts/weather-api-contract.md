# Weather API Source Contract

## Source

Open-Meteo Historical Weather API

## Purpose

Provide daily environmental context that can later be joined with
retail sales data to analyze the relationship between weather conditions
and sales performance.

## API Data

The source provides daily weather observations for a specified location
and date range.

## Required Fields

| API Field | Target Meaning | Unit | Data Type |
|---|---|---|---|
| time | Weather date | ISO date | date |
| temperature_2m_max | Maximum daily temperature | °F | decimal |
| temperature_2m_min | Minimum daily temperature | °F | decimal |
| precipitation_sum | Total precipitation | mm | decimal |
| rain_sum | Total rainfall | mm | decimal |
| wind_speed_10m_max | Maximum wind speed | mph | decimal |

## Location

Dallas, Texas

Latitude: 32.794376  
Longitude: -96.8215

## Time Zone

America/Chicago

## Initial Sample Period

2025-01-01 through 2025-01-05

## Expected Grain

One record per location per calendar date.

## Initial Business Use

Weather data will eventually be joined with retail sales data by date
and geographic location to support analysis such as:

- Sales performance during rainy days
- Sales performance during extreme temperatures
- Weather-related changes in customer demand