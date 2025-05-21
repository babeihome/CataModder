# Contributing to CataclysmModder

Thank you for considering contributing to CataclysmModder! Your help is appreciated in making this tool more robust and feature-rich for the Cataclysm: Dark Days Ahead modding community.

## Getting Started

Before diving into the code, here's some basic information:

*   **Technology Stack:** The project is built with **C#** and **Windows Forms** on the **.NET Framework 4.0**.
*   **Basic Setup:** Refer to the `README.md` file for instructions on how to get the project set up for development.
*   **Application Usage:** To understand how the application works from a user's perspective, please consult the `UserGuide.txt`.
*   **Understanding the Codebase:** To get familiar with the project's architecture, components, and file layout, please review the following documents:
    *   `ProjectOverview.md`: For a high-level understanding of the project's goals.
    *   `Architecture.md`: For details on the software architecture and data flow.
    *   `NavigationGuide.md`: For help locating key files and directories.

## Areas for Contribution

There are many ways you can contribute to the project. Here are some suggestions:

*   **Adding Support for New JSON File Types:**
    *   This is a significant way to enhance the tool's capabilities, allowing users to edit more aspects of the game.
    *   **Process:**
        1.  Understand the structure of the target JSON file from Cataclysm: Dark Days Ahead (e.g., by examining existing game files or community documentation).
        2.  Create a new custom `UserControl` (e.g., `MyNewTypeValues.cs`) similar to existing `*Values.cs` files. This control will provide the UI for editing the fields specific to the new data structure.
        3.  Add a new entry for your file type to the `Storage.FileType` enum in `Storage.cs`.
        4.  Define the new file type's properties (like its display member, associated schema, and the new UserControl) in `Storage.InitializeFileDefs()`. You might also need to associate the new `UserControl` with the `FileType` in `Form1.cs` (e.g., in `ShowControlForFileType`).
        5.  Implement or update JSON parsing/loading logic in `Storage.LoadFile()` if the new file type has a unique structure (e.g., not a simple array of objects, or requires special pre-processing).
        6.  Implement or update JSON serialization logic in `Storage.Serialize()` to correctly write the data back to a file. This may involve updating `JsonSchema.cs` or creating a new schema definition file in the `CataclysmModder/schemas/` directory.

*   **Improving Existing UI/UX:**
    *   Enhance the usability and clarity of the existing item editor forms (the `*Values.cs` files).
    *   Improve error handling mechanisms and provide more informative feedback messages to the user.
    *   Make the UI more intuitive or visually appealing, while keeping in mind the constraints and conventions of Windows Forms development.

*   **Enhancing JSON Handling:**
    *   Improve the robustness of the JSON parsing and serialization processes to gracefully handle unexpected data or edge cases.
    *   Add more detailed validation of data against the defined schemas to catch errors before saving.
    *   Optimize performance when loading, processing, or saving very large JSON files.

*   **Adding New Features:**
    *   Implement new tools or utilities that would be beneficial for modders (e.g., a batch editing feature, more advanced search and filtering capabilities across multiple files, data validation tools against game rules).
    *   Expand existing features, such as adding more options or formats to the item export functionality.

*   **Bug Fixes:**
    *   Identify and fix bugs that have been reported by users or discovered during development. This includes issues with data handling, UI behavior, or file operations.

*   **Writing Documentation:**
    *   Improve existing developer or user documentation.
    *   Add new guides or tutorials for specific features or contribution areas.

## General Workflow for Code Contributions

1.  **Understand the Task:**
    *   Thoroughly understand the requirements of the feature or bug fix.
    *   If dealing with JSON data, make sure you understand the specific JSON structures involved by looking at the game's files or relevant Cataclysm: Dark Days Ahead documentation.

2.  **Familiarize Yourself with Relevant Code:**
    *   **UI Changes:** `Form1.cs` (for main window interactions) and the specific `*Values.cs` control you are modifying or creating.
    *   **Data Logic / New File Types:** `Storage.cs` (core data handling), `ItemDataWrapper.cs` (individual JSON object wrapper), `JsonSchema.cs` (serialization logic), and potentially the `CataclysmModder/schemas/` directory.

3.  **Implement Changes:**
    *   **UI Controls:** If adding a new editor, create a new `UserControl`. Populate it with appropriate input fields (TextBoxes, ComboBoxes, CheckBoxes, etc.). Design the layout logically.
        *   Consider using helper functions from `WinformsUtil.cs` (if available and applicable, e.g., `WinformsUtil.ControlsLoadItem`, `WinformsUtil.ControlsResetValues`) to simplify UI updates.
    *   **Data Binding:** Ensure that data from the `ItemDataWrapper` is correctly loaded into your UI controls when an item is selected, and that changes made in the UI are saved back to the `ItemDataWrapper`'s underlying dictionary.
        *   Remember that `ItemDataWrapper.NotifyKeyChanged()` should be called after a value is programmatically changed in the data dictionary. This is crucial for the UI to reflect the change (if bound) and for `Storage.cs` to recognize that the file has unsaved changes.
    *   **Storage Updates (for new file types/structures):**
        *   Modify `Storage.cs` to handle any new file types. This includes adding to the `FileType` enum, creating a `CataFile` definition in `InitializeFileDefs()`, and updating loading (`LoadFile()`) and saving (`Serialize()`, `SaveJsonItem()`, etc.) logic as needed.
    *   **Schema:** If you are dealing with complex JSON structures or adding support for a new file type that requires specific formatting, you might need to create or update a schema definition file in the `CataclysmModder/schemas/` directory. Ensure `JsonSchema.cs` can correctly use this schema for serialization.

4.  **Test Thoroughly:**
    *   Load various relevant JSON files, including edge cases if possible.
    *   Edit different fields, testing different types of input.
    *   Save your changes and meticulously verify that the output JSON is correctly formatted and contains the data as expected by Cataclysm: Dark Days Ahead.
    *   Check for any unintended side effects in other parts of the application.
    *   Test loading the modified JSON back into the application to ensure it parses correctly.

5.  **Code Style:**
    *   Try to follow the existing code style (naming conventions, formatting, commenting) for consistency. While there isn't a formal style guide provided for this project, observing the patterns in the existing codebase is the best approach.

6.  **Submit Your Contribution:** (The project's contribution mechanism isn't explicitly defined, but assuming a standard GitHub workflow):
    *   Fork the repository on GitHub.
    *   Create a new branch in your fork for your feature or bug fix (e.g., `feature/add-new-item-type` or `fix/resolve-recipe-bug`).
    *   Commit your changes with clear and descriptive commit messages. Each commit should ideally represent a logical unit of work.
    *   Push your branch to your fork on GitHub.
    *   Open a Pull Request (PR) from your branch to the main repository's `master` or `main` branch.
    *   Clearly describe the changes you made, the problem you solved, and any specific testing instructions in the Pull Request description.

## Important Considerations

*   **Backup Game Files:** **Crucially important!** Always back up your Cataclysm: Dark Days Ahead `data/json` or `raw` folder before testing your development changes with the modder. Incorrectly formatted or structured JSON can cause issues with the game, including crashes or data corruption.
*   **Error Handling:** Implement robust error handling, especially around file I/O operations (reading/writing files) and JSON parsing/serialization. Aim to prevent application crashes and provide informative error messages to the user.
*   **User Feedback:** Ensure the UI provides good feedback to the user. For example, indicate when operations are in progress, when they complete successfully, or if an error occurs.

By following these guidelines, you can help contribute effectively to the CataclysmModder project. Thank you for your interest!
