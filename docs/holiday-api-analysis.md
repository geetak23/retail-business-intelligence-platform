# Holiday API Analysis

## Source

Nager.Date Public Holiday API

## Endpoint

PublicHolidays/{year}/US

## Purpose

Provide holiday and calendar context for analyzing potential changes
in retail sales around public holidays.

## Observed Response Grain

One record per holiday occurrence and applicable
jurisdiction/type combination.

## Observed Fields

| Field | Data Type | Description |
|---|---|---|
| date | date | Holiday date |
| localName | string | Local holiday name |
| name | string | Holiday name |
| countryCode | string | Country code |
| fixed | boolean | Whether the holiday date is fixed |
| global | boolean | Whether the holiday applies globally |
| counties | array/null | Applicable regional codes |
| launchYear | integer/null | Holiday introduction year |
| types | array | Holiday classifications |

## Important Observation

The API can return multiple records for the same date.
Holiday applicability can vary by jurisdiction and holiday type.

Texas-specific applicability must therefore be evaluated using the
`counties` and `global` fields rather than assuming every US holiday
applies to Texas.

## Sample Period

2025

## Country

United States (US)