# Description
add Latitude and Longitude coordinates to csv metadata spreadsheets based on State and County values. It uses a reference worksheet containing geolocation mappings sourced from [U.S. Census Gazetteer Files](https://www.census.gov/geographies/reference-files/time-series/geo/gazetteer-files.html) and appends the resulting coordinates to the main data sheet without overwriting any existing data.

## How to Enable Macros in Excel

- When opening the Lat-LongCalculator.xlsm file, click "Enable Macros" if prompted with "This workbook contains macros. Do you want to disable macros before opening the file?"

## Required Inputs
 ### Main Data Sheet (Titled: Sheet1)
A dataset with at least two labeled columns:

- State
- County

### Reference Data Sheet (Titled: ReferenceData)
Must contain the following four pre-filled columns:

- State
- County
- Latitude
- Longitude

## How to Run the Macro

- On "Sheet1" Click the "Run Lat/Long Calculator" button
- After a few seconds you will see a success prompt "Latitude and Longitude added successfully!"

OR

- Go to the Developer tab.
- Click "Macros"
- Then click "Run"
- After a few seconds you will see a success prompt "Latitude and Longitude added successfully!"


## Outputs
- Adds two new columns: Latitude and Longitude
- Populated using matches from the ReferenceData sheet








