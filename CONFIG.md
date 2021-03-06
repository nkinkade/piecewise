## Configuring and Deploying a Piecewise VM

We’re now ready to customize your copy of Piecewise for the region you care about. We recommend starting a text file to document information about your application that you’ll need to configure Piecewise, and that you might want to have on hand for your own documentation.

### Quick start, tl;dr

Configuring and deploying Piecewise will involve the following steps:

  * Ask M-Lab to whitelist a Google Developer account 
  * Setup a project in Google Developer Console
  * Configure Piecewise for your location

and will require the following information:
  * Geographic coordinates forming a bounding box, for example: [-122.6733398438,47.3630134401,-121.9509887695,47.8076208172]
  * Shapefile(s) containing the sub-geographic areas by which the consumed raw data will be aggregated 
  * Topojson or geojson file(s) containing the same sub-geographic areas for the map front-end.
  * A time range for which data should be consumed from M-Lab in format: Jan 1 2014 00:00:00

### Whitelist a Google Account to use with your Piecewise instance

Piecewise requires a Google Account to be configured for ingesting M-Lab data from BigQuery. We recommend creating a separate account to use specifically for this purpose, rather than a personal account.

1. Create/Identify a Google account to use for your instance of Piecewise.

2. Email [support@measurementlab.net](mailto:support@measurementlab.net) to request that your Google account be whitelisted to query the M-Lab dataset.

3. Once M-Lab confirms your account is whitelisted, proceed.

### Configure an API project in the Google Developers console

1. Go to Google Developers console and log in using the account that was whitelisted by M-Lab:[ https://console.developers.google.com/project](https://console.developers.google.com/project)![image alt text](image_0.png)

2. Create a Google-API project (or choose an already existing project) and turn on permissions for the BigQuery API

3. Create a new client ID for the project, selecting "Installed Application" for Application Type  and “Other” for Installed Application Type![image alt text](image_1.png)

4. Save the client_secrets.json file to the piecewise/piecewise folder (it should replace the old client_secrets.json file already in there)

5. Turn on billing for the project in Google console (you will not be billed because your account is whitelisted, but Big Query requires API applications to have billing enabled)

4. In the Overview section of your new project, note the Project ID number

### Configure Piecewise for your location

The purpose of the Piecewise server is to consume M-Lab data from a particular geographic region and to aggregate it by sub-regions within that area. In its current state (Nov.2015), configuring a new Piecewise server for a new region will require you to gather some information, modify a few files, and then run the Piecewise ingest and aggregate scripts.

If you're not ready to prepare your own customization, you can check out the **examples** branch of Piecewise using Git:
<pre>git checkout examples</pre>

The examples branch contains several examples of Piecewise customizations for different geographic areas. There are several directories ending with **_example** which you can use, copy or customize. For this documentation, we’ll walk through how the **baltimore_example** was created.

#### Creating your own configuration folder

You can copy one of these example folders to start your own configuration. We'll make a copy of an existing example, obtain and import the files you need for your location, and then update the content of several files to complete the customization. For this example, we'll walk through how the Baltimore example was created on the linux command line:

1. Copy the folder, and rename key files:

        cp -rf seattle_example baltimore_example
        mv seattle_tasks.yml baltimore_tasks.yml
        mv seattle_center.py baltimore_center.py

2. Remove unneeded files:

        rm seattle_council_districts.geojson
        rm seattle_census10_blockgroups.topojson
        rm -rf seattle_blkgrpce10
        rm seattle_bigquery_results.sql.gz

3. Create a folder for the shapefiles to be used by your Piecewise server:
        mkdir maryland_blkgrps

##### Gather your map files and coordinates

Next, we need to gather some information to configure your Piecewise server:

  * Geographic bounding box coordinates from which raw data will be consumed
  * A shapefile containing the regions by which raw data will be aggregated by piecewise
  * A topojson file corresponding to the same regions as the above shapefile, which will be used by the map front-end

###### Select a geographic bounding box

First, we need to tell Piecewise the geographic area for which it should ingest M-Lab data. This will be in the form of four  coordinates. Use the tool of your choice to select the geographic area you’re interested in. We used [http://boundingbox.klokantech.com/](http://boundingbox.klokantech.com/) and searched for Baltimore, MD. You can also draw an arbitrary bounding box. After selecting an area, copy/paste the coordinates at the bottom of the page to use as the coordinates of your bounding box. Your coordinates should be in a format that looks like this:
-76.711519,39.197207,-76.529453,39.372206

Save these values in a text file to use a bit later.

###### Find the coordinates for the center of your map

You'll also need to tell Piecewise where the center of your map will be. Find the center of your map on Google Maps and enter the latitude and longitude. If we search [https://www.google.com/maps](https://www.google.com/maps) for Baltimore, we get a map with Baltimore at the center and this URL:
<pre>https://www.google.com/maps/place/Baltimore,+MD/@39.2848182,-76.6906973,12z/data=!3m1!4b1!4m2!3m1!1s0x89c803aed6f483b7:0x44896a84223e758</pre>

The map center is the two coordinates after the @ sign in the URL:
<pre>39.2848182,-76.6906973</pre> 

Save these values into the same text file. We'll use them a bit later.

###### Obtain shapefiles for your data aggregation areas

Piecewise will download raw test data from M-Lab that was submitted from within the four coordinates you gathered in the previous step. Its real power, however, is that Piecewise will aggregate the raw data into smaller shaped areas within that bounding box. You can define multiple aggregations and use them as different layers in the same map or visualization. For example, we might use city council districts, counties, countries, census blocks or other shapes to aggregate M-Lab data.

To do these aggregations, Piecewise requires one or more shapefiles containing the areas you wish M-Lab data to be aggregated into. Download your shapefile and topojson file. The US Census Bureau provides downloadable shapefiles for a variety of boundaries in the US, such as census tracts: [https://www.census.gov/geo/maps-data/data/cbf/cbf_blkgrp.html](https://www.census.gov/geo/maps-data/data/cbf/cbf_blkgrp.html). In the US, cities often publish shapefiles for their communities that often have been amended or corrected. If you're interested in a non-US location, check [?this site?](#) for shapefiles related to a specific country. 

Once you locate the shapefile(s) you need, it’s good practice to open the shapefile in QGIS or another program to confirm it’s ok. Then, save the files into your Piecewise directory. In our example, we saved the shapefiles to the folder we previously created:

```
maryland_blkgrps/
    cb_2014_24_bg_500k.cpg    cb_2014_24_bg_500k.shp.ea.iso.xml
    cb_2014_24_bg_500k.dbf    cb_2014_24_bg_500k.shp.iso.xml
    cb_2014_24_bg_500k.prj    cb_2014_24_bg_500k.shp.xml
    cb_2014_24_bg_500k.shp    cb_2014_24_bg_500k.shx
```

##### Obtain a topojson file for your front-end map

If you plan to use a map to display your data, the map front-end Piecewise provides by default requires a topojson file based on the same shapefile you downloaded in the previous section. You can use QGIS or another desktop program to create this file, or use a service like [http://mapshaper.org/](http://mapshaper.org/) to upload your shapefile and then choose to save it in topojson format.
Mapshaper lets you upload a shapefile and convert it to topojson. 

Save the downloaded topojson file to the main directory of your Piecewise install. In our example, this file is called **maryland_blkgrps_2014.json**

We end with this list of files in our Piecewise directory:

```
baltimore_center.py            	
baltimore_tasks.yml         	
center.js        	
extra_data.py
maryland_blkgrps/
  	cb_2014_24_bg_500k.cpg  	cb_2014_24_bg_500k.shp.ea.iso.xml
    cb_2014_24_bg_500k.dbf  	cb_2014_24_bg_500k.shp.iso.xml
    cb_2014_24_bg_500k.prj  	cb_2014_24_bg_500k.shp.xml
    cb_2014_24_bg_500k.shp  	cb_2014_24_bg_500k.shx
maryland_blkgrps_2014.json
piecewise_config.json
README.md
```

#### Update Piecewise configuration files with your values

Now we have all the information we need to configure your Piecewise server. This will involve updating the values in several files in your configuration folder.

1) **piecewise/baltimore_example/center.js** 
This file provides the geographic center of your map. Copy the values from your text file that you located on Google Maps and paste them between the brackets on this line:
<pre>var center = [39.2847064,-76.620486];</pre>

2) **piecewise/baltimore_example/baltimore_tasks.yml**
This is the Ansible tasklist for the Baltimore example. The lines below preceded with a # sign highlight modifications you need to make to the lines below the comment.

```yml
---
- name: Copy geo data to server

# "baltimore_example" is the name of our folder. Change it to the
# name of the folder for your Piecewise instance.

  copy: src=baltimore_example/{{ item.src }} 

  dest=/opt/bq2geojson/html/{{ item.dest }}

  with_items:

# "maryland_blkgrps_2014.json" is the topojson file for our 
# Baltimore example. Change this to the name of your file.

   	- { src: maryland_blkgrps_2014.json, dest: maryland_blkgrps_2014.json }

# "maryland_blkgrps" is the name of our folder containing shapefiles
# for the city of Baltimore, MD. Chnage this to the name of the 
# folder containing your shapefiles.

  	- { src: maryland_blkgrps, dest: '' }
  	- { src: center.js, dest: js/center.js }

- name: Copy extra_data.py to server

# "baltimore_example" is the name of the folder containing our
# configuration of Piecewise for Baltimore, MD. Change this to the 
# name of the folder containing your configuration of Piecewise.

  copy: src=baltimore_example/extra_data.py dest=/opt/piecewise/

- pip: name=ipaddress state=latest

- name: Ingest census blocks to postgres

# In the line below, change "maryland_blkgrps" to the name of the 
# folder containing the shapefiles for your instance of Piecewise.
# The last line also contains the name of the .shp file to be 
# used. Change the name of the .shp file to reflect your intended 
# shapefile.

  command: ogr2ogr -f PostgreSQL -t_srs EPSG:4326 -nln maryland_blkgrps -nlt MultiPolygon 'PG:user=postgres dbname=piecewise' /opt/bq2geojson/html/maryland_blkgrps/cb_2014_24_bg_500k.shp

- name: Install piecewise configuration

# "baltimore_example" is the name of the folder containing our
# configuration of Piecewise for Baltimore, MD. Change this to the 
# name of the folder containing your configuration of Piecewise.

  copy: src=baltimore_example/piecewise_config.json dest=/etc/piecewise/config.json

- name: Restart uwsgi so piecewise config is detected
  service: name=uwsgi state=restarted

- command: python extra_data.py chdir=/opt/piecewise

```

3) **piecewise/baltimore_example/piecewise_config.json**
This is the main configuration file for your Piecewise deployment. You will be updating some sections of this file for your deployment, most importantly information about the aggregations you desire. We’ll use the Baltimore, MD example here and highlight what was changed. At the bottom of this file, in the **filters** json object, you'll edit the beginning and end dates, telling Piecewise the date range of M-Lab data to be consumed, as well as the geographic area from which to pull, using the bounding box coordinates you gathered earlier. 

When you first open it, you will find the options used for Seattle example, where the aggregation is done for council districts and census blocks. You can delete one of the "aggregations" blocks:
```
"aggregations": [{
# DELETE from this line
        "name": "by_council_district",    
        "statistics_table_name": "district_statistics",
        "bins": [
            { "type" : "spatial_join", "table" : "seattle_council_districts", "geometry_column" : "wkb_geometry", "key" : "district", "join_custom_data" : true },
            { "type" : "time_slices", "resolution" : "month" },
            { "type" : "isp_bins", "maxmind_table" : "maxmind", 
                "rewrites" : {
...
            { "type" : "AverageUpload" },
            { "type" : "MedianUpload" }
        ]
    }, {    
# DELETE to the line above
```

Define the remaining "aggregations" block for your instance:

```
"aggregations": [{

      # change "by_county" to any name that is significant

       "name": "by_county", 

      # change "county_statistics" to a table name significant
      #     to your project. This defines the table name to store
      #     aggregated statistics for your Piecewise server.

       "statistics_table_name": "county_statistics", 

       "bins": [

           { "type" : "spatial_join", "table" : "newengland", "geometry_column" : "wkb_geometry", "key" : "geoid", "key_type" : "string" ,"join_custom_data" : true }, <- look in the GeoJSON file and see what key you can use to aggregate, here we use the geoid for aggregation, we also have to mention the type of the key (string)

...

	"filters": [
      # Change the dates below to reflect the start date from which 
      # M-Lab data should be ingested. Leaving the end date far in 
      # the future ensures data will be collected until that date

    	{ "type": "temporal", "after": "Jan 1 2014 00:00:00", "before" : "Jan 1 2050 00:00:00" },

      # Replace the coordinates below with the bounding box 
      # coordinates you obtained earlier.

    	{ "type": "bbox", "bbox": [-76.711519,39.197207,-76.529453,39.372206] },
```

4) **piecewise/playbook.yml**
Lastly, we’ll modify the main Ansible playbook YAML file to point at our new Baltimore example tasks file.

```yml
---
- hosts: all
  tasks:
	- include: system_tasks.yml
  	sudo: True
	- include: user_tasks.yml
	- include: baltimore_example/baltimore_tasks.yml
  	sudo: True
```

Next move on to [Deploying your Piecewise instance](DEPLOY.md)