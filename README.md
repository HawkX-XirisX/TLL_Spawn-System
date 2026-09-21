TLL Spawn System

TLL Spawn System is a reusable AI spawning, respawning, dynamic despawning, and AI leash system for Arma Reforger Scenario Framework.

It is designed for scenario makers who want to place preconfigured AI spawn areas into their scenarios while still being able to choose which vanilla or modded AI units are spawned.

The system handles AI lifecycle management automatically while allowing the spawned AI to retain normal autonomous combat behavior.

Features

Scenario Framework AI spawning

Configurable AI respawn timer

Player-distance-based dynamic despawning

Automatic restoration when players return

Independent SlotAI respawning

Dynamic despawn is distinguished from actual AI death

Configurable AI leash system

AI remains autonomous while inside its leash

Temporary Forced Move return order when AI exceeds its leash

Automatic return to autonomous behavior after reaching the spawn area

Supports multiple independent SlotAI entities

Scenario makers can select their own AI character prefabs

Compatible with vanilla and compatible modded AI character prefabs

Designed to avoid unnecessary persistent AI

How the System Works

The basic TLL Spawn System structure is:

TLL_SpawnArea
└── Layer
    └── SlotAI

The TLL_SpawnArea handles Scenario Framework activation and dynamic despawning.

Each SlotAI controls an individual AI spawn and contains the TLL respawn/leash logic.

Scenario makers can duplicate SlotAI entities to create larger AI spawn areas.

Example:

TLL_SpawnArea
└── Layer
    ├── SlotAI1
    ├── SlotAI2
    ├── SlotAI3
    └── SlotAI4

Each SlotAI can spawn a different AI character if desired.

Scenario Requirements

[!IMPORTANT]
Simply placing TLL_SpawnArea.et into a scenario is not enough.

The scenario must have the required Game Mode, Scenario Framework manager, faction configuration, AIWorld, and navigation data.

1. Game Mode

The tested setup uses:

GameMode_Plain

The Game Mode also requires:

SCR_GameModeSFManager

The tested configuration uses:

Dynamic Despawn: Enabled
Update Rate: 4

GameMode_Plain with SCR_GameModeSFManager is the configuration used during development and testing of TLL Spawn System.

2. Faction Manager

Your scenario needs a Faction Manager containing the factions used by the scenario.

For example, a vanilla US/USSR scenario can use:

FactionManager_USxUSSR

Use the appropriate Faction Manager for your scenario and its factions.

3. AIWorld and Navmesh

[!CAUTION]
This is one of the most important requirements.

Your scenario needs an appropriate:

SCR_AIWorld

for the terrain being used.

For example, the TLL Arland test scenario uses:

SCR_AIWorld_Arland

Do not blindly copy the Arland AIWorld into another terrain.

The correct AIWorld depends on the terrain.

Soldiers Navmesh

The AIWorld must also have usable navigation data.

During development, AI could:

spawn

detect enemies

shoot

move during some combat behavior

while the Soldiers navigation world was missing its navmesh.

However, navigation orders such as the TLL Forced Move leash failed.

The log reported:

PATHFINDING(W): No navmesh file specified! Will initialize empty navmesh world.

For the Arland test scenario, the Soldiers NavmeshWorldComponent was configured with:

Navmesh Project: Soldiers
Navmesh File: CTI_Campaign_Arland.nmn

Once the correct navmesh was assigned, TLL's Forced Move leash worked correctly.

Custom Maps

TLL Spawn System does not provide navigation data for every terrain.

Custom terrain/scenario authors are responsible for providing and configuring the correct:

SCR_AIWorld
Navmesh

for their map.

[!TIP]
If AI spawn and fight correctly but fail to return when the TLL leash activates, check the AIWorld and Soldiers navmesh before modifying the TLL scripts.

4. Perception Manager

The tested scenario contains:

PerceptionManager

This should be included as part of the scenario's AI/gameplay infrastructure.

5. Player Spawn Setup

If the scenario requires normal player spawning, it also needs the appropriate:

SpawnPoint
LoadoutManager

For example, the vanilla US/USSR test scenario uses:

SpawnPoint_US
LoadoutManager_USxUSSR

During testing, using only:

LoadoutManager_Base

caused player spawning problems.

This is not part of the TLL AI respawn system itself, but the scenario still needs a correctly configured player spawn system.

Example Scenario Setup

A basic scenario may contain:

FactionManager
GameMode_Plain
PerceptionManager
SCR_AIWorld_<YourMap>
SpawnPoint
LoadoutManager
TLL_SpawnArea

Exact prefab names depend on your terrain, factions, and scenario configuration.

Adding TLL Spawn System to a Scenario

Place:

TLLSpawnSystem/Prefabs/SpawnSystem/TLL_SpawnArea.et

into your scenario.

The prefab contains the Scenario Framework Area, Layer, SlotAI, and TLL AI respawn/leash configuration required by the system.

Selecting Which AI Spawns

Open the placed TLL_SpawnArea and select its SlotAI.

Find:

SCR_ScenarioFrameworkSlotAI

Then locate:

Object To Spawn

Set this to the AI character prefab you want to spawn.

For example:

Object To Spawn: Character_US_Rifleman.et

or another compatible vanilla/modded AI character prefab.

TLL Spawn System does not need to contain every possible AI prefab.

The scenario maker chooses the desired unit through:

Object To Spawn

Adding Multiple AI

To add more independently managed AI to the same area, duplicate the existing SlotAI.

Example:

TLL_SpawnArea
└── Layer
    ├── SlotAI1
    ├── SlotAI2
    ├── SlotAI3
    └── SlotAI4

Move each SlotAI to the desired spawn position.

Each SlotAI can also use a different:

Object To Spawn

During testing, multiple SlotAI entities operated independently.

Killing one spawned AI did not cause all other living SlotAI entities to respawn or duplicate.

Respawn System

Each SlotAI contains:

TLL_AIRespawnComponent

The primary respawn setting is:

Respawn Delay

Example:

Respawn Delay: 600

This represents a 600-second / 10-minute respawn cooldown.

For development testing, a shorter value was used:

Respawn Delay: 15

Respawn Flow

When the AI/group is eliminated:

AI/group eliminated
        ↓
Respawn cooldown starts
        ↓
Cooldown finishes
        ↓
System checks for a player inside the Area range
        ↓
SlotAI is restored
        ↓
AI spawns again

If a player is already within the Area's activation range when the cooldown finishes, the SlotAI can be restored immediately.

If no player is nearby, the system waits until a player enters the Area range.

Dynamic Despawn

The parent TLL Spawn Area uses Scenario Framework Dynamic Despawn.

The tested/default configuration is:

Dynamic Despawn: ON
Dynamic Despawn Range: 500

When players move sufficiently far away, Scenario Framework can dynamically remove the spawned AI.

When players return, the AI is restored.

Dynamic Despawn Is Not Death

TLL distinguishes between:

AI killed

and:

AI dynamically despawned

A dynamically despawned AI does not start the TLL death/respawn cooldown.

This prevents normal Scenario Framework optimization from being mistaken for an AI death.

The behavior has been tested successfully with multiple SlotAI entities.

AI Leash System

TLL includes an optional leash system that prevents spawned AI from wandering indefinitely away from its assigned spawn location.

The tested/default settings are:

Leash Enabled: ON
Leash Radius: 50
Return Release Radius: 10

Normal AI Behavior

While the AI remains within the configured leash radius, TLL does not interfere with it.

The AI can behave normally:

Detect enemies

Engage enemies

Reposition

Seek cover

React to combat

Use normal autonomous AI behavior

For example:

Leash Radius: 50

allows the AI to behave autonomously while it remains within approximately 50 metres of its SlotAI spawn position.

Exceeding the Leash

If the AI travels farther than the configured leash radius:

AI exceeds 50 m
        ↓
TLL detects the distance
        ↓
Temporary Forced Move waypoint is created
        ↓
AI is ordered back toward its SlotAI spawn position

TLL uses:

AIWaypoint_ForcedMove.et

for this temporary return order.

Returning to the Spawn Area

The default:

Return Release Radius: 10

means that once the AI gets within 10 metres of its original SlotAI position:

AI returns within 10 m
        ↓
Temporary Forced Move waypoint is removed
        ↓
AI returns to autonomous behavior

The leash therefore does not permanently assign a waypoint to the AI.

It only intervenes when the AI exceeds its configured roaming distance.

Why TLL Uses Forced Move

A normal:

AIWaypoint_Move.et

was tested during development.

The AI could continue following autonomous/combat behavior instead of reliably obeying the return order.

TLL therefore uses:

AIWaypoint_ForcedMove.et

for the temporary leash return.

Once the AI returns to the configured release radius, TLL removes the Forced Move waypoint and allows normal autonomous behavior again.

Leash Radius vs Dynamic Despawn Range

These settings perform completely different jobs.

Leash Radius

Example:

50 m

Controls how far the AI itself can travel from its SlotAI position before TLL orders it back.

Dynamic Despawn Range

Example:

500 m

Controls Scenario Framework's player-distance-based AI despawning.

In simple terms:

50 m  = AI roaming/leash limit
500 m = Player activation/dynamic despawn range

The Dynamic Despawn Range is not an AI patrol radius.

Waypoints

TLL does not require a permanent Defend waypoint for its leash system.

The working design is:

No TLL return waypoint
        ↓
AI behaves autonomously
        ↓
AI exceeds leash radius
        ↓
Temporary Forced Move waypoint created
        ↓
AI returns toward spawn
        ↓
AI enters Return Release Radius
        ↓
Temporary waypoint deleted
        ↓
AI becomes autonomous again

During development, AIWaypoint_Defend was tested but did not provide the desired behavior for this system.

For the standard TLL setup:

WP To Spawn: None

is used.

Recommended Configuration

TLL Spawn Area

Activation Type: ON_INIT
Spawn Children: ALL
Random Percent: 100
Repeated Spawn: OFF

Dynamic Despawn: ON
Dynamic Despawn Range: 500

Layer

Spawn Children: ALL
Random Percent: 100
Activation Type: SAME_AS_PARENT
Repeated Spawn: OFF

Dynamic Despawn: OFF

SlotAI

Object To Spawn: <YOUR AI PREFAB>

Use Existing World Asset: OFF
Can Be Garbage Collected: ON

Activation Type: SAME_AS_PARENT
Repeated Spawn: OFF

Dynamic Despawn: OFF
Exclude From Dynamic Despawn: OFF
Exclude From Dynamic Despawn On Vehicle Entered: ON

Randomize Per Faction: OFF
Entity Catalog Type: NONE

WP To Spawn: None
Spawn AI On WP Pos: OFF

Balance On Players Count: OFF
Min Units In Group: 1

AI Group Formation: Wedge
AI Skill: REGULAR
Group Prefab: Group_Base.et

Importance: HIGH
Enabled: ON

TLL AI Respawn Component

Recommended starting configuration:

Respawn Delay: 600

Leash Enabled: ON
Leash Radius: 50
Return Release Radius: 10

Scenario makers can adjust these values depending on the desired gameplay.

Troubleshooting

AI Does Not Spawn

Verify that your scenario contains and correctly configures:

GameMode_Plain
SCR_GameModeSFManager
FactionManager
SCR_AIWorld
PerceptionManager

Also verify:

SlotAI
└── Object To Spawn

points to a valid AI character prefab.

AI Spawns and Shoots but Cannot Navigate

Check the AIWorld and its Soldiers navmesh.

If the log contains:

No navmesh file specified! Will initialize empty navmesh world.

your navigation configuration is incomplete.

AI may still appear to perform some combat behavior while navigation orders fail.

AI Crosses 50 m but Does Not Return

First verify:

Leash Enabled: ON
Leash Radius: 50

Then check the log for:

[TLL_SpawnSystem] Group exceeded leash radius.
[TLL_SpawnSystem] Temporary Forced Move return waypoint created.

If those messages appear but the AI cannot return, check the map's Soldiers navmesh.

AI Immediately Returns During Combat

If the AI crosses the configured leash radius, this is expected.

Increase:

Leash Radius

if the scenario should allow AI to pursue enemies farther away.

For example:

Leash Radius: 100

would allow a larger autonomous area than the default 50 m.

AI Disappears When Players Leave

If:

Dynamic Despawn: ON

this is expected behavior.

The AI is being dynamically removed for optimization.

It has not been killed.

When players return within the appropriate range, the AI should be restored.

AI Does Not Respawn Immediately After Death

Check:

Respawn Delay

TLL waits for this cooldown to finish.

Afterward, the system also checks whether a player is within the parent Area's activation/dynamic-despawn range.

If nobody is nearby, the SlotAI waits until a player returns.

Player Cannot Spawn

Check your scenario's:

Faction Manager
SpawnPoint
Loadout Manager

For the vanilla US/USSR test scenario, the following worked correctly:

FactionManager_USxUSSR
SpawnPoint_US
LoadoutManager_USxUSSR

Player spawning is scenario configuration and is separate from the TLL AI respawn system.

Tested Behavior

The current TLL Spawn System has been tested for:

Initial AI spawning

Multiple independent SlotAI spawns

Normal AI combat behavior

Autonomous AI movement

50 m leash detection

Forced Move return

Return waypoint removal

Return to autonomous behavior

AI death detection

Configurable respawn cooldown

Independent SlotAI respawning

Killing one SlotAI does not duplicate other living AI

Multiple killed SlotAI entities respawn independently

Dynamic despawn when players leave the Area

Dynamic restoration when players return

Dynamic despawn does not trigger the death cooldown

AI navigation after correct terrain navmesh configuration

Quick Setup Checklist

Before troubleshooting TLL Spawn System, verify:

TLL Spawn System addon is enabled

GameMode_Plain exists

SCR_GameModeSFManager is configured

Correct Faction Manager exists

Correct AIWorld for the terrain exists

Soldiers navmesh is assigned

PerceptionManager exists

Correct Loadout Manager exists if player spawning is required

Required SpawnPoint exists

TLL_SpawnArea.et is placed

SlotAI Object To Spawn points to a valid AI prefab

TLL_AIRespawnComponent is enabled

Dynamic Despawn range is configured

Leash settings are configured as desired

Example Working Arland Setup

The development/test scenario used the following general configuration:

FactionManager_USxUSSR
GameMode_Plain
PerceptionManager
SCR_AIWorld_Arland
SpawnPoint_US
LoadoutManager_USxUSSR
TLL_SpawnArea

For the Soldiers navigation world:

Navmesh Project: Soldiers
Navmesh File: CTI_Campaign_Arland.nmn

TLL settings during final testing:

Respawn Delay: 15
Leash Enabled: ON
Leash Radius: 50
Return Release Radius: 10
Dynamic Despawn Range: 500

The 15-second respawn delay was used for testing and does not need to be the production value.

Important Notes for Custom Terrains

TLL Spawn System handles AI spawn lifecycle, respawning, dynamic despawn awareness, and leash behavior.

It does not replace Arma Reforger's AI navigation system.

The terrain/scenario author is responsible for providing a functioning:

SCR_AIWorld
+
Soldiers Navmesh

for the terrain.

If the terrain does not provide usable AI navigation data, TLL may successfully issue a Forced Move return order while the AI itself is unable to calculate a path.

Debug Logging

TLL Spawn System writes useful events to the log using:

[TLL_SpawnSystem]

Examples include:

[TLL_SpawnSystem] New AI group detected.

[TLL_SpawnSystem] Group exceeded leash radius.
[TLL_SpawnSystem] Temporary Forced Move return waypoint created.

[TLL_SpawnSystem] Group returned to spawn area.
[TLL_SpawnSystem] Return order removed. Group is autonomous again.

[TLL_SpawnSystem] Group eliminated. Respawn cooldown started.

[TLL_SpawnSystem] Respawn cooldown finished. Waiting for a player to enter the Area.

[TLL_SpawnSystem] Player entered Area range. Restoring SlotAI.

[TLL_SpawnSystem] SlotAI restored after player returned.

When troubleshooting, filtering the Workbench log for:

TLL_SpawnSystem

is usually the quickest way to see what the system is doing.

TLL — The Last Light

Developed for The Last Light Arma Reforger scenarios and made reusable for other scenario makers.

If you encounter an issue, please include the following when reporting it:

Map/terrain being used

AI prefab being spawned

AIWorld being used

Navmesh configuration

TLL Spawn Area settings

TLL SlotAI settings

Relevant [TLL_SpawnSystem] log output
