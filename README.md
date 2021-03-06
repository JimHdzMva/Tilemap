<img src="http://blog.safecast.org/wp-content/uploads/2014/01/header.png?w=460&amp;h=120&amp;crop=1" alt="Safecast">

 

# Tilemap

Code and content for the current production Safecast web map, http://safecast.org/tilemap

 

## Abstract

The Safecast Tilemap is a single-page web application using the Google Maps API v3 to provide visualization and analysis of the Safecast dataset.  This is principally accomplished through the display of standard Web Mercator map tiles.  Other features include a bGeigie log viewer and query interface, a raster cell value lookup tool, and display of the Safecast real-time sensors.

 

 

## Components
<img width="669" height="378" src="https://github.com/Safecast/Tilemap/raw/master/docs/webmap_code_1338x755.png" />

(todo: add `slideout.min.js`)

<img width="669" height="376" src="https://github.com/Safecast/Tilemap/raw/master/docs/webmap_content_1338x752.png" />

 

 

 

 

## Overview - Files
<img width="728" height="546" src="https://github.com/Safecast/Tilemap/raw/master/docs/webmap_files_full_2048x1536.png" />

(todo: add `slideout.min.js`, `menu_tooltips_512x8704.png`)
 

 

 

 

## Overview - Data Flow
<img width="728" height="546" src="https://github.com/Safecast/Tilemap/raw/master/docs/webmap_dataflow_full_2048x1536.png" />

(todo: change `gamma.tar.bz` to `107.161.164.166`)  
(todo: add the large number of snapshot / time slice tile buckets on S3 or make a note of them)

 

 

 

 

## External Dependencies

NOTE: A required component, `bitstore.js`, is not included here.  That's because it already has its own Github repo: https://github.com/Safecast/bitstore.js

 

 

## Data Processing Methodology

Please refer to: http://safecast.org/tilemap/methodology.html

 

 

## Licensing

Individual licenses are displayed at the top of each .js file.  In general, most things here created by Safecast are public domain/CC0.  The Safecast logo and name are not.


#### Reuse

Although this is mostly an internally-focused repository, you are welcome to re-use whatever parts are useful to you.  Attribution is always appreciated, but may not be required depending on the specific licensing for the code/content in question.

Note that thus far, only one component of the tilemap, `bitstore.js`, has been expressly created as a general-purpose library.  Other components are designed specifically to work with Safecast data, and will likely require modification to be used for other things.

The servers used to provide data to the tilemap have finite capacity.  If you use them in your own application, please be curteous of this.

 

 

## Setup and Installation


#### "Compiling"

The references within these files are for minified versions of .js files, typically a _min suffix.  Thus, to deploy, the code here must be minified using a tool such as http://jscompress.com


#### Local Testing Environment

When testing locally, you should use a local HTTP server, rather than file://.  The reason being that almost all code and content is now loaded asynchronously and that requires HTTP

Any local server will work -- Apache, IIS, etc.  Note you may need set permissions on all files appropriately.


#### Server Requirements

There are no particular server requirements for hosting these files, and they may be served from any HTTP server.  However, please see "Domain-Specific References" below.


#### Client Requirements

IE10+, any semi-recent version of Chrome, Safari, Firefox, etc.  Please make sure to account for mobile devices, UI paradigms, and screen sizes with any changes made.


#### Domain-Specific References

1. `index.html` -- has a hardcoded meta origin tag.  This is so things don't break when the QueryString gets long.
2. `safemap.js` -- contains hardcoded absolute URL references to bitstore_min.js and png_zlib_worker_min.js.  Necessary because bitstore.js uses inlined web workers, which require absolute URLs.  You don't have to change these for testing, unless you're using a different version of bitstore.js than the server.


#### Deployments

Please coordinate any production or test deployments with Nick Dolezal.  Thanks!

 

 

## To Do

1. Japanese language support - partially done with new menu UI
2. Per-log metadata display for the bGeigie log viewer
3. Hack-ish spatial query to find logs at a location (which the API does not support a spatial query for)
4. Addition of content from Azby
5. Addition of NNSA Washington DC data/follow-up
6. Further documentation
7. etc.

 

 


# Technical Reference

 

## Main Content Files and Scripts


#### index.html

index.html is the main content page the tilemap is accessed from.  It contains the general UI/layout for the map, and coordinates loading the initial file requirements.  It also contains a number of divs used as databinds for asynchronously loaded content and scripts

In general, heavy use is made of asynchronous and deferred loads to improve initial load times.  This even applies to the images, which were found in testing to delay loading of other scripts.  Much of this is related to the nature of HTTP 1.0/1.1.


#### safemap.js

`safemap.js` is the main "codebehind" for index.html, containing the majority of the scripts used by the tilemap.  It manages all tile content for the map, facilitates UI events, and controls asynchronous loads of other code modules.

As with `index.html`, the codebase relies heavily on asynchronous loading of scripts and content.  This creates a certain degree of code complexity.

To safely handle asynchronous loads of other code modules, proxy classes are used.  Other code should only refer to these proxy classes, and never the objects they retain which may or may not be loaded.

To purely display a tile overlay, no script other than `safemap.js` is necessarily required. (note: as of 2016-09-09, this will prevent user interaction unless `slideout.min.js` is also loaded)

##### A note about maintainability and `safemap.js`

Since its original inception, `safemap.js` has grown considerably in size.  Unfortunately, the nature of HTTP loads and latency make organizing code by file problematic, and for performance, a lot of code has been inlined into `safemap.js` that would not be with native code development, including things such as localized strings for the UI.  The following principles are used to mitigate these issues to some degree:

1. Only essential code that is always loaded goes in `safemap.js`.  Code for optional use components should be loaded asynchronously.
2. Any code that is loaded asynchronously should have a proxy pseudo-class that manages the asynchronous load that all other code interacts with.  This is to guarantee safety and prevent race conditions.
3. All code in `safemap.js` should be organized into pseudo-classes.  (this is currently in-progress for older code)
4. Pseudo-classes should be semi-independent, functional modules, which should be theoretically easily transportable to another file.  Circular references should be avoided.
5. References to other pseudo-classes or globals should be contained in functions at the top of each pseudo-class, which serve as a declaration to help control dependencies.
6. The goal is to avoid a mess of top-level functions in the `window` namespace with unclear dependencies.

`safemap.js` is always loaded.


#### bitstore.js

`bitstore.js` was the first external script created for the tilemap.  It provides automated indexing of raster tiles, implemented in the form of myBitstore.ShouldLoadTile().  This provides a significant performance improvement, especially at higher zoom levels, as the webserver hosting the tiles has much greater latency when a tile request is 404 than when it is present.

`bitstore.js` also updates the layer's date in the UI via a callback function.

`bitstore.js` was implemented as a general-purpose library, suitable for re-use elsewhere.  Unlike the other modules, it contains no dependencies on the Google Maps API, or Safecast dataset.

`bitstore.js` is always loaded.


#### bgeigie_viewer.js

`bgeigie_viewer.js` is perhaps the single most complex component.  It is responsible for the query and display of one or more bGeigie log files, and is capable of displaying a relatively large number of point features / markers on the client.  The complexity arises from the manual marker management and data processing involved to accomplish the degree of scalability it provides.

Despite its complexity, `bgeigie_viewer.js` may be useful elsewhere if modified.  To the best of my knowledge, no other JavaScript library provides anything remotely near its scalability and performance, enabling large point feature datasets to be viewed without a dedicated GIS server.

`bgeigie_viewer.js` is loaded only on-demand.


#### rt_viewer.js

`rt_viewer.js` is currently likely the simplest component to re-use in another map, able to be instantiated and work from a single line of code.  It displays all Safecast real-time sensors reported to it from rt.safecast.org, both online and offline, as markers on the map.  `rt_viewer.js` is a simplified version of `beigie_viewer.js`'s code, rewritten to support the requirements for real-time sensors.

`rt_viewer.js` replaced Google Fusion Tables, which had limited symbology that showed no contextual information about a particular sensor.  `rt_viewer.js` displays the dose rate as a color matching the other map symbology, and indicates online or offline status with a ring around it.  It also rescales the markers at lower zoom levels such that they obscure the map (and each other) less.

Note that due to current Safecast API limitations, there is no concept of a "metasensor" consisting of multiple individual radiation sensors.  Because of this, `rt_viewer.js` has some particular handling that implicitly recognizes dual-sensor hardware.  For sensors sharing the exact same lat/lon (no epsilon), the maxima is used as the value for all of them.  When displaying an infowindow, a chart is shown for again, all sensors at that exact lat/lon.

`rt_viewer.js` is always loaded.


#### hud.js

`hud.js` contains the query reticle, which displays the classification range and median value for a color used by a pixel in a raster map tile.

Note that any new layers added must match existing symbology, or it will need to be updated with additional lookup tables in `hud_worker.js` to perform this action.

`hud.js` works by iterating through a batch of one or more URLs representing active layer(s), retrieving the tile specified by the URL from the server, and attempting to find the nearest pixel within a search radius.  If no match is found, the next tile in the batch is attempted, and so on.  If a match is found, the RGB colors are matched to a premade lookup table to determine the value and range, with some tolerance given as RGB color matches are often not 100%.

`hud.js` is unique among the tilemap modules in that it optionally may call `bitstore.js`'s ShouldLoadTile to improve performance.  None of the other modules currently reference each other, or really have any external dependencies other than the Google Maps API v3 in most cases.  In the current implementation, this is performed by retaining a function reference to `bitstore.js`'s proxy in safemap.js.  It also learns bad URLs, but testing showed this did not improve performance to the same degree as both learning bad URLs and using `bitstore.js` could.

`hud.js` is loaded only on-demand.


#### slideout.min.js  
  
`slideout.min.js` is version 0.1.12 available [here](https://github.com/Mango/slideout).  This provides basic styles and behavior for a slideout menu, which is now integral to the core UI.  
  
However, as this did not work directly with the Google Maps API as intended, code and styles in `index.html` and `safemap.js` disable almost all of the events, and override many of the styles.  It may be worth exploring removing this dependency entirely in the future.  
  
`slideout.min.js` is always loaded.  
 

 

 

 

## Secondary Content Files and Scripts

#### menu_tooltips_512x8704.png  
  
`menu_tooltips_512x8704.png` is a spritesheet containing images used by the menu's tooltip animations.  This is structured such that each image is 256x256 pixels.  Each 512x256 pixel row in the image contains the start and end animation keyframes for a tooltip.  The order of existing images should generally not be changed, as there references to them.  
  
#### scale64_60x854.png

`scale64_60x854.png` is the symbology scale displayed at the bottom-right hand corner of the screen.

#### schoriz_362x44.png

`schoriz_362x44.png` is the primary Safecast logo referenced by index.html, about_inline.html, and methodology.html.

#### xa_13x13.png  
  
`xa_13x13.png` is not in active use, having been inlined as a base64-encoded PNG text string in several content files.  The source file is present for maintainability.


#### bgeigie_viewer_inline.html

`bgeigie_viewer_inline.html` contains HTML elements representing the UI for bGeigie log queries.  It depends on styles in `bgeigie_viewer_inline.css` being loaded first.

`bgeigie_viewer_inline.html` is not a standalone HTML document, and is intended only to be injected or inlined into a content div after being loaded asynchronously.

After the HTML content in `bgeigie_viewer_inline.html` has been fully loaded into the DOM, event listeners are databound to these form elements in `safemap.js`.


#### bgeigie_viewer_inline.css

`bgeigie_viewer_inline.css` contains CSS styles representing the UI for bGeigie log queries, and styles used by `bgeigie_viewer.js` for Google Maps InfoWindows and the Data Transfer view.

`bgeigie_viewer_inline.css` is not a standalone CSS document, and is intended only to be injected or inlined into a style element after being loaded asynchronously.

`bgeigie_viewer_inline.css` should be loaded before `bgeigie_viewer_inline.html` or `bgeigie_viewer.js`.


#### bgpreview_118x211.png

`bgpreview_118x211.png` is referenced by `bgeigie_viewer_inline.html` and is simply a preview image screenshot displayed for illustrative purposes to the user.


#### world_155a_z0.png

`world_155a_z0.png` is referenced by `bgeigie_viewer.js`.  It is a 256x256 pixel Web Mercator map tile at zoom level 0 representing the entire world.

`world_155a_z0.png` is the background image used by `bgeigie_viewer.js`'s Data Transfer view when showing a preview map as logs are being downloaded and processed.


#### about_inline.html

`about_inline.html` contains HTML content displayed when the user clicks "About."

`about_inline.html` is not a standalone HTML document, and is intended only to be injected or inlined into a content div after being loaded asynchronously.

`about_inline.html` contains links to data sources and methodology.html.  It also directly invokes a global JavaScript function to show "What's New".

(todo: resize or redesign to work better with smaller mobile displays)


#### methodology.html

`methodology.html` is a full HTML document describing the data processing methdology of the Tilemap for users.  It is accessed from "About", and opens in a new window.


#### webmap_dataflow_2048x1536.png

`webmap_dataflow_2048x1536.png` is a diagram image referenced by methodology.html.  It contains a simplified overview of the Tilemap's data flow and sources.

#### webmap_files_2048x1536.png

`webmap_files_2048x1536.png` is a diagram image referenced by methodology.html.  It contains a simplified overview of the Tilemap's file structure.

#### whatsnew_en_inline.html

`whatsnew_en_inline.html` contains English-language HTML content displayed once during April 2015, or when the user clicks "What's New" in "About."

`whatsnew_en_inline.html` is not a standalone HTML documents, and is intended only to be injected or inlined into a content div after being loaded asynchronously.

#### whatsnew_ja_inline.html

`whatsnew_ja_inline.html` contains Japanese-language HTML content displayed once during April 2015, or when the user clicks "What's New" in "About."

`whatsnew_ja_inline.html` is not a standalone HTML documents, and is intended only to be injected or inlined into a content div after being loaded asynchronously.

#### whatsnew_en_847x793.jpg

`whatsnew_en_847x793.jpg` is an English-language content image referenced by `whatsnew_en_inline.html`.  It is a diagrammed screenshot illustrating major changes made in an April 2015 release of the Tilemap.

#### whatsnew_ja_847x793.jpg

`whatsnew_ja_847x793.jpg` is a Japanese-language content image referenced by `whatsnew_ja_inline.html`.  It is a diagrammed screenshot illustrating major changes made in an April 2015 release of the Tilemap.

#### flag-jp_50x27.png

`flag-jp_50x27.png` is a borderless Japanese flag icon.  It is referenced by `whatsnew_en_inline.html` and ``whatsnew_ja_inline.html` and used to provide the user with language selection.

#### flag-usuk_50x27.png

`flag-usuk_50x27.png` is a borderless split US/UK flag icon.  It is referenced by `whatsnew_en_inline.html` and `whatsnew_ja_inline.html` and used to provide the user with language selection.

 

 

 

 

## Server Data Sources

#### safecast.org

safecast.org is the primary webhost of the files in this repository.  It provies automatic gzip compression, though that is non-essential.

Contingency planning: the files here may be hosted on any webserver.  There are no special server requirements.  However, all users will currently be referencing this server to use the tilemap.

#### api.safecast.org

api.safecast.org is used directly by the tilemap for right click -> Query API here and the bGeigie log query interface.  More significantly, api.safecast.org is the principal data source for all other data sources.  

api.safecast.org is a custom Ruby app with a PostgreSQL backend that has a specific API used by the tilemap and rt.safecast.org.  It also runs cron jobs to export the DB outside of the Ruby app, which are used directly for tile creation.

Contingency planning: unknown, but the Ruby app is on Github, and daily DB backups occur.  There are dependencies to specific URLs for these servers with no fallback.

#### safecast.media.mit.edu

safecast.media.mit.edu is a VM on one of the last Apple Xserves made, running OS X 10.9.  It provides all raster map tiles used by the tilemap.

safecast.media.mit.edu's tiles are principally created by the Safecast app for OS X running as a cron job, using source data from a job on api.safecast.org.  The Safecast app for OS X obviously must be running on OS X.

safecast.media.mit.edu's tiles are also created by Lionel Bergeret's Python interpolation script running as a cron job, which uses a different source file from api.safecast.org than the app.  This constitutes the majority of the server's workload.  Theoretically, this process does not need to run on OS X.

In both cases, Retile is also run in a cron job to reprocess the tiles.  The OS X app exports all zoom levels of tiles, but Retile is used to assemble these 256x256 tiles into 512x512 tiles for Retina/HDPI displays.  Both 256x256 and 512x512 are actively in use.  For Lionel's interpolated tiles, Retile creates all zoom levels other than 13 for 256x256 tiles, and further assembles them into 512x512 tiles for Retina/HDPI displays.

Contingency planning: Any OS X machine can generate tiles with the Safecast app for OS X, and these can be hosted on any web server.  However, the logistics of the number of files involved make it somewhat impractical to update a remote host daily.  Realistically, another OS X server would need to be set up and configured.  For lesser disasters, the server is partially backed up via CrashPlan, but anything necessitating a new OS install would require some configuration work.  The daily tile sets are not backed up, but archives of the static tilesets are.  There are dependencies to specific URLs for this server with no fallback.  
  
As of 2015-10-27, all tilesets are now hosted on Amazon S3, rather than this server directly.  While this server is still required to generate new tiles and update the tilesets, in the event of a server failure the last synchronized version of the tiles on S3 will remain accessible to users.
  
#### s3.amazonaws.com  
  
This Amazon S3 region (`us-east-1`) is located in North Virginia, USA and this text refers to map tile-specific buckets on `s3.amazonaws.com`, of which there are many.  Only `te512.safecast.org.s3.amazonaws.com` and `tg512.safecast.org.s3.amazonaws.com` are updated on a regular, daily basis.  All other buckets contain static tilesets.  Note the origin of the data for the static tilesets and its processing is beyond the scope of this document.  
  
Cron jobs on `safecast.media.mit.edu` run `s3cmd sync` to push any new tiles as needed.  `s3cmd put` is used to always upload the single master tile used as an index by bitstore.js such that the client displays the date correctly, as not all changes to the dataset will manifest in that master index tile changing.  
  
#### s3-ap-northeast-1.amazonaws.com  
  
This S3 region is located in Tokyo, Japan and is used to provide better accessibility to Japanese users, as the latency between Japan and a US east coast server is significant.  
  
After tiles on `safecast.media.mit.edu` are synchronized with `s3.amazonaws.com`, another script runs on  to synchronize `te512.safecast.org.s3.amazonaws.com` with `te512jp.safecast.org.s3-ap-northeast-1.amazonaws.com`, and `tg512.safecast.org.s3.amazonaws.com` with `tg512jp.safecast.org.s3-ap-northeast-1.amazonaws.com`.  
  
Note that latency between `safecast.media.mit.edu` and `s3-ap-northeast-1.amazonaws.com` is very significant for a large number of files, which is why the bucket-to-bucket sychronization is used -- it takes much, much longer to sync to Japan from `safecast.media.mit.edu`.  
  
All of the buckets used for static tilesets follow the same naming convention, such that the Japan region has "jp" suffixed to the first part of the bucket name.  
  
#### Safecast Amazon S3

This refers specifically to the `safecast-production` bucket on `s3.amazonaws.com` (`us-east-1` region).

`safecast-production.s3.amazonaws.com` is used by api.safecast.org to store bGeigie log files after they have been submitted as a long-term, static archive.  These are referenced by the bgeigie_imports.json API on api.safecast.org as a URL, and bgeigie_viewer.js downloads and parses them directly for display.

Contingency planning: none, other than the redundancies provided by S3.
  
#### rt.safecast.org

`rt.safecast.org` is the Real-Time sensor server, which uses a combination of data pulled from api.safecast.org and data stored within WordPress to provide higher-level and admin functionality.

`rt.safecast.org` provides `devices.json`, a list of all RT sensors which is used directly by rt_viewer.js.

Contingency planning: unknown.  There are dependencies to specific URLs for this server with no fallback.

#### 107.161.164.166

`107.161.164.166` generates chart images used by `rt_viewer.js`.  It uses data from `api.safecast.org` and `rt.safecast.org`.

Contingency planning: unknown.  It is likely the code could be redeployed from a GitHub repo but may not be in sync.  There are dependencies to specific URLs for this server with no fallback.

Note: eventually, the chart generation should be moved to `rt.safecast.org`.  This will require updates to `rt_viewer.js`.



## Localization Notes

As of 2016-09-09, this project is partially localized.

- Primary (slideout) menu: Full English and Japanese localizations, user selectable, with the preference stored in HTML5 local storage.  Defaults to Japan localization if the user's timezone is JST.
- bGeigie Viewer (multiple files): English-only.
- RT Sensor Viewer: Bilingual English/Japanese labels.
- Snapshot "time" slider labels: Bilingual English/Japanese labels.
- About: English-only.
- Documentation: English-only.

(further documentation is forthcoming)
