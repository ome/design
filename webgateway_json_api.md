

 * [API Overview](#api-overview)
    * [Pagination](#pagination)
    * [Error handling](#error-handling)
 * [Groups and experimenters](#groups-and-experimenters)
    * [List Groups](#list-groups)
    * [List Experimenters](#list-experimenters)
    * [Get a single experimenter](#get-a-single-experimenter)
 * [Image containers](#image-containers)
    * [List all top-level containers](#list-all-top-level-containers)
    * [List projects](#list-projects)
    * [Create a project](#create-a-project)
    * [Get a single project](#create-a-single-project)
    * [Delete a project](#delete-a-project)
    * [List datasets](#list-datasets)
    * [Create a dataset](#create-a-dataset)
    * [Get a single dataset](#create-a-single-dataset)
    * [Delete a dataset](#delete-a-dataset)
    * [List datasets in a project](#list-datasets-in-a-project)
    * [Create a dataset in a project](#create-a-dataset-in-a-project)
    * [List screens](#list-screens)
    * [Create a screen](#create-a-screen)
    * [List plates](#list-plates)
    * [List plates in a screen](#list-plates-in-a-screen)
    * [List plate Runs](#list-plate-runs)
    * [List Runs in a plate](#list-runs-in-a-plate)
    * [List Paths to an Object](#list-paths-to-an-object)
 * [Shares and Discussions](#shares-and-discussions)
    * [List Shares](#list-shares)
    * [List Discussions](#list-discussions)
    * [List Shares and Discussions](#list-shares-and-discussions)
    * [List images in Share](#list-images-in-share)
 * [Images](#images)
    * [List images](#list-images)
 * [Tags](#tags)
    * [List Tags](#list-tags)
    * [List Tagged Objects](#list-tagged-objects)
 * [Linking and Unlinking Objects](#linking-and-unlinking-objects)
    * [Create Links](#create-links)
    * [Delete Links](#delete-links)



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


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    page        Number      Page number, starting at page=1 (default)
                            Can use page=0 to disable pagination return all results
    limit       Number      The size of each page. The default can be set in
                            $ bin/omero config set omero.web.page_size 100

Error handling
--------------

Invalid parameters or invalid JSON will result in a ```400 Bad Request```:

    {"message": "Invalid parameter value"}


Unhandled exceptions are handled with a ```500``` error response that will
include the error



Groups and Experimenters
========================

List groups
-----------

You can list all the groups on the OMERO server

    GET     api/p/groups/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    member      Number      Only return Groups that the experimenter is a member of
    owner       Number      Only return Groups that the experimenter is an owner of


**Response**

    {
      groups: [
        {
          omero:details: {
            @type: "TBD#Details",
            permissions: {
              canDelete: false,
              perm: "rwr---",
              canEdit: false,
              canAnnotate: false,
              canLink: false,
              @type: "TBD#Permissions"
            }
          },
          @id: 12,
          @type: "http://www.openmicroscopy.org/Schemas/OME/2015-01#ExperimenterGroup",
          Name: "Swedlow Lab"
        }
      ]
    }


List experimenters
------------------

You can list all the experimenters on the OMERO server

    GET     api/p/experimenters/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return experimenters that belong to this group
    admin       Boolean     If true, only return Admin experimenters
    active      Booleean    If true, only return active experimenters


**Response**

    {
      experimenters: [
        {
          UserName: "ben",
          FirstName: "Ben",
          MiddleName: "",
          omero:details: {
            @type: "TBD#Details",
            permissions: {
              canDelete: false,
              perm: "rw----",
              canEdit: false,
              canAnnotate: false,
              canLink: false,
              @type: "TBD#Permissions"
            }
          },
          Email: "newtest2@emaildomain.com",
          Institution: "",
          LastName: "Nevis",
          @id: 4,
          @type: "http://www.openmicroscopy.org/Schemas/OME/2015-01#Experimenter"
        }
      ]
    }


Get a single experimenter
-------------------------


    GET     /api/p/experimenters/:id


**Response**

    // as above



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


List datasets
-------------

    GET     /datasets/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return datasets from this group
    owner       Number      Only return datasets owned by this user
    project     Number      List datasets that are children of this project
    orphaned    Boolean     If true, only return datasets not contained
                            within a project. Default False


**Response**

    {
      data: [
        {
          id: 5001,
          owner: {
            id: 3
          },
          childCount: 3,
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          name: "My dataset"
        }
      ]
    }


Create a dataset
----------------

    POST    /datasets/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      The group to which the dataset is assigned
    name        String      Required. The name of the dataset
    description String      The description of the dataset


**Response**

    201 Created
    {
      data: {
        id: 8377,
        name: "New dataset Name",
        description: "Description text",
        owner, {
          id: 4
        },
        group, {
          id: 3
        }
      }
    }


Get a single dataset
--------------------

    GET   /datasets/:datasetId


**Response**

    {
      data: {
        "id": 879,
        "name": "Big dataset",
        "description": "Lots of images here"
      }
    }


Delete a dataset
----------------

    POST  /datasets/:datasetId


**Response**

    204   No Content


List datasets in a project
--------------------------

This is the equivalent of ```/datasets/?project=:projectId```

    GET     /project/:projectId/datasets/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return datasets owned by this user


**Response**

    # As for List datasets above


Create a dataset in a project
-----------------------------

    POST    /projects/:projectId/datasets/


**Parameters**

    # As for Create dataset above


**Response**

    # As for Create dataset above


List screens
------------

    GET     /screens/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return screens from this group
    owner       Number      Only return screens owned by this user


**Response**

    {
      data: [
        {
          id: 3141,
          owner: {
            id: 4
          },
          childCount: 31,
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          name: "siRNAi screen 1"
        }
    ]}


Create a screen
---------------

    POST    /screens/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      The group to which the screen is assigned
    name        String      Required. The name of the screen
    description String      The description of the screen


**Response**

    201 Created
    {
      data:
        id: 8247,
        name: "My new screen",
        description: "Testing this",
        owner, {
          id: 4
        },
        group, {
          id: 3
        }
      }
    }


List plates
-----------

    GET     /plates/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return plates from this group
    owner       Number      Only return plates owned by this user
    screen      Number      List plates that are children of this screen
    orphaned    Boolean     If true, only return plates not contained
                            within a screen. Default False


**Response**

    {
      data: [
        {
          id: 5721,
          owner: {
            id: 4
          },
          childCount: 25,
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          name: "Treatment plate"
        }
    ]}


List plates in a screen
-----------------------

This is the equivalent of ```/plates/?screen=:screenId```

    GET     /screens/:screenId/plates/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return plates owned by this user


**Response**

    # As for List plates above


List plate Runs
---------------

    GET   /runs/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return plates from this group
    owner       Number      Only return plates owned by this user
    plate       Number      List Runs that are children of this plate


**Response**

    {
      data: [
        {
          id: 52,
          owner: {
            id: 3
          },
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          name: "Run Name 3"
        }
      ]
    }


List Runs in a plate
--------------------

This is the equivalent of ```/runs/?plate=:plateId```

    GET     /plates/:plateId/runs/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return Runs owned by this user


**Response**

    # As for List Runs above


List Paths to an Object
-----------------------

In order to reveal a specified object in the above hierarchies,
it is useful to know the path(s) from parent top-level objects.
Object types supported are ```project```, ```dataset```, ```image```,
```screen```, ```plate```, ```run```, ```well```.
If type is ```image``` and the image is in a dataset, then we
do not also check if the image is in a plate and Well.

    GET   /paths/:objectType/:objectId/

**Parameters**

TODO: do we need these parameters??

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return paths that include this experimenter
                            (where they own the top level object)
    project     Number      Only return paths that include this project
    dataset     Number      Only return paths that include this dataset
    screen      Number      Only return paths that include this screen
    plate       Number      Only return paths that include this plate


**Response**

TODO: should include "well" and "image" in path (not needed in jsTree but API should provide them)

Image in a dataset(s):

    {
      data: [
        [
          {
            type: "experimenter",
            id: 3
          },
          {
            type: "project",
            id: 7301
          },
          {
            type: "dataset",
            id: 9701
          },
          {
            type: "image",
            id: 6901
          }
        ],
        [
          {
            type: "experimenter",
            id: 3
          },
          {
            type: "dataset",
            id: 10851
          },
          {
            type: "image",
            id: 6901
          }
        ]
      ]
    }

Image in a plate and Well:

    {
      data: [
        [
          {
            type: "experimenter",
            id: 3
          },
          {
            type: "screen",
            id: 701
          },
          {
            type: "plate",
            id: 752
          },
          {
            type: "run",
            id: 402
          }
        ]
      ]
    }



Shares and Discussions
======================

Shares are virtual containers that allow users to view
a collection of images that are not accessible to them under
the regular OMERO permissions, and to add Comments.
Discussions simply allow a series of Comments without including
images.


List Shares
-----------

    GET   /shares/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return shares owned by this user
    member      Number      Return shares where this user is a member


**Response**

    {
      data: [
        {
            id: 4602,
            owner: {
              id: 3
            },
            active: false,
            expired: false,
            childCount: 3,
        }
      ]
    }


List Discussions
----------------

    GET   /discussions/


**Parameters**

    # As for List Shares above


**Response**

    {
      data: [
        {
            id: 25775,
            owner: {
              id: 3
            },
            active: true,
            expired: false
        }
      ]
    }


List Shares and Discussions
---------------------------

    GET   /publicContainers/


**Parameters**

    # As for List Shares above


**Response**

    {
      data: {
        discussions: [
          {
              id: 25775,
              owner: {
                id: 3
              },
              active: true,
              expired: false
          }
        ],
        shares: [
          {
              id: 4602,
              owner: {
                id: 3
              },
              active: false,
              expired: false,
              childCount: 3,
          }
        ]
      }
    }


List images In Share
--------------------

    GET   /shares/:share_id/images/


**Response**

    {
      data: [
        {
          id: 4718,
          name: "GFP-INCENP01.tif",
          fileset: {
            id: 3666
          },
          pixels: {
            sizeX: 1344,
            sizeY: 1040,
            sizeZ: 1
          }
          creationDate: "2014-09-10T16:36:29Z",
          owner: {
            id: 4
          },
          share: {
            id: 134198
          }
        }
      ]
    }


Images
======

List images
-----------

    GET   /images/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return images from this group
    owner       Number      Only return images owned by this user
    dataset     Number      List images that are children of this dataset
    orphaned    Boolean     If true, only return images not contained
                            within a dataset or Well. Default False
    embed       String      List of objects to include.
                            E.g. ?embed=thumbnail,pixels.sizeX,pixels.sizeY

**Response**

    {
      data: [
        {
          id: 16783,
          owner: {
            id: 3
          },
          name: "test.dv",
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          fileset: {
            id: 9751
          },
          acquisitionDate: "2009-02-27T10:25:32Z",
          creationDate: "2015-10-22T11:57:06Z",
          pixels: {
            sizeX: 512,
            sizeY: 512,
            sizeZ: 29
          },
          thumbnail: {
            version: 0
          },
        }
      ]}


Tags
====

Tags can be organised into Tag sets which are simply Tags with a
defined namespace. Listing Tags will give a collection that may
include Tags and Tag sets. Tag sets will have ```tagset: true``` attribute.

List Tags
---------

    GET   /tags/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    group       Number      Only return Tags from this group
    owner       Number      Only return Tags owned by this user
    tagset      Number      List Tags that are children of this Tag set
    orphaned    Boolean     If true, only return Tags not contained
                            within a Tag set. Default False


**Response**

    {
      data: [
        {
          id: 655,
          owner: {
            id: 3
          },
          tagset: false,
          permissions: {canEdit: true, canAnnotate: true, canLink: true, canDelete: false}
          value: "Control",
          childCount: 5
        }
      ]
    }


List Tagged Objects
-------------------


    GET   /tagged/:tagId/


**Parameters**

    Name        Type        Description
    ------------------------------------------------------------------
    owner       Number      Only return objects owned by this user


**Response**


    {
      data: {
        projects: [],
        datasets:[],
        images: [],
        screens: [],
        plates: [],
        runs: [],
        tags: []
      }
    }


Linking and Unlinking Objects
=============================

Links between the following objects can be created or deleted:

 - project and dataset
 - dataset and image
 - screen and plate
 - Tag / Tag set and Tag
 - Tag and:
   - project
   - dataset
   - image
   - screen
   - plate 


Create Links
------------

    POST  /links/


**Input**

You need to provide a list of link objects to create.

    Name        Type        Description
    ------------------------------------------------------------------
    parentType  String      One of project, dataset, screen, tag, tagset
    parentId    Number      Id of the parent object
    childType   String      One of dataset, image, plate, tag
    childId     Number      Id of the child object


**Example**

    {
      links: [
        {
          "parentType": "project",
          "parentId": 10,
          "childType": "dataset",
          "childId": 5
        },
        {
          "parentType": "tagset",
          "parentId": 42,
          "childType": "tag",
          "childId": 49
        }
      ]
    }


**Response**

    {
      data: [
        {
          "id": 879,
          "parentType": "project",
          "parentId": 10,
          "childType": "dataset",
          "childId": 5
        },
        {
          "id": 668,
          "parentType": "tagset",
          "parentId": 42,
          "childType": "tag",
          "childId": 49
        }
      ]
    }


Delete Links
------------

Links between objects can be deleted in the same way as created.
In this case, we return a list of any links that remain on the child objects
so that we can tell if they have become Orphaned.

    DELETE  /links/


**Input**

    # Links defined as above for Create Links


**Response**

    {
      data: [
        {
          "id": 879,
          "parentType": "project",
          "parentId": 10,
          "childType": "dataset",
          "childId": 5
        }
      ]
    }





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



