# Architecture & Code Flow Documentation

## Table of Contents
- [Overview](#overview)
- [Project Structure](#project-structure)
- [Code Flow](#code-flow)
- [Core Systems](#core-systems)
- [Design Patterns](#design-patterns)
- [Game Architecture](#game-architecture)

---

## Overview

This is a 2D procedurally-generated dungeon RPG built in Unity using C#. The game features:
- Binary Space Partitioning (BSP) dungeon generation
- A* pathfinding for enemy AI
- Object pooling for performance optimization
- Singleton pattern for centralized game state management

**Technology Stack:**
- Unity 2022.3+
- C# 9.0+
- Unity 2D Feature Set
- TextMesh Pro for UI
- Cinemachine for camera control

**Important Note:** This project does NOT use Lua scripting. All game logic is implemented in C#. Extension points exist through ScriptableObjects and prefab configurations.

---

## Project Structure

```
/dungeon-adventure-rpg-2d/
├── Assets/
│   ├── _Scripts/                    # All C# game logic (41 files)
│   │   ├── Player/                  # Player-related scripts
│   │   ├── Enemy/                   # Enemy and boss AI
│   │   ├── ProceduralGeneration/    # Dungeon generation algorithms
│   │   ├── Map/                     # Room and corridor generation
│   │   ├── Pathfinfing/             # A*, Dijkstra, Bellman-Ford
│   │   ├── UI/                      # Menu and HUD systems
│   │   ├── Data/                    # ScriptableObject definitions
│   │   └── [Various systems]        # Coins, potions, walls, etc.
│   │
│   ├── Scenes/                      # Unity scenes (4 total)
│   │   ├── Main Menu.unity          # Scene 0 - Title screen
│   │   ├── SampleScene.unity        # Scene 1 - Main gameplay
│   │   ├── SampleScene 2.unity      # Debug scene
│   │   └── Victory Scene.unity      # Scene 2 - Victory screen
│   │
│   ├── Prefabs/                     # Game object prefabs (19 total)
│   │   ├── Player.prefab
│   │   ├── Boss.prefab
│   │   ├── Enemy1/2/3.prefab
│   │   ├── Bullet.prefab
│   │   ├── Coin.prefab
│   │   ├── HealthPotion.prefab
│   │   └── [etc.]
│   │
│   ├── Tiles/                       # Tileset assets
│   │   ├── Floor/
│   │   └── Wall/                    # 12 wall variants for auto-tiling
│   │
│   ├── my_sprites/                  # Sprite assets
│   │   ├── basic asset pack/       # Enemy animations
│   │   ├── Kyrise's icons/         # UI icons
│   │   ├── DungeonTileset/         # Background tiles
│   │   ├── characters/             # Character sprites
│   │   └── freefantasygui/         # UI graphics
│   │
│   ├── CodeMonkey/                  # Third-party utilities
│   ├── TextMesh Pro/                # Text rendering
│   └── Editor/                      # Custom Unity editor tools
│
├── Packages/                        # Unity package dependencies
│   ├── manifest.json
│   └── packages-lock.json
│
└── README.md                        # Project documentation
```

---

## Code Flow

### 1. Game Initialization Flow

```
[Unity Scene Load]
        ↓
[Main.cs - Start()]
        ↓
[RoomFirstDungeonGenerator.playRunProceduralGeneration()]
        ↓
    ┌───────────────────────────────────┐
    │  Binary Space Partitioning (BSP)  │
    │  Create room rectangles           │
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Generate Simple Rooms            │
    │  Create floor tiles               │
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Connect Rooms with Corridors     │
    │  L-shaped paths (closest neighbor)│
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Generate Walls                   │
    │  Auto-tile based on neighbors     │
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Place Player                     │
    │  First room in generation list    │
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Place Boss                       │
    │  Farthest room from player        │
    └───────────────┬───────────────────┘
                    ↓
    ┌───────────────────────────────────┐
    │  Generate Entities                │
    │  - 5 coins per room               │
    │  - 3 enemies per room             │
    │  - 2 health potions per room      │
    │  - 5 props per room               │
    └───────────────┬───────────────────┘
                    ↓
        [Game Ready to Play]
```

### 2. Game Loop Flow

```
[Player Input] ───────────────┐
    ↓                         │
[PlayerMovement.cs]           │
    - WASD/Arrow keys         │
    - Animation updates       │
    - Hitbox positioning      │
    ↓                         │
[PlayerAimWeapon.cs]          │
    - Mouse position tracking │
    - Gun rotation            │
    - Bullet firing (pooled)  │
    ↓                         │
[Bullet Physics] ─────────────┤
    - Layer mask collision    │
    - IDamageable interface   │
    - Return to pool          │
                              │
[Enemy AI Loop] ──────────────┤
    ↓                         │
[EnemyScript.cs]              │
    - Find player position    │
    - Calculate A* path       │
    - Move along path         │
    - Update every 0.5s       │
    ↓                         │
[Collision Detection] ────────┤
    - Player vs Enemy         │
    - Bullet vs Enemy         │
    - Player vs Items         │
                              │
[GameManager.cs] ←────────────┘
    - Track score
    - Manage health potions
    - Handle pause/resume
    - Trigger scene transitions
```

### 3. Combat Flow

```
[Player Fires Bullet]
        ↓
[ObjectPool provides Bullet instance]
        ↓
[Bullet.OnTriggerEnter2D()]
        ↓
[Check if hit.gameObject has IDamageable]
        ↓
    ┌───YES───┐
    ↓         ↓
[Enemy]   [Boss]
    ↓         ↓
[TakeDamage(30)]
    ↓
[Health reduced by 30]
    ↓
[Update HealthBarEnemy]
    - Animate health drain (0.25s)
    - Update color gradient
    ↓
[Health <= 0?]
    ↓
┌───YES───┬───NO───┐
↓         ↓        ↓
[Boss]  [Enemy]  [Continue]
↓         ↓
[Victory] [Destroy]
[Scene]   [Spawn Effect]
```

### 4. Pathfinding Flow (A* Algorithm)

```
[Enemy needs path to player]
        ↓
[Pathfinding.FindPath(startNode, targetNode)]
        ↓
[Initialize open set (PriorityQueue) and closed set]
        ↓
    ┌─────────────┐
    │ MAIN LOOP   │
    └──────┬──────┘
           ↓
[Get node with lowest F-cost from open set]
           ↓
[Is this the target node?]
           ↓
    ┌──YES──┬──NO───┐
    ↓       ↓       ↓
[Retrace] [Add to closed set]
[Path]          ↓
    ↓    [Get walkable neighbors]
    │            ↓
    │    [For each neighbor...]
    │            ↓
    │    [Calculate new G-cost]
    │            ↓
    │    [Is new path better OR neighbor not in open set?]
    │            ↓
    │        ┌──YES──┬──NO───┐
    │        ↓       ↓       ↓
    │    [Update]  [Skip]  [Continue]
    │    [costs]            ↓
    │        ↓          [Next neighbor]
    │    [Set parent]        │
    │        ↓               │
    │    [Add to open]       │
    │        └───────────────┘
    │                ↓
    └────[Loop until target found or open set empty]
                     ↓
            [Return List<Node> path]
```

---

## Core Systems

### 1. Player System

**Location:** `/Assets/_Scripts/Player/`

**Components:**

| Component | File | Responsibility |
|-----------|------|---------------|
| **PlayerMovement** | `PlayerMovement.cs` | Movement, animation, input handling |
| **PlayerHealth** | `PlayerHealth.cs` | Health management, damage, healing |
| **PlayerScript** | `PlayerScript.cs` | Collision damage to enemies |
| **PlayerAimWeapon** | `PlayerAimWeapon.cs` | Mouse-aimed weapon, bullet firing |
| **HealthBar** | `HealthBar.cs` | UI health display |

**Key Parameters:**
```csharp
// PlayerMovement
public static float pspeed = 25f;  // Movement speed

// PlayerHealth
public int maxhealth = 200;        // Maximum HP
public int health = 200;           // Current HP
enemyDamage = 10;                  // Damage from regular enemies
bossDamage = 20;                   // Damage from boss

// PlayerAimWeapon
public Transform firePoint;        // Bullet spawn location
public float bulletSpeed = 20f;    // Bullet velocity
public float fireRate = 0.5f;      // Time between shots
```

**Movement System:**
- Uses Unity's Input system (`Input.GetAxis("Horizontal")` / `"Vertical"`)
- 8-directional movement (cardinal + diagonals)
- Animator parameters: `IsRunning`, `IsUp`, `IsDown`, `IsAttacking`
- Sprite flipping for left/right direction
- Hitbox offset positioning based on facing direction

**Combat System:**
- Mouse-aimed weapon rotation
- Bullet pooling via `ObjectPool<GameObject>`
- Collision-based damage using `IDamageable` interface
- Direct collision damage to enemies (30 HP)

### 2. Enemy System

**Location:** `/Assets/_Scripts/Enemy/`

**Enemy Types:**
1. **Regular Enemies** (Enemy1, Enemy2, Enemy3)
   - Speed: 3 units/sec
   - Max Health: 100 HP
   - Uses A* pathfinding
   - Path updates every 0.5 seconds
   - Knockback support (5 force, 0.2s duration)

2. **Boss**
   - Speed: 5 units/sec
   - Max Health: 500 HP
   - Direct chase (no pathfinding)
   - Triggers victory screen on death

**EnemyScript.cs Key Code:**
```csharp
// Pathfinding initialization
void Start()
{
    pathfinding = GetComponent<Pathfinding>();
    InvokeRepeating("UpdatePath", 0f, 0.5f);
}

// A* path calculation
void UpdatePath()
{
    if (player != null)
    {
        pathfinding.targetPos = player.position;
    }
}

// Movement along path
void FollowPath()
{
    if (pathfinding.path != null && currentNode < pathfinding.path.Count)
    {
        Vector2 direction = (pathfinding.path[currentNode].WorldPosition -
                           (Vector2)transform.position).normalized;
        transform.position = Vector2.MoveTowards(transform.position,
                           pathfinding.path[currentNode].WorldPosition,
                           speed * Time.deltaTime);
    }
}
```

### 3. Procedural Generation System

**Location:** `/Assets/_Scripts/ProceduralGeneration/` & `/Assets/_Scripts/Map/`

**Generation Hierarchy:**
```
AbstractDungeonGenerator (base class)
    ├── SimpleRandomWalkDungeonGenerator
    │   ├── RoomFirstDungeonGenerator ★ PRIMARY
    │   └── CorridorFirstDungeonGenerator
```

**RoomFirstDungeonGenerator Algorithm:**

```csharp
// Phase 1: Binary Space Partitioning
List<BoundsInt> roomBounds = ProceduralGenerationAlgorithms.BinarySpacePartitioning(
    new BoundsInt((Vector3Int)startPosition,
    new Vector3Int(dungeonWidth, dungeonHeight, 0)),
    minRoomWidth,
    minRoomHeight
);

// Phase 2: Create simple rooms
List<Vector2Int> floor = new HashSet<Vector2Int>();
foreach (var roomBound in roomBounds)
{
    roomCenters.Add((Vector2Int)Vector3Int.RoundToInt(roomBound.center));
    HashSet<Vector2Int> newRoomFloor = CreateSimpleRoom(roomBound);
    floor.UnionWith(newRoomFloor);
}

// Phase 3: Connect rooms with corridors
floor.UnionWith(ConnectRooms(roomCenters));

// Phase 4: Generate walls
WallGenerator.CreateWalls(floor, tilemapVisualizer);

// Phase 5: Place entities
PlacePlayer(roomBounds[0]);
PlaceBossInFarthestRoom(roomBounds);
GenerateCoins(roomBounds);
GenerateEnemies(roomBounds);
GenerateHealthPotions(roomBounds);
GenerateProps(roomBounds);
```

**Room Connection Algorithm:**
```csharp
private HashSet<Vector2Int> ConnectRooms(List<Vector2Int> roomCenters)
{
    HashSet<Vector2Int> corridors = new HashSet<Vector2Int>();
    Vector2Int currentRoom = roomCenters[Random.Range(0, roomCenters.Count)];
    roomCenters.Remove(currentRoom);

    while (roomCenters.Count > 0)
    {
        Vector2Int closest = FindClosestPoint(currentRoom, roomCenters);
        roomCenters.Remove(closest);

        // Create L-shaped corridor
        HashSet<Vector2Int> corridor = CreateCorridor(currentRoom, closest);
        corridors.UnionWith(corridor);

        currentRoom = closest;
    }

    return corridors;
}
```

**Entity Placement:**
- Uses `Physics2D.OverlapPointAll()` to avoid overlaps
- Checks for existing entities with tags: "Player", "Enemy", "Boss", "Coin", "Prop"
- Random position selection from valid floor tiles
- Configurable spawn counts per room:
  - Coins: 5 per room
  - Enemies: 3 per room (rotates through Enemy1/2/3)
  - Health Potions: 2 per room
  - Props: 5 per room (minimum 1.5 unit spacing)

### 4. Pathfinding System

**Location:** `/Assets/_Scripts/Pathfinfing/`

**Key Components:**

| Component | File | Purpose |
|-----------|------|---------|
| **Pathfinding** | `Pathfinding.cs` | A* implementation, path calculation |
| **GridMap** | `GridMap.cs` | Tilemap-to-grid conversion, node management |
| **Node** | `Node.cs` | Grid cell data structure |
| **PriorityQueue** | `PriorityQueue.cs` | Min-heap for A* open set |

**A* Algorithm Details:**
```csharp
public List<Node> FindPath(Node startNode, Node targetNode)
{
    PriorityQueue<Node> openSet = new PriorityQueue<Node>();
    HashSet<Node> closedSet = new HashSet<Node>();

    openSet.Enqueue(startNode, startNode.FCost);

    while (openSet.Count > 0)
    {
        Node currentNode = openSet.Dequeue();
        closedSet.Add(currentNode);

        if (currentNode == targetNode)
        {
            return RetracePath(startNode, targetNode);
        }

        foreach (Node neighbor in gridMap.GetNeighbours(currentNode))
        {
            if (!neighbor.IsWalkable || closedSet.Contains(neighbor))
                continue;

            int newGCost = currentNode.GCost + GetDistance(currentNode, neighbor);

            if (newGCost < neighbor.GCost || !openSet.Contains(neighbor))
            {
                neighbor.GCost = newGCost;
                neighbor.HCost = GetDistance(neighbor, targetNode);
                neighbor.Parent = currentNode;

                if (!openSet.Contains(neighbor))
                    openSet.Enqueue(neighbor, neighbor.FCost);
            }
        }
    }

    return null; // No path found
}
```

**Distance Heuristic:**
```csharp
int GetDistance(Node nodeA, Node nodeB)
{
    int distX = Mathf.Abs(nodeA.GridPosition.x - nodeB.GridPosition.x);
    int distY = Mathf.Abs(nodeA.GridPosition.y - nodeB.GridPosition.y);

    // Diagonal distance
    if (distX > distY)
        return 14 * distY + 10 * (distX - distY);
    return 14 * distX + 10 * (distY - distX);
}
```

**GridMap Conversion:**
```csharp
// Converts Unity Tilemap to pathfinding grid
public void InitializeGrid(Tilemap tilemap)
{
    BoundsInt bounds = tilemap.cellBounds;
    nodes = new Node[bounds.size.x, bounds.size.y];

    for (int x = 0; x < bounds.size.x; x++)
    {
        for (int y = 0; y < bounds.size.y; y++)
        {
            Vector3Int cellPosition = new Vector3Int(
                bounds.xMin + x,
                bounds.yMin + y,
                0
            );

            bool isWalkable = tilemap.HasTile(cellPosition);
            Vector3 worldPosition = tilemap.CellToWorld(cellPosition);

            nodes[x, y] = new Node(
                isWalkable,
                worldPosition,
                new Vector2Int(x, y)
            );
        }
    }
}
```

### 5. UI & Game State System

**Location:** `/Assets/_Scripts/UI/`

**GameManager.cs (Singleton Pattern):**
```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    public int score = 0;
    public int healthPotion = 0;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void PauseGame()
    {
        Time.timeScale = 0f;
        pauseMenuUI.SetActive(true);
    }

    public void ResumeGame()
    {
        Time.timeScale = 1f;
        pauseMenuUI.SetActive(false);
    }

    public void LoadScene(int sceneIndex)
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene(sceneIndex);
    }
}
```

**Scene Flow:**
- Scene 0: Main Menu → Scene 1 (Play button)
- Scene 1: Gameplay → Scene 2 (Boss defeated) or Game Over
- Scene 2: Victory Screen → Scene 0 (Menu button)

### 6. Object Pooling System

**Location:** `/Assets/_Scripts/ObjectPool.cs`

**Usage in PlayerAimWeapon:**
```csharp
private ObjectPool<GameObject> bulletPool;

void Start()
{
    bulletPool = new ObjectPool<GameObject>(
        () => Instantiate(bulletPrefab),
        bullet => bullet.SetActive(true),
        bullet => bullet.SetActive(false),
        bullet => Destroy(bullet),
        false,
        initialPoolSize: 20,
        maxSize: 100
    );
}

void Shoot()
{
    GameObject bullet = bulletPool.Get();
    bullet.transform.position = firePoint.position;
    bullet.transform.rotation = firePoint.rotation;

    Rigidbody2D rb = bullet.GetComponent<Rigidbody2D>();
    rb.velocity = firePoint.up * bulletSpeed;
}

// In Bullet.cs
void OnTriggerEnter2D(Collider2D hitInfo)
{
    // ... damage logic ...
    gameObject.SetActive(false); // Returns to pool
}
```

---

## Design Patterns

### 1. Singleton Pattern
**Used in:** `GameManager.cs`

**Purpose:** Centralized game state management accessible from anywhere

**Implementation:**
```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }
}

// Usage from anywhere:
GameManager.instance.score += 1;
```

### 2. Object Pool Pattern
**Used in:** `ObjectPool.cs`, `PlayerAimWeapon.cs`

**Purpose:** Reduce garbage collection overhead by reusing objects

**Benefits:**
- No allocation/deallocation overhead during gameplay
- Consistent performance (no GC spikes)
- Configurable pool size limits

### 3. Interface Pattern
**Used in:** `IDamageable.cs`

**Purpose:** Polymorphic damage handling for different entity types

**Implementation:**
```csharp
public interface IDamageable
{
    void TakeDamage(int damage);
}

public class EnemyScript : MonoBehaviour, IDamageable
{
    public void TakeDamage(int damage)
    {
        health -= damage;
        if (health <= 0) Die();
    }
}

// In Bullet.cs
IDamageable damageable = hitInfo.GetComponent<IDamageable>();
if (damageable != null)
{
    damageable.TakeDamage(30);
}
```

### 4. Template Method Pattern
**Used in:** `AbstractDungeonGenerator.cs`

**Purpose:** Define generation skeleton, allow subclasses to implement specific steps

**Implementation:**
```csharp
public abstract class AbstractDungeonGenerator : MonoBehaviour
{
    // Template method
    public void GenerateDungeon()
    {
        tilemapVisualizer.Clear();
        RunProceduralGeneration();
    }

    // Subclasses must implement
    protected abstract void RunProceduralGeneration();
}

public class RoomFirstDungeonGenerator : SimpleRandomWalkDungeonGenerator
{
    protected override void RunProceduralGeneration()
    {
        // Specific implementation for room-first generation
    }
}
```

### 5. ScriptableObject Pattern
**Used in:** `SimpleRandomWalkSO.cs`

**Purpose:** Data-driven configuration without code changes

**Implementation:**
```csharp
[CreateAssetMenu(fileName ="SimpleRandomWalkParameters",
                 menuName = "PCG/SimpleRandomWalkData")]
public class SimpleRandomWalkSO : ScriptableObject
{
    public int iterations = 10;
    public int walkLength = 10;
    public bool startRandomlyEachIteration = true;
}
```

### 6. Factory Pattern
**Used in:** Entity generation classes

**Purpose:** Centralized object creation with configuration

**Example:**
```csharp
public class EnemyGenerator : MonoBehaviour
{
    public void GenerateEnemies(List<BoundsInt> roomBounds)
    {
        foreach (var roomBound in roomBounds)
        {
            for (int i = 0; i < enemiesPerRoom; i++)
            {
                // Factory logic for enemy creation
                GameObject enemy = Instantiate(
                    enemyPrefabs[enemyIndex % 3],
                    position,
                    Quaternion.identity
                );
            }
        }
    }
}
```

---

## Game Architecture

### High-Level System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Unity Engine                             │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌───────────┐ │
│  │  Physics2D │  │ Input Sys  │  │  Animator  │  │  Tilemap  │ │
│  └────────────┘  └────────────┘  └────────────┘  └───────────┘ │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────┴─────────────────────────────────────┐
│                      Game Systems Layer                          │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              GameManager (Singleton)                     │   │
│  │  - Score tracking                                        │   │
│  │  - Health potion inventory                               │   │
│  │  - Pause/Resume                                          │   │
│  │  - Scene transitions                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │  Player System  │  │  Enemy System   │  │  UI System     │  │
│  │  - Movement     │  │  - AI           │  │  - Menus       │  │
│  │  - Combat       │  │  - Pathfinding  │  │  - HUD         │  │
│  │  - Health       │  │  - Boss         │  │  - Health bars │  │
│  └─────────────────┘  └─────────────────┘  └────────────────┘  │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │         Procedural Generation System                    │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │ Room Gen     │→ │ Corridor Gen │→ │ Wall Gen     │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │ Entity Gen   │  │ Item Gen     │  │ Prop Gen     │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │         Pathfinding System                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │    │
│  │  │ A* Algorithm │  │ Grid Map     │  │ Node System  │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │         Utility Systems                                 │    │
│  │  - Object Pooling                                       │    │
│  │  - IDamageable Interface                                │    │
│  │  - ScriptableObject Configs                             │    │
│  └─────────────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────────────┘
```

### Data Flow Diagram

```
[User Input]
     ↓
[Input System] ─────→ [Player Movement] ─────→ [Animation System]
     │                      ↓
     │                [Collision Detection]
     │                      ↓
     ├──────────→ [Combat System] ←──────────┐
     │                ↓                       │
     │           [Bullet Pool]                │
     │                ↓                       │
     │          [IDamageable]                 │
     │                ↓                       │
     │          [Enemy Health] ───────────────┘
     │                ↓
     │          [Death Handler]
     │                ↓
     └─────→ [GameManager] ←─────────────────────┐
                   ↓                              │
            [Score/Inventory]                     │
                   ↓                              │
            [UI Updates]                          │
                   ↓                              │
            [Scene Manager] ───→ [Victory/GameOver]


[Dungeon Generation]
     ↓
[BSP Algorithm] ───→ [Room Creation] ───→ [Corridor Creation]
     ↓                     ↓
[Grid Map]           [Wall Generation]
     ↓                     ↓
[Pathfinding]        [Entity Placement]
     ↓                     ↓
[Enemy AI]           [Item Spawning]
```

### Component Communication

**1. Player → GameManager:**
```csharp
// Coin collection
GameManager.instance.score += coinValue;

// Health potion inventory
GameManager.instance.healthPotion += 1;
```

**2. Enemy → Player:**
```csharp
// Collision damage
void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.CompareTag("Player"))
    {
        PlayerHealth playerHealth = collision.gameObject.GetComponent<PlayerHealth>();
        playerHealth.TakeDamage(10); // Regular enemy
    }
}
```

**3. Pathfinding → Enemy:**
```csharp
// Enemy uses pathfinding component
private Pathfinding pathfinding;

void Start()
{
    pathfinding = GetComponent<Pathfinding>();
    InvokeRepeating("UpdatePath", 0f, 0.5f);
}

void UpdatePath()
{
    pathfinding.targetPos = player.position;
}
```

**4. Bullet → Enemy:**
```csharp
// Interface-based damage
IDamageable damageable = hitInfo.GetComponent<IDamageable>();
if (damageable != null)
{
    damageable.TakeDamage(bulletDamage);
}
```

---

## Performance Considerations

### 1. Object Pooling
- **Bullets:** Pooled to avoid constant instantiation
- **Initial pool size:** 20 bullets
- **Max pool size:** 100 bullets
- **Impact:** Eliminates GC spikes during combat

### 2. Pathfinding Optimization
- **Update frequency:** 0.5 seconds (not every frame)
- **Grid-based:** Pre-calculated walkable nodes
- **A* algorithm:** Optimal path finding with heuristics
- **Priority queue:** Efficient node selection

### 3. Collision Detection
- **Layer masks:** Filter collision checks
- **Trigger colliders:** Use triggers for pickups (more efficient)
- **Tag-based checks:** Fast string comparisons

### 4. Entity Generation
- **Overlap prevention:** Physics2D.OverlapPointAll() before placement
- **Batch generation:** All entities generated at dungeon creation
- **No runtime spawning:** Enemies don't spawn during gameplay

### 5. UI Updates
- **Event-driven:** UI updates only when values change
- **Cached references:** No GetComponent() in Update()
- **Animator-based:** Use animation events instead of polling

---

## Extension Points

### 1. New Dungeon Generators
Inherit from `AbstractDungeonGenerator` and implement `RunProceduralGeneration()`:
```csharp
public class MyCustomGenerator : AbstractDungeonGenerator
{
    protected override void RunProceduralGeneration()
    {
        // Your custom generation logic
    }
}
```

### 2. New Enemy Types
1. Create new prefab with required components:
   - SpriteRenderer
   - Animator
   - Rigidbody2D
   - Collider2D
   - EnemyScript (or custom AI script implementing IDamageable)
   - Pathfinding component
   - HealthBarEnemy

2. Add to EnemyGenerator's enemy array

### 3. New Item Types
1. Create script inheriting from MonoBehaviour
2. Implement OnTriggerEnter2D for collection
3. Create prefab with:
   - SpriteRenderer
   - Trigger Collider2D
   - Your collection script

### 4. ScriptableObject Configs
Create data-driven parameters:
```csharp
[CreateAssetMenu(menuName = "MyGame/CustomConfig")]
public class MyConfigSO : ScriptableObject
{
    public int myParameter;
    public float myValue;
}
```

---

## Debug Features

### 1. Dungeon Regeneration
Press `X` key during gameplay to regenerate the dungeon (defined in `Main.cs`)

### 2. Grid Visualization
`GridMap.cs` includes Gizmo drawing for pathfinding visualization:
```csharp
void OnDrawGizmos()
{
    if (nodes != null)
    {
        foreach (Node node in nodes)
        {
            Gizmos.color = node.IsWalkable ? Color.white : Color.red;
            Gizmos.DrawCube(node.WorldPosition, Vector3.one * 0.5f);
        }
    }
}
```

### 3. Path Visualization
Enemies draw their paths using Debug.DrawLine (visible in Scene view)

---

## Summary

This architecture provides:
- ✅ Clear separation of concerns (Player, Enemy, Generation, Pathfinding, UI)
- ✅ Extensible through inheritance and interfaces
- ✅ Data-driven configuration via ScriptableObjects
- ✅ Optimized performance through pooling and efficient algorithms
- ✅ Maintainable code through design patterns
- ✅ No external scripting dependencies (pure C#)

For implementation guides and code modification tutorials, see `MODDING_GUIDE.md`.
For quick setup instructions, see `QUICKSTART.md`.
