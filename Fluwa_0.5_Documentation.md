# Fluwa 0.5 — Full Documentation

Lua scripting for **Minecraft 1.20.1 (Forge 47)** in the style of Roblox Studio.

- **Part I** is a guide that explains how Fluwa works.
- **Part II** is the API reference: every class, property, method and event. It was built from the mod's source code.

---

## Contents
1. Installation
2. ZSpace (F8) and the tools
3. Scripts: folders, order, threads
4. Lua in Fluwa: what is different
5. Globals
6. Objects, hierarchy, properties
7. Events and signals
8. Network events (server ↔ client)
9. Data types: Vector3, CFrame, Color, UDim2
10. World: Workspace, Dimensions, Blocks, Chunks, Structures
11. Players and entities
12. Attributes
13. Tweens (smooth movement) and collision
14. Block ↔ BlockEntity
15. Hitbox and models
16. Lighting and lighting technologies
17. UI (client)
18. Services (`getAccess`)
19. Forge and Java access
20. Configuration and security
21. Commands
22. Recipes (examples)
23. Common errors

Part II — API reference

---

# Part I — Guide

## 1. Installation
1. Install Forge 1.20.1 (47.x) and put `fluwa-0.5.jar` into `mods/`. LuaJ is bundled inside the jar.
2. The mod must be installed on the **server and on every client**, and the versions must match.
3. When you create a world, example scripts appear in `<world>/fluwa/`.

Fluwa folders:

| Folder | Purpose |
|---|---|
| `<world>/fluwa/InitScripts`, `ServerScripts`, `ClientScripts`, `UI` | World scripts (`*.lua`) |
| `<world>/fluwa/models/` | Models `.obj/.gltf/.glb/.fbx` (+ `.png` texture) |
| `<world>/fluwa/bin/` | FASM binaries |
| `<world>/fluwa/web/`, `<game>/fluwa/web/` | Local web pages |
| `<game>/fluwa/sounds/` | Local sounds `.ogg/.wav` |
| `<game>/fluwa/videos/` | Videos for VideoFrame |
| `<game>/fluwa/screens/` | Edited Minecraft / mod screens (GUI Maker) |
| `<world>/datapacks/fluwa_generated` | Generated dimensions (apply after restart) |

## 2. ZSpace (F8) and the tools
Press **F8** in a world to open ZSpace. It is available to **Creators**: players with op level ≥ `creatorPermissionLevel` (default 2), or players listed in `coCreators`.

- **Explorer.** The tree contains Workspace (Lighting, Terrain, Entities, Players, Dimensions, Models), UI, InitScripts, ServerScripts and ClientScripts. It has search, a property editor, and a **“+”** button for inserting objects.
- **Code editor.** It has tabs, syntax highlighting, syntax checking and smart autocomplete. Autocomplete infers types, shows members after `.`/`:`, completes strings inside `Instance.new("…")`/`getAccess("…")`, and supports fuzzy search.
  - Ctrl+S saves the script.
  - With **Reload scripts on save** turned on, scripts reload when you save. With it off, use Ctrl+Shift+R.
- **Console** at the bottom shows output and has a command line.
- **Camera mode** (bottom-left button) gives a free spectator camera while your body stays in place. Use WASD + Space/Shift to move and the mouse to look.
- **☰ → Tools:**
  - **World Editor.** Its modes are Blocks, Entities and Structures (switch with Tab).
    - Blocks: place and edit blocks.
    - Entities: spawn and edit entities (including other players). Fields: Name, Position, Yaw, Health, MaxHealth, Model, ModelScale, **Hitbox**. You can also edit model parts.
    - Structures: place, save and edit structures.
  - **GUI Maker.** Visual UI editor. “Open / Import…” opens existing GUIs, running ScreenGUIs, and vanilla or mod screens.
    - Minecraft and mod screens are edited in place and saved to `fluwa/screens`. The original buttons keep working.
    - New buttons get an Action: `/command`, `close`, `screen:<class>`, `url:`, `lua:`.
    - “Restore original” undoes the edits. F8 in the main menu opens GUI Maker (`mainMenuGuiMaker=true`).
  - **Shader DK** for shaders, and **Terrain DK** for terrain generation.
- **Options** include Reload scripts on save and World Editor UI size.

## 3. Scripts: folders, order, threads
| Folder | Where it runs | When |
|---|---|---|
| `InitScripts` | Server **and** every client | First. Create network Events and modules for `CAR` here |
| `ServerScripts` | Server only | After InitScripts |
| `ClientScripts` | Every client | After join (the server sends them) |
| `UI` | Every client | After ClientScripts |

- Every script file runs in its own thread, and every script has a `script` object in the hierarchy (`script.Parent` is its folder).
- If a script runs longer than `scriptTimeoutMs` (default 3000 ms) without calling `wait()`, it stops with the error **“Script timeout”**.
- Lua runs on helper threads. Every access to the world is automatically passed to the game thread, so it is safe.
- Event handlers (`Connect`) run in their own coroutines, so **`wait()` works inside handlers** (`asyncEventHandlers=true`). If you need a synchronous handler, use `ConnectSync`.
- Errors are throttled: the same message is shown once per 5 s. A handler that fails 100 times in a row is disconnected.

## 4. Lua in Fluwa: what is different
- The Lua VM is LuaJ 3.0.1 (Lua 5.2).
- **`require` is replaced by `CAR(name)`** (Check And Require). It loads a module from InitScripts or from the same folder and caches the result. If the module is missing or has an error, it returns `nil` with a warning instead of throwing.
- **`getAccess(name)`** gives a service, like `game:GetService` in Roblox. See section 18.
- `io`, the full `os`, `luajava`, `debug`, `dofile`, `loadfile`, `require` and `package` are removed.
- Methods can usually be called with both `:` and `.`, and many have short aliases (`Player:PS` = `PlaySound`, `Event:FS` = `FireServer`).
- `obj:Method()` methods are listed in the reference as `:Method(args)`. Arguments in `[ ]` are optional, and `=value` shows the default.

## 5. Globals
| Name | Meaning |
|---|---|
| `World`, `Workspace`, `workspace`, `game` | Workspace object |
| `Players`, `Lighting`, `Terrain` | Shortcuts to `World.Players` / `World.Lighting` / `World.Terrain` |
| `Dimension` | Overworld on the server; the current dimension on the client |
| `UI` | Client UI root |
| `Events` | Folder with all network Events (by Name) |
| `InitScripts`, `ServerScripts`, `ClientScripts` | Script folders (objects created there are visible by name, e.g. `InitScripts.MainEvent`) |
| `script` | The object of the current script file |
| `Instance`, `Vector3`/`Vec3`, `Vector2`, `CFrame`, `UDim`, `UDim2`, `Color`/`Color3` | Constructors |
| `CAR`, `getAccess`/`GetAccess` | Modules / services |
| `IsServer`, `IsClient` | Booleans |
| `print(...)`, `warn(...)` | Write to the ZSpace console and the log |
| `tick()` | Seconds since start |
| `wait(sec)` | Pause the current thread; returns the elapsed time |
| `spawn(fn, ...)` | Run fn in a new thread on the next tick |
| `delay(sec, fn, ...)` | Run fn after sec seconds |
| `task.wait / task.spawn / task.delay` | Same as above |

## 6. Objects, hierarchy, properties
Every Fluwa object works like a Roblox Instance:
- **Universal properties:** `Name` (RW; read-only for world objects), `ClassName` (R), `Parent` (RW for created objects).
- **Universal methods:** `Destroy()`, `GetChildren()`, `GetDescendants()`, `FindFirstChild(name)`, `WaitForChild(name, [timeout])`, `FindFirstChildOfClass`, `FindFirstChildWhichIsA`, `FindFirstAncestor*`, `IsA(class)`, `IsDescendantOf`, `IsAncestorOf`, `GetFullName`, `GetAncestors`, `ClearAllChildren()`, `Clone()` (Folder / Values / UI).
- **Universal events:** `ChildAdded`, `ChildRemoved`.
- **Custom fields:** `obj.MyField = 5` stores your own value on the object.

Create objects with `Instance.new("ClassName", [parent])` (`Instance:new` also works):
```lua
local folder = Instance.new("Folder", workspace)
local coins = Instance.new("IntValue", folder)
coins.Name = "Coins"
coins.Value = 10
coins.Changed:Connect(function(v) print("Coins:", v) end)
```
Value classes: `IntValue`, `NumberValue`, `BoolValue`, `StringValue`, `TableValue`, `Vector3Value`, `ObjectValue`. Values and Folders placed under a Player are replicated to that player's client.

## 7. Events and signals
Every event is a signal:
```lua
local conn = signal:Connect(function(...) end)   -- async handler (wait() allowed)
signal:ConnectSync(fn)                           -- synchronous handler
signal:Once(fn)                                  -- fires once
local args = signal:Wait()                       -- wait for the next fire
conn:Disconnect(); signal:DisconnectAll()
```
- Most objects have events with the **`E_` prefix**, such as `player.E_Death`, `item.E_Used`, `button.E_Click`. Part II lists each class's events.
- Some events also have Roblox-style names: `Changed`, `Died`, `Chatted`, `MouseButton1Click`, `Activated`, `OnServerFire`, and so on.
- `BindableEvent` sends events within one side: `be:Fire(...)`, `be.Event:Connect(fn)`.

## 8. Network events (server ↔ client)
Create an `Event` in **InitScripts** so it exists on both sides. Events are matched by `Name`.
```lua
-- InitScripts/Events.lua
local ev = Instance.new("Event", InitScripts)
ev.Name = "MainEvent"

-- ClientScripts
local mouse = Players.LocalPlayer:GetMouse()
InitScripts.MainEvent:FireServer(mouse.Target)

-- ServerScripts
InitScripts.MainEvent.OnServerFire:Connect(function(plr, block)
    print(plr.Name, "clicked", block.Position)
end)
```
- Methods: `FireServer` (`FS`), `FirePlayer`/`FireClient` (`FP`), `FireAllPlayers`/`FireAllClients` (`FAP`), `FireAll` (`FA`).
- Events: `OnServerFire` (on the server; the first argument is the player) and `OnClientEvent` / `OnPlayerFire` (on the client).
- Allowed arguments: nil, boolean, number, string, tables, Vector3, CFrame, Color, UDim2, Player, Entity, Block and hierarchy objects. Objects arrive as objects, not names.

## 9. Data types
**Vector3**: `Vector3.new(x, y, z)`.
- Fields: `X, Y, Z`, `Magnitude`, `Unit`.
- Operators: `+ - * /` (with a Vector3 or a number), `==`.
- Methods: `Dot`, `Cross`, `Lerp`.
- ⚠ `Vector3.new` takes **three numbers**, not three Vector3 values.

**CFrame** (position + rotation): `CFrame.new(x,y,z)`, `CFrame.new(pos, lookAt)`, `CFrame.Angles(rx, ry, rz)`.
- Fields: `Position`, `LookVector`, `RightVector`, `UpVector`, `X/Y/Z`.
- `cf * cf` combines CFrames; `cf * Vector3` transforms a point; `cf + Vector3` moves it.
- Methods: `Inverse`, `Lerp`, `ToWorldSpace`, `ToObjectSpace`.

**Color**: `Color.new(r, g, b)` with values 0–255 (`Color3` is an alias).

**UDim2**: `UDim2.new(scaleX, offsetX, scaleY, offsetY)`; **UDim**: `UDim.new(scale, offset)`; **Vector2**: `Vector2.new(x, y)`.

```lua
-- move 3 blocks right, up and forward relative to the block's orientation
local cf = arg.CFrame
local goal = arg.Position + cf.RightVector * 3 + cf.UpVector * 3 + cf.LookVector * 3
```

## 10. World
- **Workspace (`World`):** containers `Lighting`, `Terrain`, `Time`, `Entities`, `Players`, `Dimensions`.
- **Dimension:** read and write blocks and areas.
  - `GetBlock`, `SetBlock`, `SetBox`, `SetSphere`, `BlocksReplace`.
  - Entity search: `GetEntitiesFromBox` (`GEFB`).
  - `Raycast` returns a RayResult.
  - `GetChunk(...)` returns two values: Chunk and isLoaded.
- **Fill settings** for SetBox / SetSphere / BlocksReplace / Terrain:Fill: `{ add = {"minecraft:stone", "minecraft:andesite"}, weights = {1, 2, 2} }`. `weights[1]` is for the main block.
- **Block:** `Position`, `CFrame`, `Id`, `CanCollide` (RW), NBT (`ReadNBT`/`WriteNBT`/`SetNBT`), `Break`, `Gravity`, `ConnectScreen(ScreenGUI)`, tweens, `ToBlockEntity`.
- **Chunk:** blocks of a chunk, `Regenerate([decorate])`, tweens (the whole chunk moves).
- **Structure:**
  - Blocks: `GetSize`, `GetBlocks`, `GetBlock`, `SetBlock(pos, id, [nbt])`, `RemoveBlock`, `Fill`.
  - Entities: `GetEntities`, `AddEntity`.
  - `GetPivot`/`SetPivot`, `Clone`, placing in the world, tweens.
- **Terrain:** `Fill`, `FillSphere`, `Clear`, `Generate`.
- **Time:** `Add`, `Set`, `Secs`, `AddSec`, `Pause`, `Go`, `Speed`, `Slow`, `Normal`.
- **Writes per call** are limited by `maxBlocksPerCall`.

## 11. Players and entities
- `Players.LocalPlayer` (client), `Players:GetPlayers()`, and events `E_Join` / `E_Leave` on Players.
- **Player** has everything Entity has, plus:
  - Chat and screen: `SendMessage`, `ActionBar`, `Title`.
  - Inventory: `GiveItem`, `GetInventory`, `GetHotBar`, `GetArmor`, `GetMainHand`.
  - Survival: `Feed`, `GiveExp`, `Freeze`, `Sleep`.
  - Sync: `Async`/`Sync` (detach the player from server sync).
  - Client only: `GetMouse` / `GetCamera`.
  - Check methods: `isRun`, `isShift`, `isFly`, …
- **Entity:**
  - Health: `Hurt`, `Kill`, `Heal`.
  - Position: `PivotTo`/`Teleport`, `ApplyImpulse`, `GetDistance`.
  - Fire and effects: `Burn`, `Stew`/`Extinguish`, `GetEffect({effect, sec, level})`, `ClearEffect`.
  - NBT: `SetNBT`/`ReadNBT`.
  - Model and hitbox: `SetModel(name, [scale])`, `GetModel()` (model parts with `Offset`, `Rotation`, `Scale`, `Visible`), `SetHitbox`.
  - Tweens.
- **Models:**
  - `SetModel("name.obj", scale)` loads a model from `<world>/fluwa/models/`.
  - `SetModel("minecraft:zombie")` morphs the entity into a zombie; `SetModel(nil)` resets it.

## 12. Attributes
**Any object** (`SetAttribute`, `GetAttribute`, `GetAttributes`, event `E_AttributeChanged`).

**Player** has its own versions:
```lua
plr:AddAttribute("Mana", 100, function(new, old, name) print(name, old, "->", new) end)
plr:EditAttribute("Mana", 80)              --> true (runs the callback)
local has, v = plr:GetAttribute("Mana")    --> true, 80
plr:DelAttribute("Mana")                   --> true
plr:GetAttribute("Mana")                   --> false
plr:EditAttribute("MovementSpeed", 0.2)    -- Minecraft attributes work too
```
- `GetAttribute` returns `true, value` if the player has the attribute, otherwise `false`.
- `EditAttribute` and `DelAttribute` return `false` if there is no such attribute.
- **Minecraft attributes** such as `MaxHealth`, `MovementSpeed`, `AttackDamage` and `minecraft:generic.armor` also work: Edit changes the base value, and Del resets it to the default.
- Custom attribute values are not saved after a server restart.

## 13. Tweens (smooth movement) and collision
Available for **Entity, Player, Block, Chunk, Structure, BlockEntity**:
```lua
obj:TweenPosition(endPos, [dir="Out"], [style="Quad"], [time=1], [override=false], [callback], [canCollide=true])
obj:TweenCFrame(endCFrame, dir, style, time, override, callback, canCollide)
obj:CancelTween()   obj:IsTweening()
```
- **dir:** `In`, `Out`, `InOut`.
- **style:** `Linear`, `Quad`, `Cubic`, `Quart`, `Quint`, `Sine`, `Exponential`, `Circular`, `Back`, `Elastic`, `Bounce`.
- **callback** receives `"Completed"` or `"Canceled"`.
- **canCollide** can come after the callback or replace it: `..., override, false)`.
- **Movement is smooth.** The client draws the movement every frame. Blocks travel as a “ghost” and are placed for real at the end.
- **Collision:** with `canCollide=true`, moving blocks stop players and mobs and push them out of the way. Entities standing on top ride along.

```lua
InitScripts.MainEvent.OnServerFire:Connect(function(plr, block)
    local cf = block.CFrame
    local goal = block.Position + cf.RightVector * 3 + cf.UpVector * 3 + cf.LookVector * 3
    block:TweenPosition(goal, "In", "Quad", 6, false, function(state) print(state) end)
end)
```

## 14. Block ↔ BlockEntity
```lua
local be = block:ToBlockEntity()          -- or :ToBlockEntity(false) = no collision
be:TweenPosition(be.Position + Vector3.new(0, 5, 0), "Out", "Quad", 3, false, function()
    local b = be:ToSolidBlock()           -- back to a real block
end)
```
- **What is kept:** the block state, container contents / NBT and Fluwa block data.
- **`ToSolidBlock()`** places the block at the rounded position. It throws an error if that position is occupied.
- **Extras:** `SetCanCollide(bool)` and `IsBlockEntity()`.

## 15. Hitbox and models
- **ModelScale** changes only how the entity looks. The **hitbox** is set separately:
  - World Editor → Entities → **Hitbox** field: `width height`. One number makes a cube; blank or `reset` restores the normal hitbox.
  - Lua: `Entity:SetHitbox(w, h)`, `Entity:SetHitbox(nil)`, `Entity:GetHitbox()` → `w, h`.
- The hitbox is saved with the entity and synced to all players.

## 16. Lighting and lighting technologies
`Lighting` properties include:
- Time: `ClockTime`/`TimeOfDay`.
- Brightness and colour: `Brightness`, `Ambient`, `OutdoorAmbient`, exposure.
- Shadows: `GlobalShadows`, `ShadowSoftness`.
- Fog: `FogColor`/`FogStart`/`FogEnd`.
- Sun and moon: size, colour, texture, visibility.
- Stars: `StarCount`.
- `Technology`.

**Technology** (like Roblox):

| Value | Description |
|---|---|
| `Legacy` (0) | Vanilla Minecraft lighting |
| `Compatibility` (1) | Soft ambient light + tone mapping |
| `Voxel` (2) | Soft voxel light from light sources |
| `ShadowMap` (3) | Directional shadows from the sun |
| `Future` (4) | ShadowMap + point lights, specular, bloom |

Effect objects go into `Lighting`: `Atmosphere`, `Bloom`, `Blur`, `ColorCorrection`, `DepthOfField`, `SunRays`, `Sky`, `Clouds`. Light sources: `LightPoint`, `LightBeam`.
```lua
Lighting.Technology = "Future"
Lighting.ClockTime = 18
local bloom = Instance.new("Bloom", Lighting); bloom.Intensity = 0.6
```

## 17. UI (client)
- **Containers:** `ScreenGUI` (a screen with the mouse; `Open`, `Close`, `Toggle`, `PauseGame`, `CloseOnEsc`) and `Overlay` (HUD).
- **Elements:** `Frame`, `TextLabel`, `TextButton`/`Button`, `TextBox`, `ImageLabel`/`Image`, `ImageButton`, `AnimImage`, `ScrollingFrame`, `VideoFrame`, `ViewportFrame`.
- **Modifiers:** `UICorner`, `UIStroke`, `UIGradient`, `UIPadding`, `UIListLayout`, `UIGridLayout`.
- **Common properties:** `Position`/`Size` (UDim2), `AnchorPoint`, `BackgroundColor`, `BackgroundTransparency`, `Visible`, `ZIndex`, `Text`, `TextColor`, `Image = "namespace:path.png"`.
- **Events:** `E_Click`/`MouseButton1Click`/`Activated`, `MouseEnter`, `MouseLeave`, …
- UI elements also support tweens (`TweenPosition`/`TP`, `TweenSize`/`TS`, `TweenSizeAndPosition`/`TSP`).
- `Block:ConnectScreen(gui)` opens the screen when the block is right-clicked.

```lua
local gui = Instance.new("ScreenGUI", UI)
local b = Instance.new("TextButton", gui)
b.Size = UDim2.new(0, 200, 0, 40); b.Position = UDim2.new(0.5, -100, 0.5, -20)
b.Text = "Hello"
b.E_Click:Connect(function() gui:Close() end)
gui:Open()
```

- **Input:** `getAccess("UserInputService")` covers the keyboard, mouse, gamepad and touch (`E_InputBegan`, `E_InputEnded`, `IsKeyDown`, …).
- **Mouse:** `X`, `Y`, `Hit`, `Target`.
- **Camera:** `CFrame`, `FieldOfView`, `ThirdPerson`, `FreeCam`.

## 18. Services (`getAccess`)
| Service | What it does | Requirement |
|---|---|---|
| `OS` | OS info, `time/clock/date`; `Execute` | `Execute` needs `fullOsAccess` |
| `Runtime` | Runtime info, memory, GC | — |
| `BigInt` / `BigDec` | Huge integers / precise decimals | — |
| `LuaRT` | Run LuaRT (Windows) | `fullOsAccess`, luart.exe |
| `ShaderDT` | Post-processing GLSL shaders (client) | — |
| `Chromium` | HTTP (`Get/Post/Request`, async), JSON, Browser | `httpEnabled`, `httpAllowedDomains` |
| `NET` | TCP/UDP servers and clients, WebSocket | `netEnabled=true` |
| `FASM` | Flat Assembler | `fullOsAccess`, fasm |
| `Projector` | 3D models, bones, animations, materials | — |
| `UserInputService` | Input (client) | — |
| `Forge` | Forge mods, registries, events, configs, Java | see section 19 |

`AsyncTask` has `Done`, `Ok`, `Result`, `Error` and `task:Await()`.

## 19. Forge and Java access
`getAccess("Forge")` gives access to:
- Mods: `ForgeMod`.
- Registries: `ForgeRegistry` (blocks, items, entities…), with tags (`ForgeTag`).
- Recipes: `ForgeRecipe`.
- Commands: `ForgeCommand`.
- Configs: `ForgeConfig`.
- Events: the Forge event bus (`ForgeEventBus`, `ForgeEvent`, which can be cancelled).
- Capabilities: `ForgeCapability`.
- Deep reflection: `JavaObject` (calling Java classes and objects). Use it carefully.

## 20. Configuration and security
`config/fluwa-common.toml`:

| Key | Default | Meaning |
|---|---|---|
| `scriptTimeoutMs` | 3000 | Max run time without `wait()` |
| `creatorPermissionLevel` | 2 | Op level needed for ZSpace |
| `coCreators` | [] | Extra creators by name |
| `maxBlocksPerCall` | 262144 | Block write limit per call |
| `createExampleScripts` | true | Examples in new worlds |
| `fullOsAccess` | false | OS.Execute, LuaRT, FASM, absolute paths |
| `httpEnabled`, `httpAllowedDomains`, `httpAllowLocal` | false / [] / false | Chromium |
| `netEnabled` | false | NET service |
| `luartPath`, `fasmPath` | — | Paths to external programs |
| `asyncEventHandlers` | true | Handlers in coroutines |
| `mainMenuGuiMaker` | true | F8 in the main menu |

Only Creators can edit scripts. ClientScripts are sent by the server.

## 21. Commands
`/fluwa reload` · `/fluwa run <lua>` · `/fluwa scripts` · `/fluwa zspace`

## 22. Recipes
**Elevator platform** (collidable):
```lua
-- a structure saved in World Editor (Structures mode) under the name "elevator"
local s = World:GetStructure("elevator")
s:Place(Dimension, Vector3.new(0, 64, 0))
while true do
    s:TweenPosition(Vector3.new(0, 80, 0), "InOut", "Sine", 4); wait(5)
    s:TweenPosition(Vector3.new(0, 64, 0), "InOut", "Sine", 4); wait(5)
end
```
**Mana system:**
```lua
Players.E_Join:Connect(function(plr)
    plr:AddAttribute("Mana", 100, function(new) plr:ActionBar("Mana: " .. new) end)
end)
```
**Flying block on click:**
```lua
InitScripts.MainEvent.OnServerFire:Connect(function(plr, block)
    local be = block:ToBlockEntity()
    be:TweenPosition(be.Position + Vector3.new(0, 10, 0), "Out", "Back", 2, false, function()
        be:ToSolidBlock()
    end)
end)
```

## 23. Common errors
| Error | Cause / fix |
|---|---|
| `bad argument: double expected, got userdata` | A Vector3/object was passed where a number is expected (e.g. `Vector3.new(v1, v2, v3)`). Add the vectors with `+` instead |
| `X is not a valid member of Class "Name"` | Wrong property or child name; check the reference |
| `Script timeout` | A long loop without `wait()` |
| `attempt to call nil` | Method name typo, or a client-only method used on the server |
| `Player ... is not online` | The player left; check `plr.Online` |
| `position ... is occupied` | `ToSolidBlock` target is not empty |

---

# Part II — API reference

Generated from the source of Fluwa 0.5.
- `:Method(a, b)` gives the argument types: `string`, `number`, `integer`, `boolean`, `table`, `function`, an object class, or `any`. `[type=default]` means the argument is optional.
- Properties are marked R (read), W (write) or RW.
- Events are accessed as `obj.E_Name:Connect(fn)`.
- Methods of “Every object” and “Every event object” work on all classes.
- Some classes in the list are internal groupings (for example ServerEvents and ClientEvents). Their event names are fired on players, the world or Players.

### Every object (universal)

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `ChildAdded` | R |
| `ChildRemoved` | R |
| `ClassName` | R |
| `Name` | RW |
| `Parent` | RW |

**Methods**

- `:Destroy()` — alias: `destroy`, `Remove`
- `:GetChildren()` — alias: `getChildren`
- `:GetDescendants()`
- `:FindFirstChild(string, [boolean=false])`
- `:WaitForChild(string)`
- `:FindFirstChildOfClass(string)`
- `:FindFirstChildWhichIsA(string)`
- `:FindFirstAncestor(string)`
- `:FindFirstAncestorOfClass(string)`
- `:FindFirstAncestorWhichIsA(string)`
- `:IsDescendantOf(any)`
- `:IsAncestorOf(any)`
- `:GetFullName()`
- `:GetAncestors()` — GetAncestors(): parent, grand-parent, ... up to game
- `:IsA(string)`
- `:ClearAllChildren()`
- `:Clone(any, string, [number=-1])`

### Every event object (universal)

**Methods**

- `:SetAttribute(string, any)`
- `:GetAttribute(string)`
- `:GetAttributes()`
- `:SetParent(any)`
- `:GetParent()`
- `:SetName(string)`
- `:GetName()`

**Events:** `E_AttributeChanged`, `E_ChildAdded`, `E_ChildRemoved`

### Player

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Asynced` | R |
| `CharacterAdded` | R |
| `Chatted` | R |
| `CoreGui` | R |
| `Died` | R |
| `Dimension` | W |
| `Effects` | W |
| `Frozen` | R |
| `GameMode` | RW |
| `Hunger` | RW |
| `Hungry` | RW |
| `IsCreator` | R |
| `NBTs` | RW |
| `Name` | R |
| `Online` | R |
| `Ping` | RW |
| `PlayerGui` | R |
| `ResourcePack` | RW |
| `Saturation` | RW |
| `Shader` | RW |
| `UserName` | R |
| `XP` | RW |
| `XPPoints` | R |

**Methods**

- `:GetAttribute(string)` — GetAttribute(name) -> true, value \| false
- `:AddAttribute(string, number, any)` — AddAttribute(name, value, [callback(newValue, oldValue, name)]) -> true
- `:EditAttribute(string, number)` — EditAttribute(name, value) -> true if changed, false if the player has no such attribute
- `:DelAttribute(string)` — DelAttribute(name) -> true if removed (Minecraft attributes are reset to their default)
- `:Kick([string="Kicked by script"])`
- `:Ping()`
- `:Async()`
- `:Sync()`
- `:AsyncTick(integer)`
- `:Feed(integer, [integer=0])`
- `:Say(string)`
- `:SendMessage(string)` — alias: `Message`
- `:ActionBar(string)`
- `:Title(string, string, [integer=10], [integer=60], [integer=20])`
- `:Command(string)`
- `:PlaySound([number], [number=1], [number=1])` — alias: `PS`
- `:PlayLocalSound(string, [number=1], [number=1])` — alias: `PLS`
- `:Freeze([number=0], [boolean=false])`
- `:UnFreeze()` — alias: `Unfreeze`
- `:GetExp(integer)` — alias: `GiveExp`
- `:Sleep()`
- `:WakeUp()`
- `:SetNetworkOwner(any)`
- `:GetNetworkOwner()`
- `:GetMouse()`
- `:GetCamera()`
- `:GetModel()`
- `:GetInv()` — alias: `GetInventory`
- `:GetHotBar()` — alias: `GHB`
- `:GetArmor()`
- `:GetMainHand()` — alias: `GMH`
- `:GetSecondaryHand()` — alias: `GSH`
- `:GiveItem(string, [integer=1])`
- `:ClearInventory()`
- `:DrownStart()`
- `:DrownPause()`
- `:DrownEnd()`
- `:HideName()`
- `:ShowName()`
- `:isHungry([integer=0])` — ---- check methods
- `:isSleep()`
- `:isMHand()`
- `:isSHand()`
- `:isDrown()`
- `:isRun()`
- `:isShift()`
- `:isSit()`
- `:isFly()`
- `:isSwim()`
- `:isOnGround()`

**Events:** `E_ArmorChanged`, `E_Attack`, `E_AttributeChanged`, `E_Chatted`, `E_ChunkChange`, `E_Damaged`, `E_Death`, `E_DimensionChanged`, `E_DropItem`, `E_EffAdd`, `E_EffDel`, `E_Feed`, `E_Interact`, `E_Join`, `E_Jump`, `E_Leave`, `E_Move`, `E_PickItem`, `E_Respawn`, `E_Respawned`, `E_Sleep`, `E_Tick`, `E_Wake`

### Entity

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Alive` | R |
| `CFrame` | RW |
| `Dimension` | R |
| `Effects` | R |
| `Entity` | R |
| `EntityId` | R |
| `Frozen` | R |
| `HP` | RW |
| `HasCustomModel` | R |
| `Health` | RW |
| `Id` | RW |
| `IsHallucination` | R |
| `MaxHP` | RW |
| `MaxHealth` | RW |
| `Model` | R |
| `NBTs` | R |
| `Name` | R |
| `Offset` | RW |
| `OnGround` | R |
| `Orientation` | RW |
| `Owner` | R |
| `Parent` | R |
| `Position` | RW |
| `Rotation` | RW |
| `Scale` | RW |
| `Size` | RW |
| `Spawned` | R |
| `Type` | RW |
| `UUID` | R |
| `Velocity` | RW |
| `Visible` | R |

**Methods**

- `:ToSolidBlock()`
- `:SetCanCollide([boolean=true])`
- `:IsBlockEntity()`
- `:SetHitbox(number, [number=a.checkdouble(1)])` — custom hitbox (independent of ModelScale)
- `:GetHitbox()`
- `:Hurt(number)`
- `:Kill()`
- `:Heal(number)`
- `:PivotTo(CFrame)` — alias: `Teleport`
- `:Burn([integer=5])`
- `:Stew()` — alias: `Extinguish`
- `:GetEffect()` — alias: `GE`
- `:ClearEffect()` — alias: `CE`
- `:ClearAllEffects()` — alias: `CAE`
- `:SetNBT(string, any, [boolean=false])`
- `:ReadNBT(string)`
- `:isAlive()` — alias: `IsAlive`
- `:ApplyImpulse(Vector3)`
- `:Freeze([number=0], [boolean=false])`
- `:UnFreeze()` — alias: `Unfreeze`
- `:GetModel()` — alias: `GetModelParts`
- `:GetModelPart(string, [boolean=false])` — alias: `GetPart`
- `:SetModel(string, [number=1])`
- `:GetArmor()`
- `:GetMainHand()` — alias: `GMH`
- `:GetSecondaryHand()` — alias: `GSH`
- `:isMHand()`
- `:isSHand()`
- `:DrownStart()`
- `:DrownPause()`
- `:DrownEnd()`
- `:isDrown()`
- `:HideName()`
- `:ShowName()`
- `:isServer()` — alias: `isClassic`
- `:isClient()` — alias: `isHallucination`
- `:GetDistance(any)` — alias: `DistanceTo`

**Events:** `E_ArmorChanged`, `E_AttributeChanged`, `E_ChildAdded`, `E_ChildRemoved`, `E_Damaged`, `E_Death`, `E_EffAdd`, `E_EffDel`, `E_Interact`, `E_Jump`, `E_Move`, `E_Multiply`, `E_Tick`

### Block

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `CFrame` | RW |
| `CanCollide` | RW |
| `CanFall` | RW |
| `Dimension` | R |
| `Hardness` | R |
| `Id` | RW |
| `IsAir` | R |
| `Light` | R |
| `Position` | RW |
| `Screen` | R |
| `State` | RW |

**Methods**

- `:ToBlockEntity([boolean=true])`
- `:Break([boolean=true])`
- `:ReadNBTs()`
- `:ReadNBT(string)`
- `:WriteNBT(string, any)`
- `:SetNBT(string, any)`
- `:Gravity([boolean=true])`
- `:ConnectScreen(string)`
- `:DisconnectScreen()`
- `:MoveTo(Vector3)`

### Item

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Amount` | RW |
| `CFrame` | RW |
| `CanAttack` | RW |
| `CanDamage` | RW |
| `CanDrop` | RW |
| `CanEnchant` | RW |
| `CanRepair` | RW |
| `CanUse` | RW |
| `Count` | RW |
| `CustomData` | RW |
| `Damage` | RW |
| `Description` | RW |
| `Dimension` | RW |
| `Durability` | RW |
| `Empty` | R |
| `Enchantments` | RW |
| `Entity` | R |
| `GUI` | RW |
| `Icon` | RW |
| `Id` | RW |
| `MaxCount` | RW |
| `MaxDamage` | RW |
| `MaxDurability` | RW |
| `MaxStack` | RW |
| `Model` | RW |
| `NBTs` | RW |
| `Owner` | RW |
| `Position` | RW |
| `Rarity` | RW |
| `Slot` | RW |
| `Stackable` | RW |
| `Tags` | RW |
| `Type` | RW |
| `Visible` | RW |

**Methods**

- `:SetNBT(string, any)`
- `:ReadNBT(string)` — alias: `GetNBT`
- `:GetNBTs()`
- `:SetLore()`
- `:Enchant(string, [integer=1])`
- `:GetEnch(string)`
- `:GetEnchLevel(string)`
- `:DelEnch(string)`
- `:Remove([integer=1])`
- `:SetDesc(string)`
- `:GetDesc()`
- `:Damage([integer=1])`
- `:Repair([integer=1])`
- `:Use()`
- `:Attack(any)`
- `:SetGUI(any)`
- `:GetGUI()`
- `:Clone()`
- `:SetCount(integer)`
- `:GetCount()`
- `:SetId(string)`
- `:GetId()`
- `:SetType(string)`
- `:GetType()`
- `:SetStackable(any)`
- `:IsStackable()` — alias: `isStackable`
- `:SetMaxStack(any)`
- `:GetMaxStack()`
- `:SetDurability(any)`
- `:GetDurability()`
- `:SetMaxDurability(any)`
- `:GetMaxDurability()`
- `:SetModel([string=""])`
- `:GetModel()`
- `:SetIcon(any)`
- `:GetIcon()`
- `:SetRarity(string)`
- `:GetRarity()`
- `:SetTags(any)`
- `:GetTags()`
- `:HasTag(string)`
- `:AddTag(string)`
- `:RemoveTag(string)`
- `:SetCustomData(string, any)`
- `:GetCustomData(string)`
- `:Serialize()`
- `:Deserialize(any)`
- `:Compare(any)`

**Events:** `E_Attacked`, `E_AttributeChanged`, `E_ChildAdded`, `E_ChildRemoved`, `E_Cloned`, `E_CountChanged`, `E_Damaged`, `E_DescriptionChanged`, `E_Destroyed`, `E_Dropped`, `E_EnchDel`, `E_Enchant`, `E_GUIChanged`, `E_IconChanged`, `E_ModelChanged`, `E_NBTChanged`, `E_Picked`, `E_RarityChanged`, `E_Removed`, `E_Repaired`, `E_TagAdded`, `E_TagRemoved`, `E_Used`

### Chunk

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `IsCopy` | R |
| `Loaded` | R |
| `MaxY` | R |
| `MinY` | R |
| `Position` | R |
| `Transparency` | R |
| `X` | R |
| `Z` | R |

**Methods**

- `:GetBlocks([boolean=false])`
- `:Copy()`
- `:Paste(any)`
- `:Regenerate([boolean=true])` — Chunk:Regenerate([decorate=true]) - rebuild terrain with the dimension generator (noise + surface [+ trees/ores])
- `:Solid(string)`
- `:BlocksReplace(string, any)`
- `:GetEntities()`
- `:GetDistanceFromOtherChunk(any)`
- `:MoveTo(Vector3)`
- `:GetBlock(integer, integer, integer)`

**Events:** `E_ClientLoaded`

### Structure

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `BlockCount` | R |
| `EntityCount` | R |
| `Id` | R |
| `Pivot` | R |
| `Placed` | R |
| `Position` | R |
| `Size` | R |

**Methods**

- `:MoveTo(Vector3)`
- `:Place(any, Vector3, any)`
- `:Capture(any, Vector3, Vector3, [boolean=true])`
- `:Save([string])`
- `:GetSize()` — ---- v0.5
- `:SetSize(any)`
- `:GetBlocks([boolean=false])`
- `:GetBlock(any)`
- `:SetBlock(any, string, any)`
- `:RemoveBlock(any)`
- `:GetEntities()`
- `:AddEntity(string, Vector3, any)`
- `:ClearEntities()`
- `:Fill(any, any, string)`
- `:GetPivot()`
- `:SetPivot(CFrame)`
- `:Clone()`

### Dimension

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `DayTime` | R |
| `Id` | R |
| `MaxY` | R |
| `MinY` | R |
| `Players` | R |
| `Skybox` | R |
| `Weather` | R |

**Methods**

- `:GetBlock(Vector3)`
- `:SetBlock(string, Vector3)`
- `:GetChunk(Vector3)`
- `:GetSpawn()`
- `:SetSpawn(Vector3, [number=0])`
- `:GetBox(Vector3, Vector3, [boolean=true])`
- `:SetBox(string, Vector3, Vector3, any)`
- `:SetSphere(string, Vector3, number, any)`
- `:GetSphere(Vector3, number, [boolean=false])`
- `:LoadChunk(Vector3, [boolean=false])`
- `:UnforceChunk(Vector3)`
- `:GetEntitiesFromBox(Vector3, Vector3)` — alias: `GEFB`
- `:PlaySound(string, [number=1], [number=1], Vector3)`
- `:Spawn(string, Vector3, [integer=1])`
- `:Raycast(Vector3, Vector3, [number=100], [boolean=false], any)`
- `:Explode(Vector3, [number=4], [boolean=false], [boolean=true])`
- `:Fire(Vector3)`
- `:Broke(Vector3, [boolean=true])` — alias: `Break`
- `:ClearDrops()`
- `:GetPlayers()`
- `:PlaceStructure(any, Vector3, any)`
- `:ChangeWeather(string, [number=300])`

**Events:** `E_BlockDestroyed`, `E_BlockPlaced`, `E_ChunkLoaded`, `E_ChunkUnloaded`, `E_EntityAdded`, `E_EntityDestoryed`, `E_EntityDestroyed`, `E_Explosion`, `E_Tick`, `E_WeatherChanged`

### World

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Count` | R |
| `DayTime` | R |
| `Dimensions` | R |
| `FullTime` | R |
| `GameTime` | R |
| `LocalPlayer` | R |
| `Paused` | R |
| `PlayerAdded` | R |
| `PlayerRemoving` | R |
| `Rate` | R |
| `Seed` | R |
| `Structures` | R |
| `Ticks` | R |
| `Time` | R |

**Methods**

- `:GetPlayerFromName(string)`
- `:GetPlayers()`
- `:GetDimension(string)`
- `:GetStructure(string)`
- `:Copy(string)`

**Events:** `E_Join`, `E_Leave`

### Lighting

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `ClockTime` | RW |
| `Effects` | RW |
| `Lights` | RW |
| `MoonPhase` | RW |
| `SkyDarken` | RW |
| `TimeOfDay` | RW |

**Methods**

- `:SetTimeOfDay(any)`
- `:GetTimeOfDay()`
- `:SetClockTime(number)`
- `:GetClockTime()`
- `:GetMinutesAfterMidnight()`
- `:SetMinutesAfterMidnight(number)`
- `:AddEffect(any)`
- `:RemoveEffect(any)`
- `:ClearEffects()`
- `:GetEffects()`
- `:GetLights()`
- `:Raycast(Vector3, Vector3, [number=1000], any)`

**Events:** `E_AmbientChanged`, `E_BrightnessChanged`, `E_CelestialBodiesVisibilityChanged`, `E_ClockTimeChanged`, `E_EffectAdded`, `E_EffectRemoved`, `E_EnvironmentDiffuseScaleChanged`, `E_EnvironmentSpecularScaleChanged`, `E_ExposureChanged`, `E_FogChanged`, `E_GeographicLatitudeChanged`, `E_GlobalShadowsChanged`, `E_MoonColorChanged`, `E_MoonPositionChanged`, `E_MoonSizeChanged`, `E_MoonVisibilityChanged`, `E_OutdoorAmbientChanged`, `E_ShadowSoftnessChanged`, `E_SkyboxChanged`, `E_StarCountChanged`, `E_SunColorChanged`, `E_SunPositionChanged`, `E_SunSizeChanged`, `E_SunVisibilityChanged`, `E_TechnologyChanged`, `E_TimeChanged`

### AsyncTask

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Done` | R |
| `Error` | R |
| `Ok` | R |
| `Result` | R |
| `Success` | R |

**Methods**

- `:Await()` — alias: `Wait`

### BigNum

**Methods**

- `:ToString()`
- `:ToNumber()`
- `:Abs()`
- `:Pow(any)`
- `:Sqrt()`
- `:Gcd(any)`
- `:ModPow(any, any)`
- `:IsPrime([integer=30])`
- `:Round([integer=0])`
- `:Digits()`
- `:Exp()` — ---- 0.2: high precision math for very small / very precise numbers (BigDec)
- `:Ln()`
- `:Log10()`
- `:Scale()`
- `:Exponent()`
- `:ToSci([integer=20])`
- `:Div(any, [integer=MC.getPrecision()])`
- `:Floor()`
- `:Ceil()`
- `:Neg()`
- `:Sign(any)`

### Binary

**Methods**

- `:Save(string)`
- `:Run(any, [integer=10])`

### BindableEvent

**Methods**

- `:Fire()`

### Browser

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Attributes` | R |
| `Browser` | R |
| `CanGoBack` | R |
| `CanGoForward` | R |
| `Class` | R |
| `ClassName` | R |
| `Cookies` | RW |
| `DevTools` | RW |
| `Engine` | R |
| `FPS` | RW |
| `Focused` | RW |
| `GPU` | RW |
| `HTML` | RW |
| `Headless` | R |
| `Id` | R |
| `Loading` | RW |
| `Position` | RW |
| `Size` | RW |
| `Tag` | R |
| `TagName` | R |
| `Text` | RW |
| `Title` | RW |
| `Transparency` | RW |
| `Transparent` | RW |
| `URL` | RW |
| `Url` | RW |
| `UserAgent` | RW |
| `Value` | RW |
| `Visible` | RW |
| `Zoom` | RW |

**Methods**

- `:Navigate(string)` — alias: `Go`
- `:LoadHTML(string)`
- `:ExecuteJS(string)` — alias: `EJS`
- `:InjectCSS(string)`
- `:SetSize(number, number)`
- `:SetPosition(number, number)`
- `:SetTransparent([boolean=true])`
- `:SetZoom(any)`
- `:Reload()`
- `:Back()`
- `:Forward()`
- `:Stop()`
- `:Close()`
- `:Click(number, number, any)`
- `:Scroll([number=0], [number=0])`
- `:KeyPress(string)`
- `:TypeText(string)`
- `:GetDOM(string)`
- `:QuerySelector(string)`
- `:QuerySelectorAll(string)`
- `:SetCookie(string, string, [string=""], [string="/"])`
- `:GetCookie(string)`
- `:ClearCookies()`
- `:SetUserAgent(string)`
- `:OpenDevTools()`
- `:Print()`
- `:PostMessage(any)`
- `:SetText(string)`
- `:GetText()`
- `:SetHTML(string)`
- `:GetHTML()`
- `:SetStyle(string, string)`
- `:GetAttribute(string)`
- `:SetAttribute(string, any)`
- `:Remove()`
- `:GetParent()`
- `:GetChildren()`

**Events:** `E_AttributeChanged`, `E_ChildAdded`, `E_ChildRemoved`, `E_Console`, `E_Crash`, `E_Download`, `E_Error`, `E_LoadStart`, `E_Loaded`, `E_Message`, `E_Navigate`, `E_Request`, `E_Response`

### CFrame

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `LookVector` | R |
| `Position` | R |
| `RightVector` | R |
| `Rotation` | R |
| `UpVector` | R |
| `X` | R |
| `Y` | R |
| `Z` | R |
| `lookVector` | R |
| `p` | R |

**Methods**

- `:Inverse()`
- `:ToWorldSpace(any)`
- `:ToObjectSpace(any)`
- `:PointToWorldSpace(Vector3)`
- `:PointToObjectSpace(Vector3)`
- `:Lerp(any, number)`
- `:GetComponents()`
- `:ToEulerAnglesXYZ(any)`

### Chromium service

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Available` | R |
| `Engine` | R |
| `HttpEnabled` | R |
| `MCEF` | R |
| `UserAgent` | R |

**Methods**

- `:Request(table)`
- `:RequestAsync(table)`
- `:Get(string, any)`
- `:GetAsync(string, any)`
- `:Post(string, string, [string="application/json"], any)`
- `:PostAsync(string, string, [string="application/json"], any)`
- `:JSONEncode(any)`
- `:JSONDecode(string)`
- `:HtmlToText(string)`
- `:UrlEncode(string)`
- `:Open(string)`
- `:ShowPage(string)`
- `:CreateBrowser(any, any)` — alias: `CB` — ---- 0.3: Browser objects (lite engine; rendered by the client inside UI / ScreenGUI / Overlay)
- `:CreateHeadless(any)`
- `:LoadURL(string, any)`
- `:LoadHTML(string, any)`
- `:ShowText(string, string)`

### Client

**Methods**

- `:Send(any, [string=null])` — NetClient
- `:Close()` — alias: `Disconnect`
- `:GetPing()`
- `:SetTimeout(number)`

### Global events (client)

**Events:** `E_Move`, `E_Tick`

### ClientObjects

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Button1Down` | R |
| `Button1Up` | R |
| `Button2Down` | R |
| `Button2Up` | R |
| `CFrame` | R |
| `FieldOfView` | RW |
| `FreeCam` | RW |
| `Hit` | R |
| `LookVector` | R |
| `Position` | R |
| `Target` | R |
| `ThirdPerson` | RW |
| `ViewSizeX` | R |
| `ViewSizeY` | R |
| `WheelBackward` | R |
| `WheelForward` | R |
| `X` | R |
| `Y` | R |

### Color

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `B` | R |
| `G` | R |
| `Hex` | R |
| `R` | R |

### Connection

**Methods**

- `:Disconnect()` — alias: `disconnect`

### Corner

**Methods**

- `:SetCornerRadius(any)`

### FASM service

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Available` | R |
| `Bytes` | R |
| `Data` | R |
| `Hex` | R |
| `Installed` | R |
| `Listing` | R |
| `Output` | R |
| `Path` | R |
| `Size` | R |

**Methods**

- `:Assemble(string)`
- `:AssembleAsync(string)`

### FluwaUiScreen

**Events:** `E_Closed`

### ForgeCapability

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Class` | R |
| `Holder` | R |
| `ModId` | R |
| `Provider` | R |
| `Side` | R |
| `Type` | R |

**Methods**

- `:Get(any)`
- `:Attach(any, any)`
- `:Detach(any)`
- `:Has(any)`
- `:GetType()`
- `:GetName()`
- `:Serialize()`
- `:Deserialize(table)`

### ForgeClient

**Events:** `E_Pressed`, `E_Released`, `E_Reload`, `E_ResourcePackChanged`

### ForgeCommand

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Aliases` | R |
| `Description` | R |
| `ModId` | R |
| `Name` | R |
| `Permission` | R |
| `Registered` | R |

**Methods**

- `:Execute(any, [string=""])`
- `:GetName()`
- `:GetPermission()`
- `:SetPermission(any)`
- `:GetAliases()`
- `:AddAlias(string)`
- `:RemoveAlias(string)`
- `:Unregister()`

**Events:** `E_Error`, `E_Execute`

### ForgeConfig

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `ConfigType` | R |
| `Dirty` | R |
| `File` | R |
| `Loaded` | R |
| `ModId` | R |
| `Path` | R |
| `Section` | R |
| `Sections` | R |
| `Values` | R |

**Methods**

- `:Get(string, any)`
- `:Set(string, any)`
- `:GetBool(string, [boolean=false])`
- `:GetInt(string, any)`
- `:GetFloat(string, any)`
- `:GetString(string, any)`
- `:GetList(string)`
- `:GetEnum(string, any)`
- `:Has(string)`
- `:Remove(string)`
- `:Save()`
- `:Load()`
- `:Reload()`
- `:Reset()`
- `:GetSections()`
- `:GetSection(string)`
- `:AddSection(string)`
- `:RemoveSection(string)`
- `:GetKeys()`
- `:GetComment(string)`
- `:SetComment(string, string)`

**Events:** `E_Changed`, `E_Error`, `E_Loaded`, `E_Reloaded`, `E_Reset`, `E_Saved`

### ForgeEvent

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Cancelable` | R |
| `Canceled` | RW |
| `Cancelled` | RW |
| `Class` | R |
| `EventType` | R |
| `Fields` | R |
| `HasResult` | R |
| `Name` | R |
| `Parent` | R |
| `Phase` | R |
| `Result` | RW |
| `Type` | R |

**Methods**

- `:Cancel()`
- `:SetCanceled([boolean=true])` — alias: `SetCancelled`
- `:IsCanceled()` — alias: `IsCancelled`
- `:IsCancelable()`
- `:SetResult(any)`
- `:GetResult()`
- `:GetType()`
- `:Get(string)`
- `:Set(string, any)`
- `:Call(string)`
- `:Java()` — alias: `GetJava`

### ForgeEventBus

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Listeners` | R |
| `Parent` | R |
| `Priorities` | R |
| `Type` | R |

**Methods**

- `:Subscribe(string, any, any)` — alias: `On`, `Connect`
- `:Unsubscribe(string, any)` — alias: `Off`
- `:Post(any)` — alias: `Fire`
- `:Register(table)`
- `:Unregister(any)`
- `:GetListeners(string)`
- `:Clear()`
- `:HasListeners(string)`
- `:SetPriority(string, any)`
- `:GetPriority(string)`
- `:IsRealEvent(string)`

**Events:** `E_EventCancelled`, `E_EventPosted`, `E_ListenerAdded`, `E_ListenerRemoved`

### ForgeMod

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Authors` | R |
| `Config` | R |
| `Container` | R |
| `Credits` | R |
| `Dependants` | R |
| `Dependencies` | R |
| `Description` | R |
| `Dist` | R |
| `Enabled` | R |
| `EventBus` | R |
| `File` | R |
| `Id` | R |
| `License` | R |
| `Loaded` | R |
| `Logo` | R |
| `Metadata` | R |
| `Path` | R |
| `Registry` | R |
| `Script` | R |
| `Side` | R |
| `State` | R |
| `Version` | R |

**Methods**

- `:GetId()`
- `:GetName()`
- `:GetVersion()`
- `:GetDescription()`
- `:GetAuthors()`
- `:GetCredits()`
- `:GetLicense()`
- `:GetLogoFile()`
- `:GetModFile()`
- `:GetMetadata()`
- `:IsLoaded()`
- `:IsEnabled()`
- `:GetContainer()`
- `:GetEventBus()`
- `:GetConfig()`
- `:GetRegistry(string)`
- `:Register(string, any)`
- `:RegisterRecipe(string, any)`
- `:RegisterCapability(string, any)`
- `:RegisterCommand(table)`
- `:RegisterKeyBinding(string, any)`
- `:RegisterConfig(any)`
- `:RegisterEventHandler(string, any, any)`
- `:Unregister(string)`
- `:Enable()`
- `:Disable()`
- `:Reload()`
- `:Unload()`
- `:GetState()`

**Events:** `E_ConfigChanged`, `E_Disable`, `E_Enable`, `E_Error`, `E_Load`, `E_ModDisabled`, `E_ModEnabled`, `E_ModUnloaded`, `E_Register`, `E_Reload`, `E_Unload`

### ForgeRecipe

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Category` | R |
| `Group` | R |
| `Id` | R |
| `Input` | R |
| `Output` | R |
| `Script` | R |
| `Serializer` | R |
| `Special` | R |
| `Type` | R |

**Methods**

- `:GetType()`
- `:GetId()`
- `:GetOutput()`
- `:GetInput()`
- `:Matches(any)`
- `:Craft(any)`
- `:GetSerializer()`
- `:GetCategory()`
- `:GetGroup()`

### ForgeRegistry

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Entries` | R |
| `Frozen` | R |
| `Id` | R |
| `Key` | R |
| `Namespace` | R |
| `Size` | R |
| `Tags` | R |
| `Type` | R |

**Methods**

- `:Register(string, any)`
- `:Unregister(string)`
- `:Get(string)`
- `:GetAll()`
- `:GetIds()` — alias: `GetKeys`
- `:Contains(string)`
- `:GetName(any)` — alias: `GetId`
- `:GetKey(any)`
- `:GetValue(string)`
- `:GetEntries()`
- `:GetTags()`
- `:GetTag(string)`
- `:GetTagObject(string)`
- `:AddTag(string, string)`
- `:RemoveTag(string, string)`
- `:HasTag(string, string)`
- `:Clear()`
- `:Freeze()`
- `:IsFrozen()`
- `:IsBuiltin()`
- `:GetType()`

**Events:** `E_Clear`, `E_Freeze`, `E_Register`, `E_TagAdded`, `E_TagRemoved`, `E_Unregister`

### ForgeService

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Client` | R |
| `ConfigDir` | R |
| `DataPacks` | R |
| `Dist` | R |
| `EventBus` | R |
| `FMLVersion` | R |
| `ForgeVersion` | R |
| `GameDir` | R |
| `Loaded` | R |
| `Loader` | R |
| `LogDir` | R |
| `MinecraftEventBus` | R |
| `MinecraftVersion` | R |
| `ModList` | R |
| `Mods` | R |
| `ModsDir` | R |
| `RecipeManager` | R |
| `RegistryManager` | R |
| `ResourcePackDir` | R |
| `ResourcePacks` | R |
| `ScriptMods` | R |
| `Server` | R |
| `Side` | R |
| `Tags` | R |
| `Unlocked` | R |
| `Version` | R |
| `WorldDir` | R |

**Methods**

- `:GetMods()`
- `:GetClass(string)` — alias: `JavaClass` — v0.5 deep reflection
- `:Java(any)` — alias: `Wrap`
- `:IsUnlocked()`
- `:GetStatic(string, any)`
- `:GetMinecraftServer()` — alias: `JavaServer`
- `:GetMinecraftClient()` — alias: `JavaClient`
- `:GetMod(string)`
- `:IsModLoaded(string)`
- `:GetModContainer(string)`
- `:GetModMetadata(string)`
- `:GetModFile(string)`
- `:GetEventBus()` — alias: `GetMinecraftEventBus`
- `:GetModEventBus(string)`
- `:GetRegistry(string)`
- `:GetConfig(string, string)`
- `:RegisterMod(table)`
- `:Register(string, string, any)`
- `:RegisterRecipe(string, string, any)`
- `:RegisterCapability(string, string, any)`
- `:GetCapability(string)`
- `:RegisterCommand(string, table)`
- `:RegisterKeyBinding(string, string, any)`
- `:RegisterResourcePack(string)`
- `:RegisterDataPack(string)`
- `:GetSide()`
- `:GetDist()`
- `:GetMinecraftVersion()`
- `:GetForgeVersion()`
- `:GetFMLVersion()`
- `:GetLoader()`
- `:GetGameDir()`
- `:GetConfigDir()`
- `:GetModsDir()`
- `:GetWorldDir()`
- `:GetLogDir()`
- `:GetResourcePackDir()`
- `:GetServer()`
- `:GetClient()`
- `:Reload()`
- `:GetTags(string)`
- `:GetTag(string, string)`
- `:CreateTag(string, string, any)`
- `:GetRecipeManager()`
- `:GetRegistryManager()`
- `:GetDataPackList()`
- `:GetResourcePackList()`
- `:GetModList()` — alias: `GetModIds`
- `:GetLoadedMods()`
- `:GetModNames()`
- `:GetModVersions()`

**Events:** `E_ClientStarted`, `E_ClientStopping`, `E_ConfigChanged`, `E_DataPackChanged`, `E_Error`, `E_ModDisabled`, `E_ModEnabled`, `E_ModLoaded`, `E_ModUnloaded`, `E_Register`, `E_Reload`, `E_ResourcePackChanged`, `E_Server`, `E_ServerStarted`, `E_ServerStopping`

### ForgeTag

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Entries` | R |
| `Id` | R |
| `Registry` | R |
| `Size` | R |

**Methods**

- `:GetId()`
- `:GetEntries()`
- `:Contains(string)`
- `:Add(string)`
- `:Remove(string)`
- `:Clear()`

**Events:** `E_Added`, `E_Cleared`, `E_Removed`

### Fx

**Events:** `E_CelestialBodiesVisibilityChanged`, `E_EffectAdded`, `E_EffectRemoved`, `E_MoonAngularSizeChanged`, `E_MoonTextureChanged`, `E_MoonVisibilityChanged`, `E_SkyboxChanged`, `E_StarCountChanged`, `E_StarTextureChanged`, `E_SunAngularSizeChanged`, `E_SunTextureChanged`, `E_SunVisibilityChanged`

### Game

**Methods**

- `:GetService(string)`

### Gradient

**Methods**

- `:SetColor(any)`
- `:SetTransparency(any)`
- `:SetRotation(any)`
- `:SetOffset(any)`

### GridLayout

**Methods**

- `:SetCellSize(any)`
- `:SetCellPadding(any)`
- `:SetFillDirection(any)`
- `:SetFillDirectionMaxCells(any)`

### Info

**Methods**

- `:ToTable()`

### InputObject

**Methods**

- `:GetKeyCode()`
- `:GetButton()`
- `:GetPosition()`
- `:GetDelta()`
- `:GetUserInputType()`
- `:GetUserInputState()`
- `:GetGamepadNum()`
- `:IsGamepadButtonDown(any)`
- `:IsKeyDown(any)`
- `:IsMouseButtonDown(any)`

### Item events

**Events:** `E_Attacked`, `E_Damaged`, `E_Dropped`, `E_Picked`, `E_Repaired`, `E_Used`

### Java

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Class` | R |
| `ClassName` | R |
| `IsClass` | R |
| `Name` | R |
| `Parent` | R |

**Methods**

- `:Get(string)`
- `:GetRaw(string)`
- `:Set(string, any)`
- `:Call(string)`
- `:CallRaw(string)`
- `:New()`
- `:Unwrap()` — alias: `ToLua`
- `:GetClass()`
- `:Super()`
- `:IsInstance(string)` — alias: `InstanceOf`
- `:Cast(string)`
- `:GetFields()`
- `:GetMethods([string=""])`
- `:GetConstructors()`
- `:IsNull()`
- `:Equals(any)`
- `:ToString()`
- `:HashCode()`

### Light

**Methods**

- `:AttachTo(any, string)`
- `:Detach()`

**Events:** `E_AngleChanged`, `E_Attached`, `E_BrightnessChanged`, `E_ColorChanged`, `E_ContrastChanged`, `E_DecayChanged`, `E_Detached`, `E_EnabledChanged`, `E_FaceChanged`, `E_FalloffChanged`, `E_InnerAngleChanged`, `E_OuterAngleChanged`, `E_RangeChanged`, `E_ShadowsChanged`, `E_TextureChanged`, `E_TransparencyChanged`

### ListLayout

**Methods**

- `:SetPadding(any)`
- `:SetFillDirection(any)`
- `:SetHorizontalAlignment(any)`
- `:SetVerticalAlignment(any)`
- `:SetSortOrder(any)`

### LuaRT

**Methods**

- `:Run(string, [integer=10])`
- `:RunFile(string, [integer=10])`

### ModelPart

**Methods**

- `:Hide()`
- `:Show()`
- `:SetTransform(Vector3, Vector3, Vector3)` — SetTransform(offset: Vector3, rotation: Vector3 (degrees), scale: Vector3 \| number) - nil keeps the current value
- `:ResetTransform()` — alias: `Reset`
- `:GetTransform()`

### NET service

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Body` | R |
| `Clients` | R |
| `Data` | R |
| `Headers` | R |
| `Host` | R |
| `IsConnected` | R |
| `IsOpen` | R |
| `IsRunning` | R |
| `LocalIP` | R |
| `MaxClients` | RW |
| `Ping` | R |
| `Port` | R |
| `Position` | R |
| `Protocol` | R |
| `Receiver` | R |
| `RemoteIP` | R |
| `Sender` | R |
| `Size` | R |
| `Status` | R |
| `StatusCode` | R |
| `Success` | R |
| `Time` | R |
| `Timeout` | RW |
| `Timestamp` | R |
| `URL` | R |

**Methods**

- `:CreateServer(integer, [string="TCP"])` — alias: `CS`, `TCPSocket`
- `:UDPSocket(integer)`
- `:Connect(string, integer, [string="TCP"])` — alias: `Conn`
- `:HTTP(string, string, any, any)` — alias: `Request`
- `:WebSocket(string)` — alias: `WS`
- `:JSONEncode(any)` — alias: `JE`
- `:JSONDecode(string)` — alias: `JD`
- `:Hash(string, any)`
- `:Base64Encode(any)`
- `:Base64Decode(string)`
- `:Ping(string, [integer=80])`
- `:ResolveDNS(string)`
- `:GetLocalIP()`
- `:GetPublicIP()`
- `:SetTimeout(number)`
- `:GetTimeout()`
- `:CloseAll()`
- `:NewPacket()` — alias: `Packet`

**Events:** `E_ClientConnect`, `E_ClientDisconnect`, `E_Close`, `E_Connect`, `E_Disconnect`, `E_Error`, `E_Message`, `E_Open`, `E_Packet`

### Packet

**Methods**

- `:Read([string="string"])`
- `:Write(any, [string=null])`
- `:GetSize()`
- `:Clear()`
- `:Reset()`

### Players

**Methods**

- `:GetPlayers()`
- `:GetPlayerFromName(string)`

### Projector

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Anchored` | RW |
| `Animation` | R |
| `AnimationTime` | R |
| `Animations` | R |
| `Bones` | R |
| `CFrame` | RW |
| `CanCollide` | RW |
| `Children` | R |
| `Class` | RW |
| `Collision` | RW |
| `Dimension` | R |
| `Format` | R |
| `GUI` | R |
| `LOD` | R |
| `Loaded` | R |
| `Material` | RW |
| `Meshes` | R |
| `NBTs` | R |
| `Playing` | R |
| `Polygons` | R |
| `Position` | RW |
| `Scale` | RW |
| `Size` | R |
| `Source` | R |
| `Tags` | RW |
| `Texture` | RW |
| `Transparency` | RW |
| `Vertices` | R |
| `Visible` | RW |
| `WorldCFrame` | R |

**Methods**

- `:SetCFrame(any)` — alias: `PivotTo`
- `:GetCFrame()`
- `:SetPosition(any)`
- `:GetPosition()`
- `:SetScale(any)`
- `:GetScale()`
- `:SetVisible(any)`
- `:SetTransparency(any)`
- `:SetMaterial(any)`
- `:SetTexture(string)`
- `:GetBone(string)`
- `:SetBone(string, CFrame)`
- `:PlayAnimation([string=null], [number=1], [boolean=true])`
- `:StopAnimation()`
- `:PauseAnimation()`
- `:Export(string, [string=null])`
- `:Raycast(Vector3, Vector3, [number=512])`
- `:SetNBT(string, any)`
- `:GetNBT(string)` — alias: `ReadNBT`
- `:GetNBTs()`
- `:Use(any)`
- `:Attack(any)`
- `:Interact(any)`
- `:SetGUI(string)`
- `:GetGUI()`
- `:HasTag(string)`
- `:AddTag(string)`
- `:RemoveTag(string)`
- `:Inspect()`

**Events:** `E_AnimationEnd`, `E_AnimationKey`, `E_AnimationStart`, `E_Attacked`, `E_BoneMoved`, `E_ChildAdded`, `E_ChildRemoved`, `E_Clicked`, `E_Error`, `E_Exported`, `E_Imported`, `E_Interacted`, `E_Loaded`, `E_MaterialChanged`, `E_ParentChanged`, `E_TextureChanged`, `E_TouchEnded`, `E_Touched`, `E_Used`, `E_VertexChanged`

### Projector service

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `CFrame` | R |
| `Color` | RW |
| `Object` | R |
| `ParentBone` | R |
| `Position` | R |
| `Scale` | R |
| `Texture` | RW |
| `Transparency` | RW |
| `Vertices` | R |
| `Visible` | RW |

**Methods**

- `:Import(string, any)` — alias: `Imp`
- `:Load(string, any)`
- `:Export(any, string, [string=null])` — alias: `Exp`
- `:Create([string="Part"], any, any, any, any, any, any, any, any, any, any, any, any, any, any, [integer])`
- `:Preload(string)`
- `:GetCache()`
- `:ClearCache()`
- `:Raycast(any, Vector3, Vector3, [number=512])`
- `:Attach(any, Every object (universal), CFrame)`
- `:Detach(any)`
- `:Clone(any)`
- `:Merge(any, any)`
- `:Optimize(any)`
- `:SetLOD(any, integer)`
- `:GetPolygons(any)`
- `:GetVertices(any)`
- `:GetVertex(any, integer)`
- `:SetVertex(any, integer, Vector3)`
- `:SetMaterial(any, any)`
- `:NewMaterial(any)` — alias: `CreateMaterial`
- `:SetTexture(any, string)`
- `:SetBone(any, string, CFrame)`
- `:GetBone(any, string)`
- `:PlayAnimation(any, [string=null], [number=1], [boolean=true])`
- `:StopAnimation(any)`
- `:PauseAnimation(any)`
- `:SetAnimationTime(any, number)`
- `:GetAnimationTime(any)`
- `:SetCFrame(any, any)` — alias: `PivotTo`
- `:SetScale(any, any)`
- `:SetVisible(any, any)`
- `:SetTransparency(any, any)`
- `:SetCollision(any)`
- `:Convert(any, string)`
- `:Inspect(any)`
- `:GetObjects()` — alias: `GetAll`

**Events:** `E_AnimationEnd`, `E_BoneMoved`, `E_Clicked`, `E_Error`, `E_Exported`, `E_Imported`, `E_Loaded`, `E_MaterialChanged`, `E_TextureChanged`, `E_TouchEnded`, `E_Touched`, `E_VertexChanged`

### PropBag

**Events:** `E_Changed`

### RayResult

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Block` | R |
| `Distance` | R |
| `Entity` | R |
| `Hit` | R |
| `Instance` | R |
| `Normal` | R |
| `Position` | R |

### RemoteEvent

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `OnClientEvent` | R |
| `OnMeFire` | R |
| `OnPlayerFire` | R |
| `OnServerEvent` | R |
| `OnServerFire` | R |

**Methods**

- `:FireServer(any)` — alias: `FS`
- `:FirePlayer(any, any)` — alias: `FP`, `FireClient`
- `:FireAllPlayers(any)` — alias: `FAP`, `FireAllClients`
- `:FireAll(any, number, Vector3, Every object (universal))` — alias: `FA`

### Response

**Methods**

- `:JSON()`
- `:Text()`
- `:Binary()`
- `:GetHeader(string)`

### RuntimeInfo

**Methods**

- `:CollectGarbage()`

### ScreenGui

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `ClickThrough` | R |
| `CloseOnEsc` | R |
| `Closed` | R |
| `DisplayOrder` | R |
| `Enabled` | R |
| `IgnoreGuiInset` | R |
| `Modal` | R |
| `Opened` | R |
| `PauseGame` | R |
| `ResetOnSpawn` | R |
| `Transparency` | R |

**Methods**

- `:Open()`
- `:Close()`
- `:Toggle()`
- `:SetDisplayOrder(any)`
- `:GetDisplayOrder()`
- `:SetIgnoreGuiInset(any)`
- `:GetIgnoreGuiInset()`
- `:SetResetOnSpawn(any)`
- `:SetEnabled(any)`
- `:SetClickThrough(any)`

**Events:** `E_Closed`, `E_Opened`

### Script

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Path` | R |
| `RunContext` | R |
| `Section` | R |
| `Source` | R |

### Scroll

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `AbsoluteCanvasSize` | R |
| `AbsoluteWindowSize` | R |
| `AutomaticCanvasSize` | R |
| `BottomImage` | R |
| `CanvasPosition` | R |
| `CanvasSize` | R |
| `ElasticBehavior` | R |
| `HorizontalScrollBarInset` | R |
| `MidImage` | R |
| `ScrollBarImage` | R |
| `ScrollBarImageColor3` | R |
| `ScrollBarImageTransparency` | R |
| `ScrollBarThickness` | R |
| `ScrollingEnabled` | R |
| `TopImage` | R |
| `VerticalScrollBarInset` | R |

**Methods**

- `:SetCanvasSize(any)`
- `:SetScrollBarThickness(any)`
- `:SetScrollBarImage(any)`
- `:SetScrollingEnabled(any)`
- `:ScrollToTop()`
- `:ScrollToBottom()`

**Events:** `E_CanvasPositionChanged`

### Server

**Methods**

- `:Start()` — NetServer
- `:Stop()`
- `:Broadcast(any, [string=null])`
- `:SendTo(any, any, [string=null])`
- `:Kick(any, string)`
- `:GetClients()`

### Global events (server)

**Events:** `E_ArmorChanged`, `E_Attack`, `E_BlockDestroyed`, `E_BlockPlaced`, `E_Chatted`, `E_ChunkChange`, `E_ChunkLoaded`, `E_ChunkUnloaded`, `E_Damaged`, `E_Death`, `E_DimensionChanged`, `E_DropItem`, `E_EffAdd`, `E_EffDel`, `E_EntityAdded`, `E_EntityDestroyed`, `E_Explosion`, `E_Feed`, `E_Interact`, `E_Join`, `E_Jump`, `E_Leave`, `E_Move`, `E_Multiply`, `E_PickItem`, `E_Respawn`, `E_Sleep`, `E_Tick`, `E_Wake`, `E_WeatherChanged`

### Services (BigInt / BigDec / OS / LuaRT)

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Available` | R |
| `E` | R |
| `Factorial` | R |
| `Installed` | R |
| `MemoryMax` | R |
| `MemoryUsed` | R |
| `Path` | R |
| `Pi` | R |
| `SetPrecision` | R |
| `Side` | R |
| `Ticks` | R |
| `Timeout` | R |
| `Uptime` | R |
| `Version` | R |

### Shader

**Methods**

- `:Enable()`
- `:Disable()`
- `:SetUniform(string, any)`

### ShaderDT

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Available` | R |
| `CurrentPreset` | R |
| `GLVersion` | R |
| `Presets` | R |
| `Renderer` | R |

**Methods**

- `:Create(string, [boolean=true])`
- `:Compile(string)`
- `:Preset(string)`
- `:ClearPreset()`
- `:DisableAll()`

### Signal

**Methods**

- `:Connect()` — alias: `connect`
- `:Once()`
- `:ConnectSync()` — alias: `ConnectFast` — fastest: runs directly on the game thread, wait() is not allowed inside (for per-frame / per-tick handlers)
- `:DisconnectAll()`

### Skybox

**Methods**

- `:ServerChange([string=""])`
- `:ClientChange([string=""], string)`
- `:Reset()`

### Stroke

**Methods**

- `:SetColor(any)`
- `:SetThickness(any)`
- `:SetTransparency(any)`
- `:SetApplyStrokeMode(any)`

### Terrain

**Methods**

- `:Fill(string, Vector3, Vector3, any, any)`
- `:FillSphere(string, Vector3, number, any, any)`
- `:Clear(Vector3, Vector3, any)`
- `:Generate(Vector3, Vector3, [integer], [integer], any, any, any, any, any, any, any, [number], any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, [number], any, any, any, any, any, any, any, any, any, any, any, any, any, any, any, [integer])`

### TextBox

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `ClearTextOnFocus` | R |
| `CursorPosition` | R |
| `Focused` | R |
| `MultiLine` | R |
| `PlaceholderColor3` | R |
| `PlaceholderText` | R |
| `TextEditable` | R |

**Methods**

- `:CaptureFocus()`
- `:ReleaseFocus([boolean=false])`
- `:IsFocused()`
- `:SetPlaceholderText(any)`
- `:SetClearTextOnFocus(any)`
- `:SetMultiLine(any)`

**Events:** `E_Changed`, `E_FocusLost`, `E_Focused`, `E_ReturnPressed`, `E_TextChanged`

### UDim

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Offset` | R |
| `Scale` | R |

### UDim2

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `X` | R |
| `XOffset` | R |
| `XScale` | R |
| `Y` | R |
| `YOffset` | R |
| `YScale` | R |

### Ui

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `AbsolutePosition` | R |
| `AbsoluteRotation` | R |
| `AbsoluteSize` | R |
| `Active` | R |
| `AnchorPoint` | R |
| `AutoButtonColor` | R |
| `AutomaticSize` | R |
| `BackgroundColor` | R |
| `BackgroundColor3` | R |
| `BackgroundTransparency` | R |
| `BorderColor` | R |
| `BorderColor3` | R |
| `BorderSize` | R |
| `BorderSizePixel` | R |
| `ClipsDescendants` | R |
| `ContentText` | R |
| `CurrentFrame` | R |
| `FPS` | R |
| `Font` | R |
| `Frames` | R |
| `GroupColor3` | R |
| `GroupTransparency` | R |
| `GuiState` | R |
| `Hovered` | R |
| `Image` | R |
| `ImageColor` | R |
| `ImageColor3` | R |
| `ImageRectOffset` | R |
| `ImageRectSize` | R |
| `ImageTransparency` | R |
| `LayoutOrder` | R |
| `Loop` | R |
| `Looped` | R |
| `Modal` | R |
| `Playing` | R |
| `Position` | R |
| `RichText` | R |
| `Rotation` | R |
| `ScaleType` | R |
| `Selectable` | R |
| `Selected` | R |
| `Size` | R |
| `SizeConstraint` | R |
| `SliceCenter` | R |
| `SliceScale` | R |
| `Style` | R |
| `Text` | R |
| `TextBounds` | R |
| `TextCentered` | R |
| `TextColor` | R |
| `TextColor3` | R |
| `TextFits` | R |
| `TextScale` | R |
| `TextScaled` | R |
| `TextSize` | R |
| `TextStrokeColor3` | R |
| `TextStrokeTransparency` | R |
| `TextTransparency` | R |
| `TextWrapped` | R |
| `TextXAlignment` | R |
| `TextYAlignment` | R |
| `TileSize` | R |
| `Visible` | R |
| `ZIndex` | R |

**Methods**

- `:TweenPosition(UDim2)` — alias: `TP` — tweens (Roblox signature): TweenPosition(endPos, [dir], [style], [time], [override], [callback])
- `:TweenSize(UDim2)` — alias: `TS`
- `:TweenSizeAndPosition(UDim2, UDim2)` — alias: `TSP`
- `:GetPropertyChangedSignal(string)`
- `:SetBackgroundColor(any)` — alias: `SetBackgroundColor3` — Frame
- `:SetBackgroundTransparency(any)`
- `:SetBorderSizePixel(any)`
- `:SetBorderColor(any)` — alias: `SetBorderColor3`
- `:SetVisible(any)`
- `:SetPosition(any)`
- `:SetSize(any)`
- `:Click()` — Button
- `:SetAutoButtonColor(any)`
- `:SetModal(any)`
- `:SetSelected(any)`
- `:SetImage(any)` — Image
- `:SetImageColor3(any)` — alias: `SetImageColor`
- `:SetImageTransparency(any)`
- `:Play()` — AnimImage
- `:Pause()`
- `:Stop()`
- `:SetFrame(integer)`
- `:SetFPS(any)`
- `:SetLoop(any)`
- `:SetText(any)` — TextLabel
- `:GetText()`
- `:SetFont(any)`
- `:SetTextSize(any)`
- `:SetTextColor(any)` — alias: `SetTextColor3`
- `:SetTextScaled(any)`
- `:SetTextWrapped(any)`
- `:SetTextXAlignment(any)`
- `:SetTextYAlignment(any)`
- `:SetGroupTransparency(any)` — CanvasGroup
- `:SetGroupColor3(any)` — alias: `SetGroupColor`

**Events:** `E_Activated`, `E_AttributeChanged`, `E_CanvasPositionChanged`, `E_Changed`, `E_ChildAdded`, `E_ChildRemoved`, `E_Click`, `E_Closed`, `E_Deactivated`, `E_Down`, `E_Ended`, `E_Error`, `E_Finished`, `E_FocusLost`, `E_Focused`, `E_FrameChanged`, `E_InputBegan`, `E_InputChanged`, `E_InputEnded`, `E_Loaded`, `E_Looped`, `E_MouseEnter`, `E_MouseLeave`, `E_MouseMoved`, `E_MouseWheelBackward`, `E_MouseWheelForward`, `E_Opened`, `E_Pause`, `E_Play`, `E_ReturnPressed`, `E_SelectionGained`, `E_SelectionLost`, `E_TextChanged`, `E_TouchLongPress`, `E_TouchPan`, `E_TouchSwipe`, `E_TouchTap`, `E_Up`

### UiInput

**Events:** `E_SelectionGained`, `E_SelectionLost`

### UiMods

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `AbsoluteCellCount` | R |
| `AbsoluteContentSize` | R |
| `ApplyStrokeMode` | R |
| `CellPadding` | R |
| `CellSize` | R |
| `Color` | R |
| `Enabled` | R |
| `FillDirection` | R |
| `FillDirectionMaxCells` | R |
| `HorizontalAlignment` | R |
| `LineJoinMode` | R |
| `Offset` | R |
| `Padding` | R |
| `PaddingBottom` | R |
| `PaddingLeft` | R |
| `PaddingRight` | R |
| `PaddingTop` | R |
| `Rotation` | R |
| `SortOrder` | R |
| `StartCorner` | R |
| `Thickness` | R |
| `Transparency` | R |
| `VerticalAlignment` | R |

### UserInputService

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `AccelerometerEnabled` | R |
| `Button` | R |
| `ConnectedGamepads` | R |
| `CursorPosition` | RW |
| `Delta` | R |
| `DeviceType` | R |
| `FocusedTextBox` | R |
| `GamepadConnected` | R |
| `GamepadEnabled` | R |
| `GamepadNum` | R |
| `GyroscopeEnabled` | R |
| `InputType` | R |
| `KeyCode` | R |
| `KeyboardEnabled` | R |
| `MouseBehavior` | RW |
| `MouseDeltaSensitivity` | RW |
| `MouseEnabled` | R |
| `MouseIcon` | RW |
| `MouseIconEnabled` | RW |
| `MouseSensitivity` | RW |
| `MouseWheelDelta` | R |
| `OverrideMouseIcon` | RW |
| `Platform` | R |
| `Position` | R |
| `TouchCount` | R |
| `TouchEnabled` | R |
| `UserInputState` | R |
| `UserInputType` | R |
| `VREnabled` | R |
| `Velocity` | R |
| `WindowFocused` | R |
| `WindowFullscreen` | RW |
| `WindowMinimized` | R |
| `WindowPosition` | R |
| `WindowSize` | R |

**Methods**

- `:IsKeyDown(any)`
- `:IsMouseButtonPressed(any)` — alias: `IsMouseButtonDown`
- `:IsGamepadButtonDown(any, any)`
- `:IsGamepadEnabled()`
- `:IsTouchEnabled()`
- `:IsKeyboardEnabled()`
- `:IsMouseEnabled()`
- `:GetMouseLocation()`
- `:GetMouseDelta()`
- `:GetKeysPressed()`
- `:GetMouseButtonsPressed()`
- `:GetGamepadState(any)`
- `:GetConnectedGamepads()`
- `:GetFocusedTextBox()`
- `:SetMouseBehavior(string)`
- `:GetMouseBehavior()`
- `:SetMouseIconEnabled(any)`
- `:IsMouseIconEnabled()`
- `:SetMouseIcon()`
- `:GetMouseIcon()`
- `:SetTouchEnabled(any)`
- `:SetKeyboardEnabled(any)`
- `:SetGamepadEnabled(any)`
- `:SetMouseEnabled(any)`
- `:SetOverrideMouseIcon(any)`
- `:GetPlatform()`
- `:GetDeviceType()`
- `:GetInputType()` — alias: `GetLastInputType`
- `:VibrateGamepad()` — GLFW has no rumble API: accepted, no effect
- `:VibratePhone()`
- `:GetGamepadConnected(any)`
- `:GetGamepadName(any)`
- `:GetGamepadButtonState(any, any)`
- `:GetGamepadAxis(any, any)`
- `:GetMouseWheelDelta()`
- `:GetMouseSensitivity()`
- `:SetMouseSensitivity(number)`
- `:GetCursorPosition()`
- `:SetCursorPosition(any)`
- `:GetKeyCodeFromName(any)`
- `:GetKeyNameFromCode(any)`
- `:GetButtonNameFromCode(integer)`
- `:GetButtonCodeFromName(any)`

**Events:** `E_AccelerometerChanged`, `E_AttributeChanged`, `E_ChildAdded`, `E_ChildRemoved`, `E_DeviceMotion`, `E_GamepadAxisChanged`, `E_GamepadButtonDown`, `E_GamepadButtonUp`, `E_GamepadConnected`, `E_GamepadDisconnected`, `E_GyroscopeChanged`, `E_InputBegan`, `E_InputBlurred`, `E_InputChanged`, `E_InputEnded`, `E_InputFocused`, `E_KeyboardKeyDown`, `E_KeyboardKeyRepeat`, `E_KeyboardKeyUp`, `E_LastInputTypeChanged`, `E_MouseEnter`, `E_MouseLeave`, `E_MouseMoved`, `E_MouseWheel`, `E_TextBoxFocusReleased`, `E_TextBoxFocused`, `E_TouchEnded`, `E_TouchLongPress`, `E_TouchMoved`, `E_TouchPan`, `E_TouchPinch`, `E_TouchRotate`, `E_TouchStarted`, `E_TouchSwipe`, `E_TouchTap`, `E_VRDisabled`, `E_VREnabled`, `E_WindowFocusReleased`, `E_WindowFocused`, `E_WindowFullscreenChanged`, `E_WindowMinimized`, `E_WindowResized`, `E_WindowRestored`

### Value

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `BoolValue` | W |
| `IntValue` | W |
| `NumValue` | W |
| `NumberValue` | W |
| `ObjValue` | W |
| `ObjectValue` | W |
| `StrValue` | W |
| `StringValue` | W |
| `TableValue` | W |
| `Vector3Value` | W |

**Events:** `E_Changed`

### Vector2

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Magnitude` | R |
| `Unit` | R |
| `X` | R |
| `Y` | R |
| `x` | R |
| `y` | R |

**Methods**

- `:Dot(any)`
- `:Lerp(any, number)`

### Vector3

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Magnitude` | R |
| `Unit` | R |
| `X` | R |
| `Y` | R |
| `Z` | R |
| `magnitude` | R |
| `unit` | R |
| `x` | R |
| `y` | R |
| `z` | R |

**Methods**

- `:Dot(any)`
- `:Cross(any)`
- `:Lerp(any, number)`
- `:FuzzyEq(any, [number=1e-5])`
- `:Floor(any)`

### Video

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Duration` | R |
| `FrameRate` | R |
| `IsLoaded` | R |
| `Loaded` | R |
| `Loop` | R |
| `Looped` | R |
| `PlaybackSpeed` | R |
| `Playing` | R |
| `Resolution` | R |
| `TimeLength` | R |
| `TimePosition` | R |
| `Video` | R |
| `Volume` | R |

**Methods**

- `:Play()`
- `:Pause()`
- `:Stop()`
- `:SetTime(number)`
- `:SetVolume(any)`
- `:SetLoop(any)`

**Events:** `E_Ended`, `E_Error`, `E_Loaded`, `E_Pause`, `E_Play`

### Viewport

**Properties** (R = read, W = write)

| Property | Access |
|---|---|
| `Ambient` | R |
| `Camera` | R |
| `CurrentCamera` | R |
| `FieldOfView` | R |
| `LightColor` | R |
| `LightDirection` | R |

**Methods**

- `:SetCamera(any)` — alias: `SetCurrentCamera`
- `:SetAmbient(any)`
- `:SetLightColor(any)`
- `:SetLightDirection(any)`

### WS

**Methods**

- `:Send(any)` — WebSocket
- `:Close([string="closed"])`
- `:Ping()`


---
*Fluwa 0.5 · Minecraft 1.20.1 · Forge 47*
