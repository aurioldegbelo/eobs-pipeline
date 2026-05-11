# Contributing, License & Support

This document contains information about contributing to the project, licensing terms, citation guidelines, and support resources.

---

## 🤝 Contributing

We welcome contributions to the E-OBS Climate Knowledge Graph Pipeline! Here's how you can help:

### Ways to Contribute

1. **Report Bugs**: Open an issue describing the bug and how to reproduce it
2. **Suggest Features**: Propose new features or enhancements via issues
3. **Improve Documentation**: Fix typos, clarify instructions, add examples
4. **Submit Code**: Fix bugs, add features, optimize performance
5. **Share Use Cases**: Tell us how you're using the pipeline

### Getting Started

1. **Fork the Repository**
   ```bash
   # Fork on GitHub, then clone your fork
   git clone https://github.com/YOUR-USERNAME/climate-kg-pipeline.git
   cd climate-kg-pipeline
   ```

2. **Set Up Development Environment**
   ```bash
   # Create virtual environment
   python -m venv .venv
   
   # Activate (Windows)
   .\.venv\Scripts\Activate.ps1
   
   # Activate (Linux/Mac)
   source .venv/bin/activate
   
   # Install dependencies
   pip install -r requirements.txt
   ```

3. **Create a Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. **Make Your Changes**
   - Follow existing code style
   - Add comments for complex logic
   - Update documentation as needed
   - Test your changes

5. **Test Your Changes**
   ```bash
   # Run installation test
   python test_installation.py
   
   # Run pipeline test
   python code/pipeline.py --test
   
   # Test your specific changes
   ```

6. **Commit and Push**
   ```bash
   git add .
   git commit -m "Description of your changes"
   git push origin feature/your-feature-name
   ```

7. **Submit a Pull Request**
   - Go to GitHub and create a pull request
   - Describe your changes clearly
   - Reference any related issues

### Code Style Guidelines

- **Python Style**: Follow PEP 8 conventions
- **Naming**: Use descriptive variable and function names
- **Comments**: Explain "why", not just "what"
- **Docstrings**: Use for all functions and classes
- **Type Hints**: Include where helpful

Example:
```python
def create_observation(
    graph: Graph,
    variable: str,
    lat: float,
    lon: float,
    timestamp: str,
    value: float
) -> URIRef:
    """
    Create a SOSA observation triple pattern.
    
    Args:
        graph: RDFLib graph to add triples to
        variable: Variable code (e.g., 'tg', 'rr')
        lat: Latitude in decimal degrees
        lon: Longitude in decimal degrees
        timestamp: ISO 8601 timestamp string
        value: Measured value
        
    Returns:
        URI of the created observation
    """
    # Implementation...
```

### Testing Requirements

Before submitting a pull request:

1. **Run existing tests**
   ```bash
   python test_installation.py
   python code/pipeline.py --test
   ```

2. **Add new tests** if you're adding features
   - Test edge cases
   - Test error handling
   - Test with different configurations

3. **Test with real data** when possible
   ```bash
   python code/pipeline.py --start-year 2020 --end-year 2020
   ```

### Documentation Updates

If your changes affect:
- **Usage**: Update [README.md](../README.md)
- **Architecture**: Update [ARCHITECTURE.md](ARCHITECTURE.md)
- **RDF Model**: Update [RDF_MODEL.md](RDF_MODEL.md)
- **Performance**: Update [PERFORMANCE.md](PERFORMANCE.md)
- **Queries**: Update [SPARQL_EXAMPLES.md](SPARQL_EXAMPLES.md)

---

## 📄 License

### MIT License

```
MIT License

Copyright (c) 2026 Climate Knowledge Graph Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Third-Party Licenses

This pipeline uses the following open-source libraries:

| Library | License | Purpose |
|---------|---------|---------|
| **RDFLib** | BSD-3-Clause | RDF graph construction and serialization |
| **xarray** | Apache-2.0 | NetCDF data manipulation |
| **netCDF4** | MIT | NetCDF file reading |
| **NumPy** | BSD-3-Clause | Numerical operations |
| **Pandas** | BSD-3-Clause | Data handling and timestamps |

### E-OBS Dataset License

The E-OBS dataset is provided by the ECA&D project and is subject to their terms of use:

- **Source**: https://www.ecad.eu/download/ensembles/download.php
- **Terms**: Free for research and educational purposes
- **Citation Required**: See [Data Citation](#data-citation) below
- **Commercial Use**: Contact ECA&D for licensing

**Note**: This pipeline software is MIT-licensed, but the E-OBS data you process has its own license terms.

---

## 📝 Citation

### Citing This Pipeline

If you use this pipeline in your research, please cite:

**BibTeX**:
```bibtex
@software{eobs_kg_pipeline_2026,
  title = {E-OBS Climate Knowledge Graph Pipeline},
  author = {{Climate Knowledge Graph Team}},
  year = {2026},
  version = {1.0.0},
  url = {https://github.com/your-org/climate-kg-pipeline},
  note = {NetCDF to RDF/Turtle Conversion System}
}
```

**APA**:
```
Climate Knowledge Graph Team. (2026). E-OBS Climate Knowledge Graph Pipeline 
(Version 1.0.0) [Computer software]. 
https://github.com/your-org/climate-kg-pipeline
```

**Chicago**:
```
Climate Knowledge Graph Team. "E-OBS Climate Knowledge Graph Pipeline." 
Version 1.0.0. Computer software. 2026. 
https://github.com/your-org/climate-kg-pipeline.
```

### Data Citation

When using E-OBS data, you **must** cite the original dataset:

**Primary Citation**:
```
Cornes, R.C., G. van der Schrier, E.J.M. van den Besselaar, and P.D. Jones. 2018: 
An Ensemble Version of the E-OBS Temperature and Precipitation Datasets. 
J. Geophys. Res. Atmos., 123, 9391-9409. 
https://doi.org/10.1029/2017JD028200
```

**BibTeX**:
```bibtex
@article{cornes2018ensemble,
  title={An ensemble version of the E-OBS temperature and precipitation data sets},
  author={Cornes, Richard C and Van der Schrier, Gerard and Van den Besselaar, Else JM and Jones, Philip D},
  journal={Journal of Geophysical Research: Atmospheres},
  volume={123},
  number={17},
  pages={9391--9409},
  year={2018},
  publisher={Wiley Online Library},
  doi={10.1029/2017JD028200}
}
```

### Ontology Citations

If you publish SPARQL queries or use the RDF model, consider citing the ontologies:

**SOSA/SSN**:
```
W3C Semantic Sensor Network Ontology (2017)
https://www.w3.org/TR/vocab-ssn/
```

**QUDT**:
```
Quantities, Units, Dimensions and Types (QUDT) Ontology
http://qudt.org/
```

**CF Standard Names**:
```
CF Standard Name Table (Version 85, 2024)
https://cfconventions.org/standard-names.html
```

---

## 📞 Support & Contact

### Getting Help

1. **Documentation**: Check the [README](../README.md) and other [documentation files](.)
2. **Troubleshooting**: See [TROUBLESHOOTING.md](../TROUBLESHOOTING.md)
3. **Issues**: Search existing [GitHub Issues](https://github.com/your-org/climate-kg-pipeline/issues)
4. **New Issue**: If your problem isn't covered, [open a new issue](https://github.com/your-org/climate-kg-pipeline/issues/new)

### Contact Information

- **GitHub**: https://github.com/your-org/climate-kg-pipeline
- **Email**: climate-kg@example.org
- **Discussion Forum**: [GitHub Discussions](https://github.com/your-org/climate-kg-pipeline/discussions)

### Reporting Security Issues

If you discover a security vulnerability, please:
1. **Do NOT** open a public issue
2. Email security@example.org with details
3. Allow time for us to address the issue before public disclosure

---

## 🔗 External References

### E-OBS Dataset

- **Download**: https://www.ecad.eu/download/ensembles/download.php
- **Documentation**: https://www.ecad.eu/documents/
- **ECA&D Project**: https://www.ecad.eu/
- **User Guide**: https://www.ecad.eu/documents/atbd.pdf

### W3C Standards

- **SOSA/SSN Ontology**: https://www.w3.org/TR/vocab-ssn/
- **RDF 1.1 Turtle**: https://www.w3.org/TR/turtle/
- **SPARQL 1.1**: https://www.w3.org/TR/sparql11-query/
- **RDF 1.1 Concepts**: https://www.w3.org/TR/rdf11-concepts/

### Ontologies & Vocabularies

- **CF Conventions**: http://cfconventions.org/
- **CF Standard Names**: http://vocab.nerc.ac.uk/standard_name/
- **QUDT**: http://qudt.org/
- **OBOE**: http://ecoinformatics.org/oboe/
- **WGS84 Geo Positioning**: http://www.w3.org/2003/01/geo/

### Tools & Libraries

- **RDFLib**: https://rdflib.readthedocs.io/
- **xarray**: https://xarray.pydata.org/
- **netCDF4-python**: https://unidata.github.io/netcdf4-python/
- **Apache Jena**: https://jena.apache.org/
- **Fuseki (SPARQL Server)**: https://jena.apache.org/documentation/fuseki2/

### Learning Resources

- **SPARQL Tutorial**: https://www.w3.org/TR/sparql11-query/
- **RDF Primer**: https://www.w3.org/TR/rdf11-primer/
- **Linked Data**: https://www.w3.org/DesignIssues/LinkedData.html
- **Climate Data**: https://climate.copernicus.eu/

---

## 🔄 Version History

### v1.0.0 (April 2026) - Production Release

**Major Features**:
- Simplified 8-prefix RDF model
- OBOE ObservationCollection support
- Streamlined SOSA observation pattern
- Merged variable processing (one file per time batch)
- Production-ready validation and error handling

**Improvements**:
- Comprehensive documentation
- Performance optimization (~1,200 obs/sec)
- Memory-efficient processing (<4 GB)
- Flexible spatial sampling

**Breaking Changes**: None (initial release)

### Future Roadmap

Potential features for future versions:

- **v1.1**: Add spatial geometry (WKT) support
- **v1.2**: Add sensor metadata triples
- **v1.3**: Add PROV-based provenance
- **v2.0**: Support for additional climate datasets
- **v2.1**: Interactive web interface
- **v2.2**: Docker containerization

---

## 🌟 Acknowledgments

This pipeline was developed with support from:

- **E-OBS Team**: For providing high-quality climate data
- **W3C**: For semantic web standards (SOSA/SSN, RDF, SPARQL)
- **Open-Source Community**: For excellent libraries (RDFLib, xarray, etc.)
- **Climate Research Community**: For feedback and use cases

---

## 📋 Known Assumptions & Limitations

### Data Assumptions

1. **E-OBS v31.0e Format**: Pipeline expects specific NetCDF structure (CF-1.6 compliant)
2. **Daily Temporal Resolution**: Designed for daily observations
3. **Regular Grid**: Assumes regular lat/lon grid structure (0.25° resolution)
4. **Ensemble Mean Data**: Uses ensemble mean files, not individual ensemble members
5. **Standard Variables**: Expects 7 standard E-OBS variables (tg, tn, tx, rr, hu, fg, qq)

### Current Limitations

1. **No Spatial Geometry Triples**
   - Location URIs are references only
   - No WKT geometry or lat/lon properties in graph
   - Users must know coordinates from external sources

2. **No Sensor Metadata**
   - Sensor URIs are references only
   - No sensor type, manufacturer, or calibration info
   - One simplified sensor per variable

3. **No Provenance**
   - Does not include PROV-based data lineage
   - No processing history in RDF
   - No dataset attribution within graph

4. **Simplified Metadata Model**
   - Focuses on core observation data
   - No collection-level metadata (creator, date, etc.)
   - No observation quality flags
   - No uncertainty or error estimates

5. **Memory Constraints**
   - Large full-grid processing requires significant RAM
   - Spatial sampling recommended for desktop systems
   - Full grid (sample=1) best suited for server environments

### Design Trade-offs

**Simplicity vs. Completeness**:
- Chose minimal 8-prefix model for clarity
- Could add more ontologies for richer semantics
- Current design prioritizes ease of use

**File Size vs. Detail**:
- Spatial sampling reduces file size
- Loses spatial resolution detail
- Trade-off acceptable for many use cases

**Performance vs. Metadata**:
- Simplified pattern improves processing speed
- Less metadata overhead
- Faster SPARQL queries with fewer triples

### Future Enhancements

These limitations could be addressed in future versions based on user needs. See [Future Roadmap](#future-roadmap).

---

## 💡 Best Practices for Contributors

1. **Start Small**: Begin with documentation fixes or small bug fixes
2. **Ask Questions**: Open an issue before starting major changes
3. **Test Thoroughly**: Ensure your changes don't break existing functionality
4. **Document Changes**: Update relevant documentation files
5. **Follow Standards**: Adhere to Python PEP 8 and RDF best practices
6. **Be Patient**: Code reviews may take time, but ensure quality

---

## 🎓 For Researchers

If you're using this pipeline for research:

1. **Cite both the pipeline and the E-OBS dataset** (see [Citation](#📝-citation))
2. **Document your configuration** in your methods section
3. **Share your SPARQL queries** to help others
4. **Report issues** if you encounter problems
5. **Consider contributing** examples of your use cases

---

**Thank you for your interest in the E-OBS Climate Knowledge Graph Pipeline!**

[← Back to Main README](../README.md)
