Database Plugin
===============

The database plugin adds database storage using PostgreSQL (with PostGIS extension). It stores APRS 
position-updates (spatiotemporal data), APRS packets and tag-settins for later queries and analysis. It is configurable 
what packets are stored and for how long. Queries include historic movement trails, positions covered by digipeaters, etc. 
With this plugin you can for example use the "*time machine*" to go to a specific time in history and generate a map-overlay showing the 
situation at that time (for data that is stored). 

The *webapp2* client automatically detect if the database plugin is used and adds some functions if it is.
 
Client/user-owned data like managed trackers, static position objects, map-extents, map-layer setups, etc. can be stored
and queried by client apps. For example, the drawing tool uses it to store features. The plugin provides a REST API which is `documented here <https://polaricserver.readthedocs.io/en/latest/restapi2.html>`_.

The plugin also supports replication (between server-instances) with eventual consistency (CRDT) for 
some data objects. This is rather experimental still though. 

Install
-------
The database plugin is available as a deb package. To install::

    apt install polaric-db-plugin

When the plugin is installed for the first time, the database components (PostgreSQL, PostGIS) needs to be installed and the schemas need to be set up. Run the script::

    polaric-dbsetup

Then restart the aprsd (polaric-restart) to activate it. 

Maintenance
-----------

When the plugin is upgraded, it may be necessary to upgrade the database schema. Each revision of the schema is given a revision-number and the upgrade script uses this to check if what is necessary to do, so it is a good practice to run this each time the plugin is updated. Run the script::

    polaric-dbupgrade

It is also possible to access the database from the psql shell. It can be useful for advanced users and developers to see more what is going on, perform various database queries and do maintenance work and fix various issues. To start the database shell::

    su - postgres
    psql polaric

The plugin does some periodic maintenance tasks automatically: This includes doing garbage collection, removing outdated data, vacuuming, etc.. The database plugin has its own log file::

    tail -f /var/log/polaric/database.log


Configuration
-------------

The plugin will put a config file *'database.ini*' in *'/etc/polaric-aprsd/config.d'*
Edit it to suit your needs. The default file contains explanations for the settings. Here you can: 

* Define what APRS updates are saved in the database (based on channel and/or source-callsign
* Define how long this data is stored. 
* It is also possible to set shorter lifetimes for some packets or reports (based on SQL subexpressions). 

Url and login for PostgreSQL database is defined here but you will probably not need to change that.

