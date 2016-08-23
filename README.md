# osm-tiles-easy
With these scripts you can start generating your own Openstreetmaps tiles on your virtual private server. This was tested on AWS with Ubuntu Server 14.04 in August 2016.

Step 0. This is not probably needed but it removes a warning about locales.

Use text editor of your choice. I used vi.
sudo nano /etc/environment 
add following lines:
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
(reconnect ssh to see the warning is gone)
	1.	Install Mapnik
sudo add-apt-repository ppa:mapnik/nightly-2.3

sudo apt-get update

sudo apt-get install libmapnik libmapnik-dev mapnik-utils python-mapnik

sudo apt-get install mapnik-input-plugin-gdal mapnik-input-plugin-ogr mapnik-input-plugin-postgis mapnik-input-plugin-sqlite mapnik-input-plugin-osm

To test Mapnik is installed type:
python
import mapnik
(This shouldn't give errors)
exit()

	2.	Postgis

sudo apt-get install postgresql postgresql-contrib postgis
sudo apt-get install postgresql-9.3-postgis-scripts

sudo -u postgres createuser gisuser
sudo -u postgres createdb --encoding=UTF8 --owner=gisuser gis

sudo -u postgres psql
Some postgres commands:
ALTER USER gisuser WITH PASSWORD ‘kakka’;
\c gis
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
\q

	3.	Get an example OSM data file and convert it to .osm
The data file is small and it doesn't take long time to process.

sudo apt-get install osmctools
sudo wget tekieki.fi/file/oulu.osm.pbf
sudo osmconvert oulu.osm.pbf > oulu.osm

	4.	Osm2pgsql

sudo apt-get install osm2pgsql

sudo -u postgres psql
ALTER USER postgres WITH PASSWORD ‘kukka’;
\q

sudo nano /etc/postgresql/9.3/main/pg_hba.conf
Find the row that says like "local  all  postgres  peer"
Change it to: "local  all  postgres  md5"

sudo service postgresql reload

sudo osm2pgsql -s -U postgres -W -d gis oulu.osm
(password ‘kukka’)

	5.	Mapnik_stylesheets
 
sudo apt-get install git unzip zip

sudo apt-get install python-psycopg2 python-shapely python-gdal gdal-bin unifont fonts-dejavu-extra

sudo mkdir mapnik-stylesheets

cd mapnik-stylesheets

sudo git clone https://github.com/openstreetmap/mapnik-stylesheets.git 

sudo sh get-coastlines.sh
(This takes some time)

sudo ./generate_xml.py --dbname gis --host ‘localhost’ --user postgres --port 5432 --password ‘kukka'

sudo nano gen.sh
Write to this file:
sudo ./polytiles.py --bbox 25.45 64.99 25.55 65.04

sudo sh gen.sh

AAAND YOU ARE GENERATING TILES!
The tiles in this example are in /home/ubuntu/mapnik-stylesheets/mapnik-stylesheets/tiles folder.

	6.	Install Apache server and make a simple page to display your tiles

sudo apt-get install apache2

cd /var/www/html

sudo rm index.html

This is a file that contains small website that has a leaflet map displaying your tiles:
wget tekieki.fi/file/kartta.zip
sudo unzip kartta.zip 

sudo cp -r /home/ubuntu/mapnik-stylesheets/mapnik-stylesheets/tiles .

Navigate to your site with browser to see the end result:
