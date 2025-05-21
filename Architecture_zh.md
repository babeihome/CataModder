# CataclysmModder 项目架构

本文档概述了 CataclysmModder 应用程序的软件架构，详细介绍了其主要组件和常规数据流。

## 主要组件

CataclysmModder 应用程序围绕几个关键的 C# 组件构建，这些组件负责管理数据、用户界面和文件操作：

*   **`Form1.cs`**:
    *   **角色**: 这是主应用程序窗口，派生自 `System.Windows.Forms.Form`。
    *   **职责**: 它处理整体 UI 布局，包括菜单、文件选择和物品列表。它协调用户操作，例如打开游戏数据文件夹、选择文件以及选择要编辑的物品。`Form1.cs` 还负责根据正在编辑的数据类型初始化并动态显示相应的自定义用户控件（`*Values.cs` 文件）。

*   **`Storage.cs`**:
    *   **角色**: 一个静态类，作为所有数据管理的中心枢纽。
    *   **职责**:
        *   从《Cataclysm: Dark Days Ahead》的 'raw' 或 'data/json' 目录加载和解析 JSON 文件。
        *   存储解析后的 JSON 对象。每个 JSON 条目通常包装在一个 `ItemDataWrapper` 对象中。
        *   管理这些 `ItemDataWrapper` 对象的集合。
        *   将任何已修改的数据保存回相应的 JSON 文件。
        *   定义和管理 `FileType` 枚举。这些枚举对不同类型的 JSON 文件（例如，物品、配方、仿生体）进行分类。
        *   持有 `CataFile` 定义，这些定义将 `FileType` 映射到其特定属性、显示成员以及用于编辑它们的关联 UI 控件。

*   **`ItemDataWrapper.cs`**:
    *   **角色**: 字典对象的包装类，其中每个字典代表一个单独的 JSON 条目（例如，一个特定的物品、一个配方定义）。
    *   **职责**:
        *   持有单个 JSON 条目的数据。
        *   实现 `INotifyPropertyChanged` 接口。这允许 UI 元素（如 `*Values.cs` 控件中的字段）绑定到数据，并在基础数据更改时自动更新，反之亦然。
        *   跟踪包装数据的修改状态（即，是否有未保存的更改）。

*   **`*Values.cs` 文件 (例如 `GunValues.cs`, `ArmorValues.cs`, `ComestibleValues.cs`)**:
    *   **角色**: 这些是自定义用户控件，每个都派生自 `System.Windows.Forms.UserControl`。
    *   **职责**: 每个 `*Values.cs` 控件都专门设计用于显示和编辑特定 JSON 结构的数据。例如，`GunValues.cs` 提供与枪支属性（伤害、弹药等）相关的输入字段，而 `ArmorValues.cs` 提供与盔甲属性（覆盖范围、材料等）相关的字段。当选择该类型的物品时，`Form1.cs` 会在其布局中动态加载并显示正确的 `*Values.cs` 控件。

*   **`JsonSchema.cs`**:
    *   **角色**: 此类根据预定义的模式处理数据对象到 JSON 字符串的序列化。
    *   **职责**:
        *   从嵌入的文本文件（例如 `CataclysmModder.schemas.items.txt`）加载模式定义。这些模式定义了不同 JSON 类型的结构（键、强制字段）。
        *   提供一个 `Serialize` 方法，该方法接受数据字典和类型键，然后构造一个 JSON 字符串。它会遍历相关的模式键，以确保输出的 JSON 符合定义的结构。`Storage.cs` 在将数据保存回文件时使用此功能。

## 常规数据流

应用程序遵循常规的数据流模式：

1.  **文件夹选择**: 用户使用从 `Form1.cs` 初始化的对话框选择一个《Cataclysm: Dark Days Ahead》的 'raw' (或 'data/json') 文件夹。
2.  **文件加载**: 调用 `Storage.cs` 以加载所选目录及其子目录中找到的所有 `.json` 文件。
3.  **文件列表填充**: `Form1.cs` 使用成功加载的 JSON 文件的名称填充下拉菜单（通常是 `filesComboBox`）。
4.  **文件选择与解析**:
    *   用户从 `Form1.cs` 中的下拉菜单中选择一个文件。
    *   `Storage.cs` 收到选择通知。然后它会解析所选 JSON 文件的内容。文件中的每个 JSON 对象（如果是对象数组）或根对象本身都会被转换为一个 `ItemDataWrapper` 对象。这些对象存储在 `Storage.cs` 内的一个列表中。
5.  **物品列表显示**: `Form1.cs` 在列表控件（通常是 `entriesListBox`）中显示所选文件中各个条目（现在是 `ItemDataWrapper` 对象）的列表。每个条目的显示名称通常由 JSON 数据中的特定键确定，该键在 `CataFile` 定义中配置。
6.  **物品选择与控件显示**:
    *   用户从 `Form1.cs` 中的 `entriesListBox` 中选择一个物品。
    *   `Form1.cs` 指示 `Storage.cs` 设置当前物品 (`Storage.LoadItem`)。
    *   根据当前文件的 `FileType`（以及可能的物品本身的类型），`Form1.cs` 确定相应的 `*Values.cs` 自定义控件（例如，枪支物品对应 `GunValues`，配方对应 `RecipeValues`）。
    *   这个特定的 `*Values.cs` 控件变为可见，并且其字段使用所选 `ItemDataWrapper` 对象中的数据填充。将数据加载到 UI 控件中的过程通常由像 `WinformsUtil.ControlsLoadItem` 这样的实用方法辅助完成。
7.  **数据编辑**:
    *   用户与活动的 `*Values.cs` 控件中的字段进行交互以修改数据。
    *   在 UI 控件中所做的更改会传播回底层的 `ItemDataWrapper` 的字典中。这通常通过 `*Values.cs` 控件中的数据绑定或事件处理程序来处理。
    *   `Storage.cs` 收到这些更改的通知（例如，通过 `Storage.ItemApplyValue`），并且 `ItemDataWrapper` 将自身标记为已修改。`Storage.cs` 还跟踪整体未保存的更改。
8.  **数据保存**:
    *   用户启动保存操作，可以保存当前活动文件 (`Storage.SaveFile`) 或所有已修改的文件 (`Storage.SaveOpenFiles`)，通常通过 `Form1.cs` 中的菜单选项进行。
    *   `Storage.cs` 获取已更改的 `ItemDataWrapper` 对象。
    *   对于要保存的每个物品，其数据字典都会被序列化回 JSON 字符串。此序列化过程使用 `JsonSchema.cs` 来确保输出符合该数据类型的游戏预期 JSON 结构。
    *   生成的 JSON 字符串被写回其磁盘上的原始文件，覆盖先前的内容。
    *   已保存物品的修改状态被重置。
