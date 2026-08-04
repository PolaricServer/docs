Database Plugin
---------------

The database plugin makes use of the PostgreSQL database system (with PostGIS) to extend the functionality. 

Install
# apt install polaric-db-plugin

Install database system components, create tables, etc. Do this the first time the plugin is installed.  
run the script: 
# polaric-dbsetup
# polaric-restart


Upgrade the database schema. Do this when the plugin is upgraded. 
Run the script: 
# polaric-dbupgrade
# polaric-restart


It is possible to access the database from the psql shell. It can be useful for advanced users and developers to se more closely what is going on, perform various database queries and do maintenance tasks. 
To start the database shell: 

# su - postgres
# psql polaric


The plugin does periodic maintenance tasks automatically, garbage collection, removing outdated data, etc.. 


Configuration

The plugin will put a config file database.ini in /etc/polaric-aprsd/config.d
Edit it to suit your needs. The default file contains explanations for the settings. 

