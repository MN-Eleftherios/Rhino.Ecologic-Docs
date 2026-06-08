
# 0 Preface

The model description follows the ODD (Overview, Design concepts, Details) protocol for describing individual- and
agent-based models (Grimm et al. 2006), as updated by Grimm et al. (2020). 

# 1 Purpose and Patterns

The model aims to predict the presence and temporal change of plant communities in 3dimensional environments. It tracks the number and biomass of plant individuals from multiple species.   
The main proceesses are 1) tracking of habitat suitability, seperately for each species, based on soil properties, and 2) modeling competition for light via: individual plant growth; a light interception model; conversion of light energy into carbohydrate resources; and allocation of said resources into growth and reproduction. Biomass growth in turn increases light interception and leads to shading of other plants.   
Habitat suitability, growth and light requirements are species-specific. The species traits are usually inputted by the user; although care is taken to ensure the traits are within reasonable limits, the validity of the model output depends heavily on the quality of the input traits.  
The simulation is a pure ecological model and does not include evolutionary processes. It is not expected to perform well in highly stochastic environments (e.g. erratic herbivore pressure) or in environments under soil nutrient limitation, as these processes are not included in the model.  

# 2 Entities, state variables, and scales 

## Spatial and temporal resolution  

This model simulates plant communities in urban areas, in particular on building envelopes. It is envisioned to be used on spatial extents of 10 x 10 m - 200 x 200 m, at input resolution of 1m³, and to simulate 10-200 annual time steps. The model internally uses a finer resolution in z - direction (5 cm) to boost the accuracy of light and shading calculations, but outputs are aggregated to the input resolution.  
It is a parametric model, meaning the user can change extent and resolution at will, but:  
- not all inputs are scaled with resolution yet (e.g. maximum number of plants per voxel), leading to potentially wrong results.  
- Seasonal change or leaf senescene are not modelled, so time steps below one year are not supported. 
- finer resolutions or larger extents may deplete memory and increase run time.  

In summary, best results are obtained at 1 m³ voxel resolution and annual time steps. If deciding to nevertheless change the spatial resolution, it is advised to adjust the amount of layers (noStrata) to keep a 5 cm resolution in z-direction. A coars resolution in xy-directions (e.g. 10x10m) but fine resolution in z-direction is a compromise that reduces data and memory footprints but not runtime.  
\[**subject to change**; NoStrata and other parameters to scale with resolution in future versions]

## Entities  

This chapter provides a **conceptual** overview of the entities that make up the model. Implementation details may differ in details, but code and concepts are matched wherever possible. For easier understanding of code-concept relationships, differences are indicated by footnotes.

| Entity | Description |
| ---    | --- |
| Environment | (urban) habitat containing voxels; light; and seeds that are not (yet) settled in voxels|
| Voxel | a voxel may contain light; soil; and disturbances. A Community is associated with each Voxel that contains soil|
| Community<sup>1</sup> | Contains Individuals and seeds of different species. Associated with a voxel and its soil, but individuals can outgrow the voxel boundaries. |
| Species/Traits | A "species" is a set of traits<sup>2,3</sup> defining 1) Life history parameters, 2) resource allocation strategies, 3) seed biology, 4) disturbance response, 5) soil requirements and 6) shape.<sup>2</sup> 
| Individual | Roots in soil; located in/across Voxels; can grow, reproduce and die (subject to voxel attributes). Composed of leaf biomass and (belowground) resources. Biomass converts intercepted light into resource, resource is invested in biomass and reproduction. Uses soil and light resources (casting shade), and can be disturbed. Individuals of different species differ in traits.


<sup>1</sup> Conceptionally *associated with* voxel, but coded as *component of* Voxel.  
<sup>2</sup> "Species" can also represent plant functional type, Functional group or distinct population  
<sup>3</sup> No separate Species class is required in the code. Each individual is associated with one rule set ("belongs to a species"). Pointers to an immutable Traits library are used for performance reasons (flyweight pattern). There are as many Trait instances as species.

### Individuals and seeds

The principle agent is an individual plant. The plant has two life stages: a seed stage and the actual plant individual. 

The individual has the following attributes:
- it lives: it grows, reproduces and dies.
- it is photoautotrophic: it converts light into a (carbohydrate) resource; it lives from these resources and produces green (photosynthetically active) biomass
- it has a shape: it has a geometric representation, being able to e.g. cast a shade with its biomass
- it needs a place to live: the soil determines whether a plant can grow at all (habitat is suitable). Soil (space) is a limiting resource.
- it interacts with the environment: Stressors can affect each of the above properties, i.e. cause mortality, deplete resources or affect the soil.

The seeds have the following attributes:
- they live: they can die, or they can be converted into individuals
- they can form seed banks: Seeds produced in autumn may germinate the next year, or they can stay dormant and be buried in the soil for many years. The mortality of dormant seeds is lower than the mortality of active seeds that attempt to germinate (germination rate)
- they need space: the soil can accomodate a potentially large, but not infinite number of seeds. 


The attributes are naturally interdependent. Yet, in the model the interdependencies are kept to a minimum, both to constrain the logical complexity of the model, and to keep the model modular and modifiable. The following interdependencies exist:  
- plants only germinate on suitable soil types; they stop collecting resources if the soil becomes unsuitable;
- plants that run out of resources die;
- seed production (fecundity) only starts at maturity age, and requires resources.
- the general shape of the plant affects where green biomass will be located. This affects resource acquisition and tolerance to shading.

The following interdependencies were on purpose not implemented:  
- no maternal effects: life-history (age, height) and resource state do not affect seed biology, and plant growth is not affected by seed states.  
- no phenotypic plasticity: age does not affect resource acquisition, and resource availability does not affect height growth.  
- no soil quality: resource acquisition does not scale with soil variables, and soil is not conditioned by the plants or seeds living in it.  

In particular, the plant's bounding box / bounding box growth (growth model) are entirely deterministic and unaffected by resources; the biomass within that bounding box, and hence the shade it casts, may however vary by plant health state.   
Disturbance is not fully implemented, but it may independently affect resources and/or biomass, life history (height or mortality), seeds and soil. Priority will be given to disturbance affecting biomass and resources.  

### Species/Traits  
There is no single optimal strategy for life history, resource allocation and other attributes, rather the optimal strategy depends on the environment and the other individuals in the community. Hence, different trait combinations (species) have evolved. This model does NOT simulate inter-individual variation and/or inheritance (evolutionary model), but rather assumes that each individual belongs to a certain strategy type (species) with fixed traits. Individuals vary in states (e.g. current age), but there is no intraspecific trait variation (e.g. life span), no evolution and no phenotypic plasticity. Trait types may be envisioned as species, but can also be genera, distinct subpopulations, functional groups or functional types.  
 According to the attributes of Individuals and seeds given above, traits that define an individual fall into 6 categories: 
 1) Life history parameters. Life history determines key demographics (maturity onset, lifespan) and growth.
 2) resource allocation strategies. Resource allocation determines the relative investment into seeds and biomass gain (e.g. light conversion efficiency, fecundity).
 3) soil requirements. Plants can only grow on certain types of soil, and the soil needs to meet some minimum requirements (minimum depth).
 4) disturbance response. Disturbance response defines which types of disturbance ("rabbit herbivory", "tree pruning") affect the plant, and how strongly.
 5) seed biology. The seed biology determines the mortality of seeds, dormancy attributes, and germination rates.  
 6) shape. The shape determines the spatial distribution of biomass. 

For details on the state variables, see section 7 ("submodels") or UML diagram.

### Voxels  
A voxel is a three-dimensional object that is embedded in a 3D-landscape. In our model it contains a soil, a plant community, and light (and disturbance, to add).  

*Soil*  
Soil is a shared resource that is used by the Individuals, especially upon germination. It has a limited capacity to host plant individuals and its state may change over time. Some attributes are set at initialization and cannot change throughout the simulation run.  
Soil is not represented as 2- or 3Dimensional object, but as an object with one-dimensional properties (depth, type, capacity, water availability); stratification of soil types is not yet included.  
Although technically possible, it is currently not expected that a single soil spans multiple voxels. Future extensions (e.g. nutrient or water flow) may have to carefully revise this decision.    

*The plant community*  
The plant community is made up of the Individuals and seeds. Although the plant community and all its individuals have their home in one voxel, they can outgrow it in both x/y and z-direction (see section "environment").   


### Environment
The Environment contains Voxels in a spatial configuration, and plant individuals live in, and across, these Voxels. The environment takes care of calculating the amount of plant material in each voxel; intersecting light rays with the plant material; and attributing the according light energy to the plants. Light calculations also take the (built) 3D environment into account. For details see submodel "light model"  
In addition, the Environment contains a pool of "unsettled" seeds. After each time step, the seeds produced by all mature individuals are collected. They do not immediately sink to the ground of their home voxels, but are reshuffled and distributed (randomly and uniformly) across all voxels.


*Disturbance* to add  

### Notes  

There are a few implementation-specific notes affecting the general logic of the model:
- The model is written with extensibility in mind. Replacing a submodel (e.g. resource calculation) should have minimal effects on the functionality of the remaining model.
- Soil has a "capacity" that is used to simulate competition for space, but no other attributes that may be depleted. Hence, plants of different types compete for the same singular resource.
- disturbance is not implemented yet.
- Raytracing is not implemented yet, and the current calculations are confusing in some edge cases (e.g. high plants growing next to a wall, growing from shaded into unshaded voxels; plants growing on south-exposed facades being shaded by plants growing above)
- Due to the existence of plant "home" voxels and individual soils that do not cross voxel boundaries, the minimum spatial resolution must be at the largest plant's soil size requirement (i.e. stem diameter, or core root area). Plants cannot take a soil capacities from multiple voxels.  

# 3. Process overview and scheduling

![Process overview](<doc/process_overview.jpg>)

1. A Graph with Voxels is created prior to running the model. The voxels store environmental attributes that are required by the model. 
2. The model is initialized with the graph and further configuration settings. It reads the trait definitions from a default file (or use data passed in via the plant library component), create a log file and perform checks on the input data. It further runs the 0th time step and discards the results.
3. Once constructed, the model runs for a time step. This includes a simulation step (see fig. 1), and adding the results (volume, species names, biomasses and abundances) to the voxels.
4. Step 3 is repeated for the desired number of time steps.

A time step consists of the following steps (see fig. 1):
- add and disperse seeds in the environment. A fixed seed rain "from the outside" is also added.
- move seeds from global environment pool to individual voxels.
- calculate plant spillage, i.e. locate plant material in the correct voxels according to plant shape and size.
- calculate light for each voxel, and distribute it to the plants 
- update the plant community. The process includes aging, resource allocation, potentially death, production of new seeds and germination of assigned seeds.
- the seed output is limited to max. twice the nummber of voxels; the seeds are added to the environment.

The order ensures that new seeds may land in a cell and germinate in the same time step; but that the plant cannot additionally grow the same time step. It is also ensured that newly produced seeds disperse, but cannot germinate in the same time step.  

Individual steps (such as seed dispersal) can be forced off, but this is usually unrealistic and not recommended.
# 4. Design concepts

## Basic principles
The model combines habitat filtering approaches with process-based modelling, with particular focus on competition for light. The soil and nutrient balance as well as disturbance are on purpose left as simple and general as possible. The model is, however, written in a way that allows later extension.

## Emergence and interaction 

The key outcome of the simulation is the spatiotemporal change of community patterns, which emerges from the interaction of individuals within each voxel, particularly competition for light. Plant material outgrowing a voxel (i.e., large trees) can also cause very localized patterning. Abiotic factors (soil depth, soil class) further impose a general community structure via habitat filtering. The abiotic factors together lead to the emergence of complex environment-community relationships.

## Adaptation

The  model does not contain any direct objective-seeking mechanisms. The agents do not possess the ability to choose among two or more alternative behaviors e.g. via learning or condition-dependent rules (phenotypic plasticity).

## Objectives

The model does not simulate adaptive behavior (fitness/biological evolution). The rules by which agents respond to the environment (the "genotypes" or traits) are instead provided by the user (default or via plant library component).

## Interaction

Individual plants interact with one another, directly or indirectly, especially in the competition for light, which is explained in detail in the submodel section. Briefly, growth and death of plants continually changes light conditions, and, subject to light tolerances, plants may be outcompeted and die. This leads to turnover of plant communities. 
There is further simple competition for soil as a shared resource.

## Stochasticity 

The model contains the following stochastic processes:
1) whenever there is free space in a voxel, plant individuals are picked randomly from the pool of suitable types, until each voxel's capacity is reached. Their initial biomass is fixed (non-random).
2) when the total plant biomass exceeds a voxel's capacity, a random set of plant material is chooosen to lie "on top" and receive the light. The granularity of the draw (how much plant material to choose per draw/number of draws) can be adjusted.
3) At the end of the time step, all plant seeds are pooled and randomly redistributed over the simulation environment, together with the seeds that enter from the region (seed rain). 
Growth rates, lifespan and soil suitability are fixed properties without random component.

# 5. Input and Output Data
A graph with Voxels is prepared prior to running the model, and it contains all neccessary information. At the moment, the information contains soil depth, soil type, light and precipitation in each voxel. Additionally, there are configuration settings and definitions of the Plant species (Traits). The model updates the voxels as it runs, adding for each time step: 
- the total biomass of all plants rooting in the voxel, separately for each species
- the total abundance of all plants rooting in the voxel, separately for each species
- the total plant volume (independently of where plants root), summed across all species. 

# 6. Initialization
At initalization each cell is filled with the according input data and with a starting Community. The Community created by randomly drawing from a list of suitable types until the capacity (or starting population) size is reached; an according seed pool is also created. Individuals are created as 0.1 year old plant (i.e. having roots, height and diameters of a 0.1-year old) and with enough resources to survive the first year.

The model runs additionally 1 time step upon creation, discarding the 0th time step.

# 7. Submodels

There are five independent submodels to simulate plant life (growth, resources, habitat suitability, shape and disturbance); one submodel for plant seeds; and two environmental submodels that handle seed dispersal and light calculations/raycasting.  
Briefly put, the habitat suitability model restricts where a plant can grow; the growth model simulates change in plant growth; the resource model simulates the change in biomass; the shape model maps the plant biomass into 3d space; and the light model intersects the 3D plant with light beams, feeding the resource model with energy. The resource model additionally creates seeds that are dispersed by the seed dispersal model, potentially creating new individuals.

## Plant growth
This submodel lets the plant age, mature and grow. The life history traits defining the plant's growth are:    

| Type | Trait | Description |
| --- | --- |--- |
| float| MatAge | Age (in years) at which the plant reaches maturity and can start reproducing. Also used to estimate when plant growth decelerates    |
|int |LifeSpan| Maximum age (in years) that a plant can reach, if not dying of other causes  |

The plant's age is incremented in every growth cycle; if it exceeds the life span, it dies. Age and life span are entirely deterministic (no random deviations or inter-individual variation).  
The growth is resource-independent and uses a "TI form" of a Gompertz growth function (Tjorve 2017):   
```W(t) = A * exp(-exp(-KG * (t - TI)))``` with with A= maximum size, TI = inflection point, KG = slope at inflection (growth rate), t = age  
As inflection point we used the age of maturity (when plants invest into seeds, their growth decelerates). The growth rate is arbitrarily set to 1.998 / (LifeSpan - MatAge). The growth rate (i.e. slope, not max) is further reduced to 20% when water provision is insufficient (see habitat suitability submodel).
Plants can reproduce in the year they reach maturity, i.e. when ```age >= int(matAge)``` [float precision is required for the Gompertz function though].   
The growth is exposed as dimensionless variable (percentage of max) to the other submodels; these then calculate specific variables (like biomass, height, surface area) based on the general growth pattern and environment-specific effects. 

![aging](<doc/age.jpg>)
## Habitat suitability 

The habitat suitability submodel checks for each plant type in which cells it can theoretically occur (habitat filtering).
The soil requirements defining the habitat suitability are:  

| Type | Trait | Description |
| --- | --- |--- |
| int |rootingDepth| Maximum length of the roots in cm
| dict(string, bool) | acceptedSoils| List of soils on which this plant can grow. Describes nutrient and water requirements of the plant (simplified)  
| int | size | Relative plant size, describing how much soil space the plant occupies  (there is only a maximum number of "seats" available in each soil)  

While seeds may be deposited on any soil, and also germinate, they only are recruited (converted into Individuals) if the soil is at least as half as deep as the initial root size, and one of the acceptedSoils of the plant. 
As the plant ages, its roots will grow, proportionally to the increase in above-ground biomass and size (see growth submodel).

Growth may cause the roots to become larger than the soil depth. If the roots exceed twice the soil depth, the habitat becomes unsuitable. While this does not kill an already established plant, resource acquisition is inhibited, causing the plant to die after a while (depending on available resource pools).

If a soil does not contain enough water, the growth is slowed to 20%. Currently, water is sufficient for all plants if precipitation > 0, so there is no attribute defining species-specific water needs [subject to change in the future]

### List of supported soil types  
The soil and nutrient balance are on purpose left as simple and general as possible. We identify 12(13) distinct classes of soil that differ in water capacity, resistance to penetration by roots, pH, and nutrient dynamics. Our naming follows the USDA texture triangle, but is meant to be understood as a functional and not neccessarily physical attribute. According to the texture triangle, soil textures are understood as mixture of sand (coarse grains), clay (fine) and silt (intermediate). Pure Sand, Clay and Silt accordingly form three extreme edge cases, while equal mixture of all particle sizes is called Loam.  
The 12 classes have the following ecological properties:
- Sand: Coarse-grained soil with very low water and nutrient capacity but good aeration, little organic matter and little structural support. Examples: Sand dunes (embryo dunes, yellow dunes, Sossusvlei) and construction sites
- Clay: Very heavy soil with high water and nutrient capacity, but no aeration and strong penetration resistance. Relatively rare. Examples: lake sediments or quiet zones of river deltas (locally), heavily disturbed sites with exposed subsoil; contaminated brownfields (functionally equivalent)
- Silt: Fine but non-cohesive soil with high water retention and nutrient accessibility, but prone to erosion. Rare in pure form. Examples: exposed subsoils and river bank sediments in loess regions (e.g. Danube loess belt).
- Loamy Sand: Sand-dominated soil with some fine particles; higher nutrient and water-holding capacity than pure sand. Examples: heathlands, dry pine forests.
- Sandy Loam: Loam-dominated soil with a higher sand content; good aeration and drainage but lower nutrient retention than loam. Examples: sandy riverbanks, vineyard soils in warm climates, intensive green roofs.
- Loam: Balanced soil with a mix of sand, silt, and clay, providing good aeration, drainage, and water and nutrient retention. Easy for roots to penetrate. Examples: gardens and parks, productive farmland.
- Silt Loam: Silt-dominated, fine-textured soil with good drainage and high water and nutrient retention, but vulnerable to erosion and compaction. Examples: fertile agricultural plains, loess, river deltas.
- Clay Loam: Clay-rich soil with high water and nutrient retention, but prone to compaction and difficult root penetration. Poor drainage and stagnant water may limit plant growth. Examples: heavily irrigated farmland, muddy pastures.
- Sandy Clay Loam: Substantial clay content but enough sand to improve aeration and drainage compared to clay loam. High nutrient retention but moderate penetration resistance. Examples: Old agricultural soils on sandy substrates, mixed alluvial deposits; some urban soils
- Silty Clay Loam: Fine-textured soil with high water and nutrient retention but reduced drainage, intermediate between loam and clay loam. Examples: floodplains, fertile valley soils, brownfields and other compacted vacant lots.
- Sandy Clay: Clay-dominated soil with noticeable sand fraction; very high nutrient and water capacity, but still poosr drainage and accessibility. Examples: disturbed industry/construction sites
- Silty Clay: Very fine-textured, cohesive and with extremely high water/nutrient retention. Minimal aeration, high penetration resistance. Often seasonally waterlogged. Examples: backswamps, lake margins
- The class NoSoil describes the absence of any substrate that supports growth.


## Disturbance  

The disturbance model allows to provide a list of disturbances. The disturbances can destroy biomass, deplete the resource pool, or remove seeds.
Disturbance is not yet implemented.

## Shape
### logic and purpose  
The plant's shape describes the location of biomass, which collects light energy but also casts shade. 
The plant's shape generally consists of a stem which contains no green leaf biomass, and a crown representing the leaves. The crown may be one of various 3-D shapes. Currently all crown shapes have a circular base  (cylindric, conical, spherical....) to ease voxel spillage calculations.  
The stem may cover between 0 and 100% of the plants height. Small grasses and herbs are best approached with a stem of 0% and cylindrical shape, small shrubs by a stem close to 0 and spherical shape, and evergreen/deciduous trees with stems of ~ 50% and conical/spherical shapes respectively.

The plant's total leaf volume determines its resource balance, hence it is important that the volume (rather than height or diameter) is controlled accurately.  
The shape model serves two functions: first, it calculates height/diameter based on the (changing) volume to allow drawing the plant; secondly, it serves as input for the light submodel (see below for details) - the light model traverses through the site in z-dimension and redistributes plant volumes along the x/y axis in chunks of 5 cm height. It thus requires knowledge of the amount of plant volume within a certain height cross section.  The outputs are thus: height, diameter and volume between (h0, h1) as function of current total volume.

### General formulas
- The maximum volume that the plant can obtain is shape-specific but depends on diameter and height:  
```Vmax = v(maxDiameter, crownHeight) ```; where ```crownHeight= maxHeight * (1-stemRatio)```. maxHeight is the maximum plant height, stemratio the proportion of stem; maxDiameter is the maximum width of the tree crown (or simply diameter of the plant for herbs and grasses).    
- The plant's current volume is ```V[i] = size[i] * Vmax```, i.e., proportional to the growth determined by the Gompertz function (size[i] € [0,1]). 
- Height is ```h[i] = cbroot(size[i]) * maxHeight```  
- width(diameter) at height h is: ```w[i,h] = cbroot(size[i]) * maxDiameter * d(h')```; where d(h') is a shape-specific function describing diameter at height (h'). h' is height expressed in unit terms ```h'  = h/h[i]```  
- The amount of the plant's volume that falls into a height cross section [h0,h1] is ```Vsection = V[i] * s(h1',h0')```; where h0' and h1' are scaled by the cubic root and mapped to unit terms (stem sections excluded); s(h1',h0') is shape-specific.

- ### Shape functions

The shape-specific functions assume unit sizes of h =1 and r = 0.5 (except for volume calculations which use the actual diameter and heights). For cylinders these are: 
- ```v(diameter, height) = pi * (diameter/2)^2 * height```  
- ```d(h) = 1```, i.e., the diameter of the plant is constant across its height; 
- ```s(h1, h0) = (h1 - h0) / 1```, i.e., the volume in a height section is proportional to the section's height.

The functions for cones (apex on top) are:
- ```v(diameter, height) = (1/3) * pi * (diameter/2)^2 * height```
- ```d(h) = 1 - h```, i.e., the diameter declines linearly with height
- ```s(h1, h0) = V_frustum/V_total = (v(d(h1),h1)- v(d(h2),h2)) / v(1,1)```, i.e. the frustum's volume is calculated based on the diameters at h1, h0

The unit spheroid (sphere, but potentially stretched/contracted along height axis) is offset by 0.5 units in height to have the base at h=0. Its functions are:
- ```v(diameter, height) = (4/3) * pi * (diameter/2)^2 * (height/2)```
- ```d(h) = sqrt(0.5^2 - (h-0.5)^2)```, i.e. following the formula of a circle;
- ```s(h1, h0) = (V_frustumlike / V_total) = cap(h1) - cap(h0)/v(1,1)```; where cap(h) is the volume of a spherical cap, ```pi * h^2 * (3.0 * 1/2 - h)) / 3.0 ```

### Volume to surface area conversion
Plants are represented as a stack of plant surfaces, which are distributed along the height axis of the plant's volume. One layer of plant material has a thickness of stratheight = 5 cm. The shape submodel accordingly converts leaf volumes Vmax and Vsection into surface area by dividing them by 5.0. stratheight is currently fixed but may be changed in the future.   
The leaf area is both used to convert light (resource allocation model), and to cast shade (voxel model). Sometimes a plant may cause less shading than its surface area suggests (e.g. reflections, orientation of leaves), hence there is a possibility to reduce the shade cast per area by an opacity factor.  
Plants that currently do not have the ideal biomass (see resource model) cover the same volume but are less dense than under ideal growth. Density lower than 1.0 reduces the amount of shade proportionally.  

### Summary 
The volume changes over time with inputs from the growth model, and approaches Vmax. Vmax is derived from species traits MaxDiameter and MaxHeight. Plant height and width increase monotonically following a cubic root relationship with volume. Height is additionally corrected for the species trait StemRatio. The explicit shapes allow querying current diameter and volume at height h; the volume is then converted into leaf surface area, and distributed across voxels by the light model.    
The biomass is modelled indepently of the growth model, and a plant may fall behind in biomass growth. If so, it has the same volume but a lower density than a fully grown plant, and accordingly holds less surface area and casts less shade. 

In summary, the shape submodel holds the state variables size (proportional to volume) and density. It uses the following species traits:  

| Type | Trait | Description |
| --- | --- |--- |
|float (0-1)|stemRatio |ratio of stem to crown|
|float (0-1)| opacity |ratio of shade cast: leaf surface area|
|float| maxHeight | maximum height of the plant including stem |
|float| maxDiameter | maximum diameter of the plant crown |
|string| shape | shape of the plant crown; currently supported are "Cylinder", "Cone" and "Sphere" |
|| maxVolume | maximum volume of the plant crown. Derived from shape, maxDiameter and maxHeight  |

## Resource use
The resource submodel takes care of collecting light, converting it into resources, and spending resources on growth and reproduction.
The resource allocation traits that determine Resource use are:  

| Type | Trait | Description |
| --- | --- |--- |
|vector\<float\>| conversionEfficiency| Conversion efficiency of light to carbohydrates for each age, in g/KJ annual light energy. Typical values are in the order of 0.005 (https://en.wikipedia.org/wiki/Photosynthetic_efficiency )<sup>1</sup>
| float | maintenance Costs| Energy costs in g carbohydrates per g biomass that the plant has to pay annually to maintain itself. Includes respiration, leaf replacement and other costs.
|   float |biomassAllocation<sup>2</sup>| percentage of resources that is allocated into green leaf growth anually, after accounting for maintenance costs. Unit: [g biomass / g Carbohydrates] 
|float |seedAllocation<sup>2</sup>|  percentage of resources that is allocated into seed production anually, after accounting for maintenance costs. Any allocation that is "wasted" on other means not being modelled (e.g. investment into woody structure) can also be dumped here.  Unit: [g seeds / g Carbohydrates]  | 
|  float |maxBiomass|  Maximum biomass in g that can be reached

<sup>1</sup> Providing a single biologically correct conversionEfficiency value is difficult, as the plant progressively shades itself as it ages. Both the lighting model and the resource allocation model are not realistic enough yet. The parametrization procedure allows calculating a reasonable list of conversion efficiencies that match the model's logic. See parametrization section for details. [**subject to change**]  
<sup>2</sup> Ensure biomassAllocation and seedAllocation sum to 1.0;

Under ideal conditions, the plant follows the growth curve given by the growth model, i.e. a Gompertz function that is dependent on current age, life span and maturity age, and levels off at maxBiomass. If less resources are available, the growth is limited accordingly. This submodel computes the amount of available resources and invests them in biomass and seeds:

1) The plant converts light into a resource. One light unit (in KJ) is converted into conversionEfficency[i] resource units (e.g. carbohydrates in g), where i denotes age;  
2) Resources are reduced by maintenanceCosts * idealBiomass, where idealBiomass follows the age-dependent Gompertz function. 
3) The plant allocates seedAllocation * resources to reproduction, provided it is mature (age-dependent); and up to biomassAllocation * resources to biomass. The value is capped at idealBiomass however. The resource pool is reduced by potential seed and biomass investment;
5) If the plant runs out of resources, it dies.  

Notes:  
- While seed production is only resource-limited, biomass growth is capped by the Gompertz function - even if abundant resources are available. This avoids unrealistically high growth rates. 
- juvenile plants and Gompertz-capped plants do not save the unspent resources, to avoid building up unreasonable reserves.
- maintenance costs are based on ideal biomass, and *not* reduced when the plant is being shaded. This mechanism causes plants to die rather than just growing slower when deprived for sustained time.

## Seed Pool
The seed pool submodel simulates the life of seeds before they are recruited and converted into individuals. 
The seed biology definings seeds are:   

| Type | Trait | Description |
| --- | --- |--- |
| bool | Dormancy| describes whether species forms a seed bank (true/false); seed banks ensure plants persist through unfavourable conditions
| float | MortalityDormant| mortality of dormant seeds, percentage
| float | DormancyBreakRate| rate of breaking dormancy and pushing seeds back into active state (high rate = short-lived seed bank)  
| float | GerminationSuccess| probability of germination as percentage (active seeds only)  

The seed pool contains up to c seeds of a species; the voxel and its soil determine the capacity c.  
Seeds may be dormant or active; if dormant, they have a mortality of mortalityDormant in each growth cycle, if active they suffer germinatSuccess mortality upon attempting to turn into an Individual. Active seeds cannot turn into dormant seeds, but dormant seeds are converted at a rate of dormancyBreakRate into active seeds in each growth cycle.  
Seeds may be disturbed (affecting only active or in equal proportions active and dormant seeds), and new seeds (active or dormant) may be added to the seed pool.  

## Dispersal  
random uniform redistribution of any newly produced seeds + input from region.
[**subject to change**\]

## Light model
The light submodel is responsible for calculating the amount of light that each plant individual shall receive. The first step is the allocation of plant material in the correct Voxels; the second step is the calculation of light interception by this plant material.  

### Plant spillage
Plant individuals have a "home" voxel with a soil in which they root, but are otherwise relatively independent of the voxel grid. In particular, they may be larger than a Voxel.   
For the purpose of light calculation, voxels are split into flat cuboids (5 cm high but 1 voxel wide, see section "spatial and temporal resolution") that can hold a single, 2-dimensional, layer of plant material. The Shape submodel is accessed to query the amount of plant material at each height and the material is distributed in the x-y plane, according to the diameter of the plant (starting with 8 adjacent neighbours; it is implicitly assumed that xdim refers to a diameter, i.e. the plant has a circular base). If less than the required neighbours are available, plant spillage is NOT adjusted. For example, with only 4 neighbours, each neighbour still receives only one eigth of the plant material; the remaining material is thrown into void space. This prevents plants from getting strange shapes e.g. in narrow alleys.  

Because plants grow independently of the Voxel and light models, and because material spills across voxel boundaries, there may be more plant area than a voxel may support. In this case, a random selection of plant material is chosen to lie "on top" and receive light. The number of draws is a parameter (lightDistributionFactor) - more draws lead to fairer resource allocation (small intertwined leaves), while less draws indicate winner-takes-all strategies (e.g. one big leaf overshadowing all other material). For simplicity, lightDistributionFactor is a global attribute, not species-specific.

### Raycasting 
[**subject to change!**\]  
The simulation of a plant community benefits from a realistic calculation of light levels. Ideally, one would write a raycasting model, but this is obviously computationally challenging and complex, so some simplification is required.  
In this model (version), light beams are sent at a 90° angle through the Environment, getting reduced in proportion to the surface area that is blocked by plant material. Because this calculation diminishes available light very quickly, a certain proportion of the available light is "diffuse light" and cannot be reduced by biomass, hence allowing some light to arrive on the ground even after passing through multiple layers of plant material.  
To model shading by buildings etc., each voxel has its own irradiance value, which is provided as input constant and cannot be changed throughout the model run. The light energy arriving in a voxel is ```E * irradiance * (1-shading)```, where E is a fixed input parameter (typically 1000 KW/m²/year). The resulting light energy is available to all plants growing in the voxel layer, proportionate to the surface area they cover.
If input irradiance is calculated with a realistic shading model, the simulation results are affected by the 3D shape of buildings and other structures. The angle at which "biomass above" is calculated is currently 90° and contrasts with those irradiance calculations, but the angle may be changed in the future (intermediate solution between raycasting and 2d shading).


# 8 Parametrization  
## Recap of resource economy and growth logic of the model

Plants can only germinate if soil conditions are suitable. After germinating, the plant grows above ground in height, and below ground in root length. At some age it matures and is able to produce seeds, and it dies once lifespan is exceeded.  
Each plant individual has a carbohydrate resource pool, and green leaf biomass. It uses its leaves to collect sun energy and convert it into resources. The resources are used to survive, and to reinvest into more leaves and seeds.  
The seeds that are produced by the plant are dispersed across the site, and may germinate if landing on suitable soil, thus starting a new plant individual.  
Because plant biomass casts shade and because plants vary in growth through space and time, shading conditions change dynamically. Variable soil conditions, variation among plants in resource efficiency/shade tolerance, and stochasticity in seed dispersal cause a dynamic plant community to emerge.  
Further effects.
- Resource acquisition (and hence growth) is stopped if there is no more space left for the roots.   
- Seeds may also be dormant for a longer time (seed bank), but this feature is disabled by default and not yet part of the parametrization.  
- Disturbance (management) is not implemented yet.


## Parametrization challenges

As stated in the model description, plant growth and resource economics are largely determined by fixed input parameters - there are no adaptive behaviours (phenotypic plasticity) that help a struggling plant, and a negative energy balance can quickly cascade into a positive feedback loop that kills it. Because the efficiency inevitably decreases with age (self-shading), no set of input parameters could maintain a consistent positive energy balance.   
Great care was taken to ensure that all parameters are as realistic as possible; however, to offset the decline in efficiency, the parameter CONVERSION EFFICIENCY is adjusted so that the model produces sensible results, even if not grounded in theory. Future changes to the light model, but also implementation of plasticty should reduce the need for such a scaling vector, and in anticipation of the changes a realistic (but currently unused) derivation of conversion efficiency is already provided.


## Traits required by the model  

Here is, for completeness' sake, the full list of parameters that is required by the model. Many of the traits are estimated from simpler traits as described in the next sections.

|Type |Trait | Description |
| --- | --- |---|
| float| MatAge | Age (in years) at which the plant reaches maturity and can start reproducing. Also used to estimate when plant growth decelerates    |
|int |LifeSpan| Maximum age (in years) that a plant can reach, if not dying of other causes  |
| int |rootingDepth| Maximum length of the roots in cm
| dict(string, bool) | acceptedSoils| List of soils on which this plant can grow. Describes nutrient and water requirements of the plant (simplified)  
| int | size | Relative plant size, describing how much soil space the plant occupies  (there is only a maximum number of "seats" available in each soil)  
| float| HMax| Maximum height (in cm) a plant can reach, including stem if present.   |
|  float |maxDiameter| maximum diameter (in cm) of the plant at its widest point. |
|float |SLA| Leaf surface area per g dry weight(cm2/g). No longer in use.
|float|	maxVolume|	maximum volume of the plant crown. Derived from shape, maxDiameter and maxHeight
|  float |opacity| amount of shade cast per cm² surface area (0.0-1.0). Most plants have opacity of 1.0.
|string | shape| describes the plant shape. Currently supporting "cylinder" (default), "Cone" and "Sphere".
|float | stemRatio| ratio of crown height to total plant height
|vector\<float\>| conversionEfficiency| Conversion efficiency of light to carbohydrates for each age, in g/KJ annual light energy. Typical values are in the order of 0.005 (https://en.wikipedia.org/wiki/Photosynthetic_efficiency )
| float | maintenance Costs| Energy costs in g carbohydrates per g biomass that the plant has to pay annually to maintain itself. Includes respiration, leaf replacement and other costs.
|   float |biomassAllocation| percentage of resources that is allocated into green leaf growth anually, after accounting for maintenance costs. Unit: [g biomass / g Carbohydrates] 
|float |seedAllocation|  percentage of resources that is allocated into seed production anually, after accounting for maintenance costs. Any allocation that is "wasted" on other means not being modelled (e.g. investment into woody structure) can also be dumped here.  Unit: [g seeds / g Carbohydrates]  
|  float |maxBiomass|  Maximum biomass in g that can be reached
| bool | Dormancy| describes whether species forms a seed bank (true/false); seed banks ensure plants persist through unfavourable conditions
| float | MortalityDormant| mortality of dormant seeds, percentage
| float | DormancyBreakRate| rate of breaking dormancy and pushing seeds back into active state (high rate = short-lived seed bank)  
| float | GerminationSuccess| probability of germination as percentage (active seeds only)  

## Estimation of simple parameters  
The model traits can be estimated from a (simplified) set of input parameters, which should be readily available from a variety of sources (e.g. Wikipedia):   

| Trait| Estimate | 
| --- | --- |
| HMax | from pictures, wikipedia entries| 
| LifeSpan | rough estimate is usually sufficient |
| MaxDiameter | width of plant or tree crown, estimate from pictures / wikipedia |
| acceptedSoils | can be estimated from general plant description, e.g. "prefers moist soil"--> Loams and Clay-rich soils, no sand |
| growth form| may be grass,herb, shrub, deciduous tree or evergreen tree. The distinction between shrub and tree is not always easy, provide the typical growth form that is expected on the site.
| shade tolerance|Low (full-sun), medium or high (shade-tolerant).
| shape | required if different from cylinder (e.g. trees)|  

For trees and shrubs, the additional parameters are needed:

| Trait| Estimate | 
| --- | --- |
| DBH  | Diameter of the stem, at breast height, estimate from pictures /wikipedia |
| stemRatio | Onset of crown. Base the estimate on general growth patterns. 0.0 if leaf biomass starts at ground level.|

Several of the input parameters can be directly used by the model. These simple rules of thumb and simple math estimate some of the remaining traits: 

| Trait | Estimate |  Comments
| --- | --- |  ---- | 
|MatAge| ``0.5 * lifespan * (1+ShadeTolerance)`` for grasses and herbs; `` 0.1 * lifespan * (1+ShadeTolerance)`` for shrubs and trees|*ShadeTolerance correction moves the inflection point of the Gompertz function for slower-growing but more shade-tolerant plants* |
| Rooting depth| 10cm for grasses, 30 for herbs, 50 for shrubs, 100 for trees|*there are also shallow-rooted trees and deep-rooted grasses, but this estimate shall serve as reasonable entry point*|       
| Size| 10 for grasses and herbs, 30 for shrubs, 60 for trees|
| Dormancy| false| leave false if no other information is available (no seed bank)|
| MortalityDormant| 0.0 if not Dormancy|
| DormancyBreakRate| 1.0 if not Dormancy|
| GerminationSuccess| 1.0 for grasses, 0.6 for herbs, 0.005 for shrubs, 0.0025 for trees. | *Low germination rates ensure that trees and shrubs are relatively sparse on site.*
| Opacity| 1.0 |leave at 1.0 if no other information is avialable|
| Maintenance costs| 0.2 - shade tolerance|

## Estimation of MaxBiomass
Max biomass (dry weight of green leaves) of the plant can be estimated from allometric equations. Usually allometric equations are created for specific subsets of plants; to generalize and keep it simple, we use the following very coarse approximations:  

|Growth form | allometric equation| 
| --- | --- |
| grasses | bm = 0.0006890597 * (diameter)^1.669976 * (height/100)^0.5895024 * 1000 |
| deciduous trees | bm = 0.1 * (DBH)^2.5 *1000 *0.02  |
| evergreen trees | bm = 0.1 * (DBH)^2.5 *1000 *0.05 |
| shrubs | bm = 0.1 * (DBH)^2.5 *1000 *0.05  |
| herbs | bm = 0.1 * (diameter)^2.3 *1000  |
 

*tree/shrub equations estimate total biomass (including stems etc). The last factor corrects to leaf fraction.*

## Estimation of Allocation strategy

The plant may invest its resources into 1) biomass,2) seeds,3) reserve tissues (e.g. roots), or 4) structural biomass and other material that is not being modelled.  
However:
- The model does not explicitly account for "wasted" resources yet. Currently, the waste is lumped into seed production, because seeds are always overly abundant and excess production of seeds is effectively a waste of resources.   
- the trade-off between reserves and investment is not yet explicitly implemented; reserves are added on-top of biomass/seeds, via higher efficiency of shade-tolerant plants.
 
Thus, there are only two forms of investment (biomass and seeds), and the following ruls are established: 

|Growth form | biomass | seeds | notes
|--- | --- | --- | ---
Grasses | 0.75 | 0.25 | -
Deciduous trees | 0.50 | 0.50 | 10% invested in seeds, 40% "wasted" on wood structure
Evergreen trees | 0.40 | 0.60 | 10% seeds, 50%  wood and needles
Shrubs | 0.60 | 0.40 | 20% seeds, 20%  wood structure
Herbs | 0.70 | 0.30 | -

Investment into reserves is dependent on the plant's shade tolerance. More shade-tolerant plants tend to grow slower/stay smaller, but save more of their resources. Low-tolerance plants get a reserve boost of 5%, medium 10% and high tolerance plants 15%.

## Theoretical derivation of conversion efficiency  
**For reference only - the actual conversion efficiency used in the model is described below.**  

ConversionEfficiency describes how many g sucrose can be created from 1 KJ light.This data was not readily available, but can be approximated based on some chemistry/biology: Pure sucrose contains 5647 KJ/mol (combustion energy), at a mass of 342,3g/mol. Hence, a maximum of 0.06062 g could be produced with 1 KJ solar energy. For comparison: an annual light input of 1030.1 KWh/m2 =3,6*10^6KJ (Global Solar atlas, Kopenhagen) makes a maximum of 224 Kg Sucrose/y. Sugar beet production is approx 60T/ha with up to 20% sucrose = 12T/ha = 1.2 kg/m2, which makes an efficiency of 0.5%. This is a realistic estimate of photosynthetic efficiency (https://en.wikipedia.org/wiki/Photosynthetic_efficiency). Hence, a conversion efficient of ``c = 0.06062 * x``, where x is a plant-specific constant, seems reasonable. For example, x = 0.3 would make 0.00018186 g/KJ.  

Whether the conversionEfficiency is suitable for the plant and leads to a positive energy balance depends on SLA and MC; whether it is also sufficient for maintaining the expected biomass growth curve depends further on shape of the plant (height, diameter), growth rate and degree of self-shading. Quite often, it may not provide reasonable results.

## Actual derivation of conversion efficiency
This variable is the trickiest part. To understand the required conversion efficiency at each age step, we need to figure out how the plant grows and successively shades itself.  
For better readibility, all species traits are written in UPPERCASE.
A cylindric shape is assumed for simplicity.

We mostly need:
- MAXDIAMETER[cm]
- HMAX[cm]
- STEMRATIO[0-1]
- MAXVOLUME: ```maxVolume = pi * (MAXDIAMETER/2)^2 * HMAX * (1-STEMRATIO)``` [cm3]

First, we can calculate the growth of the plant in age € [0, LIFESPAN]:  
- ``size(age) = 1 * exp(-exp(-KG * (age - MATTIME)))``; with ``KG = 1.998/(LIFESPAN - MATTIME)`` [unit: percentage];   

The following state variables result from the growth function (change with age i):    
- max attainable biomass under infinite light: ```b_theo[i] = size[i] * MAXBIOMASS``` [unit: g_leaves]
- (root growth: ```RG[i] = size[i] * ROOTINGDEPTH``` [unit: cm]; not required for the caculation)  
- crown volume: ```V[i] = size[i] * MAXVOLUME```[unit : cm3]
- height: ```h[i] = cbroot(size[i]) * MAXHEIGHT```  [unit: cm]
- diameter: ```d[i] = cbroot(size[i]) * MAXDIAMETER * 1``` [unit: cm]; (cylinder shape)



The plant can in principle use the leaf volume to capture energy and produce new leaf biomass. 
- ``biomass gain = (income - maintenance) * allocation``
- ``             = (available_energy[KJ] * CONVERSIONEFFICIENCY[g sucrose/KJ] - b_theo[i+1][g leaf] * MAINTENANCE COST[g suc/g leaf]) * BIOMASS ALLOCATION[g leaf/g sucrose]``; with:
- ``available_energy[KJ] = effective_surface_area[cm2] * light input[KJ/cm2] ``  

We need to figure out the effective surface area, i.e. the area receiving light. 
- In the current light model, a light beam of 3.6\*10^6^ KJ/m2 hits each x/y pair and then travels down the z-column. Thus, each m2 of the site has its own energy model. 
- The plant covers ```area[i] = pi * (d[i]/2)^2```cm². 
- It can gather energy from ```n_columns[i] =ceiling(area[i]/VOXELAREA); min 1```; with VOXELAREA = 10000 cm2  

The following calculations are made independently for each column:

- Each layer shades the layers below. Light passes at 90° from above and there is no diffusion of light. The uppermost layer is thus most efficient, and efficiency attenuates quickly as light passes downwards.  
- The plant covers ```areaPercent[i] = area[i]/(n_columns[i] * VOXELAREA)``` % of the area.  
- light arriving on ground = ``light = (1-areaPercent)^number_layers * (1-area_percent * fracLayer)``; with ``number_layers = h[i] %/% 5cm`` and ```fracLayer = (h[i] - number_layers * 5cm)/5cm```. The second term corrects for the uppermost layer, which is usually not filled along its whole z-axis.      

Finally:
- ``effective_surface_area = (1-light) * VOXELAREA * n_columns[i]``

The effective surface area is smaller than the total leaf area.  

Using this logic, we can calculate the conversion efficiency that is required to let the plant grow according to its growth curve, plus a buffer (depending on shade tolerance). We assume an annual light of 3.6*10^6 KJ/m2 = 360 KJ/cm2:
``req_gain = b_theo[i+1] - b_theo[i]``  [b_theo was calculated in the beginning]  
``possible_gain / req_gain = 1 + ST``   
``==> (available_energy * CE - b_theo[i+1] * MC) * BMALLOC / req_gain = (1+ST)``  
``==> CE = ((1+ST) * req_gain / BMALLOC + b_theo[i+1] * MC)/available_energy``,   
with ``available_energy = (effective_surface_area * 360)``  
and `` effectiveArea ~ f(size)``  
The result is a vector (1 value per age step). The shape of conversion efficiency ~ age relationship is quite complex and species-specific.  

The conversion efficiency used in this approach is, of course, only a coarse adjustment until the model is improved and a more realistic effficiency can be implemented. Drawbacks of the current approach:
- conversion efficiency is age-dependent
- convesrion efficiency does not saturate/decline with light intensity (infinite light->infinite energy)
- The calculation is based on BiomassAlloc; investing heavily in seeds, structure or reserves is not penalized
- 

## Example parametrization
As one example, we here define the parameters for Holcus lanatus. 

### Inputs   

| trait | value |  
| ----- | ----- |
| HMax| 66.2 |
| lifespan| 5 | 
| maxDiameter | 30 |
| acceptedSoils| Loam, LoamySand, Sandyloam, SiltLoam | 
| growth form | grass |
| shade tolerance | low(0.03) |
| shape | cylinder |
| stemRatio | 0.0 |

### Trait estimation  
Here are all traits, based on estimation rules above, except conversion efficiency which is calculated below.

|Trait | Value |Comments
| --- | --- |---|
|MatAge | 1.525 | Manually adjusted|
|LifeSpan|5  |
|HMax|66.2  |
|rootingDepth| 10|
|acceptedSoils|  Loam, LoamySand, SandyLoam, SiltLoam |
|size |10||
|maxDiameter|30|
|opacity| 1.0|
|shape| Cylinder|
|stemRatio| 0.0|
|conversionEfficiency| see below | 
|maintenance Costs|0.17|
|biomassAllocation| 0.75|
|seedAllocation|  0.25|
|maxBiomass|   158.27388038098451133278826933346  | see allometric eq. for grasses |
|Dormancy| false|
|MortalityDormant| 0.0|
|DormancyBreakRate| 1.0|
|GerminationSuccess| 1.0|


### Calculation of growth

This is the expected growth as function of age (gompertz function with lifespan, matTime):  

| age | percentage | root length | ideal_biomass | volume | diameter | height
| ----| ------- |  ----------- | ------------- | --------------- | --------------- |---|
| 0 | 0.0904240 |  0.904240    | 14.31176      | 4231.303  | 13.46529 | 29.71341
| 0.1 | 0.1034192 |  1.034192  | 16.36855      | 4839.399  | 14.08170 | 31.07361 
| 1 |0.2586275  |  2.586275    | 40.93397      | 12102.219 | 19.11376 | 42.17770 
| 2 | 0.4671940 |  4.671940    | 73.94461      | 21861.887 | 23.27843 | 51.36773 
| 3 | 0.6516551 |  6.516551    | 103.13999     | 30493.565 | 26.00921 | 57.39366 
| 4 |0.7858563  | 7.858563     | 124.38053     | 36773.379 | 27.68443 | 61.09032 
| 5 | 0.8731864 |  8.731864    | 138.20260     | 40859.904 | 28.67413 | 63.27425


### Estimation of Conversion efficiency
| Age | effective_area | energy | required_CE
| ----| --------------- | ----- | ------------
| 0 | 816.9785  | 294112.3   | 0.000019065  [not required]
|0.1|929.2385  |334525.9    |0.000121651
| 1 | 2176.7167 |  783618.0| 0.000073895
| 2 | 3602.1819 | 1296785.5 | 0.000044440
| 3 | 4654.3336 | 1675560.1 | 0.000030029
| 4 | 5314.9525 | 1913382.9 | 0.000022200
| 5 | 5701.2923 |  2052465.2 | - [not required]

## Resources and biomass
Assuming resources are reset to 0 every timestep:

| age | biomass | eff_area | energy |  resources harvested | bm gained | seeds |surplus
| ----| ------- | -------- | ------ |  ---------------- | --------- | ------|----
| 0.1 | 16.36855 |  816.9785  | 294112.3  | 40.695283  |25.30238  | 8 | 0.98261672
| 1   | 40.93397 | 929.2385  |334525.9  | 57.905193 | 34.00096 |11 | 1.32042551 
| 2   | 73.94461 | 2176.7167 |  783618.0 | 57.628780 | 30.07124 |10 | 1.16781503 
| 3   | 103.13999| 3602.1819 | 1296785.5  | 50.315039  |21.87776 | 7 | 0.84962181  
| 4   | 124.38053| 4654.3336  | 1675560.1  | 42.476752  | 14.23673  |4 | 0.55288280
| 5   | 138.20260| 5701.2923 |  1913382.9 | 


# Developer notes
will be moved elsewhere soon.  
The model consists of A graph with voxels, containing soils and communities of individuals. The individual is composed of submodels.   

## Flyweight pattern for species traits  
Because many individuals of the same species are created, the species (traits) are implemented as flyweights - the runtime-constant trait variables are stored once and pointed-to by each individual. Example: Growth model has current age as state variable, and lifehistory\*   with lifespan. All individuals of the species share the same lifespan but may have different ages.  
 

## plant geometry and growth patterns
The individuals are assigned to a "home" voxel, where they principally grow, but they can of course spill into adjacent voxels if big enough. They can grow in positive z-direction only, but in any x-y direction. The plant can be split into a crown and a stem (though the stem may have 0 height). To ease calculations, all plant crowns have a circular base (cylinders, spheres, cones). They are always centered at the midpoint of a voxel base (x=0.5, y = 0.5, z=0.0). The voxels that intersect with the plant are determined by radius, height and shape of the crown, and of course height of the stem. Stems are always 100% straight and crowns perfectly round.

## Logic of  voxel ordering  

One could, in principle, store a vector\<Individuals\> in the environment. At each time step one would loop over all individuals, find voxels they grow into and assign their volume to the voxels (this assignment is needed for shading calculations). Because individuals constantly die and get recreated, individuals are unsorted, and the **voxel lookup** would have poor cache locality. Parallelization may also be tricky (race conditions).  
Instead, the model can also traverse through the voxels, going from z_min upward. It collects the individuals and their volumes along the way, and for each layer: 1) calculates diameter of each individual at height z[i], and volume between z[i] and z[i+1], 2) distributes the volume equally across the voxels falling into diameter, 3) removes the individual if no volume is left. This would be more cache-efficient as all voxels are queried exactly once and in natural ordering. The individuals are also only added once to the list until eventual removal and the list should stay reasonably small.  
There is one catch though: the plant model uses the same voxel resolution as GH for environmental variables, assigning home voxels, and soil -  but it requires finer resolution for light calculations. To accomodate the resolution differences, it traverses through the z-voxel layers and collects the individuals therein. Then, it goes through l layers (20 per z-layers, with 5cm layers in 1.0 m resolution), calculates diameters and volumes (of the 5cm sections) and spills plant volumes along the plant diameter; inside the l-loop, it may continue into layers which lie in higher z-voxels if the plant is larger than 1 voxel high; finally, when all individuals and voxels are done, it continues normally with the next z. Possibly the z-voxel resolution could be removed at some point, making the whole model run on finer resolution than GH (with some clever filtering for steps that do not require fine resolution).  
Note that fine resolution is also required in an entity-first approach. 

## shape classes

- Shape is an abstract base, ShapeCylinder, ShapeCone and ShapeSphere are derived classes. They calculate volume of a height section (frustum etc) diameter and height as percentages on a unit scale (0-1). 
- ShapeTraits is - in analogy to LifeHist, Resourcealloc etc - a class storing species-specific traits such as maximal volume and height, aspect, and Shape. Also contains the stem section. It does not store state variables (things that change over time), but enables calculating actual diameter, volume and height, taking a (relative) growth percentage as input.

- PlantShape is a class storing state variables, analogous to PlantGrowth and PlantRessource. stores relative size that follows a Gompertz-growth function and further variables for environment-dependent effects (density). Also responsibe for volume - surface area conversion.



# References
Grimm V, Berger U, Bastiansen F, Eliassen S, Ginot V, Giske J, Goss-Custard J, Grand T, Heinz SK, Huse G, Huth A, Jepsen JU, Jørgensen C, Mooij WM, Müller B, Pe’er G, Piou C, Railsback SF, Robbins AM, Robbins MM, Rossmanith E, Rüger N, Strand E, Souissi S, Stillman RA, Vabø R, Visser U, DeAngelis DL (2006). A standard protocol for describing individual-based and agent-based models. Ecological Modelling 198, 1-2, 115-126. doi: https://doi.org/10.1016/j.ecolmodel.2006.04.023.

Grimm V (2020). The ODD protocol: An update with guidance to support wider and more consistent use. Ecological Modelling, 428, 109105. doi: https://doi.org/10.1016/j.ecolmodel.2020.109105.

Tjørve KMC, Tjørve E (2017) The use of Gompertz models in growth analyses, and new Gompertz-model approach: An addition to the Unified-Richards family. PLOS ONE 12(6): e0178691. https://doi.org/10.1371/journal.pone.0178691
