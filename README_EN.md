# rebuild_atlas User Guide

## 📚 Documentation
- **[中文文档](README.md)** - 完整的中文使用文档

## Overview

`rebuild_atlas` is a tool for rebuilding atlases from modified sequence frame images. It supports batch processing of multiple animation folders and can be configured to process only specified animations.

## Directory Structure

Before using `rebuild_atlas`, ensure the directory structure is correct:

```
Skins/
├── rebuild_atlas.exe          # Executable file
├── config.cfg                  # Configuration file
├── Animations/                 # Animation data directory
│   ├── Atlases/                # Atlas directory
│   │   └── Original/          # Contains original atlas files
│   ├── Data/                   # Data directory (contains atlas metadata)
│   │   ├── atlas_data_index.json  # Atlas index file
│   │   ├── Knight.json        # GameObject data file (example)
│   │   └── [other GameObject].json
│   ├── Knight/                 # GameObject folder (example)
│   │   ├── Abyss Kneel/       # Animation folder (example)
│   │   │   └── [sequence frame images]
│   │   └── [other animation folders]
│   └── [other GameObject folders]
└── Export/                     # Output directory (auto-generated)
    └── [rebuilt atlas files]
```

### Directory Description

- **`Animations/Atlases/Original/`**: Contains original atlas files. These atlas files define the position and size information of sequence frames.

- **`Animations/Data/`**: Data directory containing atlas metadata:
  - **`atlas_data_index.json`**: Atlas index file containing mappings between all GameObjects and animations
  - **`[GameObjectName].json`**: Data file for each GameObject, containing all animation and atlas information for that GameObject

- **`Animations/[GameObjectName]/[AnimationName]/`**: Animation folder structure containing exported sequence frame images. Sequence frame images have the following characteristics:
  - All images have consistent sizes
  - Each image is wrapped by a red bounding box (if untrimmed information exists)
  - **Only content within the bounding box can be modified** - do not modify the bounding box itself or image dimensions

- **`Export/`**: Output directory where rebuilt atlas files will be saved. If a folder path ends with ' Data', the suffix will be automatically removed.

## Usage Workflow

### 1. Prepare Data

Ensure the following files are ready:
- Original atlas files in `Animations/Atlases/Original/`
- Data files in `Animations/Data/` directory (`atlas_data_index.json` and GameObject JSON files)
- Sequence frame images in `Animations/[GameObjectName]/[AnimationName]/` folders

### 2. Modify Sequence Frame Images

In the animation folders under `Animations/`, find the sequence frame images that need to be modified:
- You can modify the content **inside** the red bounding box
- **Do not** modify the red bounding box itself
- **Do not** change the image dimensions

### 3. Configure Processing Scope (Optional)

If you need to process only specific animation folders, edit the `config.cfg` file (see configuration details below).

### 4. Run the Tool

Double-click `rebuild_atlas.exe` in the `Skins` folder, or run it from the command line:

```bash
./rebuild_atlas.exe [config_file_path]
```

- If no configuration file path is specified, the default `config.cfg` will be used
- If a configuration file path is specified, the specified configuration file will be used

### 5. Wait for Processing to Complete

The tool will display processing progress, including:
- Number of atlases and sprites loaded
- Animation folders being processed
- Number of successful/failed operations

### 6. View Results

After processing is complete, the rebuilt atlas files will be saved in the `Export/` directory.

## Slice Function Detailed Guide

The Slice function is an important feature of the `rebuild_atlas` tool, used to extract each sprite from atlas files and save them as separate PNG files. This is very useful for scenarios where you need to use a specific sprite individually or perform further editing.

### Slice Function Workflow

1. **Read Atlas Files**: Read specified atlas files from the `Animations/Atlases/Original/` directory
2. **Extract Sprites**: Extract each sprite from the atlas according to data files in the `Animations/Data/` directory
3. **Process Transformations**: Automatically handle sprite rotation, cropping, and other transformations
4. **Save Files**: Save extracted sprites as independent PNG files to the `Sliced_Frames` directory

### Output File Description

- **Output Directory**: `Sliced_Frames/` (located under the application directory)
- **File Naming Format**: `[GameObjectName]_[AnimationName]_[FrameIndex].png`
  - For example: `Knight_Abyss Kneel_0.png`, `Knight_Abyss Kneel_1.png`, etc.
- **File Content**: Each file contains a complete sprite that has been processed for rotation and cropping

### How to Find Available Animation Names

When configuring the `animation_folders` for the Slice function, you need to know which GameObjects and animations are available. There are two methods to find them:

#### Method 1: Find from `atlas_data_index.json` (Recommended)

The `atlas_data_index.json` file contains complete mappings between all GameObjects and their animations. This is the most convenient method to find available animations.

**Steps:**
1. Open the `Animations/Data/atlas_data_index.json` file
2. Find the `objectAnimations` object
3. The structure of this object is as follows:
   ```json
   {
     "objectAnimations": {
       "Knight": ["Abyss Kneel", "Idle", "Walk", "Run"],
       "Knight Hatchling Anim": ["Hatch", "Emerge", "Idle"],
       "Another GameObject": ["Animation1", "Animation2"]
     }
   }
   ```
4. From this object, you can find:
   - **GameObject names**: Such as `"Knight"`, `"Knight Hatchling Anim"`, etc.
   - **Animation lists for each GameObject**: Such as `["Abyss Kneel", "Idle", "Walk"]`, etc.

**Configuration Example:**
```ini
[Slice]
enabled = true
# Process all animations for a GameObject
animation_folders = Knight

# Or process specific animations
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk
```

#### Method 2: Find from GameObject JSON Files

Each GameObject has a corresponding JSON file that contains all animation information for that GameObject.

**Steps:**
1. Open the `Animations/Data/[GameObjectName].json` file (e.g., `Knight.json`)
2. Find the `animations` object
3. The structure of this object is as follows:
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
4. All keys in the `animations` object are the animation names for that GameObject

**Configuration Example:**
```ini
[Slice]
enabled = true
# Process all animations for this GameObject
animation_folders = Knight

# Or process specific animations
animation_folders = Knight/Abyss Kneel
    Knight/Idle
```

### Slice Function Configuration Examples

```ini
# Example 1: Slice all animations for a GameObject
[Slice]
enabled = true
animation_folders = Knight

# Example 2: Slice specific animations
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# Example 3: Slice specific animations from multiple GameObjects
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight Hatchling Anim/Hatch
    Another GameObject/Animation Name
```

### Notes

- The Slice function **must** specify `animation_folders`, otherwise it will fail
- Animation names must **exactly match** those in the data files (case-sensitive)
- If a specified GameObject or animation does not exist, that entry will be skipped
- Output files will overwrite existing files with the same name, so use with caution

## Configuration File

### Configuration File Location

The default configuration file is `config.cfg`, located in the same directory as `rebuild_atlas.exe`.

You can also specify a configuration file via command line parameter:

```bash
./rebuild_atlas.exe /path/to/custom_config.cfg
```

### Configuration Format

The configuration file uses INI format and supports the following sections:

#### [RebuildAtlas] - Atlas Rebuild Configuration

```ini
[RebuildAtlas]
# Whether to enable atlas rebuild
enabled = true

# List of animation folders to process
# Supports multi-line format (one folder per line) or comma-separated format
animation_folders = Knight
    Knight Hatchling Anim
    Another Folder
```

**Configuration Items:**

- `enabled`: Whether to enable atlas rebuild functionality
  - `true`: Enable (default)
  - `false`: Disable

- `animation_folders`: List of animation folders to process (**required**)
  - **Note**: If empty or commented out, the rebuild atlas function will fail and return an error
  - Supports two formats:
    - **Multi-line format** (recommended): One folder name per line
    - **Comma-separated format** (backward compatible): Multiple folder names separated by commas
  - Supports two specification methods:
    - **`GameObjectName`**: Will process all animations for that GameObject
    - **`GameObjectName/AnimationName`**: Will process only the specific animation

**Examples:**

```ini
# Method 1: Multi-line format (recommended)
# Specify GameObject, will process all animations for that GameObject
animation_folders = Knight
    Knight Hatchling Anim

# Method 2: Specify specific animations
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# Method 3: Mixed usage
animation_folders = Knight
    Knight/Abyss Kneel
    Knight Hatchling Anim

# Method 4: Comma-separated format
animation_folders = Knight, Knight/Abyss Kneel, Knight Hatchling Anim
```

#### [DuplicateOperation] - Duplicate Operation Configuration

```ini
[DuplicateOperation]
# Whether to enable duplicate operation
enabled = false

# Operation mode
# true: Original image -> Dup image
# false: Dup image -> Original image
original_to_dup = true

# List of animation folders to operate on
# Supports specifying GameObjectName or GameObjectName/AnimationName
animation_folders = Knight
    Knight/Abyss Kneel
```

**Configuration Items:**

- `enabled`: Whether to enable duplicate operation
  - `true`: Enable
  - `false`: Disable (default)

- `original_to_dup`: Operation mode
  - `true`: Copy original image to Duplicate image location
  - `false`: Copy Duplicate image to original image location

- `animation_folders`: List of animation folders to operate on (**required**)
  - **Note**: If empty or commented out, the duplicate operation will fail and return an error
  - Supports two formats:
    - **Multi-line format** (recommended): One folder name per line
    - **Comma-separated format** (backward compatible): Multiple folder names separated by commas
  - Supports two specification methods:
    - **`GameObjectName`**: Will process all animations for that GameObject
    - **`GameObjectName/AnimationName`**: Will process only the specific animation

#### [Slice] - Slice Configuration

```ini
[Slice]
# Whether to enable atlas slicing
enabled = false

# List of animation folders to slice
# Can specify GameObjectName (will process all animations for that GameObject) or GameObjectName/AnimationName (process specific animation)
animation_folders = Knight
    Knight/Abyss Kneel
```

**Configuration Items:**

- `enabled`: Whether to enable atlas slicing functionality
  - `true`: Enable
  - `false`: Disable (default)

- `animation_folders`: List of animation folders to slice (**required**)
  - **Note**: If empty or commented out, the slice function will fail and return an error
  - Supports two formats:
    - **Multi-line format** (recommended): One folder name per line
    - **Comma-separated format** (backward compatible): Multiple folder names separated by commas
  - Supports two specification methods:
    - **`GameObjectName`**: Will process all animations for that GameObject
    - **`GameObjectName/AnimationName`**: Will process only the specific animation

**Functionality:**
- **Slice function** extracts each sprite from atlas files and saves them as separate PNG files
- Output directory is `Sliced_Frames` folder under the application directory
- Supports processing rotated sprites and duplicate sprites
- Each sprite is extracted as an independent PNG file with the naming format: `[GameObjectName]_[AnimationName]_[FrameIndex].png`

**How to Find Available Animation Names:**

When configuring `animation_folders`, you can find available GameObject and animation names from the following files:

1. **From `atlas_data_index.json` (Recommended)**:
   - Open the `Animations/Data/atlas_data_index.json` file
   - This file contains mappings between all GameObjects and their animations
   - File structure example:
     ```json
     {
       "objectAnimations": {
         "Knight": ["Abyss Kneel", "Idle", "Walk", "Run"],
         "Knight Hatchling Anim": ["Hatch", "Emerge", "Idle"]
       }
     }
     ```
   - From the `objectAnimations` object, you can find:
     - **GameObject names** (e.g., `"Knight"`, `"Knight Hatchling Anim"`)
     - **Animation lists for each GameObject** (e.g., `["Abyss Kneel", "Idle", "Walk"]`)
   - Configuration example:
     ```ini
     # Process all animations for a GameObject
     animation_folders = Knight
     
     # Or process specific animations
     animation_folders = Knight/Abyss Kneel
         Knight/Idle
     ```

2. **From GameObject JSON Files**:
   - Open the `Animations/Data/[GameObjectName].json` file (e.g., `Knight.json`)
   - This file contains all animation and atlas information for that GameObject
   - File structure example:
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
   - From the keys of the `animations` object, you can find all animation names for that GameObject
   - Configuration example:
     ```ini
     # Process all animations for this GameObject
     animation_folders = Knight
     
     # Or process specific animations
     animation_folders = Knight/Abyss Kneel
         Knight/Idle
     ```

**Configuration Examples:**

```ini
# Example 1: Slice all animations for a GameObject
[Slice]
enabled = true
animation_folders = Knight

# Example 2: Slice specific animations
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight/Idle
    Knight/Walk

# Example 3: Slice specific animations from multiple GameObjects
[Slice]
enabled = true
animation_folders = Knight/Abyss Kneel
    Knight Hatchling Anim/Hatch
    Another GameObject/Animation Name
```

## How to Configure Export of Atlases for Specific Animations Only

### Step 1: Open Configuration File

Open the `config.cfg` file with a text editor.

### Step 2: Configure Animation Folder List

In the `[RebuildAtlas]` section, uncomment `animation_folders` and add the names of animation folders to process:

```ini
[RebuildAtlas]
enabled = true
# Uncomment and add animation folders to process
animation_folders = Knight
    Knight Hatchling Anim
    Another Folder
```

### Step 3: Save Configuration File

Save the `config.cfg` file.

### Step 4: Run the Tool

Run `rebuild_atlas.exe`, and the tool will only process the animation folders specified in the configuration file.

### Notes

- **Important**: All three functions (RebuildAtlas, DuplicateOperation, Slice) **must** specify `animation_folders`, otherwise they will fail and return an error
- Animation folder names must **exactly match** the folder names under the `Animations/` directory (case-sensitive)
- Supports specifying multiple folders using multi-line format or comma-separated format
- Supports specifying `GameObjectName` or `GameObjectName/AnimationName` format
- If the specified `GameObjectName` does not exist or has no animations, the expanded list will be empty and the function will fail

## Frequently Asked Questions

### Q: How do I know which animation folders are available?

A: There are two methods to find available animation folders:

1. **Check the `atlas_data_index.json` file** (Recommended):
   - Open `Animations/Data/atlas_data_index.json`
   - Check the `objectAnimations` object, which contains all GameObject names and their corresponding animation lists
   - For example: `"Knight": ["Abyss Kneel", "Idle", "Walk"]` means GameObject `Knight` has three animations

2. **Check GameObject JSON files**:
   - Open `Animations/Data/[GameObjectName].json` (e.g., `Knight.json`)
   - Check the `animations` object, where the keys are all animation names for that GameObject

3. **Check the file system**:
   - Check subfolders under the `Animations/` directory, where each subfolder (except `Atlases` and `Data`) is a GameObject folder
   - Enter a GameObject folder, and the subfolders within are animation folders

### Q: What should I do if processing fails?

A: Check the following:
- Does the `Animations/Data/atlas_data_index.json` file exist and is it in the correct format?
- Do the `Animations/Data/[GameObjectName].json` files exist and are they in the correct format?
- Do the sequence frame images exist and are they readable?
- Do the original atlas files exist (from the path specified in `atlasPath`)?
- Do the animation folder names exactly match those specified in the configuration file?
- Do the GameObject folders actually exist?

### Q: Can I process all animations at once?

A: No. All three functions must explicitly specify the animation folders to process. If you need to process multiple animations, list all the animations you want to process in `animation_folders`, or use `GameObjectName` to specify all animations for an entire GameObject.

### Q: Where are the output files for the Slice function?

A: The Slice function saves sliced sprites to the `Sliced_Frames` folder under the application directory. Files are named in the format: `[GameObjectName]_[AnimationName]_[FrameIndex].png`. For example: `Knight_Abyss Kneel_0.png`, `Knight_Abyss Kneel_1.png`, etc.

### Q: How do I know what animations a GameObject has?

A: Open the `Animations/Data/atlas_data_index.json` file, find the corresponding GameObject name, and its value is the list of all animations for that GameObject. Alternatively, open the `Animations/Data/[GameObjectName].json` file and check all keys in the `animations` object.

### Q: Does the configuration file support comments?

A: Yes. Lines starting with `#` are treated as comments.

## Technical Notes

- The tool uses metadata from files in `Animations/Data/` directory to determine atlas structure
- Sequence frame images must maintain original dimensions - only content within the red bounding box can be modified
- The tool automatically handles image rotation, cropping, and other transformations
- Supports cross-platform operation (Windows, macOS)
- When rebuilding, if a folder path ends with ' Data', the suffix will be automatically removed
- Supports dynamic atlas loading with a maximum cache of 10 atlases, with older ones saved to temporary disk cache

## Version Information

Current version supports:
- Atlas rebuild (supports filtering by GameObject or animation)
- Automatic filtering of animations associated with existing GameObject folders
- Duplicate image operations
- Atlas slicing (Slice) functionality
- Command-line configuration file specification
- Relative path display
- Dynamic atlas loading and cache management
- Automatic removal of ' Data' suffix from save paths


