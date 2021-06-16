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

Create the function pop_latitude_from_geom()
```sql
CREATE OR REPLACE FUNCTION pop_latitude_from_geom()
	RETURNS TRIGGER AS $poplatitude$

BEGIN
	IF (TG_OP='INSERT' AND TG_OP='UPDATE) THEN
	
	UPDATE "mytable"
		SET latitude = ST_y(geom);
	
	
	END IF;
	RETURN NEW;
END;

$poplatitude$ LANGUAGE plpgsql;
```

Create the trigger to apply the function

```sql
CREATE TRIGGER poplatitude_insert
	AFTER INSERT ON "mytable"
	FOR EACH STATEMENT EXECUTE PROCEDURE pop_latitude_from_geom();

CREATE TRIGGER poplatitude_update
	AFTER UPDATE ON "mytable"
	FOR EACH ROW
	WHEN (OLD.geom IS DISTINCT FROM NEW.geom)
	EXECUTE PROCEDURE pop_latitude_from_geom();
```
