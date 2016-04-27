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

## Tree plugin

This plugin could include a set of subcommands. Each subcommand could either
be a read operation applied to a graph specified by the `graph-spec` argument:

    bin/omero tree read-operation graph-spec

or a write operations applied to a `graph-spec` argument with an optional
`object-spec` target:

    bin/omero tree write-operation graph-spec [obj-spec]

## Input specification

There are 2 proposed formats that could be supported when specifying the input
`graph-spec`:

-   either stay close to the current `obj-spec` format, i.e. `Object:id`
-   or move towards xpath-like formats e.g. `//Image/*`, `Image[name=xxx]`, `Screen:id/*/Roi:*`

If sticking to the first option, the proposed representation of orphaned
objects would be `Image:orphan`.

## Tree subcommands

Below is a list of proposed subcommands with use cases.

### Link

Move orphaned images into a dataset:

      omero tree link Image:orphan Dataset:1
      omero tree link //Image:* Dataset:1

Link images in between datasets

    omero tree link Dataset:1 Dataset:2
    omero tree link Dataset:1/Image:* Dataset:2

Link all Regions of interest to a folder

    omero tree link Image:1/Roi:* Folder:1
    omero tree link Screen:1/Roi:* Folder:2
    omero tree link Screen:1/*/Roi:* Folder:2

### Unlink

Orphan all images in a dataset

    omero tree unlink Dataset:1

### Copy

Link all images in one dataset into another

    omero tree copy Dataset:1 Dataset:2
    omero tree copy Dataset:1/* Dataset:2

### List

List containers/folders content

    omero tree ls Dataset:1
    omero tree ls Folder:1

List orphaned objects

    omero tree ls --orphan
    omero tree ls Image:orphan


List objects with filter

    omero tree ls Screen:1 --filter FileAnnotation[ns='bulk-annotation']
    omero tree ls Screen:1/*/FileAnnotation[ns='bulk-annotation']

Return the object identifiers only

    omero tree ls Dataset:1 --ids

Format the returned objects in a YML style

    omero tree ls Dataset:1 --output=yml
