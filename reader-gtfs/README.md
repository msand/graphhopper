# GraphHopper GTFS

Here is a screenshot of a public transport query. The route with the earliest arrival time is highlighted in green. The three 
non-highlighted alternatives have fewer transfers. They include a walk-only route (no vehicle boardings at all).

![gtfs preview](https://www.graphhopper.com/wp-content/uploads/2017/01/gtfs-preview.png)

# Quick Start

```bash
git clone https://github.com/graphhopper/graphhopper
cd graphhopper
git checkout pt
# download GTFS from Berlin & Brandenburg in Germany (VBB) and the 'surrounding' OpenStreetMap data for the walk network
wget -O gtfs-vbb.zip http://transitfeeds.com/p/verkehrsverbund-berlin-brandenburg/213/latest/download
wget http://download.geofabrik.de/europe/germany/brandenburg-latest.osm.pbf
mvn install -DskipTests
mvn --projects web install -DskipTests assembly:single
# The following process will take roughly 5 minutes on a modern laptop when it is executed for the first time.
# It imports the previously downloaded OSM data of the Brandenburg area as well as the GTFS.
java -Xmx5g -Xms5g -jar web/target/graphhopper-web-*-with-dep.jar datareader.file=brandenburg-latest.osm.pbf gtfs.file=gtfs-vbb.zip jetty.port=8989 jetty.resourcebase=./web/src/main/webapp graph.flag_encoders=pt prepare.ch.weightings=no graph.location=./graph-cache
# view the web UI e.g. via: 
firefox http://localhost:8989
```

# Graph schema

![Graph schema](pt-model.png)

We see three trips of two routes (top to bottom: route 1, route 1, route 2) and three stations (colored groups).
The ENTER_TEN and LEAVE_TEN edges are the entry and exit to the time expanded network proper. They enforce that
you are put on the correct node in the timeline when searching forward and backward, respectively. The STOP_ENTER
and STOP_EXIT nodes are regular, spatial nodes which can be connected to the road network.

The BOARD edge checks if the trip is valid on the requested day: Our graph is "modulo operating day", but
a pure time expanded graph which is fully unrolled is also possible. Then that check could go away. It also
counts the number of boardings.

The TRANSFER edge ensures that the third departure is only reachable from the first arrival but not from the second one.

# Attribution

* GraphHopper GTFS uses [GTFS Feed](https://opendata.rnv-online.de/datensaetze/gtfs-general-transit-feed-specification/resource/rnv-gtfs) of [Rhein-Neckar-Verkehr GmbH (rnv)](http://www.rnv-online.de), licensed under [dl-de/by-2-0](https://www.govdata.de/dl-de/by-2-0). This GTFS feed is used as a test dataset.