# Plan: Add Related Data Export Functionality to qgis2web

## Goal

Implement a feature allowing users to export related data associated with features in vector layers directly from the generated web map (both OpenLayers and Leaflet).

## Steps

1.  **Identify Related Data Access:**
    *   Research QGIS Python API (`qgis.core`) to reliably identify layers with relations defined in the QGIS project.
    *   Determine how to fetch related child features for a given parent feature using its ID and the relation object.

2.  **UI Modifications (`maindialog.py`, `ui_maindialog.ui`):**
    *   Add a new checkbox column or option to the layers list/tree in the main dialog. Label it something like "Enable Related Data Export".
    *   This option should only be enabled/visible for vector layers that have relations defined in the QGIS project.
    *   Store the state of this checkbox per layer in the export configuration.
    *   *(Optional - Phase 2)* Add a button next to the checkbox to configure basic export parameters (e.g., which relations to include if multiple exist, filename prefix). For Phase 1, export all related data for enabled layers.

3.  **Core Export Logic (`utils.py`):**
    *   In `utils.py`, when processing features for a layer, check if the "Enable Related Data Export" option is checked for that layer.
    *   The relation name should be sanitized to ensure it is a valid JSON key (e.g., replace spaces with underscores, remove special characters). This will be used to name the property in the GeoJSON output. in this document, we'll refer to it as `qgis2web_related_data`.
    *   Create a new function `get_related_data(layer, feature)`:
        *   This function will use the QGIS API to find defined relations for the `layer`.
        *   For each relation, it will fetch the related child features corresponding to the input `feature`.
        *   It will format the attributes of these related features into a list of dictionaries (or a similar JSON-friendly structure).
    *   Modify the feature's properties dictionary (that gets converted to GeoJSON properties) to include a new key, named by the relation name present in QGIS. The value will be the JSON structure returned by `get_related_data`.
    *   Ensure this process handles cases with no related data gracefully (e.g., empty list or null value).
    *   In the GeoJSON output, the related data will be included as a new property named by the relation name for each feature, see `qgis2web_related_data` for more details.
    *   *(Optional - Phase 2)* If the user configures export parameters, modify the `get_related_data` function to respect these settings (e.g., only include certain relations or attributes).
    *   Before executing the export, check if the layer has the "Enable Related Data Export" option checked. If not, skip the related data fetching for that layer.
    *   For each features in function `writeTmpLayer`, add the related data to the feature's properties dictionary.
        *   *(Optional - Phase 2)* For each features that have related data, use the existing function `exportImages` to export the related images data for each field.

4.  **Leaflet Implementation (`leafletWriter.py`, `leafletLayerScripts.py`, `leafletScriptStrings.py`, new JS file):**
    *   **`leafletLayerScripts.py` / `leafletScriptStrings.py`:** Modify the popup content generation logic:
        *   Check if `feature.properties.qgis2web_related_data` exists and is not empty/null.
        *   If it exists, add an HTML element (table) to display the related data in the popup. This should reuse the existing popup structure. So a popup in the popup should be created. Reuse the `getPopups` function that already exists and return the HTML string(table).
    

5.  **OpenLayers Implementation (`olwriter.py`, `olLayerScripts.py`, `olScriptStrings.py`, new JS file):**
    *   **`olLayerScripts.py` / `olScriptStrings.py`:** Modify the popup content generation logic similarly to Leaflet:
        *   Check if `feature.properties.qgis2web_related_data` exists and is not empty/null.
        *   If it exists, add an HTML element (table) to display the related data in the popup. This should reuse the existing popup structure. So a popup in the popup should be created.
  
6.  **Documentation:**
    *   Update `README.md` or `docs/` to explain the new feature and how to use it.

## Considerations

*   **Error Handling:** Add checks for invalid relations or errors during data fetching in the Python code. Add checks in JavaScript for missing data before attempting export.