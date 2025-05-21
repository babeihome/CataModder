# CataclysmModder 详细代码分析

本文档深入探讨 CataclysmModder 代码库，重点介绍关键类关系、数据流和核心组件，旨在帮助开发人员理解和修改此应用程序。

## 1. 关键类关系

描述主要类及其交互方式：

-   **`Form1` (主 UI 窗口):**
    -   实例化并管理各种自定义 `UserControl` 的派生类（例如 `GunValues`, `ArmorValues`, `ComestibleValues`, `GenericItemValues`, `ItemGroupValues`, `RecipeControl`, `ProfessionValues`, `VehiclePartValues`）。这些都作为成员变量存储。
    -   与静态 `Storage` 类进行大量交互，以执行所有后端数据操作（加载文件、获取物品数据、保存文件）。
    -   处理 UI 事件，并将操作委托给 `Storage` 或根据来自 `Storage` 的数据更新 UI 元素。
    -   使用 `WinformsUtil` 执行某些 UI 辅助功能。
-   **`Storage` (静态数据管理中心):**
    -   管理打开的 JSON 文件列表 (`openFiles`) 及其解析后的内容 (`openItems`，一个 `List<BindingList<ItemDataWrapper>>`)。
    -   包含不同文件类型的定义 (`FileType` 枚举) 及其属性 (`CataFile` 对象，这些对象链接到 `JsonSchema` 和 UI 控件)。
    -   使用 `JavaScriptSerializer` 将 JSON 反序列化为 `Dictionary<string, object>`。
    -   使用 `JsonSchema` 将 `ItemDataWrapper` 数据序列化回格式化的 JSON 字符串。
    -   提供用于访问和修改已加载数据的静态方法（例如 `LoadFiles`, `SelectFile`, `LoadItem`, `ItemApplyValue`, `SaveFile`）。
-   **`ItemDataWrapper` (JSON 物品表示):**
    -   包装一个 `Dictionary<string, object>`，该字典持有单个 JSON 条目（例如一个物品、一个配方）的实际数据。
    -   实现 `INotifyPropertyChanged` 接口，以向 UI 元素（特别是 `Form1` 中的 `entriesListBox`）发送更改信号。`Display` 属性对此至关重要。
    -   跟踪修改状态 (`Modified` 标志)。
    -   `memberOf` 字段将其链接到它所属文件在 `Storage.openFiles` 中的索引。
-   **`*Values.cs` 控件 (特定物品编辑器):**
    -   这些是 `UserControl` 类，每个都经过定制以编辑特定类型的 JSON 对象（例如 `GunValues` 用于 "GUN" 物品，`ComestibleValues` 用于 "COMESTIBLE" 物品）。
    -   它们包含各种输入元素（TextBoxes、ComboBoxes 等）。
    -   数据通常使用辅助方法从 `ItemDataWrapper` 的字典加载到这些控件中（通常通过 `Storage.LoadItem` 调用的 `WinformsUtil.ControlsLoadItem`）。
    -   这些控件中的更改应触发 `Storage.ItemApplyValue` 以更新底层的 `ItemDataWrapper` 并标记更改。
-   **`JsonSchema` (JSON 序列化逻辑):**
    -   从嵌入的文本文件（例如 `CataclysmModder.schemas.items.txt`）加载自定义模式定义。这些模式定义了不同物品类型的键的顺序和包含关系。
    -   `Serialize` 方法根据给定物品类型的加载模式，从 `ItemDataWrapper` 的数据字典构造一个 JSON 字符串。
-   **`CataFile` (文件类型定义):**
    -   由 `Storage` 使用的辅助类，用于定义支持的 JSON 文件类型的属性，例如主 ID 键、显示成员、关联的 `JsonSchema` 以及用于编辑其条目的主 `Control`（一个 `*Values` 用户控件）。

## 2. 数据流分析

描述主要操作的事件顺序和组件交互。

### 2.1. 应用程序启动和初始文件加载 (如果 `.conf` 文件存在)
1.  **`Program.Main()`**: 应用程序入口点。创建并运行 `Form1`。
2.  **`Form1.Form1()` (构造函数)**:
    -   初始化 `Storage.InitializeFileDefs()` 以设置 `CataFile` 定义。
    -   实例化所有 `*Values.cs` 控件和其他特定控件，如 `ItemGroupValues`、`RecipeControl` 等。通过 `Storage.FileDefSetControl()` 将它们与 `Storage.FileType` 相关联。
    -   检查 `.conf` 文件。如果存在：
        -   读取游戏 'raw' 文件夹的路径。
        -   调用 `Form1.loadFiles(path)`。
3.  **`Form1.loadFiles(string path)`**:
    -   将路径存储在 `.conf` 文件中。
    -   调用 `Storage.LoadFiles(path)`。
4.  **`Storage.LoadFiles(string path)`**:
    -   从目录（和子目录）获取所有 `*.json` 文件路径。
    -   对于每个文件：
        -   调用 `Storage.LoadFile(fileIndex)`。
5.  **`Storage.LoadFile(int index)`**:
    -   读取 JSON 文件内容。
    -   使用 `JavaScriptSerializer().DeserializeObject(json)` 将 JSON 解析为 `object[]`（字典数组）或单个字典（对于配方）。
    -   对于文件中的每个 JSON 对象（字典），创建一个 `ItemDataWrapper` 实例，存储字典和 `memberOf` 索引。
    -   将这些 `ItemDataWrapper` 实例添加到 `openItems` 中相应的 `BindingList<ItemDataWrapper>`。
    -   如果文件类型是 `ITEMS` 或 `BIONICS`，则订阅 `ListChanged` 事件以进行自动完成更新。
6.  **`Form1.loadFiles()` (续)**:
    -   使用 `Storage.OpenFiles` 填充 `filesComboBox`。
    -   如果已加载文件，则在 `filesComboBox` 中选择第一个文件，触发 `filesComboBox_SelectedIndexChanged`。

### 2.2. 从下拉列表中选择 JSON 文件
1.  **`Form1.filesComboBox_SelectedIndexChanged`**:
    -   调用 `Storage.SelectFile(filesComboBox.SelectedIndex)` 以设置 `Storage.currentFileIndex`。
    -   隐藏所有 `*Values` 和其他主面板控件。
    -   通过 `Storage.GetFileDefForCurrentFile()` 获取当前文件类型的 `CataFile` 定义。
    -   如果存在 `CataFile` 及其关联控件，则使该控件可见。
    -   将 `entriesListBox.DataSource` 设置为 `Storage.OpenItems` (它引用 `openItems[currentFileIndex]`)。
    -   如果有物品，则在 `entriesListBox` 中选择第一个物品，触发 `entriesListBox_SelectedIndexChanged`。

### 2.3. 从列表中选择物品
1.  **`Form1.entriesListBox_SelectedIndexChanged`**:
    -   如果所选索引有效且不同于 `Storage.CurrentItemIndex`：
        -   调用 `Storage.LoadItem(entriesListBox.SelectedIndex)`。
2.  **`Storage.LoadItem(int index)`**:
    -   设置 `currentItemIndex = index`。
    -   获取当前文件的 `CataFile`。
    -   如果 `CataFile.control` 不为 null（即关联了 `*Values` 控件）：
        -   调用 `WinformsUtil.ControlsLoadItem(CataFile.control, CurrentItemData)`。此方法（推测在 `WinformsUtil.cs` 中，尚未明确审查但可推断）负责使用 `CurrentItemData`（即 `openItems[currentFileIndex][currentItemIndex].data`）中的数据填充特定 `*Values` 控件的字段。
    -   根据物品的 "type" 属性动态显示/隐藏特定的“扩展”控件（如 `GunControl`、`ArmorControl`）（例如，如果物品 `type` 为 "GUN"，则 `Form1.GunControl` 变为可见）。此逻辑主要在 `Storage.LoadItem` 完成后，通过检查 `CurrentItemData["type"]` 在 `Form1` 中执行。

### 2.4. 在 `*Values` 控件中编辑数据
1.  **用户交互**: 用户在可见的 `*Values` 控件（例如 `GunValues1.nameTextBox`）内的字段（例如 TextBox、ComboBox）中更改值。
2.  **事件处理程序**: 触发该输入字段的控件事件处理程序（例如 `TextChanged`、`SelectedIndexChanged`）。
3.  **`Storage.ItemApplyValue(string key, object value, bool mandatory)`**:
    -   `*Values` 控件中的事件处理程序*应该*调用此方法。
    -   `key`: 正在更改的属性的 JSON 键（例如 "name"、"weight"）。
    -   `value`: 输入字段中的新值。
    -   `mandatory`: 指示该字段是否为必填项（如果值为空，则影响删除）。
    -   此方法更新当前 `ItemDataWrapper` (`Storage.CurrentItemData`) 内的 `Dictionary<string, object>`。
    -   设置 `Storage.unsavedChanges = true`。
    -   调用 `openItems[currentFileIndex][currentItemIndex].NotifyKeyChanged(key)`。
4.  **`ItemDataWrapper.NotifyKeyChanged(string key)`**:
    -   设置 `this.Modified = true`。
    -   如果更改的键影响显示名称（来自 `CataFile` 的 `displayMember` 或 `displaySuffix`），则以 "Display" 作为参数引发 `PropertyChanged` 事件。如果物品的显示名称更改，这将更新 `Form1.entriesListBox`。

### 2.5. 保存文件
1.  **用户操作**: 点击“保存文件”或“全部保存”。
2.  **`Form1.saveFileToolStripMenuItem_Click` 或 `Form1.saveAllToolStripMenuItem_Click`**:
    -   调用 `Storage.SaveFile(Storage.CurrentFileName)` 或 `Storage.SaveOpenFiles()`。
3.  **`Storage.SaveOpenFiles()`**: 遍历所有打开的文件，并为每个文件调用 `SaveFile(fileName, false)`。
4.  **`Storage.SaveFile(string file, bool standalone)`**:
    -   识别给定 `file` 名称的 `fileIndex`。
    -   检索 `FileType` (`ftype`)。
    -   将 `openItems[fileIndex]` 中的所有 `ItemDataWrapper.data` 字典收集到 `object[] serialData` 中。
    -   调用 `Storage.Serialize(serialData, file, ftype)`。
5.  **`Storage.Serialize(object[] serialData, string file, FileType ftype)`**:
    -   根据 `ftype`（例如 `RECIPES`、`ITEMS`、`ITEM_GROUPS`），调用特定的方法，如 `SaveJsonRecipes` 或 `SaveJsonItem`。
    -   这些方法对 `serialData` 中的每个物品使用 `JsonSchema.Serialize()`。
6.  **`JsonSchema.Serialize(Dictionary<string, object> data, string typeKey)`**:
    -   `typeKey` 通常从物品的 "type" 字段派生（例如 "GUN"、"ARMOR"），或者对于通用物品/物品组为空。
    -   为单个物品构建 JSON 字符串，仅包括该 `typeKey` 的模式中定义的键（以及模式的空 typeKey 部分中的公共键）并且存在于 `data` 字典中的键。
    -   对单个字段值使用 `JavaScriptSerializer().Serialize(value)`。
7.  **`SaveJsonItem`/`SaveJsonRecipes` (续)**:
    -   为整个文件组装完整的 JSON 字符串（例如 `[` item1_json, item2_json, ... `]`）。
    -   如果 `Options.DontFormatJson` 为 false，则使用 `Storage.SpaceJson()` 格式化 JSON 字符串。
    -   使用 `StreamWriter` 将最终字符串写入文件。
    -   成功保存后，将重置已保存物品的 `ItemDataWrapper.Modified` 标志。

## 3. 关键接口和类型 (修改重点)

-   **要添加对新 JSON 文件类型的支持：**
    -   **`Storage.FileType` (枚举):** 添加新的枚举成员。
    -   **`Storage.InitializeFileDefs()`:** 添加新的 `CataFile` 定义：
        -   指定 `displayMember` (用于在列表中显示的键)。
        -   指定 `idKey` (主标识符键)。
        -   创建并链接一个 `JsonSchema` 实例，指向 `CataclysmModder/schemas/` 中的新模式文件。
        -   创建一个新的 `YourNewValuesControl` (UserControl) 并使用 `Storage.FileDefSetControl()` 分配它。
    -   **`YourNewValuesControl.cs`:** 设计用于编辑此新类型的 UI 和逻辑。实现事件处理程序以调用 `Storage.ItemApplyValue()`。
    -   **`CataclysmModder/schemas/your_new_schema.txt`:** 创建定义字段及其顺序的模式文件。
    -   **`Storage.LoadFile()`:** 如果 JSON 结构不是简单的对象数组，则可能需要自定义逻辑。
    -   **`Storage.Serialize()`:** 如果模式适合，则可能需要新的 case 或更通用的 `SaveJsonItem` 调用。
    -   **`Form1.cs` (构造函数):** 实例化 `YourNewValuesControl`。
    -   **`Form1.cs` (物品选择逻辑):** 如果您的新类型具有需要不同“扩展”控件的子类型（例如 "GUN" 或 "ARMOR" 物品如何显示特定面板），请向 `entriesListBox_SelectedIndexChanged` 或辅助方法添加逻辑以管理这些扩展控件的可见性。
-   **要更改现有物品类型的编辑方式 (UI)：**
    -   找到相应的 `*Values.cs` 文件（例如 `GunValues.cs`）。
    -   在设计器 (`*.Designer.cs`) 中修改其 UI 元素，并在 `*.cs` 中修改其事件处理逻辑。
    -   确保对 `Storage.ItemApplyValue()` 的调用是正确的。
-   **要更改文件类型的 JSON 解析/加载：**
    -   **`Storage.LoadFile()`**: 修改特定 `FileType` 的部分。
-   **要更改文件类型的 JSON 序列化：**
    -   **`Storage.Serialize()`**: 检查现有的 `SaveJsonItem` 或 `SaveJsonRecipes` 是否足够。
    -   **`JsonSchema.cs` 或 `CataclysmModder/schemas/*.txt`**: 如果字段顺序、包含关系或特定类型格式需要为序列化而更改，请进行修改。
-   **要更改核心数据结构或物品处理：**
    -   **`ItemDataWrapper.cs`**: 如果物品数据的存储或跟踪方式需要更改。
    -   **`Storage.cs`**: 用于对物品或文件列表的管理方式进行根本性更改。

## 4. 框架焦点 (摘要)

-   **`Storage.cs`**: 应用程序数据的**绝对核心**。如果您想对数据处理、文件类型或 JSON 处理进行任何更改，请彻底了解此类。
-   **`*Values.cs` (及其 `Designer.cs` 文件):** 这些是您进行与编辑特定数据类型相关的任何 UI 更改的目标。它们是相对独立的 UI 面板。
-   **`Form1.cs`**: 用于整体应用程序流程、主 UI 事件处理以及 `Storage` 和 `*Values` 控件之间的协调。
-   **`ItemDataWrapper.cs`**: 理解单个 JSON 对象如何在内存中表示和跟踪的关键。
-   **`JsonSchema.cs` & `CataclysmModder/schemas/`**: 理解和控制数据如何写回 JSON 文件的基础。自定义模式格式在此非常重要。
-   **`WinformsUtil.cs` (假定存在):** 可能包含用于常见 UI 任务（如从数据字典填充控件并重置它们）的辅助方法。如果存在，则值得审查其可重用 UI 逻辑。

此分析应为导航 CataclysmModder 代码库和进行有针对性的修改提供坚实的基础。
