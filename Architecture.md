# CataclysmModder Project Architecture

This document outlines the software architecture of the CataclysmModder application, detailing its main components and the general flow of data.

## Main Components

The CataclysmModder application is built around several key C# components that manage data, user interface, and file operations:

*   **`Form1.cs`**:
    *   **Role**: This is the main application window, derived from `System.Windows.Forms.Form`.
    *   **Responsibilities**: It handles the overall UI layout, including menus, file selection, and item lists. It orchestrates user operations, such as opening game data folders, selecting files, and choosing items for editing. `Form1.cs` is also responsible for initializing and dynamically displaying the appropriate custom user controls (`*Values.cs` files) based on the type of data being edited.

*   **`Storage.cs`**:
    *   **Role**: A static class that serves as the central hub for all data management.
    *   **Responsibilities**:
        *   Loading and parsing JSON files from the Cataclysm: Dark Days Ahead 'raw' or 'data/json' directory.
        *   Storing the parsed JSON objects. Each JSON entry is typically wrapped in an `ItemDataWrapper` object.
        *   Managing collections of these `ItemDataWrapper` objects.
        *   Saving any modified data back to the respective JSON files.
        *   Defining and managing `FileType` enumerations. These enums categorize different types of JSON files (e.g., items, recipes, bionics).
        *   Holding `CataFile` definitions, which map `FileType`s to their specific properties, display members, and the associated UI controls used for editing them.

*   **`ItemDataWrapper.cs`**:
    *   **Role**: A wrapper class for dictionary objects, where each dictionary represents an individual JSON entry (e.g., a specific item, a recipe definition).
    *   **Responsibilities**:
        *   Holds the data for a single JSON entry.
        *   Implements the `INotifyPropertyChanged` interface. This allows UI elements (like fields in the `*Values.cs` controls) to bind to the data and automatically update when the underlying data changes, and vice-versa.
        *   Tracks the modification status of the wrapped data (i.e., whether it has unsaved changes).

*   **`*Values.cs` files (e.g., `GunValues.cs`, `ArmorValues.cs`, `ComestibleValues.cs`)**:
    *   **Role**: These are custom user controls, each derived from `System.Windows.Forms.UserControl`.
    *   **Responsibilities**: Each `*Values.cs` control is specifically designed to present and allow editing of data for a particular JSON structure. For example, `GunValues.cs` provides input fields relevant to gun properties (damage, ammo, etc.), while `ArmorValues.cs` provides fields for armor properties (coverage, material, etc.). `Form1.cs` dynamically loads and displays the correct `*Values.cs` control within its layout when an item of that type is selected.

*   **`JsonSchema.cs`**:
    *   **Role**: This class handles the serialization of data objects into JSON strings according to predefined schemas.
    *   **Responsibilities**:
        *   Loads schema definitions from embedded text files (e.g., `CataclysmModder.schemas.items.txt`). These schemas define the structure (keys, mandatory fields) for different JSON types.
        *   Provides a `Serialize` method that takes a dictionary of data and a type key, then constructs a JSON string. It iterates through the relevant schema keys to ensure the output JSON adheres to the defined structure. This is used by `Storage.cs` when saving data back to files.

## General Data Flow

The application follows a general data flow pattern:

1.  **Folder Selection**: The user selects a Cataclysm: Dark Days Ahead 'raw' (or 'data/json') folder using a dialog initiated from `Form1.cs`.
2.  **File Loading**: `Storage.cs` is invoked to load all `.json` files found within the selected directory and its subdirectories.
3.  **File List Population**: `Form1.cs` populates a dropdown menu (typically `filesComboBox`) with the names of the successfully loaded JSON files.
4.  **File Selection & Parsing**:
    *   The user selects a file from the dropdown in `Form1.cs`.
    *   `Storage.cs` is notified of the selection. It then parses the content of the selected JSON file. Each JSON object within the file (if it's an array of objects) or the root object itself is converted into an `ItemDataWrapper` object. These are stored in a list within `Storage.cs`.
5.  **Item List Display**: `Form1.cs` displays a list of the individual entries (now `ItemDataWrapper` objects) from the selected file in a list control (typically `entriesListBox`). The display name for each entry is often determined by a specific key within the JSON data, as configured in `CataFile` definitions.
6.  **Item Selection & Control Display**:
    *   The user selects an item from the `entriesListBox` in `Form1.cs`.
    *   `Form1.cs` instructs `Storage.cs` to set the current item (`Storage.LoadItem`).
    *   Based on the `FileType` of the current file (and potentially the type of the item itself), `Form1.cs` identifies the appropriate `*Values.cs` custom control (e.g., `GunValues` for a gun item, `RecipeValues` for a recipe).
    *   This specific `*Values.cs` control is made visible, and its fields are populated with the data from the selected `ItemDataWrapper` object. This data loading into the UI controls is often facilitated by a utility method like `WinformsUtil.ControlsLoadItem`.
7.  **Data Editing**:
    *   The user interacts with the fields within the active `*Values.cs` control to modify data.
    *   Changes made in the UI controls are propagated back to the underlying `ItemDataWrapper`'s dictionary. This is often handled through data binding or event handlers in the `*Values.cs` controls.
    *   `Storage.cs` is notified of these changes (e.g., via `Storage.ItemApplyValue`), and the `ItemDataWrapper` marks itself as modified. `Storage.cs` also tracks overall unsaved changes.
8.  **Saving Data**:
    *   The user initiates a save operation, either for the currently active file (`Storage.SaveFile`) or for all modified files (`Storage.SaveOpenFiles`), typically via menu options in `Form1.cs`.
    *   `Storage.cs` takes the `ItemDataWrapper` objects that have changes.
    *   For each item to be saved, its data dictionary is serialized back into a JSON string. This serialization process uses `JsonSchema.cs` to ensure the output conforms to the game's expected JSON structure for that data type.
    *   The resulting JSON strings are written back to their original files on disk, overwriting the previous content.
    *   The modification status of the saved items is reset.
