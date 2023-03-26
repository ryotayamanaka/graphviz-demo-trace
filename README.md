# Oracle Graph Visualization Demo Application 

An example Java web application based on [Micronaut](https://docs.micronaut.io/) which embeds Oracle's Graph
Visualization library. The server queries the graph data from an Oracle Database using 
[PGQL](https://pgql-lang.org/).

The key source files to look at are

* `src/main/resources/public/index.html`: the HTML file served in the browser embedding the visualization library
* `src/main/java/com/oracle/example/TraceController.java`: implements the REST endpoints called by `index.html` 
* `src/main/java/com/oracle/example/GraphClient.java`: wraps the graph server APIs, called by TraceController

## Pre-requisites

1. Oracle JDK 11 (or OpenJDK 11)
2. A running Oracle Graph Server. Download [from oracle.com](https://www.oracle.com/database/technologies/spatialandgraph/property-graph-features/graph-server-and-client/graph-server-and-client-downloads.html) and install [as per documentation](https://docs.oracle.com/en/database/oracle/property-graph/23.1/spgdg/using-rpm-installation.html#GUID-EF1C77D2-86B5-4F16-AA43-3B37BE5FE4B9).
3. A running Oracle Database (e.g. [Autonomous Database](https://www.oracle.com/autonomous-database/))
4. This example uses the [Human Resources sample dataset](https://github.com/oracle-samples/db-sample-schemas). Load the CSV files into your database using client tools (e.g. Database Action or SQL Developer).
5. Create a graph on the Graph Server out of the dataset using the following statement:

```
CREATE PROPERTY GRAPH trace_all
  VERTEX TABLES (
    trace_bom_node
      KEY (id)
      LABEL part
      PROPERTIES (id, part_id, lot)
  , trace_scn_node
      KEY (id)
      LABEL place
      PROPERTIES (id, place_id, lot)
  )
  EDGE TABLES (
    trace_bom_edge
      KEY (id)
      SOURCE KEY(child_id) REFERENCES trace_bom_node
      DESTINATION KEY(parent_id) REFERENCES trace_bom_node
      LABEL part_of
      NO PROPERTIES
  , trace_scn_edge
      KEY (id)
      SOURCE KEY(src_id) REFERENCES trace_scn_node
      DESTINATION KEY(dst_id) REFERENCES trace_scn_node
      LABEL supplied_to
      NO PROPERTIES
  , trace_b2s_edge
      KEY (id)
      SOURCE KEY(part_id) REFERENCES trace_bom_node
      DESTINATION KEY(place_id) REFERENCES trace_scn_node
      LABEL produced_at
      NO PROPERTIES
  )
```

## Usage

1. Clone this repository 
2. Download the "Oracle Graph Visualization library" [from oracle.com](https://www.oracle.com/database/technologies/spatialandgraph/property-graph-features/graph-server-and-client/graph-server-and-client-downloads.html)
3. Unzip the library into the `src/main/resources/public` directory. For example:

```
unzip oracle-graph-visualization-library-23.1.0.zip -d src/main/resources/public/
```

4. Run the following command to start the example app locally:

```
./gradlew run --args='-oracle.graph-server.url=<graph-server-url> -oracle.graph-server.jdbc-url=<jdbc-url> -oracle.graph-server.username=<username> -oracle.graph-server.password=<password>'
```

with

* `<graph-server-url>` being the URL of the Graph Server, e.g. `https://myhost:7007`
* `<jdbc-url>` being the JDBC URL of the Oracle Database the Graph Server should connect to, e.g. `jdbc:oracle:thin:@myhost:1521/orcl` 
* `<username>` being the Oracle Database username to authenticate the example application with the Graph Server, e.g. `scott`
* `<password>` being the Oracle Database password to authenticate the example application with the Graph Server, e.g. `tiger`

Then open your browser at `http://localhost:8080`.

When you click on the <em>Trace</em> button, a request is made to `/trace/by_str`, which fetches the reachable parts and inventories for the given part/inventory (by default `P1112_0269322`) from the TRACE_ALL graph using a PGQL query.

![](screenshot.png)

When you right-click on one of the resulting nodes and then select <em>Expand</em>, a request to `/trace/by_ids` is being made, which runs the traceability search from that node via another PGQL query.

## Troubleshooting

If you get any errors, 
* check the log output from the server on the terminal where the Gradle command is running
* use browser debug tools (e.g. Chrome Developer Tools) to inspect request/response and console logs