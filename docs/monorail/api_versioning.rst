# API Versioning

The current api's are all prefixed with:

    /api/1.1

This is a holdover from our earlier setups where we had some effort/structure
in place to allow for a central general API (common) and were heading to the
idea of versioned customer-specific APIs in parallel with that.

versioning in the URI structure:
--------------------------------

    /api/current/...
    /api/1.1/...

The second /[...]/ block in the URI is the version number, with a placeholder
"current" or "latest" to points to the same as the latest version of the API in
the system.

We'd manage multiple API versions in parallel, aiming for N and N-1 version
support as a minumum, more if needed.

Expectation is that as we evolve this over the next quarter or two, we may have
some significant route structure changes that we want to do, and indicating
versioning through HTTP headers is far more amenable to versioning data
structures on resources at a given URI.

versioning resources presented to or returned from an API
---------------------------------------------------------

Less obvious, but internally we're setting up a translation and validation chain
to support versioned "types" for URI resources from our system, so we can change
the internal business objects and not have an end-impact on the API consumers.
The chain is something like

    +-----------------+    +-----------+    +----------+
    | business object | -- | translate | -- | validate |
    +-----------------+    +-----------+    +----------+

We are currently intending to version the data objects with the API version -
e.g. set an expectation that /api/1.1/nodes will always be consistent,
regardless of internal business logic changes. We added in this layer so we
could change data model and data objects based on feedback and be able to
support additional versions that are specific to the resources, not just the
URI structure.

when do we call it a new API version
------------------------------------

Here's our immediate thoughts on what we thought may be good rules of thumb
for pragmatic API stability.

What would require a new API version

 - changing the semantic meaning of a URI route
 - removing a URI route

What wouldn't require a new API version

 - adding an entirely new URI route
 - changing the query parameters (pagingation, filtering, etc) accepted by the URI route
 - changing the return values on error conditions
 - changing the data structure for a resource at a given URI
