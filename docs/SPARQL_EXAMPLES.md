# SPARQL Query Examples

This document provides comprehensive SPARQL query examples for querying the E-OBS Climate Knowledge Graph.

---

## 📚 Query Basics

### Required Prefixes

All queries should include these prefix declarations:

```sparql
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#>
PREFIX observation: <http://climate-kg.org/observation/>
PREFIX qudt: <http://qudt.org/schema/qudt/>
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX unit: <http://qudt.org/vocab/unit/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
```

---

## 🔍 Basic Queries

### 1. Find All Observations for a Specific Location

Get all observations (any variable) for a single grid cell:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?property ?value ?time WHERE {
    ?obs sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
         sosa:observedProperty ?property ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
}
ORDER BY ?time
```

**Returns**: All observations at location (50.00°N, 3.00°E) with their property, value, and timestamp

### 2. Find All Temperature Observations

Get all observations where the property is air temperature:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?value ?time WHERE {
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
}
LIMIT 100
```

**Returns**: First 100 temperature observations with values and timestamps

### 3. Find Observations by Sensor

Get all observations from a specific sensor (e.g., precipitation):

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?value ?time ?location WHERE {
    ?obs sosa:madeBySensor sosa:rr_sensor ;
         sosa:hasFeatureOfInterest ?location ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
}
LIMIT 100
```

**Returns**: Precipitation observations with values, timestamps, and locations

---

## 📊 Filtering & Aggregation

### 4. Find Temperature Observations Above Threshold

Find all temperature values above 25°C:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?temp ?time ?location WHERE {
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasFeatureOfInterest ?location ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?temp .
    FILTER(?temp > 25.0)
}
ORDER BY DESC(?temp)
```

**Returns**: Hot temperature observations, sorted by value (hottest first)

### 5. Get Observations for a Time Range

Find all observations in January 2020:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX qudt: <http://qudt.org/schema/qudt/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?obs ?property ?value ?time WHERE {
    ?obs sosa:observedProperty ?property ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
    FILTER(?time >= "2020-01-01T00:00:00"^^xsd:dateTime &&
           ?time <= "2020-01-31T23:59:59"^^xsd:dateTime)
}
ORDER BY ?time
```

**Returns**: All observations in January 2020, sorted by time

### 6. Average Temperature by Location

Calculate average temperature for each location:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?location (AVG(?temp) AS ?avgTemp) (COUNT(?obs) AS ?count) WHERE {
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasFeatureOfInterest ?location ;
         sosa:hasResult ?result .
    ?result qudt:numericValue ?temp .
}
GROUP BY ?location
ORDER BY DESC(?avgTemp)
```

**Returns**: Average temperature and observation count per location, sorted by temperature

### 7. Count Observations per Variable

Count how many observations exist for each climate variable:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT ?property (COUNT(?obs) AS ?count) WHERE {
    ?obs sosa:observedProperty ?property .
}
GROUP BY ?property
ORDER BY DESC(?count)
```

**Returns**: Number of observations per property (variable)

---

## 🗂️ Collection Queries

### 8. List All ObservationCollections

Find all observation collections in the graph:

```sparql
PREFIX oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#>

SELECT ?collection WHERE {
    ?collection a oboe:ObservationCollection .
}
```

**Returns**: URIs of all collections (e.g., `observation:eobs_tg_2020_2024`)

### 9. Count Observations in Each Collection

Count members in each collection:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#>

SELECT ?collection (COUNT(?obs) AS ?count) WHERE {
    ?collection a oboe:ObservationCollection ;
                sosa:hasMember ?obs .
}
GROUP BY ?collection
ORDER BY ?collection
```

**Returns**: Collection URIs with member counts

### 10. Get All Observations from a Collection

Retrieve all observations from a specific collection:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX oboe: <http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#>
PREFIX observation: <http://climate-kg.org/observation/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?value ?time WHERE {
    observation:eobs_tg_2020_2024 sosa:hasMember ?obs .
    ?obs sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
}
ORDER BY ?time
LIMIT 100
```

**Returns**: First 100 observations from the temperature collection

---

## 🌡️ Advanced Queries

### 11. Find Coldest and Hottest Days

Find the coldest and hottest temperature observations:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

# Hottest
SELECT ?obs ?temp ?time ?location WHERE {
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasFeatureOfInterest ?location ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?temp .
}
ORDER BY DESC(?temp)
LIMIT 10
```

For coldest, change to `ORDER BY ?temp` (ascending).

### 12. Find Heavy Precipitation Events

Find precipitation observations above 50mm:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?obs ?precip ?time ?location WHERE {
    ?obs sosa:observedProperty cf:precipitation_amount ;
         sosa:hasFeatureOfInterest ?location ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?precip .
    FILTER(?precip > 50.0)
}
ORDER BY DESC(?precip)
```

**Returns**: Heavy precipitation events, sorted by amount

### 13. Time Series for a Specific Location

Get a time series of all variables for one location:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?time ?property ?value ?unit WHERE {
    ?obs sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
         sosa:observedProperty ?property ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value ;
            qudt:unit ?unit .
}
ORDER BY ?time ?property
```

**Returns**: Multi-variable time series for location (50.00°N, 3.00°E)

### 14. Compare Two Locations

Compare temperature between two grid cells:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>

SELECT ?time ?temp1 ?temp2 WHERE {
    ?obs1 sosa:observedProperty cf:air_temperature ;
          sosa:hasFeatureOfInterest geo1:grid_50p00_3p00 ;
          sosa:resultTime ?time ;
          sosa:hasResult ?result1 .
    ?result1 qudt:numericValue ?temp1 .
    
    ?obs2 sosa:observedProperty cf:air_temperature ;
          sosa:hasFeatureOfInterest geo1:grid_51p00_4p00 ;
          sosa:resultTime ?time ;
          sosa:hasResult ?result2 .
    ?result2 qudt:numericValue ?temp2 .
}
ORDER BY ?time
```

**Returns**: Side-by-side temperature comparison for the same timestamps

### 15. Monthly Temperature Averages

Calculate monthly average temperatures (requires SPARQL 1.1):

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX qudt: <http://qudt.org/schema/qudt/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT 
    (YEAR(?time) AS ?year) 
    (MONTH(?time) AS ?month) 
    (AVG(?temp) AS ?avgTemp) 
    (COUNT(?obs) AS ?count)
WHERE {
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?temp .
}
GROUP BY (YEAR(?time)) (MONTH(?time))
ORDER BY ?year ?month
```

**Returns**: Monthly average temperatures with observation counts

---

## 🛠️ Validation Queries

### 16. Check for Missing Properties

Find observations missing required properties:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT ?obs WHERE {
    ?obs a sosa:Observation .
    FILTER NOT EXISTS { ?obs sosa:hasResult ?result }
}
```

This will return observations that don't have a result value (should be empty in a valid graph).

### 17. Count Observations by Type

Verify all observations have correct type:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT (COUNT(?obs) AS ?count) WHERE {
    ?obs a sosa:Observation .
}
```

**Returns**: Total number of SOSA observations in the graph

### 18. List All Sensors

Find all unique sensors in the graph:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT DISTINCT ?sensor WHERE {
    ?obs sosa:madeBySensor ?sensor .
}
```

**Returns**: List of sensor URIs (should be 7 sensors for 7 variables)

### 19. List All Observed Properties

Find all unique climate properties:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT DISTINCT ?property WHERE {
    ?obs sosa:observedProperty ?property .
}
```

**Returns**: List of CF Standard Name URIs used in the graph

### 20. List All Locations

Find all unique grid cell locations:

```sparql
PREFIX sosa: <http://www.w3.org/ns/sosa/>

SELECT DISTINCT ?location WHERE {
    ?obs sosa:hasFeatureOfInterest ?location .
}
```

**Returns**: List of grid cell URIs

---

## 💡 Tips for Writing SPARQL Queries

### Performance Tips

1. **Use LIMIT** for exploratory queries to avoid retrieving millions of results
2. **Filter early** in the query pattern to reduce intermediate results
3. **Use specific URIs** when possible (e.g., `geo1:grid_50p00_3p00`) rather than variables
4. **Index suggestions** for triple stores: index on `sosa:resultTime`, `sosa:observedProperty`, `sosa:hasFeatureOfInterest`

### Common Patterns

**Pattern: Get observation metadata**
```sparql
?obs sosa:hasFeatureOfInterest ?location ;
     sosa:observedProperty ?property ;
     sosa:madeBySensor ?sensor ;
     sosa:resultTime ?time .
```

**Pattern: Get result value**
```sparql
?obs sosa:hasResult ?result .
?result qudt:numericValue ?value ;
        qudt:unit ?unit .
```

**Pattern: Filter by variable type**
```sparql
?obs sosa:observedProperty cf:air_temperature .
# OR
?obs sosa:madeBySensor sosa:tg_sensor .
```

### Troubleshooting

**No results returned?**
- Check your prefix declarations
- Verify URIs are correctly formatted (case-sensitive)
- Use `LIMIT 10` to see if any results exist
- Try simpler queries first, then add filters

**Too many results?**
- Add `LIMIT` clause
- Add time range filters
- Filter by specific location or property
- Use aggregation (AVG, COUNT, etc.)

---

## 📝 Query Template

Here's a template for creating custom queries:

```sparql
# Prefix declarations
PREFIX cf: <http://vocab.nerc.ac.uk/standard_name/>
PREFIX geo1: <http://www.w3.org/2003/01/geo/wgs84_pos#>
PREFIX qudt: <http://qudt.org/schema/qudt/>
PREFIX sosa: <http://www.w3.org/ns/sosa/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# SELECT clause - what to return
SELECT ?obs ?value ?time WHERE {
    
    # Triple patterns - what to match
    ?obs sosa:observedProperty cf:air_temperature ;
         sosa:hasResult ?result ;
         sosa:resultTime ?time .
    ?result qudt:numericValue ?value .
    
    # Filters - conditions
    FILTER(?time >= "2020-01-01T00:00:00"^^xsd:dateTime)
    FILTER(?value > 20.0)
}
# Ordering and limits
ORDER BY ?time
LIMIT 100
```

---

## 🔗 External Resources

- **SPARQL 1.1 Specification**: https://www.w3.org/TR/sparql11-query/
- **SOSA/SSN Ontology**: https://www.w3.org/TR/vocab-ssn/
- **CF Standard Names**: http://vocab.nerc.ac.uk/standard_name/
- **QUDT Units**: http://qudt.org/vocab/unit/

---

[← Back to Main README](../README.md)
