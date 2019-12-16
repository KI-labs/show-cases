# Stream and plot billions of GPS points

*Data hub* is a cloud implementation of scalable processing, storing, analyzing and visualization data pipeline focused on finding geographical points of interest.

## Motivation

Analysing vehicle telematic data could open a new horisonts for car manufactures like route optimisation, predictive maintenance, driver behaviour detection, etc.

In this project we built a system to collect and store world-wide vehicle telematics data in a way that different data-supported
ideas/hypotheses can be rapidly tested.

Here we are focusing on a use-case derived from analysing the
data that was collected: the determination and visualization of the
places where the vehicles tend to stop, to fuel, to load unload goods;
as well as gathering an additional information about the these places. 

## How does it work 


### Architecture 

<br>
<img src="data/figures/tdh/tdh diagram.jpg" alt="tdh diagram" width="720"/>

All of the components are running inside an Azure subscription.

- Event Hubs are used to collect vehicle streaming data (which at the
  moment is not real-time data).

- We have several clusters for different Event Hubs inside Azure
  databricks running individual spark jobs which extract, transform and
  store data in Apache Parquet format in Azure Data Lake.

- From each of these spark jobs, we also send some important progress
  metrics to a Graphite instance. We run Grafana instance on top of
  Graphite which provides interactive monitoring dashboards.

- We create another analytics cluster inside Databricks which reads our
  stored data, performs key analysis, aggregates information and stores
  it in a PostgreSQL DB. 
  
- The aggregated data is used in a React App for creating dashboards and
   visualizations.


### Web app

Even though there are tools for data visualization (PowerBI, Tableau)
they do not give us the desired level of flexibility when specific
customer requirements have to be fulfilled. For this reason we decided
to use ReactJS for the frontend and postgreSQL for data storage.

The React based UI is able to change data, without reloading the page.
It is fast, scalable, and simple. The rendering of the 3D graphics is
done with libraries like
[React-map-gl](https://uber.github.io/react-map-gl/#/) and
[Deck-gl](https://deck.gl/#/). 


The backend of the app is fetching data from the PostgreSQL DB which is
then visualized. The postGIS extension of a PostgreSQL DB allows us to
do efficient location queries. A typical example is finding points of
interest within a given distance from a given point or from a vehicle
driving trajectory. To these results we can add results from using other
APIs like Google Places.

A detailed explanation of the application capabilities with screenshots 
can be found below:

#### Explore individual vehicle trips

 - Trajectory of a vehicle for a given time period     
    The trajectory is divided 
    into different segment based on events where the truck is idle for a long period of time.
    A business person can see the following information: 
    - vehicle telematics data (figures in the left panel) for a selected trajectory
    segment (trajectory highligted in dark red)
    - start/end time of a trip (trajectory) and vehicle id in the panel in the
    upper right corner; 
    - option to choose multiple vehicles from the top panel   
    - option to show/hide different layers of the graph: Google points of interest,
    most common stop locations among all vehicles, stop locations of the selected
    vehicle (right panel)
    - option to get data for a given time range (right panel)
    <br>
    <img src="data/figures/tdh/react_vis_a1.png" alt="react ui 2" width="720"/>
     

 - Places where the selected vehicle has stopped  
    If several places are too close to each other they are visualized as a 
    single point (green dots on the map with a number equal to the number 
    of these places). You can see:     
    - a panel with trajectory events that appears if you select a point (in the figure below  
    a point with two such events is selected) 
    - a panel with information about the loaded/unloaded weight of a vehicle
    <br>
    <img src="data/figures/tdh/react_vis_a2.png" alt="react ui 2" width="720"/>

 - Places where the vehicles tend to stop   
    If several places are too close to each other they are visualized as a 
    single point (orange dots on the map with a number equal to the number 
    of these places). You can see:   
    - a panel with trajectory events that appears if you select a point (
    in the current image a point with eight such events is selected)  
    - a panel with information about the "waiting time distribution", i.e. the duration for 
    which the vehicles that tend to stop at this place are idle.  
    - a panel with information about the hours in the day in which the selected place is occupied.  
    - a panel with information about the days in the week in which the selected place is occupied.
    <br>
    <img src="data/figures/tdh/react_vis_a3.png" alt="react ui 3" width="720"/>
           

 - Google places   
   For every selected place the user has the option to get nearby places of a specific type (service). 
   These places are visualized as red dots.   
   <br>
   <img src="data/figures/tdh/react_vis_a4.png" alt="react ui 4" width="720"/>

#### Explore areas where vehicles tend to stop

For every selected hexagon the user can see:
   
 - a panel with information about the hours in the day in which the 
 selected area is occupied with idle vehicles.  
 - a panel with information about the days in the week in which the 
 selected area is occupied.
 - a panel with information about the amount of time for which the
 vehicles tend to stop in this area.
 <br>
 <img src="data/figures/tdh/react_vis_a5.jpg" alt="react ui 5" width="720"/>

## Possible project extensions:

Given the amount and quality of data being collected there are a lot of
directions in which a project can be extended and a lot of improvements
that can be added to the UI:

- Data augmentation:
  * the gps data stream is not consistent; by using map-matching tools
    like [Barefoot](https://github.com/bmwcarit/barefoot) or
    [GraphHopper](https://github.com/graphhopper/map-matching) we can
    reliably guess which route the vehicle has used if there are any
    time gaps in the gps data stream;

- New information obtained from data analysis:
  * fuel consumption, vehicle load capacity utilization

- Predictions from supervised/unsupervised ML models:

  * predict and visualize the possible vehicle routes. 

  * display possible shipping orders that can be taken by the truck by 
    taking into account the truck position at a given time.

- Real time analysis and data collection: at the moment, the data is
  processed in batches but the adjustment of the visualization ideas to
  real-time data processing and online ML is feasible.

## Tech stack

- Azure Databricks, pySpark, Scala 
- React, PostgreSQL 
- Graphite, Grafana