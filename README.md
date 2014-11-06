#Koop - Custom Setup

Information about koop can be found [here](https://github.com/Esri/koop).  In short: koop is something like an adapter between several (mapping-)APIs to serve those as ESRI Feature Service or as a geojson output. You can also describe it as a small "Feature Manipulation Service".
For more information check the above link.

##Preparing and installing the system
####  Add repositories and sources and update

 1. `sudo add-apt-repository ppa:ubuntu-toolchain-r/test`
 2. `sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt trusty-pgdg main" >> /etc/apt/sources.list`
 3. `wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | sudo apt-key add -`
 4. `sudo apt-get update`
 5. `sudo apt-get install`
 
#### Install needed software (g++ compiler, nodejs, git, postgresql, postgis)

 6. `sudo apt-get install software-properties-common  `
 7. `sudo apt-get install g++`
 8. `sudo apt-get install libpq-dev`
 9. `sudo apt-get install nodejs`
 10. `sudo apt-get install nodejs build-essential`
 12. `sudo apt-get install git`
 13. `sudo apt-get install postgresql-9.3-postgis-2.1 postgresql-contrib`

#### Configure postgresql	

 1. `sudo su -l postgres` 
 2. `postgres@ubuntu-14:~$ pg_createcluster 9.3 main --start`
 3. `postgres@ubuntu-14:~$ exit` 
 4. `sudo service postgresql start`
 5. `sudo -u postgres psql postgres`

Change password for 'postgres'-user and enable PostGIS Extension (can also be done via pgAdmin)

 6. `\password postgres`
 7. `CREATE EXTENSION POSTGIS`

#### Database configuration
Database connection to the world (of course only if you want to have this publicly available database) 
    
 1. `nano /etc/postgresql/9.3/main/pg_hba.conf`
 2.  Place the following line:  `"host all all 0.0.0.0/0 md5"` in the corresponding code-block (somewhere at the end of the file) and save the changes.
 3. `sudo nano postgresql.conf` 
 3. Change the "# CONNECTIONS AND AUTHENTICATION #" part to : `listen_addresses = '*'` (remove # and paste *)
 4. Restart postgresSQL database: `sudo /etc/init.d/postgresql restart`

----------

### koop setup preparations
 1. `git clone https://github.com/Esri/koop.git`
 1. `cd koop/`

Edit the providers.json, so custom providers also be installed by executing setup.js. A custom providers.json can look like the following

	--------
	{
	  "koop-gist": "https://github.com/chelm/koop-gist/tarball/master",
	  "koop-github": "https://github.com/chelm/koop-github/tarball/master",
	  "koop-agol": "https://github.com/Esri/koop-agol/tarball/master",
	  "koop-socrata": "https://github.com/chelm/koop-socrata/tarball/master",
	  "koop-ckan": "https://github.com/chelm/koop-ckan/tarball/master",
	  "koop-acs": "https://github.com/chelm/koop-acs/tarball/master",
	  "koop-dwd": "https://github.com/schlomm/koop-dwd/tarball/master",
	  "koop-ckan_govdata": "https://github.com/schlomm/koop-ckan_govdata/tarball/master",
	}
	--------
 1. Execute the `setup.js` via node, so the providers will be added to the package.json: `node setup.js`



#### Enable Database Support for koop to cache data in PostGIS
Set the right runtime.json in /koop/config/runtime.json

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
	### where "postgres://postgres:postgrespassword@localhost:5432/postgres"  is -> 'databasetype'://user:databasepassword@host:port/databasename

#### Install koop-application
Within the koop dir, install koop

1. `cd /home/koop`
2. `npm install`

----------

### Koop-Github Gist Module configuration:

1. For using [koop-gist](https://github.com/chelm/koop-gist) you need to have a Github Token, which has to be generated in Github under you Account preferences. This has to be stored in the models-dir of the koop-gist-module.
2. Open and Edit the config-file: `nano koop/node_modules/koop-gist/models/config.js.example`
2. Add your API Key: `exports.token = "token here";`
3. Save file under `config.js` or  simply copy it via `cp config.js.example config.js`

#### Examples:
- http://localhost:1337/gist (with Box for pasting gist)  
- http://localhost:1337/gist/784a0c08c32ca135ed15  
- http://localhost:1337/gist/7f31fce21894cc5529d9   
- http://localhost:1337/gist/784a0c08c32ca135ed15/preview
- http://localhost:1337/gist/7f31fce21894cc5529d9/preview  
- http://localhost:1337/gist/784a0c08c32ca135ed15/FeatureServer  
- http://localhost:1337/gist/7f31fce21894cc5529d9/FeatureServer   

----------

### Koop-Github Module configuration:
1. For using [koop-github](https://github.com/chelm/koop-github) you need to have a Github Token, which has to be generated in Github under you Account preferences. This has to be stored in the models-dir of the koop-gist-module.
2. Open and Edit the config-file: `nano koop/node_modules/koop-github/models/config.js.example`
2. Add your API Key: `exports.token = "token here";`
3. Save file under `config.js` or  simply copy it via `cp config.js.example config.js`

#### Examples:
- If you want to access files in a Github Repo-Folder, you need to separate the folder-folder and the folder-file with `::`   
- http://localhost:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small  
- http://localhost:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/Preview
- http://localhost:1337/github/schlomm/Random_Stuff/examples::examples::world_countries_small/FeatureServer/0  

----------

### Koop-Socrata setup and configuration:
Install koop-socrata Extension only if it is not already installed (check koop/node_modules/ beforehand) or if it is not installed by default using a customized providers.json (check [koop-setup preparations section](https://github.com/schlomm/koop/blob/master/README.md#koop-setup-preparations)

 - `npm install https://github.com/chelm/koop-socrata/tarball/master` 

#### Register a socrata-Provider 
Please note that the koop-server has to be up and running). Some example socrata portals.

 - `curl --data "host=https://data.nola.gov&id=nola" localhost:1337/socrata`
 - `curl --data "host=https://data.seattle.gov&id=seattle" localhost:1337/socrata`
 - `curl --data "host=https://data.colorado.gov&id=colorado" localhost:1337/socrata`

#### Use koop-socrata

 - Get a list of all registered socrata providers: `localhost:1337/socrata/`
 - Get more information about a specific provider by specifying the provider-id: `localhost:1337/socrata/<id>`
 - Access socrata dataset as raw geojson: `localhost:1337/socrata/<socrataportal>/<datasetid>`
 - Access socrata dataset as ESRI FeatureService: `localhost:1337/socrata/<socrataportal>/<datasetid>/FeatureServer`
 - Preview socrata dataset on a map: `localhost:1337/socrata/<socrataportal>/<datasetid>/Preview`

#### Examples:
- https://data.seattle.gov/Education/Seattle-schools/uanm-dxsk 
- http://178.62.233.145:1337/socrata/seattle/uanm-dxsk 
- http://178.62.233.145:1337/socrata/seattle/uanm-dxsk/preview 
- http://178.62.233.145:1337/socrata/seattle/uanm-dxsk/FeatureServer/0

----------

### Koop-CKAN setup and configuration:
Install koop-ckan Extension only if it is not already installed (check koop/node_modules/ beforehand) or if it is not installed by default using a customized providers.json (check [koop-setup preparations section](https://github.com/schlomm/koop/blob/master/README.md#koop-setup-preparations)

 - `npm install https://github.com/chelm/koop-ckan/tarball/master` 

#### Register a ckan-Provider 
Please note that the koop-server has to be up and running). Some example socrata portals.

 - ``curl --data "host=https://catalog.data.gov&id=datagov" localhost:1337/ckan` 
 - `curl --data "host=https://data.hdx.rwlabs.org&id=rwlabs" localhost:1337/ckan`
 - `curl --data "host=https://offenedaten.de&id=offenedaten" localhost:1337/ckan`
 - `curl --data "host=http://suche.transparenz.hamburg.de&id=hamburg" localhost:1337/ckan`
 - `curl --data "host=http://demo.ckan.org&id=demockan" localhost:1337/ckan`
 - `curl --data "host=http://data.kk.dk/&id=datakk" localhost:1337/ckan`

Please note that ckan-portals and their datasets have to follow the following structure:

- portalURL/api/3/action/package_list
- portalURL/api/3/action/package_show?id=datasetid
- portalURL/api/3/action/package_search or /api/3/action/package_search?q=datasetid

#### Use koop-ckan

 - Get a list of all registered ckan providers: `localhost:1337/ckan/`
 - Get more information about a specific provider by specifying the provider-id: `localhost:1337/ckan/<id>`
 - Access ckan dataset as raw geojson: `localhost:1337/ckan/<ckanportal>/<datasetid>`
 - Access ckan dataset as ESRI FeatureService: `localhost:1337/ckan/<ckanportal>/<datasetid>/FeatureServer`
 - Preview ckan dataset on a map: `localhost:1337/ckan/<ckanportal>/<datasetid>/Preview`

----------

### Koop-AGOL (ArcGIS Online) setup and configuration:
Install koop-agol extension only if it is not already installed (check koop/node_modules/ beforehand) or if it is not installed by default using a customized providers.json (check [koop-setup preparations section](https://github.com/schlomm/koop/blob/master/README.md#koop-setup-preparations)

- `npm install https://github.com/chelm/koop-agol/tarball/master`


#### Register a agol-Provider 
Please note that the koop-server has to be up and running). Some example socrata portals.

 - `curl -i -X POST -d "id=arcgis" -d "host=https://www.arcgis.com" localhost:1337/agol`

#### Use koop-agol

 - Get a list of all registered agolproviders: `localhost:1337/agol/`
 - Get more information about a specific provider by specifying the provider-id: `localhost:1337/agol/<id>`
 - Access agol dataset as raw geojson: `localhost:1337/agol/<agolportal>/<datasetid>`
 - Access socrata dataset as ESRI FeatureService: `localhost:1337/agol/<agolportal>/<datasetid>/FeatureServer`
 - Preview socrata dataset on a map: `localhost:1337/agol/<agolportal>/<datasetid>/Preview`

#### Examples:
- Access ArcGIS Online: http://178.62.233.145:1337/agol/arcgis  
- **Files are needed (Shapes, csv) and they need to be publicly accessible **  
- http://localhost:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38  
- http://localhost:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38/preview  
- http://localhost:1337/agol/arcgis/99de7faee9984377abc4caa2b283ac38/FeatureServer

----------

###Other koop-proviers setup and configuration
If you want to use other available providers, it's mainly the same as the above mentioned. Some do not need to have the registering-procedure, because the are portal specific.

----------
### Using koop

- Start node server: ``node server.js`` , where output is loged in current terminal session
- Start node server and keep it up and running: `nohup node server.js > output.log &'`, where output is logged to `output.log` and where the server is not terminated when terminal is closed or looses connection. You can also press CTRL+C
- Kill Server:
	- If `node server.js` was used:  `CTRL+C`
	- If  `nohup node server.js > output.log &`, you cann kill the node.js server by knowing the running port: 
		1. ``sudo fuser -v 1337/tcp` - gives you the process running on port 1337
		2. `sudo fuser -vk 1337/tcp` - kills all running processes on port 1337
- List all installed providers: `localhost:1337/providers`
- Delete ckan:services via SQL: `DELETE FROM "ckan:services" WHERE id = 'govdatadaten' OR id = 'govdatanew'`;

----------


#### Error & Fixes
If there is something to to with the default.yml, you need to install the js-yaml module for nodejs in the "koop/node_modules" folder via `npm install js-yaml`. More infos here: https://www.npmjs.org/package/js-yaml
