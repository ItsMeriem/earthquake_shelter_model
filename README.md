<div align="center">
  <img src="https://github.com/ItsMeriem/earthquake_shelter_model/blob/97b6562666e284b08d66f4309d52d75ffb57d056/Images/RClogo.png" width="30%" align="left">
  <img src="https://github.com/ItsMeriem/earthquake_shelter_model/blob/97b6562666e284b08d66f4309d52d75ffb57d056/Images/CMUHeinzlogo.png" width="30%" align="right">
</div>

<br clear="both">


<h1 align="center"> Geospatial Modeling of Real Time Earthquake Impacts to Housing </h1>

# Introduction:

For my masters capstone at Carnegie Mellon University, we worked with the American Red Cross to develop a predictive tool to estimate the number of people expected to seek shelter in the aftermath of an earthquake. This product is **the first open source earthquake model** to provide shelter demand estimates. It is expected to reduce the time needed to deploy life-saving resources by 3 days, ensuring the RC can help people as rapidly as possible.

In the event of an  earthquake, one of the most urgent needs is to rapidly assess how many impacted individuals will require public shelter. Currently, the American Red Cross relies on tools like FEMA’s HAZUS models to assess earthquake impacts; however, these tools are technically complex, inflexible, and difficult to operate in urgent, "no-notice" disaster scenarios. Additionally, no existing models provide precise estimates of shelter demand.

To address these limitations, our team developed an open source model to rapidly estimate shelter needs following an earthquake. The tool integrates real-time United 
States Geological Survey (USGS) ShakeMap data to assess earthquake intensity, along with publicly available building data, HAZUS-derived damage functions to estimate structural damage, and social vulnerability indicators. The graph below demonstrates our methodology: from earthquake to structural building vulnerability to social vulnerability. Our methodology is designed to be accessible and practical for the American Red Cross analysts, requiring minimal technical expertise and avoiding the need for specialized software.

<p align="center">
    <img src="https://github.com/ItsMeriem/earthquake_shelter_model/blob/97b6562666e284b08d66f4309d52d75ffb57d056/Images/MethodologyWorkflow.png">
</p>

# Data
The data is either automatically downloaded when the code is ran or it is already included in the repo. No additional data downloads are needed. Here's a breakdown of the data used: 

| Data Source | Description | 
|-------|--------|
|USGS Shakemap|ShakeMaps provide near-real-time maps of ground motion and shaking intensity following significant earthquakes. We use the ShakeMap API to search for earthquake information using the earthquake's Event ID. [Link](https://earthquake.usgs.gov/data/shakemap/)|
|Census Tract Geographic Boundaries| Our tool automatically downloads the spatial shapefiles for the geographic boundaries of census tracts and counties|
|The 2020 Decennial Census| We use tract-level population counts from the 2020 Decennial Census. [Link](https://data.census.gov/table/DECENNIALPL2020.P1?t=Populations+and+People&g=040XX00US06$1400000)|
|FEMA USA Structures|The USA Structures geo database has building-level data, indicating the exact spatial location of each building. [Link](https://disasters.geoplatform.gov/USA_Structures/)|
|General Building Stock Data| Tract-level breakdown of structure types. These can either be derived through the National Structure Inventory or through the Hazus software|
|HAZUS Damage Function Variables|These are a set of cumulative distribution functions whose variables are dependent on structure type and seismic design code. They provide a framework to estimate the probability that a structure might meet or exceed Slight, Moderate, Extensive, or Complete damage based on the peak ground acceleration (PGA) of the area.|
|Social Vulnerability Index|The Social Vulnerability Index (SVI) is a place-based index designed to identify and quantify communities experiencing social vulnerability. [Link](https://www.atsdr.cdc.gov/place-health/php/svi/index.html)|

# Methodology

To estimate the number of people seeking shelter after an earthquake, we first estimate the structural building damage caused by the disaster (physical vulnerability), then estimate the  habitability of those buildings, and finally assess the social vulnerability of residents.

<p align="center">
    <img src="https://github.com/ItsMeriem/earthquake_shelter_model/blob/97b6562666e284b08d66f4309d52d75ffb57d056/Images/MethodologyWorkflow.png">
</p>

Here's a breakdown of each step and the relevant script:

- `o1_getshakemap.py` : Fetches earthquake data (the ShakeMap) through USGS API and extracts key intensity metrics (MMI, PGA and PGV).

- `o2_download_census.py`: Downloads and merges TIGER/Line Census Tract shapefiles (geometries of tract boundaries) into a single GeoPackage.

- `o2_census_intersect.py`: Clips ShakeMap layers to census tract geometries so that for each census tract, we compute the max, min, and mean earthquake intensity statistics.

- `o3_get_building_structure.py` [Optional]: This script processes and aggregates large scale building-level data (a data point for each building in the country) to the Census tract level. This script is pre-run and its results are saved to the repo. The `main.py` does not run this unless specified by the user. We recommend re-running this every 5 years to account for major building changes. 

- `o3_clip_eventdata_buildingstocks.py`: Uses tract-level building data to estimate building count and structure type in tracts where the earthquake occured.

At this stage, we have the data for which tracts the earthquake occured in, what building types are in these tracts, and how intense the earthquake was at each location. Using this information, we can now **estimate the structural damage**:

- `o4_TractLevel_DamageAssessmentModel.py`: This module estimates earthquake-induced building damage at the census tract level using fragility functions and peak ground acceleration (PGA) values.

We now know how many buildings were damaged and the extent of their damage (slight, moderate, extensive, or complete). However, individuals decide to evacuate and seek shelter for a multitude of factors beyond building damage alone. Individuals are more likely to evacuate if they have lost access to utility services and are more likely to seek shelter if they are socially vulnerable:

- `o5_bhi.py`: This module estimates the number of people who are likely to evacuate based on building damage and estimated utility service disruptions.

- `o5_svi_module.py`: This module estimates shelter demand as a percentage of people evacuating, based on social vulnerability.`

# How to Run Our Model?

- Update the `params` dictionary in the `main.py` script for the earthquake you would like to run. The current parameters are set for the Napa 2014 Earthquake.
      - Required Parameter: The `Event ID` parameter must be changed as it identifies the earthquake for which to extract the shakemap.
      - Optional Parameters: All other parameters are optional and can be left as is. They represent model assumptions.

- Run `main.py` 

# References 

Data sources are linked above. Additional references:

[A New Approach to Modeling Post-Earthquake Shelter Demand: Integrating Social Vulnerability in Systemic Seismic Vulnerability Analysis by Khazai et al.](https://www.researchgate.net/publication/253276822_A_New_Approach_to_Modeling_Post-Earthquake_Shelter_Demand_Integrating_Social_Vulnerability_in_Systemic_Seismic_Vulnerability_Analysis)

[A Predictive Earthquake Model Written in Python](https://medium.com/new-light-technologies/a-predictive-earthquake-damage-model-written-in-python-e1862518fd92)
