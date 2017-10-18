# Stack4Things IoTronic (standalone version) installation guide for Ubuntu 14.04

We tested this procedure on a Ubuntu 14.04 (within a LXD container also). Everything needs to be run as root.

## Install requirements

##### Install dependencies via apt-get
```
apt -y install python-dev libyaml-dev libpython2.7-dev mysql-server nmap apache2 unzip socat bridge-utils python-pip python-httplib2
```

#### Install Crossbar.io router
```
apt-key adv --keyserver hkps.pool.sks-keyservers.net --recv D58C6920 && sh -c "echo 'deb http://package.crossbar.io/ubuntu trusty main' | sudo tee /etc/apt/sources.list.d/crossbar.list"
apt update
apt install crossbar
```

##### Install latest NodeJS (and npm) distribution:
```
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
apt-get install -y nodejs
node -v

npm install -g npm
npm config set python `which python2.7`
npm -v
```

##### Configure npm NODE_PATH variable
```
echo "export NODE_PATH=/usr/lib/node_modules" | sudo tee -a /etc/profile
source /etc/profile > /dev/null
echo $NODE_PATH
```

##### Install external NPM dependencies
```
npm install -g https://github.com/PlayNetwork/node-statvfs/tarball/v3.0.0
```
## Install IoTronic-standalone

You can choose to install IoTronic via NPM or from source-code via Git.

#### Install via NPM
```
npm install -g --unsafe iotronic-standalone

echo "export IOTRONIC_HOME=/var/lib/iotronic" >> /etc/profile
source /etc/profile

npm install -g node-reverse-wstunnel
```
during the installation the procedure asks the following information:

* Enter network interface: e.g. "eth0", "enp3s0", etc.

* Enter MySQL password: in order to access to "s4t-iotronic" database.



#### Install from source-code

* ##### Install dependencies using npm:
```
npm install -g node-reverse-wstunnel requestify mysql nconf ip express node-uuid autobahn log4js q fs-access mknod body-parser
```

* ##### Setup IoTronic environment
```
mkdir /var/lib/iotronic/
cd /var/lib/iotronic/

git clone git://github.com/MDSLab/s4t-iotronic-standalone.git
mv s4t-iotronic-standalone/ iotronic-standalone

cp /var/lib/iotronic/iotronic-standalone/etc/init.d/s4t-iotronic /etc/init.d/
chmod +x /etc/init.d/s4t-iotronic
sed -i '/^ *#/b; s%exit 0%/etc/init.d/crossbar start\n/etc/init.d/s4t-iotronic start\nexit 0%g' /etc/rc.local

mkdir /var/lib/iotronic/drivers/
mkdir /var/lib/iotronic/plugins/
mkdir /var/lib/iotronic/schemas/

echo "export IOTRONIC_HOME=/var/lib/iotronic" >> /etc/profile
source /etc/profile
```

* ##### Configure Crossbar.io router
```
mkdir /etc/crossbar
cp /var/lib/iotronic/iotronic-standalone/etc/crossbar/config.example.json /etc/crossbar/config.json
cp /var/lib/iotronic/iotronic-standalone/etc/init.d/crossbar /etc/init.d/
chmod +x /etc/init.d/crossbar
/opt/crossbar/bin/crossbar check --cbdir /etc/crossbar
```
Please, note that the config.example.json coming with the iotronic-standalone package sets the name of the WAMP realm to "s4t" and the Crossbar.io listening port to "8181". If you want to change such values, please consider that later on you will need to correctly change them in other configuration files. 


* ##### Configure IoTronic
First of all, you need to import the Iotronic database schema. During the installation of the MySQL package you should have been asked for a database root password. Please, substiture <DB_PASSWORD> with the one you chose. Also, please note that name of the database is set to "s4t-iotronic". If you want to change it, please consider that later on you will need to correctly change it in other configuration files.
```
mysql -u root -p<DB_PASSWORD> < /var/lib/iotronic/iotronic-standalone/utils/s4t-db.sql
```

Then, copy the example of IoTronic configuration file coming with the package in the correct path. 
```
cp /var/lib/iotronic/iotronic-standalone/lib/settings.example.json /var/lib/iotronic/settings.json
``` 
Please, note that the settings.example.json coming with the iotronic-standalone package sets the IoTronic listening port to "8888", the database name to "s4t-iotronic" (the database server is supposed to be running locally), the WAMP realm to "s4t" (the Crossbar.io WAMP router is supposed to be running locally on port 8181). If you want to change such values, please consider that later on you will need to correctly change them in other configuration files. 

Specify the network interface that IoTronic is supposed to use (e.g., change <INTERFACE> with "eth0").
```
sed -i "s/\"interface\": \"\"/\"interface\":\"<INTERFACE>\"/g" /var/lib/iotronic/settings.json
```

Specify the database password (use the same password you set while installing the MySQL package).
```
sed -i "s/\"password\": \"\"/\"password\":\"<DB_PASSWORD>\"/g" /var/lib/iotronic/settings.json
```

## Start Lightning-rod

#### Start services
```
/etc/init.d/crossbar start
```
Now you are ready to start Iotronic:
```
/etc/init.d/s4t-iotronic start
```
You can check logs by typing:
```
tail -f /var/log/iotronic/s4t-iotronic.log
```
