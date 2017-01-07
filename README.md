# EU Forest species
Darwin Core Archive of EU-Forest species originally published in https://dx.doi.org/10.1038/sdata.2016.123 and
with data made available at https://dx.doi.org/10.6084/m9.figshare.c.3288407

EU-Forest greatly extends the publicly available information on the distribution of European tree species by adding almost half a million of tree occurrences derived from National Forest Inventories for 21 countries in Europe.

## Data transformation
The original coordinates are given as EPSG:3035 which we transform to WGS84 using PostGIS. We also need to assign an occurrenceID which is required for GBIF indexing. For this we use a md5 hash of the entire row. 

Load the data into postgres with a table like this:

    CREATE TABLE species (
     x integer,
     y integer,
     country text,
     species text,
     dbh1 integer,
     dbh2 integer,
     nfi text,
     ff text,
     bs text,
     eeo text 
    );
    
    \copy species from 'EUForestspecies.csv' CSV

Add a WGS84 point and a key column:

    ALTER TABLE species ADD COLUMN md5 text UNIQUE;
    ALTER TABLE species ADD COLUMN point geometry(Point,4326);
    UPDATE species set md5=md5(concat_ws(',', x,y,country,species,dbh1,dbh2,nfi,ff,bs,eeo));
    UPDATE species set point=ST_Transform(ST_SetSRID(ST_MakePoint(x,y), 3035), 4326);
        
Lastly we export the relevant columns to a CSV file:

    \copy (SELECT md5, ST_X(point), ST_Y(point), country, species from species) to 'species.csv' CSV NULL AS ''
    
Done.