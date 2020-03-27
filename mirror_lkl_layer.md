## Mirror Last Known Locations layer (LKL) into a standard feature service

This script will copy data from a Last Known Locations (LKL) layer in a Location Tracking (either Location Tracking Service or Location Tracking View) layer and mirror it into a standard feature service. 

As of April 2020, the LKL layer does not support dynamic joins, so the mirroring of data using this script may be useful if the user is trying to relate additional data to their last known locations data. Standard feature services support join features.

This script is designed to be run as a scheduled task, at whatever frequency you want to get updated LKL data from your Location Tracking layer. For example, the admin may set up this script to run every 10 minutes in order to get the latest data from the field.

For information on how to set up scheduled tasks, please see [this article](https://www.esri.com/arcgis-blog/products/arcgis-pro/analytics/schedule-a-python-script-or-model-to-run-at-a-prescribed-time-2019-update/)

and [this article](https://www.esri.com/arcgis-blog/products/product/analytics/scheduling-a-python-script-or-model-to-run-at-a-prescribed-time/)

This script requires the use of a point layer that the data can be mirrored into. This layer would ideally share all the same fields as a standard LKL layer, but could have less fields if all data is not requred. You can clone your Location Tracking Service (LTS) using the `clone_items` functionality in the Python API.

To clone, do:

```python
import arcgis
gis = arcgis.gis.GIS("https://arcgis.com","myusername","mypassword")
item = gis.content.get(gis.properties["helperServices"]["locationTracking"]["id"])
cloned_item = gis.content.clone_items([item], copy_data=False)[0]
# delete Tracks layer from your cloned feature layer collection, so that you're left with only the LKL layer
arcgis.features.FeatureLayerCollection(url=cloned_item.url, gis=gis).manager.delete_from_definition({ "layers" : [{"id" : 0}]})
```

This will create a standard feature service in your organization whose item id you can pass. If you wish, you can also:

Create a blank point layer (Content > Create > Feature Layer > Build a layer > Points) and then add your requisite fields to that layer. Ensure the field_name value is the same, not just the alias.

Location Tracking must be enabled for your organization to use this script. You must be either at least a track viewer in order to use this script.

Supports Python 3.6+

----

Other than the authentication arguments (username, password, org) the script uses the following parameters:

- -item_id - Item id in your portal of the feature service you want to mirror data into. See above for how to create this service. Required
- -lkl-layer-url \<tracks_layer_url\> - The URL to the LKL layer from a Track View / Location Tracking Service you want to utilize. This URL should end in /1 (since we're targeting the second layer in the feature collection). Required
- -log-file \<logFile\> - The log file to use for logging messages. Optional

Example Usage 1  - Mirror LKL data from the lkl layer url into the layer with the listed item id
```python
python mirror_lkl_layer.py -u username -p password -org https://arcgis.com -item-id a05eee7b1cs5461db0e1ef1c1c4abe18 -lkl-layer-url https://locationservices9.arcgis.com/US6xjA1Nc8bW1aoA/arcgis/rest/services/f1087713d8934d5b8218dda736c26af4_Track_View/FeatureServer/1
```

## What it does

 1. First the script uses the provided credentials to authenticate with AGOL to get the required token
 2. Gets the LKL layer, which is provided to the script
 3. Gets the layer that you are mirroring data into 
 4. Checks whether the LKL is already in the mirrored layer or is new
 5. Posts updates with both new and existing features using the `edit_features` functionality in the Python API