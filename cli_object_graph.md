CLI Object/Graph handling
=========================

This document describes the extension of the existing CLI plugins to have a better coverage of object support including trees and graphs

Initial implementations
-----------------------

In addition to the existing CLI obj plugin used for manipulating objects, see
the [CLI children plugin](https://github.com/openmicroscopy/openmicroscopy/pull/4182).

Proposed actions
----------------

** Moving orphaned images into a dataset **

    omero obj move //Image:* Dataset:1

** Linking one image from one dataset to another **


    omero obj copy Dataset:1/Image:1 Dataset:2

** Linking all images from one dataset to another **

    omero obj copy Dataset:1/* Dataset:2


** Adding all ROIs into a folder **

    omero obj cp Image:1/Roi:* Folder:1
    omero obj cp Screen:1/Roi:* Folder:2
    omero obj cp Screen:1/*/Roi:* Folder:2

     