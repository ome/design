# OME-TIFF sub-resolution support

## Introduction

There have been several different proposals for images at different
scales in the form of sub-resolutions (image “pyramids”) for TIFF and
OME-TIFF in Bio-Formats and OME Files, which include:

- [Storage of pyramid data in
  OME-TIFF](https://github.com/openmicroscopy/design/issues/74)
  (Melissa Linkert / Glencoe)
- [Use of
  SubIFDs](https://github.com/openmicroscopy/bioformats/pull/2747#issuecomment-278966356)
  (Roger Leigh)
- [TIFF/OME-TIFF extension to support
  pyramids](https://www.openmicroscopy.org/community/viewtopic.php?f=15&t=8433)
  (Damir Sudar)

Alternative existing approaches include:

- [GeoTIFF](http://www.cogeo.org/in-depth.html) and its
  [specification](https://trac.osgeo.org/gdal/wiki/CloudOptimizedGeoTIFF)
  (Andrew Brooks)
- [GeoPackage](http://www.opengeospatial.org/standards/geopackage),
  images and extra metadata stored in an SQLite database (Andrew
  Brooks).  While a very interesting concept, and something to
  investigate in more detail for future work, it's not within the
  current scope of a TIFF container format.

This proposal will summarise the various possible approaches and their
tradeoffs, including the practical implementations I have tested while
evaluating them.

## Storage

There are several strategies we could employ for sub-resolutions:

![Strategies A, B and C](strategy-a-b-c.svg)

![Strategy D](strategy-d.svg)

![Strategy E](strategy-e.svg)


### A. Implicit ordering

This is the approach taken by the existing Pyramid TIFF reader

Pros:

- Exceedingly simple
- Any software can read and write this structure
- No data model changes required

Cons:

- No reductions in *z* (unless using libtiff extensions)
- Difficult to support multi-series files without additional tags
  (`NewSubfileType`) and/or semantics (size comparisons) to
  differentiate which IFD belongs to which series
- Not compatible with OME-TIFF multi-file structures

GeoTIFF is almost the same as the design shown here, with the
exception that the sub-resolution level order is reversed, starting
with the smallest sub-resolution and ending with the full resolution.
It has the same pros and cons.

### B. SubIFDs pointing to main IFDs

Intermediate between (A) and (C).  The `SubIFDs` tag is used to indicate that other IFDs are sub-resolutions of this IFD.  The other IFDs are part of the main IFD list, like (A).

Pros:

- Most software can read and write this structure
- No data model changes required

Cons:

- No reductions in *z* (unless using libtiff extensions)
- Sub-resolutions are still part of the main IFD list
- Unlikely to be supported by libtiff

### C. SubIFDs pointing to separate IFDs

This is how sub-resolutions in TIFF are ideally supported.

Pros:

- It’s the standard way to do sub-resolutions with TIFF
- Compatible with Photoshop sub-resolution storage
- No data model changes required
- Supported natively by libtiff (easy to write and read)
- Sub-resolutions not in the main IFD list, so are not visible to
  software which doesn’t handle `SubIFDs`

Cons:

- No reductions in *z* (unless using libtiff extensions)
- Harder to use in general (less software support)
- Sub-resolutions not visible to software which doesn’t handle `SubIFDs`


### D. External metadata with implicit resolution order

Similar to (A), but instead of using the `SubIFDs` tag the resolution
count is specified in the Image OME-XML metadata.

Pros:

- Exceedingly simple at least at first glance
- Most software can read and write this structure

Cons:

- No reductions in *z* (unless using libtiff extensions)
- Difficult to support multi-series files without additional tags
  (`NewSubfileType`) or semantics (size comparisons) to differentiate
  which IFD belongs to which series
- Sub-resolutions are still part of the main IFD list
- Unlikely to be supported by libtiff
- The implicit ordering is very fragile, and it will be easy to
  silently read the wrong plane if the assumptions are broken or there
  is an error in the implementation
- Not compatible with OME-TIFF multi-file structures
- Requires data model changes

### E. External metadata with explicit resolution order

Similar to (A), but instead of using the SubIFD tag the
sub-resolutions IFDs are specified in the Image OME-XML metadata

Pros:

- Any software can read and write this structure
- Can support reductions in *z*
- Can support OME-TIFF multi-file structures

Cons:

- Requires complex data model changes
- Further complicates the already complicated OME-TIFF reader logic


Overview of the various strategies:

Strategy                              | A    | B        | C        | D         | E
------------------------------------- | ---- | -------- | -------- | --------- | ---------
SubIFDs usage                         | None | Simple   | Full     | Optional‡ | Optional‡
*z* reduction                         | No   | No       | No       | No        | Yes
libtiff compatibility                 | Yes  | No       | Yes      | Yes‡      | Yes
Photoshop compatibility               | No   | No       | Yes      | No        | No
Sub-resolution access without SubIFDs | Yes  | Yes§     | No       | Optional‡ | Optional‡
Reading portability                   | High | Mid      | Low†     | High      | High
Writing portability                   | High | Mid      | Low      | High      | High
Implementation complexity             | Low  | Middling | Middling | High      | High
Model changes                         | No   | No       | No       | Simple    | Complex
Multi-series OME-TIFF                 | No*  | Yes      | Yes      | Yes*†     | Yes
Multi-file OME-TIFF                   | No*  | Yes      | Yes      | Yes*†     | Yes


\* Implementation complexity very high, with a great deal of potential fragility

† Would still be readable by all software, but without sub-resolutions

‡ `SubIFDs` could be used independently of additional metadata, but
  would have to refer to IFDs in the main sequence since the metadata
  will access them by index.  [If the metadata allowed access by
  offset, then proper `SubIFDs` directory entries could also be used.]

§ Accessible, but without any metadata to indicate the structure


We will implement strategy B or C in the short term.  In the longer
term, E would allow *z* reductions (or requiring HDF5 might avoid the
need for any model changes).

So long as `bfconvert` and the other Bio-Formats tools allow for
sub-resolution flattening on conversion, the native support for
separate `SubIFDs` which aren’t part of the main IFD sequence should not
be a hindrance to inter-operability.  If there is a need to access
sub-resolutions in tools without support for `SubIFDs`, `bfconvert` can
create a suitably flattened TIFF.  If we wanted to create TIFFs with
an increased range of portability, we could implement strategy B.  If
proper `SubIFDs` support was acceptable then strategy C would be
preferable.  Since strategy C is what libtiff supports natively and is
what OME-Files C++ would write, having compatible behaviour between
the C++ and Java implementations would be preferable.

SubIFD only supports reductions in *x* and *y*, not in *z* (unless
using libtiff extensions).  This will meet all the 2D cases, including
digital pathology.  For reductions in *z*, we would require support
for sub-resolutions in the metadata model.  However, this is a
TIFF-specific limitation which would not apply to e.g. HDF-5.  It
might be acceptable to require HDF-5 for this feature rather than add
TIFF-specific features to the data model.

Note that implementing strategy B or C doesn't preclude implementing
strategy E independently. If OME-XML metadata describing
sub-resolutions exists, we can simply ignore the `SubIFDs`
metadata. `SubIFDs` could remain for readers which aren't capable of
using the OME-XML metadata.  Strategy B would potentially ease the
implementation of strategy E, since the top-level IFDs can be reused.
However, the caveats with libtiff remain.  Since we could retain
strategy B or C for the plain TIFF writer, we would get sub-resolution
support essentially "for free" in the OME-TIFF writer irrespective of
the addition of support for strategy E.  Strategy E will require full
support in the data model for TiffData (or equivalent) elements for
every resolution level.

## TIFF and OME-TIFF file format changes

This is based largely on Damir Sudar’s suggestions

- The TIFF extension tag `SubIFDs` must be used to specify
  sub-resolution image directories.  The reducedimage bit of the
  Baseline tag `NewSubfileType` must be used to distinguish
  full-resolution images from reduced-size images; the page bit may
  optionally be set when appropriate
- BigTIFF is recommended for large images, while Baseline TIFF may
  suffice for smaller images
- With the exception of the above BigTIFF and `SubIFDs` usage, all TIFF
  files with sub-resolutions must be Baseline TIFF-compliant and
  readable with any reader capable of reading Baseline TIFF
- There are no changes to compression or tiling support
- Sub-resolution images may chose to use different compression
  algorithms than used by the full resolution image, for example the
  full resolution image may use no compression or lossless compression
  while the sub-resolution images use lossy compression
- Reader considerations
  - Readers should check `NewSubfileType` and ignore reduced images
    unless configured otherwise
  - Readers wishing to access sub-resolutions must query `SubIFDs` and
    check the available reduced sizes, which may then be presented to
    the user via the existing sub-resolution reader API
  - Readers must be able to handle the linked `SubIFDs` being present as
    separate IFDs or as part of the main IFD sequence; this should be
    the case if (a) and (b) are implemented correctly
- Writer considerations
  - Writers must set both `SubIFDs` and `NewSubfileType`
  - Writers should create separate `SubIFDs`, but may create them as
    part of the main IFD sequence when required, for example by
    software or inter-operability constraints
  - Writers should provide power-of-two reductions until one of the
    smallest x or y dimension reaches a size of 256 or less.
    Non-power-of-two reductions are supported, as is a size other than
    256 as the ending size.
- Sub-resolution support as described above are supported for plain
  (Baseline) TIFF.  Sub-resolutions must also work transparently with
  OME-TIFF without any additional OME-XML metadata.

Follow-up work could include:

- Modelling of sub-resolutions in the OME data model (currently being
  worked on)
- Hooking up the sub-resolution model to the sub-resolution API
- Generation of OME-TIFF with sub-resolutions described in the data
  model, including *z* reductions, with pointers to the per-plane
  reduced images
- Sub-resolution support in the model could be built on top of native
  support in the TIFF container, and could be optional for the simple
  case without reduction in *z*

### Outstanding questions

- Do we need to store metadata to describe alignment offsets between
  sub-resolution levels when the size difference between levels
  results in pixels not aligned on pixel boundaries, for example with
  non-power-of-two reductions? How is this handled by the existing
  pyramid file formats? (Question from 2018 OME annual meeting in
  Dundee.)

### Sample files

Simple scripts to convert existing file formats with sub-resolutions to
TIFF and OME-TIFF files with SUBIFDS have been created for testing
purposes:

- [makepyramid-ndpi](makepyramid-ndpi)
- [makepyramid-scn](makepyramid-scn)
- [makepyramid-svs](makepyramid-svs)

Of the three, `makepyramid-scn` generates the most compliant OME-TIFF
files with the best tile sizes and compression types.  These will be
used to test the TIFF and OME-TIFF support for sub-resolutions in
Bio-Formats and OME Files prior to the creation of a writer which can
generate the files directly.

Note that the scripts require a copy of Bio-Formats `showinf`
on the `PATH`.   They also require a copy of libtiff on
`LD_LIBRARY_PATH` and `tiffinfo` and `tiffset` on the `PATH`. libtiff
must be a release > 4.0.9 for BigTIFF SUBIFDS support in `tiffset`;
at the time of writing this means building a copy from git.


## Bio-Formats and OME-Files API and implementation changes

### Existing sub-resolution API

Implemented only for reading

| IFormatReader           | Description
| ----------------------- | --------------------------------------------------
| seriesToCoreIndex       | Convert from series (resolution 0) to linear index
| coreIndexToSeries       | Convert from linear index to series (resolution 0)
| setCoreIndex            | Set linear index (series and resolution)
| getCoreIndex            | Get linear index (series and resolution)
| getResolutionCount      | Get resolution count for current series
| setResolution           | Set resolution for current series
| getResolution           | Get resolution for current series
| hasFlattenedResolutions | True if resolutions are flattened to series
| setFlattenedResolutions | Enable resolution flattening (before setId)

| FormatReader         | Description
| -------------------- | -----------------------------------------------
| resolution           | Field storing current resolution level
| flattenedResolutions | Field storing whether resolutions are flattened

The implementation in `FormatReader` maintains the current resolution
level for the active series, and whether or not resolutions are
flattened (which affects the behaviour of the "core index" methods).

### Proposed sub-resolution writer API additions

The writer implementation needs to keep track of the number of
resolution levels in the current series, and the current resolution in
the current series.

The "core index" methods and "flattened resolutions" reader methods
are not required for writing, because these are specific to the reader
implementation and the `CoreMetadata` class which the writer interface
lacks.

The writer will need to be able to set the sub-resolution metadata and
switch between resolution levels via the writer API.  To set the
current resolution, similarly to the current series, the following
additions are required:

| IFormatWriter      | Description
| ------------------ | ---------------------------------------
| setResolution      | Set resolution for current series
| getResolution      | Get resolution for current series
| getResolutionCount | Get resolution count for current series

| FormatWriter    | Description
| --------------- | --------------------------------------
| resolution      | Field storing current resolution level

There are two strategies which could be used to implement setting of
the sub-resolution metadata.  Firstly, specification of all metadata
via writer methods to match every reader method:

| IFormatWriter      | Description
| ------------------ | ------------------------------------------------------------
| setResolutionCount | Set resolution count for current series
| setSizeX           | Set size X for current series and resolution
| setPixelType       | Set pixel type for current series and resolution
| setInterleaved     | Set interleaved for current series and resolution
| …                  | Etc. for all series/resolution properties from IFormatReader

| FormatWriter    | Description
| --------------- | -------------------------------------------------------------------
| core            | Field for storing core metadata list for all series and resolutions

Here, `setResolutionCount` would add the extra `CoreMetadata` elements
to describe the extra sub-resolutions for the current series (can be a
copy of the current series core metadata).  Then, `setSizeX`,
`setSizeY` and other methods would be used to customise the
sub-resolution metadata.  The downside of this approach is that the
metadata needs to be fully specified before `setId` is called, or else
the internal writer state will be inconsistent.  `setSeries` can't be
called before `setId`, and so this approach is incompatible with the
current writer semantics.

Secondly, by setting the core metadata list:

| IFormatWriter       | Description
| ------------------- | -------------------------------------------------------
| setCoreMetadataList | Set the core metadata list (all series and resolutions)
| getCoreMetadataList | Get the core metadata list (all series and resolutions)


| FormatWriter    | Description
| --------------- | -------------------------------------------------------------------
| resolution      | Field storing current resolution level
| core            | Field for storing core metadata list for all series and resolutions

This can be done before `setId` and remain compatible with the
existing writer API.  `setCoreMetadataList` could be used as an
alternative to `setMetadataRetrieve`, or after `setMetadataRetrieve`
by calling `getCoreMetadataList` to get the core metadata set from the
metadata store, and then modifying it before calling
`setCoreMetadataList`.

| TiffWriter          | Description
| ------------------- | -----------------------------
| prepareToWriteImage | Set `SUBIFDS` to correct size
| saveBytes           | Update SUBIFDS

`prepareToWriteImage` will need to set the `SubIFDs` tag to a size of
`resolutionCount - 1` if the number of resolutions is greater than 1.
`saveBytes` will write the currently selected sub-resolution level.
The `SubIFDs` offsets will require updating after each sub-resolution
is written; the offset to the `SubIFDs` arrays will require caching.

The main implementation choice to make is whether the `IFormatWriter`
and `FormatWriter` changes above should be made in these places, or if
they should be restricted to `TiffWriter` for the initial work.
Having them exposed makes them resusable in other writers and allows
for clean integration of sub-resolution functionality into tools like
`bfconvert`.  However, it is then a visible public API addition.
Keeping it hidden avoid this, but at the cost of accessing it
requiring hardcoding of writer-specific special cases.

### Proposed sub-resolution reader changes

| MinimalTiffReader | Description
| ----------------- | -----------------------------------------
| initFile          | Check `SUBIFDS` and set `CoreMetadata`
| openBytes         | Use current sub-resolution from `SUBIFDS`

`initFile` will check the `SubIFDs` tag, and initialise the CoreMetadata
with the sub-resolution data.  `openBytes` will read the selected
sub-resolution level (no changes required).

If we wish to write TIFF and OME-TIFF with sub-resolutions without any
corresponding data model changes, we can make use of the existing
sub-resolution API support in `IFormatReader`.  If a corresponding set
of methods were added to `IFormatWriter`, this would provide the basis
for specifying the number of resolution levels, setting the current
resolution level and writing pixel data for a specific resolution
level.  Since this uses `SubIFDs`, no metadata model changes would be
required.  With the corresponding reader support, this would provide
transparent support for reading, writing, and conversion of data files
containing sub-resolution data.

### Implementation of writing support

Writing can be broken down into these steps, which can be implemented in order:

- `CoreMetadata` support for subchannel sizes
- `MetadataTools` support for subchannels in `populatePixels`
- `FormatWriter` and `IFormatWriter` support
- `TiffSaver` support
- `OMETiffWriter` support
- `bfconvert`/`ImageConverter` support

These steps are partially implemented on several git branches:

| Branch | Description
| ------ | ---------------
| [coremetadata-subchannel](https://github.com/rleigh-codelibre/bioformats/tree/coremetadata-subchannel) | Add `sizeSubC[]` subchannel sizes to `CoreMetadata`
| [format-writer-subresolution](https://github.com/rleigh-codelibre/bioformats/tree/format-writer-subresolution) | Serialise `MetadataRetrieve` to `CoreMetadataList`; add `SubresolutionFormatWriter`
| [imageconverter-noflat](https://github.com/rleigh-codelibre/bioformats/tree/imageconverter-noflat) | Add `bfconvert` `-noflat` option
| [ometiff-pyramid-writer](https://github.com/rleigh-codelibre/bioformats/tree/ometiff-pyramid-writer) | More `bfconvert` support and `ISubResolutionFormatWriter`

The rationale for these steps is detailed here.

In order to set `CoreMetadataList` from `MetadataRetrieve`, we need to
be able to round-trip all needed metadata, including all metadata
needed for sub-resolutions.  The key defect here is that the OME-XML
`Channel` class' `SamplesPerPixel` attribute isn't representable in
`CoreMetadata`.  There's only `sizeC` and `imageCount`.  OME-Files C++
`CoreMetadata` stores `sizeC` as an array:

```
/// Number of channels.
std::vector<dimension_size_type> sizeC;
```

The size of the vector is the channel count; the size of each element
is the number of samples for that channel index.  Bio-Formats needs to
be able to store the same information.  The `coremetadata-subchannel`
branch is the start of the needed work.  It adds `sizeSubC` as a
supplement to `sizeC` (leaving `sizeC` present but unused for backward
compatibility).  The steps to do this are:

- Add `sizeSubC`.
- Comment out `sizeC`.
- Add `int getRGBChannelCount(int)` to get the sub-channel count for a
  given channel index (is compatible with OME Files C++).
- Add `int[] getRGBChannelCounts` to get all sub-channels directly (to
  make copying and transfer more direct).
- Convert all logic in `FormatReader` and all readers to use
  `sizeSubC` in place of `sizeC` (commenting out `sizeC` usage).
  All RGB channel count usage can get the subchannel count directly;
  no need to use bad assumptions dividing the image count by
  `sizeZ * sizeT` etc.; we have the actual correct numbers to hand.
- Restore `sizeC` and `sizeC` setting, but no `sizeC` getting should
  take place.
- Deprecate `sizeC` for later removal.
- Deprecate `imageCount` for later removal; can be computed directly
  as the sum of `sizeSubC` values.  Comment out as for `sizeC` to
  eliminate all uses before adding back.

Once the `CoreMetadata` changes are in place, we can create a
`SubResolutionFormatReader` to contain a `CoreMetadataList` and to
fill it from `MetadataRetrieve` when set, or alternatively from
`setCoreMetadataList` directly.  At this point, it's possible to
specify all series and resolutions before `setId`.  This work is begun
on the `format-writer-subresolution` branch.  The writer
implementation should strictly enforce correct series, resolution and
plane progression, to ensure correct ordering in the file being
written.

The TIFF saving code needs updating to allow saving of sub-resolution
planes in `SUBIFDS`.  I would suggest updating `TiffSaver` to maintain
a stack of sub-resolutions and to write them in order like libtiff.
This means it will have to maintain this state:

- `previous` top-level IFD offset (for updating the next pointer
  tranparently); currently requires specifying up front, which is
  buggy and the cause of several bugs, causing invalid TIFFs to be
  written with `next` pointers to nowhere.
- `SUBIFDS` offset and size and current SubIFD for current IFD.  This
  will allow automatic update of the parent `SUBIFDS` IFD offset when
  the current IFD is written, and for all subsequent `SUBIFDS`.  This
  needs to stack to cater for `SUBIFDS` containing `SUBIFDS`, i.e. a
  tree structure.

Next, we can make individual writers implement
`SubResolutionFormatWriter`.  Given the small number of writers, I
would suggest converting the entire lot for ease of maintenance
(writers don't need to use subresolutions if they implement the
interface).  Then we can update `OMETiffWriter` to write pyramids.
Given the current series and current resolution information in the
inherited writer implementation, the only alteration is to `saveBytes`
to set the correct `SUBIFDS` size when writing out the full resolution
plane, then writing each resolution sequentially and updating
`SUBIFDS` accordingly (if `TiffSaver` is made stateful this can be
automatic).  The writer should set `NEWSUBFILETYPE` appropriately
to full resolution or reduced-resolution image.  Also bitwise OR PAGE if
the image is a multi-plane series.

Lastly, update `ImageConverter` to set the resolutions in
`CoreMetadataList` and use `setResolution` in addition to `setSeries`
to loop over each resolution as well as each series and transfer all
the resolution levels.  The `imageconverter-noflat` and
`ometiff-pyramid-writer` branches implement most of the needed logic.

