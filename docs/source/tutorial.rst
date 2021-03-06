.. _tutorial:

Tutorial
========

Exploring basic rules
---------------------

Let's try exploring the `/lots` endpoint:

.. literalinclude:: tutorial/lot-listing.http
   :language: javascript

Just invoking it reveals empty set.

Now let's attempt creating a lot:

.. literalinclude:: tutorial/lot-post-attempt.http
   :language: javascript

Error states that the only accepted Content-Type is `application/json`.

Let's satisfy the Content-type requirement:

.. literalinclude:: tutorial/lot-post-attempt-json.http
   :language: javascript

Error states that no `data` has been found in JSON body.


.. index:: Lot

Creating lot
------------


Let's create lot with the minimal (only required) data set:

.. literalinclude:: tutorial/lot-post-2pc.http
   :language: javascript

.. literalinclude:: tutorial/lot-patch-2pc.http
   :language: javascript

.. literalinclude:: tutorial/lot-patch-2pc-verification.http
   :language: javascript

Success! Now we can see that new object was created. Response code is `201`
and `Location` response header reports the location of the created object.  The
body of response reveals the information about the created lot: its internal
`id` (that matches the `Location` segment), its official `lotID` and
`dateModified` datestamp stating the moment in time when lot was last
modified. Pay attention to the `lotType`. Note that lot is
created with `verification` status.

Let's access the URL of the created object (the `Location` header of the response):

.. literalinclude::tutorial/blank-lot-view.http
   :language: javascript

.. XXX body is empty for some reason (printf fails)

We can see the same response we got after creating lot.

.. literalinclude:: tutorial/initial-lot-listing.http
   :language: javascript

Let's see what listing of lots reveals us:


We do see the internal `id` of the lot (that can be used to construct full URL by prepending `https://lb.api-sandbox.registry.ea.openprocurement.net/api/0.1/lots/`) and its `dateModified` datestamp.

The previous lot contained only required fields. Let's try creating lot with more data
(lot has status `created`):

.. literalinclude:: tutorial/create-second-lot.http
   :language: javascript

.. literalinclude:: tutorial/pending-second-lot.http
   :language: javascript

.. XXX patching lot to pending.dissolution

.. literalinclude:: tutorial/patch-lot-to-pending.dissolution.http
   :language: javascript

And again we have `201 Created` response code, `Location` header and body with extra `id`, `lotID`, and `dateModified` properties.

Let's check what lot registry contains:

.. literalinclude:: tutorial/listing-with-some-lots.http
   :language: javascript

And indeed we have 2 lots now.

Assume that we have lot in `active.salable` status and we want
to modify this lot. For such purposes `recomposed` status exists.

.. literalinclude:: tutorial/patch-lot-to-recomposed.http
   :language: javascript

After some checks have been done, lot status will be switched to `pending` automatically.

Modifying Lot
-------------

Let's update lot description:

.. literalinclude:: tutorial/patch-lot.http
   :language: javascript

.. XXX body is empty for some reason (printf fails)

We see the added properies have merged with existing lot data. Additionally, the `dateModified` property was updated to reflect the last modification datestamp.

Checking the listing again reflects the new modification date:

.. literalinclude:: tutorial/lot-listing-after-patch.http
   :language: javascript

Deleting Lot
------------

Let's delete lot:

.. literalinclude:: tutorial/lot-delete.http
   :language: javascript

Integration with assets
-----------------------


Concierge operations
--------------------

For lot to be formed, you need to specify some id of the assets to be included 
in that lot. If all of the assets are available, they will be attached to lot 
and status will be changed to `active.salable`:

.. literalinclude:: tutorial/concierge-patched-lot-to-active.salable.http
   :language: javascript

In case of at least one of the assets is unavailable (e.g. it has already been 
attached to another lot), status of the current one will be turned to `pending`:

.. literalinclude:: tutorial/concierge-patched-lot-to-pending.http
   :language: javascript

When bot finds that status of lot is `pending.dissolution`, it
turns status of the assets being attached to that lot to `pending`. Status of the lot itself will become `dissolved`.
   
.. literalinclude:: tutorial/patch-lot-to-dissolved.http
   :language: javascript

When bot finds that status of lot is `recomposed`, it
turns status of the assets being attached to that lot to `pending`. Status of the lot itself 
becomes `pending` as well.

.. literalinclude:: tutorial/patch-lot-to-pending-from-recomposed.http
   :language: javascript

When bot finds that status of lot is `pending.sold`, it
turns status of the assets being attached to that lot to `complete`. Status of the lot itself
turns to `sold`.

.. literalinclude:: tutorial/patch-lot-to-sold-from-pending.sold.http
   :language: javascript
   
Convoy operations
-----------------

When lot is finally formed (`active.salable`) it can be used in the
procedure within CDB. For this to be done, you need to specify lot id. 
By doing this, you will find the `merchandisingObject` field with the current 
lot id in the created procedure and id of the auction within which 
it is going to be sold. Status of the lot used will be automatically changed 
to `active.awaiting` in RDB. This indicates that Organizer is creating some auction with
this lot within CDB, so it is currently unavailable for usage.

.. literalinclude:: tutorial/convoy-patched-lot-to-active.awaiting.http
   :language: javascript

When the procedure is successfully created, lot status will be changed to 
`active.auction`: 

.. literalinclude:: tutorial/convoy-patched-lot-to-active.auction.http
   :language: javascript

After lot has been sold in the auction, its status is turned to `pending.sold` within RDB:

.. literalinclude:: tutorial/convoy-patched-lot-to-pending.sold.http
   :language: javascript

In case of that lot has not been sold, its status will be changed to `active.salable` in RDB:

.. literalinclude:: tutorial/convoy-patched-lot-to-active.salable.http
   :language: javascript

