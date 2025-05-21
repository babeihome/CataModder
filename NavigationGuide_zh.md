# CataclysmModder 导航指南

欢迎来到 CataclysmModder 项目！本指南旨在帮助新开发者理解项目结构，定位关键文件和目录，并开始熟悉代码库。

项目使用 **C#** 语言，基于 **.NET Framework** (具体来说是用户指南中提到的 .NET 4.0) 和 **Windows Forms** (用于图形用户界面)。

## 关键目录和文件

以下是 CataclysmModder 仓库中重要目录和文件的明细：

*   **`CataclysmModder/`**: 这是包含所有 C# 项目文件和应用程序源代码的主目录。
    *   **`CataclysmModder.sln`**: Visual Studio 解决方案文件。在 Visual Studio 中打开此文件以加载整个项目，包括其所有设置和依赖项。
    *   **`CataclysmModder/CataclysmModder.csproj`**: C# 项目文件。它定义了项目特定的设置，例如目标 .NET Framework 版本、对外部库的依赖（如果有），以及构建过程中包含的所有源代码文件的完整列表。
    *   **`CataclysmModder/Program.cs`**: 包含 `Main` 静态方法，这是应用程序执行时的初始入口点。其主要职责是初始化并运行主应用程序窗口 (`Form1`)。
    *   **`CataclysmModder/Form1.cs`**: 此文件定义了应用程序的主窗口 (窗体)。这是主要用户界面的布局和管理位置。`Form1` 处理用户交互的事件逻辑（例如，按钮点击、菜单选择），并负责协调各种数据编辑用户控件的显示。
    *   **`CataclysmModder/Form1.Designer.cs`**: 这是 `Form1.cs` 的自动生成的伴随文件。它包含声明和初始化 UI 元素（按钮、文本框、列表等）并指定其在 Windows Forms 可视化设计器中设计的布局属性的代码。通常不建议手动编辑此文件；应使用 Visual Studio 设计器对 `Form1.cs` 进行更改。
    *   **`CataclysmModder/Storage.cs`**: 一个关键的静态类，充当所有数据管理操作的中心枢纽。其职责广泛，包括：
        *   从游戏的数据目录加载 JSON 文件。
        *   将原始 JSON 文本解析为内部结构化数据表示（主要使用 `List<BindingList<ItemDataWrapper>>`）。
        *   在整个应用程序中存储并提供对这些已加载数据的访问。
        *   将用户所做的任何修改保存回原始 JSON 文件。
        *   定义一个枚举 (`FileType`) 以对不同类型的游戏数据文件（例如，物品、配方、仿生体）进行分类。
        *   持有 `CataFile` 对象，这些对象描述了每种 `FileType` 的属性，例如主显示键和关联的编辑控件。
        *   管理并为 UI 控件提供数据源（例如，可用材料、技能列表），通常用于下拉列表或自动完成建议。
    *   **`CataclysmModder/ItemDataWrapper.cs`**: 定义一个包装单个 JSON 对象（例如，单个物品定义、特定配方）的类。每个 `ItemDataWrapper` 实例主要将其数据保存为 `Dictionary<string, object>`。它实现了 `INotifyPropertyChanged` 接口，这对于 Windows Forms 中的数据绑定至关重要，允许 UI 元素在基础数据更改时自动刷新。
    *   **`CataclysmModder/*Values.cs` 文件** (例如 `GunValues.cs`, `ArmorValues.cs`, `ComestibleValues.cs`, `RecipeControl.cs`, `MonsterValues.cs` 等): 此文件集合代表自定义 `UserControl` 类。每个控件都专门设计用于为《Cataclysm: Dark Days Ahead》中找到的特定类型的 JSON 对象提供用户界面。例如，`GunValues.cs` 将包含与枪支属性相关的字段，而 `ArmorValues.cs` 将包含与盔甲相关的字段。这些控件根据用户选择的文件或物品类型在 `Form1` 中动态加载和显示。
        *   每个 `*Values.cs` 文件都有一个对应的 **`*Values.Designer.cs`** 文件，该文件由 Windows Forms 设计器自动生成，并包含该特定用户控件的 UI 布局代码。
    *   **`CataclysmModder/JsonSchema.cs`**: 此类负责将内部数据（来自 `ItemDataWrapper` 对象的字典）序列化回格式良好的 JSON 字符串。它使用模式定义来确保输出的 JSON 符合游戏对每种数据类型的预期结构。
    *   **`CataclysmModder/schemas/`**: 此目录包含定义 JSON 模式的文本文件 (`.txt`)。示例包括 `items.txt`、`recipes.txt` 等。这些模式由 `JsonSchema.cs` 读取，以指导 JSON 序列化过程，确保输出文件具有正确的结构和字段。
    *   **`CataclysmModder/Properties/`**: 这个标准的 C# 项目目录包含项目级别的元数据和资源。
        *   `AssemblyInfo.cs`: 包含程序集元数据，如版本、标题等。
        *   `Resources.resx`: 项目的主要资源文件。它可以存储字符串、图像、图标和其他嵌入式资源。虽然模式主要位于 `schemas/` 目录中，但某些资源可能嵌入在此处。
        *   `Settings.settings`: 用于应用程序级别的设置。
    *   **`CataclysmModder/About.cs`**: 定义“关于”对话框窗口，通常显示有关应用程序、其版本和作者的信息。
    *   **`CataclysmModder/Options.cs`**: 定义用于配置应用程序特定选项或设置的窗体。
    *   **`CataclysmModder/ExportItemsForm.cs`**: 定义一个可能提供以特定格式导出游戏物品或数据功能的窗体。

*   **`README.md`**: (已存在) 位于仓库的根目录。它提供了 CataclysmModder 项目的总体概述、其目的以及基本设置或使用信息。
*   **`UserGuide.txt`**: (已存在) 位于根目录。此文本文件从最终用户的角度提供了有关如何使用 CataclysmModder 应用程序的更详细说明。
*   **`ProjectOverview.md`**: (新建) 位于根目录。此 Markdown 文件专门为开发者提供了项目的高级介绍，概述了其目标和范围。
*   **`Architecture.md`**: (新建) 位于根目录。本文档详细介绍了 CataclysmModder 的软件架构，包括对其主要组件和应用程序内常规数据流的描述。

本指南应为导航 CataclysmModder 代码库提供一个坚实的起点。请记住从 `Program.cs` 和 `Form1.cs` 开始探索代码，以了解应用程序的生命周期和主要交互。
