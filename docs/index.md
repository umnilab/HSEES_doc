# Welcome to METS-R documentation!

METS-R is a high fidelity, parallel, agent-based evacuation simulator for multi-modal energy-optimal trip scheduling in real-time (METS-R) at transportation hubs.
```{eval-rst}
.. figure:: res/framework.png
  :width: 600
  :alt: Alternative text
  :align: center
  
  figure : METS-R overview
```

## How it works?

The overall program consists of two modules. The first one is the simulation module and the second one is the high-performance computing (HPC) module that collects data from all simulation instances to train the online routing models.

Within each simulation instance, a series of contexts are created to hold all classes and variables. On top of that, an event scheduler is initialized at the beginning of the simulation to regularly call certain procedures. The procedures and corresponding contexts are connected in the framework in the figure below.

The rest of this document will introduce the detailed implementation of major functions according to the indexes shown in the framework.

```{eval-rst}
.. figure:: res/framework2.png
  :width: 600
  :alt: Alternative text
  :align: center
  
  figure : main components  of METS-R
```

# Getting Started

## Building and Running the Simulator
1. Download and install *Eclipse IDE for Java Developers* from [here](http://www.eclipse.org/downloads/packages/release/2021-03/r/eclipse-ide-java-developers)
2. Install *Repast Simphony* plugin in Eclipse using the instructions provided [here](https://repast.github.io/download.html#update-site-install)
3. Clone the METS-R repository using `git` to a suitable location
    ```
    git clone <METS-R GitHub URL>
    ```
4. Open Eclipse and goto File -> *Open Projects from File System…*
5. In the *Import Projects from File System or Archive* window click on *Directory* and open the *doe-mets-r* directory you cloned in step 3
6. Under *Folder* check *doe-mets-r/EvacSim* only and click *Finish*. This should open the METS-R java code in Eclipse
7. Go to *Run -> Run Configurations* 
8. In *Main* tab use (These values may be auto-filled, in that case skip steps 8 and 9),
    ```
    Project : EvacSim
    Main class : repast.simphony.runtime.RepastMain
    ```
9. In *Arguments* tab use, (note that VM arguments can be changed depending on available memory on your machine)
    ```
    Program Arguments : "${workspace_loc:EvacSim}/EvacSim.rs"
    VM arguments : -Xss1024M -Xms1024M -Xmx8G
    ```
10. Click Run and you should see the *Repast Symphony* simulation window (You can also use *Run* button in toolbar)
    ```{eval-rst}
    .. figure:: res/repast_window.png
      :width: 600
      :alt: Alternative text
      :align: center
      
      figure : *Repast Symphony* simulation window
    ```
11. Click on *Initialize Run* (power) and then *Start Run* button to begin the simulation

Above steps runs the simulator without the HPC module. In order to run the HPC module first complete above steps (which will build the Java code)
and then follow the steps in [*Running Simulation with HPC Module*](#running-simulation-with-hpc-module).

## Data Requirements

To run the simulation, the following data need to be prepared. These files are already included in the source repository and can be configured 
using the `Data.properties` file. 

1. Zone shapefile (Point), map of zone centroids.
2. Charging Station shapefile (Point), map of charging stations.
3. Road shapefile (Polyline), map of the roads. Should contain the information on which lane belongs to which road.
4. Lane shapefile (Polyline), map of the lanes, generated from road shapefile. This shapefile should contain lane connection information.
5. Demand profiles (csv), OD flow for different time of the day.
6. Background traffic speed (csv), traffic speed of each link for different times of the day.

## Running Simulation with HPC Module

To run the HPC module first download and build the simulator by following the steps in [*Building and Running the Simulator*](#building-and-running-the-simulator)

**NOTE** : HPC module is only tested on Ubuntu/Linux machines

* You must download the `METSR_HPC` code to run the HPC module
```
git clone git@github.com:tjleizeng/METSR_HPC.git
cd METS_HPC
```
* Modify the existing ```run.config.json``` file with the configurations you want
    * `java_path` - set the absolute path of your java installation
    * `java_options` - change the jvm mempry limits depending on your machine specs
    * `evacsim_dir` - absolute path of evacsim directory
    * `groovy_dir` - absolute path of your groovy installation
    * `repast_plugin_dir` - absolute path pf your repast plugin installation
    * `num_sim_instances` - number of parallel simulation instances you need to run
    * `socket_port_numbers` - specify a list of free socket port numbers in your machines seperated by comma. You need to specify N ports where N is the number of parallel simulation instance you want to run. If you don't know free port numbers, run ```FindFreePorts.java``` using eclipse. This will print up to 16 free ports numbers available in your machine.

* Finally run,
```
python3 run_hpc.py run.config.json
```
This will start the simulation instances in parallel and run RDCM. All the files related to each simulation run will be created in a seperate directory ```sim_<i>``` where ```i``` means i'th simulation instance.

### Operation of HPC module
`run_hpc.py` script uses this information to launch multiple simulation instances as independent processes. It also runs the Remote Data Clients (RDClient) in separate python threads to listens to the simulation instances and record the data received. RDClients are launched asynchronously i.e. simulation socket server does not have to be up before launching the corresponding RDClient. RDClient will wait until the simulation socket server is up before establishment a connection. After establishing the connection RDClient will continue to receive messages until the simulation is finished and socket connection is terminated.

**IMPORTANT** :  All messages are sent and received in JSON format. 

### Implementing the ML algorithms inside the HPC module
ML algorithms for route selection must go in `rdcm.py`. For example MAB algorithm can be called inside 'run_rdcm' function to compute the optimal route candidate. Computed route result can be sent to simulation instance in JSON format using something,
```
ws_client.ws.send(route_result_json)
``` 

## Running Demand Prediction Module
demand prediction module is hosted in METRS_HPC repo. Clone it if you already have not done that.
* Download and process the data required to train the models.
```
git clone git@github.com:tjleizeng/METSR_HPC.git
cd demand prediction
python 1.Download_NewYork_Taxi_Raw_Data.py
python 2.Process_NY_Taxi_Raw_data.py
```
You can change the `hub` variable in `2.Process_NY_Taxi_Raw_data.py` in order to get the prediction for different hubs (i.e. `PENN`, `JFK`, `LGA`)

* Next train the model and generate the prediction `.csv` file.
```
python PCA_aggregation_RF_prediction.py
```
Same as before, change the `hub` variable as necessary. This will train the prediction model and write the prediction result to a file named ```<HUB>PCA.csv```. 

Java class `DemandPredictionResult.java` can be used to query the prediction result from within the simulator code. There are two main functions in this class.
* ```public double GetResultFor(String hub, String date, int hour, int taxiZone)``` : This function can be used to query the prediction result. You need to specify the following to get the prediction result
  * `hub` : demand hub name (i.e. `PENN`, `JFK`, `LGA`)
  * `date` : use the format ```<year>-<month>-<date>``` (eg. ```2018-12-31```)
  * `hour` : hour of the day 1-24
  * `taxiZone` : taxi zone number
* ```public void RefreshResult() ``` : Use this function to repopulate the prediction result from the `.csv` file. This is useful if you want to update the demand prediction while the simulator is also running.


# Advanced Simulation Options

Various parameters related to simulation can be changed using `EvacSim/data/Data.properties` file. This section describe the meaning of the simulation parameters
available in this properties file. Note that all file locations must be specified relative to the `EvacSim` directory.
* `ROADS_SHAPEFILE` : location of the shape file containing the road network 
* `LANES_SHAPEFILE` : location of the lanes shape file
* `ZONES_SHAPEFILE` : location of the zones shape file
* `CHARGER_SHAPEFILE` : location of the charging stations shape file
* `CHARGER_CSV` : 
* `ROADS_CSV` : 
* `LANES_CSV` : 
* `BT_EVENT_FILE` : background traffic event CSV file
* `DM_EVENT_FILE1` : 
* `DM_EVENT_FILE2` : 
* `EVENT_FILE` : 
* `EVENT_CHECK_FREQUENCY` : 
* `MULTI_THREADING` : enable multi-threading for METIS graph processing algorithms (`true/false`)
* `N_THREADS` : number of threads to use for METIS graph processing 
* `N_PARTITION` : number of graph partitions in METIS 
* `PART_ALPHA` : parameters in graph partitioning 
* `PART_BETA` : parameters in graph partitioning 
* `PART_GAMMA` : parameters in graph partitioning 
* `SINGLE_SHORTEST_PATH` : enable single shortest path algorithm (`true/false`)
* `K_SHORTEST_PATH` : enable k-shortest path algorithm (`true/false`)
* `ECO_ROUTING` : enable eco routing (`true/false`)
* `ECO_ROUTING_BUS` : enable eco-routing for buses (`true/false`)
* `NUM_CANDIDATE_ROUTE` : 
* `N_SHADOW` : Number of future road segments to be considered in counting shadow vehicles

# Developer Notes

METS-R simulator is written in `java` and it consists of various classes to represent different entities in a transportation network. This section describes main components of the simulator including the variables and functionalities of the important classes used.

## Main Simulation Loop
The main simulation loop consists of different events including demand generation, vehicle movement, charging, data collection and refreshment of environmental information.

```{eval-rst}
.. figure:: res/mainloop.png
  :width: 600
  :alt: Alternative text
  :align: center
  
  figure : simulation loop
```

## Map Partitioning
We use [METIS](http://glaros.dtc.umn.edu/gkhome/metis/metis/overview) to divide the road network into several partitions to balance the computational time among different threads. 

## Event Scheduler

All procedures are managed by different event schedulers, each event is triggered regularly with a customized period. 

```
//schedule all simulation functions
scheduleStartAndEnd();
scheduleRoadNetworkRefresh();
scheduleFreeFlowSpeedRefresh();
scheduleNetworkEventHandling();
schedulePassengerArrivalAndServe();
scheduleChargingStation();


// Schedule parameters for both serial and parallel road updates
if (GlobalVariables.MULTI_THREADING){
  scheduleMultiThreadedRoadStep();

} else {
  scheuleSequentialRoadStep();

}

// set up data collection
if(GlobalVariables.ENABLE_DATA_COLLECTION){
  scheduleDataCollection();
}
```

## CityContext Class

**Description** 

In the initialization stage, the simulation program will use all provided data to create a Class called CityContext to hold all instances and variables. 

**Variables** 

 * `edgeLinkID_KeyEdge <Edge, Integer>`
 * `edgeIDNum_KeyEdge <Edge, Integer>`
 * `edgeIDs_KeyIDNum <Integer, Edge>`
 * `lane_KeyLaneID <Integer, Lane>`
 * `road_KeyLinkID <Integer, Road>`
 * `junction_KeyJunctionID <Integer, Junction>`
 * `nearestRoadCoordCache <Coordinate, Coordinate>`

**Key Functions** 

 * `CityContext()`
 * `createSubContexts()`:  Add Road, Junction, Lane, Zone and Charging Station Context
 * `buildRoadNetwork()` 

``` 
Geography<Lane> laneGeography = ContextCreator.getLaneGeography()
Iterable<Lane> laneIt = laneGeography.getAllObjects()
junc1.addRoad(road)
junc2.addRoad(road)
Iterable<Road> roadIt = roadGeography.getAllObjects()
roadMovementFromShapeFile(r)
laneConnectionsFromShapeFile(r) 
```

  1. Get geography of lane and road: laneGeograph and roadGeography
  2. Create junctions of roads, maintain two dictionaries of junctions*roads and roads*junctions.
  3. Store three dictionaries of the edge*linkID, edge*roadID and roadID*edge.
  4. Store dictionaries of lane*laneID, lane*road
  5. Set upstream and downstream roads
  6. Set upstream and downstream lanes
* `roadMovementFromShapeFile(Road road)`
* `void laneConnectionsFromShapeFile(Road road)`
* `void modifyRoadNetwork()`
* `createNearerstRoadCoordCache()` : Create dictionary of zone\_center its closest road_coordinates


## MetisPartition

**Variables**

 * `npartition`
 * `PartitionedInRoads ArrayList<ArrayList<Road>>` : 2D array list for roads that lie entirely in each partition
 * `PartitionedBwRoads ArrayList<Road>` :  Array list for roads that lie in the boundary of two partitions
 * `PartitionWeights ArrayList<Integer>`
 * `partition_duration Integer` : Last time of updating partition

**Functions**

 * `first_run()` : Unused, call partition(metisGraph, `npartition`) to get the partitioned graph
 * `check_run()` : Check if it is time to repartition the graph.
 * `run()` : The same as `first_run`, call `partition(metisGraph, npartition)` to get the partitioned graph
 * `partition(metisGraph, npart)` Partition metisGraph into n parts, there is a piece of repeated code in this function
 * `verify()` : Unused, call `metisGraph.verify()`

## GaliosGraphConverter

**Description**
To call the partition, we need to convert the original graph (RepastGraph) to GaliosGraph.

**Variables**

 * `RepastGraph`
 * `roadNetwork`
 * `GaliosGraph`
 * `metisGraph`
 * `cityContext`

**Functions**

 * `BwRoadMembership ArrayList<ArrayList<Ineger>>` : `N_bw*2` dimension array that holds the membership of each boundary road
 * `paritionWeights`
 * `RepastToGaliosGraph(mode)` : Convert from Repast graph to Galios graph mode=true, load vertex weight as the number of vehicles*1,  
mode=false, vertex weight =1
 * `GaliosToRepastGraph(resultGraph, nparts)` : Convert from Galois graph to Repast graph: Only need the the partitioned road sets and the
roads between partitions, call `cityContext.findRoadBetweenJunctionIDs`

## ROUTING 
Shortest path routing involves two layers: the first layer (`RouteV` class) converts the origin and destination coordinates into junction IDs; then the second layer (`VehicleRouting` class) calculates the shortest path between the junctions using Weighted JGraph.

## RouteV Class

**Variables**

 * `Network <Junction>` : roadNetwork
 * `VehicleRouting` : vbr

**Functions**

 * `createRoute()` : Initialize the router with necessary data.

``` 
vehicleGeography = ContextCreator.getVehicleGeography()
junctionGeography = ContextCreator.getJunctionGeography()
roadNetwork = ContextCreator.getRoadNetwork()
roadGeography = ContextCreator.getRoadGeography()
cityContext = ContextCreator.getCityContext()
geomFac = new GeometryFactory()
vbr = new VehicleRouting(roadNetwork)
```

 * `void updateRoute()`

``` 
vbr.calcRoute()
validRouteTime = (int) RepastEssentials.GetTickCount()
```

 * `vehicleRoute(Vehicle vehicle, Coordinate destination)` : For vehicle v compute a route to a given destination coordinate
``` 
Coordinate currentCoord = vehicleGeography.getGeometry(vehicle).getCoordinate()
Road destRoad = cityContext.findRoadAtCoordinates(destCoord, true)
Road currentRoad = vehicle.getRoad()
Junction curDownstreamJunc = vehicle.getRoad().getJunctions().get(1)
Junction destDownstreamJunc = getNearestDownStreamJunction(destCoord,destRoad)
return vbr.computeRoute(currentRoad, destRoad, curDownstreamJunc, destDownstreamJunc)
```

 * `void printRoute(List<Road> path)` : Print the route by `linkid`

``` 
System.out.print("Route:")
for (Road r : path) {
System.out.print(" " + r.getLinkid())}
System.out.println()
```

 *  `GetNearestRoadCord(Coordinate)` : Returns the nearest road to a given point
 *  `GetNearestJunc(Coordinate coord, Road road)` : Returns the nearest junction for a road
 *  `GetNearestUpstreamJunc (Road, Coordinate)` : Returns the nearest upstream junction
 *  `GetNearestDownstreamJunc (Road, Coordinate)` : Returns the nearest downstream junction

## VehicleRouting Class

**Variables**
 * `Network<Junction>` : network
 * `WeightedGraph<Junction, RepastEdge<Junction>>` : transformedNetwork
 * `CityContext` :  cityContext
 * `VehicleRouting(Network<Junction> network)`

**Functions**
* `VehicleRouting(Network<Junction> network)` : Initialize the data for route calculation.
``` 
this.cityContext = ContextCreator.getCityContext()
this.network = network
Graph<Junction, RepastEdge<Junction>> graphA = null
graphA = ((JungNetwork) network).getGraph()
JungToJgraph<Junction> converter = new JungToJgraph<Junction>()
this.transformedNetwork = converter.convertToJgraph(graphA)   
```

 * `calcRoute()` : Transformer network into Jgraph.

``` 
graphA = ((JungNetwork) network).getGraph()
JungToJgraph<Junction> converter = new JungToJgraph<Junction>()
this.transformedNetwork = converter.convertToJgraph(graphA)
```

 * `computeRoute(currRoad, destRoad, currJunc, destJunc)` : Compute a sequence of roads from current Road to destination road using K-shortest path algorithm or single shortest path algorithm. 

``` 
KShortestPaths<Junction, RepastEdge<Junction>> ksp = new KShortestPaths<Junction, RepastEdge<Junction> (transformedNetwork, currJunc, K)
List<GraphPath<Junction, RepastEdge<Junction>>> kshortestPath = ksp.getPaths(destJunc)	
for (GraphPath<Junction, RepastEdge<Junction>> kpath : kshortestPath) {
    pathLength.add(kpath.getWeight())
}
for (int i = 0 i < kshortestPath.size() i++) {
    total = total + Math.exp(*theta  * pathLength.get(i))
}	
```	

##  HPC Module

Multiple simulation instances can run and send data over a socket to a remote host to analyze and build useful machine learning models (such as multi-armed bandits based routing). All these connections are managed by the *RemoteDataClientManager* (i.e. RDCM). RDCM's job is to manage these data senders (i.e. multiple simulation instances) and receive their data to separate buffer to be used by the ML algorithms. RDMA is independent from the simulator and run as a separate process.

```{eval-rst}
.. figure:: res/arch.jpg
  :width: 600
  :alt: Alternative text
  :align: center

  figure : communication architecture for distributed data generation
  
```
## RemoteDataClient Class 

This class is the client end of the data communication with the simulation instance. When the simulator sends messages RemoteDataClient receives them (whenever a client receives a message its ```OnMessage``` method is invoked. For each simulation instance there is a dedicated RemoteDataClient.
    
## RemoteDataClientManager Class

This class is used to manage all the RemoteDataClients. Each RemoteDataClient is executed in a separate thread. All the data analyzes routines must be called from this class (i.e. ML algorithms).



# References

    





