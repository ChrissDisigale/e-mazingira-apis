# Mitigation Dashboard — Frontend API Guide

> **Reference date:** 21 November 2026. Reorganized from the supplied source-code-verified mapping. This is documentation of reported contracts, **not** a fresh live verification of a running backend.

## Start here

This guide tells you **which API to call for each dashboard graph**, **what filters to pass**, and **how to use the JSON response**. The complete dashboard contains **170 named graph/KPI requirements**, grouped into General, Energy, Transport, Forestry, Waste, Industry and Livestock.

### How to use this document

1. Find your graph by its **ID** (`G01`, `E10`, `T18`, etc.) or the [Quick reference](#quick-reference-all-170-graphs).
2. Copy the **gateway request URL** from its section (or use the service path in backend-to-backend calls).
3. Add applicable optional filters such as `coverage`, `yearFrom`, `yearTo` and `regionId`.
4. Map the response properties described under **Display this data**.
5. Check the **Status** and **Important note** before assuming that the chart is ready for production.

### Important limitation: GHG units

The supplied backend reference reports CO₂, CH₄, N₂O and HFC values with **unspecified gas units**. It does **not** establish valid conversion to **tCO₂e**. Therefore a graph entitled “tCO₂e” may be **PARTIAL** or **BLOCKED_BY_DATA**. **Do not add unlike gas values or label raw gas quantities as tCO₂e.** Annual points also use the activity-specific `values.year`, which is not necessarily a common reporting year.

## API basics

| Environment | Base path |
|---|---|
| Gateway / frontend | `/climate-change/api/v1/climate-change/mitigation/analytics` |
| Climate-change service | `/api/v1/climate-change/mitigation/analytics` |

**Common header:** `Authorization: Bearer <ACCESS_TOKEN>` when required by your deployment.

**Response envelope:** all documented successful responses use `success`, `message`, and `data`:

```json
{
  "success": true,
  "message": "Mitigation analytics fetched",
  "data": {}
}
```

**Available analytics route families** (six, reused across the 170 graphs):

| Route | Purpose |
|---|---|
| `/summary` | Selected-gas totals and summary |
| `/grouped` | One metric grouped by one category |
| `/trend` | One metric by year and category |
| `/gas-composition` | CO₂, CH₄, N₂O and HFC fields by category |
| `/series` | Several grouped metrics, preserving their units |
| `/heatmap` | One metric across two dimensions |

**Reference-data endpoints** (IDs and codes should come from these, not be hardcoded):

- `GET /climate-change/api/v1/config/sector-measures/sectors/MITIGATION_MEASURE`
- `GET /climate-change/api/v1/config/sector-measures/{sectorId}/MITIGATION_MEASURE`
- `GET /climate-change/api/v1/config/sector-measure-action/{sectorMeasureCategoryUid}/actions`
- `GET /climate-change/api/v1/climate-change/sector-measure-fields/{code}`
- `GET /climate-change/api/v1/climate-change/sector-measure-fields/mitigation/catalog`

### Frequently used optional filters

`coverage`, `regionId`, `councilId`, `year`, `yearFrom`, `yearTo`, `sectorId`, `sectorMeasureUid`, repeated `code`, `ownership`, `activityType`, `technologyType`, `wasteType`, `managementType`, `energySource`, `material`.

Some filters only affect records with corresponding JSONB/master-data fields. **`metric` and `dimension` are operation parameters, not universal filters.** The original guide's [Common filters and metric catalog](#common-filters-and-metric-definitions) gives valid values and units.

### Example request

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2&coverage=TANZANIA_MAINLAND&yearFrom=2021&yearTo=2026
Authorization: Bearer <ACCESS_TOKEN>
```

Here `coverage=TANZANIA_MAINLAND` is the geographic designation. It is not the same as numeric `coverageArea` (hectares) or transport route `coverageFrom`/`coverageTo`.

## Dashboard graph details

The request and response payloads below are shown as **readable, indented JSON**. Repeated `filters` and `warnings` fields are not reproduced in every sample to keep the graph instructions concise; they remain part of the actual API's documented envelope. Representative *example* values in the source are placeholders, not real production totals.

## General Mitigation

### G01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** General → MITIGATION → GENERAL INFORMATION

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; None.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### G02 — Quantity of Emission Reduced by Sector

**Status:** PARTIAL  ·  **Chart:** Donut Chart  ·  **Section:** General → MITIGATION → GENERAL INFORMATION

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2
```

**Select these parameters:** dimension=SECTOR; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; None.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### G03 — Trend of Quantity of Emission Reduced by Sector

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** General → MITIGATION → GENERAL INFORMATION

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2
```

**Select these parameters:** dimension=SECTOR; metric=<one gas metric>; None.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### G04 — Amount of GHG Emissions Reduced by Sector per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** General → MITIGATION → GENERAL INFORMATION

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=SECTOR
```

**Select these parameters:** dimension=SECTOR; None.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### G05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** General → MITIGATION → GENERAL INFORMATION

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; None.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

## Energy Sector

### E01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Energy → Energy Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={EnergySectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Energy sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### E02 — Quantity of Emission Reduced per Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Energy → Energy Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Energy sector>.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E03 — Trend of Quantity of Emission Reduced per Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Energy → Energy Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Energy sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E04 — Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Energy → Energy Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={EnergySectorId}
```

**Select these parameters:** dimension=INTERVENTION; sectorId=<database ID of Energy sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### E05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Energy → Energy Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Energy sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

### E06 — Quantity of Emission Reduced in Clean Technologies for Power Generation

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E07 — Trend of Quantity of Emission Reduced in Clean Technologies for Power Generation

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metric=<one gas metric>; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E08 — Amount of GHG Emissions Reduced by Clean Energy Source per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=TECHNOLOGY_TYPE&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### E09 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): year and four quantityReduced* fields.

</details>

### E10 — Installed and Production Capacity of Clean Technologies for Power Generation

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** MW

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): `INSTALLED_CAPACITY` mapped field, `PRODUCTION_CAPACITY` mapped field.

</details>

### E11 — Trend of Clean Technologies for Power Generation

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "INSTALLED_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** MW

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): year and INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### E12 — Installed and Production Capacity by Region

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** MW

**Important note:** Use `technologyType=HYDROPOWER|GEOTHERMAL|SOLAR_POWER|WIND|BIOENERGY|HYDROGEN_GAS` to select one technology.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### E13 — Ownership of Clean Technologies for Power Generation by Source

**Status:** IMPLEMENTED  ·  **Chart:** Source-by-ownership Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=TECHNOLOGY_TYPE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** rowDimension=TECHNOLOGY_TYPE; columnDimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "TECHNOLOGY_TYPE",
    "columnDimension": "OWNERSHIP",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "SOLAR_POWER",
        "label": "Solar Power",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "PRIVATE",
        "label": "Private",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "SOLAR_POWER",
        "columnCode": "PRIVATE",
        "value": 0,
        "unit": "entities"
      }
    ]
  }
}
```

**Display this data:** Clean-energy source labels: `rows`; ownership labels: `columns`; facility counts: `cells` keyed by `rowCode`/`columnCode`.

**Unit:** entities

**Important note:** One facility is counted once in each technology-source bucket represented by its stored nested technology item.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### E14 — Ownership of Clean Technologies for Power Generation by Region

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Clean Technologies for Power Generation

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### E15 — Quantity of Emission Reduced in the Use of Natural Gas

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=USE_OF_NATURAL_GAS.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E16 — Trend of Quantity of Emission Reduced in the Use of Natural Gas

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_NATURAL_GAS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E17 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_NATURAL_GAS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### E18 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_NATURAL_GAS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): year and four quantityReduced* fields.

</details>

### E19 — Installed and Production Capacity of Power Production

**Status:** PARTIAL  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** Installed: MW; production: unavailable

**Important note:** POWER_PRODUCTION captures `installedCapacity` and `naturalGasConsumed`, but no `productionCapacity`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): `INSTALLED_CAPACITY` mapped field, `PRODUCTION_CAPACITY` mapped field.

</details>

### E20 — Trend of Installed and Production Capacity of Power Production

**Status:** PARTIAL  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "INSTALLED_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** Installed: MW; production: unavailable

**Important note:** The production-capacity series has no source field.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): year and INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### E21 — Regional Breakdown: Installed and Production Capacity of Power Production

**Status:** PARTIAL  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Installed: MW; production: unavailable

**Important note:** Regional installed capacity works; production capacity is not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### E22 — Number of Households and Institutions Using Natural Gas

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NUMBER_OF_HOUSEHOLDS&metrics=NUMBER_OF_INSTITUTIONS&code=USE_OF_NATURAL_GAS&activityType=COOKING
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=NUMBER_OF_HOUSEHOLDS, metrics=NUMBER_OF_INSTITUTIONS; code=USE_OF_NATURAL_GAS; activityType=COOKING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** households; institutions

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): `NUMBER_OF_HOUSEHOLDS` mapped field, `NUMBER_OF_INSTITUTIONS` mapped field.

</details>

### E23 — Trend of Households and Institutions Using Natural Gas

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_HOUSEHOLDS&code=USE_OF_NATURAL_GAS&activityType=COOKING
```

**Select these parameters:** One request per metric: NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS; dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=COOKING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "NUMBER_OF_HOUSEHOLDS",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** households; institutions

**Important note:** The approved COOKING contract intentionally does not require/capture `year`, so annual points are omitted.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): year and NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS.

</details>

### E24 — Number of Households and Institutions Using Natural Gas by Region

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_HOUSEHOLDS&code=USE_OF_NATURAL_GAS&activityType=COOKING
```

**Select these parameters:** One request per metric: NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=COOKING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "NUMBER_OF_HOUSEHOLDS",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** households; institutions

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS.

</details>

### E25 — National Distribution of Natural Gas Stations by Ownership

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION
```

**Select these parameters:** dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### E26 — Trend of Natural Gas Stations by Ownership

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION
```

**Select these parameters:** dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### E27 — Regional Distribution of Natural Gas Stations by Ownership

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Use of Natural Gas

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_NATURAL_GAS`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### E28 — Quantity of Emission Reduced in Climate-Smart Rural Electrification

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E29 — Trend of Quantity of Emission Reduced in Climate-Smart Rural Electrification

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### E30 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### E31 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): year and four quantityReduced* fields.

</details>

### E32 — Installed and Production Capacity in Climate-Smart Rural Electrification

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** MW

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): `INSTALLED_CAPACITY` mapped field, `PRODUCTION_CAPACITY` mapped field.

</details>

### E33 — Trend of Installed and Production Capacity

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "INSTALLED_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** MW

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): year and INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### E34 — Regional Breakdown of Installed and Production Capacity in Climate-Smart Rural Electrification

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Energy → Climate-Smart Rural Electrification

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** MW

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `CLIMATE_SMART_RURAL_ELECTRIFICATION`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

## Transport Sector

### T01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Transport → Transport Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={TransportSectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Transport sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### T02 — Quantity of Emission Reduced per Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Transport → Transport Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Transport sector>.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T03 — Trend of Quantity of Emission Reduced per Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Transport → Transport Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Transport sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T04 — Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Transport → Transport Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={TransportSectorId}
```

**Select these parameters:** dimension=INTERVENTION; sectorId=<database ID of Transport sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### T05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Transport → Transport Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Transport sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

### T06 — Quantity of Emission Reduced in Low Emission Transport Systems

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=LOW_EMISSION_TRANSPORT_SYSTEMS.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T07 — Trend of Quantity of Emission Reduced in Low Emission Transport Systems

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LOW_EMISSION_TRANSPORT_SYSTEMS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T08 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LOW_EMISSION_TRANSPORT_SYSTEMS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=LOW_EMISSION_TRANSPORT_SYSTEMS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### T09 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LOW_EMISSION_TRANSPORT_SYSTEMS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): year and four quantityReduced* fields.

</details>

### T10 — Network Distance and Capacity of Rail Infrastructure by Type

**Status:** PARTIAL  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NETWORK_DISTANCE&metrics=CAPACITY&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=RAILWAY
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=NETWORK_DISTANCE, metrics=CAPACITY; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=RAILWAY.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** km; Tonnes

**Important note:** Data captures `railType` (SGR/MGR), but analytics has no RAIL_TYPE dimension/filter; this response only groups all RAILWAY records.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): `NETWORK_DISTANCE` mapped field, `CAPACITY` mapped field.

</details>

### T11 — Trend of Network Distance and Capacity of Rail Infrastructure

**Status:** PARTIAL  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=RAILWAY
```

**Select these parameters:** One request per metric: NETWORK_DISTANCE, CAPACITY; dimension=ACTIVITY_TYPE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=RAILWAY.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "NETWORK_DISTANCE",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** km; Tonnes

**Important note:** Annual rail totals work, but SGR/MGR series cannot be separated by the current analytics dimensions.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): year and NETWORK_DISTANCE, CAPACITY.

</details>

### T12 — Network Distance of Road Infrastructure by Type

**Status:** PARTIAL  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=ROAD_INFRASTRUCTURE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=NETWORK_DISTANCE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=ROAD_INFRASTRUCTURE.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "NETWORK_DISTANCE",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** km

**Important note:** Data captures `infrastructureType` (BRT/CYCLING), but analytics cannot group/filter that field.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): `NETWORK_DISTANCE` mapped JSON field.

</details>

### T13 — Trend of Network Distance of Road Infrastructure by Type

**Status:** PARTIAL  ·  **Chart:** Line Chart  ·  **Section:** Transport → Low Emission Transport Systems (Mass Rapid Transport System)

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=ROAD_INFRASTRUCTURE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=NETWORK_DISTANCE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=ROAD_INFRASTRUCTURE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "NETWORK_DISTANCE",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** km

**Important note:** Annual road totals work; BRT/CYCLING subtype trend is unavailable.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LOW_EMISSION_TRANSPORT_SYSTEMS`; JSONB field(s): year and `NETWORK_DISTANCE` mapped JSON field.

</details>

### T14 — Quantity of Emission Reduced in Use of Renewable Energy in Transportation System

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T15 — Trend of Quantity of Emission Reduced in Use of Renewable Energy in Transportation System

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T16 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### T17 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): year and four quantityReduced* fields.

</details>

### T18 — Number of BRT Facilities and Capacity by Energy Type

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ENERGY_SOURCE&metrics=DISTINCT_ENTITIES&metrics=CAPACITY&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT
```

**Select these parameters:** dimension=ENERGY_SOURCE; metrics=DISTINCT_ENTITIES, metrics=CAPACITY; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** entities; Tonnes

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): `DISTINCT_ENTITIES` mapped field, `CAPACITY` mapped field.

</details>

### T19 — Trend of Number of BRT Facilities by Energy Type

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### T20 — Number of BRT Facilities by Energy Type at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### T21 — Number of Locomotives by Energy Type

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### T22 — Trend of Number of Locomotives by Energy Type

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### T23 — Number of Locomotives by Energy Type at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### T24 — Number of Vehicles Using Electricity and Gas

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### T25 — Trend of Number of Vehicles Using Electricity and Gas

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### T26 — Number of Vehicles Using Electricity and Gas at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Transport → Use of Renewable Energy in Transportation System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### T27 — Quantity of Emission Reduced by Non-Motorized Transport System

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=NON_MOTORISED_TRANSPORT_SYSTEM.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T28 — Trend of Quantity of Emission Reduced by Non-Motorized Transport System

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=NON_MOTORISED_TRANSPORT_SYSTEM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### T29 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=NON_MOTORISED_TRANSPORT_SYSTEM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### T30 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=NON_MOTORISED_TRANSPORT_SYSTEM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): year and four quantityReduced* fields.

</details>

### T31 — Capacity and Population Served of Cycling and Pedestrian

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=CAPACITY&metrics=POPULATION_SERVED&code=NON_MOTORISED_TRANSPORT_SYSTEM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=CAPACITY, metrics=POPULATION_SERVED; code=NON_MOTORISED_TRANSPORT_SYSTEM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** Tonnes; people

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): `CAPACITY` mapped field, `POPULATION_SERVED` mapped field.

</details>

### T32 — Trend of Capacity and Population Served of Cycling

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=CAPACITY&code=NON_MOTORISED_TRANSPORT_SYSTEM&activityType=CYCLING
```

**Select these parameters:** One request per metric: CAPACITY, POPULATION_SERVED; dimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM; activityType=CYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** Tonnes; people

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): year and CAPACITY, POPULATION_SERVED.

</details>

### T33 — Regional Breakdown: Capacity and Population Served of Cycling

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Transport → Non-Motorized Transport System

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=CAPACITY&code=NON_MOTORISED_TRANSPORT_SYSTEM&activityType=CYCLING
```

**Select these parameters:** One request per metric: CAPACITY, POPULATION_SERVED; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM; activityType=CYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Tonnes; people

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `NON_MOTORISED_TRANSPORT_SYSTEM`; JSONB field(s): CAPACITY, POPULATION_SERVED.

</details>

## Forestry Sector

### F01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Forestry → Forestry Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={ForestrySectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Forestry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### F02 — Quantity of Emission Reduced by Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Forestry → Forestry Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Forestry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F03 — Trend of Quantity of Emission Reduced by Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Forestry → Forestry Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Forestry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F04 — Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Forestry → Forestry Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={ForestrySectorId}
```

**Select these parameters:** dimension=INTERVENTION; sectorId=<database ID of Forestry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### F05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Forestry → Forestry Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Forestry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

### F06 — Quantity of Emission Reduced by Participatory Forest Management Program

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F07 — Trend of Quantity of Emission Reduced by Participatory Forest Management Program

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F08 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### F09 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): year and four quantityReduced* fields.

</details>

### F10 — Number of CBFM and JFM per Ownership

**Status:** IMPLEMENTED  ·  **Chart:** Program-by-management Heatmap / Grouped Bar Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=ACTIVITY_TYPE&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** rowDimension=ACTIVITY_TYPE; columnDimension=MANAGEMENT_TYPE; metric=DISTINCT_ENTITIES; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "ACTIVITY_TYPE",
    "columnDimension": "MANAGEMENT_TYPE",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "COMMUNITY_BASED_FOREST_MANAGEMENT",
        "label": "Community Based Forest Management",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "COMMUNITY",
        "label": "Community",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "COMMUNITY_BASED_FOREST_MANAGEMENT",
        "columnCode": "COMMUNITY",
        "value": 0,
        "unit": "entities"
      }
    ]
  }
}
```

**Display this data:** CBFM/JFM categories: `rows`; management/ownership labels: `columns`; project counts: `cells`.

**Unit:** entities

**Important note:** The schema calls this `managementType`, not ownership; values are VILLAGE_GOVERNMENT and COMMUNITY.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### F11 — Trend of Number of CBFM and JFM

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=DISTINCT_ENTITIES; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### F12 — Number of CBFM per Ownership at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM&activityType=COMMUNITY_BASED_FOREST_MANAGEMENT
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=MANAGEMENT_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM; activityType=COMMUNITY_BASED_FOREST_MANAGEMENT.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Filter to CBFM; backend field is `managementType`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### F13 — Number of Beneficiaries and Coverage Area of CBFM and JFM

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NUMBER_OF_BENEFICIARIES&metrics=COVERAGE_AREA&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=NUMBER_OF_BENEFICIARIES, metrics=COVERAGE_AREA; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** people; Ha

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): `NUMBER_OF_BENEFICIARIES` mapped field, `COVERAGE_AREA` mapped field.

</details>

### F14 — Trend of Number of Beneficiaries of CBFM and JFM

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** One request per metric: NUMBER_OF_BENEFICIARIES; dimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "NUMBER_OF_BENEFICIARIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** people

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): year and NUMBER_OF_BENEFICIARIES.

</details>

### F15 — Regional Breakdown: Number of Beneficiaries of CBFM and JFM

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Forestry → Participatory Forest Management Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM
```

**Select these parameters:** One request per metric: NUMBER_OF_BENEFICIARIES; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "NUMBER_OF_BENEFICIARIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** people

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`; JSONB field(s): NUMBER_OF_BENEFICIARIES.

</details>

### F16 — Quantity of Emission Reduced by Landscape Restoration Program

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F17 — Trend of Quantity of Emission Reduced by Landscape Restoration Program

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### F18 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### F19 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): year and four quantityReduced* fields.

</details>

### F20 — Coverage Area and Number of Trees Planted by Landscape Restoration Category

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=COVERAGE_AREA&metrics=NUMBER_OF_TREES&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=COVERAGE_AREA, metrics=NUMBER_OF_TREES; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** Ha; trees

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): `COVERAGE_AREA` mapped field, `NUMBER_OF_TREES` mapped field.

</details>

### F21 — Trend of Coverage Area by Landscape Restoration Category

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** One request per metric: COVERAGE_AREA; dimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** Ha

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): year and COVERAGE_AREA.

</details>

### F22 — Regional Breakdown: Coverage Area by Landscape Restoration Category

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "COVERAGE_AREA",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Ha

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): COVERAGE_AREA.

</details>

### F23 — Number of Beneficiaries by Landscape Restoration Category

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=NUMBER_OF_BENEFICIARIES; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "NUMBER_OF_BENEFICIARIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** people

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): `NUMBER_OF_BENEFICIARIES` mapped JSON field.

</details>

### F24 — Trend of Number of Beneficiaries by Landscape Restoration Category

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=NUMBER_OF_BENEFICIARIES; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "NUMBER_OF_BENEFICIARIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** people

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): year and `NUMBER_OF_BENEFICIARIES` mapped JSON field.

</details>

### F25 — Number of Beneficiaries by Landscape Restoration Category at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Forestry → Landscape Restoration Program

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM
```

**Select these parameters:** One request per metric: NUMBER_OF_BENEFICIARIES; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "NUMBER_OF_BENEFICIARIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** people

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LANDSCAPE_RESTORATION_PROGRAM`; JSONB field(s): NUMBER_OF_BENEFICIARIES.

</details>

## Waste Sector

### W01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Waste → Waste Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={WasteSectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Waste sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### W02 — Quantity of Emission Reduced by Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Waste → Waste Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Waste sector>.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W03 — Trend of Quantity of Emission Reduced by Mitigation Measures

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Waste → Waste Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Waste sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W04 — Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Waste → Waste Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={WasteSectorId}
```

**Select these parameters:** dimension=INTERVENTION; sectorId=<database ID of Waste sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### W05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Waste → Waste Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Waste sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

### W06 — Quantity of Emission Reduced in Environmentally Sound Waste Management Practice

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W07 — Trend of Quantity of Emission Reduced in Environmentally Sound Waste Management Practice

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W08 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### W09 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and four quantityReduced* fields.

</details>

### W10 — Number of Recycling Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W11 — Trend of Number of Recycling Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W12 — Regional Breakdown: Number of Recycling Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### W13 — Installed Capacity and Production Capacity of Recycling Facilities

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=WASTE_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** dimension=WASTE_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** Tonnes

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `INSTALLED_CAPACITY` mapped field, `PRODUCTION_CAPACITY` mapped field.

</details>

### W14 — Trend of Installed Capacity and Production Capacity of Recycling Facilities

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=INSTALLED_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "INSTALLED_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** Tonnes

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### W15 — Regional Breakdown: Installed Capacity and Production Capacity of Recycling Facilities

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=INSTALLED_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Tonnes

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY.

</details>

### W16 — Number of Sorting Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W17 — Trend of Number of Sorting Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W18 — Regional Breakdown: Number of Sorting Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### W19 — Sorting Capacity of Sorting Facilities

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=SORTING_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "SORTING_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** Tonnes

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `SORTING_CAPACITY` mapped JSON field.

</details>

### W20 — Trend of Sorting Capacity of Sorting Facilities

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** dimension=WASTE_TYPE; metric=SORTING_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "SORTING_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** Tonnes

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and `SORTING_CAPACITY` mapped JSON field.

</details>

### W21 — Sorting Capacity of Sorting Facilities at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING
```

**Select these parameters:** One request per metric: SORTING_CAPACITY; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "SORTING_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Tonnes

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): SORTING_CAPACITY.

</details>

### W22 — Number of Waste Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W23 — Trend of Number of Waste Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W24 — Regional Breakdown: Number of Waste Facilities per Type of Waste

**Status:** PARTIAL  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Current schema only accepts ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE; requested Plastic, Metal, Rubber, Glass, Paper, and generic Organic categories are not captured.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### W25 — Capacity and Size of Landfill

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=WASTE_TYPE&metrics=ORGANIC_WASTE_RECEIVING_CAPACITY&metrics=LANDFILL_SIZE&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** dimension=WASTE_TYPE; metrics=ORGANIC_WASTE_RECEIVING_CAPACITY, metrics=LANDFILL_SIZE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** Tonnes; Ha

**Important note:** Capacity means the captured capacity to receive organic waste.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): `ORGANIC_WASTE_RECEIVING_CAPACITY` mapped field, `LANDFILL_SIZE` mapped field.

</details>

### W26 — Trend of Capacity and Size of Landfill

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=ORGANIC_WASTE_RECEIVING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** One request per metric: ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE; dimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "ORGANIC_WASTE_RECEIVING_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** Tonnes; Ha

**Important note:** Capacity means the captured capacity to receive organic waste.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): year and ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE.

</details>

### W27 — Regional Breakdown: Capacity and Size of Landfill

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Environmentally Sound Waste Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=ORGANIC_WASTE_RECEIVING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL
```

**Select these parameters:** One request per metric: ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "ORGANIC_WASTE_RECEIVING_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Tonnes; Ha

**Important note:** Capacity means the captured capacity to receive organic waste.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`; JSONB field(s): ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE.

</details>

### W28 — Quantity of Emission Reduced in Waste to Energy Technologies

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=WASTE_TO_ENERGY_TECHNOLOGIES.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W29 — Trend of Quantity of Emission Reduced in Waste to Energy Technologies

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metric=<one gas metric>; code=WASTE_TO_ENERGY_TECHNOLOGIES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### W30 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=TECHNOLOGY_TYPE&code=WASTE_TO_ENERGY_TECHNOLOGIES
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; code=WASTE_TO_ENERGY_TECHNOLOGIES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### W31 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=WASTE_TO_ENERGY_TECHNOLOGIES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): year and four quantityReduced* fields.

</details>

### W32 — Number of Biogas Plants, Production and Installed Capacity

**Status:** IMPLEMENTED  ·  **Chart:** Three-series Composite Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=DISTINCT_ENTITIES&metrics=PRODUCTION_CAPACITY_PER_DAY&metrics=INSTALLED_CAPACITY&code=BIOGAS_COW_DUNG
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metrics=DISTINCT_ENTITIES, metrics=PRODUCTION_CAPACITY_PER_DAY, metrics=INSTALLED_CAPACITY; code=BIOGAS_COW_DUNG.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** entities; m³/day; m³

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): `DISTINCT_ENTITIES` mapped field, `PRODUCTION_CAPACITY_PER_DAY` mapped field, `INSTALLED_CAPACITY` mapped field.

</details>

### W33 — Trend of Number of Biogas Plants, Production and Installed Capacity

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=DISTINCT_ENTITIES&code=BIOGAS_COW_DUNG
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES, PRODUCTION_CAPACITY_PER_DAY, INSTALLED_CAPACITY; dimension=TECHNOLOGY_TYPE; code=BIOGAS_COW_DUNG.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** entities; m³/day; m³

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): year and DISTINCT_ENTITIES, PRODUCTION_CAPACITY_PER_DAY, INSTALLED_CAPACITY.

</details>

### W34 — Installed and Production Capacity of Biogas Plants at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=BIOGAS_COW_DUNG
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=BIOGAS_COW_DUNG.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** m³; m³/day

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY.

</details>

### W35 — Number of Alternative Charcoal Production Facilities, Installed and Production Capacity

**Status:** IMPLEMENTED  ·  **Chart:** Three-series Composite Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=DISTINCT_ENTITIES&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY_PER_DAY&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** dimension=TECHNOLOGY_TYPE; metrics=DISTINCT_ENTITIES, metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY_PER_DAY; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** entities; Tonnes; Tonnes/day

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): `DISTINCT_ENTITIES` mapped field, `INSTALLED_CAPACITY` mapped field, `PRODUCTION_CAPACITY_PER_DAY` mapped field.

</details>

### W36 — Trend of Number of Alternative Charcoal Production Facilities, Installed and Production Capacity

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES, INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; dimension=TECHNOLOGY_TYPE; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** entities; Tonnes; Tonnes/day

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): year and DISTINCT_ENTITIES, INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY.

</details>

### W37 — Installed and Production Capacity of Alternative Charcoal Production Facilities at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "INSTALLED_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Tonnes; Tonnes/day

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY.

</details>

### W38 — Number of Plants per Production Material in Alternative Charcoal Production

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** dimension=MATERIAL; metric=DISTINCT_ENTITIES; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** entities

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W39 — Trend of Number of Plants per Production Material in Alternative Charcoal Production

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** dimension=MATERIAL; metric=DISTINCT_ENTITIES; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### W40 — Number of Plants per Production Material in Alternative Charcoal Production at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Waste → Waste to Energy Technologies

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=MATERIAL; code=ALTERNATIVE_CHARCOAL_PRODUCTION.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `WASTE_TO_ENERGY_TECHNOLOGIES`; JSONB field(s): DISTINCT_ENTITIES.

</details>

## Industry Sector

### I01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={IndustrySectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Industry sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### I02 — Quantity of Emission Reduced in Diversification of Clean Energy Sources

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### I03 — Trend of Quantity of Emission Reduced in Diversification of Clean Energy Sources

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=<one gas metric>; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** The relevant schema does not capture a usable year, so the trend endpoint omits these observations.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### I04 — Amount of GHG Emissions Reduced by Clean Energy Source per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ENERGY_SOURCE&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=ENERGY_SOURCE; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### I05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** No usable year is captured for this submission, so an annual gas trend cannot be produced.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): year and four quantityReduced* fields.

</details>

### I06 — Number of Industries and Energy Consumption per Clean Energy Source

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ENERGY_SOURCE&metrics=DISTINCT_ENTITIES&metrics=ENERGY_CONSUMPTION&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metrics=DISTINCT_ENTITIES, metrics=ENERGY_CONSUMPTION; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** entities; source-declared MW or MMscfd

**Important note:** An industry with multiple sources is counted once in each source bucket. Do not sum source-bucket counts to obtain a national distinct-industry total.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): `DISTINCT_ENTITIES` mapped field, `ENERGY_CONSUMPTION` mapped field.

</details>

### I07 — Trend of Number of Industries per Clean Energy Source

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Line Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** The industry schema has no year field, so trend rows are omitted.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### I08 — Number of Industries per Clean Energy Source at Regional Level

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** The industry schema does not capture Coverage, Region, or Council.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): DISTINCT_ENTITIES.

</details>

### I09 — Number of Industries per Clean Energy Source per Ownership

**Status:** IMPLEMENTED  ·  **Chart:** Heatmap  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=ENERGY_SOURCE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** rowDimension=ENERGY_SOURCE; columnDimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Rows: clean source; columns: ownership; cells: distinct industry/source identities.

**Unit:** entities

**Important note:** Do not sum across energy-source rows for a national industry total.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): energySources[].source, ownership, industryName.

</details>

### I10 — Trend of Number of Industries per Clean Energy Source per Ownership

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Line Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "DISTINCT_ENTITIES",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** entities

**Important note:** No year field is captured. If a source is selected, `energySource=<value>` can filter the request.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): year and `DISTINCT_ENTITIES` mapped JSON field.

</details>

### I11 — Number of Industries per Clean Energy Source per Ownership at Regional Level

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Industry → Diversification of Clean Energy Sources

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES
```

**Select these parameters:** One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "DISTINCT_ENTITIES",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** entities

**Important note:** No Region/Council/Coverage is captured. A three-dimensional view would otherwise call once per `energySource`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`; JSONB field(s): DISTINCT_ENTITIES.

</details>

## Livestock Sector

### L01 — Amount of GHGs Reduced (tCO₂e)

**Status:** BLOCKED_BY_DATA  ·  **Chart:** Summary Card  ·  **Section:** Livestock → Livestock Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={LivestockSectorId}
```

**Select these parameters:** gasType=CO2|CH4|N2O|HFCS; sectorId=<database ID of Livestock sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation summary fetched",
  "data": {
    "total": null,
    "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY",
    "gasTotals": {
      "CO2": 0,
      "CH4": 0,
      "N2O": 0,
      "HFCS": 0
    },
    "yearSource": "values.year (activity-specific year)"
  }
}
```

**Display this data:** Card value: `data.total` for one selected gas; `data.gasTotals` exposes all four raw reported quantities.

**Unit:** Backend returns `UNSPECIFIED_REPORTED_GAS_QUANTITY`, not tCO₂e.

**Important note:** The title requires tCO₂e, but storage does not define gas mass versus CO₂e or GWP/version. Without `gasType`, `data.total` is null; never sum `gasTotals`.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### L02 — Quantity of Emission Reduced by Livestock Management Practice and Use of Improved Animal Breeds and Feeds

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Livestock → Livestock Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Livestock sector>.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L03 — Trend of Quantity of Emission Reduced by Livestock Management Practice and Use of Improved Animal Breeds and Feeds

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Livestock → Livestock Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}
```

**Select these parameters:** dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Livestock sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L04 — Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Livestock → Livestock Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={LivestockSectorId}
```

**Select these parameters:** dimension=INTERVENTION; sectorId=<database ID of Livestock sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### L05 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Livestock → Livestock Overview

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Livestock sector>.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; JSONB field(s): year and four quantityReduced* fields.

</details>

### L06 — Quantity of Emission Reduced in Livestock Management Practice

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L07 — Trend of Quantity of Emission Reduced in Livestock Management Practice

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L08 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### L09 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): year and four quantityReduced* fields.

</details>

### L10 — Coverage Area under Livestock Management Practice per Category

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** Ha

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): `COVERAGE_AREA` mapped JSON field.

</details>

### L11 — Trend of Coverage Area under Livestock Management Practice per Category

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** Ha

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): year and `COVERAGE_AREA` mapped JSON field.

</details>

### L12 — Coverage Area under Livestock Management Practice per Category at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Livestock → Livestock Management Practice

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE
```

**Select these parameters:** One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LIVESTOCK_MANAGEMENT_PRACTICE.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "COVERAGE_AREA",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Ha

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `LIVESTOCK_MANAGEMENT_PRACTICE`; JSONB field(s): COVERAGE_AREA.

</details>

### L13 — Quantity of Emission Reduced in the Use of Improved Animal Breeds and Feeds

**Status:** PARTIAL  ·  **Chart:** Bar / Donut Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2|QUANTITY_REDUCED_CH4|QUANTITY_REDUCED_N2O|QUANTITY_REDUCED_HFCS>; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** Category: `data.data[].label`; value: `value`; percentage: `percentage`; select gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Grouping exists, but values are raw reported gas quantities and cannot be labelled tCO₂e.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L14 — Trend of Quantity of Emission Reduced in the Use of Improved Animal Breeds and Feeds

**Status:** PARTIAL  ·  **Chart:** Line / Spline Area Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `year`; series: `label`; Y: `value`; choose gas by changing `metric`.

**Unit:** Unspecified reported quantity for the selected gas.

**Important note:** Uses activity-specific `values.year`, not a common reporting year; gas unit/CO₂e basis is undefined.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): year and quantityReducedCO2/CH4/N2O/HFCs.

</details>

### L15 — Amount of GHG Emissions Reduced by Category per Type of Gas

**Status:** PARTIAL  ·  **Chart:** Stacked Bar Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ]
  }
}
```

**Display this data:** Category: `categories[].label`; stacks: `co2`, `ch4`, `n2o`, `hfcs`.

**Unit:** Each series is an unspecified reported gas quantity; gases are not converted or combined.

**Important note:** The four series are implemented, but cross-gas comparison as tCO₂e is not valid without units/GWP.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): quantityReducedCO2, quantityReducedCH4, quantityReducedN2O, quantityReducedHFCs.

</details>

### L16 — Trend of GHG Emissions Reduced by Type of Gas

**Status:** PARTIAL  ·  **Chart:** Multi-series Line Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "QUANTITY_REDUCED_CO2",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge the four responses by `year`; gas name comes from response `metric`; value is `data[].value`.

**Unit:** Per-gas unspecified reported quantity.

**Important note:** Requires four requests; years are activity-specific and gases have no declared mass/CO₂e basis.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): year and four quantityReduced* fields.

</details>

### L17 — Coverage Area per Farms with Improved Livestock Breeds and Fodders

**Status:** IMPLEMENTED  ·  **Chart:** Grouped Bar Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ]
  }
}
```

**Display this data:** X/category: `data.data[].label`; Y/value: `value`; display `unit`; `percentage` is backend-computed within each unit.

**Unit:** Ha

**Important note:** No chart-shape transformation beyond category/series mapping is required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): `COVERAGE_AREA` mapped JSON field.

</details>

### L18 — Trend of Coverage Area per Farms with Improved Livestock Breeds and Fodders

**Status:** IMPLEMENTED  ·  **Chart:** Line Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "COVERAGE_AREA",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** X: `data.data[].year`; series: `label`; Y: `value`; display `unit`.

**Unit:** Ha

**Important note:** Missing years are omitted; frontend may insert visual zeroes only if product requirements explicitly demand it.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): year and `COVERAGE_AREA` mapped JSON field.

</details>

### L19 — Coverage Area per Farms with Improved Livestock Breeds and Fodders at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "COVERAGE_AREA",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** Ha

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): COVERAGE_AREA.

</details>

### L20 — Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes)

**Status:** IMPLEMENTED  ·  **Chart:** Dual-axis Grouped Bar Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=LIVESTOCK_PRODUCTION_CAPACITY&metrics=FODDER_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** dimension=ACTIVITY_TYPE; metrics=LIVESTOCK_PRODUCTION_CAPACITY, metrics=FODDER_PRODUCTION_CAPACITY; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ]
  }
}
```

**Display this data:** Categories: each series `data[].label`; series key/legend: `series[].key` or `label`; values/units: nested `value` and `unit`.

**Unit:** animals; Tonnes

**Important note:** Keep series on separate axes when units differ.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): `LIVESTOCK_PRODUCTION_CAPACITY` mapped field, `FODDER_PRODUCTION_CAPACITY` mapped field.

</details>

### L21 — Trend of Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes)

**Status:** IMPLEMENTED  ·  **Chart:** Multi-series / Dual-axis Line Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=LIVESTOCK_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** One request per metric: LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation trend fetched",
  "data": {
    "metric": "LIVESTOCK_PRODUCTION_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "missingData": "Missing observations are omitted; the backend does not invent zeroes or interpolate years.",
    "data": [
      {
        "year": 2025,
        "code": "SERIES",
        "label": "Series",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Merge responses by `year` and category `code`; use each response `metric` as series key and each point's `unit`.

**Unit:** animals; Tonnes

**Important note:** The backend has no multi-metric trend route; parallel calls to the same verified `/trend` endpoint are required.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): year and LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY.

</details>

### L22 — Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes) at Regional Level

**Status:** IMPLEMENTED  ·  **Chart:** Regional Heatmap / Grouped Bar Chart  ·  **Section:** Livestock → Use of Improved Animal Breeds and Feeds

**Request**

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=LIVESTOCK_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS
```

**Select these parameters:** One request per metric: LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS.

**Response example**

```json
{
  "success": true,
  "message": "Mitigation heatmap fetched",
  "data": {
    "rowDimension": "REGION",
    "columnDimension": "CATEGORY",
    "metric": "LIVESTOCK_PRODUCTION_CAPACITY",
    "rows": [
      {
        "code": "1",
        "label": "Region",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "columns": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": null,
        "unit": null,
        "percentage": null
      }
    ],
    "cells": [
      {
        "rowCode": "1",
        "columnCode": "CATEGORY",
        "value": 0,
        "unit": "UNIT"
      }
    ]
  }
}
```

**Display this data:** Region labels: `rows`; category labels: `columns`; values: `cells` keyed by `rowCode`/`columnCode`; merge calls by keys.

**Unit:** animals; Tonnes

**Important note:** Render one series per metric and preserve each returned unit.

<details>
<summary>Where the data comes from</summary>

Approved, active, non-deleted `measure_action_field_data_request` rows joined to Mitigation Measure sector/action master data; intervention/action `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`; JSONB field(s): LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY.

</details>

## Common filters and metric definitions

### Verified parameters

| Parameter | Type / accepted values | Actual behavior |
|---|---|---|
| `coverage` | `TANZANIA_MAINLAND`, `ZANZIBAR` | Matches nested `coverage` first, then parent JSONB. Historical rows without coverage appear only when this filter is omitted. |
| `regionId` | Long | Validated against `region`; nested child Region/Council location takes precedence. |
| `councilId` | Long | Validated against `council` and, when supplied, `regionId`. |
| `year` | Integer 1900–2100 | Exact `values.year`; mutually exclusive with `yearFrom/yearTo`. |
| `yearFrom`, `yearTo` | Integer 1900–2100 | Inclusive activity-specific year range. |
| `sectorId` | Long | Database Sector ID; validated before querying. IDs are environment data, not hard-coded constants. |
| `sectorMeasureUid` | String UUID | Sector Measure UUID, validated against `sector_measure_category.uuid`. |
| `code` | Repeatable `SectorMeasureActionCodes` enum | Matches parent intervention or effective nested child category; repeat query parameter for multiple values. |
| `ownership` | GOVERNMENT, PRIVATE, INDIVIDUAL, COMMUNITY, VILLAGE_GOVERNMENT, PUBLIC | Case-insensitive exact match. |
| `activityType` | Verified nested activity enum | Case-insensitive exact match; service validates against its activity whitelist. |
| `technologyType` | HYDROPOWER, GEOTHERMAL, SOLAR_POWER, WIND, WIND_POWER, BIOENERGY, HYDROGEN_GAS, NATURAL_GAS, TIDAL_WAVES, NUCLEAR, SGR, MGR, BRT, CYCLING, PEDESTRIAN, OTHER | Case-insensitive exact match; note that rail `railType` and road `infrastructureType` are not wired to this filter. |
| `wasteType` | String | Case-insensitive exact match; not service-whitelisted. Stored nested schema currently offers only `ORGANIC_WASTE` and `LIVESTOCK_MANURE_WASTE`. |
| `managementType` | String | Case-insensitive exact match. |
| `energySource` | String | Matches nested `source` or `energySource`. |
| `material` | String | Matches `productionMaterial` / legacy charcoal material field. |
| `gasType` | CO2, CH4, N2O, HFCS | Only selects `summary.data.total`; grouped/trend choose gas through the `metric` parameter. It does not filter `gas-composition`. |
| `dimension` | SECTOR, SECTOR_MEASURE, INTERVENTION, ACTIVITY_TYPE, CODE, REGION, OWNERSHIP, TECHNOLOGY_TYPE, ENERGY_SOURCE, WASTE_TYPE, MANAGEMENT_TYPE, MATERIAL | Required by grouped/trend/series. `CODE` is an alias for `ACTIVITY_TYPE`. |
| `metric` / `metrics` | Values in the verified metric catalog below | `metric` is singular; `metrics` is repeatable for `series`. |

Current exact values used by the mitigation dashboard contracts:

- `activityType`: `POWER_PRODUCTION`, `COOKING`, `TRANSPORTATION`, `MICRO_RENEWABLE_GENERATION`, `MINI_GRID_RENEWABLE_GENERATION`, `RAILWAY`, `ROAD_INFRASTRUCTURE`, `BUS_RAPID_TRANSIT`, `LOCOMOTIVES`, `PRIVATE_VEHICLES`, `CYCLING`, `PEDESTRIAN`, `COMMUNITY_BASED_FOREST_MANAGEMENT`, `JOINT_FOREST_MANAGEMENT`, `PLANTED_FORESTS_AND_WOODLOTS`, `AGROFOREST`, `NATURAL_REGENERATION`, `MANGROVES_RESTORATION`, `AGROSILVIPASTORAL`, `RECYCLING`, `SORTING`, `LANDFILL`, `SILVICULTURE`, `AGRO_SILVOPASTURE`, `SILVOPASTURE`, `LAND_RANGE_RESTORATION`, `FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS`, `FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS`.
- `energySource`: current transport schema uses `ELECTRICITY`, `GAS`; current industry source schema uses `HYDROPOWER`, `SOLAR_POWER`, `WIND`, `BIOENERGY`, `GEOTHERMAL`, `HYDROGEN_GAS`, `NATURAL_GAS`, `TIDAL_WAVES`, `NUCLEAR`.
- `managementType`: `VILLAGE_GOVERNMENT`, `COMMUNITY`.
- `material`: `ORGANIC_WASTE`, `RICE_HUSK` for alternative charcoal production.
- `wasteType`: `ORGANIC_WASTE`, `LIVESTOCK_MANURE_WASTE` in the current nested waste schema.
- Relevant parent `code` values: `IMPROVE_CLEAN_POWER_INTERCONNECTION_WITH_NEIGHBOURING_COUNTRIES`, `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION`, `USE_OF_NATURAL_GAS`, `CLIMATE_SMART_RURAL_ELECTRIFICATION`, `LOW_EMISSION_TRANSPORT_SYSTEMS`, `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION`, `NON_MOTORISED_TRANSPORT_SYSTEM`, `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM`, `LANDSCAPE_RESTORATION_PROGRAM`, `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE`, `WASTE_TO_ENERGY_TECHNOLOGIES`, `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`, `LIVESTOCK_MANAGEMENT_PRACTICE`, `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS`. Child action codes shown in the graph blocks are also accepted because the repository matches both parent intervention and effective nested category.

There is no `category` query parameter on the analytics controller; use repeatable `code`. There is no generic `metric` filter field inside `MitigationAnalyticsFilter`; metric selection is an endpoint-level parameter.

### Coverage versus area and route coverage

- **Coverage** is the administrative designation `TANZANIA_MAINLAND` or `ZANZIBAR`.
- **Coverage Area** is numeric `coverageArea`, aggregated by metric `COVERAGE_AREA`, unit Ha.
- **Coverage From / Coverage To** are route strings `coverageFrom` and `coverageTo`. They are stored for transport records but are not analytics dimensions or filters.

### Verified metric catalog and units

| Metric | JSONB field / aggregation | Returned unit |
|---|---|---|
| `QUANTITY_REDUCED_CO2/CH4/N2O/HFCS` | matching `quantityReduced*`; SUM | `UNSPECIFIED_REPORTED_GAS_QUANTITY` |
| `INSTALLED_CAPACITY` | `installedCapacity`; latest-snapshot sum | MW, m³, or Tonnes according to intervention |
| `PRODUCTION_CAPACITY` | `productionCapacity`; latest-snapshot sum | MW or Tonnes |
| `PRODUCTION_CAPACITY_PER_DAY` | `productionCapacityPerDay`; latest-snapshot sum | m³/day or Tonnes/day |
| `CAPACITY` | `capacity`; latest-snapshot sum | Tonnes |
| `NATURAL_GAS_CONSUMED` | `naturalGasConsumed`; SUM | MMscfd |
| `ENERGY_CONSUMPTION` | `consumption`; SUM | source-declared `unit` (MW or MMscfd in current industry schema) |
| `ENERGY_CONSUMPTION_PER_DAY` | `energyConsumptionPerDay`; SUM | `UNSPECIFIED_PER_DAY` |
| `NETWORK_DISTANCE` | `networkDistance`; latest-snapshot sum | km |
| `COVERAGE_AREA`, `LANDFILL_SIZE` | matching fields; latest-snapshot sum | Ha |
| `NUMBER_OF_TREES` | `numberOfTrees`; latest-snapshot sum | trees |
| `NUMBER_OF_BENEFICIARIES`, `POPULATION_SERVED` | matching fields; latest-snapshot sum | people |
| `NUMBER_OF_HOUSEHOLDS`, `NUMBER_OF_INSTITUTIONS` | matching fields; latest-snapshot sum | households / institutions |
| `VEHICLES_SERVED` | `vehiclesServed`; latest-snapshot sum | vehicles |
| `SORTING_CAPACITY`, `ORGANIC_WASTE_RECEIVING_CAPACITY` | matching fields; latest-snapshot sum | Tonnes |
| `LIVESTOCK_PRODUCTION_CAPACITY` | matching field; latest-snapshot sum | animals |
| `FODDER_PRODUCTION_CAPACITY` | matching field; latest-snapshot sum | Tonnes |
| `DISTINCT_ENTITIES` | derived stable identity; COUNT DISTINCT | entities |

Snapshot metrics take the latest record per derived entity identity in the selected period. SUM metrics sum observations. Counts use available names/IDs/vehicle numbers; a database row count is not automatically treated as a facility count.

### GHG and year rules

The database stores four separate reported numbers. It does not store whether they are gas mass or already CO₂-equivalent, a unit, a GWP source/version, or conversion factors. Therefore no current endpoint can truthfully produce total tCO₂e. The API intentionally returns the warning and never adds different gases.

Trend year is nested/parent `values.year`. Depending on action it means establishment, operation, construction, restoration, connection, or operationalization year. It is not a universal reporting year and no historical points are invented.

### Chart response requirements

- Donut/bar: `CategoryResponse.data[].label/value/unit/percentage`; total is the same-unit sum of values.
- Grouped/dual-axis: `MultiMetricResponse.series[].data[]`; preserve each point's unit.
- Stacked gas: `GasCompositionResponse.categories[].co2/ch4/n2o/hfcs`; do not stack as tCO₂e.
- Line/area: `TrendResponse.data[].year/code/label/value/unit`; missing years are omitted.
- Heatmap: `HeatmapResponse.rows/columns/cells`; cells use row and column codes.
- Funnel/waffle: no dedicated response type; frontend may transform `CategoryResponse` without changing the API.

## Quick reference: all 170 graphs

| ID | Sector | Subsection | Graph Name | Chart Type | HTTP Method | Actual API Endpoint | Filters | Status |
|---|---|---|---|---|---|---|---|---|
| G01 | General | MITIGATION → GENERAL INFORMATION | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2` | gasType=CO2/CH4/N2O/HFCS; None | **BLOCKED_BY_DATA** |
| G02 | General | MITIGATION → GENERAL INFORMATION | Quantity of Emission Reduced by Sector | Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2` | dimension=SECTOR; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; None | **PARTIAL** |
| G03 | General | MITIGATION → GENERAL INFORMATION | Trend of Quantity of Emission Reduced by Sector | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2` | dimension=SECTOR; metric=<one gas metric>; None | **PARTIAL** |
| G04 | General | MITIGATION → GENERAL INFORMATION | Amount of GHG Emissions Reduced by Sector per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=SECTOR` | dimension=SECTOR; None | **PARTIAL** |
| G05 | General | MITIGATION → GENERAL INFORMATION | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=SECTOR&metric=QUANTITY_REDUCED_CO2` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; None | **PARTIAL** |
| E01 | Energy | Energy Overview | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={EnergySectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Energy sector> | **BLOCKED_BY_DATA** |
| E02 | Energy | Energy Overview | Quantity of Emission Reduced per Mitigation Measures | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}` | dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Energy sector> | **PARTIAL** |
| E03 | Energy | Energy Overview | Trend of Quantity of Emission Reduced per Mitigation Measures | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}` | dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Energy sector> | **PARTIAL** |
| E04 | Energy | Energy Overview | Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={EnergySectorId}` | dimension=INTERVENTION; sectorId=<database ID of Energy sector> | **PARTIAL** |
| E05 | Energy | Energy Overview | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={EnergySectorId}` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Energy sector> | **PARTIAL** |
| E06 | Energy | Clean Technologies for Power Generation | Quantity of Emission Reduced in Clean Technologies for Power Generation | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | dimension=TECHNOLOGY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **PARTIAL** |
| E07 | Energy | Clean Technologies for Power Generation | Trend of Quantity of Emission Reduced in Clean Technologies for Power Generation | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | dimension=TECHNOLOGY_TYPE; metric=<one gas metric>; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **PARTIAL** |
| E08 | Energy | Clean Technologies for Power Generation | Amount of GHG Emissions Reduced by Clean Energy Source per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=TECHNOLOGY_TYPE&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | dimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **PARTIAL** |
| E09 | Energy | Clean Technologies for Power Generation | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **PARTIAL** |
| E10 | Energy | Clean Technologies for Power Generation | Installed and Production Capacity of Clean Technologies for Power Generation | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | dimension=TECHNOLOGY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **IMPLEMENTED** |
| E11 | Energy | Clean Technologies for Power Generation | Trend of Clean Technologies for Power Generation | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **IMPLEMENTED** |
| E12 | Energy | Clean Technologies for Power Generation | Installed and Production Capacity by Region | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **IMPLEMENTED** |
| E13 | Energy | Clean Technologies for Power Generation | Ownership of Clean Technologies for Power Generation by Source | Source-by-ownership Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=TECHNOLOGY_TYPE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | rowDimension=TECHNOLOGY_TYPE; columnDimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **IMPLEMENTED** |
| E14 | Energy | Clean Technologies for Power Generation | Ownership of Clean Technologies for Power Generation by Region | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION | **IMPLEMENTED** |
| E15 | Energy | Use of Natural Gas | Quantity of Emission Reduced in the Use of Natural Gas | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=USE_OF_NATURAL_GAS | **PARTIAL** |
| E16 | Energy | Use of Natural Gas | Trend of Quantity of Emission Reduced in the Use of Natural Gas | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_NATURAL_GAS | **PARTIAL** |
| E17 | Energy | Use of Natural Gas | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_NATURAL_GAS` | dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS | **PARTIAL** |
| E18 | Energy | Use of Natural Gas | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_NATURAL_GAS` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_NATURAL_GAS | **PARTIAL** |
| E19 | Energy | Use of Natural Gas | Installed and Production Capacity of Power Production | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION` | dimension=ACTIVITY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION | **PARTIAL** |
| E20 | Energy | Use of Natural Gas | Trend of Installed and Production Capacity of Power Production | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION | **PARTIAL** |
| E21 | Energy | Use of Natural Gas | Regional Breakdown: Installed and Production Capacity of Power Production | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=USE_OF_NATURAL_GAS&activityType=POWER_PRODUCTION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=POWER_PRODUCTION | **PARTIAL** |
| E22 | Energy | Use of Natural Gas | Number of Households and Institutions Using Natural Gas | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NUMBER_OF_HOUSEHOLDS&metrics=NUMBER_OF_INSTITUTIONS&code=USE_OF_NATURAL_GAS&activityType=COOKING` | dimension=ACTIVITY_TYPE; metrics=NUMBER_OF_HOUSEHOLDS, metrics=NUMBER_OF_INSTITUTIONS; code=USE_OF_NATURAL_GAS; activityType=COOKING | **IMPLEMENTED** |
| E23 | Energy | Use of Natural Gas | Trend of Households and Institutions Using Natural Gas | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_HOUSEHOLDS&code=USE_OF_NATURAL_GAS&activityType=COOKING` | One request per metric: NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS; dimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=COOKING | **BLOCKED_BY_DATA** |
| E24 | Energy | Use of Natural Gas | Number of Households and Institutions Using Natural Gas by Region | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_HOUSEHOLDS&code=USE_OF_NATURAL_GAS&activityType=COOKING` | One request per metric: NUMBER_OF_HOUSEHOLDS, NUMBER_OF_INSTITUTIONS; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_NATURAL_GAS; activityType=COOKING | **IMPLEMENTED** |
| E25 | Energy | Use of Natural Gas | National Distribution of Natural Gas Stations by Ownership | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION` | dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION | **IMPLEMENTED** |
| E26 | Energy | Use of Natural Gas | Trend of Natural Gas Stations by Ownership | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION` | dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION | **IMPLEMENTED** |
| E27 | Energy | Use of Natural Gas | Regional Distribution of Natural Gas Stations by Ownership | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=USE_OF_NATURAL_GAS&activityType=TRANSPORTATION` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=USE_OF_NATURAL_GAS; activityType=TRANSPORTATION | **IMPLEMENTED** |
| E28 | Energy | Climate-Smart Rural Electrification | Quantity of Emission Reduced in Climate-Smart Rural Electrification | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **PARTIAL** |
| E29 | Energy | Climate-Smart Rural Electrification | Trend of Quantity of Emission Reduced in Climate-Smart Rural Electrification | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **PARTIAL** |
| E30 | Energy | Climate-Smart Rural Electrification | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | dimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **PARTIAL** |
| E31 | Energy | Climate-Smart Rural Electrification | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **PARTIAL** |
| E32 | Energy | Climate-Smart Rural Electrification | Installed and Production Capacity in Climate-Smart Rural Electrification | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | dimension=ACTIVITY_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **IMPLEMENTED** |
| E33 | Energy | Climate-Smart Rural Electrification | Trend of Installed and Production Capacity | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **IMPLEMENTED** |
| E34 | Energy | Climate-Smart Rural Electrification | Regional Breakdown of Installed and Production Capacity in Climate-Smart Rural Electrification | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=INSTALLED_CAPACITY&code=CLIMATE_SMART_RURAL_ELECTRIFICATION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=CLIMATE_SMART_RURAL_ELECTRIFICATION | **IMPLEMENTED** |
| T01 | Transport | Transport Overview | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={TransportSectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Transport sector> | **BLOCKED_BY_DATA** |
| T02 | Transport | Transport Overview | Quantity of Emission Reduced per Mitigation Measures | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}` | dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Transport sector> | **PARTIAL** |
| T03 | Transport | Transport Overview | Trend of Quantity of Emission Reduced per Mitigation Measures | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}` | dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Transport sector> | **PARTIAL** |
| T04 | Transport | Transport Overview | Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={TransportSectorId}` | dimension=INTERVENTION; sectorId=<database ID of Transport sector> | **PARTIAL** |
| T05 | Transport | Transport Overview | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={TransportSectorId}` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Transport sector> | **PARTIAL** |
| T06 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Quantity of Emission Reduced in Low Emission Transport Systems | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=LOW_EMISSION_TRANSPORT_SYSTEMS | **PARTIAL** |
| T07 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Trend of Quantity of Emission Reduced in Low Emission Transport Systems | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LOW_EMISSION_TRANSPORT_SYSTEMS | **PARTIAL** |
| T08 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LOW_EMISSION_TRANSPORT_SYSTEMS` | dimension=ACTIVITY_TYPE; code=LOW_EMISSION_TRANSPORT_SYSTEMS | **PARTIAL** |
| T09 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LOW_EMISSION_TRANSPORT_SYSTEMS` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LOW_EMISSION_TRANSPORT_SYSTEMS | **PARTIAL** |
| T10 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Network Distance and Capacity of Rail Infrastructure by Type | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NETWORK_DISTANCE&metrics=CAPACITY&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=RAILWAY` | dimension=ACTIVITY_TYPE; metrics=NETWORK_DISTANCE, metrics=CAPACITY; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=RAILWAY | **PARTIAL** |
| T11 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Trend of Network Distance and Capacity of Rail Infrastructure | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=RAILWAY` | One request per metric: NETWORK_DISTANCE, CAPACITY; dimension=ACTIVITY_TYPE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=RAILWAY | **PARTIAL** |
| T12 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Network Distance of Road Infrastructure by Type | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=ROAD_INFRASTRUCTURE` | dimension=ACTIVITY_TYPE; metric=NETWORK_DISTANCE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=ROAD_INFRASTRUCTURE | **PARTIAL** |
| T13 | Transport | Low Emission Transport Systems (Mass Rapid Transport System) | Trend of Network Distance of Road Infrastructure by Type | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NETWORK_DISTANCE&code=LOW_EMISSION_TRANSPORT_SYSTEMS&activityType=ROAD_INFRASTRUCTURE` | dimension=ACTIVITY_TYPE; metric=NETWORK_DISTANCE; code=LOW_EMISSION_TRANSPORT_SYSTEMS; activityType=ROAD_INFRASTRUCTURE | **PARTIAL** |
| T14 | Transport | Use of Renewable Energy in Transportation System | Quantity of Emission Reduced in Use of Renewable Energy in Transportation System | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION | **PARTIAL** |
| T15 | Transport | Use of Renewable Energy in Transportation System | Trend of Quantity of Emission Reduced in Use of Renewable Energy in Transportation System | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION | **PARTIAL** |
| T16 | Transport | Use of Renewable Energy in Transportation System | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION` | dimension=ACTIVITY_TYPE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION | **PARTIAL** |
| T17 | Transport | Use of Renewable Energy in Transportation System | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION | **PARTIAL** |
| T18 | Transport | Use of Renewable Energy in Transportation System | Number of BRT Facilities and Capacity by Energy Type | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ENERGY_SOURCE&metrics=DISTINCT_ENTITIES&metrics=CAPACITY&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT` | dimension=ENERGY_SOURCE; metrics=DISTINCT_ENTITIES, metrics=CAPACITY; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT | **IMPLEMENTED** |
| T19 | Transport | Use of Renewable Energy in Transportation System | Trend of Number of BRT Facilities by Energy Type | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT | **IMPLEMENTED** |
| T20 | Transport | Use of Renewable Energy in Transportation System | Number of BRT Facilities by Energy Type at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=BUS_RAPID_TRANSIT` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=BUS_RAPID_TRANSIT | **IMPLEMENTED** |
| T21 | Transport | Use of Renewable Energy in Transportation System | Number of Locomotives by Energy Type | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES | **IMPLEMENTED** |
| T22 | Transport | Use of Renewable Energy in Transportation System | Trend of Number of Locomotives by Energy Type | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES | **IMPLEMENTED** |
| T23 | Transport | Use of Renewable Energy in Transportation System | Number of Locomotives by Energy Type at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=LOCOMOTIVES` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=LOCOMOTIVES | **IMPLEMENTED** |
| T24 | Transport | Use of Renewable Energy in Transportation System | Number of Vehicles Using Electricity and Gas | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES | **IMPLEMENTED** |
| T25 | Transport | Use of Renewable Energy in Transportation System | Trend of Number of Vehicles Using Electricity and Gas | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES | **IMPLEMENTED** |
| T26 | Transport | Use of Renewable Energy in Transportation System | Number of Vehicles Using Electricity and Gas at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION&activityType=PRIVATE_VEHICLES` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION; activityType=PRIVATE_VEHICLES | **IMPLEMENTED** |
| T27 | Transport | Non-Motorized Transport System | Quantity of Emission Reduced by Non-Motorized Transport System | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=NON_MOTORISED_TRANSPORT_SYSTEM | **PARTIAL** |
| T28 | Transport | Non-Motorized Transport System | Trend of Quantity of Emission Reduced by Non-Motorized Transport System | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=NON_MOTORISED_TRANSPORT_SYSTEM | **PARTIAL** |
| T29 | Transport | Non-Motorized Transport System | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=NON_MOTORISED_TRANSPORT_SYSTEM` | dimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM | **PARTIAL** |
| T30 | Transport | Non-Motorized Transport System | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=NON_MOTORISED_TRANSPORT_SYSTEM` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=NON_MOTORISED_TRANSPORT_SYSTEM | **PARTIAL** |
| T31 | Transport | Non-Motorized Transport System | Capacity and Population Served of Cycling and Pedestrian | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=CAPACITY&metrics=POPULATION_SERVED&code=NON_MOTORISED_TRANSPORT_SYSTEM` | dimension=ACTIVITY_TYPE; metrics=CAPACITY, metrics=POPULATION_SERVED; code=NON_MOTORISED_TRANSPORT_SYSTEM | **IMPLEMENTED** |
| T32 | Transport | Non-Motorized Transport System | Trend of Capacity and Population Served of Cycling | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=CAPACITY&code=NON_MOTORISED_TRANSPORT_SYSTEM&activityType=CYCLING` | One request per metric: CAPACITY, POPULATION_SERVED; dimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM; activityType=CYCLING | **IMPLEMENTED** |
| T33 | Transport | Non-Motorized Transport System | Regional Breakdown: Capacity and Population Served of Cycling | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=CAPACITY&code=NON_MOTORISED_TRANSPORT_SYSTEM&activityType=CYCLING` | One request per metric: CAPACITY, POPULATION_SERVED; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=NON_MOTORISED_TRANSPORT_SYSTEM; activityType=CYCLING | **IMPLEMENTED** |
| F01 | Forestry | Forestry Overview | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={ForestrySectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Forestry sector> | **BLOCKED_BY_DATA** |
| F02 | Forestry | Forestry Overview | Quantity of Emission Reduced by Mitigation Measures | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}` | dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Forestry sector> | **PARTIAL** |
| F03 | Forestry | Forestry Overview | Trend of Quantity of Emission Reduced by Mitigation Measures | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}` | dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Forestry sector> | **PARTIAL** |
| F04 | Forestry | Forestry Overview | Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={ForestrySectorId}` | dimension=INTERVENTION; sectorId=<database ID of Forestry sector> | **PARTIAL** |
| F05 | Forestry | Forestry Overview | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={ForestrySectorId}` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Forestry sector> | **PARTIAL** |
| F06 | Forestry | Participatory Forest Management Program | Quantity of Emission Reduced by Participatory Forest Management Program | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **PARTIAL** |
| F07 | Forestry | Participatory Forest Management Program | Trend of Quantity of Emission Reduced by Participatory Forest Management Program | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **PARTIAL** |
| F08 | Forestry | Participatory Forest Management Program | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | dimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **PARTIAL** |
| F09 | Forestry | Participatory Forest Management Program | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **PARTIAL** |
| F10 | Forestry | Participatory Forest Management Program | Number of CBFM and JFM per Ownership | Program-by-management Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=ACTIVITY_TYPE&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | rowDimension=ACTIVITY_TYPE; columnDimension=MANAGEMENT_TYPE; metric=DISTINCT_ENTITIES; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **IMPLEMENTED** |
| F11 | Forestry | Participatory Forest Management Program | Trend of Number of CBFM and JFM | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | dimension=ACTIVITY_TYPE; metric=DISTINCT_ENTITIES; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **IMPLEMENTED** |
| F12 | Forestry | Participatory Forest Management Program | Number of CBFM per Ownership at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=MANAGEMENT_TYPE&metric=DISTINCT_ENTITIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM&activityType=COMMUNITY_BASED_FOREST_MANAGEMENT` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=MANAGEMENT_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM; activityType=COMMUNITY_BASED_FOREST_MANAGEMENT | **IMPLEMENTED** |
| F13 | Forestry | Participatory Forest Management Program | Number of Beneficiaries and Coverage Area of CBFM and JFM | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=NUMBER_OF_BENEFICIARIES&metrics=COVERAGE_AREA&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | dimension=ACTIVITY_TYPE; metrics=NUMBER_OF_BENEFICIARIES, metrics=COVERAGE_AREA; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **IMPLEMENTED** |
| F14 | Forestry | Participatory Forest Management Program | Trend of Number of Beneficiaries of CBFM and JFM | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | One request per metric: NUMBER_OF_BENEFICIARIES; dimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **IMPLEMENTED** |
| F15 | Forestry | Participatory Forest Management Program | Regional Breakdown: Number of Beneficiaries of CBFM and JFM | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | One request per metric: NUMBER_OF_BENEFICIARIES; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM | **IMPLEMENTED** |
| F16 | Forestry | Landscape Restoration Program | Quantity of Emission Reduced by Landscape Restoration Program | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=LANDSCAPE_RESTORATION_PROGRAM | **PARTIAL** |
| F17 | Forestry | Landscape Restoration Program | Trend of Quantity of Emission Reduced by Landscape Restoration Program | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LANDSCAPE_RESTORATION_PROGRAM | **PARTIAL** |
| F18 | Forestry | Landscape Restoration Program | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM | **PARTIAL** |
| F19 | Forestry | Landscape Restoration Program | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LANDSCAPE_RESTORATION_PROGRAM` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LANDSCAPE_RESTORATION_PROGRAM | **PARTIAL** |
| F20 | Forestry | Landscape Restoration Program | Coverage Area and Number of Trees Planted by Landscape Restoration Category | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=COVERAGE_AREA&metrics=NUMBER_OF_TREES&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; metrics=COVERAGE_AREA, metrics=NUMBER_OF_TREES; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| F21 | Forestry | Landscape Restoration Program | Trend of Coverage Area by Landscape Restoration Category | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LANDSCAPE_RESTORATION_PROGRAM` | One request per metric: COVERAGE_AREA; dimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| F22 | Forestry | Landscape Restoration Program | Regional Breakdown: Coverage Area by Landscape Restoration Category | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LANDSCAPE_RESTORATION_PROGRAM` | One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| F23 | Forestry | Landscape Restoration Program | Number of Beneficiaries by Landscape Restoration Category | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; metric=NUMBER_OF_BENEFICIARIES; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| F24 | Forestry | Landscape Restoration Program | Trend of Number of Beneficiaries by Landscape Restoration Category | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM` | dimension=ACTIVITY_TYPE; metric=NUMBER_OF_BENEFICIARIES; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| F25 | Forestry | Landscape Restoration Program | Number of Beneficiaries by Landscape Restoration Category at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=NUMBER_OF_BENEFICIARIES&code=LANDSCAPE_RESTORATION_PROGRAM` | One request per metric: NUMBER_OF_BENEFICIARIES; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LANDSCAPE_RESTORATION_PROGRAM | **IMPLEMENTED** |
| W01 | Waste | Waste Overview | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={WasteSectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Waste sector> | **BLOCKED_BY_DATA** |
| W02 | Waste | Waste Overview | Quantity of Emission Reduced by Mitigation Measures | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}` | dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Waste sector> | **PARTIAL** |
| W03 | Waste | Waste Overview | Trend of Quantity of Emission Reduced by Mitigation Measures | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}` | dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Waste sector> | **PARTIAL** |
| W04 | Waste | Waste Overview | Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={WasteSectorId}` | dimension=INTERVENTION; sectorId=<database ID of Waste sector> | **PARTIAL** |
| W05 | Waste | Waste Overview | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={WasteSectorId}` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Waste sector> | **PARTIAL** |
| W06 | Waste | Environmentally Sound Waste Management Practice | Quantity of Emission Reduced in Environmentally Sound Waste Management Practice | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE | **PARTIAL** |
| W07 | Waste | Environmentally Sound Waste Management Practice | Trend of Quantity of Emission Reduced in Environmentally Sound Waste Management Practice | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE | **PARTIAL** |
| W08 | Waste | Environmentally Sound Waste Management Practice | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE | **PARTIAL** |
| W09 | Waste | Environmentally Sound Waste Management Practice | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE | **PARTIAL** |
| W10 | Waste | Environmentally Sound Waste Management Practice | Number of Recycling Facilities per Type of Waste | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **PARTIAL** |
| W11 | Waste | Environmentally Sound Waste Management Practice | Trend of Number of Recycling Facilities per Type of Waste | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **PARTIAL** |
| W12 | Waste | Environmentally Sound Waste Management Practice | Regional Breakdown: Number of Recycling Facilities per Type of Waste | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **PARTIAL** |
| W13 | Waste | Environmentally Sound Waste Management Practice | Installed Capacity and Production Capacity of Recycling Facilities | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=WASTE_TYPE&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | dimension=WASTE_TYPE; metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **IMPLEMENTED** |
| W14 | Waste | Environmentally Sound Waste Management Practice | Trend of Installed Capacity and Production Capacity of Recycling Facilities | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=INSTALLED_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; dimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **IMPLEMENTED** |
| W15 | Waste | Environmentally Sound Waste Management Practice | Regional Breakdown: Installed Capacity and Production Capacity of Recycling Facilities | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=INSTALLED_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=RECYCLING` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=RECYCLING | **IMPLEMENTED** |
| W16 | Waste | Environmentally Sound Waste Management Practice | Number of Sorting Facilities per Type of Waste | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **PARTIAL** |
| W17 | Waste | Environmentally Sound Waste Management Practice | Trend of Number of Sorting Facilities per Type of Waste | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **PARTIAL** |
| W18 | Waste | Environmentally Sound Waste Management Practice | Regional Breakdown: Number of Sorting Facilities per Type of Waste | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **PARTIAL** |
| W19 | Waste | Environmentally Sound Waste Management Practice | Sorting Capacity of Sorting Facilities | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | dimension=WASTE_TYPE; metric=SORTING_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **IMPLEMENTED** |
| W20 | Waste | Environmentally Sound Waste Management Practice | Trend of Sorting Capacity of Sorting Facilities | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | dimension=WASTE_TYPE; metric=SORTING_CAPACITY; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **IMPLEMENTED** |
| W21 | Waste | Environmentally Sound Waste Management Practice | Sorting Capacity of Sorting Facilities at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=SORTING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=SORTING` | One request per metric: SORTING_CAPACITY; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=SORTING | **IMPLEMENTED** |
| W22 | Waste | Environmentally Sound Waste Management Practice | Number of Waste Facilities per Type of Waste | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **PARTIAL** |
| W23 | Waste | Environmentally Sound Waste Management Practice | Trend of Number of Waste Facilities per Type of Waste | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | dimension=WASTE_TYPE; metric=DISTINCT_ENTITIES; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **PARTIAL** |
| W24 | Waste | Environmentally Sound Waste Management Practice | Regional Breakdown: Number of Waste Facilities per Type of Waste | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=DISTINCT_ENTITIES&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **PARTIAL** |
| W25 | Waste | Environmentally Sound Waste Management Practice | Capacity and Size of Landfill | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=WASTE_TYPE&metrics=ORGANIC_WASTE_RECEIVING_CAPACITY&metrics=LANDFILL_SIZE&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | dimension=WASTE_TYPE; metrics=ORGANIC_WASTE_RECEIVING_CAPACITY, metrics=LANDFILL_SIZE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **IMPLEMENTED** |
| W26 | Waste | Environmentally Sound Waste Management Practice | Trend of Capacity and Size of Landfill | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=WASTE_TYPE&metric=ORGANIC_WASTE_RECEIVING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | One request per metric: ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE; dimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **IMPLEMENTED** |
| W27 | Waste | Environmentally Sound Waste Management Practice | Regional Breakdown: Capacity and Size of Landfill | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=WASTE_TYPE&metric=ORGANIC_WASTE_RECEIVING_CAPACITY&code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE&activityType=LANDFILL` | One request per metric: ORGANIC_WASTE_RECEIVING_CAPACITY, LANDFILL_SIZE; rowDimension=REGION; columnDimension=WASTE_TYPE; code=ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE; activityType=LANDFILL | **IMPLEMENTED** |
| W28 | Waste | Waste to Energy Technologies | Quantity of Emission Reduced in Waste to Energy Technologies | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES` | dimension=TECHNOLOGY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=WASTE_TO_ENERGY_TECHNOLOGIES | **PARTIAL** |
| W29 | Waste | Waste to Energy Technologies | Trend of Quantity of Emission Reduced in Waste to Energy Technologies | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES` | dimension=TECHNOLOGY_TYPE; metric=<one gas metric>; code=WASTE_TO_ENERGY_TECHNOLOGIES | **PARTIAL** |
| W30 | Waste | Waste to Energy Technologies | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=TECHNOLOGY_TYPE&code=WASTE_TO_ENERGY_TECHNOLOGIES` | dimension=TECHNOLOGY_TYPE; code=WASTE_TO_ENERGY_TECHNOLOGIES | **PARTIAL** |
| W31 | Waste | Waste to Energy Technologies | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=QUANTITY_REDUCED_CO2&code=WASTE_TO_ENERGY_TECHNOLOGIES` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=WASTE_TO_ENERGY_TECHNOLOGIES | **PARTIAL** |
| W32 | Waste | Waste to Energy Technologies | Number of Biogas Plants, Production and Installed Capacity | Three-series Composite Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=DISTINCT_ENTITIES&metrics=PRODUCTION_CAPACITY_PER_DAY&metrics=INSTALLED_CAPACITY&code=BIOGAS_COW_DUNG` | dimension=TECHNOLOGY_TYPE; metrics=DISTINCT_ENTITIES, metrics=PRODUCTION_CAPACITY_PER_DAY, metrics=INSTALLED_CAPACITY; code=BIOGAS_COW_DUNG | **IMPLEMENTED** |
| W33 | Waste | Waste to Energy Technologies | Trend of Number of Biogas Plants, Production and Installed Capacity | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=DISTINCT_ENTITIES&code=BIOGAS_COW_DUNG` | One request per metric: DISTINCT_ENTITIES, PRODUCTION_CAPACITY_PER_DAY, INSTALLED_CAPACITY; dimension=TECHNOLOGY_TYPE; code=BIOGAS_COW_DUNG | **IMPLEMENTED** |
| W34 | Waste | Waste to Energy Technologies | Installed and Production Capacity of Biogas Plants at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=BIOGAS_COW_DUNG` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=BIOGAS_COW_DUNG | **IMPLEMENTED** |
| W35 | Waste | Waste to Energy Technologies | Number of Alternative Charcoal Production Facilities, Installed and Production Capacity | Three-series Composite Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=TECHNOLOGY_TYPE&metrics=DISTINCT_ENTITIES&metrics=INSTALLED_CAPACITY&metrics=PRODUCTION_CAPACITY_PER_DAY&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | dimension=TECHNOLOGY_TYPE; metrics=DISTINCT_ENTITIES, metrics=INSTALLED_CAPACITY, metrics=PRODUCTION_CAPACITY_PER_DAY; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| W36 | Waste | Waste to Energy Technologies | Trend of Number of Alternative Charcoal Production Facilities, Installed and Production Capacity | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=TECHNOLOGY_TYPE&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | One request per metric: DISTINCT_ENTITIES, INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; dimension=TECHNOLOGY_TYPE; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| W37 | Waste | Waste to Energy Technologies | Installed and Production Capacity of Alternative Charcoal Production Facilities at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | One request per metric: INSTALLED_CAPACITY, PRODUCTION_CAPACITY_PER_DAY; rowDimension=REGION; columnDimension=TECHNOLOGY_TYPE; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| W38 | Waste | Waste to Energy Technologies | Number of Plants per Production Material in Alternative Charcoal Production | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | dimension=MATERIAL; metric=DISTINCT_ENTITIES; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| W39 | Waste | Waste to Energy Technologies | Trend of Number of Plants per Production Material in Alternative Charcoal Production | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | dimension=MATERIAL; metric=DISTINCT_ENTITIES; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| W40 | Waste | Waste to Energy Technologies | Number of Plants per Production Material in Alternative Charcoal Production at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=MATERIAL&metric=DISTINCT_ENTITIES&code=ALTERNATIVE_CHARCOAL_PRODUCTION` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=MATERIAL; code=ALTERNATIVE_CHARCOAL_PRODUCTION | **IMPLEMENTED** |
| I01 | Industry | Diversification of Clean Energy Sources | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={IndustrySectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Industry sector> | **BLOCKED_BY_DATA** |
| I02 | Industry | Diversification of Clean Energy Sources | Quantity of Emission Reduced in Diversification of Clean Energy Sources | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=ENERGY_SOURCE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **PARTIAL** |
| I03 | Industry | Diversification of Clean Energy Sources | Trend of Quantity of Emission Reduced in Diversification of Clean Energy Sources | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=ENERGY_SOURCE; metric=<one gas metric>; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| I04 | Industry | Diversification of Clean Energy Sources | Amount of GHG Emissions Reduced by Clean Energy Source per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ENERGY_SOURCE&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=ENERGY_SOURCE; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **PARTIAL** |
| I05 | Industry | Diversification of Clean Energy Sources | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=QUANTITY_REDUCED_CO2&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| I06 | Industry | Diversification of Clean Energy Sources | Number of Industries and Energy Consumption per Clean Energy Source | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ENERGY_SOURCE&metrics=DISTINCT_ENTITIES&metrics=ENERGY_CONSUMPTION&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=ENERGY_SOURCE; metrics=DISTINCT_ENTITIES, metrics=ENERGY_CONSUMPTION; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **IMPLEMENTED** |
| I07 | Industry | Diversification of Clean Energy Sources | Trend of Number of Industries per Clean Energy Source | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=ENERGY_SOURCE; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| I08 | Industry | Diversification of Clean Energy Sources | Number of Industries per Clean Energy Source at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ENERGY_SOURCE&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=ENERGY_SOURCE; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| I09 | Industry | Diversification of Clean Energy Sources | Number of Industries per Clean Energy Source per Ownership | Heatmap | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=ENERGY_SOURCE&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | rowDimension=ENERGY_SOURCE; columnDimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **IMPLEMENTED** |
| I10 | Industry | Diversification of Clean Energy Sources | Trend of Number of Industries per Clean Energy Source per Ownership | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | dimension=OWNERSHIP; metric=DISTINCT_ENTITIES; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| I11 | Industry | Diversification of Clean Energy Sources | Number of Industries per Clean Energy Source per Ownership at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=OWNERSHIP&metric=DISTINCT_ENTITIES&code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | One request per metric: DISTINCT_ENTITIES; rowDimension=REGION; columnDimension=OWNERSHIP; code=DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES | **BLOCKED_BY_DATA** |
| L01 | Livestock | Livestock Overview | Amount of GHGs Reduced (tCO₂e) | Summary Card | GET | `/api/v1/climate-change/mitigation/analytics/summary?gasType=CO2&sectorId={LivestockSectorId}` | gasType=CO2/CH4/N2O/HFCS; sectorId=<database ID of Livestock sector> | **BLOCKED_BY_DATA** |
| L02 | Livestock | Livestock Overview | Quantity of Emission Reduced by Livestock Management Practice and Use of Improved Animal Breeds and Feeds | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}` | dimension=INTERVENTION; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; sectorId=<database ID of Livestock sector> | **PARTIAL** |
| L03 | Livestock | Livestock Overview | Trend of Quantity of Emission Reduced by Livestock Management Practice and Use of Improved Animal Breeds and Feeds | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}` | dimension=INTERVENTION; metric=<one gas metric>; sectorId=<database ID of Livestock sector> | **PARTIAL** |
| L04 | Livestock | Livestock Overview | Amount of GHG Emissions Reduced by Mitigation Measures per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=INTERVENTION&sectorId={LivestockSectorId}` | dimension=INTERVENTION; sectorId=<database ID of Livestock sector> | **PARTIAL** |
| L05 | Livestock | Livestock Overview | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=INTERVENTION&metric=QUANTITY_REDUCED_CO2&sectorId={LivestockSectorId}` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; sectorId=<database ID of Livestock sector> | **PARTIAL** |
| L06 | Livestock | Livestock Management Practice | Quantity of Emission Reduced in Livestock Management Practice | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=LIVESTOCK_MANAGEMENT_PRACTICE | **PARTIAL** |
| L07 | Livestock | Livestock Management Practice | Trend of Quantity of Emission Reduced in Livestock Management Practice | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=LIVESTOCK_MANAGEMENT_PRACTICE | **PARTIAL** |
| L08 | Livestock | Livestock Management Practice | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=LIVESTOCK_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; code=LIVESTOCK_MANAGEMENT_PRACTICE | **PARTIAL** |
| L09 | Livestock | Livestock Management Practice | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=LIVESTOCK_MANAGEMENT_PRACTICE` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=LIVESTOCK_MANAGEMENT_PRACTICE | **PARTIAL** |
| L10 | Livestock | Livestock Management Practice | Coverage Area under Livestock Management Practice per Category | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=LIVESTOCK_MANAGEMENT_PRACTICE | **IMPLEMENTED** |
| L11 | Livestock | Livestock Management Practice | Trend of Coverage Area under Livestock Management Practice per Category | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE` | dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=LIVESTOCK_MANAGEMENT_PRACTICE | **IMPLEMENTED** |
| L12 | Livestock | Livestock Management Practice | Coverage Area under Livestock Management Practice per Category at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=LIVESTOCK_MANAGEMENT_PRACTICE` | One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=LIVESTOCK_MANAGEMENT_PRACTICE | **IMPLEMENTED** |
| L13 | Livestock | Use of Improved Animal Breeds and Feeds | Quantity of Emission Reduced in the Use of Improved Animal Breeds and Feeds | Bar / Donut Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; metric=<one of QUANTITY_REDUCED_CO2/QUANTITY_REDUCED_CH4/QUANTITY_REDUCED_N2O/QUANTITY_REDUCED_HFCS>; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **PARTIAL** |
| L14 | Livestock | Use of Improved Animal Breeds and Feeds | Trend of Quantity of Emission Reduced in the Use of Improved Animal Breeds and Feeds | Line / Spline Area Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; metric=<one gas metric>; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **PARTIAL** |
| L15 | Livestock | Use of Improved Animal Breeds and Feeds | Amount of GHG Emissions Reduced by Category per Type of Gas | Stacked Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=ACTIVITY_TYPE&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **PARTIAL** |
| L16 | Livestock | Use of Improved Animal Breeds and Feeds | Trend of GHG Emissions Reduced by Type of Gas | Multi-series Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=QUANTITY_REDUCED_CO2&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | Call once for each of QUANTITY_REDUCED_CO2, QUANTITY_REDUCED_CH4, QUANTITY_REDUCED_N2O, QUANTITY_REDUCED_HFCS; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **PARTIAL** |
| L17 | Livestock | Use of Improved Animal Breeds and Feeds | Coverage Area per Farms with Improved Livestock Breeds and Fodders | Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/grouped?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |
| L18 | Livestock | Use of Improved Animal Breeds and Feeds | Trend of Coverage Area per Farms with Improved Livestock Breeds and Fodders | Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; metric=COVERAGE_AREA; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |
| L19 | Livestock | Use of Improved Animal Breeds and Feeds | Coverage Area per Farms with Improved Livestock Breeds and Fodders at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=COVERAGE_AREA&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | One request per metric: COVERAGE_AREA; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |
| L20 | Livestock | Use of Improved Animal Breeds and Feeds | Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes) | Dual-axis Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=LIVESTOCK_PRODUCTION_CAPACITY&metrics=FODDER_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | dimension=ACTIVITY_TYPE; metrics=LIVESTOCK_PRODUCTION_CAPACITY, metrics=FODDER_PRODUCTION_CAPACITY; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |
| L21 | Livestock | Use of Improved Animal Breeds and Feeds | Trend of Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes) | Multi-series / Dual-axis Line Chart | GET | `/api/v1/climate-change/mitigation/analytics/trend?dimension=ACTIVITY_TYPE&metric=LIVESTOCK_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | One request per metric: LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY; dimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |
| L22 | Livestock | Use of Improved Animal Breeds and Feeds | Capacity of Farms with Improved Livestock Breeds (Number) and Fodders (Tonnes) at Regional Level | Regional Heatmap / Grouped Bar Chart | GET | `/api/v1/climate-change/mitigation/analytics/heatmap?rowDimension=REGION&columnDimension=ACTIVITY_TYPE&metric=LIVESTOCK_PRODUCTION_CAPACITY&code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | One request per metric: LIVESTOCK_PRODUCTION_CAPACITY, FODDER_PRODUCTION_CAPACITY; rowDimension=REGION; columnDimension=ACTIVITY_TYPE; code=USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS | **IMPLEMENTED** |

## Additional request and response examples

Examples show structure, not production values.

### Grouped clean-power capacity

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/grouped?dimension=TECHNOLOGY_TYPE&metric=INSTALLED_CAPACITY&code=CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION&coverage=TANZANIA_MAINLAND&yearFrom=2021&yearTo=2026
Authorization: Bearer <ACCESS_TOKEN>
```

```json
{
  "success": true,
  "message": "Grouped mitigation analytics fetched",
  "data": {
    "metric": "INSTALLED_CAPACITY",
    "aggregation": "SUM_OR_SNAPSHOT",
    "data": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "value": 0,
        "unit": "UNIT",
        "percentage": 0
      }
    ],
    "filters": {},
    "warnings": []
  }
}
```

### Gas composition

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/gas-composition?dimension=SECTOR&coverage=TANZANIA_MAINLAND
Authorization: Bearer <ACCESS_TOKEN>
```

```json
{
  "success": true,
  "message": "Mitigation gas composition fetched",
  "data": {
    "categories": [
      {
        "code": "CATEGORY",
        "label": "Category",
        "co2": 0,
        "ch4": 0,
        "n2o": 0,
        "hfcs": 0,
        "unit": "UNSPECIFIED_REPORTED_GAS_QUANTITY"
      }
    ],
    "filters": {},
    "warnings": []
  }
}
```

### Multi-metric series

```http
GET /climate-change/api/v1/climate-change/mitigation/analytics/series?dimension=ACTIVITY_TYPE&metrics=COVERAGE_AREA&metrics=NUMBER_OF_TREES&code=LANDSCAPE_RESTORATION_PROGRAM
Authorization: Bearer <ACCESS_TOKEN>
```

```json
{
  "success": true,
  "message": "Mitigation metric series fetched",
  "data": {
    "dimension": "DIMENSION",
    "series": [
      {
        "key": "METRIC",
        "label": "Metric",
        "aggregation": "LATEST_SNAPSHOT_SUM",
        "data": [
          {
            "code": "CATEGORY",
            "label": "Category",
            "value": 0,
            "unit": "UNIT",
            "percentage": 0
          }
        ]
      }
    ],
    "filters": {},
    "warnings": []
  }
}
```

### Submission source (not a dashboard endpoint)

`POST /api/v1/climate-change/measure-action-field-data-request` accepts `category`, dynamic `values`, and `sectorMeasureCategoryUid`. The analytics APIs do not read drafts; the saved record must later be approved and active.

## Frontend integration guide

```ts
const BASE = "/climate-change/api/v1/climate-change/mitigation/analytics";

type ApiResponse<T> = { success: boolean; message: string; data: T };

async function mitigationGet<T>(
  path: string,
  params: URLSearchParams,
  token?: string,
  signal?: AbortSignal,
): Promise<T> {
  const response = await fetch(`${BASE}/${path}?${params}`, {
    headers: token ? { Authorization: `Bearer ${token}` } : {},
    signal,
  });
  const body: ApiResponse<T> = await response.json();
  if (!response.ok || !body.success) throw new Error(body.message || "Mitigation analytics request failed");
  return body.data;
}

const params = new URLSearchParams({
  dimension: "TECHNOLOGY_TYPE",
  metric: "INSTALLED_CAPACITY",
  code: "CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION",
  coverage: "TANZANIA_MAINLAND",
});
const grouped = await mitigationGet<any>("grouped", params, accessToken);
const categories = grouped.data.map((point: any) => point.label);
const values = grouped.data.map((point: any) => point.value);
```

Integration rules:

1. Resolve environment-specific `sectorId` and `sectorMeasureUid` from master data; never hard-code IDs across environments.
2. Build query parameters only for active filters. Use either `year` or a range.
3. For gas trends, issue four parallel `/trend` requests with the four gas metrics; never add the returned series unless conversion semantics are introduced later.
4. For regional multi-metric charts, issue one `/heatmap` call per metric with `Promise.all`.
5. Treat empty `data`/`cells` as “No approved data for selected filters”; do not fabricate observations.
6. Cancel stale requests with `AbortController` when filters change.
7. Cache/deduplicate by endpoint plus canonicalized query string (React Query/SWR keys work well). Several graphs share the same response.
8. Preserve backend units. Do not merge MW with MMscfd, m³ with m³/day, Tonnes with Tonnes/day, or raw gases with tCO₂e.
9. For industry source counts, an industry can appear in multiple source buckets. Do not sum bucket counts to calculate a national distinct total.

## Missing APIs and data gaps

### Coverage summary

- Requirements: **170**
- IMPLEMENTED: **66**
- PARTIAL: **90**
- MISSING: **0**
- BLOCKED_BY_DATA: **14**
- Distinct actual analytics endpoints mapped: **6**

### Incomplete requirements by sector

- **General:** G01 (BLOCKED_BY_DATA), G02 (PARTIAL), G03 (PARTIAL), G04 (PARTIAL), G05 (PARTIAL).
- **Energy:** E01 (BLOCKED_BY_DATA), E02 (PARTIAL), E03 (PARTIAL), E04 (PARTIAL), E05 (PARTIAL), E06 (PARTIAL), E07 (PARTIAL), E08 (PARTIAL), E09 (PARTIAL), E15 (PARTIAL), E16 (PARTIAL), E17 (PARTIAL), E18 (PARTIAL), E19 (PARTIAL), E20 (PARTIAL), E21 (PARTIAL), E23 (BLOCKED_BY_DATA), E28 (PARTIAL), E29 (PARTIAL), E30 (PARTIAL), E31 (PARTIAL).
- **Transport:** T01 (BLOCKED_BY_DATA), T02 (PARTIAL), T03 (PARTIAL), T04 (PARTIAL), T05 (PARTIAL), T06 (PARTIAL), T07 (PARTIAL), T08 (PARTIAL), T09 (PARTIAL), T10 (PARTIAL), T11 (PARTIAL), T12 (PARTIAL), T13 (PARTIAL), T14 (PARTIAL), T15 (PARTIAL), T16 (PARTIAL), T17 (PARTIAL), T27 (PARTIAL), T28 (PARTIAL), T29 (PARTIAL), T30 (PARTIAL).
- **Forestry:** F01 (BLOCKED_BY_DATA), F02 (PARTIAL), F03 (PARTIAL), F04 (PARTIAL), F05 (PARTIAL), F06 (PARTIAL), F07 (PARTIAL), F08 (PARTIAL), F09 (PARTIAL), F16 (PARTIAL), F17 (PARTIAL), F18 (PARTIAL), F19 (PARTIAL).
- **Waste:** W01 (BLOCKED_BY_DATA), W02 (PARTIAL), W03 (PARTIAL), W04 (PARTIAL), W05 (PARTIAL), W06 (PARTIAL), W07 (PARTIAL), W08 (PARTIAL), W09 (PARTIAL), W10 (PARTIAL), W11 (PARTIAL), W12 (PARTIAL), W16 (PARTIAL), W17 (PARTIAL), W18 (PARTIAL), W22 (PARTIAL), W23 (PARTIAL), W24 (PARTIAL), W28 (PARTIAL), W29 (PARTIAL), W30 (PARTIAL), W31 (PARTIAL).
- **Industry:** I01 (BLOCKED_BY_DATA), I02 (PARTIAL), I03 (BLOCKED_BY_DATA), I04 (PARTIAL), I05 (BLOCKED_BY_DATA), I07 (BLOCKED_BY_DATA), I08 (BLOCKED_BY_DATA), I10 (BLOCKED_BY_DATA), I11 (BLOCKED_BY_DATA).
- **Livestock:** L01 (BLOCKED_BY_DATA), L02 (PARTIAL), L03 (PARTIAL), L04 (PARTIAL), L05 (PARTIAL), L06 (PARTIAL), L07 (PARTIAL), L08 (PARTIAL), L09 (PARTIAL), L13 (PARTIAL), L14 (PARTIAL), L15 (PARTIAL), L16 (PARTIAL).

### Concrete gaps

- **All sectors — tCO₂e:** no gas units, CO₂e flag, GWP version/factors, or canonical reporting unit. A correct total cannot be calculated from the four raw fields.
- **All sectors — reporting year:** `values.year` has action-specific meaning. There is no common reporting-year field.
- **Energy / natural gas:** POWER_PRODUCTION has installed capacity and natural gas consumed but no production-capacity field. COOKING has no year, blocking E23.
- **Transport / low-emission systems:** `railType` and `infrastructureType` are captured, but are absent from analytics dimensions/filters; T10–T13 cannot split SGR/MGR or BRT/CYCLING.
- **Waste:** current waste schema offers only ORGANIC_WASTE and LIVESTOCK_MANURE_WASTE. Plastic, Metal, Rubber, Glass, Paper, and a dashboard-level generic Organic taxonomy are unavailable.
- **Industry:** the schema has no year, Coverage, Region, or Council; I03, I05, I07, I08, I10, and I11 cannot be produced. Industry source totals must deduplicate by `industryName`; source-bucket counts overlap by design.
- **Entity counts:** identity is derived from category, location, available name/ID/vehicle number and sometimes subtype. Historical records lacking a stable real-world identifier can make distinct counts approximate.
- **Natural gas stations:** transportation records provide a facility/station name and ownership, but no dedicated station entity/table.
- **Energy consumption per day:** no approved unit is stored, so the backend returns `UNSPECIFIED_PER_DAY`.

No controller/service/repository/schema changes were made for these gaps.

## Testing and verification results

### Source-code verification

Verified directly against:

- `MitigationAnalyticsController`: six GET mappings and parameter binding.
- `MitigationAnalyticsFilter`, dimension/metric/gas enums: accepted filters and metric names.
- `MitigationAnalyticsService`: validation, response DTO construction, warnings, percentages and summary behavior.
- `MitigationAnalyticsRepository`: approved/active/deleted predicates, JSONB expansion, parent/legacy action normalization, location precedence, year extraction, identity logic, aggregation and units.
- `MitigationAnalyticsDtos` and common `ApiResponse`: exact response properties/envelope.
- mitigation schema SQL, schema seeder and submission controller: action codes, nested fields/options, units and submission origin.
- gateway route: `/climate-change/**` with `StripPrefix=1`.
- service security: direct `/api/v1/**` currently permitted; gateway/deployment authentication may still apply.

### Runtime verification

- Focused Maven tests passed: `MitigationAnalyticsRepositoryTest` (3) and `MitigationAnalyticsServiceTest` (7), for **10 tests, 0 failures, 0 errors, 0 skipped**.
- `BUILD SUCCESS` was produced by `./mvnw -pl climate-change -Dtest=MitigationAnalyticsServiceTest,MitigationAnalyticsRepositoryTest test`.
- Live HTTP probes to direct service port 8086 and gateway port 8085 returned connection failure because neither process was listening. Therefore this README does **not** claim live endpoint/database response verification.

### Mechanical document validation

Generated and checked for exactly 170 unique IDs: G01–G05, E01–E34, T01–T33, F01–F25, W01–W40, I01–I11 and L01–L22. Status totals: BLOCKED_BY_DATA: 14, PARTIAL: 90, IMPLEMENTED: 66. Sector totals: General 5, Energy 34, Transport 33, Forestry 25, Waste 40, Industry 11, Livestock 22.


