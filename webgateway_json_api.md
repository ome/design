

 * [API Overview](#api-overview)
    * [Pagination](#pagination)
    * [Error handling](#error-handling)
    * [Normalizing Experimenters and Groups](#normalizing-experimenters-and-groups)
 * [Image containers](#image-containers)
    * [List all top-level containers](#list-all-top-level-containers)
    * [List projects](#list-projects)
    * [Create a project](#create-a-project)
    * [Get a single project](#create-a-single-project)
    * [Delete a project](#delete-a-project)


API Overview
============

The OMERO json API described here provides create, read, update and delete
access to an underlying OMERO server. 
The majority of the API urls use [omero-marshal](https://github.com/openmicroscopy/omero-marshal)
to generate json dictionaries from OMERO **model** objects. The json dictionaries will be populated
with all fields of the OMERO model object, such that changing a single field and sending this
back to OMERO will update the object accordingly.
All these url are under the ```m``` prefix:

    <server>/api/m/


In addition, a smaller number of API urls perform customised
queries as used by the webclient. These queries use **projections** and
typically load a subset of fields for
OMERO objects in order to improve performance for large data counts.
These urls are available under the ```p``` prefix:

    <server>/api/p/


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


Error handling
--------------

Invalid parameters or invalid JSON will result in a ```400 Bad Request```:

    {"message": "Invalid parameter value"}


Unhandled exceptions are handled with a ```500``` error response that will
include the error


Image containers
================

OMERO organises images in 2 types of many-to-many hierarcies:
``screen/plate/[Run]/Well/image`` for HCS data and ``projects/datasets/images``
for other data. images can also be ``Orphaned`` if not contained within
a ``Well`` or ``dataset``.
OMERO also supports ``Shares``, which provide more permissive access 
to the list of images linked to a Share.

List all top-level containers
-----------------------------

List containers that are not children of other containers.
This includes a virtual ``Orphaned images`` container that
gives the number of orphaned images.
This is equivalent to the initial display of data in the clients.

    GET    /api/p/containers/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return containers from this group
    owner       Number      Only return containers owned by this user


**Response**

    {
      data: {
        projects: [],
        datasets: [],
        screens: [],
        plates: [],
        orphaned: {
          id: -1,     # currently this is the experimenter_id
          name: "Orphaned images",
          childCount: 56
        }
      }
    }


List projects
-------------

    GET     /api/p/projects/


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

    POST    /projects/


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

    GET   /projects/:projectId


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

    POST  /projects/:projectId


**Response**

    204   No Content




Urls for listing objects: matrix of query paramters
===================================================

                    page    limit   owner   group   member    orphaned  project dataset screen  plate   tag     tagset
    groups          x       x                       x
    experimenters   x       x               x  
    containers      x       x       x       x                                                           
    projects        x       x       x       x                                                           
    datasets        x       x       x       x                 x         x                              
    images          x       x       x       x                 x                 x
    screens         x       x       x       x 
    plates          x       x       x       x                 x                         x      
    runs            x       x       x       x                                                   x       
    tagged          x       x       x                                                                   R
    tags            x       x       x       x                 x                                                 x  
    shares          x       x       x               x


All parameters are optional except 'R' (required)

 - page: Page index for pagination
 - limit: number of objects per page
 - owner: filter objects by owner
 - group: filter objects or experimenters by their group
 - member: filter groups or shares that user belongs to
 - orphaned: only return objects that have no parent container

 - images and tagged urls: boolean parameters for getting extra info for images
    - sizeXYZ: get image sizeX, sizeY & sizeZ
    - date: get image import 'date' and acquisition date `acqDate`
    - thumbVersion (not for tagged): get current thumbnail version


Other urls
==========

    links           json: {"dataset":{"10":{"image":[1,2,3]}}}
    api_tags_and_tagged_list_DELETE  ids
    paths_to_object  experimenter, project, dataset, image, screen, plate, run/acquisition, well, group



