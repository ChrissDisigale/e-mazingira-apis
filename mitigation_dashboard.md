# Climate Change Mitigation Dashboard Analytics API

## Introduction

Only active, non-deleted `MITIGATION_MEASURE` records with `approval_status=APPROVED` are included. Pending, reviewed, rejected, deleted, and inactive records are excluded.

## Base URL and security

```text
/api/v1/climate-change/mitigation/analytics
```

The endpoints use the existing IEMIS Spring Security/JWT configuration and gateway. Send:

```http
Authorization: Bearer <ACCESS_TOKEN>
Accept: application/json
```

No deployment hostname is hardcoded. Prefix the path with the environment's existing API/gateway base URL.

All successful responses use the existing wrapper:

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {}
}
```

## Endpoint reference

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/summary` | Separate CO2, CH4, N2O, and HFC totals; a single selected-gas total is available with `gasType` |
| GET | `/grouped` | One metric grouped by one category/dimension; donut and bar charts |
| GET | `/trend` | One metric grouped by activity year and dimension; line and area charts |
| GET | `/gas-composition` | Four gas fields side by side for a dimension; stacked/grouped bars |
| GET | `/series` | Several unit-preserving metrics for a dimension; dual-axis/composite charts |
| GET | `/heatmap` | One metric grouped by two dimensions; heatmaps and regional stacked bars |

This reusable contract covers the dashboard requirements without maintaining 170 near-duplicate routes.

## Common filters

| Parameter | Type | Meaning |
|---|---|---|
| `coverage` | enum | `TANZANIA_MAINLAND` or `ZANZIBAR` |
| `regionId` | long | Existing Region ID |
| `councilId` | long | Existing Council ID; validated against Region when both are present |
| `year` | integer | Exact `values.year` |
| `yearFrom`, `yearTo` | integer | Inclusive range; cannot be combined with `year` |
| `sectorId` | long | Existing Sector ID |
| `sectorMeasureUid` | string | Existing Sector Measure UUID |
| `code` | enum list | Repeat the parameter for several mitigation codes |
| `ownership` | string | Government, Private, Individual, Community, Village Government, or Public machine value |
| `technologyType` | string | Technology code such as `SOLAR_POWER` |
| `energySource` | string | Industry/transport energy-source code |
| `wasteType` | string | Exact currently stored waste type |
| `managementType` | string | Existing forest/livestock management code |
| `material` | string | Exact currently stored charcoal-production material |
| `gasType` | enum | `CO2`, `CH4`, `N2O`, or `HFCS`; meaningful on `/summary` |

Examples:

```http
GET /api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&coverage=TANZANIA_MAINLAND&year=2026
GET /api/v1/climate-change/mitigation/analytics/grouped?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2&yearFrom=2021&yearTo=2026
GET /api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

Historical records with no Coverage participate only when `coverage` is omitted; they are never silently classified as Mainland.

## Dimensions

`dimension`, `rowDimension`, and `columnDimension` accept:

```text
SECTOR
SECTOR_MEASURE
CODE
REGION
OWNERSHIP
TECHNOLOGY_TYPE
ENERGY_SOURCE
WASTE_TYPE
MANAGEMENT_TYPE
MATERIAL
```

Region is resolved from the record Region, falling back to Council → District → Region. Unclassified values are returned explicitly as `UNCLASSIFIED` instead of being discarded.

## Metrics and semantics

| Metric | Unit behavior | Aggregation |
|---|---|---|
| `QUANTITY_REDUCED_CO2` | `UNSPECIFIED_REPORTED_GAS_QUANTITY` | SUM |
| `QUANTITY_REDUCED_CH4` | same | SUM |
| `QUANTITY_REDUCED_N2O` | same | SUM |
| `QUANTITY_REDUCED_HFCS` | same | SUM |
| `INSTALLED_CAPACITY` | MW, m3, or Tonnes according to code | latest snapshot per entity |
| `PRODUCTION_CAPACITY` | MW or Tonnes according to code | latest snapshot per entity |
| `PRODUCTION_CAPACITY_PER_DAY` | m3/day or Tonnes/day | latest snapshot per entity |
| `CAPACITY` | Tonnes | latest snapshot per entity |
| `NATURAL_GAS_CONSUMED` | MMscfd | SUM |
| `ENERGY_CONSUMPTION` | preserves each nested source unit (MW/MMscfd) | SUM by unit |
| `ENERGY_CONSUMPTION_PER_DAY` | unit currently unspecified | SUM |
| `NETWORK_DISTANCE` | km | latest snapshot per entity |
| `COVERAGE_AREA` | Ha | latest snapshot per entity |
| `NUMBER_OF_TREES` | trees | latest snapshot per entity |
| `NUMBER_OF_BENEFICIARIES`, `POPULATION_SERVED` | people | latest snapshot per entity |
| `NUMBER_OF_HOUSEHOLDS` | households | latest snapshot per entity |
| `NUMBER_OF_INSTITUTIONS` | institutions | latest snapshot per entity |
| `VEHICLES_SERVED` | vehicles | latest snapshot per entity |
| `SORTING_CAPACITY`, `LANDFILL_CAPACITY`, `ORGANIC_WASTE_RECEIVING_CAPACITY` | Tonnes | latest snapshot per entity |
| `LANDFILL_SIZE` | Ha | latest snapshot per entity |
| `LIVESTOCK_PRODUCTION_CAPACITY` | unspecified domain unit | latest snapshot per entity |
| `FODDER_PRODUCTION_CAPACITY` | Tonnes | latest snapshot per entity |
| `DISTINCT_ENTITIES` | entities | COUNT DISTINCT derived identity |

Flow values are summed. Stock/snapshot values use the latest eligible record per facility/project identity inside the selected scope. Counts use distinct identity, not `COUNT(*)`. Where the schema has no stable entity ID, the repository derives the best available business key and returns a warning/limitation in the documentation.

## API details

### Summary

```http
GET /api/v1/climate-change/mitigation/analytics/summary?coverage=TANZANIA_MAINLAND&gasType=CO2
```

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": 125.5,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {"CO2": 125.5, "CH4": 4.2, "N2O": 1.1, "HFCS": 0},
    "yearSource": "values.year (activity-specific year)",
    "filters": {"coverage": "TANZANIA_MAINLAND", "codes": []},
    "warnings": ["Gas-reduction fields do not currently declare whether values are gas mass or CO2-equivalent. Different gases are not summed into tCO2e."]
  }
}
```

Without `gasType`, `total` is `null`; use `gasTotals`. This is intentional—CO2, CH4, N2O, and unidentified HFC mass cannot be truthfully added into tCO2e.

### Grouped comparison

```http
GET /api/v1/climate-change/mitigation/analytics/grouped?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2
```

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM",
    "data": [{"code": "4", "label": "Energy", "value": 100, "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY", "percentage": 80.00}],
    "filters": {"codes": []},
    "warnings": []
  }
}
```

Map `data[].label` to chart labels and `data[].value` to numeric values. Keep `data[].unit` visible. `percentage` is calculated only against rows with the same unit, so incompatible units are never mixed.

### Annual trend

```http
GET /api/v1/climate-change/mitigation/analytics/trend?dimension=CODE&metric=COVERAGE_AREA&yearFrom=2021&yearTo=2026
```

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "LATEST_SNAPSHOT_SUM",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [{"year": 2026, "code": "COMMUNITY_BASED_FOREST_MANAGEMENT", "label": "Community Based Forest Management (CBFM)", "value": 200, "unit": "Ha"}],
    "filters": {"yearFrom": 2021, "yearTo": 2026, "codes": []},
    "warnings": []
  }
}
```

Group frontend series by `code`, order points by `year`, and leave absent observations as missing/null rather than generating zeroes.

### Gas composition

```http
GET /api/v1/climate-change/mitigation/analytics/gas-composition?dimension=SECTOR
```

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [{"code": "4", "label": "Energy", "co2": 100, "ch4": 20, "n2o": 5, "hfcs": 1, "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"}],
    "filters": {"codes": []},
    "warnings": []
  }
}
```

Use `label` as X-axis/category and the four named fields as separate series. Do not stack into a tCO2e total until the data model defines conversions.

### Unit-preserving multi-metric series

```http
GET /api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "TECHNOLOGY_TYPE",
    "series": [
      {"key": "INSTALLED_CAPACITY", "label": "Installed capacity", "aggregation": "LATEST_SNAPSHOT_SUM", "data": [{"code": "SOLAR_POWER", "label": "Solar Power", "value": 12, "unit": "MW", "percentage": 100.00}]},
      {"key": "PRODUCTION_CAPACITY", "label": "Production capacity", "aggregation": "LATEST_SNAPSHOT_SUM", "data": [{"code": "SOLAR_POWER", "label": "Solar Power", "value": 8, "unit": "MW", "percentage": 100.00}]}
    ],
    "filters": {"codes": ["CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION"]},
    "warnings": []
  }
}
```

Create one chart series per `series[]`. Never merge rows whose `unit` differs.

### Heatmap / two-dimensional grouping

```http
GET /api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "OWNERSHIP",
    "metric": "DISTINCT_ENTITIES",
    "rows": [{"code": "1", "label": "Dodoma", "value": null, "unit": null, "percentage": null}],
    "columns": [{"code": "GOVERNMENT", "label": "Government", "value": null, "unit": null, "percentage": null}],
    "cells": [{"rowCode": "1", "columnCode": "GOVERNMENT", "value": 2, "unit": "entities"}],
    "filters": {"codes": ["CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION"]},
    "warnings": []
  }
}
```

Join `cells[].rowCode` and `columnCode` to the corresponding label arrays.

## Dashboard chart-to-API mapping

All paths below are relative to:

```text
/api/v1/climate-change/mitigation/analytics
```

`{GAS_METRIC}` means one of `QUANTITY_REDUCED_CO2`, `QUANTITY_REDUCED_CH4`, `QUANTITY_REDUCED_N2O`, or `QUANTITY_REDUCED_HFCS`. These must remain separate until the data-entry model defines compatible CO2e units.

### National overview

| Chart name | API | Recommended chart |
|---|---|---|
| Total GHG Emissions Reduced | `/summary?gasType={CO2|CH4|N2O|HFCS}` | KPI |
| Quantity of Emissions Reduced by Sector | `/grouped?dimension=SECTOR&metric={GAS_METRIC}` | Donut |
| Annual Trend of Emissions Reduced by Sector | `/trend?dimension=SECTOR&metric={GAS_METRIC}` | Area/line |
| GHG Reductions by Sector and Gas Type | `/gas-composition?dimension=SECTOR` | Grouped/stacked bar |
| Annual Trend by Gas Type | `/trend?dimension=SECTOR&metric={GAS_METRIC}` | Multi-line |

### Energy dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total GHG Emissions Reduced in Energy | `/summary?sectorId={ENERGY_ID}&gasType={GAS}` | KPI |
| Emissions Reduced by Energy Mitigation Measure | `/grouped?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={ENERGY_ID}` | Donut/bar |
| Annual Energy Reduction Trend by Measure | `/trend?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={ENERGY_ID}` | Area/line |
| Energy Measures by Gas Type | `/gas-composition?dimension=SECTOR_MEASURE&sectorId={ENERGY_ID}` | Stacked bar |
| Emissions Reduced by Clean Technology | `/grouped?dimension=TECHNOLOGY_TYPE&metric={GAS_METRIC}&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Bar/donut |
| Clean Technology Reduction Trend | `/trend?dimension=TECHNOLOGY_TYPE&metric={GAS_METRIC}&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Line |
| GHG Composition by Clean Technology | `/gas-composition?dimension=TECHNOLOGY_TYPE&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Grouped bar |
| Installed and Production Capacity by Technology | `/series?dimension=TECHNOLOGY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Grouped bar |
| Annual Clean Technology Capacity Trend | `/trend?dimension=TECHNOLOGY_TYPE&metric={INSTALLED_CAPACITY|PRODUCTION_CAPACITY}&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Line |
| Clean Technology Capacity by Region | `/series?dimension=REGION&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Regional grouped bar |
| Ownership Distribution by Clean Technology | `/heatmap?rowDimension=TECHNOLOGY_TYPE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Heatmap/stacked bar |
| Clean Technology Ownership by Region | `/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Heatmap |
| Emissions Reduced by Natural Gas Activity | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=POWER_PRODUCTION&code=COOKING&code=TRANSPORTATION` | Donut |
| Natural Gas Activity Reduction Trend | `/trend?dimension=CODE&metric={GAS_METRIC}&code=POWER_PRODUCTION&code=COOKING&code=TRANSPORTATION` | Line |
| Natural Gas Activity by Gas Type | `/gas-composition?dimension=CODE&code=POWER_PRODUCTION&code=COOKING&code=TRANSPORTATION` | Stacked bar |
| Power Production Capacity | `/series?dimension=CODE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=POWER_PRODUCTION` | Grouped bar |
| Regional Power Production Capacity | `/series?dimension=REGION&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=POWER_PRODUCTION` | Regional bar |
| Households and Institutions Using Natural Gas | `/series?dimension=CODE&metrics=NUMBER_OF_HOUSEHOLDS&metrics=NUMBER_OF_INSTITUTIONS&code=COOKING` | Grouped bar |
| Annual Household and Institution Trend | `/trend?dimension=CODE&metric={NUMBER_OF_HOUSEHOLDS|NUMBER_OF_INSTITUTIONS}&code=COOKING` | Line |
| Households and Institutions by Region | `/heatmap?rowDimension=REGION&columnDimension=CODE&metric={NUMBER_OF_HOUSEHOLDS|NUMBER_OF_INSTITUTIONS}&code=COOKING` | Heatmap |
| Natural Gas Stations by Ownership | `/grouped?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=TRANSPORTATION` | Bar/donut |
| Natural Gas Stations by Region and Ownership | `/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=TRANSPORTATION` | Stacked bar |
| Rural Electrification Reduction by Category | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=MICRO_RENEWABLE_GENERATION&code=MINI_GRID_RENEWABLE_GENERATION` | Donut/bar |
| Rural Electrification Capacity | `/series?dimension=CODE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=MICRO_RENEWABLE_GENERATION&code=MINI_GRID_RENEWABLE_GENERATION` | Grouped bar |
| Regional Rural Electrification Capacity | `/series?dimension=REGION&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=MICRO_RENEWABLE_GENERATION&code=MINI_GRID_RENEWABLE_GENERATION` | Regional bar |

### Transport dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total Transport GHG Reductions | `/summary?sectorId={TRANSPORT_ID}&gasType={GAS}` | KPI |
| Emissions Reduced by Transport Measure | `/grouped?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={TRANSPORT_ID}` | Donut/bar |
| Transport Measure Reduction Trend | `/trend?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={TRANSPORT_ID}` | Area/line |
| Transport Measures by Gas Type | `/gas-composition?dimension=SECTOR_MEASURE&sectorId={TRANSPORT_ID}` | Stacked bar |
| Rail and Road Infrastructure Reductions | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=RAILWAY&code=ROAD_INFRASTRUCTURE` | Horizontal bar |
| Rail Network Distance and Capacity by Type | `/series?dimension=TECHNOLOGY_TYPE&metrics=NETWORK_DISTANCE&metrics=CAPACITY&code=RAILWAY` | Dual-axis bar |
| Annual Rail Distance and Capacity Trend | `/trend?dimension=TECHNOLOGY_TYPE&metric={NETWORK_DISTANCE|CAPACITY}&code=RAILWAY` | Line |
| Road Infrastructure Distance by Type | `/grouped?dimension=TECHNOLOGY_TYPE&metric=NETWORK_DISTANCE&code=ROAD_INFRASTRUCTURE` | Bar |
| Renewable Transport Reduction by Activity | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=BUS_RAPID_TRANSIT&code=LOCOMOTIVES&code=PRIVATE_VEHICLES` | Donut/bar |
| Renewable Transport Activity by Gas Type | `/gas-composition?dimension=CODE&code=BUS_RAPID_TRANSIT&code=LOCOMOTIVES&code=PRIVATE_VEHICLES` | Stacked bar |
| BRT Facilities by Energy Type | `/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=BUS_RAPID_TRANSIT` | Bar |
| Regional BRT Facilities by Energy Type | `/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=BUS_RAPID_TRANSIT` | Heatmap |
| Locomotives by Energy Type | `/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=LOCOMOTIVES` | Bar |
| Regional Locomotives by Energy Type | `/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=LOCOMOTIVES` | Heatmap |
| Vehicles by Energy Type and Ownership | `/heatmap?rowDimension=ENERGY_SOURCE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=PRIVATE_VEHICLES` | Heatmap/stacked bar |
| Regional Vehicles by Energy Type | `/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=PRIVATE_VEHICLES` | Heatmap |
| Non-Motorized Transport Reduction | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=CYCLING&code=PEDESTRIAN` | Bar/donut |
| Non-Motorized Transport by Gas Type | `/gas-composition?dimension=CODE&code=CYCLING&code=PEDESTRIAN` | Stacked bar |
| Capacity and Population Served by Activity | `/series?dimension=CODE&metrics=CAPACITY&metrics=POPULATION_SERVED&code=CYCLING&code=PEDESTRIAN` | Dual-axis bar |
| Regional Capacity and Population Served | `/series?dimension=REGION&metrics=CAPACITY&metrics=POPULATION_SERVED&code=CYCLING&code=PEDESTRIAN` | Regional bar |

### Forestry dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total Forestry GHG Reductions | `/summary?sectorId={FORESTRY_ID}&gasType={GAS}` | KPI |
| Forestry Reduction by Measure | `/grouped?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={FORESTRY_ID}` | Donut/bar |
| Forestry Measure Reduction Trend | `/trend?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={FORESTRY_ID}` | Area/line |
| Forestry Measures by Gas Type | `/gas-composition?dimension=SECTOR_MEASURE&sectorId={FORESTRY_ID}` | Stacked bar |
| CBFM and JFM Reductions | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=COMMUNITY_BASED_FOREST_MANAGEMENT&code=JOINT_FOREST_MANAGEMENT` | Bar |
| CBFM and JFM by Gas Type | `/gas-composition?dimension=CODE&code=COMMUNITY_BASED_FOREST_MANAGEMENT&code=JOINT_FOREST_MANAGEMENT` | Grouped bar |
| CBFM/JFM Projects by Management Type | `/heatmap?rowDimension=CODE&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=COMMUNITY_BASED_FOREST_MANAGEMENT&code=JOINT_FOREST_MANAGEMENT` | Heatmap |
| Regional Forest Projects by Management Type | `/heatmap?rowDimension=REGION&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=COMMUNITY_BASED_FOREST_MANAGEMENT&code=JOINT_FOREST_MANAGEMENT` | Heatmap |
| CBFM/JFM Beneficiaries and Coverage Area | `/series?dimension=CODE&metrics=NUMBER_OF_BENEFICIARIES&metrics=COVERAGE_AREA&code=COMMUNITY_BASED_FOREST_MANAGEMENT&code=JOINT_FOREST_MANAGEMENT` | Dual-axis bar |
| Restoration Reduction by Category | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=PLANTED_FORESTS_AND_WOODLOTS&code=NATURAL_REGENERATION&code=AGROFOREST&code=MANGROVES_RESTORATION&code=AGROSILVIPASTORAL` | Bar/donut |
| Restoration Category by Gas Type | `/gas-composition?dimension=CODE&code=PLANTED_FORESTS_AND_WOODLOTS&code=NATURAL_REGENERATION&code=AGROFOREST&code=MANGROVES_RESTORATION&code=AGROSILVIPASTORAL` | Stacked bar |
| Coverage Area and Trees by Restoration Category | `/series?dimension=CODE&metrics=COVERAGE_AREA&metrics=NUMBER_OF_TREES&code=PLANTED_FORESTS_AND_WOODLOTS&code=NATURAL_REGENERATION&code=AGROFOREST&code=MANGROVES_RESTORATION&code=AGROSILVIPASTORAL` | Dual-axis bar |
| Regional Restoration Coverage | `/heatmap?rowDimension=REGION&columnDimension=CODE&metric=COVERAGE_AREA&code=PLANTED_FORESTS_AND_WOODLOTS&code=NATURAL_REGENERATION&code=AGROFOREST&code=MANGROVES_RESTORATION&code=AGROSILVIPASTORAL` | Stacked bar |
| Restoration Beneficiaries by Region | `/heatmap?rowDimension=REGION&columnDimension=CODE&metric=NUMBER_OF_BENEFICIARIES&code=PLANTED_FORESTS_AND_WOODLOTS&code=NATURAL_REGENERATION&code=AGROFOREST&code=MANGROVES_RESTORATION&code=AGROSILVIPASTORAL` | Heatmap |

### Waste dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total Waste GHG Reductions | `/summary?sectorId={WASTE_ID}&gasType={GAS}` | KPI |
| Waste Reduction by Measure | `/grouped?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={WASTE_ID}` | Donut/bar |
| Waste Measure Reduction Trend | `/trend?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={WASTE_ID}` | Area/line |
| Waste Measures by Gas Type | `/gas-composition?dimension=SECTOR_MEASURE&sectorId={WASTE_ID}` | Stacked bar |
| Waste Management Reduction by Activity | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=RECYCLING&code=SORTING&code=LANDFILL` | Bar/donut |
| Waste Management Activity by Gas Type | `/gas-composition?dimension=CODE&code=RECYCLING&code=SORTING&code=LANDFILL` | Stacked bar |
| Recycling Facilities by Waste Type | `/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=RECYCLING` | Bar |
| Regional Recycling Facilities | `/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=RECYCLING` | Heatmap |
| Recycling Installed and Production Capacity | `/series?dimension=WASTE_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=RECYCLING` | Grouped bar |
| Regional Recycling Capacity | `/series?dimension=REGION&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=RECYCLING` | Regional bar |
| Sorting Facilities by Waste Type | `/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=SORTING` | Bar |
| Sorting Capacity by Waste Type | `/grouped?dimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=SORTING` | Bar |
| Regional Sorting Capacity | `/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=SORTING` | Heatmap |
| Landfill Capacity and Size | `/series?dimension=WASTE_TYPE&metrics=LANDFILL_CAPACITY&metrics=LANDFILL_SIZE&code=LANDFILL` | Dual-axis bar |
| Regional Landfill Capacity and Size | `/series?dimension=REGION&metrics=LANDFILL_CAPACITY&metrics=LANDFILL_SIZE&code=LANDFILL` | Regional bar |
| Waste-to-Energy Reduction by Technology | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=BIOGAS_COW_DUNG&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | Donut/bar |
| Waste-to-Energy Technology by Gas Type | `/gas-composition?dimension=CODE&code=BIOGAS_COW_DUNG&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | Stacked bar |
| Biogas Plants and Capacities | `/series?dimension=CODE&metrics=DISTINCT_ENTITIES&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY_PER_DAY&code=BIOGAS_COW_DUNG` | Composite chart |
| Regional Biogas Capacity | `/series?dimension=REGION&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY_PER_DAY&code=BIOGAS_COW_DUNG` | Regional bar |
| Alternative Charcoal Plants and Capacities | `/series?dimension=CODE&metrics=DISTINCT_ENTITIES&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY_PER_DAY&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | Composite chart |
| Charcoal Plants by Production Material | `/grouped?dimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | Bar |
| Regional Charcoal Plants by Material | `/heatmap?rowDimension=REGION&columnDimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | Heatmap |

### Industry dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total Industry GHG Reductions | `/summary?sectorId={INDUSTRY_ID}&gasType={GAS}` | KPI |
| GHG Reduction by Clean Energy Source | `/grouped?dimension=ENERGY_SOURCE&metric={GAS_METRIC}&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Bar/donut |
| Annual GHG Reduction by Clean Energy Source | `/trend?dimension=ENERGY_SOURCE&metric={GAS_METRIC}&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Line |
| Clean Energy Source by Gas Type | `/gas-composition?dimension=ENERGY_SOURCE&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Stacked bar |
| Industries and Energy Consumption by Source | `/series?dimension=ENERGY_SOURCE&metrics=DISTINCT_ENTITIES&metrics=ENERGY_CONSUMPTION&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Dual-axis bar |
| Annual Industries by Clean Energy Source | `/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Line |
| Regional Industries by Clean Energy Source | `/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Heatmap/stacked bar |
| Industries by Source and Ownership | `/heatmap?rowDimension=ENERGY_SOURCE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Heatmap |
| Annual Industries by Ownership | `/trend?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Line |
| Regional Industries by Ownership | `/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Heatmap |

### Livestock dashboard

| Chart name | API | Recommended chart |
|---|---|---|
| Total Livestock GHG Reductions | `/summary?sectorId={LIVESTOCK_ID}&gasType={GAS}` | KPI |
| Livestock Reduction by Measure | `/grouped?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={LIVESTOCK_ID}` | Donut/bar |
| Livestock Measure Reduction Trend | `/trend?dimension=SECTOR_MEASURE&metric={GAS_METRIC}&sectorId={LIVESTOCK_ID}` | Area/line |
| Livestock Measures by Gas Type | `/gas-composition?dimension=SECTOR_MEASURE&sectorId={LIVESTOCK_ID}` | Stacked bar |
| Livestock Management Reduction by Category | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=SILVICULTURE&code=AGRO_SILVOPASTURE&code=SILVOPASTURE&code=LAND_RANGE_RESTORATION` | Bar/donut |
| Livestock Management Category by Gas Type | `/gas-composition?dimension=CODE&code=SILVICULTURE&code=AGRO_SILVOPASTURE&code=SILVOPASTURE&code=LAND_RANGE_RESTORATION` | Stacked bar |
| Coverage Area by Livestock Management Category | `/grouped?dimension=CODE&metric=COVERAGE_AREA&code=SILVICULTURE&code=AGRO_SILVOPASTURE&code=SILVOPASTURE&code=LAND_RANGE_RESTORATION` | Bar |
| Regional Livestock Management Coverage | `/heatmap?rowDimension=REGION&columnDimension=CODE&metric=COVERAGE_AREA&code=SILVICULTURE&code=AGRO_SILVOPASTURE&code=SILVOPASTURE&code=LAND_RANGE_RESTORATION` | Stacked bar |
| Improved Breeds and Fodders Reduction | `/grouped?dimension=CODE&metric={GAS_METRIC}&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Bar/donut |
| Improved Farms by Gas Type | `/gas-composition?dimension=CODE&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Stacked bar |
| Coverage Area by Farm Category | `/grouped?dimension=CODE&metric=COVERAGE_AREA&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Bar |
| Regional Farm Coverage | `/heatmap?rowDimension=REGION&columnDimension=CODE&metric=COVERAGE_AREA&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Heatmap |
| Farm Counts and Production Capacity | `/series?dimension=CODE&metrics=DISTINCT_ENTITIES&metrics=LIVESTOCK_PRODUCTION_CAPACITY&metrics=FODDER_PRODUCTION_CAPACITY&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Composite chart |
| Regional Farm Counts and Capacity | `/series?dimension=REGION&metrics=DISTINCT_ENTITIES&metrics=LIVESTOCK_PRODUCTION_CAPACITY&metrics=FODDER_PRODUCTION_CAPACITY&code=FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS&code=FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | Regional composite chart |

The exact requirement-by-requirement mapping—including every annual variant, status, data source, and known limitation—is available in `MITIGATION_DASHBOARD_API_INVENTORY.md`.


## Errors

Invalid inputs use the existing exception handler and `ApiResponse` error convention. Important validation messages include `INVALID_YEAR_RANGE`, `INVALID_REGION`, `INVALID_COUNCIL`, `INVALID_LOCATION`, `INVALID_SECTOR`, `INVALID_SECTOR_MEASURE`, `INVALID_COVERAGE_LOCATION`, `INVALID_OWNERSHIP`, and `INVALID_TECHNOLOGY`.

| Status | Meaning |
|---|---|
| 400 | Invalid enum/filter, inconsistent location, bad year combination, duplicate metrics |
| 401 | Missing/invalid JWT |
| 403 | Authenticated user lacks the existing Climate Change permission |
| 500 | Unexpected database/server failure; do not interpret as zero data |

An empty, successful dataset means no eligible approved records matched:

```json
{"success":true,"message":"Grouped mitigation analytics fetched","data":{"metric":"COVERAGE_AREA","aggregation":"LATEST_SNAPSHOT_SUM","data":[],"filters":{"codes":[]},"warnings":[]}}
```


