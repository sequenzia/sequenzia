# The Complete Guide to Minecraft's `/execute` Command

**Minecraft Java Edition 1.20.x** | All Levels: Beginner to Advanced

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Modifier Subcommands](#modifier-subcommands)
5. [Condition Subcommands](#condition-subcommands)
6. [Store Subcommands](#store-subcommands)
7. [Run Subcommand](#run-subcommand)
8. [Practical Examples by Use Case](#practical-examples-by-use-case)
9. [Advanced Techniques](#advanced-techniques)
10. [Common Mistakes & Troubleshooting](#common-mistakes--troubleshooting)
11. [Quick Reference Table](#quick-reference-table)
12. [Version History](#version-history)

---

## Introduction

The `/execute` command is the most powerful and versatile command in Minecraft Java Edition. It allows you to modify **who** runs a command, **where** it runs, **when** it runs (conditionally), and **what happens** with the result. If Minecraft commands are a toolbox, `/execute` is the workbench that holds everything together.

### Why `/execute` Matters

- **Adventure maps**: Trigger events when players enter areas, interact with NPCs, or meet conditions
- **Server administration**: Manage players based on complex criteria, automate maintenance
- **Datapack development**: Build game mechanics, custom enchantments, minigames, and more

### History: The 1.13 Overhaul

Before Minecraft 1.13 (the "Flattening"), the execute command had a simple syntax:

```
/execute <entity> <x> <y> <z> <command>
/execute <entity> <x> <y> <z> detect <block> <data> <command>
```

In 1.13, the command was completely redesigned into a **chainable subcommand system**. This was a breaking change — every old `/execute` command had to be rewritten. The new system is far more powerful but has a steeper learning curve. This guide covers the modern (1.13+) syntax exclusively.

---

## Prerequisites

### Target Selectors

Target selectors let you specify which entities a command affects:

| Selector | Description | Example |
|----------|-------------|---------|
| `@p` | Nearest player | `/execute as @p run say Hello` |
| `@a` | All players | `/execute as @a run say Hello` |
| `@e` | All entities | `/execute as @e[type=zombie] run say Brains` |
| `@s` | The executing entity itself | `/execute as @a run tp @s ~ ~10 ~` |
| `@r` | Random player | `/execute as @r run give @s diamond 1` |
| `@n` | Nearest entity (1.20.5+) | `/execute as @n[type=cow] run say Moo` |

#### Common Selector Arguments

```
@e[type=minecraft:creeper]          — Filter by entity type
@a[gamemode=survival]               — Filter by game mode
@e[distance=..10]                   — Within 10 blocks of execution position
@a[scores={kills=5..}]              — Players with kills score >= 5
@e[tag=boss]                        — Entities with a specific tag
@a[team=red]                        — Players on the "red" team
@e[nbt={OnGround:1b}]              — Entities matching NBT data
@e[limit=1,sort=nearest]            — Single nearest entity
@a[level=30..]                      — Players with 30+ XP levels
@e[x=0,y=64,z=0,dx=10,dy=5,dz=10]  — Entities within a volume
@a[name="Steve"]                    — Player with specific name
@e[predicate=my_pack:is_valid]      — Using a custom predicate
```

### Coordinate Types

Minecraft has three coordinate systems you can use in commands:

**Absolute coordinates** — Exact world positions:
```
/execute positioned 100 64 200 run setblock ~ ~ ~ stone
```

**Relative coordinates (`~`)** — Offset from the current execution position:
```
/execute as @p at @s run setblock ~5 ~0 ~5 stone
```
`~5 ~0 ~5` means 5 blocks east and 5 blocks south from the executor.

**Local coordinates (`^`)** — Offset relative to the entity's facing direction:
```
/execute as @p at @s run summon arrow ^0 ^1.6 ^3
```
`^left ^up ^forward` — This summons an arrow 1.6 blocks above and 3 blocks in front of where the player is looking.

> **Important:** You cannot mix `~` and `^` in the same coordinate set. Use all relative or all local.

### Enabling Commands

- **Singleplayer**: Enable cheats when creating the world, or open to LAN with cheats enabled
- **Multiplayer**: Must be an operator (`/op <player>`) or have permission level 2+
- **Command blocks**: Enabled via `/gamerule commandBlockOutput true` (placed by operators)
- **Datapacks**: Function files automatically have permission level 2

---

## Core Concepts

### Execution Context

Every command in Minecraft runs within an **execution context** — a set of variables that define the circumstances of the command. The `/execute` command lets you modify this context before running the final command.

The context consists of:

| Property | Description | Modified by |
|----------|-------------|-------------|
| **Executor** | The entity "running" the command (`@s`) | `as`, `on`, `summon` |
| **Position** | The x/y/z coordinates of execution | `at`, `positioned`, `align`, `in` |
| **Rotation** | The pitch/yaw angles | `at`, `rotated`, `facing` |
| **Dimension** | Which world (overworld, nether, end) | `at`, `in` |
| **Anchor** | Eyes or feet reference point | `anchored` |

### Chaining

Subcommands are chained left to right. Each subcommand modifies the context, and the modified context is passed to the next subcommand:

```
/execute as @a at @s positioned ~ ~2 ~ run particle flame ~ ~ ~ 0 0 0 0.1 10
```

This chain does:
1. `as @a` — For each player (changes executor to each player)
2. `at @s` — Move position to where that player is
3. `positioned ~ ~2 ~` — Shift position 2 blocks up
4. `run particle ...` — Spawn flame particles at that modified position

### Forking

Some subcommands can **fork** execution — creating multiple branches that each continue independently. This happens when a selector matches multiple entities:

```
/execute as @e[type=zombie] run say I am a zombie!
```

If there are 5 zombies, this forks into 5 separate executions. Each zombie becomes `@s` and says the message. **Every subsequent subcommand in the chain runs once per fork.**

### Return Values

Commands produce two numeric results:
- **`result`** — The "data" output (e.g., a count, coordinate, or value)
- **`success`** — Whether the command succeeded (`1`) or failed (`0`)

These values can be captured using `store` subcommands.

---

## Modifier Subcommands

Modifier subcommands change the execution context without running a command.

### `as` — Change the Executor

Changes **who** is running the command (`@s`). Does **not** change position, rotation, or dimension.

**Syntax:**
```
execute as <targets> -> ...
```

**Examples:**
```mcfunction
# Make every cow say "Moo" (each cow becomes @s)
execute as @e[type=cow] run say Moo

# Give every player in survival mode a diamond
execute as @a[gamemode=survival] run give @s diamond 1

# Apply effect to all zombies
execute as @e[type=zombie] run effect give @s slowness 10 2
```

**Key point:** `as` only changes the executor. The execution position stays where it was (e.g., at the command block or the server console). To also move to the entity's location, follow `as` with `at @s`.

---

### `at` — Change Position, Rotation, and Dimension

Changes the position, rotation, **and** dimension to match the target entity. Does **not** change the executor.

**Syntax:**
```
execute at <targets> -> ...
```

**Examples:**
```mcfunction
# Spawn particles at every player's position
execute at @a run particle heart ~ ~2 ~ 0.5 0.5 0.5 0 5

# Summon lightning at each zombie
execute at @e[type=zombie] run summon lightning_bolt ~ ~ ~

# Place a block where each armor stand is
execute at @e[type=armor_stand,tag=marker] run setblock ~ ~ ~ redstone_block
```

**Key point:** `at` is commonly paired with `as` when you need to act both **as** an entity and **at** their location:

```mcfunction
# For each player: teleport them 5 blocks forward in their facing direction
execute as @a at @s run tp @s ^ ^ ^5
```

---

### `positioned` — Change Position Only

Changes only the x/y/z position. Does **not** change rotation or dimension.

**Syntax:**
```
execute positioned <x> <y> <z> -> ...
execute positioned as <targets> -> ...
```

**Examples:**
```mcfunction
# Run a command at a specific coordinate
execute positioned 0 100 0 run say I'm at 0 100 0

# Shift the position relatively (10 blocks up from current position)
execute positioned ~ ~10 ~ run setblock ~ ~ ~ glowstone

# Use another entity's position without changing executor
execute positioned as @e[type=marker,tag=spawn,limit=1] run spreadplayers ~ ~ 5 10 false @a
```

**`positioned as` vs `at`:** Use `positioned as` when you want an entity's position but not their rotation or dimension. `at` copies all three.

---

### `rotated` — Change Rotation Only

Changes only the pitch (up/down) and yaw (left/right) angles.

**Syntax:**
```
execute rotated <yaw> <pitch> -> ...
execute rotated as <targets> -> ...
```

**Examples:**
```mcfunction
# Face directly north (yaw 180, pitch 0)
execute rotated 180 0 run tp @s ~ ~ ~ ~ ~

# Copy another player's rotation
execute as @a[tag=puppet] rotated as @p[tag=controller] run tp @s ~ ~ ~ ~ ~

# Launch a projectile matching a player's look direction
execute as @p at @s rotated as @s run summon arrow ^ ^1.6 ^2 {Motion:[0.0,0.0,0.0]}
```

---

### `facing` — Face a Position or Entity

Sets the rotation to face a specific point or entity.

**Syntax:**
```
execute facing <x> <y> <z> -> ...
execute facing entity <target> (eyes|feet) -> ...
```

**Examples:**
```mcfunction
# Make all zombies face the nearest player
execute as @e[type=zombie] at @s facing entity @p eyes run tp @s ~ ~ ~ ~ ~

# Make a player face coordinates 0 64 0
execute as @p run tp @s ~ ~ ~ facing 0 64 0

# Summon a projectile facing toward a target
execute as @e[tag=turret] at @s facing entity @p eyes run summon arrow ^ ^0.5 ^1
```

---

### `align` — Snap Coordinates to Block Grid

Floors the x/y/z coordinates to the block grid (like `Math.floor()`). You specify which axes to align.

**Syntax:**
```
execute align <axes> -> ...
```
Where `<axes>` is any combination of `x`, `y`, `z` (e.g., `xyz`, `xz`, `y`).

**Examples:**
```mcfunction
# Snap to the corner of the block (useful for block-precise operations)
execute as @p at @s align xyz run setblock ~ ~ ~ stone

# Align only horizontally
execute positioned 5.7 64.3 -2.1 align xz run say Position: ~ ~ ~
# Result position: 5.0 64.3 -2.0

# Fill a region relative to a player's block position
execute as @p at @s align xz run fill ~ 0 ~ ~15 255 ~15 air replace stone
```

---

### `anchored` — Set Reference Point (Eyes or Feet)

Changes whether the `^` (local) coordinate system and `facing` use the entity's eyes or feet as the origin.

**Syntax:**
```
execute anchored (eyes|feet) -> ...
```

**Examples:**
```mcfunction
# Spawn particles at eye level
execute as @p at @s anchored eyes run particle smoke ^ ^ ^1 0 0 0 0 1

# Raycast from the player's eye position
execute as @p at @s anchored eyes positioned ^ ^ ^ run function my_pack:raycast

# Face an entity's eyes vs feet
execute as @e[type=zombie] at @s facing entity @p eyes run tp @s ~ ~ ~ ~ ~
```

**Default is `feet`.** Eye height varies by entity (1.62 blocks for players, different for other mobs).

---

### `in` — Change Dimension

Changes the execution dimension. Position is scaled appropriately when switching between the overworld and nether (8:1 ratio).

**Syntax:**
```
execute in <dimension> -> ...
```

**Dimensions:** `minecraft:overworld`, `minecraft:the_nether`, `minecraft:the_end`

**Examples:**
```mcfunction
# Check for a block in the nether
execute in minecraft:the_nether if block 0 64 0 netherrack run say Found netherrack

# Teleport to corresponding nether coordinates (position auto-scales)
execute in minecraft:the_nether run tp @s ~ ~ ~

# Place a marker in the end
execute in minecraft:the_end run summon marker 0 64 0 {Tags:["end_marker"]}
```

**Scaling:** When switching from overworld to nether, x and z are divided by 8. From nether to overworld, they're multiplied by 8. The end has no coordinate scaling.

---

### `on` — Relation-Based Executor Change (1.19.4+)

Changes the executor to an entity related to the current executor.

**Syntax:**
```
execute on <relation> -> ...
```

**Relations:**

| Relation | Description |
|----------|-------------|
| `attacker` | Last entity that damaged the current executor |
| `controller` | Entity riding and controlling the current executor |
| `leasher` | Entity holding the current executor's leash |
| `origin` | Entity that caused the current executor to exist (e.g., who shot an arrow) |
| `owner` | Owner of the current executor (tamed animals) |
| `passengers` | All entities riding the current executor |
| `target` | The attack target of the current executor |
| `vehicle` | Entity that the current executor is riding |

**Examples:**
```mcfunction
# Find who shot each arrow and give them points
execute as @e[type=arrow] on origin run scoreboard players add @s hits 1

# Make tamed wolves look at their owner
execute as @e[type=wolf] on owner run say My wolf is looking at me

# Find what a zombie is targeting
execute as @e[type=zombie] on target run effect give @s glowing 1 0

# Get the vehicle an entity is riding
execute as @p on vehicle run say You're riding me!
```

---

### `summon` — Spawn and Execute As (1.19.4+)

Summons an entity at the current execution position and immediately executes as that entity.

**Syntax:**
```
execute summon <entity_type> -> ...
```

**Examples:**
```mcfunction
# Summon a marker and immediately tag it
execute summon marker run tag @s add waypoint

# Summon a custom-named armor stand at a specific location
execute positioned 100 64 200 summon armor_stand run data merge entity @s {CustomName:'{"text":"Checkpoint"}',Invisible:1b,Marker:1b}

# Summon multiple entities in a chain
execute positioned ~ ~5 ~ summon pig run tag @s add flying_pig
```

---

## Condition Subcommands

Condition subcommands test whether something is true. `if` continues only when the condition passes; `unless` continues only when it fails (logical NOT).

The general pattern:
```
execute if <condition> -> ...    # Proceed if TRUE
execute unless <condition> -> ... # Proceed if FALSE
```

Conditions can be chained (implicit AND):
```
execute if <cond1> if <cond2> run <command>  # Both must be true
```

### `if/unless block` — Test Block Type

Tests whether a specific block exists at a position.

**Syntax:**
```
execute if block <pos> <block> -> ...
```

**Examples:**
```mcfunction
# Check if there's a diamond block at 0 64 0
execute if block 0 64 0 diamond_block run say Found diamonds!

# Check block below a player
execute as @a at @s if block ~ ~-1 ~ lava run effect give @s fire_resistance 5 0

# Test for a specific block state
execute if block ~ ~ ~ oak_stairs[facing=north,half=top] run say Found upside-down north stairs

# Unless variant: act only if the block is NOT air
execute at @p unless block ~ ~-1 ~ air run say You're standing on something
```

---

### `if/unless blocks` — Compare Block Volumes

Compares two rectangular regions of blocks.

**Syntax:**
```
execute if blocks <start> <end> <destination> (all|masked) -> ...
```

- `all` — Every block must match, including air
- `masked` — Only non-air blocks in the source must match

**Examples:**
```mcfunction
# Check if two 3x3x3 areas are identical
execute if blocks 0 64 0 2 66 2 10 64 10 all run say Areas match!

# Check only non-air blocks match (useful for structure validation)
execute if blocks 0 64 0 4 68 4 10 64 10 masked run say Structure detected

# Verify a player-built structure
execute positioned 100 64 100 if blocks ~ ~ ~ ~4 ~4 ~4 0 64 0 masked run say Correct build!
```

---

### `if/unless entity` — Test Entity Existence

Checks whether any entities match the selector.

**Syntax:**
```
execute if entity <targets> -> ...
```

**Examples:**
```mcfunction
# Check if any zombies exist
execute if entity @e[type=zombie] run say Zombies are nearby!

# Check if a specific player is online
execute if entity @a[name="Steve"] run say Steve is online

# Run only if no players are in creative mode
execute unless entity @a[gamemode=creative] run say Everyone is in survival

# Check for entities within a radius
execute as @a at @s if entity @e[type=creeper,distance=..5] run title @s actionbar "Creeper nearby!"
```

---

### `if/unless score` — Compare Scores

Tests scoreboard values with comparisons or ranges.

**Syntax:**
```
execute if score <target> <objective> (<|<=|=|>=|>) <source> <objective> -> ...
execute if score <target> <objective> matches <range> -> ...
```

**Range formats:** `5` (exact), `5..` (5 or more), `..5` (5 or less), `3..7` (3 to 7)

**Examples:**
```mcfunction
# Check if a player's score is above 100
execute as @a if score @s kills matches 100.. run title @s title "Century!"

# Compare two players' scores
execute if score Player1 points > Player2 points run say Player1 is winning

# Check exact value
execute as @a if score @s level matches 5 run say You are level 5

# Range check
execute as @a if score @s health matches 1..3 run title @s actionbar {"text":"Low health!","color":"red"}

# Unless: act when score is NOT in range
execute as @a unless score @s deaths matches 0 run say You have died before
```

---

### `if/unless data` — Test NBT Data

Checks whether NBT data exists at a path.

**Syntax:**
```
execute if data block <pos> <path> -> ...
execute if data entity <target> <path> -> ...
execute if data storage <storage> <path> -> ...
```

**Examples:**
```mcfunction
# Check if a chest has items
execute if data block 0 64 0 Items[0] run say Chest is not empty

# Check if a player is holding an item
execute as @a if data entity @s SelectedItem run say You're holding something

# Check for a specific item in hand
execute as @a if data entity @s SelectedItem{id:"minecraft:diamond_sword"} run say Nice sword!

# Check custom NBT data on an entity
execute as @e[type=marker] if data entity @s data.active run say This marker is active

# Test data storage
execute if data storage my_pack:config settings.enabled run say Config is enabled

# Check if a player has a specific enchantment
execute as @a if data entity @s SelectedItem.components."minecraft:enchantments".levels."minecraft:sharpness" run say Your sword has Sharpness
```

---

### `if/unless predicate` — Custom Predicate Test

Tests a JSON predicate defined in a datapack.

**Syntax:**
```
execute if predicate <predicate> -> ...
```

**Example predicate file** (`data/my_pack/predicates/is_daytime.json`):
```json
{
  "condition": "minecraft:time_check",
  "value": {
    "min": 0,
    "max": 12000
  }
}
```

**Examples:**
```mcfunction
# Use the predicate
execute if predicate my_pack:is_daytime run say It's daytime!

# Weather check predicate
execute if predicate my_pack:is_raining run say Bring an umbrella

# Combine with other conditions
execute as @a if predicate my_pack:holding_special_item run function my_pack:activate_item
```

---

### `if/unless biome` — Test Biome (1.19.3+)

Tests whether a position is in a specific biome.

**Syntax:**
```
execute if biome <pos> <biome> -> ...
```

**Examples:**
```mcfunction
# Check if player is in a desert
execute as @a at @s if biome ~ ~ ~ desert run title @s actionbar "You're in a desert"

# Conditional mob spawning by biome
execute at @e[tag=spawner] if biome ~ ~ ~ deep_dark run summon warden ~ ~ ~

# Unless: don't apply effect in the nether
execute as @a at @s unless biome ~ ~ ~ minecraft:nether_wastes run effect give @s regeneration 5 0
```

---

### `if/unless dimension` — Test Dimension (1.19.4+)

Tests whether execution is in a specific dimension.

**Syntax:**
```
execute if dimension <dimension> -> ...
```

**Examples:**
```mcfunction
# Only run in the overworld
execute if dimension minecraft:overworld run say We're in the overworld

# Different behavior per dimension
execute as @a at @s if dimension the_nether run title @s actionbar "Watch your step..."
execute as @a at @s if dimension the_end run title @s actionbar "Don't look down..."
```

---

### `if/unless loaded` — Test Chunk Loading (1.19.4+)

Tests whether the chunk at a position is fully loaded.

**Syntax:**
```
execute if loaded <pos> -> ...
```

**Examples:**
```mcfunction
# Only modify blocks in loaded chunks
execute if loaded 1000 64 1000 run setblock 1000 64 1000 diamond_block

# Safe chunk operations in a loop
execute positioned 0 64 0 if loaded ~ ~ ~ run function my_pack:process_chunk

# Check before teleporting
execute if loaded 5000 100 5000 run tp @s 5000 100 5000
```

---

### `if/unless function` — Test Function Return (1.20.2+)

Runs a function and checks its return value. The condition passes if the function returns a nonzero value.

**Syntax:**
```
execute if function <function> -> ...
```

The function must use the `return` command to produce a value:
```mcfunction
# data/my_pack/functions/check_ready.mcfunction
execute if entity @a[tag=ready,limit=1] run return 1
return 0
```

**Examples:**
```mcfunction
# Run only if the check function returns nonzero
execute if function my_pack:check_ready run say Everyone is ready!

# Use as a complex conditional
execute unless function my_pack:validate_arena run say Arena is not set up yet
```

---

### `if/unless items` — Test Inventory Items (1.20.5+)

Tests whether an entity or block has specific items in specific slots.

**Syntax:**
```
execute if items entity <target> <slots> <item_predicate> -> ...
execute if items block <pos> <slots> <item_predicate> -> ...
```

**Slot patterns:** `hotbar.*`, `inventory.*`, `weapon.mainhand`, `armor.chest`, `container.*`

**Examples:**
```mcfunction
# Check if player has a diamond sword in their hotbar
execute as @a if items entity @s hotbar.* diamond_sword run say Armed!

# Check for an item in a specific slot
execute as @a if items entity @s weapon.mainhand bow run say Ready to shoot

# Check armor
execute as @a if items entity @s armor.chest diamond_chestplate run say Diamond armor!

# Check a chest's contents
execute if items block 0 64 0 container.* diamond run say Chest has diamonds
```

---

## Store Subcommands

Store subcommands capture the **result** or **success** value from the final executed command and write it somewhere.

- `store result` — Stores the command's result value (data output)
- `store success` — Stores whether the command succeeded (`1`) or failed (`0`)

> **Important:** `store` only captures values from the command in the `run` clause. If no `run` is present (bare conditional), it captures from the condition itself.

### `store ... score` — Store in Scoreboard

**Syntax:**
```
execute store (result|success) score <target> <objective> -> ...
```

**Examples:**
```mcfunction
# Count nearby entities and store in a score
execute store result score @s mob_count if entity @e[type=zombie,distance=..20]

# Store a player's XP level
execute as @a store result score @s xp_level run experience query @s levels

# Store whether a block exists (success = 0 or 1)
execute store success score Global diamond_check if block 0 64 0 diamond_block

# Store a data query result
execute store result score @s health run data get entity @s Health
```

---

### `store ... block` — Store in Block Entity NBT

**Syntax:**
```
execute store (result|success) block <pos> <path> (byte|short|int|long|float|double) <scale> -> ...
```

**Examples:**
```mcfunction
# Store player count in a sign's NBT (pre-1.20)
execute store result block 0 64 0 front_text.messages[0] int 1 run scoreboard players get Global player_count

# Store a value in a jukebox
execute store result block 10 64 10 RecordItem.Count byte 1 run scoreboard players get @s score

# Set a command block's SuccessCount
execute store result block ~ ~-1 ~ SuccessCount int 1 if entity @a[distance=..5]
```

---

### `store ... entity` — Store in Entity NBT

**Syntax:**
```
execute store (result|success) entity <target> <path> (byte|short|int|long|float|double) <scale> -> ...
```

**Examples:**
```mcfunction
# Set a mob's health based on a score
execute as @e[type=zombie,tag=boss] store result entity @s Health float 1 run scoreboard players get @s boss_health

# Dynamically set an item's count
execute as @e[type=item] store result entity @s Item.Count byte 1 run scoreboard players get Global item_count

# Scale a value — multiply score by 0.5 when storing
execute store result entity @s Motion[1] double 0.01 run scoreboard players get @s launch_power
```

> **Note:** You cannot store to the executor's own NBT reliably in many cases because the entity data is read before the command runs. Use a two-step approach with scoreboards as an intermediary if needed.

---

### `store ... storage` — Store in Data Storage

Data storage is a persistent key-value system not tied to any entity or block.

**Syntax:**
```
execute store (result|success) storage <storage> <path> (byte|short|int|long|float|double) <scale> -> ...
```

**Examples:**
```mcfunction
# Store a player count in persistent storage
execute store result storage my_pack:data player_count int 1 run scoreboard players get Global player_count

# Store a timestamp
execute store result storage my_pack:data last_tick int 1 run time query gametime

# Use storage as a variable system
execute store result storage my_pack:temp calc_result int 1 run scoreboard players get @s math_output
```

---

### `store ... bossbar` — Store in Boss Bar

**Syntax:**
```
execute store (result|success) bossbar <id> (value|max) -> ...
```

**Examples:**
```mcfunction
# Use a bossbar as a visual health display
execute store result bossbar my_pack:boss_health value run data get entity @e[type=zombie,tag=boss,limit=1] Health

# Set bossbar max to player count
execute store result bossbar my_pack:progress max run scoreboard players get Global player_count

# Progress bar based on score
execute store result bossbar my_pack:quest_progress value run scoreboard players get Global quest_score
```

---

## Run Subcommand

The `run` subcommand terminates the chain and executes a command using the current (modified) execution context.

**Syntax:**
```
execute ... run <command>
```

You can run **any** valid Minecraft command after `run`, including another `execute`:

```mcfunction
# Nested execute
execute as @a run execute at @s run setblock ~ ~-1 ~ gold_block

# Though this is equivalent to the simpler:
execute as @a at @s run setblock ~ ~-1 ~ gold_block
```

### Execute Without `run`

An `/execute` chain can end with a condition instead of `run`. This is used to **test** conditions and returns a result:

```mcfunction
# Returns the count of matching entities (useful with store)
execute if entity @e[type=zombie]

# Store whether the condition passed
execute store success score Global test_result if block 0 64 0 diamond_block
```

---

## Practical Examples by Use Case

### Adventure Maps

#### NPC Dialogue Trigger
```mcfunction
# When a player gets close to an NPC, trigger dialogue
execute as @a at @s if entity @e[type=villager,tag=quest_npc,distance=..3] unless score @s npc_talked matches 1 run function my_map:dialogue/quest_intro

# In dialogue/quest_intro.mcfunction:
tellraw @s [{"text":"[Guide] ","color":"gold"},{"text":"Welcome, adventurer! I have a task for you...","color":"white"}]
scoreboard players set @s npc_talked 1
```

#### Area Detection with Scoreboards
```mcfunction
# Detect when a player enters a specific zone
execute as @a at @s if block ~ ~-1 ~ gold_block if entity @s[tag=!in_zone] run function my_map:enter_zone

# enter_zone.mcfunction:
tag @s add in_zone
title @s title {"text":"Danger Zone","color":"red"}
effect give @s darkness 5 0
playsound minecraft:entity.wither.ambient master @s ~ ~ ~ 0.5 1
```

#### Conditional Door Mechanism
```mcfunction
# Open a door when all 3 pressure plates are activated
execute if block 10 64 10 stone_pressure_plate[powered=true] if block 12 64 10 stone_pressure_plate[powered=true] if block 14 64 10 stone_pressure_plate[powered=true] run function my_map:open_secret_door
```

#### Boss Health Display
```mcfunction
# Tick function: update boss health bar
execute as @e[type=wither,tag=arena_boss,limit=1] store result bossbar my_map:boss value run data get entity @s Health
execute unless entity @e[type=wither,tag=arena_boss] run function my_map:boss_defeated
```

---

### Server Administration

#### AFK Detection
```mcfunction
# In a tick function: increment AFK timer, reset on movement
execute as @a store result score @s curr_pos_x run data get entity @s Pos[0] 1000
execute as @a store result score @s curr_pos_z run data get entity @s Pos[2] 1000
execute as @a unless score @s curr_pos_x = @s prev_pos_x run scoreboard players set @s afk_time 0
execute as @a unless score @s curr_pos_z = @s prev_pos_z run scoreboard players set @s afk_time 0
execute as @a if score @s curr_pos_x = @s prev_pos_x if score @s curr_pos_z = @s prev_pos_z run scoreboard players add @s afk_time 1
execute as @a at @s run scoreboard players operation @s prev_pos_x = @s curr_pos_x
execute as @a at @s run scoreboard players operation @s prev_pos_z = @s curr_pos_z

# Kick after 6000 ticks (5 minutes) of AFK
execute as @a if score @s afk_time matches 6000.. run kick @s AFK timeout
```

#### Per-World Border Effects
```mcfunction
# Damage players outside the world border area
execute as @a at @s unless entity @s[x=-500,z=-500,dx=1000,dz=1000] run effect give @s wither 2 1

# Warning as players approach the edge
execute as @a at @s unless entity @s[x=-450,z=-450,dx=900,dz=900] run title @s actionbar {"text":"Warning: Approaching world border!","color":"yellow"}
```

#### Auto Mob Cap
```mcfunction
# Kill excess hostile mobs to maintain server performance
execute store result score Global zombie_count if entity @e[type=zombie]
execute if score Global zombie_count matches 100.. as @e[type=zombie,sort=random,limit=20] run kill @s

execute store result score Global skeleton_count if entity @e[type=skeleton]
execute if score Global skeleton_count matches 80.. as @e[type=skeleton,sort=random,limit=15] run kill @s
```

---

### Datapack Development

#### Custom Health Regeneration
```mcfunction
# Tick function: players on grass regenerate slowly
execute as @a[gamemode=survival] at @s if block ~ ~-1 ~ grass_block run effect give @s regeneration 2 0 true
```

#### Scoreboard-Based Timer System
```mcfunction
# Decrement all active timers each tick
execute as @a if score @s timer matches 1.. run scoreboard players remove @s timer 1

# Trigger event when timer reaches zero
execute as @a if score @s timer matches 0 run function my_pack:timer_expired

# Set a 5-second timer (100 ticks)
scoreboard players set @s timer 100
```

#### Custom Crafting Detection
```mcfunction
# Detect when a player holds a specific combination (diamond + stick in specific slots)
execute as @a if items entity @s hotbar.0 diamond if items entity @s hotbar.1 stick if entity @s[scores={use_item=1..}] run function my_pack:craft_custom_weapon
```

#### Entity Processing Loop
```mcfunction
# Process each enemy: face nearest player, move forward
execute as @e[tag=custom_mob] at @s facing entity @p eyes run tp @s ^ ^ ^0.1 ~ ~
execute as @e[tag=custom_mob] at @s if entity @p[distance=..2] run function my_pack:attack_player
execute as @e[tag=custom_mob] at @s if block ~ ~1 ~ air unless block ~ ~ ~ air run tp @s ~ ~1 ~
```

---

## Advanced Techniques

### Raycasting

Raycasting lets you trace a line from an entity's eyes forward, detecting what they're looking at.

```mcfunction
# raycast/start.mcfunction — Called once to begin the ray
execute as @p at @s anchored eyes positioned ^ ^ ^ run function my_pack:raycast/step

# raycast/step.mcfunction — Recursive function that steps forward
# Check if we hit a block
execute unless block ~ ~ ~ air run function my_pack:raycast/hit_block

# Check if we hit an entity (excluding the caster)
execute if entity @e[type=!player,distance=..0.5] run function my_pack:raycast/hit_entity

# If nothing hit, step forward and recurse (with distance limit)
scoreboard players add @s ray_dist 1
execute if score @s ray_dist matches ..100 positioned ^ ^ ^0.5 unless block ~ ~ ~ air run function my_pack:raycast/hit_block
execute if score @s ray_dist matches ..100 positioned ^ ^ ^0.5 if block ~ ~ ~ air run function my_pack:raycast/step

# raycast/hit_block.mcfunction
particle crit ~ ~ ~ 0 0 0 0 10
setblock ~ ~ ~ glowstone
scoreboard players set @s ray_dist 999
```

A cleaner version using return (1.20.2+):

```mcfunction
# raycast/step.mcfunction
# Stop if we hit a solid block
execute unless block ~ ~ ~ air run return run function my_pack:raycast/hit_block

# Stop if we hit an entity
execute if entity @e[type=!player,distance=..0.5] run return run function my_pack:raycast/hit_entity

# Step forward
scoreboard players add @s ray_dist 1
execute if score @s ray_dist matches ..200 positioned ^ ^ ^0.5 run function my_pack:raycast/step
```

### Run-Once Per Tick Pattern

Ensures a function runs exactly once regardless of how many entities match:

```mcfunction
# Use limit=1 or a global fake player
execute if entity @e[type=zombie,limit=1] run function my_pack:zombie_logic

# Alternative: use a flag score
execute unless score Global processed matches 1 as @e[type=zombie,limit=1] run function my_pack:zombies_exist
scoreboard players set Global processed 1
# (reset Global processed to 0 at the start of each tick)
```

### Nested Execute Chains

Complex logic sometimes requires nesting. Each `execute` is a fresh chain:

```mcfunction
# For each player: if they're in the nether AND have low health, rescue them
execute as @a at @s if dimension the_nether run execute if score @s health matches ..6 run function my_pack:rescue

# This is equivalent to simply chaining (no nesting needed):
execute as @a at @s if dimension the_nether if score @s health matches ..6 run function my_pack:rescue
```

Nesting is mainly useful when you need different `store` targets:

```mcfunction
# Store the number of zombies near each player into that player's score
execute as @a at @s store result score @s nearby_zombies run execute if entity @e[type=zombie,distance=..30]
```

### Macro Patterns (1.20.2+)

Macros allow dynamic values in functions using `$(variable)` syntax:

```mcfunction
# set_block.mcfunction (macro function)
$setblock $(x) $(y) $(z) $(block)

# Calling with data storage:
data modify storage my_pack:temp args set value {x:"10",y:"64",z:"20",block:"diamond_block"}
function my_pack:set_block with storage my_pack:temp args
```

Combined with execute:

```mcfunction
# Store a value, then use it in a macro
execute store result storage my_pack:temp pos.y int 1 run data get entity @s Pos[1]
function my_pack:set_marker with storage my_pack:temp pos
```

### AND / OR / NOT Logic

```mcfunction
# AND: chain conditions (both must be true)
execute if score @s a matches 1 if score @s b matches 1 run say A and B

# NOT: use "unless"
execute unless score @s a matches 1 run say NOT A

# OR: requires multiple commands or a helper function
# Option 1: Separate commands
execute if score @s a matches 1 run function my_pack:do_thing
execute if score @s b matches 1 run function my_pack:do_thing

# Option 2: Score-based OR
scoreboard players set @s temp 0
execute if score @s a matches 1 run scoreboard players set @s temp 1
execute if score @s b matches 1 run scoreboard players set @s temp 1
execute if score @s temp matches 1 run say A or B
```

### FIFO Entity Queue Processing

Process entities one at a time across multiple ticks:

```mcfunction
# tick.mcfunction — Process one entity per tick
execute as @e[tag=queued,limit=1,sort=nearest] run function my_pack:process_entity

# process_entity.mcfunction
tag @s remove queued
tag @s add processed
# ... do expensive per-entity work ...
```

---

## Common Mistakes & Troubleshooting

### 1. Confusing `as` and `at`

**Problem:** Using `as` when you need position, or `at` when you need identity.

```mcfunction
# WRONG: Tries to teleport, but @s is still the command block/server
execute at @a run tp @s ~ ~10 ~

# CORRECT: Change executor so @s refers to the player
execute as @a at @s run tp @s ~ ~10 ~

# ALSO CORRECT: Target explicitly (no need for as/at)
execute run tp @a ~ ~10 ~
```

**Rule of thumb:**
- Need `@s` to refer to the entity? Use `as`.
- Need position/rotation from the entity? Use `at`.
- Need both? Use `as ... at @s`.

### 2. Forgetting `at @s` After `as`

**Problem:** `as` changes the executor but NOT the position.

```mcfunction
# WRONG: Particles spawn at the command block, not at each player
execute as @a run particle heart ~ ~2 ~ 0 0 0 0 5

# CORRECT: Move position to each player's location
execute as @a at @s run particle heart ~ ~2 ~ 0 0 0 0 5
```

### 3. Fork Explosion

**Problem:** Multiple selectors in a chain multiply the execution count.

```mcfunction
# If there are 10 players and 50 zombies, this runs 500 times!
execute as @a as @e[type=zombie] run say Brains

# Probably intended: for each player, check for nearby zombies
execute as @a at @s if entity @e[type=zombie,distance=..10] run say Zombies nearby!
```

### 4. Ordering Matters

**Problem:** Subcommands execute left to right. Position changes after `as` but before `at @s` are lost.

```mcfunction
# WRONG ORDER: positioned runs BEFORE at @s, so it's overwritten
execute as @a positioned ~ ~5 ~ at @s run setblock ~ ~ ~ torch

# CORRECT ORDER: at @s first, THEN offset
execute as @a at @s positioned ~ ~5 ~ run setblock ~ ~ ~ torch
```

### 5. Store + Condition Pitfall

**Problem:** When combining `store` with conditions, a failing condition stores `0`, not "nothing."

```mcfunction
# If no zombies exist, this stores 0 into the score (not "no change")
execute store result score Global zombie_count if entity @e[type=zombie]
```

This is usually the desired behavior, but be aware of it when using `store success`:

```mcfunction
# Success is 1 if ANY zombie exists, 0 if none — it's not a count!
execute store success score Global has_zombies if entity @e[type=zombie]

# For count, use "store result":
execute store result score Global zombie_count if entity @e[type=zombie]
```

### 6. Performance Tips

- **Avoid `@e` without filters** — Always add `type=`, `tag=`, `distance=`, or `limit=` to narrow selections
- **Minimize `nbt=` in selectors** — NBT checks are slow; prefer tags and scores
- **Use `limit=1` when you only need one entity** — Prevents unnecessary forking
- **Avoid per-tick raycasts for many entities** — Raycasting is recursive and expensive
- **Batch operations with functions** — Group related commands in `.mcfunction` files
- **Use tags instead of repeated selectors** — Tag entities once, then use `tag=` for fast filtering
- **Consider `schedule`** — Not everything needs to run every tick; use `schedule function` for periodic tasks

### 7. Common Syntax Errors

```mcfunction
# WRONG: Quotes around the block ID
execute if block ~ ~ ~ "diamond_block" run say Found!
# CORRECT:
execute if block ~ ~ ~ diamond_block run say Found!

# WRONG: Missing "run" before the final command
execute as @a at @s setblock ~ ~-1 ~ gold_block
# CORRECT:
execute as @a at @s run setblock ~ ~-1 ~ gold_block

# WRONG: Mixing ~ and ^ coordinates
execute as @p at @s run tp @s ~1 ^2 ~3
# CORRECT (pick one system):
execute as @p at @s run tp @s ~1 ~2 ~3
execute as @p at @s run tp @s ^1 ^2 ^3
```

---

## Quick Reference Table

| Subcommand | Category | Purpose | Version |
|------------|----------|---------|---------|
| `as <targets>` | Modifier | Change executor | 1.13+ |
| `at <targets>` | Modifier | Change position/rotation/dimension | 1.13+ |
| `positioned <x> <y> <z>` | Modifier | Change position | 1.13+ |
| `positioned as <targets>` | Modifier | Position from entity | 1.13+ |
| `rotated <yaw> <pitch>` | Modifier | Change rotation | 1.13+ |
| `rotated as <targets>` | Modifier | Rotation from entity | 1.13+ |
| `facing <x> <y> <z>` | Modifier | Face a position | 1.13+ |
| `facing entity <target> (eyes\|feet)` | Modifier | Face an entity | 1.13+ |
| `align <axes>` | Modifier | Floor coordinates | 1.13+ |
| `anchored (eyes\|feet)` | Modifier | Set reference point | 1.13+ |
| `in <dimension>` | Modifier | Change dimension | 1.13+ |
| `on <relation>` | Modifier | Relation-based executor | 1.19.4+ |
| `summon <entity>` | Modifier | Spawn + execute as | 1.19.4+ |
| `if/unless block` | Condition | Test block type | 1.13+ |
| `if/unless blocks` | Condition | Compare volumes | 1.13+ |
| `if/unless entity` | Condition | Test entity existence | 1.13+ |
| `if/unless score ... matches` | Condition | Score range test | 1.13+ |
| `if/unless score ... (<\|<=\|=\|>=\|>)` | Condition | Score comparison | 1.13+ |
| `if/unless data block` | Condition | Test block NBT | 1.13+ |
| `if/unless data entity` | Condition | Test entity NBT | 1.13+ |
| `if/unless data storage` | Condition | Test storage NBT | 1.14+ |
| `if/unless predicate` | Condition | Custom predicate | 1.15+ |
| `if/unless biome` | Condition | Test biome | 1.19.3+ |
| `if/unless dimension` | Condition | Test dimension | 1.19.4+ |
| `if/unless loaded` | Condition | Test chunk loaded | 1.19.4+ |
| `if/unless function` | Condition | Test function return | 1.20.2+ |
| `if/unless items` | Condition | Test inventory items | 1.20.5+ |
| `store result/success score` | Store | Save to scoreboard | 1.13+ |
| `store result/success block` | Store | Save to block NBT | 1.13+ |
| `store result/success entity` | Store | Save to entity NBT | 1.13+ |
| `store result/success storage` | Store | Save to data storage | 1.14+ |
| `store result/success bossbar` | Store | Save to bossbar | 1.13+ |
| `run <command>` | Terminal | Execute a command | 1.13+ |

---

## Version History

| Version | Additions |
|---------|-----------|
| **1.13** | Complete `/execute` overhaul. All core subcommands: `as`, `at`, `positioned`, `rotated`, `facing`, `align`, `anchored`, `in`, `if/unless block`, `if/unless blocks`, `if/unless entity`, `if/unless score`, `store`, `run` |
| **1.13.1** | Bug fixes for `execute store` |
| **1.14** | Added `if/unless data storage`, `store ... storage` (data storage system introduced) |
| **1.15** | Added `if/unless predicate` (custom JSON predicates) |
| **1.19.3** | Added `if/unless biome` |
| **1.19.4** | Added `on <relation>`, `summon <entity>`, `if/unless dimension`, `if/unless loaded` |
| **1.20.2** | Added `if/unless function`, `return` command, macro functions with `$()` syntax |
| **1.20.5** | Added `if/unless items` for inventory slot testing; NBT → components migration for items |

---

*This guide covers Minecraft Java Edition. Bedrock Edition uses a different `/execute` syntax.*
