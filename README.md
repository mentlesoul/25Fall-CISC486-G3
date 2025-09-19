# 25Fall-CISC486-G3

## Idea Clarity

**Overview**: A two-player cooperative tower defense game on a grid map. Two players defend a shared HQ over LAN. Enemies **dynamically re-route** whenever players build/upgrade/sell towers **during a wave**.

- **Victory**: Survive all waves with **Hearts > 0**.
- **Failure**: When an enemy reaches HQ, **Hearts** (e.g., start at 20) are reduced by that enemy’s value. **Hearts ≤ 0** = defeat.
- **Shared economy**: Kill rewards and end-of-wave rewards go to a **shared gold pool**.
- **Loadout limit**: Each player selects **2–3 tower types** before starting a level (can differ per player) to encourage complementary roles.
- **Placement rule**: Towers can only be placed on **buildable tiles**. The **server** validates that at least one **A\*** path from entry to HQ remains; otherwise placement is rejected.
- **Defense Towers (first prototype batch)**:  
  - Archer (low-cost single target)
  - Cannon (AoE)
  - Oil (slow/deceleration)
  - Tesla (chain lightning)
  - Buffer (ranged buff/debuff)
- **Enemies (first prototype batch)**:  
  - Trooper (basic)
  - Armored (high physical resist)
  - Warded (high magical resist)
  - Swarm (clustered)
  - Supporter / Enhancer (buffs/heals)
  - Elite / Champion (mini-boss)
- **Playability**: Real-time re-routing turns **blocking → diversion → focus fire** into live tactical play. Loadout limits push teammates to build **complementary tower sets**.

---

## AI Plan & FSM

**Flow:** `Spawn → Repath ↔ Move → { Goal | Dead }`

### States

#### Spawn
- Initialize unit (HP, speed, drops).
- Subscribe to **GridVersion** changes.
- **Transition:** → **Repath** immediately.

#### Repath
- Server computes path with **A\*** (Manhattan heuristic + small turn cost; cached).
- **On success:** → **Move**  
- **On failure:** retry after a short delay (server placement rules prevent permanent no-path).

#### Move
- Follow waypoints with speed/acceleration limits, corner smoothing, and light separation.
- **If tower changed:** → **Repath**

#### Goal
- **Regular enemies:** apply Hearts damage; despawn.  
- **Supply Convoy:** counts as *Escaped*; apply a **stacking buff** to future waves; despawn.

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

---

## Scripted Event

**Supply Convoy.** A convoy with elite guards spawns.

- **Condition:** Every **5** waves.  
- **Composition:** 1× Convoy (very tanky, slow, no Hearts damage, may have extra ability or mechanisms) + Guards (count scales with wave).  
- **Effect:** If destroyed, award **bonus gold** (scales with current wave). If escaped, subsequent waves’ enemies gain a **stacking “Well-Supplied” buff**.

---

## Multiplayer Plan

- **Session:** 2 players over **LAN**, **server-authoritative** (Host = Server; Client connects via LAN IP).
- **Shared:** Hearts / gold / waves / RNG seed.
- **Action flow:** Client sends **intent** (build/demolish/upgrade) → **server validates** (path remains open) and applies → server broadcasts grid/state (including **GridVersion**).
- **Waves:** Start when both players are **Ready** or a countdown ends. **During waves**, building/demolishing is allowed and **immediately** affects enemy paths.
- **Sync Objects:** Enemy transforms (snapshot interpolation), tower placements/levels, shared resources & waves, scripted events.

---

## Environment & Assets

- **Map:** URP top-down 3D; clear entry, HQ, buildable tiles, and path tiles.
- **UI:** Shared Hearts/gold/waves, two-player Ready, loadout selection, build hotkey bar.
- **Prototype assets:** Unity primitives, simple materials, placeholder SFX (to be replaced with Asset Store content).
- **Debug:** **F1** toggles grid, **A\*** paths, and on-unit FSM state text (for A2/A3).
