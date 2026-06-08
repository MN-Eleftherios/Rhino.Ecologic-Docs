
# *RHINO.ECOLOGIC®* Grasshopper components overview 

*RHINO.ECOLOGIC®* is an advanced ecological simulation framework for Rhino and Grasshopper, designed to seamlessly integrate ecological and environmental knowledge into the architectural and design process. It enables users to model and ecologically analyze architectural projects. Outputs include location- and time-specific 3D species distribution maps, as well as biomass and biodiversity simulations.

This page provides an overview of the Grasshopper components included in the *RHINO.ECOLOGIC®* framework. Here, you will find descriptions of project components, parameters, data and analysis components, as well as utilities and preview tools, each detailing its function and role within broader workflows. Under the debug components section, you’ll find a few tools used during the development process for debugging purposes (debug version).

## Logo
<img src="assets/Rhino.Ecologic.Logo/Rhino.Ecologic Icons_VV_Logo_2.svg" alt="Alt text" width="180">   or     <img src="assets/Rhino.Ecologic.Logo/Rhino.Ecologic Icons_VV_Logo_1.svg" alt="Alt text" width="180"> 

---
## 00 Parameters:

 | GH Icon    | Feature | Category/ Subcategory | Description |
|--------- |------------------------------|---------------|---------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_00_Project.svg" alt="Alt text" width="150">    |  *Project (P)* |  Rhino.Ecologic® / Parameters | Represents a Rhino.Ecologic project. The component stores and passes the data of your Rhino.Ecologic project.
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_00_Voxel.svg" alt="Alt text" width="150">   |  *Voxel (V)* | Rhino.Ecologic® / Parameters  | Represents a space boundary with specific attributes. This component stores and passes all data associated with each voxel in your Rhino.Ecologic project.
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_00_Plant.svg" alt="Alt text" width="150">   |  *Plant (PL)* | Rhino.Ecologic® / Parameters  | Represents a description of a plant.
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_00_Plant_library.svg" alt="Alt text" width="150">   |  *Plant Library (PLL)* | Rhino.Ecologic® / Parameters  | Represents a set of plants comprising a library.
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_00_Soil_Set.svg" alt="Alt text" width="150">   |  *Soil Set (S)* | Rhino.Ecologic® / Parameters  | Represents a set of soils that can provide a nuturing environment for a plant.

---
## 01 Project components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_01_Create_Project.svg" width="150">  |  *Create Project (CP)* | Rhino.Ecologic® / Project | Constructs a new Rhino.Ecologic project for analysis. The components converts geometry to mesh, voxelizes the bounding volume, and separates the graph into subgraphs such as interior and exterior envelopes.object. |Title, Location, Longitude, Latitude, Geometry, Biophilic Area, Voxel Size|Rhino.Ecologic Project|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_01_Import_Project.svg" width="150">  |  *Import Project (IP)* | Rhino.Ecologic® / Project | Imports a project from a binary (.bph) file. Json deserialization is supported in the debug version.|Project Path|Rhino.Ecologic Project|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_01_Export_Project.svg" width="150">  |  *Save Project (SP)* | Rhino.Ecologic® / Project | Saves a project as a binary (.bhp) file. Json serialization is supported in the debug version. |Project, Save Directory, Serialize |Rhino.Ecologic Project|


---
## 02 Analysis components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_02_Run_Eco_Analysis.svg" alt="Alt text" width="150">   |  *Run Ecological Analysis (EA)* | Rhino.Ecologic®/ Analysis | Runs a configurable pipeline of analysis. This pipeline includes solar, soil, precipitation and biodiversity analysis, allowing for configuratiion (which analyses are run). |Project, Parameters, Run, Custom Library |Rhino.Ecologic Project, Log|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_02_Biodiversity_Parameters.svg" alt="Alt text" width="150">    |  *Biodiversity Parameters (BP)* | Rhino.Ecologic®/ Analysis | Creates a set of parameters for the biodiversity analysis. |Simulation Duration, Strata Number, Starting Population Size, Initial Seed Count, Soil Capacity, Rain Seed, Light Factor, Diffusion, Does Dispersal, Does Soil Class, Does Soil Depth, Does Regional Model, Does Disturbance, Fix RNG| Parameters|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_02_Solar_Parameters.svg" alt="Alt text" width="150">    |  *Solar Parameters (SP)*|  Rhino.Ecologic®/ Analysis |Creates a set of customized analysis parameters to configure the solar analysis execution including radiation and daylight analysis. |Hour Interval, Annual Frequency, Noon Hour, Day of Year, Plot Scale, Accumulative, Analysis Type, Analysis Span, Max Voxel Count, Max Depth, Min Surface Area| Parameters|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_02_Soil_Depth_Parameters.svg" alt="Alt text" width="150">    |  *Soil Depth Parameters* (SDP)| Rhino.Ecologic®/ Analysis | Creates a set of customized analysis parameters to configure the soil depth analysis execution. | Lower Height Depth | Parameters|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_02_Run_Data_Analysis.svg" alt="Alt text" width="150">    | *Run Data Analysis* (DA)|  Rhino.Ecologic®/ Analysis | Runs the data analysis pipeline and produces visualizations. |Rhino.Ecologic Project, Run|Bitmaps, Vector, Log|

---
## 03 Data components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_03_Deconstruct_Project.svg" alt="Alt text" width="150">   |  *Deconstruct Project* |Rhino.Ecologic®/ Data | Deconstructs a project object to its constituent parts: Geometry-, solar-, precipitation-, soil-, and biodiversity related attributes. It only filters and outputs the voxels set by the user, e.g. only the exterior envelope voxels. |Rhino.Ecologic Project|Title, Id, Location, Longitude, Latitude, Elevation, North, Timestamp, Voxels, Mesh|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_03_Deconstruct_Voxel.svg" alt="Alt text" width="150">    |  *Deconstruct Voxel* | Rhino.Ecologic®/ Data |Deconstructs a voxel to its constituent parts. |Voxel, Time Step|Id, Mesh, Position, Size, Inclination, Solar, Soil Depth, Soil Volume, Soil Type, Precipitation, Timestamp, Species, Biomass, Abundance, Plant Volume|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_03_Set_Project_Data.svg" alt="Alt text" width="150">   |  *Set Project Data*| Rhino.Ecologic®/ Data | Sets the value of the selected project field. | Project, Parameter, Value | Rhino.Ecologic Project |
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_03_Set_Voxel_Data.svg" alt="Alt text" width="150">    | *Set Voxel Data*| Rhino.Ecologic®/ Data |Sets the value of the selected voxel field. | Voxels, Analysis Type, Analysis Values  | Voxels  |

---
## 04 Visualization components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_04_Visualize_Analysis.svg" alt="Alt text" width="150">   |  *Visualize Analysis (VA)* | Rhino.Ecologic®/ Visualization | Visualize the data of the analysis. |Project, Voxel Type, Attribute, Time Step, Show |Visualization|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_04_Graph_Display.svg" alt="Alt text" width="150">    |  *Graph Display (GD)* | Rhino.Ecologic®/ Visualization  | Visualizes graphs in the form of a bitmap or svg. |Project Data| Graph Display|

---
## 05 Utilities components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Annual_Sun_Positions.svg" alt="Alt text" width="150">   |  *Get Annual Sun Positions (ASP)* | Rhino.Ecologic®/ Utilities | Calculates the sun positions of the location specified in the project. |Project, Annual Frequency, Hour Interval |Sun Positions|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Daily_Sun_Positions.svg" alt="Alt text" width="150">    |  *Get Daily Sun Positions (DSP)* | Rhino.Ecologic®/ Utilities  | Gets the sun positions of the location specified in the project on the specific day. |Project, Day | Sun Positions|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Biophilic_Voxels.svg" alt="Alt text" width="150">    |  *Get Biophilic Voxels (GBV)* | Rhino.Ecologic®/ Utilities  | Gets voxels marked as biophilic. |Project, Day | Voxels|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Plant_Voxels.svg" alt="Alt text" width="150">    |  *Get Plant Voxels (GPV)* | Rhino.Ecologic®/ Utilities  | Gets all the voxels that have valid biomass values. |Project, Time Step | Voxels, Time Step|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Voxel_Value.svg" alt="Alt text" width="150">    |  *Get Voxel Value (GVV)* | Rhino.Ecologic®/ Utilities  | Gets a specific analysis value of the voxel. |Voxel, Analysis Type, Time Step | Value|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_05_Get_Voxel_By_Type.svg" alt="Alt text" width="150">    |  *Get Voxels By Type (SVD)* | Rhino.Ecologic®/ Utilities  | Sets the value of the selected voxel field. |Voxel, Analysis Type, Analysis Values | Voxels|

---
## 06 Statistics components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Biodiversity_Summaries.svg" alt="Alt text" width="150">   |  *Get Biodiversity Summaries (BS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the biodiversity attributes of a project and returns its summary statistics. |Project|Minimum, Maximum, Average, Median|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Inclination_Summaries.svg" alt="Alt text" width="150">    |  *Get Inclination Summaries (IS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the inclination attributes of a project and returns its summary statistics. |Project | Minimum, Maximum, Average, Median|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Precipitation_Summaries.svg" alt="Alt text" width="150">    |  *Get Precipitation Summaries (PS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the preciptitaion attributes of a project and returns its summary statistics. |Project | Minimum, Maximum, Average, Median|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Soil_Summaries.svg" alt="Alt text" width="150">    |  *Get Soil Summaries (SS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the soil attributes of a project and returns its summary statistics. |Project | Minimum Soil Depth, Maximum Soil Depth, Average Soil Depth, Median Soil Depth, Minimum Soil Volume, Maximum Soil Volume, Average Soil Volume, Median Soil Volume, Soil Type Distribution (Types), Soil Type Distribution (Count) |
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Solar_Summaries.svg" alt="Alt text" width="150">    |  *Get Solar Summaries (SS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the solar attributes of a project and returns its summary statistics. |Project | Minimum Soil Depth, Maximum, Average, Median|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_06_Get_Voxel_Summaries.svg" alt="Alt text" width="150">    |  *Get Voxel Summaries (VS)* | Rhino.Ecologic®/ Statistics | Performs descriptive analysis on the voxel attributes of a project and returns its summary statistics. |Project | Biophilic Areas Count, Type Distribution (Types), Type Distribution (Count), Voxel Index, Degree Centrality|


---
## 07 Species Library components:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Compute_Preset_Values.svg" alt="Alt text" width="150">   |  *Compute Preset Values (CPV)* | Rhino.Ecologic®/ Species Library | Calculates values for a plant definition provided presets.  |Life Span, Growth Form, Leaf Form, Shade Tolerance |Maturation Time, Rooting Depth, Size, Opacity, Shade Tolerance|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Create_Plant.svg" alt="Alt text" width="150">    |  *Create Plant (CPL)* | Rhino.Ecologic®/ Species Library  | Creates a plant definition. |Name, Max Geight, Width, Crown Width, Life Span, Growth Form, Maturation Time, Rooting Depth, Size, Soil Set, Germination Rate, Opacity, Conversion Efficiency, Shade Tolerance| Plant|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Create_Plant_Lib.svg" alt="Alt text" width="150">    |  *Create Plant Library (CPLL)* | Rhino.Ecologic®/ Species Library  | Creates a plant library provided a series of plant definitions. |Name, Plants| Plant Library|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Create_Soil_Set.svg" alt="Alt text" width="150">    |  *Create Soil Set (CSS)* | Rhino.Ecologic®/ Species Library  | Creates a soil set for a plant definition. |Accepted Soils| Soil Set|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Deconstruct_Plant.svg" alt="Alt text" width="150">    |  *Deconstruct Plant (DPL)* | Rhino.Ecologic®/ Species Library  | Deconstructs a plant to its consituent parts. |Plant | Name, Max Height, Life Span, Maturation Time, Rooting Depth, Size, Soil Set, Dormancy, Dormancy Break Rate, Germination Success, Max Biomass, Density, Biomass Allocation |
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Deconstruct_Plant_Lib.svg" alt="Alt text" width="150">    |  *Deconstruct Plant Library (DPLL)* | Rhino.Ecologic®/ Species Library  | Soils contained in the soil set. |Project Data| Graph Display|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Deconstruct_Soil_Set.svg" alt="Alt text" width="150">    |  *Deconstruct Soil Set (DSS)* | Rhino.Ecologic®/ Species Library  | Visualizes graphs in the form of a bitmap or svg. |Soil Set| Soils|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Export_Plant_Lib.svg" alt="Alt text" width="150">    |  *Export Plant Library (EPLL)* | Rhino.Ecologic®/ Species Library  | Imports a plant library. |Library Path| Plant Library|
|  <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_07_Import_Plant_Lib.svg" alt="Alt text" width="150">    |  *Save Plant Library (SPLL)* | Rhino.Ecologic®/ Species Library  | Saves a plant library to a file. |Plant Library, Save Directory, Save| - |

---

## Debug:

| GH Icon    | Feature | Category/ Subcategory | Description |Input parameters |Output|
|--------- |------------------------------|---------------|---------------|------------------------------|------------------------------|
| <img src="assets/Rhino.Ecologic.Icons/Rhino.Ecologic Icons_VV_08_Debug.svg" alt="Alt text" width="150">   |  *-* | xx | xx |xx|





---
`old`

### *Debug components*
| GH Icon     | Feature | Description |Input parameters |Output parameters |
|--------- |------------------------------|------------------------------|------------------------------|------------------------------|
| <img src="GH_Icons/Rhino.Ecologic_Debug.svg" alt="Alt text" width="150">   |  *Variable Parameter Component* | Used to test and debug variable parameter component features.
| <img src="GH_Icons/Rhino.Ecologic_Debug.svg" alt="Alt text" width="150">   | *Serialize Mesh*|  Serializes a mesh, using Rhino's default mesh serialization functionality.
| <img src="GH_Icons/Rhino.Ecologic_Debug.svg" alt="Alt text" width="150">   | *Deserialize Mesh*| Deserializes an encoded string to a mesh object, using Rhino's default mesh serialization functionality.
| <img src="GH_Icons/Rhino.Ecologic_Debug.svg" alt="Alt text" width="150">   | *Get Envelope Mesh*| Gets the envelope mesh, by shrink wrapping (target length 0.1) and quad remeshing (target length 0.5) the provided meshes.
| <img src="GH_Icons/Rhino.Ecologic_Debug.svg" alt="Alt text" width="150">   | *Run Rhino.Compute Definition*| Runs an analysis definition using Rhino.Compute.

### *Experimental components*
| GH Icon     | Feature | Description |Input parameters |Output parameters |
|--------- |------------------------------|------------------------------|------------------------------|------------------------------|
| ![Deconstruct Project VP](xx.svg)   | *Deconstruct Project VP*| Deconstructs a project to its constituent parts, having variable output pins that can be added or removed.
| ![Deconstruct Voxel VP](xx.svg)   | *Deconstruct Voxel VP*| Runs the analysis pipeline on a different thread (not blocking the UI thread) and reports the progress of the pipeline.
| ![Run Analysis Async](xx.svg)   | *Run Analysis Async*| Runs the data analysis pipeline and produces visualizations.
| ![Run Surrogate Analysis](xx.svg)   | *Run Data Analysis*| Runs the inference on the trained ML model and outputs the predicted project values.

### *Extra*
| GH Icon     | Feature | Description |Input parameters |Output parameters |
|--------- |------------------------------|------------------------------|------------------------------|------------------------------|
|<img src="GH_Icons/Rhino.Ecologic_Precipitation.svg" alt="Alt text" width="100">   |  *Precipitation* | Custom definition of precipitation.
|<img src="GH_Icons/Rhino.Ecologic_Parameter.svg" alt="Alt text" width="100">   |  *Param* | Custom definition of parameters.
|<img src="GH_Icons/Rhino.Ecologic_Soil_Type.svg" alt="Alt text" width="100">   |  *Soil Type* | Custom definition of soil type.
|<img src="GH_Icons/Rhino.Ecologic_Soil_Depth.svg" alt="Alt text" width="100">   |  *Soil Depth* | Custom definition of soil depth.

&copy; 2025 McNeel Europe S.L. All rights reserved.



