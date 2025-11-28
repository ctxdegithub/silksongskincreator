# rebuild_atlas 使用文档

## 📚 文档
- **[English Documentation](README_EN.md)** - Complete user guide in English

## 概述

`rebuild_atlas` 是一个用于重构图集的工具，它可以从修改后的序列帧图片重新生成图集文件。该工具支持批量处理多个动画文件夹，并可以配置只处理指定的动画。

## 目录结构

在使用 `rebuild_atlas` 之前，需要确保目录结构正确：

```
Skins/
├── rebuild_atlas.exe          # 可执行文件
├── config.cfg                  # 配置文件
├── Animations/                 # 动画数据目录
│   ├── Atlases/                # 图集目录
│   │   └── Original/          # 包含原始图集文件
│   ├── Data/                   # 数据目录（包含图集元数据）
│   │   ├── atlas_data_index.json  # 图集索引文件
│   │   ├── Knight.json        # GameObject 数据文件（示例）
│   │   └── [其他 GameObject].json
│   ├── Knight/                 # GameObject 文件夹（示例）
│   │   ├── Abyss Kneel/       # 动画文件夹（示例）
│   │   │   └── [序列帧图片]
│   │   └── [其他动画文件夹]
│   └── [其他 GameObject 文件夹]
└── Export/                     # 输出目录（自动生成）
    └── [重构后的图集文件]
```

### 目录说明

- **`Animations/Atlases/Original/`**: 包含原始图集文件。这些图集文件定义了序列帧的位置和大小信息。

- **`Animations/Data/`**: 数据目录，包含图集元数据：
  - **`atlas_data_index.json`**: 图集索引文件，包含所有 GameObject 和动画的映射关系
  - **`[GameObjectName].json`**: 每个 GameObject 的数据文件，包含该 GameObject 的所有动画和图集信息

- **`Animations/[GameObjectName]/[AnimationName]/`**: 动画文件夹结构，包含导出的序列帧图片。序列帧图片具有以下特点：
  - 所有图片大小一致
  - 每个图片由红色包围盒包裹（如果有 untrimmed 信息）
  - **只能修改包围盒内部的内容**，不要修改包围盒本身或图片尺寸

- **`Export/`**: 输出目录，重构后的图集文件将保存在此目录中。如果保存路径的文件夹以 ' Data' 结尾，会自动去掉该后缀。

## 使用流程

### 1. 准备数据

确保以下文件已准备好：
- `Animations/Atlases/Original/` 中的原始图集文件
- `Animations/Data/` 目录下的数据文件（`atlas_data_index.json` 和各 GameObject 的 JSON 文件）
- `Animations/[GameObjectName]/[AnimationName]/` 下各动画文件夹中的序列帧图片

### 2. 修改序列帧图片

在 `Animations/` 下的各个动画文件夹中，找到需要修改的序列帧图片：
- 可以修改红色包围盒**内部**的内容
- **不要**修改红色包围盒本身
- **不要**改变图片的尺寸

### 3. 配置处理范围（可选）

如果需要只处理特定的动画文件夹，请编辑 `config.cfg` 文件（详见下方配置说明）。

### 4. 运行工具

在 `Skins` 文件夹中双击运行 `rebuild_atlas.exe`，或通过命令行运行：

```bash
./rebuild_atlas.exe [config_file_path]
```

- 如果不指定配置文件路径，将使用默认的 `config.cfg`
- 如果指定了配置文件路径，将使用指定的配置文件

### 5. 等待处理完成

工具会显示处理进度，包括：
- 加载的图集和精灵数量
- 正在处理的动画文件夹
- 处理成功/失败的数量

### 6. 查看结果

处理完成后，重构后的图集文件将保存在 `Export/` 目录中。

## Slice 功能详细说明

Slice 功能是 `rebuild_atlas` 工具的一个重要功能，用于从图集文件中提取每个 sprite 并保存为单独的 PNG 文件。这对于需要单独使用某个 sprite 或进行进一步编辑的场景非常有用。

### Slice 功能工作流程

1. **读取图集文件**：从 `Animations/Atlases/Original/` 目录中读取指定的图集文件
2. **提取 Sprite**：根据 `Animations/Data/` 目录中的数据文件，从图集中提取每个 sprite
3. **处理变换**：自动处理 sprite 的旋转、裁剪等变换
4. **保存文件**：将提取的 sprite 保存为独立的 PNG 文件到 `Sliced_Frames` 目录

### 输出文件说明

- **输出目录**：`Sliced_Frames/`（位于应用程序目录下）
- **文件命名格式**：`[GameObjectName]_[AnimationName]_[FrameIndex].png`
  - 例如：`Knight_Abyss Kneel_0.png`、`Knight_Abyss Kneel_1.png` 等
- **文件内容**：每个文件包含一个完整的 sprite，已经过旋转和裁剪处理

### 如何查找可用的动画名称

在配置 Slice 功能的 `animation_folders` 时，您需要知道哪些 GameObject 和动画是可用的。有两种方法可以查找：

#### 方法 1：从 `atlas_data_index.json` 查找（推荐）

`atlas_data_index.json` 文件包含了所有 GameObject 及其动画的完整映射关系，这是查找可用动画最方便的方法。

**步骤：**
1. 打开 `Animations/Data/atlas_data_index.json` 文件
2. 查找 `objectAnimations` 对象
3. 该对象的结构如下：
   ```json
   {
     "objectAnimations": {
       "Knight": ["Abyss Kneel", "Idle", "Walk", "Run"],
       "Knight Hatchling Anim": ["Hatch", "Emerge", "Idle"],
       "Another GameObject": ["Animation1", "Animation2"]
     }
   }
   ```
4. 从该对象中可以找到：
   - **GameObject 名称**：如 `"Knight"`、`"Knight Hatchling Anim"` 等
   - **每个 GameObject 的动画列表**：如 `["Abyss Kneel", "Idle", "Walk"]` 等

**配置示例：**
```ini
[Slice]
enabled = true
# 处理整个 GameObject 的所有动画
animation_folders = Knight

# 或处理特定动画
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk
```

#### 方法 2：从 GameObject JSON 文件查找

每个 GameObject 都有一个对应的 JSON 文件，其中包含了该 GameObject 的所有动画信息。

**步骤：**
1. 打开 `Animations/Data/[GameObjectName].json` 文件（例如 `Knight.json`）
2. 查找 `animations` 对象
3. 该对象的结构如下：
   ```json
   {
     "animations": {
       "Abyss Kneel": {
         "sprites": [ ... ]
       },
       "Idle": {
         "sprites": [ ... ]
       },
       "Walk": {
         "sprites": [ ... ]
       }
     },
     "atlases": [ ... ]
   }
   ```
4. `animations` 对象的所有 key 就是该 GameObject 的所有动画名称

**配置示例：**
```ini
[Slice]
enabled = true
# 处理该 GameObject 的所有动画
animation_folders = Knight

# 或处理特定动画
animation_folders = Knight/Abyss Kneel
    Knight/Idle
```

### Slice 功能配置示例

```ini
# 示例 1：切割整个 GameObject 的所有动画
[Slice]
enabled = true
animation_folders = Knight

# 示例 2：切割特定动画
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# 示例 3：切割多个 GameObject 的特定动画
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight Hatchling Anim/Hatch
    Another GameObject/Animation Name
```

### 注意事项

- Slice 功能**必须**指定 `animation_folders`，否则将失败
- 动画名称必须与数据文件中的名称**完全匹配**（区分大小写）
- 如果指定的 GameObject 或动画不存在，该条目将被跳过
- 输出文件会覆盖同名的现有文件，请谨慎操作

## 配置文件说明

### 配置文件位置

默认配置文件为 `config.cfg`，位于 `rebuild_atlas.exe` 同目录下。

也可以通过命令行参数指定配置文件：

```bash
./rebuild_atlas.exe /path/to/custom_config.cfg
```

### 配置格式

配置文件使用 INI 格式，支持以下配置节：

#### [RebuildAtlas] - 图集重构配置

```ini
[RebuildAtlas]
# 是否启用图集重构
enabled = true

# 要处理的动画文件夹列表
# 支持多行格式（每行一个文件夹）或逗号分隔格式
animation_folders = Knight
    Knight Hatchling Anim
    Another Folder
```

**配置项说明：**

- `enabled`: 是否启用图集重构功能
  - `true`: 启用（默认）
  - `false`: 禁用

- `animation_folders`: 要处理的动画文件夹列表（**必须指定**）
  - **注意**: 如果为空或注释掉，重建图集功能将失败并返回错误
  - 支持两种格式：
    - **多行格式**（推荐）：每行一个文件夹名称
    - **逗号分隔格式**（向后兼容）：使用逗号分隔多个文件夹名称
  - 支持两种指定方式：
    - **`GameObjectName`**: 会处理该 GameObject 的所有动画
    - **`GameObjectName/AnimationName`**: 只处理特定动画

**示例：**

```ini
# 方式1：多行格式（推荐）
# 指定 GameObject，会处理该 GameObject 的所有动画
animation_folders = Knight
    Knight Hatchling Anim

# 方式2：指定特定动画
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# 方式3：混合使用
animation_folders = Knight
    Knight/Abyss Kneel
    Knight Hatchling Anim

# 方式4：逗号分隔格式
animation_folders = Knight, Knight/Abyss Kneel, Knight Hatchling Anim
```

#### [DuplicateOperation] - Duplicate 操作配置

```ini
[DuplicateOperation]
# 是否启用 duplicate 操作
enabled = false

# 操作模式
# true: 原始图片 -> Dup图片
# false: Dup图片 -> 原始图片
original_to_dup = true

# 要操作的动画文件夹列表
# 支持指定 GameObjectName 或 GameObjectName/AnimationName
animation_folders = Knight
    Knight/Abyss Kneel
```

**配置项说明：**

- `enabled`: 是否启用 duplicate 操作
  - `true`: 启用
  - `false`: 禁用（默认）

- `original_to_dup`: 操作模式
  - `true`: 将原始图片复制到 Duplicate 图片位置
  - `false`: 将 Duplicate 图片复制到原始图片位置

- `animation_folders`: 要操作的动画文件夹列表（**必须指定**）
  - **注意**: 如果为空或注释掉，Duplicate 操作将失败并返回错误
  - 支持两种格式：
    - **多行格式**（推荐）：每行一个文件夹名称
    - **逗号分隔格式**（向后兼容）：使用逗号分隔多个文件夹名称
  - 支持两种指定方式：
    - **`GameObjectName`**: 会处理该 GameObject 的所有动画
    - **`GameObjectName/AnimationName`**: 只处理特定动画

#### [Slice] - 切割配置

```ini
[Slice]
# 是否启用图集切割
enabled = false

# 要切割的动画文件夹列表
# 可以指定 GameObjectName（会处理该 GameObject 的所有动画）或 GameObjectName/AnimationName（处理特定动画）
animation_folders = Knight
    Knight/Abyss Kneel
```

**配置项说明：**

- `enabled`: 是否启用图集切割功能
  - `true`: 启用
  - `false`: 禁用（默认）

- `animation_folders`: 要切割的动画文件夹列表（**必须指定**）
  - **注意**: 如果为空或注释掉，切割功能将失败并返回错误
  - 支持两种格式：
    - **多行格式**（推荐）：每行一个文件夹名称
    - **逗号分隔格式**（向后兼容）：使用逗号分隔多个文件夹名称
  - 支持两种指定方式：
    - **`GameObjectName`**: 会处理该 GameObject 的所有动画
    - **`GameObjectName/AnimationName`**: 只处理特定动画

**功能说明：**
- **切割功能**会从图集文件中提取每个 sprite，并保存为单独的 PNG 文件
- 输出目录为应用程序目录下的 `Sliced_Frames` 文件夹
- 支持处理旋转的 sprite 和 duplicate sprite
- 每个 sprite 会被提取为独立的 PNG 文件，文件名格式为：`[GameObjectName]_[AnimationName]_[FrameIndex].png`

**如何查找可用的动画名称：**

在配置 `animation_folders` 时，您可以从以下文件中查找可用的 GameObject 和动画名称：

1. **从 `atlas_data_index.json` 查找（推荐）**：
   - 打开 `Animations/Data/atlas_data_index.json` 文件
   - 该文件包含所有 GameObject 及其动画的映射关系
   - 文件结构示例：
     ```json
     {
       "objectAnimations": {
         "Knight": ["Abyss Kneel", "Idle", "Walk", "Run"],
         "Knight Hatchling Anim": ["Hatch", "Emerge", "Idle"]
       }
     }
     ```
   - 从 `objectAnimations` 对象中可以找到：
     - **GameObject 名称**（如 `"Knight"`、`"Knight Hatchling Anim"`）
     - **每个 GameObject 的动画列表**（如 `["Abyss Kneel", "Idle", "Walk"]`）
   - 配置示例：
     ```ini
     # 处理整个 GameObject 的所有动画
     animation_folders = Knight
     
     # 或处理特定动画
     animation_folders = Knight/Abyss Kneel
         Knight/Idle
     ```

2. **从 GameObject JSON 文件查找**：
   - 打开 `Animations/Data/[GameObjectName].json` 文件（如 `Knight.json`）
   - 该文件包含该 GameObject 的所有动画和图集信息
   - 文件结构示例：
     ```json
     {
       "animations": {
         "Abyss Kneel": { ... },
         "Idle": { ... },
         "Walk": { ... }
       },
       "atlases": [ ... ]
     }
     ```
   - 从 `animations` 对象的 key 中可以找到该 GameObject 的所有动画名称
   - 配置示例：
     ```ini
     # 处理该 GameObject 的所有动画
     animation_folders = Knight
     
     # 或处理特定动画
     animation_folders = Knight/Abyss Kneel
         Knight/Idle
     ```

**配置示例：**

```ini
# 示例 1：切割整个 GameObject 的所有动画
[Slice]
enabled = true
animation_folders = Knight

# 示例 2：切割特定动画
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# 示例 3：切割多个 GameObject 的特定动画
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight Hatchling Anim/Hatch
    Another GameObject/Animation Name
```

## 如何配置只导出指定动画关联的图集

### 步骤 1: 打开配置文件

使用文本编辑器打开 `config.cfg` 文件。

### 步骤 2: 配置动画文件夹列表

在 `[RebuildAtlas]` 节中，取消注释 `animation_folders` 并添加要处理的动画文件夹名称：

```ini
[RebuildAtlas]
enabled = true
# 取消注释并添加要处理的动画文件夹
animation_folders = Knight
    Knight Hatchling Anim
    Another Folder
```

### 步骤 3: 保存配置文件

保存 `config.cfg` 文件。

### 步骤 4: 运行工具

运行 `rebuild_atlas.exe`，工具将只处理配置文件中指定的动画文件夹。

### 注意事项

- **重要**: 三个功能（RebuildAtlas、DuplicateOperation、Slice）都必须指定 `animation_folders`，否则将失败并返回错误
- 动画文件夹名称必须与 `Animations/` 目录下的文件夹名称**完全匹配**（区分大小写）
- 支持使用多行格式或逗号分隔格式指定多个文件夹
- 支持指定 `GameObjectName` 或 `GameObjectName/AnimationName` 格式
- 如果指定的 `GameObjectName` 不存在或没有动画，展开后的列表为空，功能也会失败

## 常见问题

### Q: 如何知道哪些动画文件夹可用？

A: 有两种方法可以查找可用的动画文件夹：

1. **查看 `atlas_data_index.json` 文件**（推荐）：
   - 打开 `Animations/Data/atlas_data_index.json`
   - 查看 `objectAnimations` 对象，其中包含所有 GameObject 名称和对应的动画列表
   - 例如：`"Knight": ["Abyss Kneel", "Idle", "Walk"]` 表示 GameObject `Knight` 有三个动画

2. **查看 GameObject JSON 文件**：
   - 打开 `Animations/Data/[GameObjectName].json`（如 `Knight.json`）
   - 查看 `animations` 对象，其中的 key 就是该 GameObject 的所有动画名称

3. **查看文件系统**：
   - 查看 `Animations/` 目录下的子文件夹，每个子文件夹（除了 `Atlases` 和 `Data`）都是一个 GameObject 文件夹
   - 进入 GameObject 文件夹，其中的子文件夹就是动画文件夹

### Q: 处理失败怎么办？

A: 检查以下内容：
- `Animations/Data/atlas_data_index.json` 文件是否存在且格式正确
- `Animations/Data/[GameObjectName].json` 文件是否存在且格式正确
- 序列帧图片是否存在且可读
- 原始图集文件是否存在（从 `atlasPath` 指定的路径）
- 动画文件夹名称是否与配置文件中指定的名称完全匹配
- GameObject 文件夹是否实际存在

### Q: 可以同时处理所有动画吗？

A: 不可以。三个功能都必须明确指定要处理的动画文件夹。如果需要处理多个动画，请在 `animation_folders` 中列出所有需要处理的动画，或使用 `GameObjectName` 来指定整个 GameObject 的所有动画。

### Q: Slice 功能的输出文件在哪里？

A: Slice 功能会将切割后的 sprite 保存到应用程序目录下的 `Sliced_Frames` 文件夹中。文件命名格式为：`[GameObjectName]_[AnimationName]_[FrameIndex].png`。例如：`Knight_Abyss Kneel_0.png`、`Knight_Abyss Kneel_1.png` 等。

### Q: 如何知道某个 GameObject 有哪些动画？

A: 打开 `Animations/Data/atlas_data_index.json` 文件，查找对应的 GameObject 名称，其值就是该 GameObject 的所有动画列表。或者打开 `Animations/Data/[GameObjectName].json` 文件，查看 `animations` 对象的所有 key。

### Q: 配置文件支持注释吗？

A: 支持。使用 `#` 开头的行将被视为注释。

## 技术说明

- 工具使用 `Animations/Data/` 目录下的数据文件来确定图集结构
- 序列帧图片必须保持原始尺寸，只能修改红色包围盒内的内容
- 工具会自动处理图片的旋转、裁剪等变换
- 支持跨平台运行（Windows、macOS）
- 重建时，如果保存路径的文件夹以 ' Data' 结尾，会自动去掉该后缀
- 支持动态加载图集，最多缓存 10 个图集，超过的会保存到临时磁盘缓存

## 版本信息

当前版本支持的功能：
- 图集重构（支持按 GameObject 或动画过滤）
- 自动筛选存在的 GameObject 文件夹关联的动画
- Duplicate 图片操作
- 图集切割（Slice）功能
- 命令行配置文件指定
- 相对路径显示
- 动态图集加载和缓存管理
- 自动去除保存路径中的 ' Data' 后缀


