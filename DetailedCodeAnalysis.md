# Detailed Code Analysis for CataclysmModder

This document provides a deeper dive into the CataclysmModder codebase, highlighting key class relationships, data flows, and critical components for developers looking to understand and modify the application.

## 1. Key Class Relationships

Describe the primary classes and how they interact:
-   **`Form1` (Main UI Window):**
    -   Instantiates and manages various custom `UserControl` descendants (e.g., `GunValues`, `ArmorValues`, `ComestibleValues`, `GenericItemValues`, `ItemGroupValues`, `RecipeControl`, `ProfessionValues`, `VehiclePartValues`). These are stored as member variables.
    -   Interacts heavily with the static `Storage` class for all backend data operations (loading files, getting item data, saving files).
    -   Handles UI events and delegates actions to `Storage` or updates UI elements based on data from `Storage`.
    -   Uses `WinformsUtil` for some UI helper functions.
-   **`Storage` (Static Data Management Hub):**
    -   Manages the list of open JSON files (`openFiles`) and their parsed content (`openItems`, a `List<BindingList<ItemDataWrapper>>`).
    -   Contains definitions for different file types (`FileType` enum) and their properties (`CataFile` objects, which link to `JsonSchema` and UI controls).
    -   Uses `JavaScriptSerializer` for deserializing JSON into `Dictionary<string, object>`.
    -   Uses `JsonSchema` for serializing `ItemDataWrapper` data back into formatted JSON strings.
    -   Provides static methods for accessing and modifying loaded data (e.g., `LoadFiles`, `SelectFile`, `LoadItem`, `ItemApplyValue`, `SaveFile`).
-   **`ItemDataWrapper` (JSON Item Representation):**
    -   Wraps a `Dictionary<string, object>` which holds the actual data for a single JSON entry (e.g., an item, a recipe).
    -   Implements `INotifyPropertyChanged` to signal changes to UI elements (specifically the `entriesListBox` in `Form1`). The `Display` property is key for this.
    -   Tracks modification status (`Modified` flag).
    -   `memberOf` field links it to the index of the file it belongs to in `Storage.openFiles`.
-   **`*Values.cs` Controls (Specific Item Editors):**
    -   These are `UserControl` classes, each tailored to edit a specific type of JSON object (e.g., `GunValues` for "GUN" items, `ComestibleValues` for "COMESTIBLE" items).
    -   They contain various input elements (TextBoxes, ComboBoxes, etc.).
    -   Data is typically loaded into these controls from an `ItemDataWrapper`'s dictionary using helper methods (often via `WinformsUtil.ControlsLoadItem` called by `Storage.LoadItem`).
    -   Changes in these controls should trigger `Storage.ItemApplyValue` to update the underlying `ItemDataWrapper` and mark changes.
-   **`JsonSchema` (JSON Serialization Logic):**
    -   Loads custom schema definitions from embedded text files (e.g., `CataclysmModder.schemas.items.txt`). These schemas define the order and inclusion of keys for different item types.
    -   The `Serialize` method constructs a JSON string from an `ItemDataWrapper`'s data dictionary, respecting the loaded schema for the given item type.
-   **`CataFile` (File Type Definition):**
    -   Helper class used by `Storage` to define properties of supported JSON file types, such as the primary ID key, display member, associated `JsonSchema`, and the main `Control` (a `*Values` user control) used for editing its entries.

## 2. Data Flow Analysis

Describe the sequence of events and component interactions for major operations.

### 2.1. Application Startup & Initial File Load (if `.conf` exists)
1.  **`Program.Main()`**: Application entry point. Creates and runs `Form1`.
2.  **`Form1.Form1()` (Constructor)**:
    -   Initializes `Storage.InitializeFileDefs()` to set up `CataFile` definitions.
    -   Instantiates all `*Values.cs` controls and other specific controls like `ItemGroupValues`, `RecipeControl`, etc. Associates them with `Storage.FileType` via `Storage.FileDefSetControl()`.
    -   Checks for `.conf` file. If it exists:
        -   Reads the path to the game's 'raw' folder.
        -   Calls `Form1.loadFiles(path)`.
3.  **`Form1.loadFiles(string path)`**:
    -   Stores the path in `.conf`.
    -   Calls `Storage.LoadFiles(path)`.
4.  **`Storage.LoadFiles(string path)`**:
    -   Gets all `*.json` file paths from the directory (and subdirectories).
    -   For each file:
        -   Calls `Storage.LoadFile(fileIndex)`.
5.  **`Storage.LoadFile(int index)`**:
    -   Reads the JSON file content.
    -   Uses `JavaScriptSerializer().DeserializeObject(json)` to parse the JSON into `object[]` (array of dictionaries) or a single dictionary (for recipes).
    -   For each JSON object (dictionary) in the file, creates an `ItemDataWrapper` instance, storing the dictionary and the `memberOf` index.
    -   Adds these `ItemDataWrapper` instances to the appropriate `BindingList<ItemDataWrapper>` in `openItems`.
    -   If the file type is `ITEMS` or `BIONICS`, subscribes to `ListChanged` event for autocomplete updates.
6.  **`Form1.loadFiles()` (Continued)**:
    -   Populates `filesComboBox` with `Storage.OpenFiles`.
    -   If files are loaded, selects the first file in `filesComboBox`, triggering `filesComboBox_SelectedIndexChanged`.

### 2.2. Selecting a JSON File from Dropdown
1.  **`Form1.filesComboBox_SelectedIndexChanged`**:
    -   Calls `Storage.SelectFile(filesComboBox.SelectedIndex)` to set `Storage.currentFileIndex`.
    -   Hides all `*Values` and other main panel controls.
    -   Gets the `CataFile` definition for the current file type via `Storage.GetFileDefForCurrentFile()`.
    -   If a `CataFile` and its associated control exist, makes that control visible.
    -   Sets `entriesListBox.DataSource` to `Storage.OpenItems` (which refers to `openItems[currentFileIndex]`).
    -   If there are items, selects the first item in `entriesListBox`, triggering `entriesListBox_SelectedIndexChanged`.

### 2.3. Selecting an Item from the List
1.  **`Form1.entriesListBox_SelectedIndexChanged`**:
    -   If the selected index is valid and different from `Storage.CurrentItemIndex`:
        -   Calls `Storage.LoadItem(entriesListBox.SelectedIndex)`.
2.  **`Storage.LoadItem(int index)`**:
    -   Sets `currentItemIndex = index`.
    -   Gets the `CataFile` for the current file.
    -   If `CataFile.control` is not null (i.e., a `*Values` control is associated):
        -   Calls `WinformsUtil.ControlsLoadItem(CataFile.control, CurrentItemData)`. This method (presumably in `WinformsUtil.cs`, not explicitly reviewed yet but inferred) is responsible for populating the fields of the specific `*Values` control with data from `CurrentItemData` (which is `openItems[currentFileIndex][currentItemIndex].data`).
    -   Dynamically shows/hides specific "extension" controls (like `GunControl`, `ArmorControl`) based on the item's "type" property (e.g., if item `type` is "GUN", `Form1.GunControl` is made visible). This logic is primarily in `Form1` after `Storage.LoadItem` completes, by checking `CurrentItemData["type"]`.

### 2.4. Editing Data in a `*Values` Control
1.  **User Interaction**: User changes a value in a field (e.g., TextBox, ComboBox) within a visible `*Values` control (e.g., `GunValues1.nameTextBox`).
2.  **Event Handler**: The control's event handler (e.g., `TextChanged`, `SelectedIndexChanged`) for that input field is triggered.
3.  **`Storage.ItemApplyValue(string key, object value, bool mandatory)`**:
    -   The event handler in the `*Values` control *should* call this method.
    -   `key`: The JSON key for the property being changed (e.g., "name", "weight").
    -   `value`: The new value from the input field.
    -   `mandatory`: Indicates if the field is mandatory (affects removal if value is empty).
    -   This method updates the `Dictionary<string, object>` within the current `ItemDataWrapper` (`Storage.CurrentItemData`).
    -   Sets `Storage.unsavedChanges = true`.
    -   Calls `openItems[currentFileIndex][currentItemIndex].NotifyKeyChanged(key)`.
4.  **`ItemDataWrapper.NotifyKeyChanged(string key)`**:
    -   Sets `this.Modified = true`.
    -   If the changed key affects the display name (`displayMember` or `displaySuffix` from `CataFile`), it raises the `PropertyChanged` event with "Display" as the argument. This updates `Form1.entriesListBox` if the display name of the item changes.

### 2.5. Saving a File
1.  **User Action**: Clicks "Save File" or "Save All".
2.  **`Form1.saveFileToolStripMenuItem_Click` or `Form1.saveAllToolStripMenuItem_Click`**:
    -   Calls `Storage.SaveFile(Storage.CurrentFileName)` or `Storage.SaveOpenFiles()`.
3.  **`Storage.SaveOpenFiles()`**: Iterates through all open files and calls `SaveFile(fileName, false)` for each.
4.  **`Storage.SaveFile(string file, bool standalone)`**:
    -   Identifies the `fileIndex` for the given `file` name.
    -   Retrieves the `FileType` (`ftype`).
    -   Collects all `ItemDataWrapper.data` dictionaries from `openItems[fileIndex]` into an `object[] serialData`.
    -   Calls `Storage.Serialize(serialData, file, ftype)`.
5.  **`Storage.Serialize(object[] serialData, string file, FileType ftype)`**:
    -   Based on `ftype` (e.g., `RECIPES`, `ITEMS`, `ITEM_GROUPS`), calls specific methods like `SaveJsonRecipes` or `SaveJsonItem`.
    -   These methods use `JsonSchema.Serialize()` for each item in `serialData`.
6.  **`JsonSchema.Serialize(Dictionary<string, object> data, string typeKey)`**:
    -   `typeKey` is often derived from the item's "type" field (e.g., "GUN", "ARMOR") or is empty for generic items/item groups.
    -   Builds a JSON string for the single item, including only keys defined in the schema for that `typeKey` (and common keys from the schema's empty typeKey section) and present in the `data` dictionary.
    -   Uses `JavaScriptSerializer().Serialize(value)` for individual field values.
7.  **`SaveJsonItem`/`SaveJsonRecipes` (Continued)**:
    -   Assemble the full JSON string for the entire file (e.g., `[` item1_json, item2_json, ... `]`).
    -   Format the JSON string using `Storage.SpaceJson()` if `Options.DontFormatJson` is false.
    -   Write the final string to the file using `StreamWriter`.
    -   After successful save, `ItemDataWrapper.Modified` flags are reset for the saved items.

## 3. Key Interfaces and Types (Focus for Modification)

-   **To Add Support for a New JSON file type:**
    -   **`Storage.FileType` (enum):** Add new enum member.
    -   **`Storage.InitializeFileDefs()`:** Add a new `CataFile` definition:
        -   Specify `displayMember` (key used for display in lists).
        -   Specify `idKey` (primary identifier key).
        -   Create and link a `JsonSchema` instance, pointing to a new schema file in `CataclysmModder/schemas/`.
        -   Create a new `YourNewValuesControl` (UserControl) and assign it using `Storage.FileDefSetControl()`.
    -   **`YourNewValuesControl.cs`:** Design the UI and logic for editing this new type. Implement event handlers to call `Storage.ItemApplyValue()`.
    -   **`CataclysmModder/schemas/your_new_schema.txt`:** Create the schema file defining fields and their order.
    -   **`Storage.LoadFile()`:** May need custom logic if the JSON structure is not a simple array of objects.
    -   **`Storage.Serialize()`:** May need a new case or a more generic `SaveJsonItem` call if it fits the pattern.
    -   **`Form1.cs` (Constructor):** Instantiate `YourNewValuesControl`.
    -   **`Form1.cs` (Item Selection Logic):** If your new type has sub-types that require different "extension" controls (like how "GUN" or "ARMOR" items show specific panels), add logic to `entriesListBox_SelectedIndexChanged` or a helper method to manage visibility of these extension controls.
-   **To Change How an Existing Item Type is Edited (UI):**
    -   Locate the corresponding `*Values.cs` file (e.g., `GunValues.cs`).
    -   Modify its UI elements in the designer (`*.Designer.cs`) and its event handling logic in `*.cs`.
    -   Ensure calls to `Storage.ItemApplyValue()` are correct.
-   **To Change JSON Parsing/Loading for a File Type:**
    -   **`Storage.LoadFile()`**: Modify the section for the specific `FileType`.
-   **To Change JSON Serialization for a File Type:**
    -   **`Storage.Serialize()`**: Check if the existing `SaveJsonItem` or `SaveJsonRecipes` is sufficient.
    -   **`JsonSchema.cs` or `CataclysmModder/schemas/*.txt`**: Modify if field order, inclusion, or specific type formatting needs to change for serialization.
-   **To Change Core Data Structures or Item Handling:**
    -   **`ItemDataWrapper.cs`**: If the way item data is stored or tracked needs to change.
    -   **`Storage.cs`**: For fundamental changes to how lists of items or files are managed.

## 4. Framework Focus Points (Summary)

-   **`Storage.cs`**: The **absolute heart** of the application for data. Understand this class thoroughly if you want to make any changes to data handling, file types, or JSON processing.
-   **`*Values.cs` (and their `Designer.cs` files):** These are your targets for any UI changes related to editing specific data types. They are relatively self-contained UI panels.
-   **`Form1.cs`**: For overall application flow, main UI event handling, and coordination between `Storage` and the `*Values` controls.
-   **`ItemDataWrapper.cs`**: Key to understanding how individual JSON objects are represented and tracked in memory.
-   **`JsonSchema.cs` & `CataclysmModder/schemas/`**: Essential for understanding and controlling how data is written back to JSON files. The custom schema format is important here.
-   **`WinformsUtil.cs` (Assumed existence):** Likely contains helper methods for common UI tasks like populating controls from data dictionaries and resetting them. If it exists, it's worth reviewing for reusable UI logic.

This analysis should provide a solid foundation for navigating the CataclysmModder codebase and making targeted modifications.
