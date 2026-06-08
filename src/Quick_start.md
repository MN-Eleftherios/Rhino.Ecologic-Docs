# Quick-start guide for *Rhino.Ecologic®* <img src="assets/Rhino.Ecologic.Logo/Rhino.Ecologic Icons_VV_Logo_2.svg" alt="Alt text" width="100">   

**Version**: 0.1.7 alpha  
**Compatible with**: Rhino 8+, Grasshopper  
**Platform**: Windows  
**Release date**: xx 2026 <br>
**Draft**: Verena Vogler

*Rhino.Ecologic®* is an advanced ecological simulation framework for Rhino and Grasshopper, designed to seamlessly integrate ecological and environmental knowledge into the architectural and design process. It enables users to model and ecologically analyze architectural projects. Outputs include location- and time-specific 3D species distribution maps, as well as biomass and biodiversity simulations.

This page provides quick start guidelines for the *Rhino.Ecologic* framework. 

---

## Step 1: Launch Rhino & Grasshopper
1. Open **Rhino 8**.<br>

2. Go to **File > Open** in Rhino.<br>

3. Open one of the sample `.3dm` files located in the `/examples` directory. The file is named *“Rhino.Ecologic_Example_File.3dm”* and here: https://drive.google.com/drive/folders/1-BJFjhQ7BCHT9mP9yyBOuZhKPZ1J2fXm<br>

4. Start **Grasshopper** by typing `_Grasshopper` in the Rhino command line.<br>

5. Ensure the **Rhino.Ecologic** tab is visible in the Grasshopper ribbon.<br>

 <div align="center"><img src="assets/images/Sceenshots/Toolbar_Rhino.Ecologic.png" alt="Alt text" width="1000">  <br>
*Loaded Rhino.Ecologic® toolbar in Grasshopper.*</div><br>

 The Rhino.Ecologic toolbar includes tabs such as:

- 00_Parameters
- 01_Project
- 02_Analysis
- 03_Data
- 04_Visualisations
- 05_Utilities
- 06_Statistics
- 07_Species Library

> **Tip:** For more information on the Rhino.Ecologic® User Interface, see User Guide, Chapter 1 ([`link`]). <br>

---

## Step 2: Load an example file

1. Go to **File > Open** in Grasshopper.<br>

2. Open one of the example `.gh` files located in the `/examples` directory. The file is named *“Rhino.Ecologic_Example definition.gh”* and here: https://drive.google.com/drive/folders/1-BJFjhQ7BCHT9mP9yyBOuZhKPZ1J2fXm<br><br>
   
---
 
## Step 3: Set-up your Rhino.Ecologic Project

1. Set-up your `Rhino.Ecologic Project(P)` by referencing your project in the `Create Project` Grasshopper component.<br>

2. Set the project parameters such as the project's `Title(T)`, the project's geographic `Location (L)` including `Longitude(Lon)` and `Latitude(Lat)`, and define the `Voxel Size(VS)`.<br>

3. Reference your Rhino geometry (e.g., building, terrain 3D model) by right-clicking the `Geometry Parameter` component and selecting `Set one geometry` or `Set multiple geometries`. You can also input your Grasshopper model.<br>

4. To run the environmental and ecological analysis on the entire project, leave the  `Biophilic Areas(BA)` input empty.<br>

    <div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_00_Create_Project.gif" alt="Alt text" width="1000"></div><br>

5. To analyze only specific regions of your project, reference a `bounding box` (or multiple boxes) around those areas to the `Biophilic Areas` input.<br>

<div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_00_Create_Project_2.gif" width="1000"></div><br><br>

---

## Step 4: Run ecological analysis 

1. To run the **ecological analysis**, use the default settings for `Parameters` and `Custom Library` in the `Run Ecological Analysis` component. <br>
      
2. To start the environmental and ecological analysis pipeline, set the `Run` boolean toggle from `False` (analysis inactive) to `Tru`e (analysis active).<br>
    
3. The `Run Ecological Analysis` component outputs data associated with geometry, environmental, and ecological parameters, including `Inclination`, `Soil Depth`, `Soil Volume`, `Soil Type`, `Solar`, and `Biodiversity` analysis results.<br>

4. Review the `Log` output to ensure the analyses finished successfully.<br>

<div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_03_Run_Eco_Analysis_1.gif" width="1000"></div><br>
   
5. Run a custom environmental and ecological analysis for your project using a custom `Plant Library`. The `Plant Library` is a JSON file. The JSON files for different geographic locations can be downloaded here: https://drive.google.com/drive/folders/1Z-OJwC5NY61Qaf6LhKkmaPWLYdzkRh0O <br>

6. Reference the JSON files using the **absolute file path**.

<div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_03_Run_Eco_Analysis_2.gif" width="1000"></div><br><br>
  
---

## Step 5: Data outputs 

1. Components in the **Data tab** return environmental and ecological data at the predefined resolution (`Voxel Size`). The resulting data can be used for further processing and visualization in Grasshopper.<br>
   
2. Use the analyzed project output from the `Run Ecological Analysis` component as the input for the `Deconstruct Voxel` component. Retrieve the data outputs for the different categories: geometry (`Id`, `Mesh`, `Position`, `Size`, `Inclination`), environmental (`Solar`, `Soil Depth`, `Soil Volume`, `Soil Type`, `Precipitation`), ecological (`Timestamp`, `Species`, `Biomass`, `Abundance/Biodiversity`, `Plant Volume`).<br>
   
3. Access the analysis data outputs for your project using the `Get Voxels by Type` component. Select `Exterior Envelope` to obtain the results for your project’s outer envelope.<br>

 <div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_05_Data_1.gif" width="1000"></div><br>
   
4. Adjusting the `Time Step` (years) input in the `Deconstruct Voxel` component returns ecological results for each year of the simulation.

 <div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_05_Data_4.gif" width="1000"></div><br><br>

 
---

## Step 6: Visualize & export

1. Use standard Grasshopper components such as `Domains` from the **Math tab**, `Gradient` from the Params tab, and `Custom Preview` from the **Display tab** to visualize the data outputs in Rhino.<br>

2. Use the provided components to visualise the `Solar` analysis results on your project in Rhino.<br>

 <div align="center"><img src="assets/images/png/Rhino.Ecologic_User_Guide_05_Data_3.png" width="1000"></div><br>

3. Use the provided components to visualise the `Soil Depth` analysis results on your project in Rhino.<br>

<div align="center"><img src="assets/images/png/Rhino.Ecologic_User_Guide_05_Data_2.png" width="1000"></div><br>

4. Use the provided components to visualise the `Plant Volume` analysis results on your project in Rhino.<br>

<div align="center"><img src="assets/images/gifs/Rhino.Ecologic_User_Guide_05_Data_3.gif" width="1000"></div><br>

5. Save your **Rhino.Ecologic Project** using the Save Project Grasshopper component in the **Project tab**.<br>

6. For more details on customizing your analysis setup, and information on ecological analysis, see the **Rhino.Ecologic User Guide**.<br><br>


---

## Help and feedback

If you encounter any issues or have questions, feel free to reach out. Please share your feedback, bug reports, or feature requests using this short form:  
   [Submit Feedback](https://docs.google.com/forms/d/e/1FAIpQLScrWsFKbOuNfe3xqFSj6IBbicJy6n7YHwASiWTutuiII5RmzA/viewform?usp=pp_url)


---
#### Referencing

When referring to Rhino.Ecologic® in a scientific publication, cite as follows:

**Vogler, V., Kourkopoulos, E., Fraguada, L., Mimet, A., & Joschinski, J. (2025).** Integrating ecological modeling into the 3D CAD system Rhinoceros. *JoDLA – Journal of Digital Landscape Architecture, Issue 10–2025*, 86–100. Berlin/Offenbach: Wichmann Verlag im VDE VERLAG. e-ISSN 2511-624X. https://doi.org/10.14627/537754009

**Vogler, V., Kourkopoulos, E., Joschinski, J., & Eckelt, K. (2025).** Developing volumetric data models for ML training datasets using Grasshopper. *JoDLA – Journal of Digital Landscape Architecture*, Issue 10–2025, 101–113. Berlin/Offenbach: Wichmann Verlag im VDE VERLAG. e-ISSN 2511-624X. https://doi.org/10.14627/537754010


---
<br>
<br>
<br>

<div align="center">Have fun using Rhino.Ecologic!

*Eleftherios, Jens, and Verena.*</div><br>

 <div align="center"><img src="assets/Rhino.Ecologic.Logo/Rhino.Ecologic_Logo_3.png" alt="Alt text" width="300"><br>
 &copy; 2025 McNeel Europe S.L. All rights reserved.</div><br>
