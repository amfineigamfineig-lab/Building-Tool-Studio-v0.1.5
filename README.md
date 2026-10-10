# Building Tool

A Windows editor for building places, editing scripts, and running a Roblox-inspired Luau runtime with simple physics.

Keep this folder together: the executable expects `assets/`, icons, configs, and helpers next to it.

---

## Requirements

- **Windows** (Win32)
- No separate installer required for basic use

---

## Quick start

1. Open this folder.
2. Double-click **`buildingTool.exe`**.
3. If Windows shows a security prompt, choose **Run** only if you trust the source.
4. On the welcome screen:
   - **New Project** — choose a single `.hlx` file or a multi-file project folder
   - **Open Project** — load an existing `.hlx` file or `.btproj` manifest
   - **Recent Projects** — reopen a recent path

**Do not move `buildingTool.exe` out of this folder** unless you know it does not depend on the files beside it.

---

## Project files

- **Single-file projects** use the compressed **`.hlx`** format and embed scripts and assets.
- **Project folders** contain a compressed **`.btproj`** manifest plus separate `Scripts\` and `Assets\` files. The selected manifest is stored inside the project folder.
- Use **File → Save / Save As** (or the top bar equivalents) to write a project.
- Autosave may write a backup under the working directory while you edit.
- Use **File → Export** to create a standalone Windows runtime executable. Multi-file project exports include a `*_Export\` folder with the executable and asset copies; scripts are compiled to Luau bytecode in the runtime.

The exported executable starts directly in the game runtime without the editor interface. It embeds the project and required shaders; a per-process shader cache is extracted to the system temporary directory and removed when the game exits.

### File association (optional)

To open `.hlx` and `.btproj` files by double-click for the **current Windows user**:

1. Keep the application folder intact.
2. First launch of `buildingTool.exe` can register the association automatically for that user.
3. To repair or re-register manually, run:

   `install_hlx_file_association.bat`

This maps `.hlx` to `buildingTool.exe` and uses `assets\icons\File.ico`. Administrator rights are not required.

---

## Main UI

| Area | Role |
|------|------|
| **Viewport** | 3D view of Workspace; gizmo move / rotate / scale |
| **Explorer** | Instance tree (services, models, scripts, …) |
| **Properties** | Selected instance properties (including Enum dropdowns) |
| **Output** | `print` / `warn` / runtime errors |
| **Script editor** | Edit Script / LocalScript / ModuleScript source |
| **Asset Manager** | Import and browse project assets |

Explorer starts **collapsed** when a project is loaded or created. Scrollbars scale with content length.

### Themes

Choose a theme in **Studio Settings → Appearance**; the selected theme is saved and applied to the editor and icon tints. Custom theme JSON files load from `themes/`. Icon color overrides can be grouped under `icons.set`, per-instance under `icons.single` (or the older `icons.instances`), and toolbar/ribbon colors under `ribbons` or `icons.ribbons`. `icons.misc` remains available for shared non-instance icons.

### Action bar (under the top menu)

| Control | Action |
|---------|--------|
| **Play** | Start runtime in **Server** mode |
| **Play Player** | Start runtime in **Client** mode |
| **Stop** | Stop runtime and restore play-time transforms where applicable |
| **SV / CL** | Toggle preferred Server / Client POV; if already playing, restarts in the other mode |

---

## Instances & hierarchy

Common classes: **Part**, **Model**, **Folder**, **Script** / **LocalScript** / **ModuleScript**, **Sound**, **Texture**, **WeldConstraint**, **Motor6D**, **BindableEvent** / **RemoteEvent**, **Humanoid**, **Camera**, lighting effects, and fixed **Services** under `game`.

### 2D GUI

- **ScreenGui**, **SurfaceGui**, **BillboardGui**
- **Frame**, **ScrollingFrame**, **TextLabel**, **ImageLabel**, **TextButton**, **ImageButton**, **TextBox**
- Modifiers: **UICorner**, **UIListLayout**, **UIStroke**, **UIPadding**, **UIGridLayout**, **UIAspectRatioConstraint**

Author UI under **`game.StarterGui`**. On **client** Play, children are **cloned per player** into `LocalPlayer.PlayerGui` (separate from the shared Starter tree). ScreenGui draws as a viewport overlay.

Also: **StarterPlayer**, **StarterPack** services. Gui types appear in Insert Object (Ctrl+I).

- **Motor6D** is stored as a child of **Part1** when Part1 is assigned.
- Insert instances from the editor insert menu / hierarchy tools.
- Drag in Explorer to reparent (where allowed).

---

## Scripts

### RunContext (Properties)

| RunContext | Class | Runs when |
|------------|--------|-----------|
| **Server** | Script | Server or Client play |
| **Client** | LocalScript | Client play only |
| **Module** | ModuleScript | Only via `require(...)` |

Changing **RunContext** updates the instance type and Explorer icon.

### Enabled

If **Enabled** is off, the script is **not** started at Play.

### Editor features

- Syntax-oriented editing with completions from `assets/luau_config.json`
- Member completion (`.` / `:`) lists real children and class members only — not free user words
- `script` global points at the running script instance
- Modules can use `module.__index = module` for normal `self` method calls

Full API: see **`Luau_API_lib.md`**.

---

## Play runtime

When Play starts, the host injects a Luau prelude (`buildLuauDefinitions`) and runs eligible scripts as coroutines.

- **RunService** — `Heartbeat`, `Stepped`, `RenderStepped`; `IsServer()`, `IsClient()`, `IsStudio()`, `IsRunning()`
- **task** — `wait`, `spawn`, `defer`, `delay`
- **Instance.new**, services via `game:GetService`, events (Bindable / Remote prototypes)
- **TweenService** driven by Heartbeat
- Unanchored Parts get **physics** (gravity / collision) for the session

Client mode can create a LocalPlayer character path for `pointer` / character tests. Remotes are **same-process** prototypes for testing, not real multiplayer.

Stop ends the VM session, clears runtime-created instances, and restores snapshotted transforms.

---

## Output window

- Shows `print`, `warn`, and runtime errors
- **Auto-scrolls** to new messages while you are at the bottom; scrolling up pauses follow; returning to the bottom resumes it
- Keeps at most **300** messages (oldest dropped)
- Mouse wheel works over the **entire** Output panel
- Ctrl+A / Ctrl+C with line selection for copy

---

## Properties panel

- Booleans toggle with a checkbox
- Colors open a picker
- Instance references can use pointer-pick where supported
- **Enum** properties open a dropdown of defined options (e.g. PartType, Face, RunContext, Camera Mode) — not free text
- Point/spot/surface light Brightness and Range are limited to 0–200; spot and surface Angle is limited to 0–180 degrees.

---

## Assets

- Import media and mesh files via **File → Import Asset**. The file picker supports
  multi-select; each imported mesh creates a MeshPart in Workspace with MeshId,
  TextureId, and CollisionFidelity properties.
- Add an empty **MeshPart** from **Insert Object** or create one from Luau with
  `Instance.new("MeshPart")`; until MeshId is assigned, it uses the default block shape.
- Mesh imports support OBJ, GLB, IQM, VOX, and M3D files. Mesh data is embedded
  in the project file, uses the part lighting shader, and TextureId can reference
  an imported image asset.
- Image imports support PNG, JPEG, BMP, TGA, GIF, PSD, HDR, PIC, and PNM files.
- Sound and image previews depend on supported formats and `assets/` layout.
- `assets/luau_config.json` drives editor theme keywords and completion word lists
- `assets/Headers/physic.h` is used by the physics step when building from source

---

## Building from source (developers)

Primary source is the single translation unit:

- `buildingTool_.cpp`

Dependencies (paths as used in the source includes):

- **raylib** (and raymath / rlgl)
- **Luau** VM + Compiler headers/libs
- Windows SDK (dialogs, etc.)

Build with your usual MSVC / toolchain setup matching the include paths in the file. Output should remain next to `assets/` for runtime resource loading.

Luau API surface for the embedded runtime is documented in **`Luau_API_lib.md`**.

---

## Folder layout (typical)

```
BuildingTool_Pending/
  buildingTool.exe          # run this
  buildingTool_.cpp         # source
  READ_ME.md                # this file
  Luau_API_lib.md           # Luau runtime API
  install_hlx_file_association.bat
  assets/
    icons/
    Headers/                # e.g. physic.h, Compress.h
    luau_config.json
    ...
  shaders/
```

---

## Troubleshooting

| Issue | What to try |
|-------|-------------|
| Double-click does nothing | Run `buildingTool.exe` explicitly; check antivirus quarantine |
| Missing icons / blank UI | Keep `assets/` beside the exe; do not run a lone copied exe |
| `.hlx` does not open on double-click | Run `install_hlx_file_association.bat` once for that Windows user |
| Scripts error on `RunService:IsClient()` | Call with parentheses; use `game:GetService("RunService")` |
| Script does not run | Check **Enabled**, **RunContext**, and Server vs Client play mode |
| Output flooded | Cap is 300 lines; scroll up to pause auto-follow |

---

## Related docs

- **`Luau_API_lib.md`** — complete Luau globals, Instance API, services, events, examples
- Dummy Studio test scripts (if present): `DummyScripts_StudioTest/`

---

*Building Tool is an independent editor runtime inspired by Roblox/Luau patterns. It is not affiliated with Roblox Corporation.*
