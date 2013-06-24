.. 
   ****************************************************************************
    pgRouting Manual
    Copyright(c) pgRouting Contributors

    This documentation is licensed under a Creative Commons Attribution-Share  
    Alike 3.0 License: http://creativecommons.org/licenses/by-sa/3.0/
   ****************************************************************************

.. _routing:

pgRouting Algorithms
===============================================================================

pgRouting was first called *pgDijkstra*, because it implemented only shortest path search with *Dijkstra* algorithm. Later other functions were added and the library was renamed.

.. image:: images/route.png
	:width: 250pt
	:align: center
	
This chapter will explain selected pgRouting algorithms and which attributes are required. 


.. note::

	If you run :doc:`osm2pgrouting <osm2pgrouting>` tool to import *OpenStreetMap* data, the ``ways`` table contains all attributes already to run all shortest path functions. The ``ways`` table of the ``pgrouting-workshop`` database of the :doc:`previous chapter <topology>` is missing several attributes instead, which are listed as **Prerequisites** in this chapter.


.. _dijkstra:

Shortest Path Dijkstra
-------------------------------------------------------------------------------

Dijkstra algorithm was the first algorithm implemented in pgRouting. It doesn't require other attributes than ``source`` and ``target`` ID, ``id`` attribute and ``cost``. It can distinguish between directed and undirected graphs. You can specify if your network has ``reverse cost`` or not.

.. rubric:: Prerequisites

To be able to use ``reverse cost`` you need to add an additional cost column. We can set reverse cost as length.

.. code-block:: sql

	ALTER TABLE ways ADD COLUMN reverse_cost double precision;
	UPDATE ways SET reverse_cost = length;

.. rubric:: Description

Returns a set of ``pgr_costResult`` (seq, id1, id2, cost) rows, that make up a path.

.. code-block:: sql

	pgr_costResult[] pgr_dijkstra(text sql, integer source, integer target, boolean directed, boolean has_rcost);


.. rubric:: Parameters

:sql: a SQL query, which should return a set of rows with the following columns:

	.. code-block:: sql

		SELECT id, source, target, cost [,reverse_cost] FROM edge_table


	:id: ``int4`` identifier of the edge
	:source: ``int4`` identifier of the source vertex
	:target: ``int4`` identifier of the target vertex
	:cost: ``float8`` value, of the edge traversal cost. A negative cost will prevent the edge from being inserted in the graph.
	:reverse_cost: (optional) the cost for the reverse traversal of the edge. This is only used when the ``directed`` and ``has_rcost`` parameters are ``true`` (see the above remark about negative costs).

:source: ``int4`` id of the start point
:target: ``int4`` id of the end point
:directed: ``true`` if the graph is directed
:has_rcost: if ``true``, the ``reverse_cost`` column of the SQL generated set of rows will be used for the cost of the traversal of the edge in the opposite direction.

Returns set of ``pgr_costResult``:

:seq:   row sequence
:id1:   node ID
:id2:   edge ID (``-1`` for the last row)
:cost:  cost to traverse from ``id1`` using ``id2``


.. rubric:: Example query

.. code-block:: sql

	SELECT seq, id1 AS node, id2 AS edge, cost FROM pgr_dijkstra('
			SELECT gid AS id, 
				 source::integer, 
				 target::integer, 
				 length::double precision AS cost 
				FROM ways', 
			10, 60, false, false); 


.. rubric:: Query result

.. code-block:: sql

	 seq | node | edge |        cost         
	-----+------+------+---------------------
	   0 |   10 | 3163 |   0.427103399132954
	   1 | 1084 | 2098 |   0.441091435851107
	   2 |   35 |   27 |     0.1005403350897
	   3 |   34 | 1984 |   0.278250260547731
	...   
	  40 |   59 |   56 |  0.0452819891352444
	  41 |   60 |   -1 |                   0
	(42 rows)


.. _astar:

Shortest Path A*
-------------------------------------------------------------------------------

A-Star algorithm is another well-known routing algorithm. It adds geographical information to source and target of each network link. This enables the shortest path search to prefer links which are closer to the target of the search.

.. rubric:: Prerequisites

For A-Star you need to prepare your network table and add latitute/longitude columns (``x1``, ``y1`` and ``x2``, ``y2``) and calculate their values.

.. code-block:: sql

	ALTER TABLE ways ADD COLUMN x1 double precision;
	ALTER TABLE ways ADD COLUMN y1 double precision;
	ALTER TABLE ways ADD COLUMN x2 double precision;
	ALTER TABLE ways ADD COLUMN y2 double precision;
	
	UPDATE ways SET x1 = ST_x(ST_startpoint(the_geom));
	UPDATE ways SET y1 = ST_y(ST_startpoint(the_geom));
	
	UPDATE ways SET x2 = ST_x(ST_endpoint(the_geom));
	UPDATE ways SET y2 = ST_y(ST_endpoint(the_geom));
	
	UPDATE ways SET x1 = ST_x(ST_PointN(the_geom, 1));
	UPDATE ways SET y1 = ST_y(ST_PointN(the_geom, 1));
	
	UPDATE ways SET x2 = ST_x(ST_PointN(the_geom, ST_NumPoints(the_geom)));
	UPDATE ways SET y2 = ST_y(ST_PointN(the_geom, ST_NumPoints(the_geom)));


.. rubric:: Description

Shortest Path A-Star function is very similar to the Dijkstra function, though it prefers links that are close to the target of the search. The heuristics of this search are predefined, so you need to recompile pgRouting if you want to make changes to the heuristic function itself.

Returns a set of ``pgr_costResult`` (seq, id1, id2, cost) rows, that make up a path.

.. code-block:: sql

	pgr_costResult[] pgr_astar(sql text, source integer, target integer, directed boolean, has_rcost boolean);


.. rubric:: Parameters

:sql: a SQL query, which should return a set of rows with the following columns:

	.. code-block:: sql

		SELECT id, source, target, cost, x1, y1, x2, y2 [,reverse_cost] FROM edge_table


	:id: ``int4`` identifier of the edge
	:source: ``int4`` identifier of the source vertex
	:target: ``int4`` identifier of the target vertex
	:cost: ``float8`` value, of the edge traversal cost. A negative cost will prevent the edge from being inserted in the graph.
	:x1: ``x`` coordinate of the start point of the edge
	:y1: ``y`` coordinate of the start point of the edge
	:x2: ``x`` coordinate of the end point of the edge
	:y2: ``y`` coordinate of the end point of the edge
	:reverse_cost: (optional) the cost for the reverse traversal of the edge. This is only used when the ``directed`` and ``has_rcost`` parameters are ``true`` (see the above remark about negative costs).

:source: ``int4`` id of the start point
:target: ``int4`` id of the end point
:directed: ``true`` if the graph is directed
:has_rcost: if ``true``, the ``reverse_cost`` column of the SQL generated set of rows will be used for the cost of the traversal of the edge in the opposite direction.

Returns set of ``pgr_costResult``:

:seq:   row sequence
:id1:   node ID
:id2:   edge ID (``-1`` for the last row)
:cost:  cost to traverse from ``id1`` using ``id2``


.. rubric:: Example query

.. code-block:: sql

	SELECT seq, id1 AS node, id2 AS edge, cost FROM pgr_astar('
			SELECT gid AS id, 
				 source::integer, 
				 target::integer, 
				 length::double precision AS cost, 
				 x1, y1, x2, y2
				FROM ways', 
			10, 60, false, false); 
		

.. rubric:: Query result

.. code-block:: sql
		
	 seq | node | edge |        cost         
	-----+------+------+---------------------
	   0 |   10 | 3163 |   0.427103399132954
	   1 | 1084 | 2098 |   0.441091435851107
	   2 |   35 |   27 |     0.1005403350897
	   3 |   34 | 1984 |   0.278250260547731
	...
	  40 |   59 |   56 |  0.0452819891352444
	  41 |   60 |   -1 |                   0
	(42 rows)


.. _kdijkstra:

Multiple Shortest Paths with kDijkstra
-------------------------------------------------------------------------------

The kDijkstra functions are very similar to the Dijkstra function but they allow to set multiple destinations with a single function call.

.. rubric:: Prerequisites

kDijkstra doesn't require additional attributes to Dijkstra algorithm.


.. rubric:: Description

If the main goal is to calculate the total cost, for example to calculate multiple routes for a distance matrix, then ``pgr_kdijkstraCost`` returns a more compact result. In case the paths are important ``pgr_kdijkstraPath`` function returns a result similar to A* or Dijkstra for each destination.

Both functions return a set of ``pgr_costResult`` (seq, id1, id2, cost) rows, that summarize the path cost or return the paths.

.. code-block:: sql

    pgr_costResult[] pgr_kdijkstraCost(text sql, integer source,
                     integer[] targets, boolean directed, boolean has_rcost);

    pgr_costResult[] pgr_kdijkstraPath(text sql, integer source,
                     integer[] targets, boolean directed, boolean has_rcost);


.. rubric:: Parameters

:sql: a SQL query, which should return a set of rows with the following columns:

    .. code-block:: sql

        SELECT id, source, target, cost [,reverse_cost] FROM edge_table


    :id: ``int4`` identifier of the edge
    :source: ``int4`` identifier of the source vertex
    :target: ``int4`` identifier of the target vertex
    :cost: ``float8`` value, of the edge traversal cost. A negative cost will prevent the edge from being inserted in the graph.
    :reverse_cost: (optional) the cost for the reverse traversal of the edge. This is only used when the ``directed`` and ``has_rcost`` parameters are ``true`` (see the above remark about negative costs).

:source: ``int4`` id of the start point
:targets: ``int4[]`` an array of ids of the end points
:directed: ``true`` if the graph is directed
:has_rcost: if ``true``, the ``reverse_cost`` column of the SQL generated set of rows will be used for the cost of the traversal of the edge in the opposite direction.


``pgr_kdijkstraCost`` returns set of ``pgr_costResult``:

:seq:   row sequence
:id1:   path vertex source id (this will always be source start point in the query).
:id2:   path vertex target id
:cost:  cost to traverse the path from ``id1`` to ``id2``. Cost will be -1.0 if there is no path to that target vertex id.


``pgr_kdijkstraPath`` returns set of ``pgr_costResult``:

:seq:   row sequence
:id1:   path vertex target id (identifies the target path).
:id2:   path edge id
:cost:  cost to traverse this edge or -1.0 if there is no path to this target


.. rubric:: Example query ``pgr_kdijkstraCost``

.. code-block:: sql

	SELECT seq, id1 AS source, id2 AS target, cost FROM pgr_kdijkstraCost('
			SELECT gid AS id, 
				 source::integer, 
				 target::integer, 
				 length::double precision AS cost
				FROM ways', 
			10, array[60,70,80], false, false); 


.. rubric:: Query result

.. code-block:: sql
		
	 seq | source | target |       cost       
	-----+--------+--------+------------------
	   0 |     10 |     60 | 13.4770181770774
	   1 |     10 |     70 | 16.9231630493294
	   2 |     10 |     80 | 17.7035050077573
	(3 rows)


.. rubric:: Example query ``pgr_kdijkstraPath``

.. code-block:: sql

	SELECT seq, id1 AS path, id2 AS edge, cost FROM pgr_kdijkstraPath('
			SELECT gid AS id, 
				 source::integer, 
				 target::integer, 
				 length::double precision AS cost
				FROM ways', 
			10, array[60,70,80], false, false); 


.. rubric:: Query result

.. code-block:: sql
		
	 seq | path | edge |        cost         
	-----+------+------+---------------------
	   0 |   60 | 3163 |   0.427103399132954
	   1 |   60 | 2098 |   0.441091435851107
	...
	  40 |   60 |   56 |  0.0452819891352444
	  41 |   70 | 3163 |   0.427103399132954
	  42 |   70 | 2098 |   0.441091435851107
	...
	 147 |   80 |  226 |  0.0730263299529259
	 148 |   80 |  227 |  0.0741906229622583
	(149 rows)


.. _ksp:

Alternative Routes with K-Shortest-Path
-------------------------------------------------------------------------------

The "K" shortest paths algorithm returns alternative routes. The algorithm not only finds the shortest path, but also "K" other paths in order of increasing cost. "K" is the number of shortest paths to find.


.. rubric:: Prerequisites

K-Shortest-Path doesn't require additional attributes to Dijkstra algorithm.


.. rubric:: Description

The algorithm returns a set of ``pgr_costResult`` (seq, id1, id2, cost) rows, that make up a path.

.. code-block:: sql

	pgr_costResult[] pgr_ksp(sql text, source integer, target integer, paths integer, has_rcost boolean);


.. rubric:: Parameters

:sql: a SQL query, which should return a set of rows with the following columns:

  .. code-block:: sql

    SELECT id, source, target, cost, [,reverse_cost] FROM edge_table


  :id: ``int4`` identifier of the edge
  :source: ``int4`` identifier of the source vertex
  :target: ``int4`` identifier of the target vertex
  :cost: ``float8`` value, of the edge traversal cost. A negative cost will prevent the edge from being inserted in the graph.
  :reverse_cost: (optional) the cost for the reverse traversal of the edge. This is only used when ``has_rcost`` the parameter is ``true`` (see the above remark about negative costs).

:source: ``int4`` id of the start point
:target: ``int4`` id of the end point
:paths: ``int4`` number of alternative routes
:has_rcost: if ``true``, the ``reverse_cost`` column of the SQL generated set of rows will be used for the cost of the traversal of the edge in the opposite direction.

Returns set of ``pgr_costResult``:

:seq:   route ID
:id1:   node ID
:id2:   edge ID (``0`` for the last row)
:cost:  cost to traverse from ``id1`` using ``id2``


.. rubric:: Example query

.. code-block:: sql

	SELECT seq AS route, id1 AS node, id2 AS edge, cost FROM pgr_ksp('
			SELECT gid AS id, 
				 source::integer, 
				 target::integer, 
				 length::double precision AS cost
				FROM ways', 
			10, 60, 2, true); 


.. rubric:: Query result

.. code-block:: sql
		
	 route | node | edge |        cost         
	-------+------+------+---------------------
	     0 |   10 | 3163 |   0.427103400230408
	     0 | 1084 | 2098 |   0.441091448068619
	     0 |   35 |   27 |   0.100540332496166
	...
	     0 |   59 |   79 |  0.0452819876372814
	     0 |   60 |    0 |                   0
	     1 |   10 | 3163 |   0.427103400230408
	     1 | 1084 | 2098 |   0.441091448068619
	...
	     1 |   59 |   79 |  0.0452819876372814
	     1 |   60 |    0 |                   0
	(86 rows)

