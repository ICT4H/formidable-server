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

API
---

NB: The application does not yet conform to the API specification.

All resources *MUST* be valid JSON.

Clients *MUST* ignore any additional keys that they do not know how to interpret.

###Fact

A fact represents an update in the system. Facts are relative to a specific time and are identified with their own UUID.

    {
        'id': UUID,
        'time': EPOCH,
        'origin': URN?,
        'author': URN?,
        'assert':
        {
            UUID: TREE+,
        }?,
        'retract': [UUID+]?
    }

A UUID is a [universally unique identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier), expressed as a string.
An EPOCH is a [POSIX timestamp](https://en.wikipedia.org/wiki/Unix_time).
A TREE is a map, with string keys and ATOMs or TREEs as values.
An ATOM is a string, an integer, a floating point number or a vector of ATOMs.

For example:

    {
        'id': '3bde8815-5057-4a01-9c54-5b1d5d59bbd4'
        'time': 1390750744,
        'origin': 'urn:android:34F5D5C87AC1E5AF',
        'author': 'urn:email:cgrant@example.com',
        'assert':
        {
            '5b2a5594-5edb-4527-90db-a13bc5971e5d':
            {
                'name': 'Moses Musoke',
                'age': 32
            }
        }
    }

TODO: Find appropriate URN schemes for identifying Android devices and system users.

###Entity

An entity is a record produced by combining multiple facts. 

    {
        'id': UUID,
        'facts': [LINK+],
        'state': TREE
    }

A LINK is a map with `id` and `uri` attributes.

For example:

    {
        'id': '0ecc5a29-7ae5-4d50-a73e-e10c5e2232fc',
        'facts': [{'id': '5b2a5594-5edb-4527-90db-a13bc5971e5d',
                   'uri': 'http://example.org/fact/5b2a5594-5edb-4527-90db-a13bc5971e5d'},
                  {'id': 'b3053f2b-873b-46a3-87de-481a36a72cbd',
                   'uri': 'http://example.org/fact/b3053f2b-873b-46a3-87de-481a36a72cbd'}],
        'state':
        {
                'name': 'Moses Musoke',
                'age': 34,
                'blood pressure': {'systolic': 130, 'diastolic': 85} 
        }
    }

The state of the entity *MUST* be derived via the following algorithm:
  * Order all facts pertaining to this entity by time.
  * Remove any facts that appear in any retraction list.
  * Recursively merge the remaining facts, giving precedence to the more recently asserted facts.

###Changes

The changes feed is a list of facts. They do not have to be ordered by time, but the order in which facts appear *MUST* be stable.

The feed is paginated, and the boundries between pages *MUST* remain stable.

All but the most recent page *MUST* have a next URI and all but the oldest page *MUST* have a prev URI.

    {
        'next': URI?,
        'prev': URI?,
        'facts': [LINK]
    }

For example:

    {
        'prev': 'http://example.org/changes/6af5aba4-4ead-4db7-9846-e7fab147735c',
        'facts': [{'id': '5b2a5594-5edb-4527-90db-a13bc5971e5d',
                   'uri': 'http://example.org/fact/5b2a5594-5edb-4527-90db-a13bc5971e5d'},
                  {'id': 'b3053f2b-873b-46a3-87de-481a36a72cbd',
                   'uri': 'http://example.org/fact/b3053f2b-873b-46a3-87de-481a36a72cbd'}]
    }
