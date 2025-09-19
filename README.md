# 25Fall-CISC486-G3

## Idea Clarity

XXX is a two-player cooperative tower defense game based on grid maps. Two players jointly defend the base camp (HQ) in the same local area network session. The enemy will dynamically change course based on the defense towers newly built or dismantled by the players.

**Victory**: Health>0 at the end of all waves.

**Failure**: Any enemy arriving at the HQ will have their shared Health (e.g., initially 20) deducted based on their weight. Failure occurs when Health≤0.

**Sharing economy**: Kill and turn settlements produce a gold and gold is shared between 2 players.

**Loading limit**: Each player can only carry limited types of towers before entering the level(e.g., 2 or 3 each person, can be different). Division of labor and collaboration are encouraged.

**Placement rule**: The tower must be placed in a buildable grid. After the server verification is placed, there is still A feasible path (A*) from the entry to the HQ. If it will block the path, then refuse to place it.

**Defense Tower** (First Batch prototype) : Archer (low-cost monomer)/Cannon (range sputtering)/Oil (deceleration main)/Tesla (Chain Thunder)/Buffer (Ranged Buff or Debuff)

**Enemies** (First Batch Prototypes) : Trooper/Armored/Warded/Swarm/Supporter(Enhancer)/Elite Mob(Champion)

**Playability**: The "instant route change" within Boci turns "road closure - traffic diversion - firepower coverage" into a real-time game. Loading restrictions force teammates to build complementary systems(types of towers should be complementary).

## AI Plan & FSM

**Flow:** `Spawn → Repath ↔ Move → { Goal | Dead }`

### States

#### Spawn
- Initialize unit (HP, speed, drops).
- Subscribe to **GridVersion** changes.
- **Transition:** → **Repath** immediately.

#### Repath
- Server runs **A\*** from current cell to goal (Manhattan + small turn cost; cached).
- **On success:** → **Move**  
- **On failure:** short delay then retry (placement validation prevents permanent no-path).

#### Move
- Follow waypoints (speed/acceleration limits, corner smoothing, light separation).
- **If path invalid:** → **Repath**

#### Goal
- **Regular enemies:** apply Hearts damage; despawn.  
- **Supply Convoy:** counts as *Escaped*; apply **stacking buff** to future waves; despawn.

#### Dead / Despawn
- **HP ≤ 0:** play death FX and drops (Convoy has no Hearts damage; drops per design); despawn.

### Transitions

- **Spawn → Repath:** on unit creation.  
- **Repath → Move:** when a valid path is obtained.  
- **Move ↔ Repath:** triggered by any of:
  - **GridVersion++** (any tower place/sell/upgrade),
  - Next cell becomes unwalkable/occupied,
  - Current path invalid or timed out.  
  *Debounce with 100–200 ms jitter and a per-frame replan cap.*
- **Move → Goal:** enters goal radius (HQ or Supply Exit).  
- **Move → Dead:** HP ≤ 0.

## Scripted Event

**Supply Convoy**: A convoy with elite guards spawns.

**Condition**: Every 5 waves.

**Composition**: 1× Convoy (very tanky, slow, no Hearts damage) + Guards (count scales with wave).

**Effect**: Award bonus gold (scales with current wave) if destroyed. Subsequent waves’ enemies gain a stacking "Well-Supplied" buff if escaped.

## Multiplayer Plan

**Session**: 2 players play over LAN, server-authoritative (Host = Server, Client connects via LAN IP).

**Shared**: Hearts / gold coins / waves / random seed.

**Action flow**: Client only sends instruction to build/demolish/upgrade → server validates (including path not blocked) and applies → broadcasts grid and state (including Gridversion).

**Waves**: Start when both players are Ready or the countdown reaches zero. In waves, building/demolishing is allowed and immediately affects enemy paths.

**Sync Objects**: Enemy transforms like interpolation or snapshots, tower types and levels, shared resources and waves, script events.

## Environment & Assets

**Map**: URP top-down 3D, enemy entrance, HQ, buildable tiles and path tiles will be clearly distinguished.

**UI**: Shared hearts/gold/waves, two-player Ready, loadout selection, build hotkey bar.
Prototype assets: Unity primitives, simple materials and placeholder sound effects. All of these will be replace later by Assets shop.

**Debug**: F1 shows grid, A* paths, enemy FSM text (for A2 and A3).
