 
REST API for clients
====================

This document summarises the RESTful API to the Polaric-aprsd backend server. This is used by clients running in standard web-browsers. By default it accepts HTTP requests on port 8081. It is also possible to configure a frontend webserver to act as a proxy for this, so that all HTTP requests to the server-side can go through the standard HTTP port (80). Servers where this is done do therefore not normally need to expose port 8081 externally. A cookie-based session scheme is used for logins. This means that if a user logs on a server a session is established that will used for subsequent REST requests. A session lasts until logging out or the server is restarted. For security I recommend to always use HTTPS if clients and servers are on different LANs. Cross-site requests are possible within certain limits. CORS is supported.

The URL identifies the resource (or service) to be invoked. For REST APIs, the URL identifies the resource and the operation to be performed on it is determined by the HTTP method: GET (read object), PUT (update), POST (add, post), DELETE (remove it). Where a representation of the state is required or returned, the JSON format is used. Some operations that perform search expect query parameters.

Most services will require authorization: O=open, L=login, S=SAR, A=admin.



System services
---------------
Source: `AdminApi.java` and `ShellScriptApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/authStatus`           | GET   |O| Get authorization info. Returns AuthInfo object      |
+------------------------+-------+-+------------------------------------------------------+
|`/system/tags`          | GET   |O| Get tags used                                        |
+------------------------+-------+-+------------------------------------------------------+
|`/system/ownpos`        | GET   |A| Get position of this server (symbol, latlong pos)    |
|                        +-------+-+------------------------------------------------------+
|                        | PUT   |A| Update position of this server (symbol, latlong pos) |
+------------------------+-------+-+------------------------------------------------------+
|`/system/sarmode`       | GET   |S| SAR mode info. Deprecated                            |
|                        +-------+-+------------------------------------------------------+
|                        | PUT   |S| Update SAR mode info. Deprecated                     |
+------------------------+-------+-+------------------------------------------------------+
|`/system/icons/{dir}`   | GET   |O| List of icons available in a subdirectory            |
+------------------------+-------+-+------------------------------------------------------+
|`/scripts`              | GET   |A| Get list of available commands/scripts               |
+------------------------+-------+-+------------------------------------------------------+
|`/scripts/{script-id}`  | POST  |A| Execute a script/command                             |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /authStatus

   Returns AuthInfo object with authorization info about current session 
   
   :status 200: Ok
   :>json string userid: Userid if logged in
   :>json string groupid: Current role (group)
   :>json string callsign: HAM radio callsign (can be null)
   :>json string servercall: Identfier (callsign) of the server
   :>json boolean admin: True if admin privileges
   :>json boolean sar: True if SAR privileges (derived from role)
   :>json string tagsAuth: What tags that authorize you (regular expression)
   :>json string[] services: List of services of server instance 


.. http:get:: /system/tags

   Get list of all tags used on the system
    
   :status 200: Ok
   :>jsonarr string tag: Tag used
   
   
.. http:get:: /system/ownpos

   Get position of this server (symbol, latlong pos) 
    
   :status 200: Ok
   :>json string sym: Symbol
   :>json string symtab: Symbol table
   :>json double[] pos: Position (longitude, latitude);
   

.. http:put:: /system/ownpos

   Aet position of this server (symbol, latlong pos) 
    
   :status 200: Ok
   :status 400: Couldn't parse input
   :>json string sym: Symbol
   :>json string symtab: Symbol table
   :>json double[] pos: Position (longitude, latitude);
   
   
.. http:get:: /system/sarmode

   Deprecated - do not use
   
   
.. http:put:: /system/sarmode

   Deprecated - do not use
   
   
.. http:get:: /system/icons/(dir)

   List of icons available in a subdirectory 
   
   :parameter userid: Unique indentifier of user
   :status 200: Ok 
   :status 500: Invalid file subdirectory for icons

   :>jsonarr string icon: Filename of icon
   
   
.. http:get:: /system/scripts

   Get list of available commands/scripts    
    
   :status 200: Ok
   :>jsonarr string name: Name (id) of script/command
   :>jsonarr string descr: Description
   

   
.. http:get:: /system/scripts/(script-id)

   Execute a command/scripts    

   :parameter script-id: Script identifier (name)
   :status 200: Ok 
   :status 400: Couldn't parse input
   :status 404: Script xxx not found
   :status 400: Script xxx: Missing arguments
   :status 400: Script xxx: Expected m arguments, got n
   :status 500: Script xxx: Exceeded max time. Killed!
   :status 500: Script xxx: (error-message from script)
   
   :>json string[] args: Arguments to be passed to the script

   
   
   
   
Users and clients
-----------------
Source: `UserApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/filters`              | GET   |L| Get filters available for you                        |
+------------------------+-------+-+------------------------------------------------------+
|`/mypasswd`             | PUT   |L| Change your own password                             |
+------------------------+-------+-+------------------------------------------------------+
|`/wsclients`            | GET   |A| Get (websocket) clients                              |
+------------------------+-------+-+------------------------------------------------------+
|`/loginusers`           | GET   |A| Get logged in users (list of userids)                |
+------------------------+-------+-+------------------------------------------------------+
|`/groups`               | GET   |A| Get available groups (roles)                         |
+------------------------+-------+-+------------------------------------------------------+
|`/mygroup`              | PUT   |L| Change your own group/role                           |
+------------------------+-------+-+------------------------------------------------------+
|`/usernames`            | GET   |L| Get list of all users (userids only)                 |
+------------------------+-------+-+------------------------------------------------------+
|`/users`                | GET   |A| Get list of all users                                |
|                        +-------+-+------------------------------------------------------+
|                        | POST  |A| Add a user                                           |
+------------------------+-------+-+------------------------------------------------------+
|`/users/{id}`           | GET   |A| Get info about a given user                          |
|                        +-------+-+------------------------------------------------------+
|                        | PUT   |A| Update a user                                        |
|                        +-------+-+------------------------------------------------------+
|                        | DELETE|A| Remove a user                                        |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /filters

   Returns a list of filters available for you as a logged in user. 
   
   :status 200: Ok
   :>jsonarr string[] filter: Pair - filter name, description


.. http:get:: /mypasswd

   Change your own password
   
   :status 200: Ok
   :status 404: Unknown user
   :<json string passwd: New password
   
   
.. http:get:: /wsclients

   Get currently active clients to (the websocket interface)
   
   :status 200: Ok
   :>jsonarr string uid: User id (IP-address:port)
   :>jsonarr string username: Login name (if logged in)
   :>jsonarr Date created: Time when client connection was created


.. http:get:: /wsclients

   Get currently logged in users
   
   :status 200: Ok
   :>jsonarr string uid: Userid (login name)
   
   
.. http:get:: /groups

   Get groups (roles) available for logged in user
   
   :status 200: Ok
   :>jsonarr string gid: Group id
   

.. http:get:: /mygroup

   Change your own group/role (for the session)
   
   :status 200: Ok
   :status 404: Unknown user
   :status 404: Unknown group
   :status 403: Group is not allowed
   
   :>jsonarr string group: Group id
   

.. http:get:: /usernames

   Return a list of usernames (userids only)
   
   :status 200: Ok
   :>jsonarr string uid: User id
   
   
.. http:get:: /users

   Return a list of users of the system
   
   :status 200: Ok
      
   :>jsonarr string ident: User id
   :>jsonarr string name: Name
   :>jsonarr string callsign: HAM radio callsign (can be null)
   :>jsonarr Date lastused: Time when last logged in
   :>jsonarr string group: Primary group (role)
   :>jsonarr string altgroup: Secondary group (role)
   :>jsonarr boolean sar: True if SAR user (derived from primary group)
   :>jsonarr boolean admin: True if admin user
   :>jsonarr boolean suspend: True if suspended
   :>jsonarr string passwd: Always null
   
   
.. http:post:: /users

   Add a new user to the system
   
   :status 200: Ok
   :status 400: Probable cause: User already exists
      
   :<jsonarr string ident: User id
   :<jsonarr string name: Name
   :<jsonarr string callsign: HAM radio callsign (can be null)
   :<jsonarr Date lastused: Time when last logged in
   :<jsonarr string group: Primary group (role)
   :<jsonarr string altgroup: Secondary group (role)
   :<jsonarr boolean sar: True if SAR user (derived from primary group)
   :<jsonarr boolean admin: True if admin user
   :<jsonarr boolean suspend: True if suspended
   :<jsonarr string passwd: Password for login
   

   
.. http:get:: /users/(userid)

   Return info about a particular user

   :parameter userid: Unique indentifier of user
   :status 200: Ok
   :status 404: Unknown user
      
   :>jsonarr string ident: User id
   :>jsonarr string name: Name
   :>jsonarr string callsign: HAM radio callsign (can be null)
   :>jsonarr Date lastused: Time when last logged in
   :>jsonarr string group: Primary group (role)
   :>jsonarr string altgroup: Secondary group (role)
   :>jsonarr boolean sar: True if SAR user (derived from primary group)
   :>jsonarr boolean admin: True if admin user
   :>jsonarr boolean suspend: True if suspended
   :>jsonarr string passwd: Always null


.. http:put:: /users/(userid)

   Update a user
    
   :parameter userid: Unique indentifier of user
      
   :status 200: Ok
   :status 404: Unknown user
   :status 400: Cannot parse input
   :status 404: Unknown group
   :status 404: Unknown alt group
        
   :<jsonarr string ident: User id
   :<jsonarr string name: Name
   :<jsonarr string callsign: HAM radio callsign (can be null)
   :<jsonarr Date lastused: Time when last logged in
   :<jsonarr string group: Primary group (role)
   :<jsonarr string altgroup: Secondary group (role)
   :<jsonarr boolean sar: True if SAR user (derived from primary group)
   :<jsonarr boolean admin: True if admin user
   :<jsonarr boolean suspend: True if suspended
   :<jsonarr string passwd: Password for login
   
   
.. http:delete:: /users/(userid)

   Remove a user
    
   :parameter userid: Unique indentifier of user
   :status 200: Ok
   
   
   
Items (tracker objects)
-----------------------
Source: `ItemApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/info`       | GET   |O| Get info about a tracker                             |
+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/trail`      | GET   |O| Get trail of moving tracker. List of points.         |
+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/reset`      | PUT   |S| Reset trail and other info about item                |
+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/chcolor`    | PUT   |S| Change color of trail.                               |
+------------------------+-------+-+------------------------------------------------------+
|`/item{id}/tags`        | GET   |O| Get list of tags set on an item                      |
|                        +-------+-+------------------------------------------------------+
|                        | POST  |S| Add a tag to an item                                 |
+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/tags/{tag}` | DELETE|S| Remove a tag from an item                            |
+------------------------+-------+-+------------------------------------------------------+
|`/items`                | GET   |O| Search items (query parameters)                      |
+------------------------+-------+-+------------------------------------------------------+
|`/item/{id}/alias`      | GET   |S| Alias for tracker with id (callsign).                |
|                        +-------+-+------------------------------------------------------+
|                        | PUT   |S| Set alias for tracker with id (callsign).            |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /item/(id)/info

   Returns info about a item (tracker). The full set of returned attributes depend on the type of item. Here we show what all items will have. 
   
   :parameter id: Identifier of tracker item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item
   :status 500: Error
      
   :>json string type: Type of item 
   :>json string ident: Ident of item (callsign, etc)
   :>json descr: Description text 
   :>json string source: Name of source
   :>json double pos[]: Position of item (lon, lat)
   
   
   
.. http:get:: /item/(id)/trail

   Returns a trail of moving tracker. List of points. 
   
   :parameter id: Identifier of tracker item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   
   :>jsonarr Date time: Time of point 
   :>jsonarr int speed: Speed of tracker at point (km/h)
   :>jsonarr int course: Course of tracker at point (0-360 degrees)
   :>jsonarr int dist: Distance from previous point (meters)
   :>jsonarr string path: Digipeating path of APRS report

   
         
.. http:put:: /item/(id)/reset

   Reset trail and other info about item
   
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item

   
   
.. http:put:: /item/(id)/chcolor

   Change colour of trail
   
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item
   

   
.. http:get:: /item/(id)/tags

   Returns list of tags set on the item
   
   :parameter id: Identifier of tracker item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
      
   :>jsonarr string tag: Tag 

   
.. http:post:: /item/(id)/tags

   Add tags to the item
   
   :parameter id: Identifier of tracker item (callsign)
   :parameter tag: The tag to be added
      
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item
   :<jsonarr string tag: Tag to be added 
   
   
   
.. http:delete:: /item/(id)/tags/(tag)
   
   Remove a tag from the item

   :parameter id: Identifier of tracker item (callsign)
   :parameter tag: The tag to be removed
   
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item
   
   
   
.. http:get:: /items

   Search items. Takes two query parameters: A search expression and a list of tags. The search return 
   items where the search expression matches callsign, name or description and where tags matches. 
   
   :form src: Search expression for matching ident/callsign and comment
   :form tags: Comma separated list of tags
   
   :status 200: Ok
   :>jsonarr string ident: Ident or callsign for the item
   :>jsonarr name: Name of the item
   :>jsonarr alias: Alias of the item if set
   :>jsonarr double pos[]: Position of item (lon, lat)
   :>jsonarr Date updated: Time when item was last updated
   :>jsonarr string descr: Description 
   :>jsonarr int speed: Moving speed (km/h)
   :>jsonarr int course: Moving course (degrees)
   
   
.. http:get:: /item/(id)/alias

   Returns alias (and alternative icon) set on the item
   
   :parameter id: Identifier of tracker item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item  
         
   :>json string alias: Alias (null if not set) 
   :>json string icon: Alternative icon (null if not set) 
   
   
.. http:put:: /item/(id)/alias

   Set alias (and alternative icon) on the item
   
   :parameter id: Identifier of tracker item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 401: Unauthorized for access to item  
   :status 401: Alias can only be set by owner
   :status 400: Cannot parse input
         
   :<json string alias: Alias (null if not set) 
   :<json string icon: Alternative icon (null if not set) 
   
   
   
   
APRS Telemetry
--------------
Source: `ItemApi.java`

+-------------------------+-------+-+------------------------------------------------------+
|`/telemetry/{id}/descr`  | GET   |O| Get telemetry description for a given item           |
+-------------------------+-------+-+------------------------------------------------------+
|`/telemetry/{id}/meta`   | GET   |O| Get telemetry metadata for a given item              |
+-------------------------+-------+-+------------------------------------------------------+
|`/telemetry/{id}/current`| GET   |O| Get telemetry current report for a given item        |
+-------------------------+-------+-+------------------------------------------------------+
|`/telemetry/{id}/history`| GET   |O| Get telemetry history report for a given item        |
+-------------------------+-------+-+------------------------------------------------------+

.. http:get:: /telemetry/(id)/descr

   Returns a description for the given item 
   
   :parameter id: Identifier of item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 404: No telemetry found
   :status 404: Telemetry is invalid
   
   :>json string descr: Description text
   
   
.. http:get:: /telemetry/(id)/meta

   Returns telemetry metadata for the given item 
   
   :parameter id: Identifier of item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 404: No telemetry found
   :status 404: Telemetry is invalid
   
   :>json NumChannelMeta[] num: Metadata for numerical channels (string param, string unit, float[] eqns)
   :>json BinChannelMeta[] bin: Metadata for binary channels (string param, string unit, boolean bit, boolean use)
   
   
.. http:get:: /telemetry/(id)/current

   Returns current telemetry data (last reported) for the given item 
     
   :parameter id: Identifier of item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 404: No telemetry found
   :status 404: Telemetry is invalid
   
   :>json int seq: sequence number?
   :>json Date time: Reported time 
   :>json float[] num: Numeric data values
   :>json boolean[] bin: Binary data values
   

.. http:get:: /telemetry/(id)/history

   Returns list of telemetry data reported earlier for the given item 

   :parameter id: Identifier of item (callsign)
   :status 200: Ok
   :status 404: Unknown tracker item
   :status 404: No telemetry found
   :status 404: Telemetry is invalid
   
   :>jsonarr int seq: sequence number?
   :>jsonarr Date time: Reported time 
   :>jsonarr float[] num: Numeric data values
   :>jsonarr boolean[] bin: Binary data values
   
   

Own APRS objects 
----------------
Source: `AprsObjectApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/aprs/objects`         | GET   |S| Get the list of active objects (owned by this server)|
|                        +-------+-+------------------------------------------------------+
|                        | POST  |S| Add an object                                        |
+------------------------+-------+-+------------------------------------------------------+
|`/aprs/objects/{id}`    | PUT   |S| Update an object                                     |
|                        +-------+-+------------------------------------------------------+
|                        | DELETE|S| Remove an object                                     |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /aprs/objects

   Returns a list of active APRS objects owned by this server. 
   
   :status 200: Ok
   :>jsonarr string ident: Identifier (callsign) of APRS objects
   

.. http:post:: /aprs/objects

   Add an APRS object to this server 
   
   :status 200: Ok
   :status 400: Object already exists
   :status 400: Invalid object. Couldn't post
   :status 500: Couldn't post object
   
   :<json string ident: Identifier (callsign) of APRS object
   :<json double[] pos: Position of APRS object (longitude, latitude)
   :<json char sym: Symbol 
   :<json char symtab: Symbol table 
   :<json string comment: Comment text
   :<json boolean perm: True if permanent;
   


.. http:put:: /aprs/objects

    Update an APRS object on this server 
   
   :status 200: Ok
   :status 400: Object not found 
      
   :<json double[] pos: Position of APRS object (longitude, latitude)
   :<json char sym: Symbol 
   :<json char symtab: Symbol table 
   :<json string comment: Comment text
   
   
.. http:delete:: /aprs/objects/(id)

    Delete an APRS object from this server 
    
   :parameter id: Identifier of item (callsign)
   :status 200: Ok   
   :status 400: Object not found 
   :status 500: Couldn't delete object
   

   
Short messages
--------------
Source: `MailBoxApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/mailbox`              | GET   |L| Get content of mailbox (list of messages)            |
|                        +-------+-+------------------------------------------------------+
|                        | POST  |L| Post a message                                       |
+------------------------+-------+-+------------------------------------------------------+
|`/mailbox/{msg-id}`     | DELETE|L| Delete a message from mailbox                        |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /mailbox

   Returns the content of the user's mailbox (list of messsages)
   
   :status 200: Ok
   :status 401: No mailbox available
   
   :>jsonarr long msgId: Unique identifier of message;
   :>jsonarr int  status: Delievery status -1=failure, 1=success  
   :>jsonarr Date time: Time of posting
   :>jsonarr string from: Sender (userid@node or callsign@APRS)  
   :>jsonarr string to: Recipient (userid@node or callsign@APRS)
   :>jsonarr boolean read: True if messsage is read by recipient
   :>jsonarr boolean outgoing: True if outgoing
   :>jsonarr string text: Content of messsage
        

.. http:post:: /mailbox

   Post a message to another user (or to APRS)
   
   :status 200: Ok
   :status 400: Cannot parse input
   :status 404: Unknown from-address
   :status 404: Callsign is needed for raw APRS messages
   :status 404: Couldn't deliver message
   
   :<json long msgId: Unique identifier of message;
   :<json int  status: ignored  
   :<json Date time: Time of posting or null to use time now
   :<json string from: Should be null or match user that is posting
   :<json string to: Recipient (userid@node or callsign@APRS)
   :<json boolean read: ignored
   :<json boolean outgoing: ignored
   :<json string text: Content of messsage

      

.. http:delete:: /mailbox/(msgid)
   
   Delete an IPP
   
   :parameter msgid: Unique indentifier of message

   :status 200: Ok
   :status 400: Message id must be a number
   
   
   
   
APRS Bulletins
--------------
Source: `BullBoardApi.java`

+-----------------------------------------+-----+-+-------------------------------------------------+
|`/bullboard/groups`                      | GET |O| List of active bulletin groups                  |
+-----------------------------------------+-----+-+-------------------------------------------------+
|`/bullboard/{groupid}/senders`           | GET |O| List of callsigns of senders to a given group   |
+-----------------------------------------+-----+-+-------------------------------------------------+
|`/bullboard/{groupid}/messages`          | GET |O| Get all messages in a group                     |
|                                         +-----+-+-------------------------------------------------+
|                                         | POST|L| Submit a bulletin                               |
+-----------------------------------------+-----+-+-------------------------------------------------+
|`/bullboard/{groupid}/messages/{sender}` | GET |O| Bulletins from a given sender in a group        |
+-----------------------------------------+-----+-+-------------------------------------------------+

.. http:get:: /bullboard/groups

   Returns a list of active bulleti groups
   
   :status 200: Ok
   :>jsonarr string id: Group id 

   
.. http:get:: /bullboard/(groupid)/senders

   Returns a list of callsigns that have posted messages to the group
   
   :parameter groupid: Unique indentifier of group
   :status 200: Ok
   :status 404: Group not found.
   
   :>jsonarr string id: Callsign of sender 


.. http:get:: /bullboard/(groupid)/messages

   Returns all messages in a group. Note that this returns a list of lists
    
   :parameter groupid: Unique indentifier of group
   :status 200: Ok
   :status 404: Group not found.
   
   :>jsonarr string id: Callsign of sender 
   :>jsonarr Bulletin[]: List of bulletins (several attributes)
   
   
.. http:post:: /bullboard/(groupid)/messages

   Post a bulletin to a group
   
   :parameter groupid: Unique indentifier of group
   :status 200: Ok  
   :status 401: No callsign registered for user.
   :status 400: Cannot parse input
   
   :<json string bullid: Index for bulletin
   :<json string groupid: Group identifier
   :<json string text: Text of bulletin
   

.. http:get:: /bullboard/(groupid)/messages/(sender)

   Returns messages in a group posted by a specific sender. Note that this returns a list of lists
   
   :parameter groupid: Unique indentifier of group
   :parameter sender: Callsign of sender
   
   :status 200: Ok
   :status 404: Group not found.
   
   :>jsonarr string id: Callsign of sender 
   :>jsonarr Bulletin[]: List of bulletins (several attributes)

   
   
SAR (Search and Rescue)
-----------------------
Source: `SarApi.java`

+------------------------+-------+-+------------------------------------------------------+
|`/sar/ipp`              | GET   |L| Get a list of IPPs (with distance rings) for user    |
|                        +-------+-+------------------------------------------------------+
|                        | POST  |L| Add a IPP (with distance rings)                      |
+------------------------+-------+-+------------------------------------------------------+
|`/sar/ipp/{id}`         | GET   |L| Get a specific IPP                                   |
|                        +-------+-+------------------------------------------------------+
|                        | PUT   |L| Update a specific IPP                                |
|                        +-------+-+------------------------------------------------------+
|                        | DELETE|L| Remove a IPP                                         |
+------------------------+-------+-+------------------------------------------------------+

.. http:get:: /sar/ipp

   Returns a list of IPPs for the given user.
   
   :status 200: Ok
   :status 500: No authorization info found.
   
   :>jsonarr string id: Unique identifier 
   :>jsonarr string descr: Descripttion of IPP
   :>jsonarr double[] pos: Position of IPP (longitude, latitude)
   :>jsonarr float p25: Radius (meters) of 25% distance ring 
   :>jsonarr float p50: Radius (meters) of 50% distance ring 
   :>jsonarr float p75: Radius (meters) of 75% distance ring 
   :>jsonarr float p95: Radius (meters) of 95% distance ring 
   
   
.. http:post:: /sar/ipp

   Add a IPP

   :status 200: Ok  
   :status 500: No authorization info found.
   :status 400: Cannot parse input
   
   :<json string id: Unique identifier for ipp
   :<json string descr: Description of IPP
   :<json double[] pos: Position of IPP (longitude, latitude)
   :<json float p25: Radius (meters) of 25% distance ring 
   :<json float p50: Radius (meters) of 50% distance ring    
   :<json float p75: Radius (meters) of 75% distance ring 
   :<json float p95: Radius (meters) of 95% distance ring 

   
.. http:get:: /sar/ipp/(id)

   Returns a specific IPP
   
   :parameter id: Unique indentifier of IPP
   
   :status 200: Ok
   :status 500: No authorization info found.
   :status 404: Not found.
   
   :>json string id: Unique identifier 
   :>json string descr: Descripttion of IPP
   :>json double[] pos: Position of IPP (longitude, latitude)
   :>json float p25: Radius (meters) of 25% distance ring 
   :>json float p50: Radius (meters) of 50% distance ring 
   :>json float p75: Radius (meters) of 75% distance ring 
   :>json float p95: Radius (meters) of 95% distance ring 
   
   
      
.. http:put:: /sar/ipp/(id)

   Update a specific IPP
   
   :parameter id: Unique indentifier of IPP
   
   :status 200: Ok
   :status 500: No authorization info found. 
   :status 400: Cannot parse input
   :status 404: Not found.
   
   :<json string id: Unique identifier 
   :<json string descr: Descripttion of IPP
   :<json double[] pos: Position of IPP (longitude, latitude)
   :<json float p25: Radius (meters) of 25% distance ring 
   :<json float p50: Radius (meters) of 50% distance ring 
   :<json float p75: Radius (meters) of 75% distance ring 
   :<json float p95: Radius (meters) of 95% distance ring 
   
   

.. http:delete:: /sar/ipp/(id)
   
   Delete an IPP
   
   :parameter id: Unique indentifier of IPP

   :status 200: Ok
   :status 500: No authorization info found.

