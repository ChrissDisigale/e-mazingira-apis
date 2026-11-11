# Climate Change Mitigation Measures API

> **IEMIS · Backend API and frontend integration guide**  
> Covers mitigation measure configuration, dynamic forms, record CRUD, validation, and deployment.



---

## Overview

The IEMIS Mitigation Measures module uses database master data and JSONB field schemas:

```text
MITIGATION_MEASURE → Sector → Sector Measure → Action Code → Dynamic Schema → Record
```

It supports Energy, Transport, Forest/Forestry, Waste, Industry (currently named `Mining Industry` in IEMIS master data), and Livestock. Records retain their action code, Sector Measure relationship, organization scope, Council, Region, approval state, and code-specific JSONB values.

New and updated mitigation records require `coverage`. Historical records without it remain readable.

## Base URLs and authentication

Frontend base: `${NEXT_PUBLIC_API_BASE_URL_80}` (gateway origin, without appending the service path twice).

The gateway prefixes climate endpoints with `/climate-change` and environmental master-data endpoints with `/env`. The controller-relative paths omit those two gateway service prefixes.

All endpoints use the existing JWT resource-server security and global permission aspect.

```http
Authorization: Bearer <ACCESS_TOKEN>
Content-Type: application/json
Accept: application/json
```

Standard response:

```json
{
  "success": true,
  "message": "Data saved successfully",
  "data": {}
}
```

## Dropdown dependency flow

1. Fetch sectors.
2. After selecting a sector, fetch its `MITIGATION_MEASURE` Sector Measures.
3. After selecting a Sector Measure, fetch its linked actions.
4. After selecting an action code, fetch its dynamic schema.
5. Render and validate the returned fields.
6. Submit `category`, `sectorMeasureCategoryUid`, and `values`.

When a parent selection changes, clear every child selection and its values.

## API reference

| Purpose | Method | Actual path |
|---|---:|---|
| Sectors | GET | `/env/api/v1/carbon/sectors` |
| Mitigation Sector Measures | GET | `/climate-change/api/v1/config/sector-measures/{sectorId}/MITIGATION_MEASURE` |
| Actions for Sector Measure | GET | `/climate-change/api/v1/config/sector-measure-action/{sectorMeasureUid}/actions` |
| Dynamic schema by code | GET | `/climate-change/api/v1/climate-change/sector-measure-fields/{code}` |
| Coverage options | GET | `/climate-change/api/v1/climate-change/sector-measure-fields/coverage-options` |
| Ownership options | GET | `/climate-change/api/v1/climate-change/sector-measure-fields/owner-ship-types` |
| Regions | GET | `/env/api/v1/config/regions` |
| Districts | GET | `/env/api/v1/config/regions/{regionId}/districts` |
| Councils for District | GET | `/env/api/v1/config/regions/{regionId}/districts/{districtId}/councils` |
| Councils for Region | GET | `/env/api/v1/config/regions/councils/{regionId}` |
| Create record | POST | `/climate-change/api/v1/climate-change/measure-action-field-data-request` |
| Record details/edit data | GET | `/climate-change/api/v1/climate-change/measure-action-field-data-request/{uid}` |
| Update record | PUT | `/climate-change/api/v1/climate-change/measure-action-field-data-request/{uid}` |
| List/filter records | GET | `/climate-change/api/v1/climate-change/measure-action-field-data-request` |
| Delete record | DELETE | `/climate-change/api/v1/climate-change/measure-action-field-data-request/{uid}` |

### List filters

`page`, `size`, `searchKeyword`, `measureType`, `sectorId`, `sectorMeasureUid`, `category`, `coverage`, `regionId`, `councilId`, `year`, and `approvalStatus`. Existing frontend aliases `search` and `adaptiveMitigationType` remain supported.

Example:

```http
GET /climate-change/api/v1/climate-change/measure-action-field-data-request?measureType=MITIGATION_MEASURE&coverage=TANZANIA_MAINLAND&sectorId=6&year=2026&page=0&size=20
```

## Coverage

```json
[
  { "value": "TANZANIA_MAINLAND", "label": "Tanzania Mainland" },
  { "value": "ZANZIBAR", "label": "Zanzibar" }
]
```

Coverage is stored at `values.coverage`. New and updated mitigation records reject missing or unsupported values. When `Region.coverage` is configured, the backend also rejects a Region/Coverage mismatch.

The inspected database currently has no `Region.coverage` classifications. Administrators must classify Region master data as `MAINLAND` or `ZANZIBAR` before strict geographic matching can occur. The backend deliberately does not invent that classification.

## Dynamic schema properties

| Property | Meaning |
|---|---|
| `name` | Stable JSON property submitted in `values` |
| `label` | User-facing field label |
| `type` | `string`, `number`, `textarea`, `select`, `multiselect`, or `array` |
| `dataType` | Expected serialized type |
| `required` | Whether a value is mandatory |
| `sourceApi` | API used to populate a select |
| `options` | Inline `{value,label}` objects or primitive allowed values |
| `min`, `max` | Numeric bounds validated by the backend |
| `unit` | Display/reporting unit where defined |
| `description` | Additional field guidance |
| `visibleWhen` | Map of sibling values required for conditional validation |
| `arrayFields` | Nested schema for repeatable groups |

The existing frontend renderer must map `options`, `unit`, `description`, `min`, `max`, and `visibleWhen` if it does not already support them. Backend validation is authoritative.

### Schema response example

`GET /climate-change/api/v1/climate-change/sector-measure-fields/POWER_PRODUCTION`

```json
{
  "success": true,
  "message": "Fetched all Measure Action Fields of POWER_PRODUCTION",
  "data": {
    "category": "POWER_PRODUCTION",
    "data": {
      "category": "POWER_PRODUCTION",
      "fields": [
        {"name":"coverage","label":"Coverage","type":"select","dataType":"string","required":true,"sourceApi":"/climate-change/api/v1/climate-change/sector-measure-fields/coverage-options"},
        {"name":"regionId","label":"Region","type":"select","dataType":"number","required":true,"sourceApi":"/env/api/v1/config/regions"},
        {"name":"councilId","label":"Council","type":"select","dataType":"number","required":true,"sourceApi":"/env/api/v1/config/regions/councils/{regionId}"},
        {"name":"year","label":"Year of Establishment","type":"number","dataType":"number","required":true,"min":1900,"max":2100},
        {"name":"name","label":"Name","type":"string","dataType":"string","required":true},
        {"name":"installedCapacity","label":"Installed Capacity (MW)","type":"number","dataType":"number","required":true,"min":0}
      ]
    }
  }
}
```

The response can contain additional ownership, gas-reduction, or legacy optional fields. Always render the complete returned `fields` array rather than copying this shortened example.

## Common field reference

Every migrated mitigation action includes these fields:

| Field Name | Label | Type | Required | Unit | Description |
|---|---|---:|---:|---|---|
| `coverage` | Coverage | select | Yes | — | `TANZANIA_MAINLAND` or `ZANZIBAR` |
| `regionId` | Region | select/number | Yes | — | Existing Region ID |
| `councilId` | Council | select/number | Yes | — | Existing Council ID; District is derived from Council |
| `year` | Activity-specific year label | number | Yes | year | 1900–2100 |
| `quantityReducedCO2` | Quantity Reduced (CO₂) | decimal | Yes where applicable | approved reporting unit | Stored separately; no automatic CO₂e conversion |
| `quantityReducedCH4` | Quantity Reduced (CH₄) | decimal | Yes where applicable | approved reporting unit | Stored separately |
| `quantityReducedN2O` | Quantity Reduced (N₂O) | decimal | Yes where applicable | approved reporting unit | Stored separately |
| `quantityReducedHFCs` | Quantity Reduced (HFCs) | decimal | Yes where applicable | approved reporting unit | Stored separately |

Most facility actions also include `name` and `ownership`. Fetch the schema for the final authority on required fields.

## Code-specific field reference

The following list excludes the common fields above.

| Sector | Code | Code-specific fields |
|---|---|---|
| Energy | `IMPROVE_CLEAN_POWER_INTERCONNECTION_WITH_NEIGHBOURING_COUNTRIES` | `receivedCapacity` (MW) |
| Energy | `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | repeatable `technologies[]`: `technologyType`, `name`, `installedCapacity`, `productionCapacity`, `ownership`, four gas reductions |
| Energy | `POWER_PRODUCTION` | `installedCapacity` (MW), `naturalGasConsumed` (MMscfd) |
| Energy | `COOKING` | `numberOfHouseholds`, `numberOfInstitutions`, `numberOfBeneficiaries`, `naturalGasConsumed` (MMscfd) |
| Energy | `TRANSPORTATION` | `installedCapacity`, `vehiclesServed`, `naturalGasConsumed` (MMscfd) |
| Energy | `MICRO_RENEWABLE_GENERATION` | `installedCapacity`, `productionCapacity` (MW) |
| Energy | `MINI_GRID_RENEWABLE_GENERATION` | `installedCapacity`, `productionCapacity` (MW) |
| Transport | `RAILWAY` | `transportType`, `capacity`, `networkDistance`, `coverageFrom`, `coverageTo` |
| Transport | `ROAD_INFRASTRUCTURE` | `transportType`, optional `capacity`, `networkDistance`, `coverageFrom`, `coverageTo` |
| Transport | `BUS_RAPID_TRANSIT` | `transportType`, `vehicleNumber`, `populationServed`, `energySource`, `energyConsumptionPerDay` |
| Transport | `LOCOMOTIVES` | `transportType`, `vehicleNumber`, `capacity`, `energySource`, `energyConsumptionPerDay` |
| Transport | `PRIVATE_VEHICLES` | `transportType`, `vehicleNumber`, `energySource`, `energyConsumptionPerDay` |
| Transport | `CYCLING` | `transportType`, optional `capacity`, `populationServed`, `networkDistance`, `coverageFrom`, `coverageTo` |
| Transport | `PEDESTRIAN` | `transportType`, `populationServed`, `networkDistance`, `coverageFrom`, `coverageTo` |
| Forest | `COMMUNITY_BASED_FOREST_MANAGEMENT` | `coverageArea`, `managementType`, `numberOfBeneficiaries` |
| Forest | `JOINT_FOREST_MANAGEMENT` | `coverageArea`, `managementType`, `numberOfBeneficiaries` |
| Forest | `PLANTED_FORESTS_AND_WOODLOTS` | `coverageArea`, `numberOfTrees`, `managementType`, `numberOfBeneficiaries` |
| Forest | `AGROFOREST` | `coverageArea`, `numberOfTrees`, `managementType`, `numberOfBeneficiaries` |
| Forest | `NATURAL_REGENERATION` | `coverageArea`, optional `numberOfTrees`, `managementType`, `numberOfBeneficiaries` |
| Forest | `MANGROVES_RESTORATION` | `coverageArea`, `numberOfTrees`, `managementType`, `numberOfBeneficiaries` |
| Forest | `AGROSILVIPASTORAL` | `coverageArea`, optional `numberOfTrees`, `managementType`, `numberOfBeneficiaries` |
| Waste | `RECYCLING` | `industryName`, `wasteType`, `installedCapacity`, `productionCapacity`, `numberOfBeneficiaries` |
| Waste | `SORTING` | `industryName`, `wasteType`, `sortingCapacity`, `numberOfBeneficiaries` |
| Waste | `LANDFILL` | `industryName`, `wasteType`, `landfillCapacity`, `organicWasteReceivingCapacity`, `landfillSize`, `numberOfBeneficiaries` |
| Waste | `BIOGAS_COW_DUNG` | `installedCapacity` (m³), `productionCapacityPerDay` (m³), `numberOfBeneficiaries` |
| Waste | `ALTERNATIVE_CHARCOAL_PRODUCTION` | `charcoalProductionMaterials`, `installedCapacity`, `productionCapacityPerDay`, `numberOfBeneficiaries` |
| Industry | `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | `industryName`, `installedCapacity`, `productionCapacity`, repeatable `energySources[]`: `source`, `consumption`, `unit`, `ownership`, four gas reductions |
| Livestock | `SILVICULTURE` | `activityType`, `coverageArea`, `treesCropsLivestockDetails`, `managementType`, `numberOfBeneficiaries` |
| Livestock | `AGRO_SILVOPASTURE` | same Livestock Management fields |
| Livestock | `SILVOPASTURE` | same Livestock Management fields |
| Livestock | `LAND_RANGE_RESTORATION` | same Livestock Management fields |
| Livestock | `FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS` | `coverageArea`, `livestockProductionCapacity`, `numberOfBeneficiaries` |
| Livestock | `FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` | `coverageArea`, `fodderProductionCapacity`, `numberOfBeneficiaries` |

## Create example

```http
POST /climate-change/api/v1/climate-change/measure-action-field-data-request
```

```json
{
  "category": "POWER_PRODUCTION",
  "sectorMeasureCategoryUid": "<USE-UID-RETURNED-BY-SECTOR-MEASURES-API>",
  "values": {
    "name": "Example Gas Power Facility",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 10,
    "year": 2026,
    "installedCapacity": 100.5,
    "naturalGasConsumed": 12.75,
    "ownership": "GOVERNMENT",
    "quantityReducedCO2": 25.5,
    "quantityReducedCH4": 2.3,
    "quantityReducedN2O": 0.5,
    "quantityReducedHFCs": 0
  }
}
```

Successful response data contains `id`, `uid`, `category`, `values`, `sectorMeasureCategory`, `currentStage`, `approvalStatus`, `createdDate`, `lastModifiedDate`, `createdBy`, and top-level `council`, `district`, and `region` display objects. The `values` object remains the exact editable dynamic payload, so a response can be safely reused for an update without stripping display-only properties.

### Record response example

```json
{
  "success": true,
  "message": "Data saved successfully",
  "data": {
    "id": 1201,
    "uid": "<GENERATED-RECORD-UID>",
    "category": "POWER_PRODUCTION",
    "values": {
      "name": "Example Gas Power Facility",
      "coverage": "TANZANIA_MAINLAND",
      "regionId": 1,
      "councilId": 10,
      "year": 2026,
      "installedCapacity": 100.5,
      "naturalGasConsumed": 12.75,
      "ownership": "GOVERNMENT",
      "quantityReducedCO2": 25.5,
      "quantityReducedCH4": 2.3,
      "quantityReducedN2O": 0.5,
      "quantityReducedHFCs": 0
    },
    "sectorMeasureCategory": {"name":"Use of Natural Gas","uid":"<SECTOR-MEASURE-UID>","adaptiveMitigationType":"MITIGATION_MEASURE"},
    "currentStage": null,
    "approvalStatus": "PENDING",
    "createdDate": "2026-11-11T03:00:00Z",
    "lastModifiedDate": "2026-11-11T03:00:00Z",
    "createdBy": "<AUTHENTICATED-USER>",
    "council": {"id":10,"name":"<COUNCIL-NAME>"},
    "district": {"id":5,"name":"<DISTRICT-NAME>"},
    "region": {"id":1,"name":"<REGION-NAME>"}
  }
}
```

### List response shape

The list endpoint returns the standard Spring `Page` inside `data`:

```json
{"success":true,"message":"Fetched all measure action data","data":{"content":[],"totalElements":0,"totalPages":0,"size":20,"number":0,"first":true,"last":true,"empty":true}}
```

An empty page means no records matched; it is not an error and is different from a record whose numeric values are explicitly zero.

### Error response shapes

```json
{"success":false,"message":"Coverage is required for Mitigation Measures","data":{"error":"Coverage is required for Mitigation Measures"}}
```

Schema/duplicate/field validation returns HTTP 400. A missing record or Council returns HTTP 404; unauthenticated and unauthorized calls return HTTP 401 and 403 respectively. The existing global workflow remains responsible for approval transitions; this enhancement does not add or bypass a separate approval endpoint.

## Update and edit

Load the record first:

```http
GET /climate-change/api/v1/climate-change/measure-action-field-data-request/{uid}
```

Fetch the schema for its returned `category`, populate the form using `values`, then submit the complete validated payload:

```http
PUT /climate-change/api/v1/climate-change/measure-action-field-data-request/{uid}
```

Approved and Reviewed records cannot be edited. The update excludes itself from duplicate detection and preserves its audit identity and workflow state.

## Duplicate rules

Candidates are found by code, year, and Council. If a record contains an identity field (`name`, `industryName`, `plantName`, `gasStationName`, `routeName`, or `activityName`), a conflict occurs only when that identity also matches. This permits multiple named facilities in the same Council/year. Records without an identity retain the original code/year/Council protection.

## Sector payload examples

Only code-specific portions are shortened below; include every common required field returned by the schema.

```jsonc
// Transport
{"category":"CYCLING","sectorMeasureCategoryUid":"<UID>","values":{"name":"City Cycle Route","coverage":"TANZANIA_MAINLAND","regionId":1,"councilId":10,"year":2026,"transportType":"CYCLING","populationServed":20000,"networkDistance":14.2,"coverageFrom":"A","coverageTo":"B","ownership":"GOVERNMENT","quantityReducedCO2":10,"quantityReducedCH4":0,"quantityReducedN2O":0,"quantityReducedHFCs":0}}
```

```jsonc
// Forestry
{"category":"NATURAL_REGENERATION","sectorMeasureCategoryUid":"<UID>","values":{"name":"Regeneration Area","coverage":"TANZANIA_MAINLAND","regionId":1,"councilId":10,"year":2026,"coverageArea":500,"managementType":"COMMUNITY","numberOfBeneficiaries":1200,"quantityReducedCO2":80,"quantityReducedCH4":1,"quantityReducedN2O":0.2,"quantityReducedHFCs":0}}
```

```jsonc
// Waste
{"category":"LANDFILL","sectorMeasureCategoryUid":"<UID>","values":{"coverage":"TANZANIA_MAINLAND","regionId":1,"councilId":10,"year":2026,"industryName":"Example Landfill","wasteType":"ORGANIC_WASTE","landfillCapacity":5000,"organicWasteReceivingCapacity":1200,"landfillSize":20,"numberOfBeneficiaries":50000,"ownership":"GOVERNMENT","quantityReducedCO2":20,"quantityReducedCH4":30,"quantityReducedN2O":0,"quantityReducedHFCs":0}}
```

```jsonc
// Industry
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"<UID>","values":{"industryName":"Example Industry","coverage":"TANZANIA_MAINLAND","regionId":1,"councilId":10,"year":2026,"installedCapacity":100,"productionCapacity":80,"energySources":[{"source":"HYDROPOWER","consumption":120,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":10,"quantityReducedCH4":2,"quantityReducedN2O":1,"quantityReducedHFCs":0}]}}
```

```jsonc
// Livestock
{"category":"FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS","sectorMeasureCategoryUid":"<UID>","values":{"name":"Example Farm","coverage":"TANZANIA_MAINLAND","regionId":1,"councilId":10,"year":2026,"coverageArea":75,"livestockProductionCapacity":400,"numberOfBeneficiaries":90,"ownership":"COMMUNITY","quantityReducedCO2":12,"quantityReducedCH4":5,"quantityReducedN2O":1,"quantityReducedHFCs":0}}
```

## cURL

```bash
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "$API_BASE_URL/climate-change/api/v1/climate-change/sector-measure-fields/coverage-options"
```

```bash
curl -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-Type: application/json" \
  --data @mitigation-record.json \
  "$API_BASE_URL/climate-change/api/v1/climate-change/measure-action-field-data-request"
```

## Errors and troubleshooting

| Problem | Meaning/action |
|---|---|
| `No schema defined for category ...` | Apply the mitigation schema migration and verify the code |
| `Code ... does not belong to Sector Measure ...` | Refresh actions and use the action linked to the selected measure |
| `Coverage is required` | Include one of the two exact Coverage values |
| `Selected Region does not belong to ...` | Correct Coverage or Region master-data classification |
| `Selected Council does not belong to the selected Region` | Clear stale Council after Region changes |
| `Duplicate mitigation record ...` | Change the facility identity or confirm this is an existing report |
| `Invalid field ...` | Build `values` only from the selected code schema |
| `... must be at least 0` | Capacities and reductions cannot be negative |
| HTTP 401/403 | Check JWT and generated controller-method permissions |

## Database deployment

Migration files:

- `src/main/resources/db/changelog/2026-11-11-mitigation-measure-schemas.yaml`
- `src/main/resources/db/changelog/2026-11-11-mitigation-measure-schemas.sql`

The SQL is idempotent and was validated inside a rollback transaction. This project currently has `spring.liquibase.enabled=false`; deployment must run the changelog through the environment’s normal migration process or explicitly enable Liquibase. Do not run the SQL repeatedly by hand as a substitute for migration tracking.

---

## Ready-to-use test payloads

The five examples below are based on the supplied migration configuration and its local test IDs. **These IDs and UIDs are environment-specific**: use values returned by your own lookup endpoints if they differ. Apply the migration before testing.

| Test location | Example value |
|---|---|
| Region | Dar es Salaam |
| `regionId` | `1` |
| Council | Temeke Municipal |
| `councilId` | `1` |

**Request**

```http
POST /climate-change/api/v1/climate-change/measure-action-field-data-request
Authorization: Bearer <ACCESS_TOKEN>
Content-Type: application/json
```

### 1. Energy — Power Production

```json
{
  "category": "POWER_PRODUCTION",
  "sectorMeasureCategoryUid": "48c8a872-d955-f3cb-28bb-570e1242c1d9",
  "values": {
    "name": "Temeke Natural Gas Power Plant",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 1,
    "year": 2026,
    "installedCapacity": 150.5,
    "naturalGasConsumed": 23.75,
    "ownership": "GOVERNMENT",
    "quantityReducedCO2": 125.5,
    "quantityReducedCH4": 4.2,
    "quantityReducedN2O": 1.1,
    "quantityReducedHFCs": 0
  }
}
```

### 2. Transport — Bus Rapid Transit

```json
{
  "category": "BUS_RAPID_TRANSIT",
  "sectorMeasureCategoryUid": "0cd77aee-8abe-4fbb-bf24-9b7f1cf3fb45",
  "values": {
    "name": "Temeke Electric BRT Project",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 1,
    "year": 2026,
    "transportType": "BRT",
    "vehicleNumber": "BRT-EV-001",
    "populationServed": 85000,
    "energySource": "ELECTRICITY",
    "energyConsumptionPerDay": 420.5,
    "ownership": "GOVERNMENT",
    "quantityReducedCO2": 250.75,
    "quantityReducedCH4": 3.4,
    "quantityReducedN2O": 0.8,
    "quantityReducedHFCs": 0
  }
}
```

### 3. Forestry — Natural Regeneration

```json
{
  "category": "NATURAL_REGENERATION",
  "sectorMeasureCategoryUid": "d6a4e848-e69c-ef93-55ee-39f12a41bf01",
  "values": {
    "name": "Temeke Natural Forest Regeneration Programme",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 1,
    "year": 2026,
    "coverageArea": 750.5,
    "numberOfTrees": 125000,
    "managementType": "COMMUNITY",
    "numberOfBeneficiaries": 3200,
    "quantityReducedCO2": 540.5,
    "quantityReducedCH4": 8.25,
    "quantityReducedN2O": 2.15,
    "quantityReducedHFCs": 0
  }
}
```

### 4. Waste — Landfill

```json
{
  "category": "LANDFILL",
  "sectorMeasureCategoryUid": "3a6a7608-5fd3-da69-6ff3-545209a33974",
  "values": {
    "industryName": "Temeke Municipal Organic Waste Landfill",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 1,
    "year": 2026,
    "wasteType": "ORGANIC_WASTE",
    "landfillCapacity": 12000,
    "organicWasteReceivingCapacity": 3500,
    "landfillSize": 42.75,
    "numberOfBeneficiaries": 150000,
    "ownership": "GOVERNMENT",
    "quantityReducedCO2": 180.5,
    "quantityReducedCH4": 320.75,
    "quantityReducedN2O": 1.4,
    "quantityReducedHFCs": 0
  }
}
```

### 5. Livestock — Improved Livestock Breeds

```json
{
  "category": "FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS",
  "sectorMeasureCategoryUid": "fa67ddaf-96a9-0bda-cddd-9711f1242431",
  "values": {
    "name": "Temeke Improved Dairy Breeds Farm",
    "coverage": "TANZANIA_MAINLAND",
    "regionId": 1,
    "councilId": 1,
    "year": 2026,
    "coverageArea": 85.5,
    "livestockProductionCapacity": 650,
    "numberOfBeneficiaries": 240,
    "ownership": "PRIVATE",
    "quantityReducedCO2": 45.5,
    "quantityReducedCH4": 80.25,
    "quantityReducedN2O": 12.4,
    "quantityReducedHFCs": 0
  }
}
```

**Test with cURL:**

```bash
curl -X POST \
  "$API_BASE_URL/climate-change/api/v1/climate-change/measure-action-field-data-request" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  --data @payload.json
```
