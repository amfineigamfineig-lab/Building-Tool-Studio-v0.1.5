# Building Tool Luau API

> A practical API reference for the Luau runtime embedded in Building Tool.
>
> **Runtime:** Building Tool Luau  
> **Execution:** Server and client  
> **Status:** Subset implementation; not full Roblox API parity

---

## Contents

- [Introduction](#introduction)
- [Runtime Model](#runtime-model)
- [Global Environment](#global-environment)
- [Output and Debugging](#output-and-debugging)
- [Instances](#instances)
  - [Properties](#instance-properties)
  - [Methods](#instance-methods)
  - [Signals](#signals)
- [Creating Instances](#creating-instances)
- [Scripts and Modules](#scripts-and-modules)
- [Services](#services)
- [Events and Functions](#events-and-functions)
- [Datatypes](#datatypes)
- [Enums](#enums)
- [Task and Timing](#task-and-timing)
- [Standard Libraries](#standard-libraries)
- [hmath](#hmath)
- [2D GUI](#2d-gui)
- [Class Reference](#class-reference)
- [Physics](#physics)
- [Server and Client](#server-and-client)
- [Examples](#examples)
- [Unsupported APIs](#unsupported-apis)

---

# Introduction

Building Tool provides an embedded **Luau runtime** for writing scripts that interact with the project, its instances, services, and runtime systems.

The API is intentionally familiar to Roblox developers. Many objects, services, datatypes, method names, and programming patterns follow Roblox conventions so that existing Luau knowledge transfers naturally.

However, Building Tool is **not Roblox**. The runtime implements a focused subset of the Roblox programming model, with several systems simplified or simulated for the editor.

For example, an `Instance` is represented by a proxy object rather than full Roblox engine userdata, and networking APIs operate as a same-process prototype.

> **Important:** When this documentation differs from Roblox documentation, the Building Tool behavior described here takes precedence.

---

# Runtime Model

The Building Tool runtime is available when using:

- **Play** — server-oriented execution.
- **Play Player** — client-oriented execution.

Scripts execute inside the editor's embedded Luau environment.

The runtime provides:

- A Roblox-inspired instance hierarchy.
- Common Roblox-style services.
- Luau standard libraries.
- Signals and event connections.
- Simplified physics.
- 2D GUI objects.
- Server/client execution contexts.
- A custom `hmath` expression library.

The runtime is designed primarily for **building, testing, prototyping, and scripting projects inside the editor**.

---

# Global Environment

The following globals are available to scripts.

| Global | Type | Description |
|---|---|---|
| `game` | `Instance` | Root of the project hierarchy. |
| `workspace` | `Instance` | Alias for `game.Workspace`. |
| `script` | `Instance` | The script or module currently executing. |
| `pointer` | `Instance?` | The local player's character in client play; `nil` on the server. |
| `__version` | `string` | Editor and Luau runtime version. |
| `shared` | `table` | Table shared between scripts in the same runtime session. |
| `_G` | `table` | Global environment table. |

## `script`

`script` refers to the `Script`, `LocalScript`, or `ModuleScript` currently executing.

Common properties include:

```lua
script.Name
script.ClassName
script.Parent
script.Enabled
script.RunContext
```

Useful methods include:

```lua
script:GetFullName()
```

### `script.RunContext`

A script can run in one of three contexts:

| RunContext | Purpose |
|---|---|
| `"Server"` | Server execution. |
| `"Client"` | Client execution. |
| `"Module"` | ModuleScript execution through `require`. |

---

# Output and Debugging

Building Tool exposes the standard Luau output functions:

```lua
print(...)
warn(...)
assert(condition, ...)
```

### `print`

Writes a normal message to the Output window.

```lua
print("Hello, Building Tool!")
```

### `warn`

Writes a warning message to the Output window.

```lua
warn("Something went wrong")
```

### `assert`

Checks a condition and raises an assertion failure when it is false.

```lua
assert(player ~= nil, "Player is required")
```

Output is associated with the originating source and line through the runtime's debug information.

---

# Instances

`Instance` is the base object used throughout the Building Tool object hierarchy.

Instances behave similarly to Roblox instances, but are implemented as runtime proxy objects rather than full Roblox engine userdata.

```lua
local part = workspace:FindFirstChild("Part")

if part then
    print(part.Name)
end
```

## Instance Properties

Common properties include:

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | Instance name. |
| `ClassName` | `string` | Runtime class name. |
| `Parent` | `Instance?` | Parent instance. |
| `CFrame` | `CFrame`-like | Position and orientation. |
| `Position` | `Vector3`-like | World position. |
| `Orientation` | `Vector3`-like | Orientation in degrees. |
| `Size` | `Vector3`-like | Physical dimensions. |
| `Color` | `Color3`-like | Object color. |
| `Anchored` | `boolean` | Whether physics movement is disabled. |
| `CanCollide` | `boolean` | Whether the object participates in collision. |
| `Transparency` | `number` | Transparency value. |
| `CastShadow` | `boolean` | Whether the object casts a shadow. |

Property availability depends on the instance class.

## Instance Methods

### `Instance:GetChildren`

```lua
Instance:GetChildren() -> {Instance}
```

Returns the direct children of the instance.

```lua
for _, child in workspace:GetChildren() do
    print(child.Name)
end
```

### `Instance:FindFirstChild`

```lua
Instance:FindFirstChild(
    name: string,
    recursive: boolean?
) -> Instance?
```

Finds a child by name.

When `recursive` is true, descendants are searched as well.

```lua
local part = workspace:FindFirstChild("Part")

if part then
    print(part.Name)
end
```

### `Instance:FindFirstChildOfClass`

```lua
Instance:FindFirstChildOfClass(className: string) -> Instance?
```

Returns the first direct child whose class name matches `className`.

### `Instance:FindFirstChildWhichIsA`

```lua
Instance:FindFirstChildWhichIsA(className: string, recursive: boolean) -> Instance?
```

Returns the first direct child matching the class or one of its supported base classes. When `recursive` is true, descendants are searched as well.

### `Instance:FindFirstAncestor`

```lua
Instance:FindFirstAncestor(name: string) -> Instance?
```

Searches upward from the parent and returns the nearest ancestor with the matching name.

### `Instance:FindFirstAncestorOfClass`

```lua
Instance:FindFirstAncestorOfClass(className: string) -> Instance?
```

Returns the nearest ancestor whose class name exactly matches `className`.

### `Instance:FindFirstAncestorWhichIsA`

```lua
Instance:FindFirstAncestorWhichIsA(className: string) -> Instance?
```

Returns the nearest ancestor matching the class or one of its supported base classes.

### `Instance:FindFirstDescendant`

```lua
Instance:FindFirstDescendant(name: string) -> Instance?
```

Searches descendants recursively and returns the first instance with the matching name.

### `Instance:WaitForChild`

```lua
Instance:WaitForChild(
    name: string,
    timeout: number?
) -> Instance?
```

Waits for a child to become available.

```lua
local folder = workspace:WaitForChild("Folder")
```

### `Instance:GetService`

```lua
Instance:GetService(
    serviceName: string
) -> Instance? | service
```

Returns a service associated with the requested name.

Building Tool also resolves several API-only services through this interface, including `RunService`, `TweenService`, `UserInputService`, and `SoundService`.

```lua
local RunService = game:GetService("RunService")
```

### `Workspace:Raycast`

```lua
Workspace:Raycast(
    origin: Vector3,
    direction: Vector3,
    params: RaycastParams?
) -> RaycastResult?
```

Raycasts against Workspace parts and returns `Position`, `Normal`, `Distance`,
`InstanceName`, and `InstancePath`, or `nil` when nothing is hit. `RaycastParams.new()`
supports `FilterDescendantsInstances` with `Enum.RaycastFilterType.Exclude` or
`Enum.RaycastFilterType.Include`.

```lua
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Exclude
params.FilterDescendantsInstances = { character }
local result = workspace:Raycast(origin, direction, params)
```

### `Instance:GetFullName`

```lua
Instance:GetFullName() -> string
```

Returns the instance's dotted hierarchy path.

```lua
print(script:GetFullName())
-- game.ServerScriptService.MyScript
```

### `Instance:IsA`

```lua
Instance:IsA(className: string) -> boolean
```

Checks whether an instance belongs to the specified class when supported.

```lua
if part:IsA("Part") then
    print("This is a Part")
end
```

### `Instance:Clone`

```lua
Instance:Clone() -> Instance
```

Creates a copy of an instance.

> Support is intentionally limited compared with Roblox.

### `Instance:Destroy`

```lua
Instance:Destroy()
```

Destroys the instance.

> Support is intentionally limited compared with Roblox.

---

# Signals

Many instances expose signals such as:

- `Changed`
- `Touched`
- `AncestryChanged`

Signals can be connected using:

```lua
local connection = signal:Connect(function(...)
    -- handle event
end)
```

Connections can later be disconnected:

```lua
connection:Disconnect()
```

Signals also support:

```lua
signal:Once(callback)
signal:Fire(...)
```

Example:

```lua
part.Touched:Connect(function(other)
    print("Touched by:", other.Name)
end)
```

---

# Creating Instances

## `Instance.new`

```lua
Instance.new(
    className: string,
    parent: Instance?
) -> Instance
```

Creates a new instance.

```lua
local part = Instance.new("Part")
part.Name = "MyPart"
part.Parent = workspace
```

A parent may be supplied directly:

```lua
local folder = Instance.new("Folder", workspace)
```

### Supported Creatable Classes

Commonly creatable classes include:

```text
Part
Model
Folder
Script
LocalScript
ModuleScript
Sound
Texture
WeldConstraint
Motor6D
BindableEvent
BindableFunction
RemoteEvent
RemoteFunction
Humanoid
Camera
BlurEffect
ColorCorrectionEffect
BloomEffect
SunLight
```

Runtime-created instances are tracked by the host and cleaned up when Play stops.

---

# Scripts and Modules

Building Tool follows Roblox-style script execution conventions.

| RunContext | Class | Execution |
|---|---|---|
| `Server` | `Script` | Server Play and Client Play |
| `Client` | `LocalScript` | Client Play |
| `Module` | `ModuleScript` | Loaded through `require` |

A `ModuleScript` does not automatically execute.

```lua
local Shared = require(
    game.ReplicatedStorage.SharedModule
)
```

`require` caches the first returned module value for the runtime session.

Inside a module, `script` refers to the `ModuleScript` instance.

---

# Services

Services can be obtained through `game:GetService()`.

```lua
local RunService = game:GetService("RunService")
```

## Runtime Services

The following API services are always available:

| Service | Purpose |
|---|---|
| `RunService` | Frame timing and runtime context. |
| `TweenService` | Property interpolation. |
| `UserInputService` | Keyboard and mouse input helpers. |
| `SoundService` | Sound routing helpers. |

## Tree Services

Depending on the project, the following services may exist under `game`:

```text
Workspace
Players
Lighting
ReplicatedStorage
ServerStorage
ServerScriptService
StarterGui
StarterPack
StarterPlayer
```

---

## RunService

`RunService` provides frame events and runtime context checks.

```lua
local RunService = game:GetService("RunService")
```

### `RunService:IsClient`

```lua
RunService:IsClient() -> boolean
```

Returns whether the current execution context is the client.

### `RunService:IsServer`

```lua
RunService:IsServer() -> boolean
```

Returns whether the current execution context is the server.

### `RunService:IsStudio`

```lua
RunService:IsStudio() -> boolean
```

Returns whether the runtime is executing inside the editor.

### `RunService:IsRunning`

```lua
RunService:IsRunning() -> boolean
```

Returns whether Play mode is currently active.

### `RunService.Heartbeat`

```lua
RunService.Heartbeat:Connect(
    callback: (dt: number) -> ()
)
```

Fires every frame while the runtime is running.

```lua
RunService.Heartbeat:Connect(function(dt)
    print("Delta time:", dt)
end)
```

### `RunService.RenderStepped`

```lua
RunService.RenderStepped:Connect(
    callback: (dt: number) -> ()
)
```

Client-oriented per-frame signal.

### `RunService.Stepped`

```lua
RunService.Stepped:Connect(
    callback: (time: number, dt: number) -> ()
)
```

Fires during the runtime step.

---

# TweenService

`TweenService` interpolates supported properties over time.

```lua
local TweenService = game:GetService("TweenService")

local tween = TweenService:Create(
    part,
    TweenInfo.new(1),
    {
        Position = Vector3.new(0, 10, 0)
    }
)

tween:Play()
```

Tween controls include:

```lua
tween:Play()
tween:Cancel()
```

Interpolation support is simplified compared with Roblox and primarily covers numbers and common vector/color-like values.

---

# Events and Functions

## BindableEvent

`BindableEvent` communicates between scripts in the same execution context.

```lua
event.Event:Connect(function(message)
    print(message)
end)

event:Fire("Hello")
```

Bindable events do not provide network communication.

## RemoteEvent

`RemoteEvent` follows Roblox-style server/client event patterns.

### Server

```lua
remote.OnServerEvent:Connect(function(player, ...)
    -- client → server
end)

remote:FireClient(player, ...)
remote:FireAllClients(...)
```

### Client

```lua
remote.OnClientEvent:Connect(function(...)
    -- server → client
end)

remote:FireServer(...)
```

> Building Tool currently implements remotes as a same-process prototype. They should not be treated as real multiplayer networking.

## BindableFunction

```lua
fn.OnInvoke = function(...)
    return ...
end

local result = fn:Invoke(...)
```

## RemoteFunction

### Server

```lua
remote.OnServerInvoke = function(...)
    return ...
end
```

### Client

```lua
local result = remote:InvokeServer(...)
```

```lua
local result = remote:InvokeClient(player, ...)
```

---

# Datatypes

Building Tool provides several Roblox-inspired datatypes.

## Vector2

```lua
Vector2.new(x?: number, y?: number)
```

Common constants:

```lua
Vector2.zero
Vector2.xAxis
Vector2.yAxis
```

Components:

```lua
vector.X
vector.Y
```

Supported operations include:

```text
Vector2 + Vector2
Vector2 - Vector2
-Vector2
Vector2 * scalar
Vector2 / scalar
```

## Vector3

```lua
Vector3.new(
    x?: number,
    y?: number,
    z?: number
)
```

Common constants include:

```lua
Vector3.zero
Vector3.xAxis
Vector3.yAxis
Vector3.zAxis
Vector3.forward
Vector3.back
```

Components:

```lua
vector.X
vector.Y
vector.Z
```

Supported operations include:

```text
Vector3 + Vector3
Vector3 - Vector3
-Vector3
Vector3 * scalar
Vector3 / scalar
```

## Color3

```lua
Color3.new(
    r?: number,
    g?: number,
    b?: number
)

Color3.fromRGB(
    r: number,
    g: number,
    b: number
)
```

Components are exposed through:

```lua
color.R
color.G
color.B
```

`Color3.new` uses the normalized `0–1` range, while `Color3.fromRGB` uses the `0–255` range.

## CFrame

```lua
CFrame.new(
    x?: number,
    y?: number,
    z?: number
)

CFrame.new(vector3Like)

CFrame.Angles(
    x?: number,
    y?: number,
    z?: number
)
```

Common properties:

```lua
cframe.Position
cframe.Rotation
cframe.LookVector
cframe.RightVector
cframe.UpVector
```

When available, CFrame proxies support:

```lua
cframe:Lerp(goal, alpha)
```

## TweenInfo

```lua
TweenInfo.new(
    time?: number,
    easingStyle?,
    easingDirection?,
    repeatCount?,
    reverses?,
    delayTime?
)
```

## Additional Datatypes

The runtime also exposes simplified versions of:

```text
UDim
UDim2
BrickColor
NumberRange
NumberSequence
ColorSequence
Ray
Region3
Random
```

Their available constructors and behavior may differ from Roblox.

---

# Enums

Enums use Roblox-style access:

```lua
Enum.Material.Plastic
Enum.PartType.Block
Enum.KeyCode.W
Enum.UserInputType.MouseButton1
Enum.EasingStyle.Quad
Enum.EasingDirection.Out
Enum.NormalId.Top
Enum.CameraType.Scriptable
Enum.HumanoidStateType.Running
```

Additional enum-like namespaces may include:

```text
ScaleType
Font
SurfaceType
Shape
```

`Shape` is an alias of `PartType` where provided.

> Enum values may be represented internally as numbers or strings. Use the Enum API rather than relying on their underlying representation.

---

# Task and Timing

Building Tool exposes Luau-style task scheduling:

```lua
task.wait(seconds?)
task.spawn(callback, ...)
task.defer(callback, ...)
task.delay(seconds, callback, ...)
```

Legacy timing functions may also be available:

```lua
tick()
time()
wait(seconds?)
delay(seconds, callback)
spawn(callback)
```

Prefer the `task` library for new code.

---

# Standard Libraries

The runtime provides common Luau libraries:

```text
math
string
table
bit32
coroutine
debug
os
utf8
buffer
```

Common global helpers include:

```text
pairs
ipairs
tonumber
tostring
type
typeof
select
unpack
pcall
xpcall
setmetatable
getmetatable
rawget
rawset
rawequal
next
```

---

# hmath

> **Building Tool extension**

`hmath` is a custom mathematical expression library designed specifically for the Building Tool runtime.

Unlike Luau's standard `math` library, `hmath` can evaluate mathematical expressions represented as **strings**. This is useful for formulas that need to be stored, configured, reused, or treated as data.

```lua
local damageFormula =
    "\\[1] * (1 + \\[2] / 100)"

local damage = hmath.formula(
    damageFormula,
    100,
    25
)

print(damage)
-- 125
```

## `hmath.formula`

```lua
hmath.formula(
    formulaString: string,
    ...: number
) -> number
```

Evaluates a mathematical expression.

### Formula Arguments

Arguments are referenced using:

```text
\[1]
\[2]
\[3]
...
```

The index is **one-based**.

```lua
hmath.formula("\\[1] + \\[2]", 10, 20)
-- 30
```

### Operators

`hmath` supports normal arithmetic, C/C++-style bitwise operators, comparisons, and exponentiation.

| Operator | Operation |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulo |
| `**` | Exponentiation |
| `&` | Bitwise AND |
| `^` | Bitwise XOR |
| `\|` | Bitwise OR |
| `~` | Bitwise NOT |
| `<<` | Left shift |
| `>>` | Right shift |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |
| `==` | Equal |
| `~=` | Not equal |

Bitwise operations use Luau's `bit32` semantics and operate on 32-bit integer values.

> **Important:** `^` is **bitwise XOR**, not exponentiation. Use `**` or `pow(a, b)` for exponentiation.

```lua
hmath.formula("2 ** 10")
-- 1024

hmath.formula("1 << 8")
-- 256

hmath.formula("255 & 15")
-- 15

hmath.formula("4 | 2")
-- 6

hmath.formula("15 ^ 8")
-- 7

hmath.formula("~0")
-- -1

hmath.formula("256 >> 4")
-- 16
```

Operators can be combined:

```lua
hmath.formula("(1 << 8) | 15")
-- 271

hmath.formula("(255 & 15) ^ 3")
-- 12

hmath.formula("(16 >> 2) << 4")
-- 64
```

### Operator Precedence

From highest to lowest:

1. Parentheses and function calls
2. Unary `+`, `-`, `~`
3. Exponentiation `**`
4. Multiplication, division, modulo `*`, `/`, `%`
5. Addition and subtraction `+`, `-`
6. Bit shifts `<<`, `>>`
7. Bitwise AND `&`
8. Bitwise XOR `^`
9. Bitwise OR `|`
10. Comparisons `<`, `<=`, `>`, `>=`, `==`, `~=`

For example:

```lua
hmath.formula("1 + 2 << 3")
-- 24
```

Use parentheses when the intended grouping is not obvious:

```lua
hmath.formula("(1 + 2) << 3")
-- 24

hmath.formula("1 + (2 << 3)")
-- 17
```

### Parentheses

```lua
hmath.formula("(2 + 3) * 4")
-- 20
```

### Unary Operators

```lua
hmath.formula("-5")
-- -5

hmath.formula("-(2 + 3)")
-- -5

hmath.formula("~15")
-- bitwise NOT of 15
```

### Constants

Built-in constants include:

```text
pi
tau
e
inf
phi
sqrt2
sqrt3
ln2
ln10
zero
one
```

Examples:

```lua
hmath.formula("2 * pi")
-- 6.283185...

hmath.formula("sqrt2 ** 2")
-- 2
```

### Mathematical Functions

#### Basic

```text
abs(x)
sign(x)

min(...)
max(...)

floor(x)
ceil(x)
trunc(x)

sqrt(x)
cbrt(x)

square(x)
cube(x)
recip(x)
average(a, b)
```

Examples:

```lua
hmath.formula("abs(-10)")
-- 10

hmath.formula("max(10, 20, 5)")
-- 20

hmath.formula("square(7)")
-- 49
```

#### Powers and Logarithms

```text
pow(a, b)
exp(x)
log(x)
ln(x)
log10(x)
```

Examples:

```lua
hmath.formula("pow(2, 10)")
-- 1024

hmath.formula("log10(1000)")
-- 3
```

#### Trigonometry

```text
sin(x)
cos(x)
tan(x)

asin(x)
acos(x)
atan(x)
atan2(y, x)
```

`atan2(y, x)` is useful when the angle must account for the quadrant.

#### Hyperbolic Functions

```text
sinh(x)
cosh(x)
tanh(x)

asinh(x)
acosh(x)
atanh(x)
```

#### Angle Conversion

```text
deg(x)
rad(x)
```

#### Interpolation and Ranges

```text
clamp(x, min, max)
lerp(a, b, t)
inverseLerp(a, b, value)
remap(value, inMin, inMax, outMin, outMax)
```

Examples:

```lua
hmath.formula("clamp(150, 0, 100)")
-- 100

hmath.formula("lerp(0, 100, 0.25)")
-- 25

hmath.formula("remap(50, 0, 100, 0, 1)")
-- 0.5
```

#### Interpolation Curves

```text
step(edge, x)
smoothstep(edge0, edge1, x)
smootherstep(edge0, edge1, x)
```

#### Miscellaneous

```text
fract(x)
hypot(a, b)
distance(a, b)
mod(a, b)
fmod(a, b)
neg(x)
```

### Random Functions

```text
random()
random(max)
random(min, max)

randomf()
randomf(min, max)
```

Examples:

```lua
hmath.formula("random(1, 100)")

hmath.formula("randomf(0, 1)")

hmath.formula("randomf(-10, 10)")
```

`random()` uses the runtime's `math.random` generator.

### Bitwise Expressions

Bitwise operators are intended to provide compact expressions similar to C/C++:

```lua
hmath.formula("flags & 7")
```

When using formula arguments:

```lua
hmath.formula("\\[1] & 0xFF", 1025)
```

> Hexadecimal literals such as `0xFF` are **not currently supported**. Use decimal values or `pow`/shift expressions instead.

For example:

```lua
hmath.formula("\\[1] & 255", 1025)
-- 1

hmath.formula("1 << 8")
-- 256
```

### Implicit Multiplication

Common mathematical shorthand is supported:

```lua
hmath.formula("2pi")
hmath.formula("2(3 + 4)")
hmath.formula("2sqrt(16)")
```

This is equivalent to:

```text
2 * pi
2 * (3 + 4)
2 * sqrt(16)
```

### Comparisons

Comparisons return numeric values:

```text
true  -> 1
false -> 0
```

Examples:

```lua
hmath.formula("10 > 5")
-- 1

hmath.formula("10 == 5")
-- 0

hmath.formula("10 ~= 5")
-- 1
```

They can therefore be used inside arithmetic expressions:

```lua
hmath.formula("(10 > 5) * 100")
-- 100
```

### Formula Design

The purpose of `hmath` is to separate **mathematical logic from the code that uses it**.

Instead of:

```lua
local result = damage * (1 + bonus / 100)
```

a formula can be represented as:

```lua
local formula = "\\[1] * (1 + \\[2] / 100)"
```

and evaluated wherever needed:

```lua
local result = hmath.formula(formula, damage, bonus)
```

This makes formulas useful for configuration, balancing, calculators, and other data-driven systems.

# 2D GUI

Building Tool provides a Roblox-inspired 2D GUI hierarchy.

On client Play, children of `StarterGui` are cloned into the player's `PlayerGui`.

## Layer Containers

| Class | Purpose |
|---|---|
| `ScreenGui` | Full-screen 2D interface. |
| `SurfaceGui` | GUI rendered on a 3D surface. |
| `BillboardGui` | World-space GUI attached to an object. |

## GUI Objects

Supported GUI objects include:

```text
Frame
ScrollingFrame
TextLabel
ImageLabel
TextButton
ImageButton
TextBox
```

Common properties include:

```text
Size
Position
AnchorPoint
BackgroundColor3
BackgroundTransparency
BorderSizePixel
Visible
ZIndex
```

Text and image objects additionally expose their corresponding text/image properties.

## UI Modifiers

Common UI modifier classes include:

```text
UICorner
UIListLayout
UIStroke
UIPadding
UIGridLayout
UIAspectRatioConstraint
```

## StarterGui

`StarterGui` is the source hierarchy for client GUI objects.

On client Play:

```text
StarterGui
    ↓
PlayerGui
```

The editor can preview `ScreenGui` content from `StarterGui` in edit mode or from `PlayerGui` during client Play.

---

# Class Reference

## Part

A physical primitive used for geometry and physics.

Common properties:

```text
Position
Size
Orientation
Color
Anchored
CanCollide
Transparency
CastShadow
PartType
```

Supported `PartType` shapes include:

```text
Block
Sphere
Cylinder
Cone
Wedge
CornerWedge
```

## Model

A container for related instances.

Common properties include:

```text
PrimaryPart
PivotOffset
WorldPivot
```

`WorldPivot` is read-only.

## Sound

Common properties:

```text
SoundId
Volume
PlaybackSpeed
Looped
Playing
TimePosition
```

Methods:

```lua
sound:Play()
sound:Stop()
```

## Texture

Common properties:

```text
Texture
Face
StudsPerTileU
StudsPerTileV
Transparency
Color3
```

Supported faces:

```text
Top
Bottom
Front
Back
Left
Right
```

## WeldConstraint

Connects two parts.

```text
Part0
Part1
Enabled
```

## Motor6D

A joint used for articulated part relationships.

```text
Part0
Part1
Enabled
```

Motor6D instances are typically parented under `Part1`.

## Humanoid

Provides character-like properties and state information.

Common properties:

```text
Health
MaxHealth
WalkSpeed
JumpPower
HipHeight
RootPart
Mode
Target
```

## Camera

Common properties:

```text
Mode
Target
CFrame
```

Supported camera modes include:

```text
Default
Scriptable
```

## Lighting

`Lighting` exposes environment and scene lighting controls.

Common properties include:

```text
Ambient
OutdoorAmbient
ColorShift_Top
ColorShift_Bottom
FogColor
FogStart
FogEnd
Brightness
ExposureCompensation
EnvironmentDiffuseScale
EnvironmentSpecularScale
GlobalShadows
ClockTime
GeographicLatitude
```

Lighting effects include:

```text
Blur
```

---

# Physics

During Play, unanchored `Part` instances participate in simplified physics.

The runtime provides:

- Gravity
- Collision
- C++-side physics stepping

When Play stops, physics velocity is restored/reset by the host.

For deterministic scripting, prefer explicitly controlling:

```lua
part.Anchored
part.Position
part.CFrame
```

The full Roblox velocity API is not exposed.

---

# Server and Client

Building Tool supports two execution perspectives.

| Mode | Scripts | Context |
|---|---|---|
| Server Play | `Script` | Server |
| Client Play | `Script`, `LocalScript` | Client |

`RunService` can be used to determine the current context:

```lua
if RunService:IsServer() then
    print("Server")
elseif RunService:IsClient() then
    print("Client")
end
```

## Client Pointer

During client Play:

```lua
pointer
```

refers to:

```lua
game.Players.LocalPlayer.Character
```

On the server, `pointer` is `nil`.

## Replication

Client-side structural changes to pre-existing instances are restricted.

Instances created at runtime are handled differently and may be modified according to the runtime's rules.

> Remote objects are currently a same-process simulation. They do not provide real multiplayer networking.

---

# Examples

## Hello World

```lua
print("Hello from Luau!")

task.wait(1)

print("One second later")
```

## Detect Runtime Context

```lua
local RunService = game:GetService("RunService")

print("Server:", RunService:IsServer())
print("Client:", RunService:IsClient())
print("Studio:", RunService:IsStudio())
```

## Frame Loop

```lua
local RunService = game:GetService("RunService")

RunService.Heartbeat:Connect(function(dt)
    print("Frame time:", dt)
end)
```

## Find and Modify a Part

```lua
local part = workspace:FindFirstChild("Part")

if part then
    part.Anchored = false
    part.Position = Vector3.new(0, 10, 0)
end
```

## Load a Module

```lua
local Shared = require(
    game.ReplicatedStorage:WaitForChild("SharedModule")
)
```

## Server/Client Remote Prototype

```lua
local RunService = game:GetService("RunService")
local remote = game.ReplicatedStorage:FindFirstChild("TestRemote")

if remote and RunService:IsServer() then

    remote.OnServerEvent:Connect(function(player, message)
        print("From client:", message)

        remote:FireClient(player, "ack")
    end)

elseif remote and RunService:IsClient() then

    remote.OnClientEvent:Connect(function(message)
        print("From server:", message)
    end)

    remote:FireServer("hello")
end
```

---

# Unsupported APIs

Building Tool intentionally does not expose the complete Roblox API.

The following systems are currently unavailable or substantially incomplete:

```text
PathfindingService
DataStoreService
HttpService
TeleportService
Chat
Teams
MarketplaceService
MeshPart
Union / CSG
AnimationController
Full Roblox UI hierarchy
Full Roblox CFrame API
Real multiplayer networking
Most constraints beyond WeldConstraint and Motor6D
```

This list describes the current Building Tool runtime rather than Roblox API availability.

---

# Compatibility Notes

Building Tool intentionally uses Roblox-like naming and behavior where practical, but compatibility is **conceptual rather than complete**.

Code written for Roblox may require changes because:

- Some classes do not exist.
- Some properties are simplified.
- Some methods are unavailable.
- Engine userdata is represented by proxy objects.
- Physics is simplified.
- Networking is simulated.
- Several services are editor-only implementations.
- Datatypes do not necessarily implement the complete Roblox surface.

When writing code intended specifically for Building Tool, prefer this reference over assumptions based on the full Roblox API.

---

# API Design Philosophy

The Building Tool API follows three principles:

### Familiar

Roblox developers should recognize common patterns such as:

```lua
game:GetService(...)
Instance.new(...)
object:FindFirstChild(...)
signal:Connect(...)
Vector3.new(...)
```

### Focused

Only APIs that are meaningful inside the editor's runtime are exposed.

### Extensible

Building Tool can provide custom runtime libraries alongside the Roblox-inspired API.

`hmath` is one example: it extends Luau with a data-driven mathematical expression system without pretending to be part of the standard Roblox API.

---

> **Runtime note:** This reference describes the API injected by `buildLuauDefinitions()` and the corresponding Building Tool runtime. Behavior follows the current editor implementation and should not be assumed to provide full Roblox compatibility.
