# Procedure

## Requirements

-   GIS Data
    
    Roads, Parcels, Building Footprints, Imagery, 2013 LiDAR data

-   Software
    
    ArcGIS v10.1+ with 3D Analyst Extension
    
    Esri CityEngine 2014 (county employees should contact the GIS department for a license)

-   [Redlands Redevelopment Tutorial](http://www.esri.com/software/cityengine/industries/redlands-redevelopment)

## Prepare the data

1.  Open a new MXD and add Roads, Parcels, Building Footprints and Imagery

2.  Create a Staging Folder for the data to be imported into CityEngine and save the MXD in that folder.

3.  Create a Staging File Geodatabase in the Staging folder created in the last step.

4.  Create a new polygon feature class in the file geodatabase created in the previous step. Name it Data\_Extent.

5.  Add the Data\_Extent feature class to your MXD and start editing.

6.  Create a new rectangle polygon in the Data\_Extent feature class and draw the box around the data you want to use in CityEngine. Typically this should be a small area of only a few blocks. Too much data may cause performance issues. Stop editing and save your edits.

7.  Use the Clip geoprocessing tool to clip the Roads, Parcels and Building footprints feature classes to the Data\_Extent polygon. Save the results in the staging file geodatabase.

8.  Since we don't have an existing Tree inventory, create a new point feature class in the staging file geodatabase called "Trees". You may want to Import fields from the trees feature class in the [Redlands Redevelopment Tutorial](http://www.esri.com/software/cityengine/industries/redlands-redevelopment).

9.  Add the new Trees feature class to your MXD and use aerial imagery to place tree locations. You may want to symbolize the points using a 2D tree symbol. You don't have to fill in the attribute table to move on.

10. Make sure the 3D Analyst extension is enabled in ArcMap by going to Customize - Extensions and making sure the 3D Analyst extension is checked.

11. Create a new LAS Dataset in the staging folder. Use the tips [here](http://resources.arcgis.com/en/help/main/10.2/index.html#//015w0000005r000000).

12. Add the LAS dataset to your MXD and enable the LAS Dataset toolbar.

## Create elevation rasters

1.  Select your LAS dataset in the dropdown box in the LAS Dataset toolbar, click on the Filters box and select Ground.

2.  Run the LAS Dataset to Raster geoprocessing tool (Under Conversion Tools - To Raster) using the following options to create a Digital Terrain Model (DTM) showing the elevation of only the ground.
    -   Input LAS Dataset: <Select your LAS Dataset from dropdown list>
    
    -   Output Raster: Save to your staging file geodatabase as DTM
    
    -   Value Field: ELEVATION
    
    -   Interpolation Type: Triangulation with NATURAL\_NEIGHBOR interpolation method and NO\_THINNING.
    
    -   Output Data Type: FLOAT
    
    -   Sampling Type: CELLSIZE
    
    -   Sampling Value: 3
    
    -   Z Factor: 1
    
    -   VERY IMPORTANT: Click on the Environments button and set the Processing Extent to the Same as your Data\_Extent layer. This will ensure you only create a DTM for your area of interest rather than spending hours creating a countywide DTM.

3.  Again, select your LAS dataset in the dropbox box on the LAS Dataset toolbar. This time set the Filter to First Return.

4.  We'll run the LAS Dataset to Raster geoprocessing tool again but with different parameters as below to create a Digital Surface Model (DSM) showing elevation of objects.
    -   Input LAS Dataset: <Select your LAS Dataset from dropdown list>
    
    -   Output Raster: Save to your staging file geodatabase as DSM
    
    -   Value Field: ELEVATION
    
    -   Interpolation Type: Binning with MAXIMUM cell assignment type and NATURAL\_NEIGHBOR void fill method.
    
    -   Output Data Type: FLOAT
    
    -   Sampling Type: CELLSIZE
    
    -   Sampling Value: 3
    
    -   Z Factor: 1
    
    -   VERY IMPORTANT: Click on the Environments button and set the Processing Extent to the Same as your Data\_Extent layer. This will ensure you only create a DSM for your area of interest rather than spending hours creating a countywide DSM.

5.  Next, we want to create a Normalised Digital Surface Model (nDSM) to get heights of objects above the ground. We do this using the Minus geoprocessing tool under 3D Analyst Tools - Raster Math. Use the options below.
    -   Input Raster or Constant Value 1: Your DSM
    
    -   Input Raster or Constant Value 2: Your DEM
    
    -   Output Raster: Save to your staging file geodatabase as nDSM

6.  Finally, we'll create a .tif file of our DTM. Open the Clip geoprocessing tool under Data Management Tools - Raster - Raster Processing and set the following parameters.
    -   Input Raster: Your DTM layer
    
    -   Output Extent: Your Data\_Extent layer
    
    -   Rectangle: Ignore this. The numbers are automatically filled in
    
    -   Output Raster Dataset: DTM.tif in your staging folder (not in geodatabase)
    
    -   Leave other values as default

## Determine elevation of buildings

Now we have a normalised digital surface model we can use to determine the heights of buildings by sampling at random locations. 
1.  Find and open the "Create Random Points" geoprocessing tool under Data Management Tools - Feature Class. Use the following parameters.
    -   Output Location: Your staging file geodatabase
    
    -   Output Point Feature Class: Random\_Points
    
    -   Constraining Feature Class: Building Footprints
    
    -   Number of Points: Long option, value 100
    
    -   Minimum Allowed Distance: Linear unit set to 0 Meters

2.  Next we'll use the normalised DSM to assign height values to those points. Open the Add Surface Information geoprocessing tool under 3D Analyst Tools - Functional Surface and use the following parameters.
    -   Input Feature Class: Select Random\_Points from the dropdown
    
    -   Input Surface: This is your nDSM raster dataset
    
    -   Output Property: Check the box for Z
    
    -   Method: BILINEAR
    
    -   Sampling Distance: Leave blank

3.  Next we'll find the average height of the Random\_Points for each building using the Summary Statistics geoprocessing tool under Analysis Tools - Statistics. Use the following parameters.
    -   Input Table: Random\_Points
    
    -   Output Table: Summary\_Statistics.dbf in your staging folder
    
    -   Statistics Field(s): Field Z, Statistic Type MEAN
    
    -   Case Field(s): CID

4.  The new table has a MEAN\_Z field showing the average height for each building. Note: Large trees next to buildings can sometimes skew this number since the trees and buildings cannot be differentiated in the normalised digital surface model. You can manually look for and delete outlier points from the Random\_Points before running the Summary Statistics tool.

5.  Next we'll assign each building its average height. Use the Add Join geoprocessing tool under Data Management - Joins with the following parameters.
    -   Layer Name or Table View: Your Building Footprints layer
    
    -   Input Join Field: ObjectID
    
    -   Join Table: Summary\_Statistics.dbf
    
    -   Output Join Field: CID

6.  Now examine the attribute table for Building Footprints. You should see a MEAN\_Z field added to each feature.

7.  Now we'll create a new feature class from this called Buildings\_Final. Right click on the Building Footprints layer and choose Data > Export Data. Save the new feature class to your staging file geodatabase but don't add it to the MXD yet.

8.  Find the new Buildings\_Final feature class in ArcCatalog or your Catalog window in ArcMap. Open the properties of the feature class and go to the fields tab. Rename the MEAN\_Z field and field alias to totalHeight.

9.  Now add the Buildings\_Final feature class to your MXD and open the attribute table.

10. The CityEngine tools we'll be working with were developed in Europe and they don't use our imperial measurement system. So we need to change our heights from feet to meters. Right-click on the totalHeight field in the attribute and select Field Calculator. Click Yes on the warning dialog box to continue.

11. In the Field Calculator's expression box type "[totalHeight] \* 0.3048" and click "OK". This will convert the heights from feet to meters.

## Get an aerial image for your Area of Interest

CityEngine doesn't work well with large rasters. So we'll create a lower resolution JPG of our area of interest.
1.  Turn off all layers in your MXD except the aerial photo.

2.  Click on File - Export Map and use the following parameters to export a georeferenced aerial image to your staging folder.
    -   File name: Orthophoto.jpg
    
    -   Save as type: JPEG
    
    -   General Tab
        -   Resolution: 100-150 dpi
        
        -   Check the box for "Write World File"
    
    -   Format Tab
        -   Color Mode: 24-bit True Color
        
        -   JPEG Quality: About 75 is good
        
        -   Background Color: White
        
        -   Check the box for "Progressive"

3.  After you save, check the size of your JPG image. For best results, try to keep it between 500KB - 1MB. You may have to raise or lower the resolution to get the optimal size.

4.  Add the image to ArcMap and check that it's properly referenced to your data. If it isn't, go back and make sure you selected "Write World File" when you exported the image.

5.  Now we'll clip the image to match our DTM. Open the Clip geoprocessing tool located in Data Management Tools - Raster - Raster Processing. Set the following parameters.
    -   Input Raster: Orthophoto.jpg
    
    -   Output Extent: Your DTM layer
    
    -   Rectangle: Ignore. It's automatically populated
    
    -   Output Raster Dataset: Orthophoto\_final.jpg in your staging folder
    
    -   Other options leave as default.

## Setting up Esri CityEngine with your GIS data

I have only tested these steps with the Advanced version of CityEngine, but I believe they should work with the Basic version, too.
1.  If you haven't already done so, download and extract the Redlands Redevelopment tutorial data from [here](http://www.arcgis.com/home/item.html?id%3Dff937324c7f2479d8895ec8f79278a4a&_ga%3D1.172552032.292721704.1399587999).

2.  Open CityEngine 2014 and go to File-New and create a new CityEngine Project. Name it after your area of interest (ex. Eastsound, LopezVillage, FridayHarbor). You may also want to change the default location.
    
    ![img](images/MyCityEngineProject.png)

3.  Expand your new project in the Navigator window and right-click on the "scenes" folder. Create a new CityEngine scene and name it something like "Existing Conditions.cej". Set the coordinate system to match your data (i.e. EPSG:2285).
    
    ![img](images/CityEngine_Scene.png)

4.  In the Navigator window, right-click the top level folder that is the name of your project and select Import.

5.  On the Import wizard, select Archive File under the Files into Existing Project folder. Then click Next.
    
    ![img](images/CityEngine_ImportData.png)

6.  Browse for the location you downloaded the Redlands Redevelopment data to and open the folder called 3D\_City\_Design\_Training and double-click the file named DataForCityEngineImport.zip.

7.  Select the top level folder in the left pane and click the "Deselect All" button to remove all checkboxes.

8.  Now expand the top level folder and check only the boxes for "assets", "bin", "maps" and "rules". The Into folder box should be the name of your project. Click Finish.
    
    ![img](images/CityEngine_ArchiveFile.png)

9.  When the tutorial data is done importing, we'll want to import our GIS data.

10. Right-click on the "data" folder in your project and select Import.

11. Under CityEngine layers, select File GDB Import and click Next.
    
    ![img](images/CityEngine_ImportFGDB.png)

12. Locate your staging file geodatabase using the Browse button.

13. Check only the boxes for the Roads, Buildings\_Final, Parcels and Trees layers.
    
    ![img](images/CityEngine_ImportFGDB_2.png)

14. Leave all other settings default and click Finish. Now you should see nodes and shading in the Viewport window.
    
    ![img](images/CityEngine_Viewport.png)

15. Navigate to your staging folder in Windows Explorer. Select the DTM.tif and Orthophoto\_final.jpg and right-click and select Copy.

16. Now navigate to your CityEngine project folder in Windows Explorer. Open the "maps" folder and paste the DTM.tif and Orthophoto\_final.jpg files in there.

17. Back in CityEngine, right click on the "maps" folder in the Navigator window and select Refresh. You should see both files in there now.
    
    ![img](images/CityEngine_Maps.png)

18. Notice the Scene window. You can turn on or off layers by clicking the eye icons next to them.
    
    ![img](images/CityEngine_SceneWindow.png)

## Making it pretty in CityEngine

1.  Click on File and select Import.

2.  Under CityEngine Layers, select Terrain Import and click Next.
    
    ![img](images/CityEngine_TerrainImport.png)

3.  For the Heightmap file browse to the "maps" folder and select DTM.tif.

4.  For the Texture file use the Orthophoto\_final.jpg also located in the "maps" folder.

5.  Leave the other settings as default and click Finish.
    
    ![img](images/Terrain.png)

6.  Right-click on the Terrain DTM layer in the Scene window and select "Frame Layer" to view the extent.

7.  In the Scene window, right-click on Roads and select "Align Graph to Terrain". Use the following parameters:
    -   Align function: Project All
    
    -   Heightmap: Terrain DTM

8.  Your Roads layer should now be draped over the terrain.
    
    ![img](images/Terrain_Viewport.png)

9.  Select all streets by right-clicking Roads again and choose Select - Select Objects in Same Layer.

10. Then click the "Align Terrain to Shapes" tool on the toolbar.
    
    ![img](images/Align_Terrain_To_Shapes.png)

11. Set the following parameters for Align Terrain to Shapes
    -   Terrain: Terrain DTM
    
    -   Raise Terrain, Lower Terrain, Add border: All checked
    
    -   Maximal raise distance and Maximal lower distance: 100
        
        ![img](images/Align_Terrain_To_Shapes_2.png)

12. Now we'll align the other vector data. Hold Ctrl and select Buildings and Trees in the Scene window.

13. Right-click the selected layers and choose "Align Shapes to Terrain". Use the following parameters:
    -   Align function: Translate to Maximum
    
    -   Heightmap: Terrain DTM
        
        ![img](images/Align_Terrain_To_Shapes_3.png)

14. Now the buildings and trees layer are draped on top of the terrain.

15. Next, we'll assign rules to the GIS Data to create 3D models.

16. Right-click the Buildings layer in the Scene window and select "Assign Rule File".

17. In the 3D\_City\_Design\_Rules folder double-click on Building Construction.cga.
    
    ![img](images/Building_Construction.png)

18. Right-click the Buildings layer again and this time, select "Generate". The buildings should start to extrude.

19. Right-click the Buildings layer and click "Select - Select Objects in the Same Layer".

20. In the Inspector window, find and expand the "Zoning" tab. Under "3D FORM HEIGHT LIMIT" set "Height\_Method" to "Limit Height to Max\_Height".
    
    ![img](images/Height_method.png)

21. Also under "3D FORM HEIGHT LIMIT" click on the Attribute Connector button next to "Max\_Height".
    
    ![img](images/Max_Height.png)

22. In the Attribute Connection Editor window, change the selected button to "Layer attribute". Set the following parameters:
    -   Select Layer: Buildings
    
    -   Select Source: totalHeight
        
        ![img](images/Attribute_Connection_Editor_buildings.png)

23. Now the buildings should be the correct heights.

24. While the buildings are still selected, click the "Align Terrain to Shapes" button on the toolbar. Leave the default settings.
    
    ![img](images/Align_Terrain_To_Shapes.png)

25. Again, with the buildings still selected, open the Facade Construction tab in the Inspector Window. Set the "Generate\_Facades" switch to "On".
    
    ![img](images/Generate_Facades.png)

26. Optionally, change the Wall\_Texture under "Facade Construction - WALLS" to your desired look.
    
    ![img](images/Wall_Texture.png)

27. Right-click the Roads layer in the Scene window and "Assign Rule File", choosing "Street Construction.cga".

28. Right-click the Roads layer and click "Generate".

29. Right-click the Trees layer and choose the "Points to Trees.cga" rule to assign to it. Then Generate the trees.

30. The Trees may appear with black wireframes. Click the "View Settings" button on the toolbar and click "Wireframe on Shaded/Textured" to toggle it off.
    
    ![img](images/Wireframe_Shaded.png)

31. Optionally, select all or individual trees and change the Name and Height in the "Tree" tab of the Inspector window. Remeber, heights are in meters, not feet.
    
    ![img](images/Trees.png)

## Sharing your CityEngine Model

Now we'll share our 3D Model to ArcGIS Online as a web scene so others can view it.
1.  Use the Move tools on the toolbar to pan, zoom and rotate your view.
    
    ![img](images/Move_tools.png)

2.  Click the "Bookmark" button on the toolbar to save a view. This will be a selectable option in the web scene. Do this several times with different views.
    
    ![img](images/Bookmark_button.png)

3.  Now, turn on all layers in the Scene window that you want in the web scene.

4.  Right-click inside the viewport window and click "Select - Select All".

5.  In the File menu, click "Export Models".

6.  For the Model Export, choose "CityEngine Web Scene" and click "Next".

7.  Use the following parameters for the "CityEngine WebScene" dialog box.
    -   Output Path: <Use the "models" folder in your workspace>
    
    -   Scene Name: <The name of your scene>
    
    -   Exported Content: Export Generatable Models
    
    -   Terrain Layers: Export all visible terrain layers
    
    -   Leave default settings for the rest and click "Next"
        
        ![img](images/CityEngine_WebScene.png)

8.  Make sure all layers are enabled in the "Per Layer Options" dialog box and click "Finish".

9.  Next we'll view the web scene before sharing it. 
    NOTE: At the time of writing, only Mozilla Firefox and Google Chrome can view the web scenes. Internet Explorer will not work. Please inform the people you share with.

10. Under the "File" menu, click "Refresh Workspace".

11. In the Navigator window, expand the "models" folder in your workspace.

12. Right-click the 3ws file, select "Open With - 3D Web Scene Viewer (Offline)". The web scene should open in Firefox or Chrome.
    
    ![img](images/3D_Web_Scene_Viewer_offline.png)

13. If the Web Scene doesn't load in either Firefox or Chrome, you may need to update your video card drivers. See your IT person for help.

14. Look for your bookmarks. They should appear as selectable thumbnails at the bottom of the webscene.

15. Pan and rotate around the web scene by holding different mouse keys and dragging the mouse. Zoom in or out using your mouse wheel.

16. You can turn layers on and off by clicking the "eye" icons on the right side of the web scene.

17. Open the "Settings" tab and check the boxes under "Shadowing". Drag the Sunlight slider to alter the lighting.
    
    ![img](images/WebScene_Settings.png)

18. To share the web scene with others you have to upload it to ArcGIS Online.

19. If you don't have an ArcGIS Online account, check with your GIS Staff or Esri Account Manager to get one set up.

20. Back in CityEngine, click "File - Sign In&#x2026;" and enter your ArcGIS Online account information.

21. Right-click on the 3ws file in the "models" folder and select "Share As".

22. In the CityEngine Web Scene Package dialog box, select "Upload package to my ArcGIS Online or Portal account".

23. On the left, select "Item Description" and fill in details about your web scene.
    
    ![img](images/CityEngine_Web_Scene_Package.png)

24. On the left again, select the "Sharing" tab and check the box for "Everyone (public)".

25. Finally click the "Share" button to upload your web scene.

26. After the upload succeeds, open <http://arcgis.com/home/> in Firefox or Chrome and sign in to your account.

27. Once you are signed in type the name of your 3ws file in the Search box at the top right and click Enter.
    
    ![img](images/ArcGIS_Search.png)

28. Click on the "Open" link below your scene icon to open the web scene.

29. Copy the entire URL text in the URL bar and paste into an email to share the web scene with others.
    
    ![img](images/ArcGIS_URL.png)

# CityEngine Resources

-   [Quick intro to CityEngine and brief tutorials](http://resources.arcgis.com/en/help/cityengine/10.2/index.html#//02w100000003000000)

-   [Redlands Redevelopment Tutorial](http://www.esri.com/software/cityengine/industries/redlands-redevelopment)
        I found this one extremely helpful