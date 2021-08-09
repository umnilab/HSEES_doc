# [Doc] A-RESCUE 3.0

# About

A-RESCUE 3.0 is an agent-based road traffic simulator specifically designed for urban hurricane evacuation simulation. It is created as a part of the NSF-funded [Hazard SEES](https://www.nsf.gov/awardsearch/showAward?AWD_ID=1520338) project. The conceptual basis and justification for this simulator are presented in the first two papers: [**Ukkusuri et al. (2017)**](https://doi.org/10.1007/s11067-016-9323-0) (for v1.0) and **[Gehlot et al. (2019)](https://doi.org/10.1061/(ASCE)CP.1943-5487.0000802)** (for v2.0). It is built on [Repast Simphony](https://repast.github.io/repast_simphony.html) 2.6.

![res/image.png](res/image.png)

Screenshot of A-RESCUE 3.0 running a preloaded simulation scenario.

# Getting started

## Quickstart

An online demo has been hosted at [https://engineering.purdue.edu/HSEES/EvacVis/](https://engineering.purdue.edu/HSEES/EvacVis/). This demo uses pre-stored vehicle trajectory files simulated using a base inputs scenario and stored on a server at Purdue University owned by Dr. Ukkusuri. This demo shows the user interface and controls that are described in the [Visualization Interface]() section.

## Building and running the simulator

1. Download and install **Eclipse** IDE for Java Developers from [here](http://www.eclipse.org/downloads/packages/release/2021-03/r/eclipse-ide-java-developers).
2. Install **Repast Simphony** plugin in Eclipse using the instructions provided [here](https://repast.github.io/download.html#update-site-install).
3. Clone this A-RESCUE repository to a suitable location: `git clone [https://rverma95@bitbucket.org/purduesimulation/evacsim.git](https://rverma95@bitbucket.org/purduesimulation/evacsim.git) <target directory>`.
4. Load the `EvacSim` project in Eclipse:
    1. Go to `File → Open Projects from File System`
    2. Click on the `Directory` button and select the target directory where you cloned the repository.
    3. Note: Uncheck the `EvacSim/Evacsim` project.

    ![res/Untitled.png](res/Untitled.png)

5. Modify the inputs according to your need in the configuration file as described in the ['Input Data' section]() below.

    ![res/Untitled%201.png](res/Untitled%201.png)

6. Setup the run configuration (`Run → Run Configurations`) with the following details:

    ![res/Untitled%202.png](res/Untitled%202.png)

    1. Main tab: Project: `EvacSim`, with main class: `repast.simphony.runtime.RepastMain`
    2. Arguments tab: VM arguments: `Xss256M -Xms1024M -Xmx25000M`. You may change these memory size settings based on your requirement and the server's memory. For more information about these variables, see [this](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html).
7. Click the Run button (or go to `Run → Run`) to open the Repast Simphony simulation window.

    ![res/Untitled%203.png](res/Untitled%203.png)

8. Run the simulation (`Run → Init`). You should see the *Console* window in Eclipse printing the program runtime log. After a few seconds, you should see an image of a road network showing up in the Repast Simphony simulation window but without any vehicle moving on them. Unless there is an error in the *Console* window, this means that the simulation is running properly. Depending on the input demand size, the simulation may take a long time to finish. When it finishes without error, the console will terminate logging.

    ![res/Untitled%204.png](res/Untitled%204.png)

9. Look in the output directory specified in the configuration file (see ['Input Data' section]()). Unless JSON or CSV data collection was disabled, you should see several JSON or CSV files there.

    ![res/Untitled%205.png](res/Untitled%205.png)

## Using the visualization interface (VI)

This is the main module for the users of this software. Steps for using the VI are:

1. Clone the EvacVis repository to a suitable location: `gh repo clone tjleizeng/HSEES_Viz <target directory>` .
2. Follow the instructions in `~/EvacVis/README.md` ([link](https://github.com/tjleizeng/HSEES_Viz/blob/master/EvacVis/README.md)). You will need `Node.js` and `NPM` for running a local server.

## Installing the online task manager (OTM)

End user clients like the visualization interface connect to the online task manager, a gateway for running the simulation remotely.  This section summaries the steps needed to install a copy of the OTM and configure it.  It is composed of two major components:

- the gateway control program, and
- a copy of the simulation prepared to run in "batch mode" without the GUI.

There are three major steps to the setup process.  First, the server will need to have the prerequisites installed.  Next, the code for the gateway control program is retrieved and configured with the server details.  Finally, the simulation copy is added and configured.

More detailed instructions can be found within the gateway control program's code repository at [https://bitbucket.org/purduesimulation/control-program/src/master/README.md](https://bitbucket.org/purduesimulation/control-program/src/master/README.md).

### Installing the OTM prerequisites

- Running the simulation requires a working copy of the Java Runtime Environment (JRE). The simulation was developed with version 8 of the Java language. It may or may not work well with newer versions of the language. If the server where the OTM will execute does not have Java or has a newer version of the Java language which has compatibility issues, a copy of OpenJDK can be installed following the instructions found at: [https://openjdk.java.net/install/index.html](https://openjdk.java.net/install/index.html)
- The OTM control program is written in the Javascript language using the Node.JS framework and uses a few support scripts written in the bash shell scripting language. Installing the Node.JS runtime engine can be done by following the instructions at: [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

### Getting and configuring the control program

- The code for the control program and the base files needed to create a template of the simulation can be retrieved by cloning the repository with the command:

    ```bash
    git clone [https://bitbucket.org/purduesimulation/control-program.git](https://bitbucket.org/purduesimulation/control-program.git)
    ```

- In the cloned repository, the `code` directory holds the Node.js code needed for the control program. It is configured by modifying the values within the `config.js` file. Most values should be left unaltered, but appropriate file paths and URLs specific to the server where this is being installed need to be set for:

    ```jsx
    global.SCRATCH_ROOT
    global.WEB_ROOT_DIR
    global.WEB_GW_DIR
    global.URL_DOMAIN
    global.URL_ROOT
    ```

### Creating and configuring a template copy of the simulation

- Within the Repast GUI of the simulation, there is an option to package the model and all of its dependencies into one JAR file that can be copied to a server and executed without the GUI.
    - A copy of this is provided in the git repository and can be found within the `tmpl_files` directory named `complete_model.jar`.
    - If a newer version is available, place it here before creating a simulation template. At any point in the future when creating updated templates, delete the existing `complete_model.jar` or `complete_model.zip` file found here in the `tmpl_files` directory and replace it with the newer copy.
    - If a custom version is required, build and test the simulation code as normal within Eclipse and then use the instructions for exporting a model JAR to create the custom `complete_model.jar` file: <link to instructions>
- Within the `tmpl_files` folder is a helper script which will unpack the model JAR and add additional files needed for the gateway program to control the simulation. Change into the `tmpl_files` directory and run the command: `./make_template.sh`
- This will produce a `template` directory in the same location as the `code` and `tmpl_files` directories. This `template` directory will be copied and used as the base each time a user executes the simulation. The extra bash scripts added to this directory should function without alterations, but they may need small changes to account for unique conditions of the server.
- The `Data.properties` file found at `template/data/Data.properties` is the configuration file for the simulation and is where several values are set to optimize the simulation for the server. Some of these values will be overwritten at runtime by the user's input selections, but some of them relate to specific details about the server which the user will not know or have to change.  Be sure to set these values now:
    - `N_THREADS` to set the number of concurrent threads the simulation will use.
    - `N_PARTITIONS` to set the number of network partitions the simulation will use.
    - `ENABLE_NETWORK` should be set to `true` so the model will attempt to contact the control program after starting.  The control program can still start and provide basic monitoring of a simulation that does not enable network communication, but there will be no ability to send it real-time events and reduced capacity to receive status updates from it as the control program will have to passively scan output files.
    - `GATEWAY_ADDRESS` should be set to `127.0.0.1` and the `GATEWAY_PORT` should be set to match the same value in the `config.js` file from the previous control program configuration step. By default it should be `47907`, but it can be changed to any number desired between `1001` and `65535` so long as both of these configuration files match and the particular port number is not already in use by another server.
    - Any other values needed in the case of customized model versions.

At this point the online task manager should have a working copy of the simulation and be configured to execute it on behalf of the users. Within the `code` directory is a `Makefile` and the gateway can be started from within the `code` directory with the command: `make run` .  It can be accessed by connecting a client like the VI web-site to the address of the server (domain or IP) on the port chosen above.

If desired, documentation for the specific distribution and version of Linux which the server is running can be consulted for how to set it as a daemon or service to run in the background unattended.

---

# How it works

## Framework

A-RESCUE is based on Repast Simphony's simulation framework where agent and environment objects are stored in "contexts" that then interact with each other. The key components and their interaction as currently implemented in A-RESCUE are shown in the following figure and explained in the subsequent sections. 

![res/Untitled%206.png](res/Untitled%206.png)

Key components of the simulation framework using A-RESCUE.

## Object contexts and geometry layers

Repast Simphony uses a context manager class factory for working with collections of objects and providing other utilities. The entry point of the simulation setup is the `evacSim.ContextCreator` class. The following high-level tasks are contained in this class:

- Setting up event handlers, including for the simulation run and end and recurring events;
- Setting up the scenario name, output directory, and the logger;
- Creating the contextual objects as well as the data collection context;

The simulator makes use of the following principal objects and their contexts.

- **City**: The `citycontext.CityContext` object serves as a container of other subcontexts — roads, junctions, lanes, and zones. It also has helper functions to deal with resolving road network components, such as finding a road at given coordinates and between junctions and nearby shelters.
- **Road and lane**: The road network is encapsulated in `citycontext.RoadContext` containing `citycontext.Road` objects. Roads are directed (source → target) and may contain one or more `citycontext.Lane`s. Roads serve as the main longitudinal network links while lanes are just used for lateral movement once a vehicle's road is resolved.

    Note: Roads, lanes, and shelters each require as inputs both a shapefile as well as a copy of its attributes in a CSV file.

- **Zone**: This represents the geographical unit of analysis, hereby referred to as the traffic analysis zone (TAZ). It is typically taken as a county, zip code area, a smaller unit, or a census-level region. The zone shapefile must contain the coordinates of the centroid of each analysis zone as the table row. TAZ IDs are natural numbers starting arbitrarily from 1. For Jacksonville, we used census block groups defined as per the 2017 American Community Survey as the zone areas.
- **Shelter**: This includes public shelters during evacuation such as schools, hospitals, nursing homes, malls, hangars, etc. These are identified as point objects. For the Jacksonville area, we filtered the public shelters listed in the [2018 Florida Statewide Emergency Shelter Plan](https://www.floridadisaster.org/globalassets/dem/response/sesp/2018/2018-sesp-entire-document.pdf) that had a nonzero capacity in terms of the registered number of vacancies.

    In the simulator, shelters are coded as `citycontext.Zone` objects with the `type` code '1' (whereas normal TAZs are coded with type '0') and with two special attributes — `capacity` (no. of people it can accommodate when empty) and `occupancy` (the no. of people it has currently accommodated). These two shelter-specific attributes are required in routing shelter-bound vehicles. The zone IDs of shelter objects are negative integers arbitrarily starting from -1.

- **Household**: The evacuation demand is created at the household level. One household can be associated with more than one evacuating trip, each of which is represented by one vehicle. For ease of analysis, the demand associated with each household is assumed to emanate from the centroid of its encompassing TAZ (zone).
- **Vehicle**: This is the main agent in the simulator. The `vehiclecontext.Vehicle` class includes most of the relevant methods related to the traffic simulator's core operational principles such as movement and car-following model and shelter routing management.

## Input data

All the required inputs are listed and described in the **configuration file** `evacSim/data/Data.properties`. The paths of the input files are to be provided in the top section of the file. All uppercase constants in this document can be found in this configuration file.

### Evacuation demand

This is the primary input to the simulator that contains the origin-destination and departure time of each vehicle, along with the specification of the destination as either a regular demand zone or a shelter.

The evacuation demand has a very specific file format that the user must provide for the simulator to run properly. For a given scenario, a CSV file needs to be specified for the variable `ACTIVITY_SEQ_CSV` in the configuration file. It has to have two consecutive rows for each trip in the following format:

           UID   Location       Duration
100000001            23               602
100000001            19                 -1
100000002             ...                   ...
100000002             -4                  0

Here, `UID` represents the unique vehicle ID, which must be identical in the pair of rows corresponding to one trip. `Location` on the first and second lines represent the zone ID of the origin and destination zones, respectively. Positive integers represent evacuation demand zones whereas negative integers correspond to shelters. `Duration` on the first line represents the departure time of this vehicle in minutes since the beginning of the simulation period. On the second line, this value is -1 for demand zones and 0 for shelters.

### Background speed information

Other than the evacuating vehicles, the simulator also needs to know the free-flow speed of each road in the network which serves as the initial speed values of the roads when there is no vehicle on them. When one/many vehicles arrive(s) on a road, its speed is recalculated at each simulation tick based on those vehicles' space mean speed, so that there is no longer the need for the background speed. This information is to be provided in the following tabular format as a CSV file next to the variable `BT_EVENT_FILE`:

   linkID        FREEFL01      FREEFL02       ...
100000                   50             52.05       ...
100001             26.945          28.885       ...

Here, each row specifies a road identified with its `linkID` and has its mean hourly free-flow speed in miles per hour for each hour since the beginning of the simulation period. In the case of Jacksonville, this information was obtained from [INRIX Inc.](https://inrix.com/) based on the speed profile of Jacksonville's roads using probe speed data during October 2016.

Note: In the current version of the program, the end-user also has to make sure that the number of hours (i.e., no. of columns - 1) in the `BT_EVENT_FILE` must also be specified next to the variable `HOUR_OF_SPEED`. We will remove this requirement in the next version.

### Supply-side events

The program also allows for explicitly adding external events related to changing the supply-side factors of the evacuation traffic system such as road/lane closure, lane reversal, and shelter closure. This additional feature allows analysts to look at how traffic controllers can manipulate the traffic in real-time by making external adjustments to the infrastructure network. Currently, the simulator supports only road closure events, though we are planning to accommodate other events as well. In the current setting, these events operate by overriding the speeds of roads and lanes from the usual car following model output. This in turn allows the user to control movement on a set of roads and lanes.

The list of supply-side events, if any, are to be provided in a CSV file listed next to the variable `EVENT_FILE` and should be in the following format:

startTime       endTime       eventID       roadID       value1       value2
             0           82800                  1      101210               5          -999
            ...                   ...                 ...                ...               ...              ...

Here, for each event (row), `startTime` and `endTime` respectively represent its starting and ending time in simulation ticks (1 tick = 0.3 second by default). `eventID` is used to distinguish different types of events. In the current implementation, only '1' is used. `roadID` represents the affected road, though this is meant to be more generalizable in the future as the context object ID, which can accommodate lanes and shelters as well. `value1` and `value2` are placeholder variables for the different events. For road closure, `value1` represents the reduced speed of the given road in miles per hour.

## Traffic simulation

### Overview of the simulation process

- At the beginning of the simulation, the demand file is read in the `demand.DatasetOfHouseholdsPerZones` class and the demand data stored as a TreeMap listing the household (and thus vehicle) IDs for each zone ID and each departure hour.
- During the first run, the simulator generates and stores a mapping of the closest downstream road junction corresponding to each zone centroid using Euclidean distance in `citycontext.CityContext.createNearestRoadCache()` . Thus, vehicles effectively start from intersections instead of zone centroids.
- Next, each road maintains a TreeMap of new vehicles planned to enter it at different times. The `citycontext.Road.step()` function calls those vehicles to trigger the `vehiclecontext.Vehicle`'s `travel()` and `move()` methods. The `move()` function calls methods that use the car-following model to compute acceleration and distance to be traveled. Then, it moves the vehicle that distance along the control points of the road's geometry (since a road is a line feature with multiple linear segments) and updates the state variables.
- In the current version, if the simulator encounters a gridlock due to minimum required gaps and distances, it reroutes the stuck vehicles after a certain amount of time, facilitated by the `MAX_STUCK_TIME` and `MAX_STUCK_TIME2` constants. First, if a vehicle is stationary in front of an intersection for `MAX_STUCK_TIME`, it triggers the routing function to assign a new route for this vehicle. For some cases (e.g., one-link deadlock by two U-turning vehicles) in which this mechanism fails and the stuck time can exceed `MAX_STUCK_TIME2`, it forces the stuck vehicle to enter the next (if there is any) available link and call the rerouting function.

### Car following model (CFM)

- The simulator uses a 3-regime car following model (CFM) which forms the core of the traffic movement — free flow, normal, and emergency. Most of the CFM is implemented in the `vehiclecontext.Vehicle` class. These regimes are governed by the limits on the vehicle headway (`H_LOWER`and`H_UPPER`), the longitudinal time difference between the nose of the current vehicle and the tail of the first vehicle in front of it.
- Free-flowing regime (headway>`H_UPPER`): the vehicle will accelerate/decelerate to the target speed ([Yang et al., 1997](https://journals.sagepub.com/doi/abs/10.3141/1710-14?casa_token=FK2i9SVVwGMAAAAA:GZCar8ugwsdAHYChuWkFHs5IkCOleJHyEX47jI2Ay54YnR4PwrmuRbue3NylWeBPChY38OtoTzRR)). The acceleration rate is determined by a constant called `maxAcceleration_`, while the deceleration rate is calculated based on the headway space.
- Car-following regime (headway within [`H_LOWER`, `H_UPPER`]): in this case, the acceleration rate is calculated using the Herman model ([Herman et al., 1959](https://pubsonline.informs.org/doi/abs/10.1287/opre.7.1.86)).
- Emergency decelerating regime (headway<`H_LOWER`): the vehicle will do the emergency deceleration. The deceleration rate is calculated according to ([Yang et al., 1997](https://journals.sagepub.com/doi/abs/10.3141/1710-14?casa_token=FK2i9SVVwGMAAAAA:GZCar8ugwsdAHYChuWkFHs5IkCOleJHyEX47jI2Ay54YnR4PwrmuRbue3NylWeBPChY38OtoTzRR)).

### Adaptive routing

The simulator provides two types of routing methods for evacuating vehicles on the road network. These are implemented in `routing.VehicleRouting` and `routing.RouteV` classes.

1. **Selfish adaptive routing**: This type of routing is called selfish because each individual looks for their best, often shortest, route which may change depending on the current location and traffic conditions, notwithstanding the effect of their choice on other people's choices. In this case, the simulator computes the shortest path for each evacuating vehicle using its current location and destination whenever it changes the road (close to intersections) and before entering the network. Other than this, the simulator also updates the routes of vehicles at a fixed interval known as the network refresh interval when the road network state is refreshed for repartitioning.
2. **Familiar routing**: As part of the larger adaptive routing mechanism, the simulator also provides an option to realistically simulate the routing behavior of evacuees. Studies have shown that travelers do not always use optimal selfish routing but often choose routes familiar to them or deliberately not the shortest, such as for scenic beauty or leisure ([Peeta et al., 2001](https://link.springer.com/article/10.1023/A:1012827724856)). Furthermore, people who do not have access to advanced traveler information systems (ATIS) tools, such as Google Maps, Waze, or local radio, may not know the dynamically updated shortest path. These people may also rely on routes familiar to them, especially during a time like disaster evacuation.

    To accommodate such behavior, the simulator uses a "sticky" routing mechanism where some vehicles/drivers are randomly assigned an "indifference band", $\eta$, based on the work of [Mahmassani and Jayakrishnan (1991)](https://www.sciencedirect.com/science/article/abs/pii/019126079190145G). At each point of rerouting, the driver first selects the top $k$ shortest remaining paths to the destination. Then, the driver randomly selects one of those routes if that route provides a sufficient benefit over the current route in terms of travel time as shown in the following formula:

    $\delta=\begin{cases}
    1 & t_{cur}-t_{best} > \max(\eta \; t_{cur}, \tau) \\
    0 & \text{otherwise}
    \end{cases}$

    Here, the decision $\delta$ to switch from the current path to the best (shortest) path at an intersection depends on the savings of the remaining trip time, $t_{cur}-t_{best}$, and the stickiness/indifference $\eta$. $\tau$ is an absolute minimum travel time improvement below which a driver will not switch routes. This type of routing allows people to stick to their current route unless a much better alternative is available, depending on what qualifies for each individual as "much better".

    We assigned vehicles/drivers this indifference band, $\eta$, by drawing from an isosceles triangular distribution as suggested in [Mahmassani and Jayakrishnan (1991)](https://www.sciencedirect.com/science/article/abs/pii/019126079190145G) with the parameters $\eta=0.2$ and $\tau=1$ minute (`ETA` and `TAU` in the configuration file).

### Shelter routing

Most evacuees prefer to evacuate to hotels and houses of friends and family. From an evacuation management perspective, public shelters play an important role in sheltering the most vulnerable evacuees, especially those who evacuate much later and close to hurricane landfall. Since public shelters are often abandoned schools, hospitals, nursing homes, etc. which authorities can regulate, an important consideration for shelter design and allocation is their accommodation capacity.

Through this simulator, we explore a relatively unexplored problem — what happens when an evacuee (vehicle/driver) arrives at an evacuation shelter but is rejected on the grounds of no remaining capacity? We assume that that evacuee will seek refuge in another shelter. With this assumption, we consider three types of "shelter routing":

1. **Blind search**: The evacuee on rejection goes to the nearest other shelter without knowing if that one has vacancies or not. This is characteristic of individuals/families that do not have information about the current status of shelters.
2. **Cognizant search**: Here, the rejected evacuee goes to the closest shelter with nonzero vacancies. This is a more realistic scenario. It also drastically reduces the travel time of the travelers as seen in the simulation test results.
3. **Shelter matching**: Finally, we also consider a hypothetical central service, like a mobile application, which can assign available shelters to all the rejected shelter seekers of the city at the same time. This is a kind of system optimal (SO) routing where the objective is to find a matching between the demand (shelter seekers) and the supply (available shelters) to minimize the total impedance (time spent in traveling between shelters). This is implemented in the `routing.SOShelterRouting` class in the form of 3 matching algorithms: (i) an exact [Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm), (ii) LEDA maximum weight bipartite matching (exact), and (iii) greedy weighted matching (1/2-approximate). The simulator uses [jgrapht](https://jgrapht.org/) library's [matching](https://jgrapht.org/javadoc/org.jgrapht.core/org/jgrapht/alg/matching/package-summary.html) package for the implementation.

## Performance

Since the amount of computation of simulation can significantly increase when the number of vehicles is large, a parallelization strategy is introduced to enhance the computational performance of the simulator.

### Parallelization

The parallelization strategy works as follows:

- A Java Thread pool is implemented which is a fixed group of threads that constantly waits for computation jobs to execute.
- The idea is to have the threads always existing to reduce computation overhead
due to thread creation and destruction. At every simulation tick, each thread is assigned to a particular network partition and updates vehicle movements on all roads in the network partition.

Properly balancing the computational load across subnetworks is essential to achieve the best parallelization performance and thus a network partitioning strategy is developed as follows.

### Network partitioning

The simulator uses a dynamic load-balancing strategy that periodically repartitions the road network into subnetworks with approximately equal computational load using a predictive network weight scheme.

- The simulator uses a partitioning algorithm called **GMetis** for partitioning the road network into subnetworks based on a computational load graph. GMetis is the Java implementation of the Metis algorithm.
- Metis partitions the network using a three-stage process ([Karypis and Kumar, 1995](https://www.researchgate.net/publication/246815679_METIS_--_Unstructured_Graph_Partitioning_and_Sparse_Matrix_Ordering_System_Version_20)):
    - The network first collapses a local cluster of connected nodes into a contracted node, hence coarsening the origin network to a much smaller graph.
    - Next, a series of bisections of the coarsened graph is performed to obtain the desired number of partitions based on the weights of contracted nodes.
    - Finally, partitions are projected back toward the original graph (finer graph), and gradually refining each partition toward the partitioning objective (equal node weights and minimum boundary edge weights).
- On completion of the GMetis algorithm, the road network is divided into various partitions where each road belongs to one of the categories: in-partition road or boundary road.
    - In-partition roads are roads whose both upstream and downstream nodes belong to the same partition of the network and boundary roads are roads whose upstream and downstream nodes belong to different partitions.
    - The partition assignment for boundary roads is sequentially performed by merging each road to the neighboring partition with a lower total edge weight, which ensures the computational load of each partition remains approximately equal.
- Repartitioning of the network is scheduled every fixed interval of time, but the actual repartitioning is executed only when the number of vehicles in the network is above a threshold.
- To partition the traffic network into subnetworks/partitions with approximately equal computational load, the computational load in the traffic network within the future time period (until the next repartitioning period) needs to be predicted.
    - The computation load graph is a weighted graph, with weights on nodes (intersections) and edges (roads) approximating the computational load.
    - The routing computation is more costly compared to the car-following and lane-changing updates. Thus, computational loads on roads (represented as edge weights) are estimated as the linear combination of three components: the number of current vehicles on the roads (denoted as $N_c$), predicted number of vehicles that would be traveling on the road until the next repartitioning period (given by $N_t$), and the predicted number of vehicles that would perform routing on the road until the next repartitioning period (denoted as $N_r$).
    - The computation of $N_c$ is trivial and thus we focus on $N_t$ and $N_r$. For each vehicle, up to a given threshold downstream reachable roads on the vehicle’s current route are tracked. Reachability is examined by computing the cumulative travel time starting from the current road. If the cumulative travel time on a downstream road segment is smaller than the length of the repartitioning period, the count $N_t$ will increment by 1 on that road segment.
    - Up to a given number of downstream reachable roads are tracked using cumulative travel time starting from the current road. However, only those roads for which cumulative travel time exceeds an integral multiple of network refresh period are considered, and
    then the road’s $N_r$ is incremented by 1. This is because routing is performed every time the network is refreshed. The maximum possible future routing roads of a vehicle is less than or equal to equal to the ratio of repartitioning time period to the network refresh period. Once this procedure is performed for every vehicle, the $N_r$ value for each road in the network is obtained.
    - Note that the weights corresponding to $N_c$, $N_t$, and $N_r$ are determined by several tests where one parameter is varied while keeping others to be fixed, and the combination of the weights that maximizes the computational performance is chosen. Finally, the weight of each node (intersection) is determined as half of the sum of all neighboring edge (road) weights.

## Collected output

The simulator periodically stores "snapshots" of the current state of the simulation run as output streams (i.e., continuously, not at the end of the run). This data collection is governed by the settings in the *Data collection* and *Output data storage* sections of the configuration file. Data collection can be toggled with the inputs: `ENABLE_JSON_WRITE` and `ENABLE_CSV_WRITE` and the output directory is specified by `DEFAULT_OUTPUT_DIR`.

The JSON files are mainly meant to be sent for simulation visualization to the client over the Internet. Since these files can become large when the evacuation demand is large, each output JSON file contains a small number of snapshots, given by `JSON_TICK_LIMIT_PER_FILE` (by default, 2). Each JSON object within each file stores the following information for each snapshot period:

- `vehicles`: This is usually the most space-consuming part of the data as it contains the vehicle trajectory data. At each fixed time interval, given by `FREQ_RECORD_VEH_SNAPSHOT_FORVIZ`, the simulator stores the snapshot information of each vehicle currently on the road network. This is stored as a comma-separated string that contains the following information:
    - Vehicle ID: Unique, automatically generated identifier for each vehicle object.
    - Current coordinates (longitude, latitude) of the vehicle.
    - Current speed in miles per hour.
    - Bearing of the vehicle showing its orientation. The value represents the angle made by the movement direction of the vehicle from the North in degrees, measured clockwise.
- `newVehs`: List of new vehicles that appeared on the road network within the given snapshot period, i.e., in the range *(lastPeriodTick, currentTick]*. This is mainly helpful in generating the progress curve in the visualization interface. It includes three attributes of each vehicle: the vehicle ID, and zone IDs of its origin and destination.
- `arrVehs`: Same as `newVehs` but for vehicles that arrived at their destination and thus got eliminated from the road network within the given snapshot period.
- `roads`: In addition to vehicle snapshots, the simulator also periodically stores the status of roads. This interval, given by `FREQ_RECORD_ROAD_SNAPSHOT_FORVIZ`, is different than that of vehicles but must be a multiple of `FREQ_RECORD_VEH_SNAPSHOT_FORVIZ`. If a given tick is equal to this interval, three attributes of only those roads whose state changed within the period *(lastRoadSnapshotTick, currentRoadSnapshotTick]* are stored: road ID, the updated number of vehicles on that road, and its updated space mean speed in miles per hour.
- `shelters`: The simulator also stores the availability of evacuation shelters at the interval given by `FREQ_RECORD_SHELT_SNAPSHOT_FORVIZ`. The attributes stored for each updated shelter are: shelter's zone ID and its updated occupancy (number of evacuees it is currently housing).

An example output snapshot file reads as shown below:

```json
{
  "18200": { // tick at which the snapshot is taken (1 tick = 0.3 second)
    "vehicles": [ // vehicles currently on the road network
      "229969,-81.82101,30.55136,13.83,147.60", // ID, lon, lat, speed, bearing
      "217283,-81.82292,30.55402,13.83,148.28",
      ...],
    "newVehs": [ // new vehicles that have been generated within this snapshot period
      "201564,52,168", // vehicle ID, origin ID, destination ID
      ...],
    "arrVehs": [ // new vehicles that have arrived within this snapshot period
      "194532,146,-25", // vehicle ID, origin ID, destination ID
      ...],
    "roads": [ // roads whose state (speed) has been updated during this road snapshot period
      "100482,0,17.135", // road ID, number of vehicles on it, space mean speed
      ...],
    "shelters": [ // shelters whose state (occupancy) has been updated during this shelter snapshot period
      "-7,53", // shelter zone ID, current occupancy
      ...]
  }, ...
}
```

The CSV output is optional and mainly used for data post-processing as it can be more compact and easily manipulated than JSON. It only contains the vehicle snapshots in a tabular format with four columns for each vehicle: its ID, current longitude and latitude, and current speed (mph).

## Visualization interface (VI)

A web-based VI is developed to display simulation information. The VI is written using [React](https://reactjs.org/) with layers powered by [Deck.gl](https://deck.gl/). An online demo is available at [https://engineering.purdue.edu/HSEES/EvacVis/](https://engineering.purdue.edu/HSEES/EvacVis/).

![res/Untitled%207.png](res/Untitled%207.png)

The VI consists of five components (as shown in the above screenshot):

- **A** is the evacuation map with road networks, shelters and vehicles displayed.
- **B** is the client panel that controls the connection between the user interface and the remote server, in offline mode, the user just needs to specify the location of the simulation outputs.
- **C** is the control panel that enables changing the playback speed and the display layers, one can choose.
- **D** is the information panel that shows the number of on-road vehicles, speed, shelter status, and information of the selected vehicle.
- **E** is the chart panel for displaying the dynamics of the departures and arrivals of the evacuation vehicles.

Online mode: When the VI successfully connects with the online task manager, it enters the online mode. The user can specify the parameters of running a new simulation instance, like the demand scenarios, the routing strategy (shortest path, k-shortest path), and the simulation duration. Similar to the offline mode, the simulation outputs would be visualization through the user interface. Besides this, during the simulation the user can also specify online event inputs (e.g., road closure on specific roads), set up the event start and end time, and monitor the traffic state changes induced by these events.

### Network gateway (@Christopher Thompson)

### Network communication (@Christopher Thompson)

---

# Notes

## Version history

### New in V3.0

- Improvements in performance and scalability
    - Removed the computationally expensive Galois coordinate mechanism that uses unnecessary location methods (e.g., quadtree) and replaced it with a simple 2D coordinate system.
    - Added checks to remove or reduce gridlocks at intersections during complicated maneuvers. This was done by setting thresholds on the time it practically takes to clear intersections in case of mild traffic jams.
- User interface and functionality improvements in the VI module
    - Revamped the user interface with a Material UI theme.
    - Added charts and information panels for status updates.
    - Added dark theme.
- Integration with VI module
    - Modified the output data structure to optimize data transfer speed to the visualization interface (VI), its performance in the online mode, and reduce data local storage (for example, by removing redundant decimal places after vehicle snapshot information).
- Shelter routing strategies
    - Added three routing strategies for shelter-bound evacuees who are rejected by their first shelter. This includes a blind search, a relatively wise selfish search, and a central matching recommendation. See [Shelter routing]() section for details.
- K-shortest paths and stickiness to initial route
    - Extended shortest path-based vehicle routing to a probabilistic k-shortest path-based routing.

### New in V2.0

- Parallelization, network partitioning, and load balancing
    - Implemented Metis partitioning to partition the road network into parallelizable subgraphs based on the latest state of the vehicles during the simulation run.
    - Used efficient techniques to balance the expected computational load among the network partitions. See [Gehlot et al. (2019)]() and the [Performance]() section for more details.

## References

1. Ukkusuri, S. V., Hasan, S., Luong, B., Doan, K., Zhan, X., Murray-Tuite, P., & Yin, W. (2017). A-RESCUE: An agent-based regional evacuation simulator coupled with user enriched behavior. *Networks and Spatial Economics*, 17(1), 197-223. DOI: [10.1007/s11067-016-9323-0](https://doi.org/10.1007/s11067-016-9323-0).
2. Gehlot, H., Zhan, X., Qian, X., Thompson, C., Kulkarni, M., & Ukkusuri, S. V. (2019). A-RESCUE 2.0: A high-fidelity, parallel, agent-based evacuation simulator. *Journal of Computing in Civil Engineering*, 33(2), 04018059. DOI: [10.1061/(ASCE)CP.1943-5487.0000802](https://doi.org/10.1061/(ASCE)CP.1943-5487.0000802).