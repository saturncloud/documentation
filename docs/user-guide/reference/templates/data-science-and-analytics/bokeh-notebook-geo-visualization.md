# Bokeh Notebook — Geo Visualization

![Bokeh Geo Visualization Preview](/images/bokeh-notebook/bokeh-interactive-dashboard-hero.png "doc-image")

## Overview

This template provides a production-ready setup for geospatial data analysis and interactive mapping on Saturn Cloud. Optimized for CPU resources, it demonstrates how to process geographic coordinates and render high-performance interactive maps using **Bokeh**.

Key features of Bokeh include:
*   High Interactivity
*   Browser-Based Rendering
*   Handles Large Datasets
*   Geospatial Mapping
*   Integration

## Why use this template?

Quickly build geospatial projects with built-in CRS transformations and interactive rendering. This Bokeh and GeoPandas foundation is ideal for mapping logistics, incidents, or environmental datasets out of the box.

## Getting Started

1. Simply log in to Saturn Cloud.
2. Click on the **dashboard** in the top-left sidebar.
3. Scroll down to the **Templates & Starter Kits** section 
4. Select the DataScience category
5. Click on **bokeh notebook**

![Select Bokeh Notebook from Dashboard](/images/bokeh-notebook/bokeh-dashboard-selection.png "doc-image")

6. In the **Create Resource from Template** modal, click **Submit**.

![Submit Bokeh Template Resource](/images/bokeh-notebook/bokeh-template-submit.png "doc-image")

7. After clicking on the **Submit** button, you will be taken to the **Workspace** page where all the configuration about the resource is located. Under the overview tab, click on the **Start** button to start the template.

![Start Template Resource](/images/bokeh-notebook/bokeh-start-template.png "doc-image")

8. The resource takes a few minutes to start and provision all the **necessary software**. The software **bar** will gradually fill up at the bottom of the resource information box. 

![Resource Provisioning](/images/bokeh-notebook/bokeh-resource-provisioning.png "doc-image")

9. Once the template resource is running, you can **click** on the **JupyterLab** tab to open the notebook and begin your analysis.

![Open JupyterLab](/images/bokeh-notebook/bokeh-open-jupyter.png "doc-image")

*Note: A new tab will open with the notebook, as seen in the image below.*

![JupyterLab Interface](/images/bokeh-notebook/bokeh-jupyterlab-interface.png "doc-image")

10. In JupyterLab, double-click the notebook file in the left sidebar to open it. Then, press **Run All** in the top menu to execute the code and view the interactive map.

![Run All Notebook Cells](/images/bokeh-notebook/bokeh-run-all-cells.png "doc-image")

11. Once the execution completes, the interactive Bokeh map will render directly in the notebook for you to explore.

![Final Bokeh Output](/images/bokeh-notebook/bokeh-final-output.png "doc-image")

## What to Expect in the Notebook

The included notebook is self-contained and guides you through a complete geospatial workflow:

1.  **Dependency Management:** Installing specialized libraries (`bokeh`, `geopandas`, `shapely`) into your active environment.
2.  **Data Ingestion & Preparation:** Initializing a GeoDataFrame and projecting coordinates into the **Web Mercator (EPSG:3857)** format required for web-based tiles.
3.  **Data Cleaning:** Managing spatial columns to avoid serialization errors during rendering.
4.  **Interactive Rendering:** Building a Bokeh figure with **OpenStreetMap** background tiles and custom **HoverTool** configurations.

## Troubleshooting & Best Practices

*   **Serialization Errors:** If you encounter a `SerializationError`, ensure you have dropped or converted the `geometry` column before passing the DataFrame to a Bokeh `ColumnDataSource`.
*   **Map Projection:** Bokeh maps require coordinates in Web Mercator meters. Always use `gdf.to_crs("EPSG:3857")` for your spatial data if using standard tile providers like OSM.
*   **Deprecation Warnings:** You may see a warning about `circle()` being deprecated. For future-proofing, you can use `p.scatter(size=...)` instead.

    ![Bokeh Deprecation Warning](/images/bokeh-notebook/bokeh-deprecation-warning.png "doc-image")

## Key Features

*   **Interactive Hover Filters:** Inspect localized data points dynamically with custom tooltips.
*   **High Performance:** Optimized to handle coordinate transformations and rendering on standard CPU resources.
*   **Standard Tech Stack:** Built with Python, GeoPandas for spatial logic, and Bokeh for web-ready interactivity.
*   **Ready for Production:** A benchmark for spatial joins and coordinate system management.

## Conclusion

This template demonstrates how to quickly build interactive geospatial visualizations on Saturn Cloud. By leveraging the power of GeoPandas and Bokeh, you can transform raw coordinates into insightful, interactive maps that are ready for deeper analysis or stakeholder presentations.

## Resources and Support

*   **Platform:** [Saturn Cloud Dashboard](https://app.saturncloud.io/)
*   **Support:** [Saturn Cloud Documentation](/docs/)
*   **Library:** [Bokeh Documentation](https://docs.bokeh.org/)
*   **Library:** [GeoPandas Documentation](https://geopandas.org/)
