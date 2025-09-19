# 25Fall-CISC486-G3

## Idea Clarity

XXX is a two-player cooperative tower defense game based on grid maps. Two players jointly defend the base camp (HQ) in the same local area network session. The enemy will dynamically change course based on the defense towers newly built or dismantled by the players.

Victory: Health>0 at the end of all waves.

Failure: Any enemy arriving at the HQ will have their shared Health (e.g., initially 20) deducted based on their weight. Failure occurs when Health≤0.

Sharing economy: Kill and turn settlements produce a gold and gold is shared between 2 players.

Loading limit: Each player can only carry limited types of towers before entering the level(e.g., 2 or 3 each person, can be different). Division of labor and collaboration are encouraged.

Placement rule: The tower must be placed in a buildable grid. After the server verification is placed, there is still A feasible path (A*) from the entry to the HQ. If it will block the path, then refuse to place it.

Tower (First Batch prototype) :
Archer (low-cost monomer)/Cannon (range sputtering)/Oil (deceleration main)/Tesla (Chain Thunder)/Buffer (Ranged Buff or Debuff)

Enemies (First Batch Prototypes) : Trooper/Armored/Warded/Swarm/Supporter(Enhancer)/Elite Mob(Champion)

Playability: The "instant route change" within Boci turns "road closure - traffic diversion - firepower coverage" into a real-time game. Loading restrictions force teammates to build complementary systems(types of towers should be complementary).

## AI Plan & FSM

pass!!!

## Scripted Event

pass!!!

## Multiplayer Plan

Session: 2 players play over LAN, server-authoritative (Host = Server, Client connects via LAN IP).

Shared: Hearts / gold coins / waves / random seed.

Action flow: Client only sends instruction to build/demolish/upgrade → server validates (including path not blocked) and applies → broadcasts grid and state (including Gridversion).

Waves: Start when both players are Ready or the countdown reaches zero. In waves, building/demolishing is allowed and immediately affects enemy paths.

Sync Objects: Enemy transforms like interpolation or snapshots, tower types and levels, shared resources and waves, script events.

## Environment & Assets

Map: URP top-down 3D, enemy entrance, HQ, buildable tiles and path tiles will be clearly distinguished.

UI: Shared hearts/gold/waves, two-player Ready, loadout selection, build hotkey bar.
Prototype assets: Unity primitives, simple materials and placeholder sound effects. All of these will be replace later by Assets shop.

Debug: F1 shows grid, A* paths, enemy FSM text (for A2 and A3).

