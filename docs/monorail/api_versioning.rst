API Versioning
===============

All current APIs are prefixed with:

    /api/1.1

RackHD extenders can supplement the *central* API (common) with versioned customer-specific APIs in parallel.

Referencing API Versions in URIs
--------------------------------

    /api/current/...

    /api/1.1/...

The second /[...]/ block in the URI is the version number. The "current" or "latest" placeholder points to the latest version of the API in the system.

Multiple API versions can be added in parallel. Use N, N-1, N-2, etc. as the naming convention.

All API versioning information should be conveyed in HTTP headers.

Versioning Resources
---------------------------------------------------------

A translation and validation chain is used to support versioned "types" for URI resources from the RackHD system. The chain flow is:

    BUSINESS OBJECT --- TRANSLATE --- VALIDATE

Data objects should be versioned in line with the API version.

API Version Guidelines
------------------------------------

Use the following guide lines when determining if a new API version is needed.

The following changes require a new API version:

 - changing the semantic meaning of a URI route
 - removing a URI route

The following changes do not require a new API version:

 - adding an entirely new URI route
 - changing the query parameters (pagination, filtering, etc.) accepted by the URI route
 - changing the return values on error conditions
 - changing the data structure for a resource at a given URI
