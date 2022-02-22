

# Pure MySQL Closure Table Implementation
## Infinite Unlimited depth Hierarchical Data Solutionx
#### Storage and Retrieval of Hierarchical Data in MySQL

Copyright (c) Evgeny Kalashnikov 2019 - Infinity
ekalashnikov@gmail.com

## This is a retracted documentation of a purchasable item on CodeCanyon
Published here for exposure and examples of capabilities.

\*\*\*\*\*\*\*\*\*\*\*\*
are retracted lines

### To purchase email me at ekalashnikov@gmail.com

## Infinite Levels Categories System

### Features:
1. Elegant and Efficient design (Pure triggers, no application code needed for implementation for ex. PHP)
2. This design supports fully the multiple or unlimited ROOT nodes
3. Select Queries to Sort by a custom order_index, created_at date, Alphabetical order (Can add by updated_at also easily)
4. Triggers that update the closure table on node move with a tree / subtree automatically updates closure table
5. SQL Queries to get parents, children
6. SQL Query to get amounts of children for each object
7. Moving SubTrees to it's own children will fail and error out via MySQL
8. If you update having parent_id = id, it will error out via MySQL as that is not proper format
9. Changing order_index in object table will intelligently change it without duplication, if something in specific tree already has that order_index, the new order_index will be generated that is prior to it, that way order_index always remains unique within a tree, as it will also  subdivide into a fraction if order_index falls between 2 and 3 it will make 2.5, etc, so it always maintains uniqueness and correct position, order_index also supports negative numbers like -1, -2 etc.
10. Hiding a node in nodes table it will hide the child nodes via closure table, but you will have the data  in objects table  of only that parent object being hidden by having hidden_at set to Datetime in objects table, while all other objects will not have this data in objects table, they will have hidden_at data only in objects_tree (ClosureTable), making it an intelligent hide / unhide system, so when you unhide the ParentObject that is hidden in objects table it will unhide all children trees in closure table.
11. hidden_at DATETIME stores a date rather than just 1 or 0 so you can track when the object was hidden as well
12. Bonus! SQL query that will output only a certain amount of sub-children from a sub-tree, so that a system like comments with only 2 or (n) loaded sub-comments can be made
13. It is very fast queries taking 0.000s - 0.010s for simple trees

### To purchase email me at ekalashnikov@gmail.com

## Select Objects by order_index
### Selecting all ROOT nodes
#### Order by order_index

order_crumbs is generated for order_index to order correctly,
order_crumbs takes into account negative numbers and fractional numbers
to do  that it does some  tricks with the number 10,000,000,000,000, 10 trillion, so your maximum orderindex is 10 trillion,
you can increase or decrease this number if you want. (10 trillion is large enough that you will never likely to exceed it)

If the order_index is negative, we deduct the negative number from 10 trillion.
If the order_index is positive we LPAD to 30 and prepend a 1 in front to make positive numbers always bigger than the negative numbers.
DECIMAL(30, 15) allows fractions to 15 decimal places to account for any fractional order_index.

**Result:**

NOTE:
\*\*\*\*\*\*\*\*\*\*\*\*
are retracted lines, to purchase email me at ekalashnikov@gmail.com

```SQL
SELECT SQL_CALC_FOUND_ROWS
 object.`id`,
 object.`parent_id`,
 CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
 object.`title` as title,
 tree.`depth`
-- Start order_crumbs generation for order_index:
  , (***************************)
 FROM `objects` o
 INNER JOIN `objects_tree` t
  ON t.`ancestor_id` = o.`id`
 WHERE t.`descendant_id` = object.`id`
 ) as order_crumbs
 -- End of order_crumbs
             FROM
           `objects` AS object
    LEFT JOIN (SELECT
    ******************* *************************
        FROM `objects_tree` as t
 -- GROUP BY CASE  solves an issue of ROOT nodes
      GROUP BY (CASE WHEN t.`ancestor_id` = t.`descendant_id`
             THEN t.`ancestor_id`
             ELSE t.`depth` END),
             t.`descendant_id`
    ORDER BY t.`depth` DESC
   ) AS tree
 -- This ON Clauses also resolve the ROOT Nodes
  ON (******************** *****************)
OR (object.`id` = tree.`descendant_id`
 AND object.`parent_id` IS NULL
 AND object.`id` = tree.`ancestor_id`)
 -- Get correct Hidden nodes
  JOIN `objects_tree` AS hidden
   ON tree.`descendant_id` = hidden.`descendant_id`
  AND tree.`descendant_id` = hidden.`ancestor_id`
       WHERE (tree.`depth` >= 0
   AND tree.`depth` <= '9999'
   AND hidden.`hidden_at` = 0)
        GROUP BY object.`id`
ORDER BY order_crumbs asc
```
The order  is always handled by order_crumbs, but order_crumbs calculation is different for order_index, created_at or Alphabetical(by title)

## Select ORDER BY created_at datetime
- Different order_crumbs from order_index
- created_at order_crumbs are much simpler

```SQL
SELECT SQL_CALC_FOUND_ROWS
  object.`id`,
  object.`parent_id`,
  CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
  object.`title` as title,
  tree.`depth`
-- Generate the order_crumbs out of created_at dates
   , (****************************
   ***************************
) as order_crumbs
       FROM
      `objects` AS object
-- Tree select to select the correct items accounting for ROOT Nodes
 LEFT JOIN (SELECT
    t.`ancestor_id`,
  t.`descendant_id`,
  t.`hidden_at`,
  t.`depth`
     FROM `objects_tree` as t
 -- GROUP BY CASE  solves an issue of ROOT nodes
   GROUP BY (*****************
   ****************),
          t.`descendant_id`
 ORDER BY t.`depth` DESC
 -- Order by depth DESC is needed
) AS tree
   -- This ON Clauses also resolve the ROOT Nodes
     ON (*****************
   ****************)
     -- Get correct Hidden nodes
    JOIN `objects_tree` AS hidden
     ON tree.`descendant_id` = hidden.`descendant_id`
    AND tree.`descendant_id` = hidden.`ancestor_id`
 -- Get Tree with starting id = 1
        WHERE (tree.`depth` >= 0
   AND tree.`depth` <= '9999'
   AND hidden.`hidden_at` = 0)
        GROUP BY object.`id`
ORDER BY order_crumbs asc
```


## Select ORDER BY Alphabet

```SQL
SELECT SQL_CALC_FOUND_ROWS
  object.`id`,
  object.`parent_id`,
  CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
  object.`title` as title,
  tree.`depth`
-- Generate the order_crumbs out of title
, (*******************************
************************) as order_crumbs
-- order_crumbs end
       FROM `objects` AS object
  LEFT JOIN (SELECT
     t.`ancestor_id`,
   t.`descendant_id`,
   t.`hidden_at`,
   t.`depth`
      FROM `objects_tree` as t
    GROUP BY (*************************
    ***********************),
           t.`descendant_id`
  ORDER BY t.`depth` DESC
 ) AS tree

      ON (**************************
      ***************************)


JOIN `objects_tree` AS hidden
ON tree.`descendant_id` = hidden.`descendant_id`
AND tree.`descendant_id` = hidden.`ancestor_id`
         WHERE (tree.`depth` >= 0
    AND tree.`depth` <= '9999'
    AND hidden.`hidden_at` = 0)
         GROUP BY object.`id`
ORDER BY order_crumbs asc
```
## Select a Specific Tree by its id
- To select a specific tree you just have to add ```tree.ancestor_id = '{id}'``` to the WHERE CLAUSE
 - this will get the specific tree with all the children
 - here is the sample query bellow
```SQL
SELECT SQL_CALC_FOUND_ROWS
    object.`id`,
    object.`parent_id`,
    CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
    object.`title` as title,
    tree.`depth`,
    (*****************************
    *******************************)
   FROM `objects` o
   INNER JOIN `objects_tree` t
    ON t.`ancestor_id` = o.`id`
   WHERE t.`descendant_id` = object.`id`
   ) as order_crumbs
         FROM `objects` AS object
    LEFT JOIN (SELECT
     t.`ancestor_id`,
     t.`descendant_id`,
     t.`hidden_at`,
     t.`depth`
        FROM `objects_tree` as t
      GROUP BY (***********************
      ************************),
             t.`descendant_id`
    ORDER BY t.`depth` DESC
   ) AS tree

  ON (***********************
  ************************)

   JOIN `objects_tree` AS hidden
    ON tree.`descendant_id` = hidden.`descendant_id`
   AND tree.`descendant_id` = hidden.`ancestor_id`
           WHERE (
 -- Get the tree of id = 1
      tree.`ancestor_id` = '1'
             AND tree.`depth` >= 0
       AND tree.`depth` <= '9999'
       AND hidden.`hidden_at` = 0)
      GROUP BY object.`id`
ORDER BY order_crumbs asc
```
## Select Only Children of a Tree?
Change above query WHERE so that tree.depth is greater than 0 as 0 will refer to the tree depth that we select
it will always be 0, even if we select a sub-sub-tree because it would refer to itself in closure table and the depth of itself to itself is always 0. Result:
```SQL
SELECT SQL_CALC_FOUND_ROWS
  object.`id`,
  object.`parent_id`,
  CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
  object.`title` as title,
  tree.`depth`
   , (***************************
   ******************************)
 FROM `objects` o
 INNER JOIN `objects_tree` t
  ON t.`ancestor_id` = o.`id`
 WHERE t.`descendant_id` = object.`id`
 ) as order_crumbs

           FROM
         `objects` AS object
  LEFT JOIN (******************
  ****************************************
 ) AS tree

  ON (*************************
  ***************************)

   JOIN `objects_tree` AS hidden
    ON tree.`descendant_id` = hidden.`descendant_id`
   AND tree.`descendant_id` = hidden.`ancestor_id`
           WHERE (tree.`ancestor_id` = '1'
-- To select only children make tree.`depth` '>' instead '>=' to not have a 0 depth reference
    AND tree.`depth` > 0
    AND tree.`depth` <= '9999'
    AND hidden.`hidden_at` = 0)
      GROUP BY object.`id`
ORDER BY order_crumbs asc
```


## Selecting All Parents of an Object
### What changes here as opposed to regular tree selects:

- Note that this query is the same as all the above queries but has to have  id in WHERE
```tree.descendant_id = '1'``` instead of  ```tree.ancestor_id = '1'``` , where 1 is a reference to some id and also to order the Parents in the order of their ancestry you have to add ```ORDER BY tree.depth DESC``` at the end
- Also the ON CLAUSE for 'tree' here is simpler only ```ON object.id = tree.ancestor_id```
##### Result:
```SQL
SELECT SQL_CALC_FOUND_ROWS
 object.`id`,
 object.`parent_id`,
 object.`title` AS title_with_depth,
 object.`title` as title,
 tree.`depth`
    FROM
         `objects` AS object
  LEFT JOIN (*********************
  *****************************
 ) AS tree
-- Simpler ON Clause for parents
      ON object.`id` = tree.`ancestor_id`
     JOIN `objects_tree` AS hidden
      ON *************************
      *********************************
         WHERE (
         -- Note here the descendant_id instead of ancestor_id as in other queries
         tree.`descendant_id` = '1'
             AND tree.`depth` >= 0
    AND tree.`depth` <= '9999'
    AND hidden.`hidden_at` = 0)
              GROUP BY object.`id`
   -- Correct ordering by depth to get Breadcrumbs
ORDER BY tree.`depth` DESC
```

## Select Trees with Limited # of Children

This is the most complicated query  and is experimental, it uses custom functions SPLIT_STRING and
REPLACE_SPLIT_STRING to keep track of depth of each  object in a tree sequence.

```SQL
SELECT SQL_CALC_FOUND_ROWS * FROM (
  SELECT
     @row := IF(@depth != depth,
            IF(depth > @depth, 1, SPLIT_STRING(@rowinc, ',', depth + 1) + 1),
            SPLIT_STRING(@rowinc, ',', @depth + 1) + 1) AS RowNum,
   @depth := IF(@depth != depth, depth, @depth) as depthTrack,
  @rowinc := REPLACE_SPLIT_STRING(@rowinc, ',', @depth + 1, @row) as RowInc,
   tab.*
FROM (
    SELECT @row :=0, @rowinc := '0', @depth := 0) r,
    -- This inside query is the same as any query above
(SELECT
   object.`id`,
   object.`parent_id`,
   CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
   object.`title` as title,
   tree.`depth`,
   (SELECT
			COUNT(*)
		FROM `objects_tree` ot
		WHERE ot.`ancestor_id` = object.`id`
		AND ot.`depth` = 1
	) as ChildrenCount,
    (**********************************
    *****************************
  ) as order_crumbs
            FROM
          `objects` AS object
   LEFT JOIN (SELECT
      t.`ancestor_id`,
    t.`descendant_id`,
    t.`hidden_at`,
    t.`depth`
       FROM `objects_tree` as t
     GROUP BY (************************
     *********************),
            t.`descendant_id`
   ORDER BY t.`depth` DESC
  ) AS tree

       ON (***********************
       ***********************)

  JOIN `objects_tree` AS hidden
   ON tree.`descendant_id` = hidden.`descendant_id`
  AND tree.`descendant_id` = hidden.`ancestor_id`
      WHERE (tree.`depth` >= 0
 AND tree.`depth` <= '9999'
 AND hidden.`hidden_at` = 0)
      GROUP BY object.`id`
ORDER BY order_crumbs asc

          ) as tab ) ranked
  -- rownum refers to the max number of children rows tha you want
  -- OR (depth = 0) makes sure the ROOT nodes are shown always and not limited by rownum

WHERE (rownum <= 3 and depth > 0) OR (depth = 0)
```
- rownum refers to the max number of children rows that you want
- ```OR (depth = 0)``` makes sure the ROOT nodes are shown always and not limited by rownum

- Change the last WHERE clause's rownum to limit by (n) number of rows per child
```SQL
WHERE (rownum <= {n} and depth > 0) OR (depth = 0)
```
## Getting Specific depth
In all the queries above in WHERE clause you see this
```SQL
WHERE (tree.`depth` >= 0
 AND tree.`depth` <= '9999' ...
```

By changing the first tree.depth >= {{0}} you can control the minimum depth to display from that tree.
By changing the second tree.`depth <= {{9999}} you can control the maximum depth.

## Getting the Children Count  for each object

  If you want to get a children count for each Object you have to add this column select SQL
```SQL
   (SELECT
      COUNT(*)
    FROM `objects_tree` ot
    WHERE ot.`ancestor_id` = object.`id`
    AND ot.`depth` = 1
	) as ChildrenCount,
```
to the COLUMNS select,
```ot.depth = 1``` always refers to the children depth of 1, even in any sub-sub-tree it will always be  1

Here is full query with order_index :
```SQL
SELECT SQL_CALC_FOUND_ROWS
 object.`id`,
 object.`parent_id`,
 CONCAT( REPEAT('— ', tree.`depth`), object.`title` ) AS title_with_depth,
 object.`title` as title,
 tree.`depth`,
 -- Get Children Count
    (SELECT
			COUNT(*)
		FROM `objects_tree` ot
		WHERE ot.`ancestor_id` = object.`id`
		AND ot.`depth` = 1
	) as ChildrenCount,
-- Start order_crumbs generation for order_index :
   (*****************************
   ****************************)
 FROM `objects` o
 INNER JOIN `objects_tree` t
  ON t.`ancestor_id` = o.`id`
 WHERE t.`descendant_id` = object.`id`
 ) as order_crumbs
 -- End of order_crumbs
             FROM
           `objects` AS object
    LEFT JOIN (*************************
    **********************

 -- GROUP BY CASE  solves an issue of ROOT nodes
      GROUP BY (***********************
      ***********************),
             t.`descendant_id`
    ORDER BY t.`depth` DESC
   ) AS tree

   -- This ON Clauses also resolve the ROOT Nodes
  ON (*********************
  **********************)
 -- Get correct Hidden nodes
  JOIN `objects_tree` AS hidden
   ON tree.`descendant_id` = hidden.`descendant_id`
  AND tree.`descendant_id` = hidden.`ancestor_id`
       WHERE (tree.`depth` >= 0
   AND tree.`depth` <= '9999'
   AND hidden.`hidden_at` = 0)
        GROUP BY object.`id`
ORDER BY order_crumbs asc
```

## Hiding an Object from a tree

```SQL
UPDATE objects SET hidden_at = NOW() WHERE id = 1
```
This will hide all objects under id = 1 tree, it will update ClosureTable and set hidden_at field for all the children rows, where ancestor_id = descendant_id (self-referential row for an object in closure table)
### Unhide the object
```SQL
UPDATE objects SET hidden_at = 0 WHERE id = 1
```
This will unhide the object and all its children, except those that were also Hidden from objects table, making hidden_at system also very intelligent.



## order_index Explanation:

### Regarding the order_index:

This design is 2 tables:
**objects**  (with 1 constraint on parent_id field referencing )
id VARCHAR / INT
parent_id VARCHAR / INT
title VARCHAR
order_index VARCHAR
created_at DATETIME
hidden_at DATETIME

**objects_tree**  (with 2 constriants referencing objects table id)
ancestor_id VARCHAR / INT
descendant_id VARCHAR / INT
depth INT
hidden_at DATETIME

**objects**  table has 4 triggers (before_insert, afer_insert, before_update, after_update)


### This design does not require any PHP code or even MySQL procedures
As MySQL triggers handle everything, when we update  the  **objects**  table.

When Object is being inserted or updated, the trigger detects the order_index, if you insert an Object like
```SQL
INSERT INTO objects (id,parent_id,title, order_index, created_at) VALUES('1', NULL, 'title 1', NULL, NOW());
```

See I did not specify the order_index and left it NULL,
that will automatically add this object into that specific tree (in this case no parent_id, hence this is root node) and if there are no  objects in the objects table yet, it will create a row id = '1', parent_id = NULL, title = 'title 1' order_index = 0;
If we start adding objects more with order_index = NULL or order_index = '' then it will append the order_index by 1;

Creating rows like :

id = '2', parent_id = NULL, title = 'title 2' order_index = 1;
id = '3', parent_id = NULL, title = 'title 3' order_index = 2;
id = '4', parent_id = NULL, title = 'title 4' order_index = 3;

The order_index refers to order_index *within a subtree only*, so if we add another node that has parent id '2' like this

id = '5', parent_id = '2', title = 'title 5' order_index = NULL
the trigger will make this into:
id = '5', parent_id = '2', title = 'title 5' order_index = 0;

then if we add more objects under parent_id = '2' like this

id = '6', parent_id = '2', title = 'title 6' order_index = NULL;
the triggers will convert to
id = '6', parent_id = '2', title = 'title 5' order_index = 1;

So as you see the sub-tree under parentId = '2' now has it's own order_index, that is also 0, 1, 2, 3 .... (n)


### Now there is also this syntax: order_index = '+' or '^'

If order_index = '^' it will prepend order_index making first in the specific sub-tree or root tree:
id = '7', parent_id = '2', title = 'title 7' order_index = '^';

The triggers will prepend order_index and make this now:
id = '7', parent_id = '2', title = 'title 7' order_index = '-1';

See "negative 1" or -1 is now the first order_index as -1 is less than 0

If order_index = '+" it will behave just as order_index = NULL meaning it will append object to that sub tree or increment the order_index by 1


### Updating the order_index has its own specification:

When you update order_index but you did not update the parent_id, it means the object is being moved within the sub-tree in
which it already exists, and if we change order_index, we have to account for that objects already existing position
and whether we are moving it forwards or backwards in the Index, suppose there is already order_indexed objects with

order_indexes like 0, 1, 2, 3, 4, 5 which is basic appending of +1 in order_index, but what if we want to move object with order_index 2 to position of the order_index 3, to do that we assign order_index(2) = 3, but 2 is before 3 so to replace 3 it has to become 3.5 because once it moves to position of 3, the number of elements before position 3 decreases by 1.

But this is different in case if we want to move object order_index 4 to position of 3, then the order_index(4) = 3 will not result in 3.5, it has to result in 2.5, because it needs to replace 3 in the order_index. So all of this is handled automatically in the triggers.

You can also specify exact value for order_index also, and if that order_index is not existent (in that sub-tree) it will assign that number exactly like 3.456 or 2.41 or whatever, but if you give order_index 100, but the max order_index is 5 in that sub tree, it will convert order_index(100) to order_index 6,  or if you put order_index -100, but the most negative order_index is -3, it will make it order_index -4.

So the triggers keep everything on track, so that you do not end up with messed up order_index.

The Beauty of this design is that I've exported everything to MySQL and  to implement this into PHP is very easy  as it requires 0 application code from PHP, every thing is handled by triggers and constraints in MySQL by changing the values of parent_id and order_index, created_at and hidden_at parameters and the objects_tree gets updated automatically every time, keeping track of order_index, depth, and hidden_at.

And if you move things in wrong directions like assigning a parent_id to it's own children or parent_id = id,  the trigger will fail and error out via MySQL which is caught by standard mysql error via php.

### Issues
Report any issues and bugs to ekalashnikov@gmail.com


To purchase email me at ekalashnikov@gmail.com
