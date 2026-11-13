# IotaWatt

IotaWatt

Author

Benedict Thekkel

## Step 1

- gcc + vscode hello world
- influx server (endpoint v1)
- get ioitawat code influxUploader.cpp in repo
- adapt to run locally at influx

------------------------------------------------------------------------

## Step 2

- Install postgresql and timescale (docker ok)
- Get phisaver repo
- Run phitimescale:
- [TimeScaleDB](https://brettbeeson.com.au/timescaledb/)
- [PostgreSQL Tricks](https://brettbeeson.com.au/postgres-tricks/)
- optionally pgAdmin4 desktop (or vscode probs better)

------------------------------------------------------------------------

## Step 2.1: write SQL to insert data

- Step 2+
- ! write the code sqlUploader.cpp ! against psql
- POST request (as influxUploader.cpp) in SQL syntax (see insert_reading in PhiTimeScale.py)
- test, etc

------------------------------------------------------------------------

Back to top
