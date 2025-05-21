# 为 CataclysmModder 做贡献

感谢您考虑为 CataclysmModder 做出贡献！您的帮助将使这个工具更加健壮和功能丰富，更好地服务于《Cataclysm: Dark Days Ahead》的模组制作社区。

## 入门指南

在深入代码之前，请了解以下基本信息：

*   **技术栈:** 项目使用 **C#** 和 **Windows Forms** 构建，基于 **.NET Framework 4.0**。
*   **基本设置:** 请参阅 `README.md` 文件，了解如何设置项目以进行开发。
*   **应用程序用法:** 要从用户角度了解应用程序的工作方式，请查阅 `UserGuide.txt`。
*   **理解代码库:** 要熟悉项目架构、组件和文件布局，请查看以下文档：
    *   `ProjectOverview.md`: 用于高层次理解项目目标。
    *   `Architecture.md`: 用于了解软件架构和数据流的详细信息。
    *   `NavigationGuide.md`: 用于帮助定位关键文件和目录。

## 贡献领域

您可以通过多种方式为项目做出贡献。以下是一些建议：

*   **添加对新 JSON 文件类型的支持:**
    *   这是增强工具功能的重要途径，允许用户编辑游戏的更多方面。
    *   **流程:**
        1.  了解目标 JSON 文件在《Cataclysm: Dark Days Ahead》中的结构（例如，通过检查现有的游戏文件或社区文档）。
        2.  创建一个新的自定义 `UserControl`（例如 `MyNewTypeValues.cs`），类似于现有的 `*Values.cs` 文件。此控件将提供用于编辑新数据结构特定字段的 UI。
        3.  在 `Storage.cs` 的 `Storage.FileType` 枚举中为您的文件类型添加一个新条目。
        4.  在 `Storage.InitializeFileDefs()` 中定义新文件类型的属性（例如其显示成员、关联的模式和新的 UserControl）。您可能还需要在 `Form1.cs` 中将新的 `UserControl` 与 `FileType` 相关联（例如，在 `ShowControlForFileType` 中）。
        5.  如果新文件类型具有独特的结构（例如，不是简单的对象数组，或需要特殊的预处理），则在 `Storage.LoadFile()` 中实现或更新 JSON 解析/加载逻辑。
        6.  在 `Storage.Serialize()` 中实现或更新 JSON 序列化逻辑，以正确地将数据写回文件。这可能涉及更新 `JsonSchema.cs` 或在 `CataclysmModder/schemas/` 目录中创建一个新的模式定义文件。

*   **改进现有的 UI/UX:**
    *   增强现有物品编辑器窗体（`*Values.cs` 文件）的可用性和清晰度。
    *   改进错误处理机制，并向用户提供更具信息性的反馈消息。
    *   在牢记 Windows Forms 开发的约束和惯例的同时，使 UI 更直观或视觉上更具吸引力。

*   **增强 JSON 处理:**
    *   提高 JSON 解析和序列化过程的稳健性，以优雅地处理意外数据或边缘情况。
    *   添加更详细的数据验证功能，以根据定义的模式在保存前捕获错误。
    *   优化加载、处理或保存非常大的 JSON 文件时的性能。

*   **添加新功能:**
    *   实现对模组制作者有益的新工具或实用程序（例如，批量编辑功能、跨多个文件的更高级搜索和筛选功能、针对游戏规则的数据验证工具）。
    *   扩展现有功能，例如为物品导出功能添加更多选项或格式。

*   **错误修复:**
    *   识别并修复用户报告或在开发过程中发现的错误。这包括与数据处理、UI 行为或文件操作相关的问题。

*   **编写文档:**
    *   改进现有的开发者或用户文档。
    *   为特定功能或贡献领域添加新的指南或教程。

## 代码贡献的一般工作流程

1.  **理解任务:**
    *   彻底理解功能或错误修复的需求。
    *   如果处理 JSON 数据，请通过查看游戏文件或相关的《Cataclysm: Dark Days Ahead》文档，确保您了解所涉及的特定 JSON 结构。

2.  **熟悉相关代码:**
    *   **UI 更改:** `Form1.cs`（用于主窗口交互）以及您正在修改或创建的特定 `*Values.cs` 控件。
    *   **数据逻辑 / 新文件类型:** `Storage.cs`（核心数据处理）、`ItemDataWrapper.cs`（单个 JSON 对象包装器）、`JsonSchema.cs`（序列化逻辑）以及可能的 `CataclysmModder/schemas/` 目录。

3.  **实施更改:**
    *   **UI 控件:** 如果添加新的编辑器，请创建一个新的 `UserControl`。用适当的输入字段（TextBoxes、ComboBoxes、CheckBoxes 等）填充它。合理设计布局。
        *   考虑使用 `WinformsUtil.cs` 中的辅助函数（如果可用且适用，例如 `WinformsUtil.ControlsLoadItem`、`WinformsUtil.ControlsResetValues`）来简化 UI 更新。
    *   **数据绑定:** 确保在选择物品时将 `ItemDataWrapper` 中的数据正确加载到 UI 控件中，并且将在 UI 中所做的更改保存回 `ItemDataWrapper` 的底层字典中。
        *   请记住，在以编程方式更改数据字典中的值后，应调用 `ItemDataWrapper.NotifyKeyChanged()`。这对于 UI 反映更改（如果已绑定）以及 `Storage.cs` 识别文件具有未保存的更改至关重要。
    *   **存储更新 (对于新的文件类型/结构):**
        *   修改 `Storage.cs` 以处理任何新的文件类型。这包括添加到 `FileType` 枚举，在 `InitializeFileDefs()` 中创建 `CataFile` 定义，以及根据需要更新加载 (`LoadFile()`) 和保存 (`Serialize()`、`SaveJsonItem()` 等) 逻辑。
    *   **模式:** 如果您正在处理复杂的 JSON 结构或添加对需要特定格式的新文件类型的支持，则可能需要创建或更新 `CataclysmModder/schemas/` 目录中的模式定义文件。确保 `JsonSchema.cs` 可以正确使用此模式进行序列化。

4.  **彻底测试:**
    *   加载各种相关的 JSON 文件，如果可能，包括边缘情况。
    *   编辑不同的字段，测试不同类型的输入。
    *   保存您的更改，并仔细验证输出的 JSON 格式是否正确，并且包含《Cataclysm: Dark Days Ahead》预期的数据。
    *   检查应用程序其他部分是否有任何意外的副作用。
    *   将修改后的 JSON 加载回应用程序中进行测试，以确保其能够正确解析。

5.  **代码风格:**
    *   尽量遵循现有的代码风格（命名约定、格式、注释）以保持一致性。虽然本项目没有提供正式的风格指南，但观察现有代码库中的模式是最佳方法。

6.  **提交您的贡献:** (项目的贡献机制未明确定义，但假设是标准的 GitHub 工作流程):
    *   在 GitHub 上 Fork 仓库。
    *   在您的 Fork 中为您的功能或错误修复创建一个新分支（例如 `feature/add-new-item-type` 或 `fix/resolve-recipe-bug`）。
    *   使用清晰且描述性的提交消息提交您的更改。理想情况下，每个提交都应代表一个逻辑工作单元。
    *   将您的分支推送到您在 GitHub 上的 Fork。
    *   从您的分支向主仓库的 `master` 或 `main` 分支发起一个 Pull Request (PR)。
    *   在 Pull Request 描述中清楚地描述您所做的更改、解决的问题以及任何特定的测试说明。

## 重要注意事项

*   **备份游戏文件:** **至关重要！** 在使用模组编辑器测试您的开发更改之前，请务必备份您的《Cataclysm: Dark Days Ahead》的 `data/json` 或 `raw` 文件夹。格式不正确或结构错误的 JSON 可能会导致游戏出现问题，包括崩溃或数据损坏。
*   **错误处理:** 实现稳健的错误处理，尤其是在文件 I/O 操作（读/写文件）和 JSON 解析/序列化方面。旨在防止应用程序崩溃并向用户提供信息丰富的错误消息。
*   **用户反馈:** 确保 UI 向用户提供良好的反馈。例如，指示操作何时正在进行、何时成功完成或发生错误。

通过遵循这些指南，您可以有效地为 CataclysmModder 项目做出贡献。感谢您的关注！
