# baarle_nassau

Demo project for the Baarle-Nassau municipality.

## Netherlands

Download buildings from https://3dbag.nl/en/download?tid=8-432-304

Result:  8-432-304.gpkg (33MB)

Import in PostGIS:

```sql
ogr2ogr -f PostgreSQL pg:"host=localhost user=postgres password=postgres" -t_srs epsg:7415 8-432-304.gpkg lod22_3d -nln baarle_nl_panden
```

Create 3d Tiles

```shell
pg2b3dm -h localhost -U postgres -c geom -d postgres -t baarle_nl_panden -a identificatie -o baarle_nl_panden --default_color #FF0000
```

## Vlaanderen

3D GRB? Maar aanmelden.. https://download.vlaanderen.be/product/971-3d-grb

Download GRAR.zip from https://download.vlaanderen.be/product/10142-gebouwen-en-adressenregister (1.3 GB)

Select Baaarle-Nassau buildings:

```shell
ogr2ogr -f "ESRI Shapefile" gebouw_baarle.shp  gebouw.shp  -spat 186487.5371 235348.1929 192141.8334 238585.0111 -t_srs epsg:31370 -nlt multipolygon
```

Download DEM from https://download.vlaanderen.be/product/939-digitaal-hoogtemodel-vlaanderen-ii-dtm-raster-1-mper (1.13 GB GeoTIFF)

Drape the Baarle Nassau buildings on the DEM: 

```shell
qgis_process-qgis run native:setzfromraster --INPUT=".\GRAR\20240906_GRAR_Data\Shapefile\gebouw_baarle.shp" --RASTER=".\DHMVIIDTMRAS1m_k08\GeoTIFF\DHMVIIDTMRAS1m_k08.tif" --OUTPUT="gebouw_baarle_draped.gpkg"
```

Import the draped buildings in PostGIS:

```shell
ogr2ogr -f PostgreSQL pg:"host=localhost user=postgres password=postgres"  gebouw_baarle_draped.gpkg -nln baarle_be_panden -t_srs epsg:6190 
```

Create 3d Tiles

```shell
pg2b3dm -h localhost -U postgres -c geom -d postgres -t baarle_be_panden -a id -o baarle_be_panden --default_color #00ff00
```
## Result

https://bertt.github.io/baarle_nassau/demo/geoid/

![image](https://github.com/user-attachments/assets/62e0f209-ca77-4370-b2ca-24f223d3a1da)

## Check

Cultuur centrum Baarle: Pastoor de Katerstraat 5- 7 Baarle-Hertog, Pastoor de Katerstraat 7, 5111 CM Baarle-Nassau, België

NL: identificatie = 'NL.IMBAG.Pand.0744100000000249'

![image](https://github.com/user-attachments/assets/fe5ff2ce-76ff-4803-8722-d9901b88a521)

Altitude = 26.009000778198242

BE: id = https://data.vlaanderen.be/id/gebouw/16500761

![image](https://github.com/user-attachments/assets/5b7388f0-0a5a-4cf4-9e27-ed2ad66000f2)

Altitude = 28.540000915527344

## Converting to ellipsoid heights

NL:

```shell
ogr2ogr -f PostgreSQL pg:"host=localhost user=postgres password=postgres" -t_srs epsg:4979 8-432-304.gpkg lod22_3d -nln baarle_nl_panden
pg2b3dm -h localhost -U postgres -c geom -d postgres -t baarle_nl_panden -a identificatie -o baarle_nl_panden --default_color #FF0000
```

BE:
```shell
ogr2ogr -f PostgreSQL pg:"host=localhost user=postgres password=postgres"  gebouw_baarle_draped.gpkg -nln baarle_be_panden -s_srs epsg:6190  -t_srs epsg:4979 
pg2b3dm -h localhost -U postgres -c geom -d postgres -t baarle_be_panden -a id -o baarle_be_panden --default_color #00ff00
```

Altitude NL = 70.27569416553462

Altitude BE = 70.43654769997616

## Result

https://bertt.github.io/baarle_nassau/demo/ellipsoid/

![image](https://github.com/user-attachments/assets/2d2bf784-1ec0-43c0-96db-4f273ec26577)

