# Pac-Man Game Design Diagrams

## Overview

This directory contains the UML and system-design diagrams for the **CNAM Pac-Man Unity
game** (`Project/Assets/Scripts`). The diagrams describe the code as it actually exists:
a Unity MonoBehaviour-based game with a singleton `GameManager`, component-based ghost AI,
a static settings/difficulty layer, an in-game **Map Editor**, and an **ML-Agents**
autopilot/training mode.

All diagrams use [Mermaid](https://mermaid.js.org/) syntax (`.mmd`).

## Architecture at a glance

- **No EventBus / dependency-injection container.** Communication is via the `GameManager`
  singleton (`GameManager.Instance`), the `IGameEvents` interface it implements, and a few
  public `System.Action` events that the ML agent subscribes to.
- **No `LevelManager` / `PathfindingService`.** Levels are Unity scenes/tilemaps. Ghosts
  navigate with local decisions at `Node` trigger points (greedy direction choice), not A*.
- **Ghost AI is component-based, not Strategy-ScriptableObjects.** Each ghost is a `Ghost`
  with several `GhostBehavior` MonoBehaviours (`GhostScatter`, a chase variant,
  `GhostFrightened`, `GhostHome`) that enable/disable each other on timers.
- **Difficulty/config is a static layer** (`GameSettings`) written by `DifficultyModal` and
  applied by `GameSettingsApplier` and `GameManager`.

## Diagram Catalog

### Structural

| File | Description |
| --- | --- |
| `class.mmd` | Core class diagram: `GameManager` (+ `ScoreManager`/`LivesManager`/`UIManager`), `Pacman`, `Movement`/`IMovable`, `Pellet`/`PowerPellet`, `Node`, `Ghost` + `GhostBehavior` hierarchy, `GameSettings`, interfaces (`IGameEvents`, `IMovable`, `IBehavior`, `IDurableBehavior`). |
| `components.mmd` | High-level subsystems and wiring: Core, Entities, Ghost AI, Settings & Difficulty, Menus, Map Editor, ML. |
| `entity_relationship.mmd` | Data/ownership relationships between manager, entities, ghosts, settings, and map data. |
| `unity_deployment.mmd` | Scenes (`PageAccueil`, `ChoixTypeJeu`, `MapEditor`, `Pacman`, `Pacman_Autopilot`, `Pacman_Training`), the Pacman scene hierarchy, prefabs, `Scripts/` folders, and data assets (`.onnx`, map JSON). |
| `mapeditor.mmd` | Map Editor subsystem: `MapData`/`CellType`, static helpers (`MapCoordinates`, `BoundaryPlacer`, `GhostHouseGenerator`, `NodeGenerator`, `MapValidator`), persistence (`MapRepository`/`MapSession`/`MapDataManager`), and controllers (`MapEditorManager`, `MapEditorUI`, `MapLoader`). |
| `ml_training.mmd` | ML-Agents subsystem: `PacmanAgent`, `PacmanRewardHandler`, `TrainingManager` and its helpers (`TrainingModeDetector`, `AgentActivator`, `TimeScaleController`, `TrainingUIController`). |

### Behavioral

| File | Description |
| --- | --- |
| `activity.mmd` | Full game flow from scene load (Awake/Start/NewRound) through the Unity loop, collisions, scoring, death/respawn, round completion, and pause. |
| `gamemanager_state.mmd` | Implicit `GameManager` phases (Booting → Playing → PacManDied / RoundComplete / GameOver) driven by method calls + `Invoke` delays, plus the `PauseManager` pause state and training-mode variant. |
| `ghost.mmd` | Component-based ghost behaviour FSM: `Home → Scatter ↔ Chase`, `→ Frightened` on power pellet, `→ Eaten → Home`, implemented by enabling/disabling `GhostBehavior` components on timers. |
| `pacman_state.mmd` | Pac-Man `Inactive ↔ Active (Controlled/AI) → Dead` lifecycle via `ResetState()` / `DeathSequence()`. |
| `pacman.mmd` | Per-frame update flow: input (or `PacmanAgent`) → `Movement.SetDirection`/`Occupied` BoxCast → `FixedUpdate` move → Unity triggers/collisions. |

### Interaction (sequence)

| File | Description |
| --- | --- |
| `pallet_eaten_sequence.mmd` | Power-pellet consumption: `PowerPellet.Eat()` → `GameManager.PowerPelletEaten` → enable `GhostFrightened` on all ghosts, score, multiplier-reset timer. |
| `ghost_eaten_sequence.mmd` | Eating a frightened ghost: `GhostFrightened.Eaten()` → teleport to home + `GhostHome.Enable` → `GameManager.GhostEaten` (escalating `points * ghostMultiplier`). |
| `game_start_sequence.mmd` | From `DifficultyModal` (settings + `MapSession`) through the ordered scene boot: `MapLoader (-200)` → `GameSettingsApplier (-150)` → `GameManager (-100)` → first round. |

### Use Case

| File | Description |
| --- | --- |
| `usecase.mmd` | Player and developer use cases: menus, difficulty/ghost/autopilot config, map editing, gameplay, pause, and ML training/inference. |

## Key Implementation Notes

- **Singleton + interface events.** `GameManager` is a `[DefaultExecutionOrder(-100)]`
  singleton implementing `IGameEvents`. `Pellet`/`PowerPellet` call
  `GameManager.Instance.PelletEaten/PowerPelletEaten`; `Ghost` calls `GhostEaten`/`PacmanEaten`.
- **`IMovable` / `Movement`.** `Movement` (on Pac-Man and ghosts) owns speed, direction, and
  obstacle checks (`Physics2D.BoxCast` against `obstacleLayer`). AI behaviours read
  `IMovable.direction`.
- **`IDurableBehavior` / `GhostBehavior`.** Behaviours `Enable(duration)` themselves and
  `Invoke(Disable, duration)`; each `OnDisable` hands control to the next behaviour.
- **Chase variants** (auto-detected into `Ghost.chase` in `Awake`):
  `GhostChase` (Blinky, direct), `GhostChaseAmbusher` (Pinky, look-ahead),
  `GhostChaseBoo` (Inky, reverse-on-approach), `GhostChaseFlanker` (Clyde, pivot off Blinky).
- **Endless rounds.** Clearing all pellets increments `currentRound`; speeds ramp via
  `GameSettings.ComputeGhostSpeedForRound` / `ComputePacManSpeedForRound` (exponential curve).
- **Difficulty layer.** `DifficultyModal` writes `GameSettings`; `GameSettingsApplier`
  applies speeds, power-pellet duration, and per-ghost exit delays; disabled ghosts are
  filtered out in `GameManager.FilterEnabledGhosts`.
- **Custom maps.** `MapData` (+ `CellType`) is painted in the Map Editor, validated by
  `MapValidator`, persisted by `MapRepository`, carried across scenes by `MapSession`, and
  rebuilt into the Pacman scene by `MapLoader` (`-200`, before everything else).
- **ML / Autopilot.** `PacmanAgent` (Unity ML-Agents `Agent`) drives `Movement` when
  `Pacman.aiControlled`; `TrainingManager` wires detection, activation, time-scale, and UI.
  Rewards come from `PacmanRewardHandler`, fed by `GameManager` events.

## Interfaces

| Interface | Implemented by | Purpose |
| --- | --- | --- |
| `IGameEvents` | `GameManager` | Entry points for pellet/ghost/Pac-Man events. |
| `IMovable` | `Movement` | Exposes `direction` + `speedMultiplier` to ghost AI. |
| `IBehavior` | (`GhostBehavior` via `IDurableBehavior`) | `Enable()` / `Disable()`. |
| `IDurableBehavior` | `GhostBehavior` | Adds timed `Enable(duration)` + `duration`. |

## Document Information

- Project: CNAM Pac-Man Unity Game (`Project/`)
- Diagrams updated to match source under `Project/Assets/Scripts` as of 2026-05-31.
- Architecture: Unity MonoBehaviour components, singleton manager, component-based AI,
  static configuration, plus Map Editor and ML-Agents subsystems.
