# postgresql-gis_notes
Series of functions and scripts for Postgresal/PostGIS 

## PostGIS scripts
### Extract coordinates from geom column and update coordinate fields with these values

```sql

SELECT ST_x(geom) from "mytable"; -- extract the x (longitude value)

SELECT ST_x(geom) from "mytable"; -- extract the y (latitude value)
```
Update fields

```sql
UPDATE "mytable" SET longitudine = ST_x(geom); -- extract the x (longitude value)
UPDATE "mytable" SET latitude = ST_y(geom); -- extract the x (longitude value

```

#### Create a trigger for automatically update values when a geometry is created or modified

```sql
UPDATE "mytable" SET longitudine = ST_x(geom); -- extract the x (longitude value)
UPDATE "mytable" SET latitude = ST_y(geom); -- extract the x (longitude value

```
