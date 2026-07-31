 
Aprsd Configuration Reference
=============================

Polaric-aprsd is a rather advanced piece of software. It can be configured many ways. The most used settings can be handled by the web-interface. All settings can be found in config files that can be edited manually. The configuration files to be edited manually are located in */etc/polaric-aprsd/* The most important one is *server.ini*. We describe the options in detail below. 

Configuration file: server.ini
------------------------------
    
The most used properties are handled by a web-interface and are stored in in another file: */var/lib/polaric/config.xml*. This file should normally not be edited by hand. For more advanced needs we can edit *server.ini* to configure aprsd. It contains some additional «properties» consisting of name-value pairs. Comment lines start with '#'. There are explanations in the file. The table below summarizes what properties which may be set in this file. It is not complete though... 

=============================== ===================================================================
 Property                       Meaning
=============================== ===================================================================
``timezone``                    Time zone. In Norway we use "Europe/Oslo"
``httpserver.port``             HTTP server listens on this port
``httpserver.proxy``            Set to true if used with a proxy (for mdns announcement)
``httpserver.secure``           Set HTTPS mode for server
``httpserver.keystore.pw``      Password for keystore. This is authomatically set by scripts. 
``httpserver.securesession``    Use secure flag (force HTTPS) for login session
``httpserver.alloworigin``      Origin-URLs accepted for CORS access (regular expression).
``httpserver.filedir``          Location of static files available to backend webserver
``aprsd.log.level``             What to be reported in log file (level 0-4)
``channel.<chname>.logpackets`` Turn on logging of packets for channel <chname>
``channel.<chname>.log.level``  Set log level for channel <chname> (level 0-4)
``remotectl.userinfo``          Send logon usernames to as APRS messages [1]_ Regex on server id
``remotectl.encrypt``           Encrypt remotectl messages to [2]_ Regex on server id.
``remotectl.encrypt.old``       Old method of encrypting only usernames. Regex on server id.
``map.icon.default``            What icon to be used by default for APRS items
``offlinedetector.on``          Turn on the offline detector (that pings a known server)
``offlinedetector.host1``       Base URL of server to ping. Host2 and 3 can be set as backups      
``offlinedetector.interval``    Check interval in seconds (default 120)
``offlinedetector.timeout``     Default 5 seconds
=============================== ===================================================================

.. [1] Be careful to check the regulations before encrypting messages to be sent over HAM radio
       We should respect privacy regulations. These settings only make sense if using short-messaging 
       between different servers. 
       
       
Other config files
------------------

Here is a summary of the files and directories under /etc/polaric-aprsd used for configuration

**config.d** (directory)
    Config files which defines properties put here will be automatically run after server.ini. Configs for plug-ins are put here. 
    
**groups** 
    Polaric Server supports role-based authorization in the sense that a user can be associated with a group (or role). In this file we define the possible roles and what authorizations they have. Roles defined here can also be referenced in the *view.profiles* script.

**aprsfilters**
    Predefined APRS-IS filters. When using the extended filter-language on Polaric Server instances, we can use P/*name* where *name* identifies a filter defined in this file. The filter used must be defined on the server where it is used. 

**symbols**
    An APRS symbol may correspond to an icon (graphic symbol) to be placed on the map. In Polaric Server we may use icons with a size of 22×22 pixels. Icons are placed in the subdirectory '/usr/share/polaric/icons'. The file '/etc/polaric-aprsd/symbols' defines a mapping between APRS symbols (two characters: symbol table and symbol code) and a corresponding icon file. Note that we in this file can use regular expressions to match symbols. 
    
**signs** (optional)
    This can be used to define simple, static geographical objects to be placed on the map. Note that these are not APRS objects and are therefore local to the server instance. For each object, define the position (UTM), what map-scales in which to display the object, a URL (or '-' if no URL), an icon (I recommend to use smaller icons placed in the directory 'icons/signs') and a short text. 
    
**view.profiles**
    This is a script where we set up filters on what to show on the map and how to show it. Filters can be based on tags and other attributes of point objects, etc.. See documentation on the filter script language here. 
    
**trailcolour**
    A simple setup of colours combinations to be used for trails. 

**init.tnc** (optional)
    Commands to be sent to TNC when starting (TNC2 channel type). Lines starting with # are comments.
    
**scripts.conf**
    Here we can configure shell-scripts to be run by aprsd. Can be requested by clients through the REST API. See this file for more information. 
    
**scripts.conf.d** (directory)
    Config files for scripts put here will be automatically run after scripts.conf. Same format as scripts.conf. Config for scripts belonging to plugins are put here.  
    
**scripts** (directory)
    The scripts (for example bash-scripts) are put here. 

**keys** (directory)
    Crypto keys or other secret keys.

**keys/peers**
    Secret keys for server-to-server authentication. Each line in this file is a service-name and a key. It is recommended that keys are 64 character base64 encoded strings generated by a secure random generator. 

