# RDF Model & Ontologies

This document describes the RDF model, ontology patterns, URI schemes, and design principles used in the E-OBS Climate Knowledge Graph Pipeline.

---

## 🌐 Namespace Prefixes

The pipeline uses a simplified 8-prefix namespace design for clarity and efficiency:

```turtle
@prefix cf: <http://vocab.nerc.ac.uk/standard_name/> .
@prefix geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#> .
@prefix oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#> .
@prefix observation: <http://climate-kg.org/observation/> .
@prefix qudt: <http://qudt.org/schema/qudt/> .
@prefix sosa: <http://www.w3.org/ns/sosa/> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
```

### Namespace Descriptions

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| `cf` | CF Standard Names | Observed properties (climate variables) |
| `geo1` | WGS84 Geo Positioning | Geographic locations (grid cells) |
| `oboe` | OBOE Core Ontology | Observation collections |
| `observation` | Climate-KG Observations | Observation and collection instances |
| `qudt` | QUDT Schema | Quantity values and measurements |
| `sosa` | SOSA/SSN Ontology | Observations, sensors, and sensing patterns |
| `unit` | QUDT Units | Measurement units |
| `xsd` | XML Schema Datatypes | Data types (dateTime, decimal, etc.) |

---

## 🔗 URI Patterns

### Observation URIs

**Pattern**: `observation:{var}_{lat}_{lon}_{timestamp}`

**Example**: `observation:tg_50p00_3p00_20200101T000000`

**Components**:
- `{var}`: Variable code (tg, tn, tx, rr, hu, fg, qq)
- `{lat}`: Latitude with 2 decimals, "p" for decimal point (e.g., `50p00` = 50.00°)
- `{lon}`: Longitude with 2 decimals, "p" for decimal point (e.g., `3p00` = 3.00°)
- `{timestamp}`: ISO 8601 basic format (YYYYMMDDTHHmmss)

**Uniqueness**: Guaranteed unique per location + timestamp combination

### ObservationCollection URIs

**Pattern**: `observation:eobs_{var}_{year_start}_{year_end}`

**Example**: `observation:eobs_tg_2020_2024`

**Components**:
- `eobs`: Dataset identifier (E-OBS)
- `{var}`: Variable code
- `{year_start}`: First year in collection
- `{year_end}`: Last year in collection

### Sensor URIs

**Pattern**: `sosa:{var}_sensor`

**Example**: `sosa:tg_sensor`

**Components**:
- `{var}`: Variable code
- `_sensor`: Suffix indicating sensor type

**Note**: One sensor per variable (simplified model)

### Location URIs

**Pattern**: `geo1:grid_{lat}_{lon}`

**Example**: `geo1:grid_50p00_3p00`

**Components**:
- `grid_`: Prefix indicating grid cell
- `{lat}`: Latitude (formatted as above)
- `{lon}`: Longitude (formatted as above)

**Note**: Location URIs are references only; no WKT geometry or coordinate properties are added to the graph

### Property URIs

**Pattern**: `cf:{cf_standard_name}`

**Example**: `cf:air_temperature`

**Description**: Direct references to CF Standard Names vocabulary

**Common Properties**:
- `cf:air_temperature` (tg, tn, tx)
- `cf:precipitation_amount` (rr)
- `cf:relative_humidity` (hu)
- `cf:wind_speed` (fg)
- `cf:surface_downwelling_shortwave_flux_in_air` (qq)

### Unit URIs

**Pattern**: `unit:{QUDT_unit_code}`

**Example**: `unit:DEG_C`

**Common Units**:
- `unit:DEG_C` (temperature in Celsius)
- `unit:MilliM` (precipitation in millimeters)
- `unit:PERCENT` (humidity percentage)
- `unit:M-PER-SEC` (wind speed in m/s)
- `unit:W-PER-M2` (radiation in W/m²)

---

## 📦 SOSA Observation Pattern

Each climate observation follows the SOSA (Sensor, Observation, Sample, and Actuator) ontology pattern:

### Basic Structure

```turtle
observation:tg_50p00_3p00_20200101T000000 a sosa:Observation ;
    sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
    sosa:hasResult [
        a qudt:QuantityValue ;
        qudt:numericValue 1.523000e+01 ;
        qudt:unit unit:DEG_C
    ] ;
    sosa:madeBySensor sosa:tg_sensor ;
    sosa:observedProperty cf:air_temperature ;
    sosa:resultTime "2020-01-01T00:00:00"^^xsd:dateTime .
```

### Triple-by-Triple Explanation

1. **Type Declaration**
   ```turtle
   observation:tg_50p00_3p00_20200101T000000 a sosa:Observation .
   ```
   Declares the resource as a SOSA Observation

2. **Feature of Interest** (Location)
   ```turtle
   sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 .
   ```
   Links to the grid cell location (50.00°N, 3.00°E)

3. **Result** (Measurement Value)
   ```turtle
   sosa:hasResult [
       a qudt:QuantityValue ;
       qudt:numericValue 1.523000e+01 ;
       qudt:unit unit:DEG_C
   ] .
   ```
   Blank node containing:
   - Type: `qudt:QuantityValue`
   - Numeric value: 15.23 (scientific notation)
   - Unit: Degrees Celsius

4. **Sensor**
   ```turtle
   sosa:madeBySensor sosa:tg_sensor .
   ```
   Links to the sensor that made the observation

5. **Observed Property**
   ```turtle
   sosa:observedProperty cf:air_temperature .
   ```
   Links to the CF Standard Name for the climate variable

6. **Result Time**
   ```turtle
   sosa:resultTime "2020-01-01T00:00:00"^^xsd:dateTime .
   ```
   Timestamp when the observation was made (ISO 8601 format)

---

## 📚 ObservationCollection Pattern

Observations are grouped into collections using the OBOE (Extensible Observation Ontology) `ObservationCollection` class:

### Collection Structure

```turtle
observation:eobs_tg_2020_2024 a oboe:ObservationCollection ;
    sosa:hasMember observation:tg_50p00_3p00_20200101T000000,
                   observation:tg_50p00_3p00_20200102T000000,
                   observation:tg_50p00_3p00_20200103T000000,
                   ... .
```

### Purpose

- **Organization**: Group all observations for a variable and time period
- **Discovery**: Enable queries to find all observations in a collection
- **Metadata**: Potential anchor point for collection-level metadata (not currently used)

### One Collection Per Variable

For each variable in the processed batch, one collection is created:

```turtle
# Temperature collection
observation:eobs_tg_2020_2024 a oboe:ObservationCollection ;
    sosa:hasMember <temperature observations> .

# Precipitation collection
observation:eobs_rr_2020_2024 a oboe:ObservationCollection ;
    sosa:hasMember <precipitation observations> .

# Wind speed collection
observation:eobs_fg_2020_2024 a oboe:ObservationCollection ;
    sosa:hasMember <wind speed observations> .
```

---

## 🎨 Design Principles

### 1. Minimal Prefixes

**Principle**: Use only essential namespaces (8 total)

**Rationale**:
- Reduces cognitive load for users
- Simplifies SPARQL queries
- Improves file readability
- Faster parsing and loading

**Excluded Namespaces**:
- PROV (provenance) - not included in simplified model
- GeoSPARQL - locations are references only
- SKOS - no concept schemes needed
- DCTERMS - minimal metadata approach

### 2. No Metadata Overhead

**Principle**: Focus on core observation data

**What's Included**:
- Observation timestamp
- Measurement value and unit
- Observed property
- Location reference
- Sensor reference

**What's Excluded**:
- Data provenance (PROV)
- Spatial geometry (WKT, GeoJSON)
- Sensor metadata (type, manufacturer, calibration)
- Collection metadata (creation date, creator)
- Quality flags or uncertainty

**Rationale**: Simplicity and performance

### 3. Direct CF Alignment

**Principle**: Use CF Standard Name URIs directly as observed properties

**Implementation**:
```python
# config.py
VARIABLE_MAPPINGS = {
    "tg": {
        "cf_standard_name": "air_temperature",
        ...
    }
}

# Result in RDF:
sosa:observedProperty cf:air_temperature .
```

**Benefits**:
- Direct compatibility with CF conventions
- No intermediate mapping ontology needed
- Clear semantics for climate data users

### 4. Standardized Sensors

**Principle**: One sensor URI per variable

**Implementation**:
- `sosa:tg_sensor` for all temperature observations
- `sosa:rr_sensor` for all precipitation observations
- etc.

**Rationale**:
- Simplified model (E-OBS is an ensemble, not individual sensors)
- Consistent querying across all observations
- Can be extended later if needed

### 5. Standardized Locations

**Principle**: Grid cell URIs using WGS84 namespace

**Implementation**:
```turtle
geo1:grid_50p00_3p00
geo1:grid_50p00_3p25
geo1:grid_50p00_3p50
```

**Benefits**:
- Consistent spatial referencing
- Standard namespace (WGS84)
- Can be extended with actual coordinate properties later

**Note**: Current implementation uses location URIs as references only. No lat/lon coordinate properties are added to the graph.

### 6. Collection-Based Organization

**Principle**: Group observations by variable and time period

**Implementation**:
- One `oboe:ObservationCollection` per variable per batch
- All observations linked via `sosa:hasMember`

**Benefits**:
- Easier discovery of related observations
- Support for collection-level queries
- Potential for future collection metadata

---

## 🔄 Variable to CF Standard Name Mapping

| Variable | CF Standard Name | QUDT Unit |
|----------|------------------|-----------|
| **fg** | `wind_speed` | `M-PER-SEC` |
| **hu** | `relative_humidity` | `PERCENT` |
| **qq** | `surface_downwelling_shortwave_flux_in_air` | `W-PER-M2` |
| **rr** | `precipitation_amount` | `MilliM` |
| **tg** | `air_temperature` | `DEG_C` |
| **tn** | `air_temperature` | `DEG_C` |
| **tx** | `air_temperature` | `DEG_C` |

**Note**: Multiple variables (tg, tn, tx) can map to the same CF standard name. They are distinguished by their sensor URIs and observation URIs.

---

## 📊 Complete Example

Here's a complete example showing one observation and its collection:

```turtle
@prefix cf: <http://vocab.nerc.ac.uk/standard_name/> .
@prefix geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#> .
@prefix oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#> .
@prefix observation: <http://climate-kg.org/observation/> .
@prefix qudt: <http://qudt.org/schema/qudt/> .
@prefix sosa: <http://www.w3.org/ns/sosa/> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

# ============================================================
# ObservationCollection for mean temperature (2020-2024)
# ============================================================
observation:eobs_tg_2020_2024 a oboe:ObservationCollection ;
    sosa:hasMember observation:tg_50p00_3p00_20200101T000000,
                   observation:tg_50p00_3p00_20200102T000000 .

# ============================================================
# Individual Observation: Temperature on Jan 1, 2020
# ============================================================
observation:tg_50p00_3p00_20200101T000000 a sosa:Observation ;
    sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
    sosa:hasResult [
        a qudt:QuantityValue ;
        qudt:numericValue 1.523000e+01 ;
        qudt:unit unit:DEG_C
    ] ;
    sosa:madeBySensor sosa:tg_sensor ;
    sosa:observedProperty cf:air_temperature ;
    sosa:resultTime "2020-01-01T00:00:00"^^xsd:dateTime .

# ============================================================
# Individual Observation: Temperature on Jan 2, 2020
# ============================================================
observation:tg_50p00_3p00_20200102T000000 a sosa:Observation ;
    sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
    sosa:hasResult [
        a qudt:QuantityValue ;
        qudt:numericValue 1.612000e+01 ;
        qudt:unit unit:DEG_C
    ] ;
    sosa:madeBySensor sosa:tg_sensor ;
    sosa:observedProperty cf:air_temperature ;
    sosa:resultTime "2020-01-02T00:00:00"^^xsd:dateTime .
```

---

## 🎯 Known Limitations

### Current Model Simplifications

1. **No Spatial Geometry**
   - Location URIs are references only
   - No WKT (Well-Known Text) geometries
   - No lat/lon coordinate properties in the graph
   - Users must know grid coordinates from external sources

2. **No Sensor Metadata**
   - Sensor URIs are references only
   - No sensor type, manufacturer, or calibration info
   - One sensor per variable (simplified abstraction)

3. **No Provenance**
   - No PROV-based data lineage
   - No processing history
   - No dataset attribution in graph

4. **Minimal Metadata**
   - No collection-level metadata (creator, creation date, etc.)
   - No observation quality flags
   - No uncertainty information

### Design Trade-offs

These limitations are intentional design decisions to:
- **Simplify the model** for clarity and ease of use
- **Improve performance** (less triples = faster processing and querying)
- **Reduce file size** for easier distribution and loading

The model can be extended in future versions if needed.

---

## 🔧 Extending the Model

### Adding Spatial Geometry

To add WKT geometries for locations:

```turtle
geo1:grid_50p00_3p00 a geo:Feature ;
    geo:hasGeometry [
        a geo:Point ;
        geo:asWKT "POINT(3.00 50.00)"^^geo:wktLiteral
    ] .
```

### Adding Sensor Metadata

To add sensor details:

```turtle
sosa:tg_sensor a sosa:Sensor ;
    rdfs:label "Mean Temperature Sensor"@en ;
    sosa:observes cf:air_temperature .
```

### Adding Provenance

To add data provenance:

```turtle
observation:tg_50p00_3p00_20200101T000000
    prov:wasGeneratedBy [
        a prov:Activity ;
        prov:used <https://www.ecad.eu/download/ensembles/data/tg_ens_mean_0.25deg_reg_v31.0e.nc> ;
        prov:startedAtTime "2026-04-14T16:38:51"^^xsd:dateTime
    ] .
```

---

[← Back to Main README](../README.md)
