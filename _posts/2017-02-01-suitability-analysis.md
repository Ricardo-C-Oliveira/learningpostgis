---
layout: post
title:  "Suitability Analysis"
date:   2017-02-01 15:07:38 -0700
categories: tutorial 
---
*Created by Rafael Moreno, PhD and Ricardo Oliveira*   
*University of Colorado Denver*

### Suitability Analysis of Big Horn Sheep in Rocky Mountain National Park

#### 1.	Exercise presentation:   


The purpose of this exercise is to introduce some of the most basic spatial analytical function in PostGIS using a simple suitability analysis problem. Specifically, we will use the following statements and functions: 


- Create buffers from points, lines or polygons: *ST_Buffer()* [http://postgis.net/docs/ST_Buffer.html](http://postgis.net/docs/ST_Buffer.html )
- Combine a set of geometries in different spatial tables into a single spatial table:  *ST_Union()* [http://postgis.net/docs/ST_Union.html](http://postgis.net/docs/ST_Union.html)
- Find the geometries of geometry A that does not intersect with geometry B: *ST_Difference()* [http://postgis.net/docs/ST_Difference.html](http://postgis.net/docs/ST_Difference.html)
- UNION clause with option ALL as part of a SELECT statement to combine the results of several SELECT statements. [http://www.postgresql.org/docs/9.3/static/sql-select.html#SQL-UNION](http://www.postgresql.org/docs/9.3/static/sql-select.html#SQL-UNION)
- Create a new table: CREATE TABLE table_name AS [http://www.postgresql.org/docs/9.5/static/sql-createtableas.html](http://www.postgresql.org/docs/9.5/static/sql-createtableas.html)
- Query for specific records: SELECT statement [http://www.postgresql.org/docs/current/static/tutorial-select.html](http://www.postgresql.org/docs/current/static/tutorial-select.html)
- Find the areas where the geometries of two different tables intersect: *ST_Intersection()* [http://postgis.net/docs/ST_Intersection.html](http://postgis.net/docs/ST_Intersection.html)
- Return TRUE if the geometries of two different tables intersect, if not, return FALSE: *ST_Intersects()* [http://postgis.net/docs/ST_Intersects.html](http://postgis.net/docs/ST_Intersects.html)
- Update the Spatial Reference System Identifier (SRID) of the geometries in a table: *UpdateGeometrySRID()*  [http://postgis.org/docs/UpdateGeometrySRID.html](http://postgis.org/docs/UpdateGeometrySRID.html)
- Define a new index for the records in a table: CREATE INDEX statement in PostgreSQL [http://www.postgresql.org/docs/9.4/static/sql-createindex.html](http://www.postgresql.org/docs/9.4/static/sql-createindex.html) using index type Generalized Search Tree (GIST)
 [http://www.postgresql.org/docs/9.1/static/textsearch-indexes.html](http://www.postgresql.org/docs/9.1/static/textsearch-indexes.html)

#### 2.	Scenario:

You have been requested to identify areas in the Rocky Mountain National Park (ROMO) that meet the following criteria. These areas are of interest for their potential to broadly be good habitat for Big Horn Sheep.  

- Away from roads, trails, and campsites;
- Within a specific distance from lakes and streams;
- Pine forests. 

#### 3. Procedures flowchart:

The first step is to create a flowchart of the analysis we want to perform. This is helpful to clarify and discuss the data that will be required, the analysis approach, and the analytical functions that will be used. 

![Flowchart]({{ site.baseurl }}/images/flowchart_suitability.png)
*Conceptual flowchart for identifying areas potentially suitable for Big Horn Sheep habitat.*


#### 4. Data:

The data for this tutorial can be found [here](https://s3.us-east-2.amazonaws.com/learningpostgis/ROMO_NP.zip). All analysis will be conducted in the romo schema, to create a schema on your existing 
database run the following query:
```sql
CREATE SCHEMA romo;
```

#### 5. Analysis: 

Now we can proceed to apply the necessary PostGIS spatial analytical functions to identify the desired areas. First we will apply one function at the time for clarity and to practice writing SQL statements. Later, at the end of the exercise, we will combine the individual SQL statement into a single SQL statement that will execute all the necessary functions in one step. The later will also help us to practice the creation of embedded SQL statements. 

#### 5.1 Identifying areas away from humans: 

To create the intermediate result away_humans (see Figure 4.1) we have to first create three buffers from roads (150 meters), trails (100 meters) and campsites (200 meters). This step is achieved using the ST_Buffer ( ) function to create a new table for each buffer: roads_buff; trails_buff; and camps_buff. Then we have to combine the polygons identified in each of these buffers into a single combined buffer area. This is accomplished using UNION clause option ALL as part of SELECT statement to create a new table called near_humans. Finally, we have to identify the areas inside the ROMO that are not inside the near_humans buffer to create a new table called away_humans. This step is achieved by using the ST_Difference( ) function using as input parameters the extent of the ROMO minus the geometries in the near_humans table.  Next we detail each of these steps. 

The buffers from roads, trails and campsites can be created using a composite statement of the  the following SQL statements: CREATE TABLE table_name AS combined with a SELECT statement that uses the ST_Buffer( ) function and the clause AS as follows: 

```sql
CREATE TABLE romo.roads_buff AS 
  SELECT gid, 
         St_buffer(geom, 150) AS geom 
  FROM   romo.roads; 

CREATE TABLE romo.trails_buff AS 
  SELECT gid, 
         St_buffer(geom, 100) AS geom 
  FROM   romo.trails; 

CREATE TABLE romo.camps_buff AS 
  SELECT gid, 
         St_buffer(geom, 200) AS geom 
  FROM   romo.campsites; 
```

We can visualize the resulting buffers using QGIS.

![Road Buffer]({{ site.baseurl }}/images/road_buffer.png)

here is the approach using the UNION clause with option ALL as part of a SQL SELECT statement. In the following composite SQL statement, first the near_human table is created with the CREATE TABLE 
table_name AS statement to store the results of the following SELECT statements, then three buffers are created as part of SELECT statements that invoke the ST_Buffer() function, then the UNION clause with option ALL is used to combined the results from buffer functions. All steps are executed through a single SQL statement.

```sql
CREATE TABLE romo.near_human AS 
  SELECT gid, 
         St_buffer(geom, 150) AS geom 
  FROM   romo.roads 
  UNION ALL 
  SELECT gid, 
         St_buffer(geom, 100) AS geom 
  FROM   romo.trails 
  UNION ALL 
  SELECT gid, 
         St_buffer(geom, 200) AS geom 
  FROM   romo.campsites;
```

We can visualize the resulting combined buffer near_humans in QGIS.

![Near Human Buffer]({{ site.baseurl }}/images/near_human.png)

Let’s take a closer look at the resulting near_human table. By running a simple query

```sql
SELECT * 
FROM   romo.near_human; 
```

We see that this table is composed by 8546 rows. This means that for each campsite point, road segment and stream segment in the original roads, streams and campsites datasets, a buffer polygon was
 created. This is important to know because for our last step we want to obtain the areas inside the ROMO that fall outside of all buffers. For this we will use the ST_Difference( ) function as explained later. By using the near_human as it is currently, we will end-up obtaining the difference of each of these individual 8546 buffers against the total extent of the ROMO, resulting in 8546 areas, each corresponding to the area within the ROMO but outside the buffers in near_human. To avoid this we will use the ST_Union( ) function to merge all the individual buffers in near_human into a single buffer which in the idea intended for the near_human partial result.

```sql
CREATE TABLE romo.all_near_human AS 
  SELECT St_union(geom) 
  FROM   romo.near_human;
```

In all_near_human now we have a single geometry representing the union of the buffers from roads, trails and campsites. The last step is to generate the away_human partial result to find the areas inside the ROMO that are outside the buffer near_human_buff we just created. To achieve this we will use polygon we will use the ST_Difference ( ) function using as input parameters the ROMO park boundary (romo_bnd table) and the near_human_buff table. The ST_Differnce( ) function takes two geometries and return what is not common between them. The SQL statement would be as follows:

```sql
CREATE TABLE romo.away_human AS 
  SELECT St_difference(b.geom, n.geom) 
  FROM   romo.romo_bnd AS b, 
         romo.all_near_human AS n;
```

Just for the purpose of learning more SQL structures and syntax we will combine the previous two SQL statements (i.e. the merging of the buffers using ST_Union( ) and obtaining the areas outside the near_human_buff through ST_Difference( ) ) into a single SQL statement as follows:

```sql
CREATE TABLE romo.away_human AS 
  SELECT St_difference(b.geom, (SELECT St_union(n.geom)AS geom 
                                FROM   romo.near_human AS n)) AS geom 
  FROM   romo.romo_bnd AS b; 
  ```

Here we are introducing some new SQL features: aliases, embedded statements, and referencing columns from a table using the structure tableName.columName. Aliases are used to give “nicknames” to tables so that we can invoke them again without having to write the whole table name again. Here we used the alias “b” applied to the romo_bnd table and the alias “n” for the near_human table. Now we will use these aliases to point to specific columns in each table using the structure “alias.column_name” such as b.geom (i.e. column geom in table b) or n.geom (i.e column geom in table n). Next we have a Select statement embedded in another Select statement. The most inner Select invokes a ST_Union( ) function to merge all the geometries from the “n” table (i.e. near_human table). The next outer Select invokes the ST_Difference( ) function to obtain the difference between the areas in the room_bnd table (a.k.a. b table)  that represent the extent of the ROMO, and the areas in the n table (a.k.a. near_human). Finally, the result of these select and function processes are stored in a table called away_human as specified in through the CREATE TABLE statement. The dissection of this statement illustrates the way composed SQL statements should be read. From the most inner statement moving outward to the most external ones.   

So far for learning purposes we have run our analyses creating several intermediate results each stored in a table. Once we are proficient with SQL statements, we could run the whole analysis to identify the areas away from humans in a single composed SQL statement as follows: 

```sql
CREATE TABLE romo.away_human AS 
  SELECT St_difference(b.geom, (SELECT St_union(n.geom) 
                                FROM   (SELECT gid, 
                                               St_buffer(geom, 150) AS geom 
                                        FROM   romo.roads 
                                        UNION ALL 
                                        SELECT gid, 
                                               St_buffer(geom, 100) AS geom 
                                        FROM   romo.trails 
                                        UNION ALL 
                                        SELECT gid, 
                                               St_buffer(geom, 200) AS geom 
                                        FROM   romo.campsites) AS n)) AS geom 
  FROM   romo.romo_bnd AS b;
```

#### 5.2 Identifying areas near water: 
 
The next step on our analysis is to create 100 m buffers around both lakes and streams, very similar to how we created buffers for roads, trails and campsites.

```sql
create table romo.near_h2o as 
select ST_Union(n.geom) as geom
from(
select gid, ST_Buffer(geom, 100) as geom from romo.lakes
union all
select gid, ST_Buffer(geom, 100) as geom from romo.streams) as n;

```

#### 5.3 Identifying areas covered by pine forests:

The next step is to find all areas in the Rocky Mountain National Park that have a pine forest (pure or mixed with other tree species) land cover. For this we will use a wildcard. Wildcards are used
 to retrieve records in a table that contain at least part of a specific string, in this case: Pine.

```sql
create table pine as select * from romo.vegetation where mapclass like '%Pine%';

```

The where clause is used to limit the results to only those records in the vegetation spatial table that contain the text string Pine.


#### 5.4 Finding the areas the fulfill all the requirements for habitat:

Finally, the last step is to find the intersect of the areas in the partial results away_human, near_h2o and pine tables. For this we will use the ST_Intersection( ) function which returns geometries that overlap between the geometries contained in two spatial tables. 

One important aspect of the ST_Intersect( ) is that it accepts as parameters only two sets geometries at a time, therefore we will need to conduct our last step in two separate statements. First let’s intersect away_human and the near_h2o tables

```sql
create table intersect_human_h2o as select ST_Intersection(a.geom, h.geom) as geom from romo.near_h2o as h, away_human as a;
```

The result would look like this:

![Intersection Human Water]({{ site.baseurl }}/images/int_human_h2o.png)

Finally we will use the UpdateGeometrySRID( ) function to update the spatial reference system identifier (SRID) of our final result final_habitat. In this case we will set it to 26913 which is UTM NAD 83 Zone 13 (http://spatialreference.org/ref/epsg/nad83-utm-zone-13n/).

```sql
select UpdateGeometrySRID('romo.final_habitat', 'geom', 26913);
```

One optional step that is highly recommended is to create spatial indexes on tables that will be used frequently. Index are important as they greatly speed up queries and other processes. For this we use the CREATE INDEX statement.

```sql
Create index geom_idx ON romo.final_habitat USING GIST (geom);
```

To close this tutorial, let’s see how all the analytical steps done above could be combined into a single SQL query as follows (we use the name final_habitat_2 to avoid over writing our previous 
final result): 

```sql
-- Define the name of the table will be created
create table romo.final_habitat_2 as 
-- Find areas that are common between the Pine and inhydrohuman
select ST_Intersection(inthydrohuman.geom, (
-- Union all geoemetries from the vegetation table where mapclass contains any string with Pine on it
select ST_Union(pine.geom) from (
select * from romo.vegetation where mapclass like '%Pine%') as pine, 
romo_bnd as boundary
-- Use only the Pine geometries that are within the park boundary
where ST_Intersects(pine.geom, boundary.geom))) as geom 
from (
-- Find the areas that are within the hydro bufffers but far from the humand buffers
select ST_Intersection(outhumanbuffers.geom, hydrobuffers.geom) as geom from 
-- Union the geometries of the unioned hydrobuffers
(select ST_Union(hydrobuffers.geom) as geom
from(
-- Buffer streams and lakes and combine the geometries into a single table
-- Since geometries still separeted we need the ST_Union above
select gid, ST_Buffer(geom, 100) as geom from romo.lakes
union all
select gid, ST_Buffer(geom, 100) as geom from romo.streams) as hydrobuffers) as hydrobuffers, 
(select ST_Difference(boundary.geom,
-- Union the geometries of the unioned humanbuffers
(select ST_Union(humanbuffers.geom) from
-- Buffer roads, trail, and campsite and combine the geometries into a single table
-- Since geometries still separeted we need the ST_Union above
(select gid, ST_Buffer(geom, 150) as geom from romo.roads
union all
select gid, ST_Buffer(geom, 100) as geom from romo.trails
union all
select gid, ST_Buffer(geom, 200) as geom from romo.campsites) as humanbuffers)) 
as geom from romo.romo_bnd as boundary) as outhumanbuffers) as inthydrohuman;
```




