# Data not included

The analysis uses two files that are not included in this repository because of their size (about 1.2 GB together) and data-use restrictions:

- `combin-all.csv`: raw voyage-level records of dry bulk vessels (about 3.8 million voyages, January 2013 to June 2025), including departure country, distance, speed, fuel consumption by type (VLSFO, HSFO, MGO), CO2 emissions and deadweight tonnage.
- `prepared_panel_data.csv`: the balanced vessel-month panel built from these records (15,781 vessels, 1,755,789 observations).

To reproduce the results, place both files in this `Data/` folder and run the scripts in `Scripts/`. Contact the authors for details on data access.
