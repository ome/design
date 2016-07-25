

 * [API Overview](#api-overview)
    * [Omero-marshal and Projection-based APIs](#omero-marshal-and-projection-based-apis)
    * [Versioning](#versioning)
    * [Pagination](#pagination)
    * [Normalizing Experimenters and Groups](#normalizing-experimenters-and-groups)
    * [Child Counts](#child-counts)
    * [Error handling](#error-handling)
 * [Image containers](#image-containers)
    * [List projects](#list-projects)
    * [Create a project](#create-a-project)
    * [Get a single project](#create-a-single-project)
    * [Delete a project](#delete-a-project)


API Overview
============

The OMERO json API described here provides create, read, update and delete
access to an underlying OMERO server.


Omero-marshal and Projection-based APIs
---------------------------------------
The majority of the API urls use [omero-marshal](https://github.com/openmicroscopy/omero-marshal)
to generate json dictionaries from OMERO **model** objects. The json dictionaries will be populated
with all fields of the OMERO model object, such that changing a single field and sending this
back to OMERO will update the object accordingly.
All these url are under the ```m``` prefix:

    <server>/webgateway/api/v1.0/m/


In addition, a smaller number of API urls perform customised
queries as used by the webclient. These queries use **projections** and
typically load a subset of fields for
OMERO objects in order to improve performance for large data counts.
These urls are available under the ```p``` prefix:

    <server>/webgateway/api/v1.0/p/


Versioning
----------
The JSON API uses major and minor version numbers to reflect breaking
and non-breaking changes respectively. Non-breaking changes include simple
addition of attributes to JSON data or addition of new urls.
The API version is not strictly tied to the version of OMERO.server.
The format of JSON returned will also depend on version of omero-marshal
that is installed.
Versions are included in the url as shown above.


Pagination
----------
Requests that return a number of items will be paginated by default.
This can be specified using the ```page``` and ```limit``` query
parameters. See table below.


Normalizing Experimenters and Groups
------------------------------------
When returning a list of JSON objects that each contain ```omero:details``` with
```owner``` and ```group``` data, these will typically be nested many times
within the list. In order to avoid this duplication, we can remove objects from
within each ```omero:details``` and place them under top-level ```experimenters```
and ```groups``` lists.
You can specify this with the ```?normalize=true``` query parameter.


Child Counts
------------
For container objects such as Projects, Datasets, Screens and Plates it is
often useful to know the number of children within them. This can be
specified with ```?childCount=true``` parameter.


JSON-P Callbacks
----------------
To wrap the response in a JSON function, use the callback query parameter
to specify the name of the function. E.g. ```?callback=foo```.


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    page        Number      Page number, starting at page=1 (default)
                            Can use page=0 to disable pagination and return all results
    limit       Number      The size of each page. The default is 200 and can be set by
                            $ bin/omero config set omero.web.page_size 100
    normalize   Boolean     Place Experimenters and Groups into top-level lists instead
                            of nesting within objects
    childCount  Boolean     Use ?childCount=true to include an omero:childCount attribute
                            for container objects
    callback    String      Name of function to wrap the JSON data


Error handling
--------------

Invalid parameters or invalid JSON will result in a ```400 Bad Request```:

    {"message": "Invalid parameter value"}


Unhandled exceptions are handled with a ```500``` error response that will
include the error


Getting Started
===============


List supported versions
-----------------------

You need to find which versions of the API are supported by your server,
as described above. These are provided by the base url:

    GET     /webgateway/api/


**Response**

    {
      versions: [
        {
          version: "1.0",
          base_url: "https://<server>/webgateway/api/v1.0/"
        }
      ]
    }


List starting urls
------------------

The base url for the chosen version will list a number of urls for
logging on and getting started.

    GET     /webgateway/api/v1.0/


**Response**

    {
      login_url: "https://<server>/webgateway/api/v1.0/login/",
      token_url: "https://<server>/webgateway/api/v1.0/token/",
      projects_url: "https://<server>/webgateway/api/v1.0/m/projects/",
      servers_url: "https://<server>/webgateway/api/v1.0/servers/"
    }


List available OMERO servers
----------------------------

Your API may allow you to connect to several different OMERO servers.

    GET     /webgateway/api/v1.0/servers/


**Response**

    {
      servers: [
        {
          host: "nightshade.openmicroscopy.org",
          server: "omero",
          id: 1,
          port: 4064
        }
      ]
    }


Get CSRF token
--------------

In order to prevent CSRF attacks, will need to obtain a CSRF token
for your session and use it for all subsequent POST, PUT and DELETE
requests in that session. You can obtain this from the ```csrftoken```
cookie of any request or from the following request:

    GET     /webgateway/api/v1.0/token/


**Response**

    {
      token: "eNoVq528bOqlhQqbCzKuviODTRX3PUO2"
    }


Login
-----

You can login to create an OMERO session. You must also include the CSRF token.
The EventContext for this session will be returned to you.

    POST    /webgateway/api/v1.0/login/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    server      Number      ID of the server
    username    String      User's username
    password    String      User's password


**Response**


    {
      eventContext: {
        userName: "ben",
        eventId: -1,
        sessionUuid: "0b30ee4a-c0b2-4b0f-9c61-f48b31bcad8c",
        eventType: "User",
        userId: 3,
        sessionId: 171319,
        groupName: "Nevis Lab",
        isAdmin: False,
        memberOfGroups: [5, 1, 4],
        leaderOfGroups: [],
        groupId: 5
      },
      success: True
    }


Image containers
================

OMERO organises images in 2 types of many-to-many hierarcies:
``screen/plate/[Run]/Well/image`` for HCS data and ``projects/datasets/images``
for other data. images can also be ``Orphaned`` if not contained within
a ``Well`` or ``Dataset``.
OMERO also supports ``Shares``, which provide more permissive access 
to the list of images linked to a Share.


List projects
-------------

    GET     /webgateway/api/v1.0/m/projects/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return projects from this group
    owner       Number      Only return projects owned by this user


**Response**

    {
      projects: [
        {
          @id: 9601,
          @type: "http://www.openmicroscopy.org/Schemas/OME/2015-01#Project",
          Name: "Recent Data",
          Description: "Recently created images",
          childCount: 5008,
          omero:details: {
            owner: # see experimenter above,
            group: # see group above,
            @type: "TBD#Details",
            permissions: {
              canDelete: true,
              perm: "rwra--",
              canEdit: true,
              canAnnotate: true,
              canLink: true,
              @type: "TBD#Permissions"
            }
          }
        }
      ]
    }


Create a project
----------------

    POST    /webgateway/api/v1.0/m/projects/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      The group to which the project is assigned
    name        String      Required. The name of the project
    description String      The description of the project


**Response**

    201 Created
    {
      data: {
        id: 8247,
        name: "My new project",
        description: "Testing this",
        owner, {
          id: 4
        },
        group, {
          id: 3
        }
      }
    }


Get a single project
--------------------

    GET   /webgateway/api/v1.0/m/projects/:projectId


**Response**

    {
      data: {
        "id": 668,
        "name": "Yeast Mitosis",
        "description": ""
      }
    }


Update a project
----------------

    PATCH   /webgateway/api/v1.0/m/projects/:projectId


**Response**

    {
      data: {
        "id": 668,
        "name": "Yeast Mitosis",
        "description": ""
      }
    }


Delete a project
----------------

    DELETE  /webgateway/api/v1.0/m/projects/:projectId


**Response**

    204   No Content

