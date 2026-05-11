# Architecture & Pipeline Workflow

This document provides detailed information about the repository structure and pipeline workflow of the E-OBS Climate Knowledge Graph Pipeline.

---

## 📁 Repository Structure

```
code/
├── code/                          # Core pipeline modules
│   ├── config.py                  # Configuration, namespaces, URI patterns
│   ├── rdf_builder.py            # RDF graph construction (SOSA patterns)
│   ├── netcdf_reader.py          # NetCDF data extraction (xarray)
│   ├── pipeline.py               # Main orchestration & CLI
│   └── README.md                 # Module documentation
│
├── netcdf/                        # Input NetCDF files (E-OBS v31.0e)
│   ├── fg_ens_mean_0.25deg_reg_v31.0e.nc    (~2.1 GB)
│   ├── hu_ens_mean_0.25deg_reg_v31.0e.nc    (~10 GB)
│   ├── qq_ens_mean_0.25deg_reg_v31.0e.nc    (~1.5 GB)
│   ├── rr_ens_mean_0.25deg_reg_v31.0e.nc
│   ├── tg_ens_mean_0.25deg_reg_v31.0e.nc
│   ├── tn_ens_mean_0.25deg_reg_v31.0e.nc
│   └── tx_ens_mean_0.25deg_reg_v31.0e.nc
│
├── ttl/                           # Output RDF/Turtle files
│   └── eobs_climate_kg_YYYY_YYYY_TIMESTAMP.ttl
│
├── docs/                          # Documentation files
│   ├── ARCHITECTURE.md            # This file
│   ├── RDF_MODEL.md              # RDF model details
│   ├── SPARQL_EXAMPLES.md        # Query examples
│   ├── PERFORMANCE.md            # Performance benchmarks
│   └── CONTRIBUTING.md           # License, citation, contributing
│
├── .venv/                         # Python virtual environment
├── requirements.txt               # Python dependencies
├── test_installation.py           # Installation verification script
├── quickstart.sh                  # Quick start script (Linux/Mac)
├── PROJECT_OVERVIEW.txt           # Detailed project guide
├── CONVERSION_FLOW.md             # Pipeline flow diagram
├── PRODUCTION_READY_STATUS.md     # Production status report
└── TROUBLESHOOTING.md             # Problem-solving guide
```

---

## 🔄 Pipeline Workflow

The pipeline transforms NetCDF climate data into RDF/Turtle format through five main stages:

### 1. NetCDF Reading

```
┌──────────────────┐
│  NetCDF Files    │  E-OBS v31.0e climate data
│  (7 variables)   │  Resolution: 0.25°, Period: 1950-2024
└────────┬─────────┘
         │
         ▼
```

**Module**: `netcdf_reader.py`

**Functions**:
- Load NetCDF files using xarray
- Extract latitude and longitude coordinates
- Filter data by time range (start_year, end_year)
- Sample spatial grid (every Nth cell)
- Extract timestamp and value arrays

**Input**: NetCDF files from `netcdf/` directory

**Output**: Structured data arrays (coordinates, timestamps, values)

### 2. Data Processing

```
┌──────────────────┐
│ NetCDF Reader    │  Extract coordinates, timestamps, values
│ (netcdf_reader)  │  Sample grid cells, filter time range
└────────┬─────────┘
         │
         ▼
```

**Module**: `netcdf_reader.py` + `config.py`

**Processing Steps**:
1. Apply spatial sampling (default: every 10th grid cell)
2. Apply temporal filtering (start_year to end_year)
3. Format coordinates (lat/lon with 2 decimal places, "p" for decimal point)
4. Format timestamps (ISO 8601 format, basic notation)
5. Map variable codes to CF standard names
6. Map CF standard names to QUDT units

### 3. RDF Graph Construction

```
┌──────────────────┐
│  RDF Builder     │  Create SOSA observations
│ (rdf_builder)    │  Link to ObservationCollections
└────────┬─────────┘  Assign sensors, locations, properties
         │
         ▼
```

**Module**: `rdf_builder.py`

**RDF Creation Steps**:

1. **Initialize Graph**: Create rdflib Graph with 8 namespace prefixes
2. **Create ObservationCollections**: One collection per variable and year range
3. **Create Observations**: For each data point, create:
   - Observation URI (unique per location + timestamp)
   - Link to sensor (one sensor per variable)
   - Link to observed property (CF standard name)
   - Link to location (grid cell URI)
   - Result value (blank node with numeric value + unit)
   - Result timestamp (xsd:dateTime)
4. **Link to Collections**: Add `sosa:hasMember` triples from collection to observations

**Output**: rdflib Graph object with all triples

### 4. Validation

```
┌──────────────────┐
│   Validation     │  Check required properties
│                  │  Verify graph structure
└────────┬─────────┘
         │
         ▼
```

**Module**: `rdf_builder.py` (validation functions)

**Validation Checks**:
- All observations have required SOSA properties:
  - `sosa:hasFeatureOfInterest`
  - `sosa:hasResult`
  - `sosa:madeBySensor`
  - `sosa:observedProperty`
  - `sosa:resultTime`
- All result values have:
  - `qudt:numericValue`
  - `qudt:unit`
- Collections properly link to observations via `sosa:hasMember`

### 5. Serialization

```
┌──────────────────┐
│   TTL Output     │  Serialize as Turtle format
│                  │  One file per year range
└──────────────────┘
```

**Module**: `pipeline.py` + rdflib

**Output Formatting**:
- Serialize graph to Turtle format
- Generate filename: `eobs_climate_kg_{year_start}_{year_end}_{timestamp}.ttl`
- Write to `ttl/` directory
- Log statistics (observation count, file size)

---

## 🧩 Module Details

### config.py

**Purpose**: Central configuration for namespaces, URI patterns, and variable mappings

**Key Components**:
- Namespace definitions (8 prefixes)
- URI generation functions
- Variable to CF standard name mappings
- CF standard name to QUDT unit mappings
- Processing parameters (batch sizes, sampling rates)

### netcdf_reader.py

**Purpose**: Read and extract data from NetCDF files

**Key Functions**:
- `load_netcdf_data(file_path, start_year, end_year)`: Load and filter NetCDF file
- `sample_spatial_data(dataset, sample_every_n)`: Sample grid cells
- `extract_observations(dataset, variable_code)`: Extract all observations

**Dependencies**: xarray, numpy, pandas

### rdf_builder.py

**Purpose**: Construct RDF graphs following SOSA ontology patterns

**Key Functions**:
- `create_graph()`: Initialize graph with namespaces
- `add_observation_collection(graph, variable, year_start, year_end)`: Create collection
- `add_observation(graph, variable, lat, lon, timestamp, value)`: Add single observation
- `validate_graph(graph)`: Validate RDF structure

**Dependencies**: rdflib

### pipeline.py

**Purpose**: Main orchestration and command-line interface

**Key Functions**:
- `process_batch(variables, year_start, year_end)`: Process one batch
- `main()`: Parse arguments, run pipeline
- Command-line argument parsing

**Dependencies**: All other modules

---

## 🔧 Configuration Options

### Processing Parameters

```python
# In config.py

# Time batching
YEARS_PER_BATCH = 5              # Years per output file
START_YEAR = 1950                # Default start year
END_YEAR = 2024                  # Default end year

# Spatial sampling
SPATIAL_SAMPLE_EVERY_N = 10      # Sample every Nth grid cell (1 = all)

# Performance
CHUNK_SIZE = 10000               # Observations per chunk
MAX_MEMORY_GB = 8                # Memory limit

# Validation
VALIDATION_ENABLED = True        # Enable graph validation
```

### Command-Line Arguments

All configuration parameters can be overridden via command-line arguments in `pipeline.py`:

```bash
python pipeline.py \
  --variables tg rr \           # Select variables
  --start-year 2000 \           # Override start year
  --end-year 2010 \             # Override end year
  --batch-years 5 \             # Years per file
  --spatial-sample 10           # Spatial sampling rate
```

---

## 📊 Data Flow Diagram

```
NetCDF File (*.nc)
    │
    ├─→ xarray.open_dataset()
    │
    ├─→ Filter by time range
    │
    ├─→ Sample spatial grid
    │
    ├─→ Extract arrays: lat, lon, time, values
    │
    ↓
For each observation:
    │
    ├─→ Format coordinates (lat_lon)
    │
    ├─→ Format timestamp (ISO 8601)
    │
    ├─→ Create observation URI
    │
    ├─→ Create RDF triples:
    │   ├─→ Observation type
    │   ├─→ hasFeatureOfInterest → location URI
    │   ├─→ madeBySensor → sensor URI
    │   ├─→ observedProperty → CF standard name URI
    │   ├─→ hasResult → blank node
    │   │   ├─→ numericValue → literal
    │   │   └─→ unit → QUDT unit URI
    │   └─→ resultTime → xsd:dateTime
    │
    └─→ Add to ObservationCollection
    │
    ↓
Serialize to Turtle (.ttl)
```

---

## 🚀 Extensibility

### Adding New Variables

1. Add NetCDF file to `netcdf/` directory
2. Update `VARIABLE_MAPPINGS` in `config.py`:
   ```python
   "new_var": {
       "cf_standard_name": "new_property",
       "unit": "UNIT_CODE",
       "label": "Variable Label"
   }
   ```
3. Update `CF_TO_QUDT_UNITS` if needed
4. Run pipeline with `--variables new_var`

### Customizing URI Patterns

Edit URI generation functions in `config.py`:

```python
def get_observation_uri(variable_code, lat, lon, timestamp):
    # Custom pattern here
    return f"{OBSERVATION_NS}custom_{variable_code}_{lat}_{lon}_{timestamp}"
```

### Adding Metadata

Extend `add_observation()` in `rdf_builder.py`:

```python
def add_observation(graph, variable, lat, lon, timestamp, value, metadata=None):
    # ... existing code ...
    
    # Add custom metadata
    if metadata:
        graph.add((obs_uri, CUSTOM_NS.metadata, Literal(metadata)))
```

---

## 📝 Best Practices

1. **Always run test mode first** (`--test`) when changing configuration
2. **Validate output** after processing using built-in validation
3. **Monitor memory usage** when processing full grids
4. **Use spatial sampling** for large datasets to manage file size
5. **Batch processing** by year ranges for better organization

---

[← Back to Main README](../README.md)
