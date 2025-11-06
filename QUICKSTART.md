# Quick Start Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Understanding the Project](#understanding-the-project)
- [Your First Changes](#your-first-changes)
- [Development Workflow](#development-workflow)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

1. **Unity Hub** (Latest version)
   - Download from: https://unity.com/download

2. **Unity Editor** (2022.3 or higher)
   - Install via Unity Hub
   - Required modules:
     - Windows/Mac/Linux Build Support (based on your OS)
     - 2D Game Development tools

3. **Code Editor** (Choose one)
   - Visual Studio 2022 (recommended for Windows)
   - Visual Studio Code with C# extension
   - JetBrains Rider

4. **Git** (for version control)
   - Download from: https://git-scm.com/downloads

### Recommended Knowledge

- Basic C# programming
- Basic Unity concepts (GameObjects, Components, Prefabs)
- Understanding of 2D game development
- Familiarity with version control (Git)

---

## Setup Instructions

### 1. Clone the Repository

```bash
# Clone the project
git clone https://github.com/zachary013/dungeon-adventure-rpg-2d.git

# Navigate to project directory
cd dungeon-adventure-rpg-2d
```

### 2. Open in Unity

1. Launch **Unity Hub**
2. Click **"Add"** (or **"Open"**)
3. Navigate to the cloned project folder
4. Select the root folder (`dungeon-adventure-rpg-2d`)
5. Unity Hub will detect the Unity version
6. If the version isn't installed, Unity Hub will prompt you to install it
7. Click on the project to open it in Unity Editor

### 3. Initial Setup Verification

Once Unity opens:

1. **Check Console for Errors**
   - Window → General → Console
   - Should have no red errors
   - Warnings are usually okay

2. **Verify Scene Hierarchy**
   - Open `Assets/Scenes/SampleScene.unity`
   - Should see: Main Camera, DungeonGenerator, GridMap, Player, etc.

3. **Test Play Mode**
   - Click the Play button (▶) at the top
   - Dungeon should generate automatically
   - Player should be controllable with WASD
   - Press X to regenerate dungeon (debug feature)

### 4. Configure Your Code Editor

#### For Visual Studio:
1. Edit → Preferences → External Tools
2. Set "External Script Editor" to Visual Studio
3. Check "Regenerate project files"
4. Assets → Open C# Project

#### For VS Code:
1. Install C# extension from marketplace
2. Edit → Preferences → External Tools
3. Set "External Script Editor" to VS Code
4. Install Unity extension for better IntelliSense

---

## Understanding the Project

### Project Structure at a Glance

```
dungeon-adventure-rpg-2d/
│
├── Assets/_Scripts/          ← All game logic (your main workspace)
│   ├── Player/               ← Player movement, combat, health
│   ├── Enemy/                ← Enemy AI, boss behavior
│   ├── ProceduralGeneration/ ← Dungeon generation algorithms
│   ├── Map/                  ← Room and corridor generation
│   ├── Pathfinfing/          ← A* pathfinding system
│   └── UI/                   ← Menus and HUD
│
├── Assets/Scenes/            ← Game scenes
│   ├── Main Menu.unity       ← Starting point (Scene 0)
│   └── SampleScene.unity     ← Main gameplay (Scene 1)
│
├── Assets/Prefabs/           ← Reusable game objects
│   ├── Player.prefab
│   ├── Enemy1/2/3.prefab
│   └── [items, UI, etc.]
│
└── Assets/Tiles & Sprites/   ← Art assets
```

### Key Files to Know

| File | Location | Purpose |
|------|----------|---------|
| **Main.cs** | `_Scripts/` | Entry point, initializes dungeon generation |
| **GameManager.cs** | `_Scripts/UI/` | Singleton, manages game state, score, scenes |
| **PlayerMovement.cs** | `_Scripts/Player/` | Handles player input and movement |
| **EnemyScript.cs** | `_Scripts/Enemy/` | Enemy AI and pathfinding |
| **RoomFirstDungeonGenerator.cs** | `_Scripts/Map/` | Main dungeon generation algorithm |
| **Pathfinding.cs** | `_Scripts/Pathfinfing/` | A* pathfinding implementation |

### Game Flow Overview

```
1. Unity loads "Main Menu" scene (Scene 0)
2. Player clicks "Play" → loads "SampleScene" (Scene 1)
3. Main.cs Start() → calls RoomFirstDungeonGenerator
4. Dungeon generates (rooms, corridors, walls, entities)
5. Game loop begins:
   - Player moves and shoots
   - Enemies pathfind toward player
   - Collision detection handles damage
   - Items collected update GameManager
6. Boss defeated → Victory Scene (Scene 2)
   OR Player dies → Game Over screen
```

---

## Your First Changes

### Change 1: Modify Player Speed

**Goal:** Make the player move faster

**Steps:**
1. Open `Assets/_Scripts/Player/PlayerMovement.cs`
2. Find line 10: `public static float pspeed = 25f;`
3. Change to: `public static float pspeed = 35f;`
4. Save the file (Ctrl+S / Cmd+S)
5. Return to Unity (it will auto-recompile)
6. Press Play and test the change

**Why this works:**
- `pspeed` is the movement speed multiplier
- It's `public static` so other scripts can reference it
- Higher value = faster movement

### Change 2: Increase Player Health

**Goal:** Give the player more HP

**Steps:**
1. In Unity, select the **Player** GameObject in the Hierarchy
2. Look at the Inspector panel on the right
3. Find the **Player Health** component
4. Change `Maxhealth` from `200` to `300`
5. Change `Health` from `200` to `300`
6. Press Play to test

**Why this works:**
- Unity Inspector exposes public variables
- Changes in Inspector override script defaults
- This is per-prefab/instance configuration

### Change 3: Spawn More Coins

**Goal:** Increase coins per room

**Steps:**
1. Open `Assets/_Scripts/ProceduralGeneration/CoinGenerator.cs`
2. Find line ~25: `for (int i = 0; i < 5; i++)` (coins per room)
3. Change `5` to `10`
4. Save and test in Unity
5. You should see double the coins in each room

### Change 4: Change Dungeon Size

**Goal:** Make the dungeon bigger

**Steps:**
1. In Unity Hierarchy, select **DungeonGenerator** GameObject
2. In Inspector, find **Room First Dungeon Generator** component
3. Modify these values:
   - `Dungeon Width`: Change from `20` to `30`
   - `Dungeon Height`: Change from `20` to `30`
4. Press Play to see a larger dungeon

**Pro tip:** Larger dungeons take longer to generate and may have performance implications.

---

## Development Workflow

### Standard Development Cycle

```
1. [Plan] → Identify what you want to change
         ↓
2. [Find] → Locate the relevant script/prefab
         ↓
3. [Edit] → Make your changes in code editor
         ↓
4. [Save] → Save file (Unity auto-compiles C#)
         ↓
5. [Test] → Press Play in Unity to test
         ↓
6. [Debug] → Check Console for errors
         ↓
7. [Iterate] → Repeat until satisfied
         ↓
8. [Commit] → Save changes to Git
```

### Using the Unity Console

**Accessing:** Window → General → Console (or Ctrl+Shift+C)

**Error Types:**
- 🔴 **Red (Error):** Code won't compile, must fix
- 🟡 **Yellow (Warning):** Code works but might have issues
- ⚪ **White (Log):** Information messages (from Debug.Log)

**Common Errors:**
```csharp
// Error: Missing semicolon
public int health = 100  // ❌ Missing semicolon

// Fixed:
public int health = 100; // ✅ Correct

// Error: Variable not declared
healthValue = 100;       // ❌ Type not specified

// Fixed:
int healthValue = 100;   // ✅ With type

// Error: NullReferenceException
player.transform.position // ❌ player might be null

// Fixed:
if (player != null)      // ✅ Check first
    player.transform.position
```

### Testing Best Practices

1. **Always test in Play Mode first**
   - Don't push untested code
   - Use Debug.Log() to track values

2. **Test edge cases**
   - What if player has 0 health?
   - What if no enemies spawn?
   - What if dungeon is 1x1 room?

3. **Use Debug Features**
   - Press X to regenerate dungeon (Main.cs:18)
   - Scene view for visual debugging
   - Gizmos for pathfinding visualization

4. **Check performance**
   - Stats window: Game view → Stats button
   - FPS counter: Window → Analysis → Profiler

---

## Common Tasks

### Task 1: Add Debug Logging

**When:** You want to see what's happening in your code

**Example:**
```csharp
// In PlayerHealth.cs TakeDamage method
public void TakeDamage(int damage)
{
    Debug.Log($"Player took {damage} damage. Health: {health}");
    health -= damage;
    healthBar.UpdateHealthBar(health, maxhealth);
}
```

**View output:** Console window while in Play Mode

### Task 2: Create a New Enemy Type

**Steps:**
1. Duplicate existing enemy prefab:
   - In Project: `Assets/Prefabs/Enemy1.prefab`
   - Right-click → Duplicate
   - Rename to `Enemy4.prefab`

2. Customize in Inspector:
   - Change sprite/animation
   - Adjust EnemyScript values (speed, health)

3. Add to spawn rotation:
   - Open `EnemyGenerator.cs`
   - Add new enemy prefab to array
   - Update rotation logic if needed

### Task 3: Modify Fire Rate

**Goal:** Make player shoot faster/slower

**Steps:**
1. Select **Player** GameObject in Hierarchy
2. Find **Aim** child GameObject
3. In Inspector, locate **Player Aim Weapon** component
4. Adjust `Fire Rate` (lower = faster shooting)
   - Default: 0.5 seconds
   - Fast: 0.2 seconds
   - Slow: 1.0 seconds

### Task 4: Change Bullet Damage

**Steps:**
1. Open `Assets/_Scripts/Bullet.cs`
2. Find line 42: `damageable.TakeDamage(30);`
3. Change `30` to desired damage (e.g., `50`)
4. Save and test

### Task 5: Add a New Item

**Example: Super Potion (heals 60 HP)**

1. **Create Script:**
```csharp
// File: Assets/_Scripts/SuperPotion.cs
using UnityEngine;

public class SuperPotion : MonoBehaviour
{
    public int healAmount = 60;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            PlayerHealth playerHealth = other.GetComponent<PlayerHealth>();
            if (playerHealth != null)
            {
                playerHealth.Heal(healAmount);
                Destroy(gameObject);
            }
        }
    }
}
```

2. **Create Prefab:**
   - Create empty GameObject: Right-click Hierarchy → Create Empty
   - Add SpriteRenderer component
   - Add CircleCollider2D (set to Trigger)
   - Add SuperPotion script
   - Drag to Prefabs folder

3. **Generate in Dungeon:**
   - Create `SuperPotionGenerator.cs` (copy from `HealthPotionGenerator.cs`)
   - Call from `RoomFirstDungeonGenerator.cs`

### Task 6: Change Enemy Pathfinding Speed

**Steps:**
1. Select any enemy prefab: `Assets/Prefabs/Enemy1.prefab`
2. In Inspector, find **Enemy Script** component
3. Change `Speed` value (default: 3)
4. Apply to prefab (Overrides dropdown → Apply All)
5. All instances will update

### Task 7: Modify Dungeon Generation

**Make rooms bigger:**
1. Select **DungeonGenerator** in Hierarchy
2. In **Room First Dungeon Generator** component:
   - `Min Room Width`: Increase from 4 to 6
   - `Min Room Height`: Increase from 4 to 6
3. Test in Play Mode

**More enemies per room:**
1. Open `EnemyGenerator.cs`
2. Change `enemiesPerRoom` from 3 to 5
3. Save and test

---

## Troubleshooting

### Issue: Scripts Won't Compile

**Symptoms:**
- Red errors in Console
- Can't enter Play Mode
- "All compiler errors have to be fixed before you can enter playmode!"

**Solutions:**
1. Read the error message carefully (double-click to go to line)
2. Common fixes:
   - Add missing semicolon `;`
   - Fix typos in variable names
   - Add missing `using` statements at top
   - Match opening/closing braces `{}`
3. If stuck, revert your changes (Ctrl+Z)

### Issue: Changes Not Appearing

**Symptoms:**
- Modified code but game behaves the same

**Solutions:**
1. **Ensure file is saved** (check for * in tab)
2. **Wait for Unity to recompile** (progress bar bottom-right)
3. **Check you're editing the right script:**
   - Select GameObject in Hierarchy
   - In Inspector, click script name
   - Should open the file
4. **Prefab overrides:**
   - If you edited a prefab, did you apply changes?
   - Overrides dropdown → Apply All

### Issue: NullReferenceException

**Symptoms:**
- Error: "NullReferenceException: Object reference not set to an instance of an object"

**Common Causes:**
1. Forgot to assign reference in Inspector
2. GameObject was destroyed
3. GetComponent returned null

**Solutions:**
```csharp
// Always check for null before using
if (player != null)
{
    player.transform.position = newPosition;
}

// Or use null-conditional operator
player?.transform.Translate(movement);

// Log to find out what's null
Debug.Log($"Player is null: {player == null}");
```

### Issue: Player/Enemies Not Visible

**Symptoms:**
- Game runs but can't see entities

**Check:**
1. **Camera position:** Is it at Z = -10?
2. **Sprite renderer:** Does GameObject have SpriteRenderer component?
3. **Sprite assigned:** Is sprite field populated in SpriteRenderer?
4. **Layer/Sorting:** Check Sorting Layer and Order in Layer
5. **Scale:** Is transform scale (1, 1, 1)?

### Issue: Pathfinding Not Working

**Symptoms:**
- Enemies don't move toward player
- Enemies stuck in walls

**Check:**
1. **GridMap initialized:** DungeonGenerator must call GridMap.InitializeGrid()
2. **Tilemap reference:** GridMap has reference to floor tilemap
3. **Walkable nodes:** Use Gizmos to visualize (Scene view)
4. **Path calculation:** Add Debug.Log in Pathfinding.FindPath()

```csharp
// In Pathfinding.cs FindPath method
List<Node> path = FindPath(startNode, targetNode);
Debug.Log($"Path found: {path != null}, Length: {path?.Count ?? 0}");
```

### Issue: Performance Problems

**Symptoms:**
- Low FPS
- Stuttering
- Lag when many enemies present

**Solutions:**
1. **Reduce dungeon size** (smaller width/height)
2. **Fewer entities per room** (reduce spawn counts)
3. **Increase pathfinding update interval** (EnemyScript.cs: 0.5f → 1.0f)
4. **Profile the game:**
   - Window → Analysis → Profiler
   - Identify bottlenecks
5. **Disable Gizmos** (toggle Gizmos button in Scene view)

### Issue: Build Errors

**Symptoms:**
- Game works in Editor but not in build

**Check:**
1. **Scenes in Build Settings:**
   - File → Build Settings
   - Add all required scenes
   - Correct scene order (Main Menu = 0)
2. **Resources loading:** Use Resources.Load() for runtime loading
3. **Platform-specific code:** Use preprocessor directives
4. **Console in build:** Check Player.log file

---

## Next Steps

### Beginner Level
✅ Complete all tasks in "Your First Changes"
✅ Experiment with public variables in Inspector
✅ Add Debug.Log statements to understand flow
✅ Modify existing prefabs

### Intermediate Level
✅ Read `ARCHITECTURE.md` to understand systems
✅ Create new item types
✅ Modify generation algorithms
✅ Implement new enemy behaviors
✅ Follow guides in `MODDING_GUIDE.md`

### Advanced Level
✅ Add new procedural generation algorithms
✅ Implement new pathfinding algorithms
✅ Create complex game mechanics
✅ Optimize performance with profiling
✅ Add multiplayer or online features

---

## Useful Unity Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + P` / `Cmd + P` | Play/Pause game |
| `Ctrl + Shift + P` | Pause game |
| `F` | Frame selected GameObject in Scene |
| `Ctrl + D` | Duplicate selected GameObject |
| `Ctrl + Shift + N` | Create new empty GameObject |
| `Ctrl + Shift + C` | Open Console |
| `Ctrl + 1-9` | Switch between Scene/Game/Other tabs |

---

## Additional Resources

### Official Documentation
- Unity Manual: https://docs.unity3d.com/Manual/index.html
- Unity Scripting API: https://docs.unity3d.com/ScriptReference/
- C# Programming Guide: https://docs.microsoft.com/en-us/dotnet/csharp/

### Learning Resources
- Unity Learn: https://learn.unity.com/
- Brackeys YouTube: https://www.youtube.com/@Brackeys
- Unity Forums: https://forum.unity.com/

### Project-Specific Docs
- `ARCHITECTURE.md` - Detailed code architecture and system design
- `MODDING_GUIDE.md` - Step-by-step guides for modifying core systems
- `README.md` - Project overview and installation

---

## Getting Help

### Before Asking for Help

1. ✅ Check Console for error messages
2. ✅ Read the error message completely
3. ✅ Search error message on Google
4. ✅ Review relevant documentation
5. ✅ Try reverting your changes

### When Asking for Help

Include:
- What you're trying to achieve
- What you've tried
- Complete error message
- Relevant code snippet
- Unity version

### Contact

- **GitHub Issues:** https://github.com/zachary013/dungeon-adventure-rpg-2d/issues
- **Email:** azarkanzakariae@gmail.com
- **LinkedIn:** [Zakariae Azarkan](https://www.linkedin.com/in/zakariae-azarkan-8b49a12b8/)

---

## Summary

You now know how to:
- ✅ Set up the project
- ✅ Navigate the codebase
- ✅ Make simple modifications
- ✅ Test your changes
- ✅ Debug common issues
- ✅ Find additional resources

**Ready for more?** Check out `MODDING_GUIDE.md` for advanced modification tutorials!

Happy coding! 🎮
