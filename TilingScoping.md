Bio-Formats 5.3 Scoping - Tiling

This document is designed to provide an initial evaluation of the current tiling options within Bio-Formats 5.2 and act as a starting point for discussion on solutions for Bio-Formats 5.3

Current scenario
A full breakdown of the current status of tiling implementation in each reader and writer can be found here:  Bio-Formats 5.2 Tiling Status

API
The main point for providing the reading and writing of tiles is provided via the Bio-Formats API

Every Reader and writer implements the following methods:
byte[] openBytes(int no, byte[] buf, int x, int y, int w, int h)
void saveBytes(int no, byte[] buf, int x, int y, int w, int h)

We also provide functions for returning suitable Width and Height parameters for tiling. These are largely only used by Tiff readers and will return the full image height or width unless the tiff tag TILE_WIDTH is set.
int getOptimalTileWidth()
int getOptimalTileHeight()

Below is an example of using the API for tiling which we previously provided:
int tileWidth = 1024;
int tileHeight = 1024;

for (int series=0; series<reader.getSeriesCount(); series++) {
  reader.setSeries(series);
  writer.setSeries(series);
  int tileRows = reader.getSizeY() / tileHeight;
  int tileColumns = reader.getSizeX() / tileWidth;

  for (int image=0; image<reader.getImageCount(); image++) {
    for (int row=0; row<tileRows; row++) {
      for (int col=0; col<tileColumns; col++) {
        // open a tile - in addition to the image index, we need to specify
        // the (x, y) coordinate of the upper left corner of the tile,
        // along with the width and height of the tile

        int xCoordinate = col * tileWidth;
        int yCoordinate = row * tileHeight;
        byte[] tile = reader.openBytes(image, xCoordinate, yCoordinate, tileWidth, tileHeight);
        writer.saveBytes(image, tile, xCoordinate, yCoordinate, tileWidth, tileHeight);
      }
    }
  }
}
Other than through the API there are also limited tiling options available when using the command line tools or through the ImageJ plugin:


Command Line Tools
When using command line tools tiling options are provided to specify the size of the tiles required. The default tile size is determined by the input format, and can be overridden like this:

bfconvert -tilex 512 -tiley 512 /path/to/input output-512x512-tiles.tiff

Not all formats will support this tiling and specific parameters must be used. This information is included as part of the documentation provided on converting images using the command line.


Stitching Existing Tiles - ImageJ Plugin
The ImageJ plugin has the ability to stitch existing tiles together using simple non overlapped stitching. Alternative plugins are recommended are more detailed tiling solutions.

The option for stitching is provided through the Import Dialog. If selected then the ImageJ plugin uses a TileStitcher wrapper around the ImageReader to stitch. It does not provide the capabilities for exporting tiles.




Summary of Existing Limitations

Positives:
The current functionality provides strong support for the reading of tiles.
We also a suitable wrapper for the stitching of existing tiles, although this is currently only used by the ImageJ plugin it could be leveraged or better documented elsewhere.

Issues:
As each reader and writer implements the functions used for tiling, it gives the impression to the user that tiling will be supported for all formats and with any provided parameters.
No documentation or functionality is available to inform a user which formats support tiling and which do not.
No documentation is provided to explain how a user can find suitable tile sizes.

Limitations:
Very few of the writers support a complete tiling implementation.
Though our readers support the reading of tiles a large number of them do so by reading the entire plane and performing an array copy, thus negating a lot of the benefit of tiling and in fact making it much more resource intensive if calling the function multiple times.
Many of the writers also need to write a full blank plane first before writing a tile.

Improvements:
The getOptimalWidth and Height functions are only really supported through readers implementing MinimalTiffReader, the functionality is also poorly documented. This could be rolled out further and should be included in any documentation.
Some of the writers support partial tiling, ie writing of smaller regions but requiring them to be written in the correct order. If this is a limitation of the format then it should be better documented and feedback should be provided to the users.
Others will require certain parameters for width and height to be used without any way for the user to discover this information. The existing getOptimalWidth and Height functions could be used a basis for providing this information.
Other writers will ignore the user provided parameters to use hard coded settings. 



Possible Solutions

Option1 - Existing API, improved documentation

As the existing API is implemented across all existing readers and writers, it may be that the cost of reworking it is too large. As such it may be best to keep the existing functions and improve upon them.

The current readers largely support tiling and require little change. The main issues are with regards the writers and documentation of current support levels.

How can users discover if a format supports tiling?
We currently have no way to provide this information to users. We could improve our current handling by providing the following feedback:

Improve documentation regarding supported formats
Use isFullPlane to return suitable exceptions when a format is not supported
Implement a new function in FormatWriter to allow the user to query tiling support
boolean supportsTiling()

How can users find valid parameters for tiling?
The other major issue we currently have is that we have no way of informing the users on which parameters are suitable. Adding to this is the fact that not all writers make use of the user provided parameters. Improvements to be made here include:

Make greater use of existing optimalWidth and Height,
Improve documentation around these functions
Fix existing methods to make full use of user provided parameters

Pros		
Reduces the effort required to deliver a solution
Keeping the existing API means we will not be making breaking changes

Cons
Functions are still accessible for all writers even if not supported


Option2 - New TiledWriter Interface

Reader Functionality - Keep existing API
As the existing API function for reading a tile is implemented across the board to a good standard I would propose that we keep the existing reader API. 

byte[] openTile(int no, byte[] buf, int x, int y, int w, int h)
int getOptimalTileWidth()
int getOptimalTileHeight()
 
Deprecate Existing Writer API function 
As the writing of tiles is only supported by a subset of our writers I would suggest deprecating the existing API function.

void saveBytes(int no, byte[] buf, int x, int y, int w, int h) 

New TiledWriter Interface
For format writers which do support tiling a separate interface to handle the writing of tiles could be implemented by this subset. The functionality from the existing saveBytes would be transferred (and improved to make full use of parameters) to a new saveTile function. Additionally, I would suggest adding new functions to retrieve suitable height and width (similar to the reader API) for supported format writers.

TileWriter


+void saveTile(int no, byte[] buf, int x, int y, int w, int h)
+int getOptimalTileHeight()
+int getOptimalTileWidth()

getTileHeightMultiple()
getTileWidthMultiple()

Setting tile sizes in advance

Maximum width and height

Extra dimensions - chunk writing

Benchmarking

Documentation would be required to demonstrate the new tiling interface

Pros
Users can use ‘if(writer instanceof TileWriter)’ to determine if a writer supports tiling
Keep existing Reader functionality reduces the level of breaking changes and effort
Functions are only available when they can be used

Cons
Breaking API changes
API mismatch between readers and writers (openBytes missing a corresponding saveBytes)
Larger effort required to deprecate existing functions and improve use or feedback of users parameters
