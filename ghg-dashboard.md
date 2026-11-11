# GHG Emissions Analytics API

## Purpose

The GHG Emissions Analytics API supplies the three GHG dashboard KPI cards and all 27 dashboard charts from saved GHG Inventory records. The API performs aggregation in PostgreSQL and returns chart-ready category, trend, gas-by-sector, and regional heatmap datasets.

The API does not generate demo values or missing historical observations. An empty array means no eligible saved records matched the supplied filters.

## Connection details

| Item | Value |
|---|---|
| HTTP method | `GET` for every analytics operation |
| Service endpoint prefix | `/api/v1/ghg-emissions/analytics` |
| Gateway endpoint prefix | `/climate-change/api/v1/ghg-emissions/analytics` |
| Frontend gateway environment variable | `NEXT_PUBLIC_API_BASE_URL_80` |
| Authentication | JWT bearer token |
| Response content type | `application/json` |
| Analytics output unit | `tCO2e` |

## Required headers

```http
Authorization: Bearer <access-token>
Accept: application/json
Content-Type: application/json
```

## Standard response wrapper

Every successful endpoint uses the existing IEMIS `ApiResponse<T>` wrapper:

```json
{
  "success": true,
  "message": "GHG dashboard summary retrieved",
  "data": {}
}
```

Errors use the same wrapper:

```json
{
  "success": false,
  "message": "year cannot be combined with startYear or endYear",
  "data": null
}
```

## Supported filters

| Query parameter | Type | Meaning |
|---|---:|---|
| `year` | integer | Exact reporting year. Do not combine with `startYear` or `endYear`. |
| `startYear` | integer | First reporting year, inclusive. |
| `endYear` | integer | Last reporting year, inclusive. |
| `sectorId` | positive integer | Existing sector database identifier. |
| `contributorId` | positive integer | Existing contributor database identifier. |
| `gasType` | string | Gas display name, UUID, or supported code such as `CO2`, `CH4`, `N2O`, or `HFCs`. |
| `regionId` | positive integer | Existing region identifier; Land Use removal analytics only. |
| `districtId` | positive integer | Existing district identifier. The legacy Land Use form calls its council selection “District”, so a selected council ID is also accepted. |
| `wasteSubcategoryId` | positive integer | Existing Waste subcategory identifier. |
| `transportMode` | string | Existing transport enum/display value, for example `ROAD`. |
| `fuelCategoryId` | positive integer | Existing fuel-category identifier. |
| `carbonSinkId` | positive integer | Existing carbon-sink/restoration-type identifier. |

Years must be between 1900 and 3000. `startYear` must not be greater than `endYear`.

## Error handling

| Status | Meaning | Frontend action |
|---:|---|---|
| `200` | Successful aggregate, including a legitimate empty dataset | Render data or a no-data state. |
| `400` | Invalid filter or filter combination | Correct the filter request and show the returned `message`. |
| `401` | Missing, invalid, or expired JWT | Use the existing refresh/login flow. |
| `403` | The user lacks the generated analytics permission | Show a permission message; do not retry continuously. |
| `404` | Incorrect gateway or endpoint path | Verify the `/climate-change` gateway prefix. |
| `500` | Controlled backend failure | Show an error state and log the request context. |

## Measurement units

- Data entry stores emission and removal amounts as `GgCO2e`.
- Analytics responses return `tCO2e`.
- Conversion: `1 GgCO2e = 1,000 tCO2e`.
- Conversion: `1 MtCO2e = 1,000,000 tCO2e`.
- Every analytics response includes its unit in `unit`, or inside a KPI `Metric`.
- Do not add values carrying different physical units.

## Complete API reference

All paths below are relative to:

```text
${NEXT_PUBLIC_API_BASE_URL_80}/climate-change/api/v1/ghg-emissions/analytics
```

| Dashboard | Graph / operation | Method | API endpoint | Applicable filters |
|---|---|---|---|---|
| Overview | Dashboard Summary — 3 KPI cards | GET | `/summary` | all applicable emission and removal filters |
| Overview | Amount of GHG Emissions per Sector | GET | `/emissions-by-sector` | year/range, sector, contributor, gas, Waste/Transport classification |
| Overview | Trend of GHG Emissions per Sector | GET | `/emissions-trend-by-sector` | year/range, sector, contributor, gas, Waste/Transport classification |
| Overview | Emissions by Gas Type per Sector | GET | `/emissions-by-gas-sector` | year/range, sector, contributor, gas, Waste/Transport classification |
| Overview | Trend of Emissions by GHG Type | GET | `/emissions-trend-by-gas` | year/range, sector, contributor, gas, Waste/Transport classification |
| Energy | Emissions per Contributor | GET | `/sectors/energy/contributors` | year/range, contributor, gas |
| Energy | Contributor Emission Trends | GET | `/sectors/energy/trends` | year/range, contributor, gas |
| Industrial Process | Emissions per Contributor | GET | `/sectors/industrial-process/contributors` | year/range, contributor, gas |
| Industrial Process | Contributor Emission Trends | GET | `/sectors/industrial-process/trends` | year/range, contributor, gas |
| Mining | Emissions per Contributor | GET | `/sectors/mining/contributors` | year/range, contributor, gas |
| Mining | Contributor Emission Trends | GET | `/sectors/mining/trends` | year/range, contributor, gas |
| Agriculture | Emissions per Contributor | GET | `/sectors/agriculture/contributors` | year/range, contributor, gas |
| Agriculture | Contributor Emission Trends | GET | `/sectors/agriculture/trends` | year/range, contributor, gas |
| Livestock | Emissions per Contributor | GET | `/sectors/livestock/contributors` | year/range, contributor, gas |
| Livestock | Contributor Emission Trends | GET | `/sectors/livestock/trends` | year/range, contributor, gas |
| Human Settlement | Emissions per Contributor | GET | `/sectors/human-settlement/contributors` | year/range, contributor, gas |
| Human Settlement | Contributor Emission Trends | GET | `/sectors/human-settlement/trends` | year/range, contributor, gas |
| Waste | Emissions by Subcategory | GET | `/sectors/waste/subcategories` | year/range, subcategory, contributor, gas |
| Waste | Subcategory Trends | GET | `/sectors/waste/subcategories/trends` | year/range, subcategory, contributor, gas |
| Waste | Emissions by Contributor | GET | `/sectors/waste/contributors` | year/range, subcategory, contributor, gas |
| Waste | Contributor Trends | GET | `/sectors/waste/contributors/trends` | year/range, subcategory, contributor, gas |
| Transport | Emissions by Transport Mode | GET | `/sectors/transport/modes` | year/range, mode, contributor, fuel, gas |
| Transport | Transport Mode Trends | GET | `/sectors/transport/modes/trends` | year/range, mode, contributor, fuel, gas |
| Transport | Emissions by Fuel Category | GET | `/sectors/transport/fuels` | year/range, mode, contributor, fuel, gas |
| Transport | Fuel Category Trends | GET | `/sectors/transport/fuels/trends` | year/range, mode, contributor, fuel, gas |
| Land Use | Removals by Carbon Sink | GET | `/sectors/land-use/removals-by-sink` | year/range, region, district, sink, gas |
| Land Use | Carbon Sink Removal Trends | GET | `/sectors/land-use/removals-trend` | year/range, region, district, sink, gas |
| Land Use | Regional Removal Heatmap | GET | `/sectors/land-use/removals-by-region` | year/range, region, district, sink, gas |

## Individual API documentation

The numeric values in examples illustrate the actual DTO shape. Production values always come from PostgreSQL.

### 1. GHG Dashboard Summary

**Endpoint:** `/summary`  
**Method:** `GET`  
**Purpose:** Supplies Total GHG Emissions, Amount of GHGs Reduced, and emission totals per gas. `totalGhgReduced` is sourced from saved Amount of GHGs Removed rows.  
**Supported filters:** `year`, `startYear`, `endYear`, `sectorId`, `contributorId`, `gasType`, `regionId`, `districtId`, `wasteSubcategoryId`, `transportMode`, `fuelCategoryId`, `carbonSinkId`  
**Example request:** `/summary`

**Example response:**

```json
{"success":true,"message":"GHG dashboard summary retrieved","data":{"totalEmissions":{"value":0,"unit":"tCO2e","source":null},"totalGhgReduced":{"value":15883300,"unit":"tCO2e","source":"AMOUNT_OF_GHGS_REMOVED"},"gasBreakdown":[],"filters":{"year":null,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** Render `data.totalEmissions.value`, `data.totalGhgReduced.value`, and `data.gasBreakdown`. Use each metric's `unit`; use `gasBreakdown[].name` as the label and `gasBreakdown[].total` as the value.

### 2. Amount of GHG Emissions per Sector

**Endpoint:** `/emissions-by-sector`  
**Method:** `GET`  
**Purpose:** Returns eligible positive emissions grouped by sector. Land Use removals are not mixed into these totals.  
**Supported filters:** `year`, `startYear`, `endYear`, `sectorId`, `contributorId`, `gasType`, `wasteSubcategoryId`, `transportMode`, `fuelCategoryId`  
**Example request:** `/emissions-by-sector?startYear=2021&endYear=2026`

**Example response:**

```json
{"success":true,"message":"GHG emissions by sector retrieved","data":{"contextCode":"GHG_INVENTORY","contextName":"GHG Inventory","unit":"tCO2e","data":[{"id":"6","code":"58a6560e-05e0-462d-9969-f4ecf46bb749","name":"Energy","total":1250}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** Horizontal bar chart: `data.data[].name` → category label; `data.data[].total` → bar value.

### 3. Trend of GHG Emissions per Sector

**Endpoint:** `/emissions-trend-by-sector`  
**Method:** `GET`  
**Purpose:** Returns one yearly series for each sector using saved reporting years.  
**Supported filters:** same emission filters as API 2  
**Example request:** `/emissions-trend-by-sector?startYear=2021&endYear=2026`

**Example response:**

```json
{"success":true,"message":"GHG sector trends retrieved","data":{"contextCode":"GHG_INVENTORY","contextName":"GHG Inventory","unit":"tCO2e","years":[2024,2025,2026],"series":[{"id":"6","code":"ENERGY","name":"Energy","values":[1000,null,1250]}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** `data.years` is the shared X-axis. Each `data.series[]` becomes one line; align `series.values[index]` with `years[index]`. Preserve `null` as a gap.

### 4. Emissions by Gas Type per Sector

**Endpoint:** `/emissions-by-gas-sector`  
**Method:** `GET`  
**Purpose:** Returns sector/gas cells for a stacked sector chart.  
**Supported filters:** same emission filters as API 2  
**Example request:** `/emissions-by-gas-sector?year=2026&gasType=CO2`

**Example response:**

```json
{"success":true,"message":"GHG emissions by gas and sector retrieved","data":{"unit":"tCO2e","data":[{"sectorId":"6","sectorCode":"ENERGY","sectorName":"Energy","gasTypeId":"10509","gasTypeCode":"CO2","gasTypeName":"CO2","total":1250}],"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":"CO2","regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** Group rows by `sectorName`; create stack keys from `gasTypeName`; use `total` for each stack cell.

### 5. Trend of Emissions by GHG Type

**Endpoint:** `/emissions-trend-by-gas`  
**Method:** `GET`  
**Purpose:** Returns yearly CO2, CH4, N2O, HFCs, and any other actually reported gas series.  
**Supported filters:** same emission filters as API 2  
**Example request:** `/emissions-trend-by-gas?startYear=2021&endYear=2026`

**Example response:**

```json
{"success":true,"message":"GHG gas trends retrieved","data":{"contextCode":"GHG_INVENTORY","contextName":"GHG Inventory","unit":"tCO2e","years":[2024,2025,2026],"series":[{"id":"10509","code":"CO2","name":"CO2","values":[1000,null,1250]}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** Multi-line chart using `years`, `series[].name`, and aligned `series[].values`.

### 6. Energy Emissions per Contributor

**Endpoint:** `/sectors/energy/contributors`  
**Method:** `GET`  
**Purpose:** Groups Energy emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/energy/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"ENERGY","contextName":"Energy","unit":"tCO2e","data":[{"id":"10552","code":"85cfd72c-15da-426c-bee9-fa235fa13de1","name":"Energy - Public","total":1250}],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Lollipop/bar chart: `data.data[].name` → label; `data.data[].total` → value.

### 7. Energy Contributor Emission Trends

**Endpoint:** `/sectors/energy/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Energy emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/energy/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"ENERGY","contextName":"Energy","unit":"tCO2e","years":[2025,2026],"series":[{"id":"10552","code":"ENERGY_PUBLIC","name":"Energy - Public","values":[null,1250]}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Line chart with `years` on X and `series[].values` on Y.

### 8. Industrial Process Emissions per Contributor

**Endpoint:** `/sectors/industrial-process/contributors`  
**Method:** `GET`  
**Purpose:** Groups Industrial Process and Product Use emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/industrial-process/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"INDUSTRIAL_PROCESS_AND_PRODUCT_USE","contextName":"Industrial Process and Product Use","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Contributor comparison bar chart using `data[].name` and `data[].total`; show no-data UI when `data.length === 0`.

### 9. Industrial Process Contributor Trends

**Endpoint:** `/sectors/industrial-process/trends`  
**Method:** `GET`  
**Purpose:** Returns yearly Industrial Process emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/industrial-process/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"INDUSTRIAL_PROCESS_AND_PRODUCT_USE","contextName":"Industrial Process and Product Use","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Multi-line chart; render no-data UI when `series.length === 0`.

### 10. Mining Emissions per Contributor

**Endpoint:** `/sectors/mining/contributors`  
**Method:** `GET`  
**Purpose:** Groups Mining Industry emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/mining/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"MINING","contextName":"Mining Industry","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Donut chart: `name` → slice label; `total` → slice value.

### 11. Mining Contributor Emission Trends

**Endpoint:** `/sectors/mining/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Mining emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/mining/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"MINING","contextName":"Mining Industry","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Line chart using `years`, `series[].name`, and `series[].values`.

### 12. Agriculture Emissions per Contributor

**Endpoint:** `/sectors/agriculture/contributors`  
**Method:** `GET`  
**Purpose:** Groups Agriculture emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/agriculture/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"AGRICULTURE","contextName":"Agriculture","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Horizontal bar chart using `name` and `total`.

### 13. Agriculture Contributor Emission Trends

**Endpoint:** `/sectors/agriculture/trends`  
**Method:** `GET`  
**Purpose:** Returns yearly Agriculture emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/agriculture/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"AGRICULTURE","contextName":"Agriculture","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Area/line chart; `null` means no observation and should remain a gap.

### 14. Livestock Emissions per Contributor

**Endpoint:** `/sectors/livestock/contributors`  
**Method:** `GET`  
**Purpose:** Groups Livestock emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/livestock/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"LIVESTOCK","contextName":"Livestock","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Horizontal bar chart using `name` and `total`.

### 15. Livestock Contributor Emission Trends

**Endpoint:** `/sectors/livestock/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Livestock emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/livestock/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"LIVESTOCK","contextName":"Livestock","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Stepped/area chart using the aligned trend arrays.

### 16. Human Settlement Emissions per Contributor

**Endpoint:** `/sectors/human-settlement/contributors`  
**Method:** `GET`  
**Purpose:** Groups Human Settlement emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/human-settlement/contributors?year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"HUMAN_SETTLEMENT","contextName":"Human Settlement","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Lollipop/bar chart using `name` and `total`.

### 17. Human Settlement Contributor Trends

**Endpoint:** `/sectors/human-settlement/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Human Settlement emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `contributorId`, `gasType`  
**Example request:** `/sectors/human-settlement/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"HUMAN_SETTLEMENT","contextName":"Human Settlement","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Area/line chart using `years` and `series`.

### 18. Waste Emissions by Subcategory

**Endpoint:** `/sectors/waste/subcategories`  
**Method:** `GET`  
**Purpose:** Groups eligible Waste emissions by the saved Waste subcategory relationship.  
**Supported filters:** `year`, `startYear`, `endYear`, `wasteSubcategoryId`, `contributorId`, `gasType`  
**Example request:** `/sectors/waste/subcategories?year=2026`  
**Example response:** `{"success":true,"message":"Waste emissions by subcategory retrieved","data":{"contextCode":"WASTE","contextName":"Waste","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Donut chart using `data[].name` for labels and `data[].total` for values.

### 19. Waste Emission Trends by Subcategory

**Endpoint:** `/sectors/waste/subcategories/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Waste emissions grouped by subcategory.  
**Supported filters:** `year`, `startYear`, `endYear`, `wasteSubcategoryId`, `contributorId`, `gasType`  
**Example request:** `/sectors/waste/subcategories/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Waste subcategory trends retrieved","data":{"contextCode":"WASTE","contextName":"Waste","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Multi-line chart: `years` → X-axis; one `series` entry per subcategory.

### 20. Waste Emissions by Contributor

**Endpoint:** `/sectors/waste/contributors`  
**Method:** `GET`  
**Purpose:** Groups Waste emissions by the actual contributor selected under each subcategory.  
**Supported filters:** `year`, `startYear`, `endYear`, `wasteSubcategoryId`, `contributorId`, `gasType`  
**Example request:** `/sectors/waste/contributors?wasteSubcategoryId=12&year=2026`  
**Example response:** `{"success":true,"message":"Contributor emissions retrieved","data":{"contextCode":"WASTE","contextName":"Waste","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":12,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Lollipop chart using `name` and `total`.

### 21. Waste Contributor Trends

**Endpoint:** `/sectors/waste/contributors/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Waste emissions by contributor.  
**Supported filters:** `year`, `startYear`, `endYear`, `wasteSubcategoryId`, `contributorId`, `gasType`  
**Example request:** `/sectors/waste/contributors/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Contributor emission trends retrieved","data":{"contextCode":"WASTE","contextName":"Waste","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Multi-line chart using aligned `years` and `series[].values`.

### 22. Transport Emissions by Mode

**Endpoint:** `/sectors/transport/modes`  
**Method:** `GET`  
**Purpose:** Groups Transport emissions by the saved means of transport.  
**Supported filters:** `year`, `startYear`, `endYear`, `transportMode`, `contributorId`, `fuelCategoryId`, `gasType`  
**Example request:** `/sectors/transport/modes?year=2026&transportMode=ROAD`  
**Example response:** `{"success":true,"message":"Transport emissions by mode retrieved","data":{"contextCode":"TRANSPORT","contextName":"Transport","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":"ROAD","fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Polar/circular bar or category bar chart using `name` and `total`. Current local database has no Transport category records, so an empty result is expected until records are submitted.

### 23. Transport Mode Trends

**Endpoint:** `/sectors/transport/modes/trends`  
**Method:** `GET`  
**Purpose:** Returns annual Transport emissions by mode.  
**Supported filters:** `year`, `startYear`, `endYear`, `transportMode`, `contributorId`, `fuelCategoryId`, `gasType`  
**Example request:** `/sectors/transport/modes/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Transport mode trends retrieved","data":{"contextCode":"TRANSPORT","contextName":"Transport","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Area/line chart. Current local database has no Transport category history.

### 24. Transport Emissions by Fuel Category

**Endpoint:** `/sectors/transport/fuels`  
**Method:** `GET`  
**Purpose:** Aggregates only fuel-specific saved amounts. It never assigns a contributor total to every selected fuel.  
**Supported filters:** `year`, `startYear`, `endYear`, `transportMode`, `contributorId`, `fuelCategoryId`, `gasType`  
**Example request:** `/sectors/transport/fuels?year=2026&fuelCategoryId=21`  
**Example response:** `{"success":true,"message":"Transport emissions by fuel retrieved","data":{"contextCode":"TRANSPORT","contextName":"Transport","unit":"tCO2e","data":[],"unclassifiedAmount":0,"filters":{"year":2026,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":21,"carbonSinkId":null}}}`  
**Frontend usage:** Horizontal bar chart using `name` and `total`. Current local database contains no fuel allocations, so this endpoint is expected to be empty until fuel-specific records are saved.

### 25. Transport Fuel Category Trends

**Endpoint:** `/sectors/transport/fuels/trends`  
**Method:** `GET`  
**Purpose:** Returns annual fuel-specific Transport emissions.  
**Supported filters:** `year`, `startYear`, `endYear`, `transportMode`, `contributorId`, `fuelCategoryId`, `gasType`  
**Example request:** `/sectors/transport/fuels/trends?startYear=2021&endYear=2026`  
**Example response:** `{"success":true,"message":"Transport fuel trends retrieved","data":{"contextCode":"TRANSPORT","contextName":"Transport","unit":"tCO2e","years":[],"series":[],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2021,"endYear":2026,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}`  
**Frontend usage:** Multi-line chart. Current local database contains no historical fuel allocations.

### 26. Amount of GHGs Removed by Carbon Sink

**Endpoint:** `/sectors/land-use/removals-by-sink`  
**Method:** `GET`  
**Purpose:** Groups authoritative Amount of GHGs Removed rows by saved carbon-sink/restoration type.  
**Supported filters:** `year`, `startYear`, `endYear`, `regionId`, `districtId`, `carbonSinkId`, `gasType`  
**Example request:** `/sectors/land-use/removals-by-sink?carbonSinkId=102`

**Example response:**

```json
{"success":true,"message":"GHG removals by carbon sink retrieved","data":{"contextCode":"LAND_USE_CHANGE","contextName":"Land Use Change","unit":"tCO2e","data":[{"id":"102","code":"102","name":"Afforestation","total":1012000}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":102}}}
```

**Frontend usage:** Horizontal bar chart using `name` and `total`.

### 27. Trends of GHGs Removed by Carbon Sink

**Endpoint:** `/sectors/land-use/removals-trend`  
**Method:** `GET`  
**Purpose:** Returns annual greenhouse-gas removals by carbon sink.  
**Supported filters:** `year`, `startYear`, `endYear`, `regionId`, `districtId`, `carbonSinkId`, `gasType`  
**Example request:** `/sectors/land-use/removals-trend?startYear=2020&endYear=2023`

**Example response:**

```json
{"success":true,"message":"GHG removal trends retrieved","data":{"contextCode":"LAND_USE_CHANGE","contextName":"Land Use Change","unit":"tCO2e","years":[2020,2021,2022,2023],"series":[{"id":"102","code":"102","name":"Afforestation","values":[210400,458900,null,null]}],"unclassifiedAmount":0,"filters":{"year":null,"startYear":2020,"endYear":2023,"sectorId":null,"contributorId":null,"gasType":null,"regionId":null,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** Area chart using `years` and one `series` per carbon sink.

### 28. Regional Carbon Removal Heatmap

**Endpoint:** `/sectors/land-use/removals-by-region`  
**Method:** `GET`  
**Purpose:** Returns region/carbon-sink cells plus region and sink totals for a heatmap.  
**Supported filters:** `year`, `startYear`, `endYear`, `regionId`, `districtId`, `carbonSinkId`, `gasType`  
**Example request:** `/sectors/land-use/removals-by-region?regionId=9`

**Example response:**

```json
{"success":true,"message":"Regional GHG removals retrieved","data":{"unit":"tCO2e","regions":[{"id":"9","code":"1","name":"Arusha","total":1975200}],"carbonSinkTypes":[{"id":"111","code":"111","name":"Grassland Reseeding","total":412800}],"data":[{"regionId":"9","regionCode":"1","regionName":"Arusha","carbonSinkId":"111","carbonSinkCode":"111","carbonSinkName":"Grassland Reseeding","amountRemoved":412800}],"filters":{"year":null,"startYear":null,"endYear":null,"sectorId":null,"contributorId":null,"gasType":null,"regionId":9,"districtId":null,"wasteSubcategoryId":null,"transportMode":null,"fuelCategoryId":null,"carbonSinkId":null}}}
```

**Frontend usage:** `regions[].name` supplies region labels, `carbonSinkTypes[].name` supplies sink labels/legend, and `data[].amountRemoved` supplies the heatmap cell value. Match cells by `regionId` and `carbonSinkId`.

## GHG Dashboard Chart-to-API Mapping

| Chart | API | Recommended chart type | X-axis / label property | Y-axis / value property |
|---|---|---|---|---|
| Amount of GHG Emissions per Sector | `/emissions-by-sector` | Horizontal bar | `data.data[].name` | `data.data[].total` |
| Trend of GHG Emissions per Sector | `/emissions-trend-by-sector` | Multi-line | `data.years[]` | `data.series[].values[]` |
| Emissions by Gas Type per Sector | `/emissions-by-gas-sector` | Stacked bar | `data.data[].sectorName` | `data.data[].total`, stack by `gasTypeName` |
| Trend of Emissions by GHG Type | `/emissions-trend-by-gas` | Multi-line | `data.years[]` | `data.series[].values[]` |
| Energy: Emissions per Contributor | `/sectors/energy/contributors` | Lollipop/bar | `data.data[].name` | `data.data[].total` |
| Energy: Contributor Emission Trends | `/sectors/energy/trends` | Line | `data.years[]` | `data.series[].values[]` |
| Industrial Process: Emissions per Contributor | `/sectors/industrial-process/contributors` | Bar | `data.data[].name` | `data.data[].total` |
| Industrial Process: Emission Trends | `/sectors/industrial-process/trends` | Multi-line | `data.years[]` | `data.series[].values[]` |
| Mining: Emissions per Contributor | `/sectors/mining/contributors` | Donut | `data.data[].name` | `data.data[].total` |
| Mining: Contributor Emission Trends | `/sectors/mining/trends` | Line | `data.years[]` | `data.series[].values[]` |
| Agriculture: Emissions per Contributor | `/sectors/agriculture/contributors` | Horizontal bar | `data.data[].name` | `data.data[].total` |
| Agriculture: Contributor Emission Trends | `/sectors/agriculture/trends` | Area/line | `data.years[]` | `data.series[].values[]` |
| Livestock: Emissions per Contributor | `/sectors/livestock/contributors` | Horizontal bar | `data.data[].name` | `data.data[].total` |
| Livestock: Contributor Emission Trends | `/sectors/livestock/trends` | Stepped area | `data.years[]` | `data.series[].values[]` |
| Human Settlement: Emissions per Contributor | `/sectors/human-settlement/contributors` | Lollipop | `data.data[].name` | `data.data[].total` |
| Human Settlement: Contributor Trends | `/sectors/human-settlement/trends` | Area/line | `data.years[]` | `data.series[].values[]` |
| Waste: Emissions by Subcategory | `/sectors/waste/subcategories` | Donut | `data.data[].name` | `data.data[].total` |
| Waste: Subcategory Trends | `/sectors/waste/subcategories/trends` | Line | `data.years[]` | `data.series[].values[]` |
| Waste: Emissions by Contributor | `/sectors/waste/contributors` | Lollipop | `data.data[].name` | `data.data[].total` |
| Waste: Contributor Trends | `/sectors/waste/contributors/trends` | Line | `data.years[]` | `data.series[].values[]` |
| Transport: Emissions by Mode | `/sectors/transport/modes` | Polar/category bar | `data.data[].name` | `data.data[].total` |
| Transport: Mode Trends | `/sectors/transport/modes/trends` | Area | `data.years[]` | `data.series[].values[]` |
| Transport: Emissions by Fuel | `/sectors/transport/fuels` | Horizontal bar | `data.data[].name` | `data.data[].total` |
| Transport: Fuel Trends | `/sectors/transport/fuels/trends` | Multi-line | `data.years[]` | `data.series[].values[]` |
| Land Use: Removals by Carbon Sink | `/sectors/land-use/removals-by-sink` | Horizontal bar | `data.data[].name` | `data.data[].total` |
| Land Use: Carbon Sink Trends | `/sectors/land-use/removals-trend` | Area | `data.years[]` | `data.series[].values[]` |
| Land Use: Regional Removal Heatmap | `/sectors/land-use/removals-by-region` | Heatmap | `regionName` / `carbonSinkName` | `data.data[].amountRemoved` |

