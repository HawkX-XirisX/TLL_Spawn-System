# TLL_Spawn-System
TLL Spawn System — Scenario Maker Guide What is TLL Spawn System?  TLL Spawn System is a reusable Scenario Framework AI spawning system for Arma Reforger.  It is designed to let scenario makers place preconfigured spawn areas into their own scenarios and choose which AI character prefab should spawn from each slot.

The system provides:

AI spawning through Arma Reforger's Scenario Framework
Player-distance-based dynamic despawning
Automatic restoration when players return
Configurable AI death/respawn cooldown
Respawning only after the spawned AI/group has actually been eliminated
Independent SlotAI respawning
Configurable AI leash radius
Automatic return of AI that travels too far from its spawn location
Normal autonomous AI behavior while inside the allowed area
Protection against dynamic despawning being mistaken for AI death

The default TLL prefab is configured around a 500 m player activation/despawn range and a 50 m AI leash, but these values can be adjusted by the scenario maker.

IMPORTANT — Scenario Setup Requirements

Simply placing TLL_SpawnArea.et into a scenario is not enough.

The scenario itself needs the appropriate game-mode and AI/navigation infrastructure for Scenario Framework AI to function correctly.

1. Game Mode

The tested setup uses:

GameMode_Plain

The GameMode also needs:

SCR_GameModeSFManager

For the tested setup:

Dynamic Despawn: Enabled
Update Rate: 4

GameMode_Plain + SCR_GameModeSFManager is the combination used and validated during development of TLL Spawn System.

2. Faction Manager

Your scenario needs a Faction Manager containing the factions used by your scenario.

For example, our vanilla US/USSR test scenario uses:

FactionManager_USxUSSR

Use the appropriate Faction Manager for your scenario and its factions.

3. AIWorld — VERY IMPORTANT

Your scenario must contain an appropriate:

SCR_AIWorld

for the terrain/map being used.

Do not blindly copy the Arland AIWorld into another terrain.

For example, our Arland test scenario uses:

SCR_AIWorld_Arland

The AIWorld needs the correct navigation data for that terrain.

Navmesh is critical

During testing, AI could spawn, detect enemies and fight even though the Soldiers navigation world had no navmesh file assigned.

However, when the TLL leash attempted to issue a Forced Move order, the AI could not calculate a path and movement failed.

The log showed:

No navmesh file specified! Will initialize empty navmesh world.

For our Arland test scenario, the Soldiers NavmeshWorldComponent was configured with:

Navmesh Project: Soldiers
Navmesh File: CTI_Campaign_Arland.nmn

Once the correct navmesh was assigned, TLL's Forced Move leash worked properly.

Therefore:

Scenario makers must ensure their terrain has the correct AIWorld and Soldiers navmesh configured.

If AI spawn and shoot but fail to properly navigate or return when the leash activates, check the navmesh first.

Custom terrains will need navigation data appropriate for that terrain.

4. Player Spawning / Loadout Manager

If the scenario needs normal player spawning, make sure the scenario has the appropriate:

SpawnPoint
LoadoutManager

and that the Loadout Manager actually supports the factions being used.

For example, our vanilla US/USSR test scenario uses:

SpawnPoint_US
LoadoutManager_USxUSSR

Using only LoadoutManager_Base caused player spawning problems during our testing.

This isn't the TLL AI respawn system itself, but it is required for a correctly configured playable scenario.

5. Perception Manager

The tested scenario also contains:

PerceptionManager

This should be part of the scenario's normal AI/gameplay infrastructure.

Recommended Scenario Hierarchy

A basic test scenario may look something like:

FactionManager
GameMode_Plain
PerceptionManager
SCR_AIWorld_<YourMap>
SpawnPoint
LoadoutManager
TLL_SpawnArea

The exact prefab names will depend on the map, factions and scenario.

Using TLL_SpawnArea

Place:

TLL_SpawnArea.et

into your scenario.

The prefab internally uses a Scenario Framework structure equivalent to:

TLL_SpawnArea
└── Layer
    └── SlotAI

The included components handle the Area, Scenario Framework spawning and TLL respawn/leash behavior.

Choosing What AI Spawns

Select the SlotAI inside the placed TLL Spawn Area.

Find:

SCR_ScenarioFrameworkSlotAI

Then change:

Object To Spawn

to the AI character prefab you want that SlotAI to spawn.

For example, this could be a vanilla rifleman or an AI character supplied by another mod.

Important

TLL Spawn System does not require the spawn system itself to contain every possible AI prefab.

The scenario maker chooses the desired AI through:

Object To Spawn

This makes the system reusable with different factions, units and compatible modded AI character prefabs.

Adding More AI to an Area

If you want several independently spawned AI, duplicate the existing SlotAI.

For example:

TLL_SpawnArea
└── Layer
    ├── SlotAI1
    ├── SlotAI2
    ├── SlotAI3
    └── SlotAI4

Each SlotAI can have its own:

Object To Spawn

and its own position.

You can therefore build an area containing several different units.

During testing, multiple SlotAI entities operated independently: killing one unit started the respawn process for that SlotAI without duplicating or respawning all of the other living units.

Respawn Settings

Each SlotAI contains:

TLL_AIRespawnComponent

The important setting is:

Respawn Delay

This controls how long the system waits after that AI/group has been eliminated.

For example:

Respawn Delay: 600

means a 600-second / 10-minute cooldown.

For development testing we used:

Respawn Delay: 15

Once the cooldown expires, the SlotAI waits until a player is within the parent Area's activation range before restoring the spawn.

If the player is already inside that range when the timer finishes, restoration can happen immediately.

Dynamic Despawn

The parent TLL Spawn Area uses Scenario Framework Dynamic Despawn.

Our default/test configuration is:

Dynamic Despawn: ON
Dynamic Despawn Range: 500

This means AI can be removed when players move sufficiently far away and restored when players return.

Dynamic despawn is NOT death

TLL specifically distinguishes between:

AI actually being killed

and

Scenario Framework dynamically despawning the AI.

If the AI is dynamically despawned because players leave the area, TLL does not start the death respawn cooldown.

When players return, Scenario Framework can restore those AI normally.

This behavior has been tested successfully.

AI Leash System

Each SlotAI can optionally keep its spawned AI near its original spawn location.

Default/test settings:

Leash Enabled: ON
Leash Radius: 50
Return Release Radius: 10
How it works

While the AI remains within 50 m of its SlotAI position, TLL leaves it alone.

The AI can:

detect enemies
engage
seek cover
reposition
use normal autonomous AI behavior

If the AI moves farther than:

50 m

TLL creates a temporary:

AIWaypoint_ForcedMove

at the SlotAI spawn location.

The AI is ordered back toward its spawn area.

Once it returns within:

10 m

the temporary Forced Move waypoint is removed.

The AI then returns to normal autonomous behavior.

So the leash is not a permanent waypoint. It only intervenes when the AI exceeds the configured distance.

Why Forced Move Is Used

A normal Move waypoint was tested during development.

The AI could continue following autonomous/combat behavior rather than reliably returning to the spawn area.

TLL therefore uses:

AIWaypoint_ForcedMove.et

for the temporary return order.

This allows the leash to override autonomous movement when the AI has exceeded its allowed range.

Once the AI returns, TLL removes that order.

Leash Radius vs Dynamic Despawn Range

These are two completely different settings.

Leash Radius

Example:

50 m

Controls how far the spawned AI is allowed to autonomously travel from its SlotAI position before TLL orders it back.

Dynamic Despawn Range

Example:

500 m

Controls Scenario Framework's player-distance-based dynamic despawning.

So:

50 m = AI roaming/leash limit
500 m = player activation/dynamic despawn range

Do not treat the 500 m Dynamic Despawn Range as an AI patrol or roaming radius.

Waypoints

The TLL SlotAI should not require a permanent Defend waypoint for the leash system.

During development, AIWaypoint_Defend was tested, but its behavior was not appropriate for the desired TLL spawn system because the AI could move according to the Defend behavior rather than simply remaining naturally autonomous around its spawn.

The working TLL approach is:

Normal state:
No TLL return waypoint
↓
AI behaves autonomously
↓
AI exceeds leash
↓
Temporary Forced Move waypoint created
↓
AI returns
↓
Temporary waypoint deleted
↓
AI becomes autonomous again
Recommended Default TLL Settings

For a general-purpose spawn area:

AREA
Activation Type: ON_INIT
Spawn Children: ALL
Random Percent: 100
Repeated Spawn: OFF
Dynamic Despawn: ON
Dynamic Despawn Range: 500

Layer:

Spawn Children: ALL
Random Percent: 100
Activation Type: SAME_AS_PARENT
Repeated Spawn: OFF
Dynamic Despawn: OFF

SlotAI:

Object To Spawn: YOUR AI PREFAB
Use Existing World Asset: OFF
Can Be Garbage Collected: ON
Activation Type: SAME_AS_PARENT
Repeated Spawn: OFF
Dynamic Despawn: OFF
Exclude From Dynamic Despawn: OFF
WP To Spawn: None
AI Skill: REGULAR
Group Prefab: Group_Base.et
Importance: HIGH
Enabled: ON

TLL AI Respawn:

Respawn Delay: 600
Leash Enabled: ON
Leash Radius: 50
Return Release Radius: 10

Scenario makers are free to adjust the respawn delay and leash distances to suit their scenario.

Troubleshooting
AI does not spawn at all

Check that the scenario has:

GameMode_Plain
SCR_GameModeSFManager
FactionManager
SCR_AIWorld
PerceptionManager

Also verify that the SlotAI's Object To Spawn points to a valid AI character prefab.

AI spawns but cannot navigate / leash fails

Check the AIWorld and especially the Soldiers navmesh.

If the log contains:

No navmesh file specified! Will initialize empty navmesh world.

your navigation setup is incomplete.

The AI may still appear capable of some combat behavior, but navigation orders can fail.

AI crosses the leash radius and immediately returns

That's expected.

Example:

Leash Radius: 50

means crossing 50 m causes TLL to issue its temporary Forced Move return order.

Increase the leash radius if you want AI to pursue enemies farther.

AI disappears when players leave

If Dynamic Despawn is enabled, this is expected.

It is an optimization feature, not a death.

When players return within the appropriate range, the AI should reappear.

AI doesn't immediately respawn after death

Check:

Respawn Delay

After the cooldown completes, TLL also requires a player to be within the parent Area's activation/dynamic-despawn range.

Player cannot spawn

Check your scenario's SpawnPoint, Faction Manager and Loadout Manager.

For our US/USSR test scenario:

FactionManager_USxUSSR
SpawnPoint_US
LoadoutManager_USxUSSR

worked correctly.

This is scenario configuration rather than a TLL Spawn System failure.

Tested Behavior

The current TLL Spawn System has been tested for:

✔ Initial AI spawning
✔ Multiple independent SlotAI spawns
✔ AI combat behavior
✔ AI autonomous movement
✔ 50 m leash detection
✔ Forced Move return
✔ Return waypoint removal
✔ Return to autonomous AI behavior
✔ AI death detection
✔ Configurable respawn cooldown
✔ Independent SlotAI respawning
✔ No duplication when only one of several SlotAI units dies
✔ Dynamic despawn when players leave the Area
✔ Dynamic restoration when players return
✔ Dynamic despawn not triggering the death cooldown
Important note for custom maps

TLL Spawn System does not provide navigation data for every terrain.

The scenario/map author is responsible for configuring the correct:

SCR_AIWorld
Navmesh

for their terrain.

If those are missing, TLL can detect that an AI exceeded its leash and issue the return order, but the AI may be unable to calculate a path back.

Quick Setup Checklist

Before reporting a TLL Spawn System problem, verify:

[ ] TLL Spawn System addon is enabled
[ ] GameMode_Plain exists
[ ] SCR_GameModeSFManager is present/configured
[ ] Correct Faction Manager exists
[ ] Correct AIWorld for the terrain exists
[ ] Soldiers navmesh is actually assigned
[ ] PerceptionManager exists
[ ] Correct Loadout Manager exists if player spawning is required
[ ] Required SpawnPoint exists
[ ] TLL_SpawnArea.et is placed
[ ] SlotAI Object To Spawn points to a valid AI prefab
[ ] TLL_AIRespawnComponent is enabled
[ ] Dynamic Despawn range is configured
[ ] Leash settings are configured as desired

Most important troubleshooting rule: if AI can spawn and fight but cannot properly move back when the TLL leash activates, check the terrain's AIWorld/navmesh configuration before changing TLL Spawn System.
