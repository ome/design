# CLI graph/tree handling

This document scopes the extension of the OMERO Command Line Interface to
have a better support for trees and graphs.

## Context

We already introduced the CLI [obj plugin](http://www.openmicroscopy.org/site/support/omero5.2/developers/cli/obj.html) to manipulate individual objects. The generic syntax of this plugin is of
the form:

    bin/omero obj subcommand  Object:id field1=value1 field2=value2 ...

We are facing more use cases where we need to manipulate graphs and sets of 
objects. A first implementation of such plugin has been provided in the context
of IDR with the [CLI children plugin](https://github.com/openmicroscopy/openmicroscopy/pull/4182).

The current consensus is to preserve the `obj` plugin for the manipulation of
individual objects and design a new plugin for graph/set tasks. For now, we will use the term `tree` to refer to this new plugin.

This plugin could include a set of subcommands so that individual calls would take the form:

    bin/omero tree subcommand source [target]

## Proposed subcommands

Below is a list of proposed subcommands with use cases

### Move

#### Moving orphaned images into a dataset

    omero tree move Image:orphan Dataset:1
    omero tree move //Image:* Dataset:1

#### Moving  images in between datasets

    omero tree move Dataset:1 Dataset:2
    omero tree move Dataset:1/Image:* Dataset:2

#### Move all Regions of interest into a folder

    omero tree move Image:1/Roi:* Folder:1
    omero tree move Screen:1/Roi:* Folder:2
    omero tree move Screen:1/*/Roi:* Folder:2

### Delete

This has to do with unlinking

#### Orphan all images in a dataset

    omero tree delete Dataset:1

### Copy

#### Link all images in one dataset into another

    omero tree copy Dataset:1 Dataset:2
    omero tree copy Dataset:1/* Dataset:2

### List

#### Listing containers/folders content

    omero tree ls Dataset:1
    omero tree ls Folder:1

#### Listing orphaned objects

    omero tree ls --orphan
    omero tree ls Image:orphan

#### Listing objects with filter

    omero tree ls Screen:1 --filter FileAnnotation[ns='bulk-annotation']
    omero tree ls Screen:1/*/FileAnnotation[ns='bulk-annotation']

Open questions
--------------

- orphaned representation (`orphan` string vs option vs graph representation)
- output of the listing commands? for piping the output of the listing into bulk annotation scripts, retrieving a list of ids would be good
- subcommand vs options, i.e. `tree copy` vs `tree --copy`
- naming: link vs copy? move vs cut&paste? delete vs unlink? 
