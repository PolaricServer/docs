 
Installing Polaric Server
=========================

This is a description how to install the *Polaric Server* software on a *Debian* based Linux platform. It consists of two main components: The APRS daemon (*polaric-aprsd*) and the Web application (*polaric-webapp2*). In addition there may be plugins. If you are unknown to Linux and Apache system administration, I recommend exploring this a little before attempting to install Polaric Server or that you get help from a friend. If you plan to put up a server permanently on the internet, you should know what you are doing, i.e. you should have some basic knowledge on internet security and firewall setup. I assume you login as *'root'* to be able to perform the installation ('`sudo su`') or prefix commands with *'sudo'*.

The packages should work on Debian stable systems. From mid-june 2023 this will mean 'bookworm'. 'Bullseye' should still work (the next version of Polaric will require Java-17). It should probably also work on 'unstable' or 'testing' as well as recent Debian based distros like Ubuntu, Mint, Raspbian, Armbian, etc. Windows 10/11 with Linux subsystem (With Debian or Ubuntu) should probably work as well though I haven't tested much. 

.. note:: 
 If you install *polaric-database-plugin* on a 'bookworm', 'testing' or 'unstable' based system, there may be an issue with the newest version of *libpostgis-java*. This can be solved by using version 2.4.0 (available in 'bullseye'). Polaric Server 3.0 will fix this. 

.. note::
 It is important that the computer on which to install *Polaric Server* has a clock with correct time. It is recommended to install a *`ntp`* client ('`apt install ntp`'). *Polaric Server* can alternatively use a GPS to adjust the host computer's clock. 

Debian Repository
-----------------

We offer DEB packages to help you install the software. To get started you first need to add a package repository. Do as follows (as root)::

    echo "deb http://aprs.no/debian-rep binary-dev/">> /etc/apt/sources.list
    
You will need to install my cryptographic public key (PGP) to verify the authenticity of repository and the packages. You may do the following. This means that you trust packages signed with my key::

    apt-get install gnupg2 dirmngr
    gpg --keyserver keys.openpgp.org --recv-keys 89E7229CFFD59B2F
    gpg --export --armor 3E61003E24632585EB3DFE3D89E7229CFFD59B2F | apt-key add -

To re-load metadata from the repositories type the command::
 
    apt-get update
    
Installing the APRS daemon
--------------------------

The APRS daemon (polaric-aprsd) is a server program that processes APRS data (from/to APRS-IS and/or TNC) and presents it to the web-application (or it may be set up as an igate). It can be run as a standalone application (e.g. as a igate) or function as a backend server for the web-application and can respond to HTTP requests and deliver XML-data or HTML pages::

   apt-get install polaric-aprsd

The program will be installed and started automatically. To check that it works, take a look at the log file::

   tail -f /var/log/polaric/aprsd.log

If things are ok, you will see new APRS messages arrive. Type CTRL-C to quit.

You should now configure the server with your callsign and you should also review other configuration parameters like the aprs-is server, passcode, etc.. There is a web-interface for basic configuration. By default, it runs on port 8081. Start your web-browser and type in the url: 

`http://<hostname>:8081/config_menu` or if the server runs on the same computer: http://localhost:8081/config_menu You will be prompted for a username and a password. The package initially comes with an admin-user ('admin') with password 'polaric'. You should of course change the password. The configuration you usually need to do can be done through this web interface. It covers the most important settings. There are also some setting (for more advanced users) that can by changed manually by editing the file: `/etc/polaric-aprsd/server.ini` .

For all details, see the configuration reference. Note that configuration through the web interface overrides the manual config in the server.ini file which then can be viewed as default settings.

The server can be restarted through the web config interface or by issuing the command::

    polaric-restart 
    
Installing the web-application
------------------------------

Webapp2 is the new client software. Source code is on `Github <https://github.com/PolaricServer/webapp2>`_. The deb package installs this automatically assuming that an aprsd instance is running on the same machine. It is possible to install it without aprsd (for advanced users). You can install the web-application component as follows::

    apt-get install polaric-webapp2

And you will get a basic installation with OSM (OpenStreetMap) and Norwegian maps. It connects to a local aprsd through port 8081. If you don't install the aprsd, the backend connection will simply fail. You may configure it to use a server at another location if you want to. It also installs mapcache (a plugin for Apache webserver) to cache map-tiles. It depends on the Apache Webserver and will configure it for this application.

By default the webapp can be accessed through http://hostname/aprs where hostname is the ip adress or host name of the machine it is running on. If it is on your own computer, localhost or 127.0.0.1 should do.

You can log in as *'admin'* using this web-interface. Then you have access to configuration and user-management of the system.

Configuring the webapp
----------------------

The following file is the most important place to change configuration. You may review it and edit if necessary. Here you may change setup of backend-server, map-layers, default map views and filters: It is a Javascript file which is run on clients, but even if you don't know Javascript very well, it should be fairly self-explanatory wrt. the most important configuration options::

    /etc/polaric-webapp2/config.js 

The server runs a `Mapcache <https://mapserver.org/mapcache/>`_ instance. It is configured in the following file (See mapcache documentation and the explanations in the file itself for more info)::

    /etc/polaric-webapp2/mapcache.xml

To change the Apache webserver setup for the application, you may edit::

    /etc/apache2/sites-enabled/aprs.conf
    
If you are outside Norway you may want to change the map-layer setups (`config.js` and `mapcache.xml`). I hope to be able to provide better documentation for this. Anyway, you may find information on how to set up map layers in the `OpenLayers documentation <http://www.openlayers.org>`_. Map-layers may also be added in the web interface for individual users. If anyone wants to share their setups, it would be helpful! 
 
  
Installing plugins
------------------

Plugins are optional and easy to install. Plugins with available deb packages are:

 * **polaric-db-plugin**. It uses a `PostgreSQL <https://www.postgresql.org>`_ database for storage and search. It can store APRS traffic 
   to generate historical trails, it can store user-data, etc. It comes with a scripts to help installing 
   and configuring the database, but it may need some additional configuration.
 * **polaric-ais-plugin**. It implements integration of AIS datastream (over TCP). It depends on polaric-aprsd.
 
 
Making it a public service
--------------------------

If you want to have a permanently publicly available online instance on the internet (like aprs.no) you should know what you are doing. The server should be secured properly and configured to be reachable from the internet.

What to consider:

* Where to run the server. In a data center? How to secure it, run it in a DMZ?
* Domain name? Virtual host setup?
* Secure the (frontend) webserver using TLS/SSL. Then you will need certificates for your domain. Consider if you want to force the users to use https always or when logging in to avoid that passwords or other sensitive information is sent in clear.
* The backend (aprsd) by default uses a special port (8081). If the server is to be used across different subnets, I recommend to set up the frontend webserver as a proxy for this. This can easily be done with Apache for both REST API and websocket connections.
* You may need to set up some redirects and URL rewrites to make it work smoothly.



