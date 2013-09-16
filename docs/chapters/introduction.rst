.. 
   ****************************************************************************
    pgRouting Manual
    Copyright(c) pgRouting Contributors

    This documentation is licensed under a Creative Commons Attribution-Share  
    Alike 3.0 License: http://creativecommons.org/licenses/by-sa/3.0/
   ****************************************************************************

.. _introduction:

Introduction
===============================================================================

.. rubric:: Abstract

`pgRouting <http://www.pgrouting.org>`_ adds routing functionality to `PostGIS <http://www.postgis.org>`_. This introductory workshop will show you how. 
It gives a practical example of how to use the new pgRouting release with `OpenStreetMap <http://www.openstreetmap.org>`_ road network data. 
It explains the steps to prepare the data, make routing queries, assign costs, write a custom function 'plpgsql' function and use the new `OpenLayers 3 <http://ol3js.org>`_ to show your route in a web-mapping application. 

Navigation for road networks requires complex routing algorithms that support turn restrictions and even time-dependent attributes. pgRouting is an extendable open-source library that provides a variety of tools for shortest path search as extension of PostgreSQL and PostGIS. The workshop will explain about shortest path search with pgRouting in real road networks and how the data structure is important to get faster results. Also you will learn about difficulties and limitations of pgRouting in GIS applications. 

To give a practical example the workshop makes use of OpenStreetMap data of Nottingham. You will learn how to convert the data into the required format and how to calibrate the data with “cost” attributes. Furthermore we will tell you what else pgRouting provides beside “Dijkstra”, “A-Star” and “Shooting-Star” and what has been added recently to the library. By the end of the workshop you will have a good understanding of how to use pgRouting and how to get your network data prepared.

To learn how to get the output from rows and columns to be drawn on a map, we will build a basic map GUI with OpenLayers 3. We listened to the students feedback of the last year’s and want to guide you through the basic steps to build a simple browser application. Our goal is to make this as easy as possible, and to show that it’s not difficult to integrate with other FOSS4G tools. Writing a custom PostgreSQL stored procedure in 'plpgsql' will allow us to make shortest path queries through Geoserver in a convenient way.


.. rubric:: Prerequisites

* Workshop level: intermediate
* Attendee's previous knowledge: SQL (PostgreSQL, PostGIS), Javascript, HTML
* Equipments: This workshops will make use of the OSGeo-Live DVD if possible. Otherwise it will require VirtualBox installed to load a virtual machine image.


.. rubric:: Presenters and Authors

* *Hal Seki* is founder and CEO of `Georepublic Japan <http://georepublic.info>`_. He is founding member of Code for Japan, organizer of Geomedia Summit and various Hackathons as well as other technology and OpenStreetMap events in Japan.

* *Yves Jacolin* works at the French office of `Camptocamp <http://www.camptocamp.com>`_. He is head of training and support at Camptocamp, and he is president of the Francophone local OSGeo chapter.

* *Daniel Kastl* is founder and CEO of `Georepublic <http://georepublic.info>`_ and works in Germany and Japan. He is moderating and promoting the pgRouting community and development since the beginning of the project, and he's an active OSM contributor in Japan.

* *Frédéric Junod* works at the Swiss office of `Camptocamp <http://www.camptocamp.com>`_ for about six years. He's an active developer of many open source GIS projects from the browser (GeoExt, OpenLayers) to the server world (MapFish, Shapely, TileCache) and he is member of the pgRouting PSC.

..
	Daniel and Frédéric are the authors of the previous pgRouting workshops, that have been held at previous FOSS4G events local conferences in Japan.


.. rubric:: License

This work is licensed under a `Creative Commons Attribution-Share Alike 3.0 License <http://creativecommons.org/licenses/by-sa/3.0/>`_.

.. image:: images/license.png


.. rubric:: Supported by

.. image:: images/camptocamp.png
	:alt: Camptocamp

`Camptocamp <http://www.camptocamp.com>`_

.. image:: images/georepublic.png
	:alt: Georepublic
	
`Georepublic <http://georepublic.info>`_


