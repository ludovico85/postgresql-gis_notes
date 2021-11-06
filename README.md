# postgresql-gis_notes
Series of functions and scripts for Postgresql/PostGIS
## UPGRADING PSQL FOR WINDOWS
1) Create a copy of the configuration file (pg_hba.conf and postgresql.conf) located in `C:\Program Files\PostgreSQL\XX\data`
2) Obtaining a complete backup of all databases using the `pg_dump` command
`cd C:\Program Files\PostgreSQL\11\bin`
`pg_dump -U postgres -F t punti_investimeno > I:\Backup_db\punti_investimento.tar`
3) Install the new version of Postgresql
4) Stop services of both the versions
5) Create e temporary folder (i.e. I:\temp)
6) Move on the temp folder
7) Run the following comand `"C:\Program Files\PostgreSQL\14\bin\pg_upgrade.exe" -d "C:\Program Files\PostgreSQL\11\data" -D "C:\Program Files\PostgreSQL\14\data" -b "C:\Program Files\PostgreSQL\14\bin" -B "C:\Program Files\PostgreSQL\14\bin" --link --check -U Postgres`

https://www.sqlservercentral.com/articles/upgrade-postgresql-9-x-to-12-x-in-windows

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

#### Create a trigger for automatically update coordinates fields when a geometry is created or modified

Create the function pop_latitude_from_geom()
```sql
CREATE OR REPLACE FUNCTION pop_latitude_from_geom()
	RETURNS TRIGGER AS $poplatitude$

BEGIN
	IF (TG_OP='INSERT' OR TG_OP='UPDATE') THEN

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

The same function for the longitude
```sql
CREATE OR REPLACE FUNCTION pop_longitude_from_geom()
	RETURNS TRIGGER AS $poplongitude$

BEGIN
	IF (TG_OP='INSERT' OR TG_OP='UPDATE') THEN

	UPDATE "mytable"
		SET longitude = ST_x(geom);


	END IF;
	RETURN NEW;
END;

$poplongitude$ LANGUAGE plpgsql;

CREATE TRIGGER poplongitude_insert
	AFTER INSERT ON "mytable"
	FOR EACH STATEMENT EXECUTE PROCEDURE pop_longitude_from_geom();

```


With only a function for both latitude and longitude

```sql
CREATE OR REPLACE FUNCTION pop_latlon_from_geom()
	RETURNS TRIGGER AS $poplatlon$

BEGIN
	IF (TG_OP='INSERT' OR TG_OP='UPDATE') THEN

	UPDATE "mytable"
		SET longitude = ST_x(geom),
			latitude = ST_y(geom);


	END IF;
	RETURN NEW;
END;

$poplatlon$ LANGUAGE plpgsql;
```

Trigger

```sql

CREATE TRIGGER poplatlon_insert
	AFTER INSERT ON "mytable"
	FOR EACH STATEMENT EXECUTE PROCEDURE pop_latlon_from_geom();

CREATE TRIGGER poplatlon_update
	AFTER UPDATE ON "mytable"
	FOR EACH ROW
	WHEN (OLD.geom IS DISTINCT FROM NEW.geom)
	EXECUTE PROCEDURE pop_latlon_from_geom();
```
### Spatial join
#### Update a field based on a spatial relation

When a point is digitized, the information on field regione, comune and provincia will populated automatically based on the spatial relationship between the point layer and the limiti_amministrativi layer.

```sql
CREATE OR REPLACE FUNCTION pop_com_pro_reg_from_geom()
	RETURNS TRIGGER AS $popcomproreg$

BEGIN
	IF (TG_OP='INSERT' OR TG_OP='UPDATE') THEN

	UPDATE "mytable" as r
		SET regione = l."DEN_REG", -- field region
		comune = l."COMUNE", --field comune
		provincia = l."DEN_PROV" --field provincia
		FROM limiti_amministrativi as l
		WHERE ST_Contains(l.geom, r.geom);


	END IF;
	RETURN NEW;
END;

$popcomproreg$ LANGUAGE plpgsql;
```
Create the Trigger
```sql
CREATE TRIGGER pop_com_pro_reg_insert
	AFTER INSERT ON "mytable"
	FOR EACH STATEMENT EXECUTE PROCEDURE pop_com_pro_reg_from_geom();

CREATE TRIGGER pop_com_pro_reg_update
	AFTER UPDATE ON "mytable"
	FOR EACH ROW
	WHEN (OLD.geom IS DISTINCT FROM NEW.geom)
	EXECUTE PROCEDURE pop_com_pro_reg_from_geom();
```

#### Update a field based on a simple join

```sql
CREATE OR REPLACE FUNCTION pop_nome_scientifico_rettili()
	RETURNS TRIGGER AS $popnomscierettili$
	
BEGIN
	IF (TG_OP='INSERT' OR TG_OP='UPDATE') THEN

	UPDATE "Rettili" as r
		SET specie_nome_scientifico = lr."nome_scientifico"
		FROM lista_rettili as lr
		WHERE r.specie_nome_comune = lr.nome_comune;
	END IF;
	RETURN NEW;
END;


$popnomscierettili$ LANGUAGE plpgsql;
```
Create the Trigger
```sql
CREATE TRIGGER pop_nom_sci_insert_rettili
	AFTER INSERT ON "Rettili"
	FOR EACH STATEMENT EXECUTE PROCEDURE pop_nome_scientifico_rettili();

CREATE TRIGGER pop_nom_sci_update_rettili
	AFTER UPDATE ON "Rettili"
	FOR EACH ROW
	WHEN (OLD.geom IS DISTINCT FROM NEW.geom)
	EXECUTE PROCEDURE pop_nome_scientifico_rettili();
```
