Database Plugin
===============

The database plugin adds database storage using PostgreSQL (with PostGIS). It stores APRS 
position-updates (spatiotemporal data) and APRS packets for later analysis. It is configurable what callsigns 
are stored and for how long. Queries include movement trails, positions covered by digipeaters, etc. 
With this plugin you can go to a speficic time in history and generate a map-overlay showing the 
situation at that time (for data that is stored).
 
Client/user-owned data like trackers, static position objects, map-extents, map-layer setups, etc. can be stored
and queried through a REST API. For example, the drawing tool uses it to store features. 

The plugin also supports replication (between server-instances) with eventual consistency (CRDT) for 
some data objects.

Install
-------
The database plugin is available as a deb package. To install::

    apt install polaric-db-plugin

When the plugin is installed for the first time, the database components (PostgreSQL, PostGIS) needs to be installed and the schemas need to be set up. Run the script::

    polaric-dbsetup

Then restart the aprsd (polaric-restar) to activate it. 

Maintenance
-----------

When the plugin is upgraded, it may be necessary to upgrade the database schema. To do this, run the script::

    polaric-dbupgrade

It is possible to access the database from the psql shell. It can be useful for advanced users and developers to se more closely what is going on, perform various database queries and do maintenance tasks. 
To start the database shell::

    su - postgres
    psql polaric

The plugin does periodic maintenance tasks automatically, garbage collection, removing outdated data, etc.. 


Configuration
-------------

The plugin will put a config file database.ini in /etc/polaric-aprsd/config.d
Edit it to suit your needs. The default file contains explanations for the settings. 

