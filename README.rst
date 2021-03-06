===========================
Frontera-SeedLoader-MongoDB
===========================

.. image:: https://img.shields.io/pypi/v/frontera-seedloader-mongodb.svg
   :target: https://pypi.python.org/pypi/frontera-seedloader-mongodb
   :alt: PyPI Version

.. image:: https://img.shields.io/travis/grammy-jiang/frontera-seedloader-mongodb/master.svg
   :target: http://travis-ci.org/grammy-jiang/frontera-seedloader-mongodb
   :alt: Build Status

.. image:: https://img.shields.io/badge/wheel-yes-brightgreen.svg
   :target: https://pypi.python.org/pypi/frontera-seedloader-mongodb
   :alt: Wheel Status

.. image:: https://img.shields.io/codecov/c/github/grammy-jiang/frontera-seedloader-mongodb/master.svg
   :target: http://codecov.io/github/grammy-jiang/frontera-seedloader-mongodb?branch=master
   :alt: Coverage report

Overview
========

Frontera is a great framework for broad crawling, especially working with
scrapy. This package provides a seed loader from MongoDB in a sync ways for
frontera:

* Querying seeds can be customized

Requirements
============

* pymongo

* Tests on Python 3.5

* Tests on Linux, but it's a pure python module, should work on other platforms
  with official python and Twisted supported

Installation
============

The quick way::

    pip install frontera-seedloader-mongodb

Or put this middleware just beside the scrapy project.

Documentation
=============

Set Seed Loader in ``SPIDER_MIDDLEWARES`` in ``settings.py``, for example::

    # -----------------------------------------------------------------------------
    # FRONTERA SEEDLOADER MONGODB ASYNC
    # -----------------------------------------------------------------------------

    SPIDER_MIDDLEWARES.update({
        'frontera_seedloader_mongodb.contrib.scrapy.middlewares.seeds.mongodb.MongoDBAsyncSeedLoader': 500,
    })

    SEEDS_MONGODB_USERNAME = 'user'
    SEEDS_MONGODB_PASSWORD = 'password'
    SEEDS_MONGODB_HOST = 'localhost'
    SEEDS_MONGODB_PORT = 27017
    SEEDS_MONGODB_DATABASE = 'test_mongodb_async_db'
    SEEDS_MONGODB_COLLECTION = 'test_mongodb_async_coll'

    # SEEDS_MONGODB_OPTIONS_ = 'SEEDS_MONGODB_OPTIONS_'

    SEEDS_MONGODB_SEEDS_QUERY = {
        'filter': {'websites': {'$exists': 1}}
    }
    SEEDS_MONGODB_SEEDS_BATCH_SIZE = 1000

    SEEDS_MONGODB_SEEDS_PREPARE = 'scrapy_project.utils.seeds_prepara'

Settings Reference
==================

SEEDS_MONGODB_USERNAME
----------------------

A string of the username of the database.

SEEDS_MONGODB_PASSWORD
----------------------

A string of the password of the database.

SEEDS_MONGODB_HOST
------------------

A string of the ip address or the domain of the database.

SEEDS_MONGODB_PORT
------------------

A int of the port of the database.

SEEDS_MONGODB_DATABASE
----------------------

A string of the name of the database.

SEEDS_MONGODB_COLLECTION
------------------------

A list of the indexes to create on the collection.

SEEDS_MONGODB_OPTIONS_*
-----------------------

Options can be attached when the seed loader start to connect to MongoBD.

If any options are needed, the option can be set with the prefix
``SEEDS_MONGODB_OPTIONS_``, the pipeline will parse it.

For example:

+---------------+-------------------------------------+
| option name   | in ``settings.py``                  |
+---------------+-------------------------------------+
| authMechanism | SEEDS_MONGODB_OPTIONS_authMechanism |
+---------------+-------------------------------------+

For more options, please refer to the page:

* `Connection String URI Format — MongoDB Manual 3.4`_

.. _`Connection String URI Format — MongoDB Manual 3.4`: https://docs.mongodb.com/manual/reference/connection-string/#connections-standard-connection-string-format


SEEDS_MONGODB_SEEDS_QUERY
-------------------------

A dictionary as the keyword arguments to query. The default value is::

   SEEDS_MONGODB_SEEDS_QUERY = {
       'filter': None,
       'projection': None,
       'skip': 0,
       'limit': 0,
       'no_cursor_timeout': False,
       'cursor_type': CursorType.NON_TAILABLE,
       'sort': None,
       'allow_partial_results': False,
       'oplog_replay': False,
       'modifiers': None,
       'manipulate': True
   }

The keys and values in this setting is followed the keyword arguments of the
method ``find`` of ``collection`` in ``pymongo``. Please refer to
the following documents for more details.

* `collection – Collection level operations — PyMongo 3.5.1 documentation`_

.. _`collection – Collection level operations — PyMongo 3.5.1 documentation`: http://api.mongodb.com/python/current/api/pymongo/collection.html#pymongo.collection.Collection.find

SEEDS_MONGODB_SEEDS_BATCH_SIZE
------------------------------

A int of The batch size that each query will return, the default value is 100.

SEEDS_MONGODB_SEEDS_PREPARE
---------------------------

A string of the module path to the function to process the task (seed) from
MongoDB.

The input will be the document queried from the collection set in
``SEEDS_MONGODB_COLLECTION``, and the output should be a iterator which will
return tuples with two elements: ``(url, doc)``. The ``url`` will be the
argument ``url`` of ``Request``, and the ``doc`` will be given to
``request.meta``.
As an example, the default function in this middleware::

    class MongoDBSeedLoader(SeedLoader):

        ...

        def open_spider(self, spider: Spider):
            try:
                if self.settings.get(SEEDS_MONGODB_SEEDS_PREPARE):
                    self.prepare = load_object(
                        self.settings.get(SEEDS_MONGODB_SEEDS_PREPARE))
                else:
                    self.prepare = lambda x: map(
                        lambda y: (y, {'seed': x}),
                        x['websites'])
            except:
                raise NotConfigured

            ...

        ...

NOTE
====

The database drivers may have different api for the same operation, this
seed loader adopts pymongo as the sync driver for MongoDB. If you want to
customize this seed loader, please read the following documents for more
details.

* `pymongo – Python driver for MongoDB`_

.. _`pymongo – Python driver for MongoDB`: http://api.mongodb.com/python/current/api/pymongo/

TODO
====

* add an async way to load the seed from MongoDB
* split the MongoDB to backend, make this seed load more flexible
