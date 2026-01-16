# 📘 EasySavePro Plugin Documentation

## 1. Overview
**EasySavePro** is a robust, asynchronous save/load system for Unreal Engine 5. Designed for flexibility and performance, it allows you to save any actor, handle open worlds, manage persistent player data, and capture save thumbnails without freezing your game.

---

## 2. Getting Started (3 Steps)

### Step 1: Enable the Plugin
Enable `EasySavePro` in *Edit > Plugins*.

> **Important:** Most nodes require the **EasySaveSubsystem**.  
> To get it, right-click in your Blueprint (Context Sensitive checked) and search for **"Get EasySaveSubsystem"**. Connect this return value to the "Target" pin of EasySave nodes.

### Step 2: Mark Actors to Save
To save an actor (Player, Chest, Enemy, etc.), simply add the `EasySaveComponent` to it in Blueprint or C++.
*   The component automatically adds the tag `"SaveMe"` to the actor.
*   It handles saving Transform, Physics, and variables marked as "SaveGame".

### Step 3: Implement the Interface (Optional)
To run custom logic when saving/loading (e.g. refill health, reset AI), implement the `EasySaveInterface` in your Actor Blueprint.
*   **OnSaveStarted**: Called just before saving. Use it to update variables.
*   **OnActorLoaded**: Called after this specific actor has been restored.
*   **OnGameLoaded**: Called after the *entire world* has been restored.

### 🛠️ Utility Nodes
Helper nodes to make your UI life easier (Category: EasySave | Utils).

| Node | Description |
| :--- | :--- |
| **Import Save Screenshot** | Loads the `.png` from a save slot as a `Texture2D` for your UI Image. |
| **Format Played Time** | Converts seconds (float) to `HH:MM:SS` string. |
| **Format Save Date** | Converts DateTime to `DD/MM/YYYY - HH:MM` string. |

---

## 3. Core Features

### 💾 Saving the World
Use the **Save World Async** node.
*   *Note: The synchronous 'SaveWorld' function is internal and hidden to prevent game freezes.*
*   **Slot Name**: Name of the save folder (e.g. "Slot_01").
*   **Capture Screenshot**: If true, captures the viewport (Async).
*   **Extra Metadata**: A Map (String->String) for custom data (Quest, Level, etc.).
*   **Resolution**: Optional Width/Height to resize the screenshot.

### 📂 Loading the World
*   **Load World Full**: Reloads the map and restores all actors. Use for Main Menu -> Load Game.
*   **Load World Instant**: Restores actors *without* reloading the map. Use for Checkpoints.

### ⚙️ Auto-Save System
A built-in timer system to handle rolling auto-saves.
*   **Start Auto Save**: Frequency (seconds) and Max Slots (e.g. 3).
*   **Update Auto Save Settings**: Change frequency on the fly.
*   **Stop Auto Save**: Pauses the timer.
*   Files are named `AutoSave_0`, `AutoSave_1`, etc.

### 🌍 Global Save (Settings)
Use **Save Global** and **Load Global** to save data independent of the current level (e.g. Audio Settings, Player Accessibility).
*   **How it works**: It saves the variables marked as "SaveGame" inside your **GameInstance**.
*   **Slot Name**: Defaults to "GlobalSettings" if empty.

---

## 4. Advanced Features

### 🖼️ Asynchronous Screenshots
The plugin captures and compresses screenshots in a background thread to avoid frame drops.
You can resize them (e.g. 640x360) to save disk space using the `ScreenshotWidth` / `ScreenshotHeight` pins.

### 📝 Custom Metadata
You can attach custom data (Dictionary) to any save file (e.g. "Quest": "Kill Dragon", "Money": "500").
*   **Write**: Pass the Map to the `Save World Async` node.
*   **Read**: Call `GetAllSaveSlots`. The `Structure` contains your `CustomData`. Useful for UI.

### 🐛 Debug Mode
Type `EasySave.ShowDebug 1` in the console to visualize saved actors.
*   **Green Sphere**: Actor is saved correctly (ID visible).
*   **Cyan Sphere**: **DUPLICATE ID DETECTED** (Critical warning).
*   **Red Sphere**: Invalid ID (Re-generate it in editor).
*   **Orange Sphere**: Tag present but Component missing.

---

## 5. Blueprint Node Reference

| Node Name | Category | Description |
| :--- | :--- | :--- |
| **Save World Async** | Async | Saves world, screenshot, and metadata in background. Triggers "On Success". |
| **Load World Full** | easySave | Reloads level and restores state. |
| **Load World Instant** | EasySave | Restores state immediately (Checkpoint). |
| **Save Global** | EasySave | Saves GameInstance "SaveGame" variables (Settings). |
| **Load Global** | EasySave | Loads GameInstance variables. |
| **Open Level With Data** | Travel | Opens a new level but keeps Player/Controller state active. |
| **Get All Save Slots** | File | Returns an array of `FSaveMetadata` struct: `SlotName`, `Date`, `MapName`, `PlayedTime`, `ScreenshotPath`, `CustomData`. |
| **Delete Save** | File | Permanently deletes a slot folder. |
| **Start Auto Save** | AutoSave | Activates the rolling auto-save timer. |
| **Stop Auto Save** | AutoSave | Pauses/Stops the auto-save timer. |
| **Update Auto Save Settings** | AutoSave | Changes interval/max slots on the fly. |
| **Is Auto Save Active** | AutoSave | Returns true if the timer is running. |
| **Does Save Exist** | File | Checks if a slot is valid. |

---

## 6. Best Practices

1.  **Unique IDs (Optional)**: The `EasySaveComponent` automatically generates a GUID. However, if you duplicate an actor in the level editor, it's reliable to click **"Force Generate ID"** in the details panel to ensure no conflicts occur.
2.  **Variables**: Only variables marked with the **"SaveGame"** checkbox (Advanced Display) are saved!
3.  **Level Travel**: Use `OpenLevelWithData` to keep your Inventory/Stats when changing maps (hub -> dungeon).
