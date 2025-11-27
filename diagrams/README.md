# Pac-Man Game Design Diagrams

## Overview

This directory contains the complete set of UML and system design diagrams for the Pac-Man Unity game project. All diagrams adhere to SOLID principles and object-oriented programming best practices.

## Diagram Catalog

### **Structural Diagrams**

#### 1. class.mmd - Class Diagram

Provides the complete object-oriented class structure including all interfaces and their implementations.

**Key Features:**

- Interface-based design (IEntity, IMovement, IDirectionProvider, etc.)
- Strategy Pattern implementation for Ghost AI
- Dependency Inversion Principle (managers depend on interfaces)
- Clear separation between Unity MonoBehaviours and pure C# classes
- ScriptableObjects for data-driven design

**Main Components:**

- Core Interfaces (IEntity, IMovement, IEventBus, IPathfindingService, etc.)
- Managers (GameManager, LevelManager, ScoreManager)
- Entities (PacManController, GhostController, Pellet, PowerPellet)
- Movement System (GridMovement, IDirectionProvider implementations)
- AI System (GhostStateMachine, Strategy implementations)
- Services (PathfindingService, AudioService)

---

#### 2. components.mmd - Component Diagram

Illustrates the high-level system architecture and component relationships.

**Components:**

- Core Systems (GameManager, ScoreManager, LevelManager, EventBus, UI)
- Input System (InputReader)
- Entities (PacMan, Ghosts, Pellets)
- Movement System (GridMovement)
- AI System (GhostStateMachine, Strategies)
- Services (Pathfinding, Audio)
- Interface-based dependencies

---

#### 3. entity_relationship.mmd - Entity Relationship Diagram

Defines the data structure and relationships between game entities.

**Relationships:**

- GameManager → Level → Entities relationships
- Node grid structure
- Entity data attributes
- State relationships

---

#### 4. unity_deployment.mmd - Deployment Diagram

Documents the Unity-specific project structure and scene hierarchy.

**Structure:**

- Scene structure (MainMenu, GameScene)
- GameObject hierarchy
- Component attachments
- ScriptableObject assets
- Script organization

---

### **Behavioral Diagrams**

#### 5. activity.mmd - Activity Diagram

Describes the complete game flow from initialization to termination.

**Sequence:**

1. Game initialization (load level, spawn entities)
2. Main game loop
3. Pause system (Resume/Restart/Quit)
4. Victory condition (level completion)
5. Entity updates and movement
6. Collision detection and handling
7. Death and respawn logic
8. Game over conditions

---

#### 6. gamemanager_state.mmd - GameManager State Diagram

Documents all game state machine transitions and their triggers.

**States:**

- MainMenu → Playing
- Playing ↔ Paused
- Playing → LevelComplete → Playing (next level)
- Playing → PacManDied → Playing/GameOver
- Paused → MainMenu (quit)
- GameOver → MainMenu/Playing

---

#### 7. ghost.mmd - Ghost AI State Diagram

Defines the ghost behavior state machine and AI logic.

**States:**

- Scatter (patrol mode)
- Chase (hunt Pac-Man)
- Frightened (vulnerable after power pellet)
- Eaten (returning to ghost house)

**Transitions:**

- Scatter ↔ Chase (timed cycles)
- Any → Frightened (power pellet eaten)
- Frightened → Eaten (caught by Pac-Man)
- Eaten → Scatter (reached ghost house)

---

#### 8. pacman_state.mmd - Pac-Man State Diagram

Defines the Pac-Man entity state machine and behavioral transitions.

**States:**

- Spawning
- Normal (idle)
- Moving
- Eating (pellet)
- PoweredUp (after power pellet)
- ChaseGhost (eating frightened ghost)
- Dying
- Dead

---

#### 9. pacman.mmd - Pac-Man Update Flow

Provides a detailed flowchart of the Pac-Man update cycle.

**Process:**

1. Get direction from IDirectionProvider (InputReader)
2. Check if movement is valid (GridMovement.CanMoveTo)
3. Move to new position
4. Detect collisions
5. Publish appropriate events
6. Loop

---

### **Interaction Diagrams**

#### 10. pallet_eaten_sequence.mmd - Power Pellet Sequence

Documents the interaction sequence when Pac-Man consumes a power pellet.

**Participants:**

- PacMan
- Pellet (PowerPellet)
- EventBus
- ScoreManager
- GameManager
- Ghost

**Flow:**

1. PacMan collides with PowerPellet
2. PowerPellet publishes event via EventBus
3. ScoreManager adds bonus points
4. GameManager notifies all ghosts
5. Ghosts switch to Frightened state

---

#### 11. ghost_eaten_sequence.mmd - Ghost Eaten Sequence

Documents the interaction sequence when Pac-Man consumes a frightened ghost.

**Sequence:**

1. PacMan collides with frightened Ghost
2. Ghost publishes GhostEaten event
3. ScoreManager adds escalating points (200/400/800/1600)
4. GameManager commands ghost to return to house
5. Ghost enters Eaten state and navigates home
6. Ghost respawns in Scatter state

---

#### 12. game_start_sequence.mmd - Game Start Sequence

Documents the complete game initialization sequence.

**Sequence:**

1. MainMenuUI triggers StartGame
2. GameManager resets ScoreManager
3. LevelManager loads level data
4. LevelManager creates grid and pellets
5. Spawn Pac-Man with components
6. Spawn all ghosts with AI
7. Set game state to Playing
8. Update UI

---

### **Use Case Diagram**

#### 13. usecase.mmd - Use Case Diagram

Defines all player interactions with the game system.

**Use Cases:**

- Start Game
- Control Pac-Man Movement
- Pause/Resume/Restart/Quit Game
- View Score/Lives
- Eat Pellets/Power Pellets/Ghosts
- Advance to Next Level
- Navigate Menus

---

## Design Principles

### SOLID Principles

- **Single Responsibility Principle**: Each class maintains a single, well-defined purpose
- **Open/Closed Principle**: Strategy pattern enables extension without modification of existing code
- **Liskov Substitution Principle**: PowerPellet correctly extends Pellet base class
- **Interface Segregation Principle**: Interfaces are small and focused on specific contracts
- **Dependency Inversion Principle**: All dependencies utilize abstraction through interfaces

### Design Patterns

- **Strategy Pattern**: Implemented for Ghost AI behaviors (Chase, Scatter, Frightened)
- **Observer Pattern**: EventBus provides decoupled communication between components
- **State Pattern**: Applied to GameManager and Ghost state machines
- **Dependency Injection**: Services are injected via interfaces

### Coupling Strategy

- Entities communicate through EventBus rather than direct references
- Managers depend on interfaces rather than concrete implementations
- Movement logic is separated from entity logic
- AI logic is decoupled from movement implementation

---

## Implementation Roadmap

### Phase 1: Core Infrastructure

1. Interfaces (IEntity, IMovement, IEventBus, etc.)
2. EventBus implementation
3. GameManager with state machine
4. LevelManager and grid system

### Phase 2: Movement System

1. GridMovement class
2. IDirectionProvider interface
3. InputReader implementation

### Phase 3: Entities

1. Base Pellet class
2. PowerPellet (extends Pellet)
3. PacManController
4. GhostController

### Phase 4: AI System

1. IGhostStrategy interface
2. Strategy implementations (ScriptableObjects)
3. GhostStateMachine
4. GhostAI with pathfinding

### Phase 5: Services

1. PathfindingService (A* algorithm)
2. AudioService
3. ScoreManager

### Phase 6: UI

1. UIManager
2. PauseMenuUI
3. GameOverUI

---

## Technical Notes

- All diagrams utilize Mermaid syntax for compatibility with standard Markdown viewers
- Unity-specific annotations distinguish between MonoBehaviour, C# class, and ScriptableObject types
- Interface dependencies are explicitly marked throughout all diagrams
- Event-driven architecture facilitates testability and maintainability

---

## Diagram Dependencies

```bash
usecase.mmd ────> Drives requirements for ────> class.mmd
                                                    │
class.mmd ──────> Implements ──────────────────────┤
                                                    ▼
                                              components.mmd
                                                    │
                                                    ▼
                                           unity_deployment.mmd

activity.mmd ───> Defines flow for ─────> gamemanager_state.mmd
                                                    │
                                                    ├─> ghost.mmd
                                                    └─> pacman_state.mmd

pallet_eaten_sequence.mmd ──┐
ghost_eaten_sequence.mmd ───┼──> Detail interactions from ──> activity.mmd
game_start_sequence.mmd ────┘

entity_relationship.mmd ──> Defines data for ──> class.mmd
```

---

## Document Information

- Last Updated: November 27, 2025
- Project: CNAM Pac-Man Unity Game
- Architecture: SOLID Principles, Event-Driven Design, Data-Oriented Design
