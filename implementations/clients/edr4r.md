# edr4r

This page shows how to use [edr4r](https://github.com/ksonda/edr4r), a tidy [R](https://www.r-project.org/) client, to connect to an API that implements OGC API - Environmental Data Retrieval - Part 1: Core.

edr4r is general purpose, but is most commonly used against in-situ monitoring networks (stream gauges, weather stations, snow and reservoir telemetry) that expose their stations and time series as EDR collections. It handles discovery, request construction (including WKT coordinate encoding and retries), and parsing of responses into tidy R objects: CoverageJSON becomes long-format tibbles, GeoJSON becomes `sf` objects, and CSV is parsed directly into tibbles.

## Links

- [edr4r on CRAN](https://cran.r-project.org/package=edr4r)
- [edr4r source and documentation](https://github.com/ksonda/edr4r)

## Software version

This description uses the latest CRAN release of edr4r. It requires R ≥ 4.1; the [`sf`](https://r-spatial.github.io/sf/) package is optional but recommended for spatial output.

## Required and supported Conformance classes

The API must support the following conformance classes:

- [Core](https://www.opengis.net/spec/ogcapi-edr-1/1.2/req/core)
- [Queries](https://www.opengis.net/spec/ogcapi-edr-1/1.2/req/queries)
- [JSON](https://www.opengis.net/spec/ogcapi-edr-1/1.2/req/json)

The client parses the following response encodings when a collection advertises them:

- [EDR GeoJSON](https://www.opengis.net/spec/ogcapi-edr-1/1.2/req/edr-geojson) (returned as `sf` objects)
- [CoverageJSON](https://www.opengis.net/spec/ogcapi-edr-1/1.2/req/covjson) (returned as long-format tibbles)
- CSV (returned as tibbles)

edr4r supports the `locations`, `items`, `position`, `area`, `cube`, `radius`, `trajectory`, and `corridor` query types (subject to server-side support advertised in the collection's `data_queries`).

## Installation

```r
# From CRAN
install.packages("edr4r")

# Development version from GitHub
# install.packages("pak")
pak::pak("ksonda/edr4r")
```

## Examples

You first create a client bound to an EDR landing page, then pass that client to the
discovery and query functions. The collection is given as the second (positional) argument.

```r
library(edr4r)

# Bind a client to an EDR landing page
client <- edr_client("https://api.waterdata.usgs.gov/ogcapi/beta")

# Discover collections
collections <- edr_collections(client)

# Locations (stations) as an sf object
sites <- edr_locations(
  client, "daily-edr",
  bbox = c(-109.06, 36.99, -102.04, 41.00)   # minx, miny, maxx, maxy
)

# Retrieve a timeseries for a single station as a tidy tibble
ts <- edr_location(
  client, "daily-edr",
  location_id    = "09058000",
  parameter_name = "00060",                   # discharge
  datetime       = "2020-01-01/2020-12-31",
  limit          = 200
)

# Position query by coordinates
pos <- edr_position(
  client, "daily-edr",
  coords         = "POINT(-105.27 40.01)",
  parameter_name = "00060"
)
```

`edr_location_batch()` runs sequential multi-station requests with per-station error
collection (`on_error = "collect"`) and can checkpoint and resume long multi-window pulls.
The parsers `covjson_to_tibble()` and `geojson_to_sf()` are exposed for responses obtained by
other means, `edr_capabilities()` / `edr_supports()` / `edr_diagnose()` report which query
types a collection advertises, and `edr_plot()` / `edr_map()` / `edr_explore()` provide
ggplot2 and Leaflet visualization of results.

## Known working endpoints

edr4r is regularly exercised against:

- [USGS Water Data OGC API](https://api.waterdata.usgs.gov/ogcapi/v0) (stream gauges and water-quality stations)
- [Western Water Datahub](https://api.wwdh.internetofwater.app/) (RISE, SNOTEL, USACE, and AWDB sources)
- [Met Office Labs EDR demonstrator](https://labs.metoffice.gov.uk/edr/)

## Contact

konda [at] lincolninst.edu
