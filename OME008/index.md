# CLI ROI handling

A plugin for the CLI to handle ROI operations.

## ROI plugin

This plugin could include a set of subcommands. A read command

    bin/omero roi get [image id] --folder=[folder id]
    
    output> [shape defs]

and a write command, which adds either a single ROI or a batch of ROIs
specified in a text file:

    bin/omero roi add [image id] --folder=[folder id] [shape def]
    bin/omero roi add [image id] --folder=[folder id] --roi-per-shape --batch-file=[shape defs file]
    
    output> [roi id(s)]

## Input specification for `add` command

`image id`: The ID of the image to get/add ROIs to/from (long)

`folder id`: Optional. The ID of a ROI folder to get/add ROIs to/from (long)

`shape specs file`: Path to a text file holding the Shape specifications (string)

`shape spec`: Shape specification (see below)

`roi-per-shape`: Flag to create a ROI per shape (otherwise add all shapes to the same ROI)


## Shape specification
[Shape | String | Ellipse/Line/Point/Polygon/Rectangle]

[Z | String | (see below)]

[T | String | (see below)]

[X/X_From | Integer | 0/1/...] 

[Y/Y_From | Integer | 0/1/...] 

[Width/RadiusX/X_To | Integer | 0/1/...] 

[Height/RadiusY/Y_To| Integer | 0/1/...]

[X_n | Integer | 0/1/...] (for polygons)

[Y_n | Integer | 0/1/...] (for polygons)

...

### Z/T plane(s) specification

For specific Z/T planes: Comma separated list of Integers.

For a range of Z/T planes: Two Integers, dash separated [From]-[To] , where
[From] and [To] can be empty (in which case [From] will be 0, and [To] will
be the maximum Z/T plane)


On one line with space delimited fields and in case of a batch file, one shape per line.


Examples:

Circle ROI with center (100,100) to radius 10, on z/t plane 0/0: 'Ellipse 0 0 100 100 10 10'

Line ROI from Point (0,0) to (100, 100) on z planes 5-10 and t plane 5: 'Line 5-10 5 10 10 100 100'

Point ROI at (10,10) on z plane 10 to max and t plane 0-10: 'Point 10- -10 10 10'

Polygon ROI with Point (10,10), (20,20) and (30,30) on all z/t planes: 'Polygon - - 10 10 20 20 30 30'

Rectangle ROI at (10,10) with width and height 5, on first two z/t planes: 'Point 0,1 0,1 10 10 5 5'


## Output specification for `add` command

In case of a single ROI addition just output the ROI ID.

In case of batch addition: If `--roi-per-shape` then a list of ROI IDs, otherwise a 
list of shape IDs. In order to be able to match output IDs with the batch shape list,
keep the order and add '-1' (or None, N/A, etc.) in case a shape addition failed.

