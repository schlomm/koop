#Koop - Custom Setup

Information about koop can be found [here](https://github.com/Esri/koop).  In short: koop is something like an adapter between several "GIS-APIs" to serve those as ESRI Feature Service or as a geojson output. 
For more information check the above link.

##Preparing and installing the system

###### Prepare System
	sudo apt-get update
	sudo apt-get install
	sudo apt-get install software-properties-common
	sudo apt-get install libpq-dev

#### Install current g++ (c compiler)	
	sudo add-apt-repository ppa:ubuntu-toolchain-r/test
	sudo apt-get update
	sudo apt-get install g++

#### Install current nodejs version
	sudo add-apt-repository ppa:chris-lea/node.js
	sudo apt-get update	
	sudo apt-get install nodejs
	sudo apt-get install nodejs build-essential
#### Install Git
	sudo apt-get install git

#### Install postgresql
	sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt trusty-pgdg main" >> /etc/apt/sources.list'
	wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | sudo apt-key add -
	sudo apt-get update
	sudo apt-get install postgresql-9.3-postgis-2.1 postgresql-contrib

#### Configure postgresql	
	sudo su -l postgres
	postgres@ubuntu-14:~$ pg_createcluster 9.3 main --start
	postgres@ubuntu-14:~$ exit
	sudo service postgresql start

	sudo -u postgres psql postgres
	\password postgres ###change password for 'postgres'-user
	### Enable PostGIS Extenson for database via SQL or via pgAdmin
	CREATE EXTENSION POSTGIS


	### Database connection to the world:
	cd /etc/postgresql/9.3/main 
	sudo nano hba.conf-file
	### place the following line: "host all all 0.0.0.0/0 md5" in the corresponding code-block (at the end of the file)
	sudo nano postgresql.conf 
	### change the "# CONNECTIONS AND AUTHENTICATION #" part to : listen_addresses = '*' (remove # and paste *)
	sudo /etc/init.d/postgresql restart

----------

### koop setup
	git clone https://github.com/Esri/koop.git
	cd koop/
	### Edit the providers.json, so custom providers also be installed by executing setup.js
	### Custom providers.json can look like the following
	--------
	{
	  "koop-gist": "https://github.com/chelm/koop-gist/tarball/master",
	  "koop-github": "https://github.com/chelm/koop-github/tarball/master",
	  "koop-agol": "https://github.com/Esri/koop-agol/tarball/master",
	  "koop-socrata": "https://github.com/chelm/koop-socrata/tarball/master",
	  "koop-ckan": "https://github.com/chelm/koop-ckan/tarball/master",
	  "koop-acs": "https://github.com/chelm/koop-acs/tarball/master",
	  "koop-dwd-wfs": "https://github.com/ubergesundheit/koop-dwd-wfs/tarball/master"
	}
	--------
	node setup.js



#### Enable Database Support for koop to cache data in PostGIS
	### Set the right runtime.json in /koop/config/runtime.json
	{
	  "db":{
    "postgis": {
      "conn": "postgres://postgres:postgrespassword@localhost:5432/postgres"
    }
	  },
	  "server": {
	  "port": 1337
	  }
	}

	### where "postgis" is the database type (so the extension has to be enabled first)
	### where "postgres://postgres:postgrespassword@localhost:5432/postgres"  is -> 'weißichnicht:('://user:databasepassword@host:port/databasename

#### Install koop-application
	cd /koop ### within koop-dir
	npm install

----------

### Koop-Github Gist Module configuration:
	cd koop/node_modules/koop-gist/models
	### Edit the example comfig.js
	nano config.js.example
#### Paste Github Token:
	// add your API Key and cp this file to config.js
	exports.token = "token here"; ### has to be generated with Github under you Account preferences
#### Copy 'config.js' to koop/node_modules/koop-github/models:
	cp config.js.example config.js

----------

### Koop-Github Module configuration:
	cd koop/node_modules/koop-github/models
	### Edit the example config.js
	nano config.js.example
#### Paste Github Token:
	// add your API Key and cp this file to config.js
	exports.token = "token here"; ### has to be generated with Github under you Account preferences
#### Copy 'config.js' to koop/node_modules/koop-github/models:
	cp config.js.example config.js

----------

### Koop-Socrata configuration:
#### Install koop-socrata Extension, 
....but only if not already installed (check koop/node_modules/ beforehand) or if not installed by default using a customized providers.json

	npm install https://github.com/chelm/koop-socrata/tarball/master
#### Register a socrata-Provider (Please note that the koop-server has to be up and running)
	curl --data "host=https://data.nola.gov&id=nola" localhost:1337/socrata
	curl --data "host=https://data.seattle.gov&id=seattle" localhost:1337/socrata
	curl --data "host=https://data.colorado.gov&id=colorado" localhost:1337/socrata
#### Get a list for installed socrata providers via:
	localhost:1337/<id>
#### Get more information about provider by specifying the provider-id:
	localhost:1337/socrata/<id>

----------

#### Koop-AGOL (ArcGIS Online) configuration:
#### Install koop-agol Extension, 
....but only if not already installed (check koop/node_modules/ beforehand) or if not installed by default using a customized providers.json

	npm install https://github.com/chelm/koop-socrata/tarball/master	
#### Add provider for koop-agol:
	curl -i -X POST -d "id=arcgis" -d "host=https://www.arcgis.com" localhost:1337/agol
#### Get a list for installed agol providers via:
	localhost:1337/agol
#### Get more information about provider by specifying the provider-id:
	localhost:1337/agol/arcgis

----------

### Using koop
``node server.js`` , where putput is loged in current terminal session
``nohup node server.js > output.log &'``, where output is logged to ``output.log`` and where server is not terminated when terminal is closed or looses connection

#### Kill Server
If ``node server.js`` was used:  ``CTRL+C``
If  ``nohup node server.js > output.log &`` was used:  ``kill <processID>``
Kill node.js Server by knowing the running port:

	sudo fuser -v 1337/tcp // gives you the process running on port 1337
	sudo fuser -vk 1337/tcp	// kills all running processes on port 1337

----------

#### Error & Fixes
If there is something to to with the default.yml, you need to install the js-yaml module for nodejs in the "koop/node_modules" folder via

	npm install js-yaml
More infos here: https://www.npmjs.org/package/js-yaml

------

#### Examples on own machine: http://178.62.233.145:1337/
#####Github: 
http://178.62.233.145:1337/github (with Box for pasting github-repo)
Please note that you can **not** use the geojson-extension at the end of the path
Raw Geojson-Viewer:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/keepsmiling
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries_small
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small

Live-Preview
http://178.62.233.145:1337/github/schlomm/Random_Stuff/keepsmiling/Preview
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries/Preview
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries_small/Preview
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/Preview

Feature Service (if you want to service as points, change "FeatureServer/0" to "FeatureServer/1"):
http://178.62.233.145:1337/github/schlomm/Random_Stuff/keepsmiling/FeatureServer/0
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries/FeatureServer/0
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries_small/FeatureServer/0
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/FeatureServer/0



#####Gist: 
http://178.62.233.145:1337/gist (with Box for pasting gist)
Raw Geojson Viewer:
http://178.62.233.145:1337/gist/784a0c08c32ca135ed15
http://178.62.233.145:1337/gist/7f31fce21894cc5529d9
http://178.62.233.145:1337/gist/9c216118fff9c58fc7fa <--- With Errors
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small

Live-Preview:
http://178.62.233.145:1337/gist/784a0c08c32ca135ed15/Preview
http://178.62.233.145:1337/gist/7f31fce21894cc5529d9/Preview
http://178.62.233.145:1337/gist/9c216118fff9c58fc7fa/Preview <--- With Errors
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/Preview

Feature Service (if you want to service as points, change "FeatureServer/0" to "FeatureServer/1"):
http://178.62.233.145:1337/github/schlomm/Random_Stuff/keepsmiling/FeatureServer/0
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries/FeatureServer/0
http://178.62.233.145:1337/github/schlomm/Random_Stuff/world_countries_small/FeatureServer/0
Subdirs:
http://178.62.233.145:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/FeatureServer/0



#####Socrata: 
List all registered socrata-providers: http://178.62.233.145:1337/socrata
https://data.colorado.gov/Geo-Data/Map-of-Colorado-County-Seats/szgw-bkcc
http://178.62.233.145:1337/socrata/colorado/szgw-bkcc/
http://178.62.233.145:1337/socrata/colorado/szgw-bkcc/preview
http://178.62.233.145:1337/socrata/colorado/szgw-bkcc/FeatureServer/0

https://data.seattle.gov/Education/Seattle-schools/uanm-dxsk
http://178.62.233.145:1337/socrata/seattle/uanm-dxsk
http://178.62.233.145:1337/socrata/seattle/uanm-dxsk/preview
http://178.62.233.145:1337/socrata/seattle/uanm-dxsk/FeatureServer/0


#####AGOL: 
List all registered agol-providers: http://178.62.233.145:1337/agol/
Access ArcGIS Online: http://178.62.233.145:1337/agol/arcgis
**Files are needed (Shapes, csv) and they need to be publicly accessible **
http://178.62.233.145:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38
http://178.62.233.145:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38/preview
http://178.62.233.145:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38/FeatureServer

Delete Datasets via Rest-get (using e.g. Postman)
http://178.62.233.145:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38/0/drop

#####DWD-WFS:
http://178.62.233.145:1337/dwd/dwd:BASISGRUNDKARTE&maxFeatures=50

