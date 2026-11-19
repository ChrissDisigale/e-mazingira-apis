# Climate Change Mitigation Sample Payloads by Sector

This document contains six complete sample payloads for each mitigation sector
(36 payloads total) and the complete list of configured mitigation category
values. Replace every `REPLACE_WITH_*_UID`, `regionId`, and `councilId` with
valid related values from the target environment.

## API and lookup flow

```http
POST /climate-change/api/v1/climate-change/measure-action-field-data-request
Content-Type: application/json
Authorization: Bearer <ACCESS_TOKEN>
```

1. Fetch sectors: `GET /api/v1/config/sector-measures/sectors/MITIGATION_MEASURE`.
2. Fetch measures: `GET /api/v1/config/sector-measures/{sectorId}/MITIGATION_MEASURE`.
3. Copy the selected measure's `uuid` into `sectorMeasureCategoryUid`.
4. Verify fields: `GET /api/v1/climate-change/sector-measure-fields/{category}`.

The UID belongs to the Sector Measure, not the child action. Location IDs below
are illustrative and must identify a Council belonging to the supplied Region
and Coverage.

## All mitigation category values

This table lists only the current categories that frontend and API clients
should use for new submissions.

| Sector | Category value | Contract status | Nested subtype values |
|---|---|---|---|
| Energy | `IMPROVE_CLEAN_POWER_INTERCONNECTION_WITH_NEIGHBOURING_COUNTRIES` | Current flat | — |
| Energy | `CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION` | Preferred parent | `HYDROPOWER`, `GEOTHERMAL`, `SOLAR_POWER`, `WIND`, `BIOENERGY`, `HYDROGEN_GAS` |
| Energy | `USE_OF_NATURAL_GAS` | Preferred parent | `POWER_PRODUCTION`, `COOKING`, `TRANSPORTATION` |
| Energy | `CLIMATE_SMART_RURAL_ELECTRIFICATION` | Preferred parent | `MICRO_RENEWABLE_GENERATION`, `MINI_GRID_RENEWABLE_GENERATION` |
| Transport | `LOW_EMISSION_TRANSPORT_SYSTEMS` | Preferred parent | `RAILWAY`, `ROAD_INFRASTRUCTURE` |
| Transport | `USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION` | Preferred parent | `BUS_RAPID_TRANSIT`, `LOCOMOTIVES`, `PRIVATE_VEHICLES` |
| Transport | `NON_MOTORISED_TRANSPORT_SYSTEM` | Preferred parent | `CYCLING`, `PEDESTRIAN` |
| Forestry | `PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM` | Preferred parent | `COMMUNITY_BASED_FOREST_MANAGEMENT`, `JOINT_FOREST_MANAGEMENT` |
| Forestry | `LANDSCAPE_RESTORATION_PROGRAM` | Preferred parent | `PLANTED_FORESTS_AND_WOODLOTS`, `AGROFOREST`, `NATURAL_REGENERATION`, `MANGROVES_RESTORATION`, `AGROSILVIPASTORAL` |
| Waste | `ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE` | Preferred parent | `RECYCLING`, `SORTING`, `LANDFILL` |
| Waste | `WASTE_TO_ENERGY_TECHNOLOGIES` | Preferred parent | `BIOGAS_COW_DUNG`, `ALTERNATIVE_CHARCOAL_PRODUCTION` |
| Industry | `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES` | Current parent | Sources listed below |
| Livestock | `LIVESTOCK_MANAGEMENT_PRACTICE` | Preferred parent | `SILVICULTURE`, `AGRO_SILVOPASTURE`, `SILVOPASTURE`, `LAND_RANGE_RESTORATION` |
| Livestock | `USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS` | Preferred parent | `FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS`, `FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS` |

### Shared select values

| Field | Accepted values |
|---|---|
| `coverage` | `TANZANIA_MAINLAND`, `ZANZIBAR` |
| Common `ownership` | `GOVERNMENT`, `PRIVATE`, `INDIVIDUAL`, `COMMUNITY` (some transport schemas allow a narrower subset) |
| `managementType` | `VILLAGE_GOVERNMENT`, `COMMUNITY` |
| Transport `energySource` | `ELECTRICITY`, `GAS` |
| `wasteType` | `ORGANIC_WASTE`, `LIVESTOCK_MANURE_WASTE` |
| Charcoal `productionMaterial` | `ORGANIC_WASTE`, `RICE_HUSK` |
| Industry `source` | `HYDROPOWER`, `SOLAR_POWER`, `WIND`, `BIOENERGY`, `GEOTHERMAL`, `HYDROGEN_GAS`, `NATURAL_GAS`, `TIDAL_WAVES`, `NUCLEAR` |
| Industry `unit` | `MW`; `NATURAL_GAS` must use `MMscfd` |

## Energy — six payloads

### E1. Clean Power Interconnection

```json
{"category":"IMPROVE_CLEAN_POWER_INTERCONNECTION_WITH_NEIGHBOURING_COUNTRIES","sectorMeasureCategoryUid":"REPLACE_WITH_INTERCONNECTION_UID","values":{"name":"Tanzania Zambia Interconnector","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2025,"receivedCapacity":400.5,"ownership":"GOVERNMENT","quantityReducedCO2":1200.5,"quantityReducedCH4":45.2,"quantityReducedN2O":12.4,"quantityReducedHFCs":0}}
```

### E2. Hydropower

```json
{"category":"CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION","sectorMeasureCategoryUid":"REPLACE_WITH_CLEAN_TECHNOLOGIES_UID","values":{"technologies":[{"technologyType":"HYDROPOWER","name":"Hydropower Station A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2022,"installedCapacity":500,"productionCapacity":420,"ownership":"GOVERNMENT","quantityReducedCO2":5000,"quantityReducedCH4":100,"quantityReducedN2O":30,"quantityReducedHFCs":0}]}}
```

### E3. Solar Power

```json
{"category":"CLEAN_TECHNOLOGIES_FOR_POWER_GENERATION","sectorMeasureCategoryUid":"REPLACE_WITH_CLEAN_TECHNOLOGIES_UID","values":{"technologies":[{"technologyType":"SOLAR_POWER","name":"Solar Plant B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"installedCapacity":250,"productionCapacity":190,"ownership":"PRIVATE","quantityReducedCO2":2200,"quantityReducedCH4":40,"quantityReducedN2O":10,"quantityReducedHFCs":0}]}}
```

### E4. Natural Gas Power Production

```json
{"category":"USE_OF_NATURAL_GAS","sectorMeasureCategoryUid":"REPLACE_WITH_NATURAL_GAS_UID","values":{"activities":[{"activityType":"POWER_PRODUCTION","name":"Natural Gas Plant A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2025,"installedCapacity":150.5,"naturalGasConsumed":23.75,"ownership":"GOVERNMENT","quantityReducedCO2":125.5,"quantityReducedCH4":4.2,"quantityReducedN2O":1.1,"quantityReducedHFCs":0}]}}
```

### E5. Natural Gas Cooking

```json
{"category":"USE_OF_NATURAL_GAS","sectorMeasureCategoryUid":"REPLACE_WITH_NATURAL_GAS_UID","values":{"activities":[{"activityType":"COOKING","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"numberOfHouseholds":500,"numberOfInstitutions":20,"numberOfBeneficiaries":2500,"naturalGasConsumed":18.75,"quantityReducedCO2":450.5,"quantityReducedCH4":12.3,"quantityReducedN2O":4.2,"quantityReducedHFCs":0}]}}
```

### E6. Rural Electrification

```json
{"category":"CLIMATE_SMART_RURAL_ELECTRIFICATION","sectorMeasureCategoryUid":"REPLACE_WITH_RURAL_ELECTRIFICATION_UID","values":{"activities":[{"activityType":"MICRO_RENEWABLE_GENERATION","name":"Micro Renewable Plant A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"installedCapacity":100,"productionCapacity":80,"ownership":"COMMUNITY","quantityReducedCO2":300,"quantityReducedCH4":20,"quantityReducedN2O":5,"quantityReducedHFCs":0},{"activityType":"MINI_GRID_RENEWABLE_GENERATION","name":"Mini Grid B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"installedCapacity":250,"productionCapacity":200,"ownership":"PRIVATE","quantityReducedCO2":500,"quantityReducedCH4":30,"quantityReducedN2O":10,"quantityReducedHFCs":0}]}}
```

## Transport — six payloads

### T1. Railway

```json
{"category":"LOW_EMISSION_TRANSPORT_SYSTEMS","sectorMeasureCategoryUid":"REPLACE_WITH_LOW_EMISSION_TRANSPORT_UID","values":{"activities":[{"activityType":"RAILWAY","name":"Central Corridor SGR","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"railType":"SGR","capacity":5000,"networkDistance":300,"coverageFrom":"Dar es Salaam","coverageTo":"Morogoro","ownership":"GOVERNMENT","quantityReducedCO2":2000,"quantityReducedCH4":100,"quantityReducedN2O":50,"quantityReducedHFCs":0}]}}
```

### T2. Road Infrastructure

```json
{"category":"LOW_EMISSION_TRANSPORT_SYSTEMS","sectorMeasureCategoryUid":"REPLACE_WITH_LOW_EMISSION_TRANSPORT_UID","values":{"activities":[{"activityType":"ROAD_INFRASTRUCTURE","name":"BRT Corridor","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"infrastructureType":"BRT","capacity":3000,"networkDistance":25,"coverageFrom":"Mbagala","coverageTo":"City Centre","ownership":"GOVERNMENT","quantityReducedCO2":1500,"quantityReducedCH4":80,"quantityReducedN2O":30,"quantityReducedHFCs":0}]}}
```

### T3. Bus Rapid Transit

```json
{"category":"USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION","sectorMeasureCategoryUid":"REPLACE_WITH_RENEWABLE_TRANSPORT_UID","values":{"activities":[{"activityType":"BUS_RAPID_TRANSIT","name":"Electric BRT A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"transportType":"BRT","capacity":500,"populationServed":20000,"energySource":"ELECTRICITY","energyConsumptionPerDay":1200,"ownership":"GOVERNMENT","quantityReducedCO2":1500,"quantityReducedCH4":50,"quantityReducedN2O":20,"quantityReducedHFCs":0}]}}
```

### T4. Locomotive

```json
{"category":"USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION","sectorMeasureCategoryUid":"REPLACE_WITH_RENEWABLE_TRANSPORT_UID","values":{"activities":[{"activityType":"LOCOMOTIVES","name":"Electric Locomotive A","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2023,"transportType":"SGR","vehicleNumber":"LOCO-001","capacity":1000,"populationServed":5000,"energySource":"ELECTRICITY","energyConsumptionPerDay":2500,"ownership":"GOVERNMENT","quantityReducedCO2":2500,"quantityReducedCH4":60,"quantityReducedN2O":30,"quantityReducedHFCs":0}]}}
```

### T5. Private Vehicle

```json
{"category":"USE_OF_RENEWABLE_ENERGY_IN_TRANSPORTATION","sectorMeasureCategoryUid":"REPLACE_WITH_RENEWABLE_TRANSPORT_UID","values":{"activities":[{"activityType":"PRIVATE_VEHICLES","vehicleNumber":"T123ABC","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2025,"populationServed":5,"energySource":"ELECTRICITY","energyConsumptionPerDay":30,"ownership":"PRIVATE","quantityReducedCO2":50,"quantityReducedCH4":2,"quantityReducedN2O":1,"quantityReducedHFCs":0}]}}
```

### T6. Cycling and Pedestrian

```json
{"category":"NON_MOTORISED_TRANSPORT_SYSTEM","sectorMeasureCategoryUid":"REPLACE_WITH_NON_MOTORISED_UID","values":{"activities":[{"activityType":"CYCLING","name":"Dodoma Cycling Network","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"capacity":1000,"populationServed":5000,"networkDistance":30,"coverageFrom":"City Centre","coverageTo":"Nzuguni","quantityReducedCO2":1000,"quantityReducedCH4":20,"quantityReducedN2O":5,"quantityReducedHFCs":0},{"activityType":"PEDESTRIAN","name":"Dodoma Pedestrian Network","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"populationServed":8000,"ownership":"GOVERNMENT","networkDistance":15,"coverageFrom":"City Centre","coverageTo":"Area C","quantityReducedCO2":500,"quantityReducedCH4":10,"quantityReducedN2O":2,"quantityReducedHFCs":0}]}}
```

## Forestry — six payloads

### F1. CBFM

```json
{"category":"PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_PARTICIPATORY_FOREST_UID","values":{"activities":[{"activityType":"COMMUNITY_BASED_FOREST_MANAGEMENT","name":"Community Forest A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2020,"coverageArea":1500,"managementType":"COMMUNITY","numberOfBeneficiaries":2000,"quantityReducedCO2":5000,"quantityReducedCH4":100,"quantityReducedN2O":50,"quantityReducedHFCs":0}]}}
```

### F2. JFM

```json
{"category":"PARTICIPATORY_FOREST_MANAGEMENT_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_PARTICIPATORY_FOREST_UID","values":{"activities":[{"activityType":"JOINT_FOREST_MANAGEMENT","name":"Joint Forest B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2021,"coverageArea":2000,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":3000,"quantityReducedCO2":7000,"quantityReducedCH4":150,"quantityReducedN2O":70,"quantityReducedHFCs":0}]}}
```

### F3. Planted Forests and Woodlots

```json
{"category":"LANDSCAPE_RESTORATION_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_LANDSCAPE_UID","values":{"activities":[{"activityType":"PLANTED_FORESTS_AND_WOODLOTS","name":"Forest Restoration A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"coverageArea":1200,"numberOfTrees":25000,"managementType":"COMMUNITY","numberOfBeneficiaries":500,"quantityReducedCO2":3000,"quantityReducedCH4":50,"quantityReducedN2O":20,"quantityReducedHFCs":0}]}}
```

### F4. Agroforest

```json
{"category":"LANDSCAPE_RESTORATION_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_LANDSCAPE_UID","values":{"activities":[{"activityType":"AGROFOREST","name":"Agroforest Project B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"coverageArea":850,"numberOfTrees":16000,"managementType":"COMMUNITY","numberOfBeneficiaries":750,"quantityReducedCO2":2200,"quantityReducedCH4":35,"quantityReducedN2O":12,"quantityReducedHFCs":0}]}}
```

### F5. Natural Regeneration

```json
{"category":"LANDSCAPE_RESTORATION_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_LANDSCAPE_UID","values":{"activities":[{"activityType":"NATURAL_REGENERATION","name":"Natural Regeneration Project","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2022,"coverageArea":2000,"numberOfTrees":30000,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":1200,"quantityReducedCO2":4500,"quantityReducedCH4":70,"quantityReducedN2O":25,"quantityReducedHFCs":0}]}}
```

### F6. Mangroves and Agrosilvipastoral

```json
{"category":"LANDSCAPE_RESTORATION_PROGRAM","sectorMeasureCategoryUid":"REPLACE_WITH_LANDSCAPE_UID","values":{"activities":[{"activityType":"MANGROVES_RESTORATION","name":"Mangrove Restoration A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"coverageArea":500,"numberOfTrees":10000,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":1000,"quantityReducedCO2":2000,"quantityReducedCH4":40,"quantityReducedN2O":15,"quantityReducedHFCs":0},{"activityType":"AGROSILVIPASTORAL","name":"Agrosilvipastoral B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2025,"coverageArea":900,"numberOfTrees":12500,"managementType":"COMMUNITY","numberOfBeneficiaries":800,"quantityReducedCO2":1800,"quantityReducedCH4":30,"quantityReducedN2O":10,"quantityReducedHFCs":0}]}}
```

## Waste — six payloads

### W1. Recycling

```json
{"category":"ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_MANAGEMENT_UID","values":{"activities":[{"activityType":"RECYCLING","name":"Recycling Industry A","wasteType":"ORGANIC_WASTE","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"installedCapacity":500,"productionCapacity":400,"ownership":"PRIVATE","numberOfBeneficiaries":1000,"quantityReducedCO2":1500,"quantityReducedCH4":100,"quantityReducedN2O":50,"quantityReducedHFCs":0}]}}
```

### W2. Sorting

```json
{"category":"ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_MANAGEMENT_UID","values":{"activities":[{"activityType":"SORTING","name":"Sorting Facility B","wasteType":"LIVESTOCK_MANURE_WASTE","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"sortingCapacity":300,"ownership":"GOVERNMENT","numberOfBeneficiaries":500,"quantityReducedCO2":700,"quantityReducedCH4":50,"quantityReducedN2O":20,"quantityReducedHFCs":0}]}}
```

### W3. Landfill

```json
{"category":"ENVIRONMENTAL_SOUND_WASTE_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_MANAGEMENT_UID","values":{"activities":[{"activityType":"LANDFILL","name":"Sanitary Landfill C","wasteType":"ORGANIC_WASTE","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2022,"organicWasteReceivingCapacity":1000,"landfillSize":50,"ownership":"GOVERNMENT","organicWasteCompactedPerDay":200,"quantityReducedCO2":2000,"quantityReducedCH4":500,"quantityReducedN2O":100,"quantityReducedHFCs":0}]}}
```

### W4. Biogas

```json
{"category":"WASTE_TO_ENERGY_TECHNOLOGIES","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_TO_ENERGY_UID","values":{"technologies":[{"technologyType":"BIOGAS_COW_DUNG","name":"Biogas Plant A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"installedCapacity":200,"productionCapacityPerDay":150,"numberOfBeneficiaries":500,"ownership":"PRIVATE","quantityReducedCO2":800,"quantityReducedCH4":200,"quantityReducedN2O":30,"quantityReducedHFCs":0}]}}
```

### W5. Alternative Charcoal

```json
{"category":"WASTE_TO_ENERGY_TECHNOLOGIES","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_TO_ENERGY_UID","values":{"technologies":[{"technologyType":"ALTERNATIVE_CHARCOAL_PRODUCTION","name":"Charcoal Plant B","productionMaterial":"RICE_HUSK","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"installedCapacity":500,"productionCapacityPerDay":200,"numberOfBeneficiaries":1000,"ownership":"GOVERNMENT","quantityReducedCO2":1200,"quantityReducedCH4":100,"quantityReducedN2O":40,"quantityReducedHFCs":0}]}}
```

### W6. Multiple Waste-to-Energy Technologies

```json
{"category":"WASTE_TO_ENERGY_TECHNOLOGIES","sectorMeasureCategoryUid":"REPLACE_WITH_WASTE_TO_ENERGY_UID","values":{"technologies":[{"technologyType":"BIOGAS_COW_DUNG","name":"Community Biogas Plant","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2025,"installedCapacity":350,"productionCapacityPerDay":240,"numberOfBeneficiaries":750,"ownership":"COMMUNITY","quantityReducedCO2":950,"quantityReducedCH4":230,"quantityReducedN2O":35,"quantityReducedHFCs":0},{"technologyType":"ALTERNATIVE_CHARCOAL_PRODUCTION","name":"Organic Charcoal Plant","productionMaterial":"ORGANIC_WASTE","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2025,"installedCapacity":600,"productionCapacityPerDay":260,"numberOfBeneficiaries":1200,"ownership":"PRIVATE","quantityReducedCO2":1400,"quantityReducedCH4":120,"quantityReducedN2O":45,"quantityReducedHFCs":0}]}}
```

## Industry — six payloads

All use `DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES`. The examples vary the
nested source. Natural Gas must use `MMscfd`; all other sources use `MW`.

### I1. Hydropower

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Cement Industry A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2020,"installedCapacity":3000,"productionCapacity":2800,"energySources":[{"source":"HYDROPOWER","consumption":500,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":5000,"quantityReducedCH4":100,"quantityReducedN2O":40,"quantityReducedHFCs":0}]}}
```

### I2. Solar Power

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Textile Industry B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2021,"installedCapacity":1500,"productionCapacity":1250,"energySources":[{"source":"SOLAR_POWER","consumption":180,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":1600,"quantityReducedCH4":40,"quantityReducedN2O":15,"quantityReducedHFCs":0}]}}
```

### I3. Wind

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Steel Industry C","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2019,"installedCapacity":4000,"productionCapacity":3600,"energySources":[{"source":"WIND","consumption":220,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":2100,"quantityReducedCH4":55,"quantityReducedN2O":18,"quantityReducedHFCs":0}]}}
```

### I4. Geothermal

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Mining Industry D","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2022,"installedCapacity":2500,"productionCapacity":2100,"energySources":[{"source":"GEOTHERMAL","consumption":300,"unit":"MW","ownership":"GOVERNMENT","quantityReducedCO2":2900,"quantityReducedCH4":65,"quantityReducedN2O":20,"quantityReducedHFCs":0}]}}
```

### I5. Natural Gas

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Food Industry E","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"installedCapacity":1200,"productionCapacity":950,"energySources":[{"source":"NATURAL_GAS","consumption":23.75,"unit":"MMscfd","ownership":"PRIVATE","quantityReducedCO2":900,"quantityReducedCH4":30,"quantityReducedN2O":8,"quantityReducedHFCs":0}]}}
```

### I6. Multiple Sources

```json
{"category":"DIVERSIFICATION_OF_CLEAN_ENERGY_SOURCES","sectorMeasureCategoryUid":"REPLACE_WITH_INDUSTRY_UID","values":{"industryName":"Manufacturing Industry F","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2024,"installedCapacity":5000,"productionCapacity":4400,"energySources":[{"source":"SOLAR_POWER","consumption":250,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":1800,"quantityReducedCH4":35,"quantityReducedN2O":10,"quantityReducedHFCs":0},{"source":"BIOENERGY","consumption":120,"unit":"MW","ownership":"PRIVATE","quantityReducedCO2":1100,"quantityReducedCH4":70,"quantityReducedN2O":20,"quantityReducedHFCs":0},{"source":"NATURAL_GAS","consumption":15.5,"unit":"MMscfd","ownership":"PRIVATE","quantityReducedCO2":600,"quantityReducedCH4":20,"quantityReducedN2O":5,"quantityReducedHFCs":0}]}}
```

## Livestock — six payloads

### L1. All Livestock Management Activities

```json
{"category":"LIVESTOCK_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_LIVESTOCK_MANAGEMENT_UID","values":{"activities":[{"activityType":"SILVICULTURE","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"coverageArea":450.75,"treesCropsLivestockLots":120,"managementType":"COMMUNITY","numberOfBeneficiaries":2500,"quantityReducedCO2":100.5,"quantityReducedCH4":12.3,"quantityReducedN2O":4.2,"quantityReducedHFCs":0.8},{"activityType":"AGRO_SILVOPASTURE","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2025,"coverageArea":600,"treesFoddersCropsLivestockLots":200,"managementType":"COMMUNITY","numberOfBeneficiaries":1800,"quantityReducedCO2":300,"quantityReducedCH4":25,"quantityReducedN2O":8,"quantityReducedHFCs":0},{"activityType":"SILVOPASTURE","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2023,"coverageArea":750,"treesLandRangeLots":150,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":2200,"quantityReducedCO2":420,"quantityReducedCH4":30,"quantityReducedN2O":9,"quantityReducedHFCs":0},{"activityType":"LAND_RANGE_RESTORATION","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2022,"coverageArea":1200,"treesFoddersLots":300,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":3500,"quantityReducedCO2":800,"quantityReducedCH4":45,"quantityReducedN2O":12,"quantityReducedHFCs":0}]}}
```

### L2. Silviculture Only

```json
{"category":"LIVESTOCK_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_LIVESTOCK_MANAGEMENT_UID","values":{"activities":[{"activityType":"SILVICULTURE","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"coverageArea":450.75,"treesCropsLivestockLots":120,"managementType":"COMMUNITY","numberOfBeneficiaries":2500,"quantityReducedCO2":100.5,"quantityReducedCH4":12.3,"quantityReducedN2O":4.2,"quantityReducedHFCs":0.8}]}}
```

### L3. Land Range Restoration Only

```json
{"category":"LIVESTOCK_MANAGEMENT_PRACTICE","sectorMeasureCategoryUid":"REPLACE_WITH_LIVESTOCK_MANAGEMENT_UID","values":{"activities":[{"activityType":"LAND_RANGE_RESTORATION","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2022,"coverageArea":1200,"treesFoddersLots":300,"managementType":"VILLAGE_GOVERNMENT","numberOfBeneficiaries":3500,"quantityReducedCO2":800,"quantityReducedCH4":45,"quantityReducedN2O":12,"quantityReducedHFCs":0}]}}
```

### L4. Improved Breeds and Fodders Together

```json
{"category":"USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS","sectorMeasureCategoryUid":"REPLACE_WITH_IMPROVED_BREEDS_FEEDS_UID","values":{"activities":[{"activityType":"FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS","name":"Improved Livestock Breed Farm A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"coverageArea":450.75,"livestockProductionCapacity":800,"ownership":"PRIVATE","numberOfBeneficiaries":2500,"quantityReducedCO2":500,"quantityReducedCH4":80,"quantityReducedN2O":20,"quantityReducedHFCs":0},{"activityType":"FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS","name":"Improved Fodder Farm B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2025,"coverageArea":650,"fodderProductionCapacity":1200,"ownership":"COMMUNITY","numberOfBeneficiaries":3000,"quantityReducedCO2":450,"quantityReducedCH4":65,"quantityReducedN2O":18,"quantityReducedHFCs":0}]}}
```

### L5. Improved Livestock Breeds Only

```json
{"category":"USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS","sectorMeasureCategoryUid":"REPLACE_WITH_IMPROVED_BREEDS_FEEDS_UID","values":{"activities":[{"activityType":"FARMS_WITH_IMPROVED_LIVESTOCK_BREEDS","name":"Improved Livestock Breed Farm A","coverage":"TANZANIA_MAINLAND","regionId":6,"councilId":105,"year":2024,"coverageArea":450.75,"livestockProductionCapacity":800,"ownership":"PRIVATE","numberOfBeneficiaries":2500,"quantityReducedCO2":500,"quantityReducedCH4":80,"quantityReducedN2O":20,"quantityReducedHFCs":0}]}}
```

### L6. Improved Livestock Fodders Only

```json
{"category":"USE_OF_IMPROVED_ANIMAL_BREEDS_AND_FEEDS","sectorMeasureCategoryUid":"REPLACE_WITH_IMPROVED_BREEDS_FEEDS_UID","values":{"activities":[{"activityType":"FARMS_WITH_IMPROVED_LIVESTOCK_FODDERS","name":"Improved Fodder Farm B","coverage":"TANZANIA_MAINLAND","regionId":7,"councilId":110,"year":2025,"coverageArea":650,"fodderProductionCapacity":1200,"ownership":"COMMUNITY","numberOfBeneficiaries":3000,"quantityReducedCO2":450,"quantityReducedCH4":65,"quantityReducedN2O":18,"quantityReducedHFCs":0}]}}
```

## Important interpretation

- `coverage` and `coverageArea` are different fields.
- Gas quantities remain separate reported values and are not automatically converted to CO₂e.
- Counts such as households, institutions, beneficiaries, trees and vehicles must be integers.
- Do not send a placeholder UID or copy a UID from another environment.
- When a parent supports multiple children, every child may use a different valid Region, Council and year.
