---
title: "Neilson Materials Project: GIS Methodology"
author: "Lara Brown"
date: "5/12/2021"
output: 
  html_document:
    keep_md: true
---



These methods assume basic familiarity with ArcGIS Pro and with the data files associated with this project, all of which are included in this repository or can be generated from the included code.

## Preparing data

1. Run neilson_routes.Rmd.

2. Save all CSV files in the repository to an offline directory. Copy the created shapefiles to that directory as well, and name it NeilsonData.

3. Download the state boundaries layer package [here](https://www.arcgis.com/home/item.html?id=540003aa59b047d7a1f465f7b1df1950) and save it to this directory as well.

## Creating the project

1. In ArcGIS Pro, create new map (or load existing).

2. Add folder connection for the NeilsonData folder containing all data files.

3. Set basemap to Imagery.

## Adding data layers

1. Add USA_State_Boundaries.lpk layer package to add state boundaries to the map.

    a. Within symbology, change all lines except for coastline changed to 1 pt, Gray 60%. Coastline changed to 0.5 pt, Gray 60%
Set the (Over 1:10m) layer extent to Out Beyond 1:22,259,312.

    b. Once you do this, I highly recommend saving these features within the project geodatabase. Sometimes ArcGIS is finicky with regard to data connections to layer packages.

2. Add points corresponding to source sites.

    a. Add data → XY Points Data

        i. For Input Table, use the file containing the site points called source_info.csv.
  
    b. Change aliases to be in capital case; change “TYPE” field to “Site Type”
    
    c. Calculate a location field (LOCATION, alias Location)
    
        i. Type: text
        
        ii. Python3 expression: location = !City! + “, “ + !State!
    
    d. Alter symbology → by Site Type
    
        i. Symbols are Tear Pin 1, size 18.5 (should be the default; can adjust to your tastes)
        
        ii. Halo 1pt in lightest gray
        
        iii. Colors: #E8B858 for extraction; #85C2CC for manufacturing
        
        iv. Set In Beyond 1:24,000, Out Beyond 1:22,259,312

    e. Configure label properties
        
        i. Tahoma size 14, white
        
        ii. In Beyond 1:24,000, Out Beyond 1:22,259,312
    
    f. Configure popups
        
        i.Show Material, Company, Site Type, Location in that order.
        
    g. Split by Attribute (within Geoprocessing Tools) with the fields set to stone_id and state.
    
        i. This will create a new layer for each set of points corresponding to a particular material. For example, the two points in Friendsville, TN, representing extraction and manufacturing of the marble will be in one layer, but the granite extraction sites in CT and MA will be in two different layers.
        
        ii. In order to make the animations look clean, for each created layer, if there are multiple points, ensure that only the extraction site is showing by removing the symbology for manufacturing. You will also need to ensure that the label for the manufacturing point does not appear in this case.
        
        iii. Within the labeling properties of each layer, set the position to a fixed region (eg right, bottom left) with an offset of 2 or 3 points to ensure that the labels don’t move as the scale of the map changes.
        
        iv. Rename each layer Material_State_pt.
        
3. Create a point for Neilson Library

    a. Within the catalog pane, right click on NeilsonMaterials2.gdb in Databases tab → New → Feature Class
    
        i. Set up a feature class just for Neilson library, ensure it’s a point layer
      
    b. Then in Edit → create → click on that new feature class and create point, right click on map to create with fixed geocoordinates
    
    c. Set In Beyond 1:24,000

4. Add routes

    a. Add material_routes.shp to the map (from the connected data folder, created by running .Rmd file).

    b. Change symbology → 3 pt, Gray 10%
    
## Preparing routes for animation

1. In Analysis → Tools → search for Densify

    a. Input feature: material_routes
    
    b. Densification method: DISTANCE
    
    c. Distance: 9 kilometers
2. In Analysis → Tools → search for Split Line at Vertices

    a. Input feature: material_routes
    
    b. Output feature: material_routes_split (ensure it’s saving in the right gdb)
    
3. Calculate Field to create a playback index for the animation.
    
    a. Input table: material_routes_split
    
    b. Field name: playback
    
    c. Field type: LONG (this is important!!)
    
    d. Expression type: Python3
    
    e. Expression: autoIncrement(!routeID!)
    
    f. Code block (this code came from [an answer on Stack Overflow](https://gis.stackexchange.com/questions/174659/how-calculate-sequential-values-based-on-group-field)):


```python
rec = 0
group = 1

def autoIncrement(route_id):
  global rec
  global group
  pStart = 1
  pInterval = 1
  if (route_id == group):
    rec += pInterval
  
  else:
    group += pInterval
    rec = pStart
  
  return rec
```


4. Run Split by Attribute where the splitting attribute is route_name. 

    a. This should be unique. If route_name doesn’t already exist, might need to rename the attribute corresponding to Material_State.
    
    b. For each of the created route layers, in the appearances tab, set In Beyond to 1:24,000.

## Create animations

1. Turn on one route and its corresponding point layer.

2. Right click on the route layer and select “properties.” Within the range section, calculate a new range from the playback field.

3. In the range tab that now appears at the top of the screen, ensure that the full extent is set to the current layer.

4. Set the minimum to the down arrow (no fixed minimum)

5. Set the maximum to the maximum value of the range.

6. Drag the range slider down to 1 so that only the first route segment is showing.

7. Go to View → animation → add. You’re now ready to start adding keyframes to your animation! You’ll add a few, detailed in the next step.

### Animation keyframes

Before you start, ensure the range slider is at first frame and both the route and point layers of interest are enabled.

1. Zoom the map to a scale less than 1:10,000 centered at the origin point (usually the quarry). Click “add keyframe.”

2. Select that first keyframe in the animation pane and add a hold.

    a. Within the animation tab, go to properties → keyframe → length and change to 1 second.
    
2. Zoom out so the map is still centered around the point of interest but a larger region is shown. You should be able to see some topographic features and should be at a zoom where both the point and label are visible. Add this as a keyframe.

3. Zoom out further so that both the origin and destination (Neilson) points are in view. Add keyframe.

5. Drag the range slider to the max value so that the full range/route is shown on the map. Add keyframe.

    a. Can adjust time of this keyframe to be 5-6 seconds instead of 3.
    
6. Check the overall duration of the animation. It should be 10-15 seconds. You can adjust it if it’s not and it’ll scale the length of each keyframe accordingly.

The animation is now ready to export! You can try to play it within ArcGIS Pro but it’ll look choppy. 

* Choose the 720 preset.

* Choose a location where you can find it again!!

* Save as mp4.

* Export.

Add a new animation by clicking the green +. Turn off the point and route that you just completed and turn on the next ones. Ensure that you change the range “Full Extent” to reflect the layer you’ve selected.

### Animation caveats

For the path from China, first show an animation of the path from quarry to port following the steps above. Then set map scale to cover full Earth (or at least, full path from China across Atlantic to eastern US. Then zoom in on Massachusetts, then Northampton, then finally Neilson.


The two granite paths are complicated since we wanted to animate them together.

* Look at the playback ranges for both granite routes (CT and MA) and calculate their least common multiple (lcm). Divide the least common multiple by the max playback value for each. For each route, calculate a new field called route_sync and ensure its type is Long (not text). The Python expression for this field should be `(lcm/max)*!playback!`, where lcm/max is an integer. Create a new range from this field.

* Within the range tab, on the left hand side, change the selected variable range from playback to route_sync. Ensure that both granite points and both routes are turned on. Set the range Full Extent to the maximum of the route_sync values (I believe the Granite_CT route_sync had a larger max, so choose that one).

    * Step through the range to ensure that the routes both reach Neilson around the same time.

Repeat the standard animation process above, with some adjustments:

1.Start zoomed into the CT quarry and add 1s hold.

2. Zoom out so both quarries are in view.

3. Zoom into MA quarry and add 1s hold.

4. Zoom out so both quarries and Neilson are in view and add keyframe.
Adjust range slider so both routes are fully shown and add this ending keyframe, and change its length to 5s.

5. You will probably need to adjust the full time so the entire animation is 10-15s long.

## Exporting a map package web map

There are two different products that can and should be produced from this data.

First, create a map layout containing all the individual point layers and individual route layers in addition to the imagery basemap and state boundary layers. Name this map "Neilson_Materials_Map." In the share tab, export this map as a map package.

Second, create a new map layout. Copy the imagery base map, state boundaries, and Neilson point layers into this new map. Also copy over just the layer containing *all* the site points and the layer containing *all* the routes before they were segmented (material_routes, not material_routes_split).

* Open the attribute table for material_routes. Calculate a new field called dist with alias Distance (mi). The Python expression for this field should be `round(!route_len!)`. If necessary, save the data through the "Edit" tab, then save the project.

* Manually add another field called "Material" that just lists the stone that traveled along each route.

* Manually add a field called "Origin" that lists the location (City, State) from which the stone began its journey.

* Configure pop-ups to show Material, Origin, and Distance (mi) in that order.

* In the "Share" tab, export a Web Map. Ensure that you are exporting just this map and not the map that contains each point and route in its own layer. This Web Map will allow interested parties to explore some of these sites and journeys at their leisure, moreso than the animations allow. Choose to share either with the Spatial Analysis lab or with everyone!
