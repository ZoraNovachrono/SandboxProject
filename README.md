# Technical Documentation — Sandbox Survival Game

Technical overview and architectural specifications for the Unreal Engine Survival/Sandbox project.

## 📋 Table of Contents

1. [Overall Project Architecture](#1-overall-project-architecture)

2. [Main Classes, Blueprints & Structures](#2-main-classes-blueprints--structures)

3. [Interaction & Inventory System](#3-interaction--inventory-system)

4. [Physics Systems](#4-physics-systems)

5. [Technical Choices & Roadmap](#5-technical-choices--roadmap)

## 1. Overall Project Architecture

The project is built on **Unreal Engine** using **Blueprints**, designed specifically as a single-player survival/sandbox game. The framework relies on a core data-driven flow:

**World Interaction** --> **Character Business Logic** --> **Inventory Data Struct Update** --> **UMG Interface Refresh**

```
 ┌─────────────────────────┐
 │   BP_GrabItem (World)   │
 └────────────┬────────────┘
              │ (Line Trace Interaction)
              ▼
 ┌─────────────────────────┐         ┌─────────────────────────┐
 │ BP_FirstPersonCharacter ├────────►│  DT_Items (S_ItemData)  │
 └────────────┬────────────┘         └─────────────────────────┘
              │ (Data Modification)
              ▼
 ┌─────────────────────────┐
 │   WBP_Inventory (UMG)   │
 └─────────────────────────┘

```

* **Core Logic Hub:** Centralized within `BP_FirstPersonCharacter`.

* **Data-Driven Architecture:** All item statistics and attributes are centralized in a primary `DataTable` powered by the `S_ItemData` structure.

* **UI Synchronization:** UMG Widgets dynamically observe and render the inventory array (`S_InventorySlot`) stored on the character.

## 2. Main Classes, Blueprints & Structures

### Blueprints & Actors

| **Asset** | **Type** | **Description** | 
| `BP_FirstPersonCharacter` | Character | Handles camera, interaction raycasting, inventory management logic (stacking, slotting), and UMG events. | 
| `BP_GrabItem` | Actor | Physical world representation of pickable items. Utilizes a `StaticMeshComponent` with physics simulation enabled. | 
| `WBP_Inventory` | User Widget | Main inventory interface container displaying the item grid. | 
| `WBP_InventorySlot` | User Widget | Individual grid slot responsible for icon rendering, stack counts, and drag-and-drop operations. | 

### Data Structures & Tables

#### `S_ItemData` (Item Definitions)

```
struct S_ItemData {
    FName ItemID;                 // Unique identifier / Row Name
    FText DisplayName;            // In-game localized name
    UTexture2D* Icon;             // UMG UI icon asset
    int32 MaxStack;               // Maximum allowed quantity per slot
    TSubclassOf<AActor> WorldActorClass; // 3D Actor class spawned when dropped
};

```

#### `S_InventorySlot` (Slot Container)

```
struct S_InventorySlot {
    FDataTableRowHandle ItemRow;  // Reference to DT_Items entry
    int32 Quantity;               // Current amount contained in this slot
};

```

#### `DT_Items` (Data Table)

Data Table asset populated using the `S_ItemData` structure.

## 3. Interaction & Inventory System

### A. Detection Phase

1. `BP_FirstPersonCharacter` executes a `LineTraceByChannel` on the interaction frame originating from the camera component.

2. If the trace hits an actor of class `BP_GrabItem`, interaction context becomes available.

### B. Item Acquisition Algorithm (`Grab Item`)

When the user triggers the interaction input, the inventory system follows this decision tree:

```
                  [Trigger Interaction]
                            │
                            ▼
           [Search existing matching item in Inventory]
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
          (YES)                            (NO)
            │                               │
[Is slot stackable?]                        │
(Quantity < MaxStack)                       │
    │         │                             │
    │ (YES)   └───────────────┐             │
    ▼                         ▼             ▼
[Stack Quantity]           (NO) ──► [Search Empty Slot]
    │                                       │
[Remaining items?]                          ├──► (FOUND) ──► [Assign to Slot] ──► [Destroy World Actor]
    │                                       │
    ├──► (NO)  ──► [Destroy World Actor]    └──► (NOT FOUND) ──► [Cancel Grab / Leave in World]
    │
    └──► (YES) ──► [Loop process with remaining count]

```

## 4. Physics Systems

* **Physics Engine:** Unreal Engine standard physics engine (PhysX / Chaos Physics).

* **`BP_GrabItem` Configuration:**

  * `Simulate Physics`: **Enabled**

  * `Collision Profile`: `PhysicsActor` / `WorldDynamic`

* **Item Dropping Mechanics:**

  1. Retrieves `WorldActorClass` from `S_ItemData` associated with the dropped slot.

  2. Spawns the actor at character location with a forward offset vector.

  3. Applies an impulse vector to push the item naturally away from the character.

## 5. Technical Choices & Roadmap

### Technical Design Choices

* **Single-Player Focus:** Networking (`Replication` / `RPC`) is omitted at this stage to maximize iteration speed.

* **Monolithic Prototyping:** Core inventory state resides directly in `BP_FirstPersonCharacter` for rapid feature validation.

* **Data-Driven Workflow:** Separation of data (`DataTable`) and logic ensures seamless addition of new resources without modifying Blueprint code.

### Development Roadmap

```
[Phase 1: Core System] ─────────► [Phase 2: Drop System] ─────────► [Phase 3: Resource Harvesting] ─────────► [Phase 4: Construction]
(Completed)                      (In Progress)                      (Upcoming)                             (Planned)
- Line Trace Interaction         - Inventory Drop Execution         - Resource Nodes (Trees/Rocks)          - Ghost Mesh Preview
- Stacking & Inventory Logic     - World Spawning                   - Tool Interaction                     - Snapping System (Ark/Valheim)
- UMG Grid Interface             - Physics Impulse                  - Loot Table Drops                     - Structural Integrity

This page has been generated with AI and myself
```
