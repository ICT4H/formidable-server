Formidable
==========

[Github](https://github.com/ICT4H-TW/formidable-server)

Prerequisites
------------

* Vagrant

Installation
------------

* vagrant up
* vagrant reload
* ./createdb

Reload is needed because of [a bug in restarting couchdb](https://bugs.launchpad.net/ubuntu/+source/couchdb/+bug/448682) when it's first installed. 

This will install a headless 12.04 Ubuntu VM and start CouchDB, [port mapped to the local machine](http://localhost:5984/_utils/). 
