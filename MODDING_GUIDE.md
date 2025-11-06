# Modding Guide - Step-by-Step Modifications

## Table of Contents
- [Introduction](#introduction)
- [Player System Modifications](#player-system-modifications)
- [Enemy System Modifications](#enemy-system-modifications)
- [Dungeon Generation Modifications](#dungeon-generation-modifications)
- [Combat System Modifications](#combat-system-modifications)
- [UI System Modifications](#ui-system-modifications)
- [Item System Modifications](#item-system-modifications)
- [Pathfinding Modifications](#pathfinding-modifications)
- [Advanced Modifications](#advanced-modifications)

---

## Introduction

This guide provides detailed, step-by-step instructions for modifying core elements of the dungeon adventure RPG. Each section includes:
- 🎯 **Goal** - What you'll accomplish
- 📋 **Prerequisites** - What you need to know
- 🔧 **Steps** - Detailed instructions
- 💡 **Tips** - Best practices and insights
- ⚠️ **Common Issues** - Troubleshooting

**Before You Start:**
1. Make a backup of your project (copy the entire folder)
2. Test in Play Mode after each change
3. Read error messages in the Console carefully
4. Use version control (Git) to track changes

---

## Player System Modifications

### Modification 1.1: Add a Dash/Sprint Ability

🎯 **Goal:** Give the player a temporary speed boost when pressing Shift

📋 **Prerequisites:** Basic understanding of Unity Input system

🔧 **Steps:**

**Step 1: Modify PlayerMovement.cs**

Open `/Assets/_Scripts/Player/PlayerMovement.cs`

Add these variables at the top of the class (around line 10):
```csharp
public static float pspeed = 25f;
public float dashSpeed = 50f;        // NEW: Speed during dash
public float dashDuration = 0.3f;    // NEW: How long dash lasts
private float dashTimeRemaining = 0f; // NEW: Track dash timer
```

**Step 2: Update the Update() Method**

Find the `Update()` method (around line 30) and modify it:

```csharp
void Update()
{
    // NEW: Handle dash input
    if (Input.GetKeyDown(KeyCode.LeftShift) && dashTimeRemaining <= 0)
    {
        dashTimeRemaining = dashDuration;
        Debug.Log("Dash activated!");
    }

    // NEW: Countdown dash timer
    if (dashTimeRemaining > 0)
    {
        dashTimeRemaining -= Time.deltaTime;
    }

    // Get input
    float moveHorizontal = Input.GetAxis("Horizontal");
    float moveVertical = Input.GetAxis("Vertical");

    // NEW: Choose speed based on dash state
    float currentSpeed = dashTimeRemaining > 0 ? dashSpeed : pspeed;

    // Calculate movement (MODIFIED to use currentSpeed)
    Vector2 movement = new Vector2(moveHorizontal, moveVertical);
    rb.velocity = movement * currentSpeed;

    // ... rest of existing code ...
}
```

**Step 3: Test**

1. Save the file
2. Return to Unity (wait for recompile)
3. Press Play
4. Hold WASD and press Left Shift - you should dash!

💡 **Tips:**
- Add a cooldown system to prevent spam
- Create a visual effect (particle trail) during dash
- Make dash invulnerable to enemy damage

⚠️ **Common Issues:**
- **Dash too fast:** Reduce `dashSpeed` value
- **Dash too short:** Increase `dashDuration`
- **Continues forever:** Make sure `dashTimeRemaining` is decreasing

---

### Modification 1.2: Add Player Stamina System

🎯 **Goal:** Create a stamina bar that depletes with actions and regenerates over time

📋 **Prerequisites:** Understanding of Unity UI and coroutines

🔧 **Steps:**

**Step 1: Create Stamina Script**

Create new file: `/Assets/_Scripts/Player/PlayerStamina.cs`

```csharp
using UnityEngine;
using UnityEngine.UI;

public class PlayerStamina : MonoBehaviour
{
    [Header("Stamina Settings")]
    public float maxStamina = 100f;
    public float currentStamina = 100f;
    public float staminaRegenRate = 10f;  // Per second
    public float dashStaminaCost = 30f;

    [Header("UI Reference")]
    public Slider staminaBar;
    public Gradient staminaGradient;
    public Image staminaFill;

    private bool isRegenerating = true;

    void Start()
    {
        currentStamina = maxStamina;
        UpdateStaminaBar();
    }

    void Update()
    {
        // Regenerate stamina over time
        if (isRegenerating && currentStamina < maxStamina)
        {
            currentStamina += staminaRegenRate * Time.deltaTime;
            currentStamina = Mathf.Min(currentStamina, maxStamina);
            UpdateStaminaBar();
        }
    }

    public bool UseStamina(float amount)
    {
        if (currentStamina >= amount)
        {
            currentStamina -= amount;
            UpdateStaminaBar();
            return true; // Stamina available
        }
        return false; // Not enough stamina
    }

    void UpdateStaminaBar()
    {
        if (staminaBar != null)
        {
            staminaBar.value = currentStamina;
            staminaBar.maxValue = maxStamina;

            if (staminaFill != null)
            {
                staminaFill.color = staminaGradient.Evaluate(
                    currentStamina / maxStamina
                );
            }
        }
    }
}
```

**Step 2: Integrate with PlayerMovement**

Modify `/Assets/_Scripts/Player/PlayerMovement.cs`:

Add at top of class:
```csharp
private PlayerStamina stamina; // NEW

void Start()
{
    // ... existing code ...
    stamina = GetComponent<PlayerStamina>(); // NEW
}
```

Modify dash input section in Update():
```csharp
// MODIFIED: Check stamina before dashing
if (Input.GetKeyDown(KeyCode.LeftShift) && dashTimeRemaining <= 0)
{
    if (stamina != null && stamina.UseStamina(stamina.dashStaminaCost))
    {
        dashTimeRemaining = dashDuration;
        Debug.Log("Dash activated!");
    }
    else
    {
        Debug.Log("Not enough stamina!");
    }
}
```

**Step 3: Create Stamina UI**

1. In Unity Hierarchy, right-click → UI → Canvas (if not exists)
2. Right-click Canvas → UI → Slider
3. Rename to "StaminaBar"
4. Position in bottom-left corner (X: 100, Y: 80)
5. In Inspector, set:
   - Min Value: 0
   - Max Value: 100
   - Whole Numbers: No
6. Add gradient coloring (optional)

**Step 4: Connect References**

1. Select **Player** GameObject
2. Add **Player Stamina** component (Add Component → Search "PlayerStamina")
3. Drag **StaminaBar** slider into the `Stamina Bar` field
4. Create a new Gradient in `Stamina Gradient`:
   - 0%: Red
   - 50%: Yellow
   - 100%: Green
5. Drag the StaminaBar → Fill image into `Stamina Fill`

**Step 5: Test**

1. Press Play
2. Press Shift to dash - stamina should decrease
3. Wait - stamina should regenerate
4. Try dashing when stamina is empty

💡 **Enhancement Ideas:**
- Stamina cost for attacking
- Different regeneration rates when idle vs moving
- Stamina boost pickups
- Sound effects for low stamina

---

### Modification 1.3: Add Weapon Switching

🎯 **Goal:** Allow player to switch between multiple weapons (pistol, shotgun, rifle)

🔧 **Steps:**

**Step 1: Create Weapon Data Structure**

Create `/Assets/_Scripts/Player/WeaponData.cs`:

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "NewWeapon", menuName = "Weapons/Weapon Data")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public GameObject bulletPrefab;
    public float fireRate = 0.5f;
    public float bulletSpeed = 20f;
    public int damage = 30;
    public int bulletsPerShot = 1;     // Shotgun fires multiple
    public float spreadAngle = 0f;     // Shotgun spread
    public Sprite weaponSprite;        // Visual representation
}
```

**Step 2: Create Weapon Assets**

1. In Project window, right-click in `Assets/_Scripts/Data/` folder
2. Create → Weapons → Weapon Data
3. Create three weapons:

**Pistol:**
- Name: "Pistol"
- Fire Rate: 0.5
- Bullet Speed: 20
- Damage: 30
- Bullets Per Shot: 1
- Spread Angle: 0

**Shotgun:**
- Name: "Shotgun"
- Fire Rate: 1.0
- Bullet Speed: 15
- Damage: 20
- Bullets Per Shot: 5
- Spread Angle: 15

**Rifle:**
- Name: "Rifle"
- Fire Rate: 0.1
- Bullet Speed: 30
- Damage: 15
- Bullets Per Shot: 1
- Spread Angle: 0

**Step 3: Modify PlayerAimWeapon.cs**

Open `/Assets/_Scripts/Player/PlayerAimWeapon.cs`:

```csharp
using UnityEngine;
using System.Collections;

public class PlayerAimWeapon : MonoBehaviour
{
    [Header("Weapon System")]
    public WeaponData[] weapons;              // NEW: Array of weapons
    private int currentWeaponIndex = 0;       // NEW: Current weapon
    private WeaponData currentWeapon;         // NEW: Active weapon

    [Header("Firing")]
    public Transform firePoint;
    private ObjectPool<GameObject> bulletPool;
    private float nextFireTime = 0f;

    [Header("UI")]
    public TMPro.TextMeshProUGUI weaponNameText; // NEW: Display weapon name

    void Start()
    {
        // Initialize weapon system
        if (weapons.Length > 0)
        {
            currentWeapon = weapons[0];
            UpdateWeaponUI();
        }

        // Initialize bullet pool
        bulletPool = new ObjectPool<GameObject>(
            () => Instantiate(currentWeapon.bulletPrefab),
            bullet => bullet.SetActive(true),
            bullet => bullet.SetActive(false),
            bullet => Destroy(bullet),
            false, 20, 100
        );
    }

    void Update()
    {
        // NEW: Weapon switching
        if (Input.GetKeyDown(KeyCode.Alpha1)) SwitchWeapon(0);
        if (Input.GetKeyDown(KeyCode.Alpha2)) SwitchWeapon(1);
        if (Input.GetKeyDown(KeyCode.Alpha3)) SwitchWeapon(2);
        if (Input.GetKeyDown(KeyCode.Q)) SwitchToNextWeapon();

        // Aim weapon at mouse
        Vector3 mousePosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector2 direction = mousePosition - transform.position;
        float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg - 90f;
        transform.rotation = Quaternion.Euler(0, 0, angle);

        // Fire weapon
        if ((Input.GetButton("Fire1") || Input.GetKey(KeyCode.Space)) &&
            Time.time >= nextFireTime)
        {
            Shoot();
            nextFireTime = Time.time + currentWeapon.fireRate;
        }
    }

    void SwitchWeapon(int index)
    {
        if (index >= 0 && index < weapons.Length)
        {
            currentWeaponIndex = index;
            currentWeapon = weapons[index];
            UpdateWeaponUI();
            Debug.Log($"Switched to {currentWeapon.weaponName}");
        }
    }

    void SwitchToNextWeapon()
    {
        currentWeaponIndex = (currentWeaponIndex + 1) % weapons.Length;
        currentWeapon = weapons[currentWeaponIndex];
        UpdateWeaponUI();
    }

    void Shoot()
    {
        for (int i = 0; i < currentWeapon.bulletsPerShot; i++)
        {
            GameObject bullet = bulletPool.Get();
            bullet.transform.position = firePoint.position;

            // Calculate spread for shotgun
            float spreadOffset = 0f;
            if (currentWeapon.bulletsPerShot > 1)
            {
                spreadOffset = Random.Range(
                    -currentWeapon.spreadAngle,
                    currentWeapon.spreadAngle
                );
            }

            bullet.transform.rotation = Quaternion.Euler(
                0, 0,
                firePoint.rotation.eulerAngles.z + spreadOffset
            );

            Rigidbody2D rb = bullet.GetComponent<Rigidbody2D>();
            rb.velocity = bullet.transform.up * currentWeapon.bulletSpeed;

            // Set damage on bullet
            Bullet bulletScript = bullet.GetComponent<Bullet>();
            if (bulletScript != null)
            {
                bulletScript.damage = currentWeapon.damage;
            }
        }
    }

    void UpdateWeaponUI()
    {
        if (weaponNameText != null)
        {
            weaponNameText.text = currentWeapon.weaponName;
        }
    }
}
```

**Step 4: Update Bullet.cs to Use Variable Damage**

Modify `/Assets/_Scripts/Bullet.cs`:

```csharp
public class Bullet : MonoBehaviour
{
    public int damage = 30; // NEW: Configurable damage
    // ... rest of code ...

    void OnTriggerEnter2D(Collider2D hitInfo)
    {
        // ... existing layer check ...

        IDamageable damageable = hitInfo.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(damage); // MODIFIED: Use variable damage
        }

        gameObject.SetActive(false);
    }
}
```

**Step 5: Setup in Unity**

1. Select **Player → Aim** GameObject
2. In **Player Aim Weapon** component:
   - Set Weapons array size to 3
   - Drag your created WeaponData assets into slots
3. Create UI text for weapon name (optional)
4. Test: Press 1, 2, 3 or Q to switch weapons

💡 **Tips:**
- Add weapon pickup items that unlock weapons
- Display weapon icon instead of just text
- Add reload mechanics for limited ammo
- Create unique visual effects for each weapon

---

## Enemy System Modifications

### Modification 2.1: Add Ranged Enemies

🎯 **Goal:** Create enemies that shoot projectiles at the player

🔧 **Steps:**

**Step 1: Create Enemy Bullet Prefab**

1. Duplicate existing Bullet prefab
2. Rename to "EnemyBullet"
3. Change sprite to different color (red)
4. Ensure it has:
   - SpriteRenderer
   - Rigidbody2D (Gravity Scale: 0)
   - CircleCollider2D (Is Trigger: true)
   - Bullet script

**Step 2: Create RangedEnemyScript.cs**

Create `/Assets/_Scripts/Enemy/RangedEnemyScript.cs`:

```csharp
using UnityEngine;
using System.Collections;

public class RangedEnemyScript : MonoBehaviour, IDamageable
{
    [Header("Movement")]
    public float speed = 2f;
    public float keepDistance = 8f;      // Stay this far from player

    [Header("Combat")]
    public int maxHealth = 80;
    private int health;
    public GameObject bulletPrefab;
    public Transform firePoint;
    public float fireRate = 2f;          // Seconds between shots
    public float bulletSpeed = 10f;
    public float detectionRange = 15f;   // How far can see player

    [Header("References")]
    private Transform player;
    private Rigidbody2D rb;
    private HealthBarEnemy healthBarEnemy;
    private float nextFireTime = 0f;

    void Start()
    {
        health = maxHealth;
        rb = GetComponent<Rigidbody2D>();
        healthBarEnemy = GetComponentInChildren<HealthBarEnemy>();
        player = GameObject.FindGameObjectWithTag("Player").transform;

        if (healthBarEnemy != null)
        {
            healthBarEnemy.UpdateHealthBar(health, maxHealth);
        }
    }

    void Update()
    {
        if (player == null) return;

        float distanceToPlayer = Vector2.Distance(transform.position, player.position);

        // Move to maintain distance
        if (distanceToPlayer < keepDistance - 1f)
        {
            // Too close - retreat
            Vector2 direction = (transform.position - player.position).normalized;
            rb.velocity = direction * speed;
        }
        else if (distanceToPlayer > keepDistance + 1f)
        {
            // Too far - approach
            Vector2 direction = (player.position - transform.position).normalized;
            rb.velocity = direction * speed;
        }
        else
        {
            // Good distance - stop moving
            rb.velocity = Vector2.zero;
        }

        // Shoot at player if in range
        if (distanceToPlayer <= detectionRange && Time.time >= nextFireTime)
        {
            ShootAtPlayer();
            nextFireTime = Time.time + fireRate;
        }
    }

    void ShootAtPlayer()
    {
        if (bulletPrefab != null && firePoint != null)
        {
            // Instantiate bullet
            GameObject bullet = Instantiate(
                bulletPrefab,
                firePoint.position,
                Quaternion.identity
            );

            // Calculate direction to player
            Vector2 direction = (player.position - firePoint.position).normalized;

            // Set bullet rotation
            float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg - 90f;
            bullet.transform.rotation = Quaternion.Euler(0, 0, angle);

            // Launch bullet
            Rigidbody2D bulletRb = bullet.GetComponent<Rigidbody2D>();
            bulletRb.velocity = direction * bulletSpeed;

            Debug.Log("Ranged enemy fired!");
        }
    }

    public void TakeDamage(int damage)
    {
        health -= damage;

        if (healthBarEnemy != null)
        {
            healthBarEnemy.UpdateHealthBar(health, maxHealth);
        }

        if (health <= 0)
        {
            Die();
        }
    }

    void Die()
    {
        Debug.Log("Ranged enemy died!");
        Destroy(gameObject);
    }
}
```

**Step 3: Modify Enemy Bullet for Player Damage**

Update `/Assets/_Scripts/Bullet.cs` to handle layer-specific damage:

```csharp
void OnTriggerEnter2D(Collider2D hitInfo)
{
    // Check layer
    if (((1 << hitInfo.gameObject.layer) & whatIsEnemy) == 0)
        return;

    // NEW: If this is an enemy bullet hitting player
    if (hitInfo.CompareTag("Player") && gameObject.CompareTag("EnemyBullet"))
    {
        PlayerHealth playerHealth = hitInfo.GetComponent<PlayerHealth>();
        if (playerHealth != null)
        {
            playerHealth.TakeDamage(15); // Enemy bullet damage
            gameObject.SetActive(false);
            return;
        }
    }

    // Existing code for player bullets hitting enemies
    IDamageable damageable = hitInfo.GetComponent<IDamageable>();
    if (damageable != null)
    {
        damageable.TakeDamage(damage);
    }

    gameObject.SetActive(false);
}
```

**Step 4: Create Ranged Enemy Prefab**

1. Create Empty GameObject: Hierarchy → Create Empty
2. Name it "RangedEnemy"
3. Add components:
   - Sprite Renderer (assign enemy sprite)
   - Rigidbody2D (Gravity Scale: 0, Freeze Rotation Z)
   - Circle Collider 2D
   - Ranged Enemy Script
4. Create child GameObject named "FirePoint"
   - Position slightly in front of enemy
5. Drag HealthBarEnemy prefab as child
6. In Ranged Enemy Script:
   - Assign EnemyBullet prefab
   - Assign FirePoint transform
7. Tag as "Enemy"
8. Save as prefab in Assets/Prefabs/

**Step 5: Add to Dungeon Generation**

Modify `/Assets/_Scripts/ProceduralGeneration/EnemyGenerator.cs`:

```csharp
public GameObject[] enemyPrefabs; // Ensure array has 4 elements now
public GameObject rangedEnemyPrefab; // NEW

public void GenerateEnemies(List<BoundsInt> roomBounds)
{
    foreach (var roomBound in roomBounds)
    {
        for (int i = 0; i < enemiesPerRoom; i++)
        {
            // ... existing collision check code ...

            // NEW: 25% chance to spawn ranged enemy
            GameObject enemyPrefab;
            if (Random.value < 0.25f && rangedEnemyPrefab != null)
            {
                enemyPrefab = rangedEnemyPrefab;
            }
            else
            {
                enemyPrefab = enemyPrefabs[enemyIndex % 3];
            }

            GameObject enemy = Instantiate(
                enemyPrefab,
                (Vector2)randomPosition,
                Quaternion.identity
            );

            enemyIndex++;
        }
    }
}
```

**Step 6: Test**

1. Select DungeonGenerator in Hierarchy
2. Find Enemy Generator component
3. Assign RangedEnemy prefab
4. Play - you should see some enemies shooting!

💡 **Enhancement Ideas:**
- Add laser sight showing where enemy will shoot
- Different bullet patterns (burst fire, spread)
- Enemy takes cover when reloading
- Warning indicator before enemy shoots

---

### Modification 2.2: Add Boss Phases

🎯 **Goal:** Make the boss fight more dynamic with different behavior phases

🔧 **Steps:**

**Step 1: Modify BossScript.cs**

Open `/Assets/_Scripts/Enemy/BossScript.cs` and replace with:

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;

public class BossScript : MonoBehaviour, IDamageable
{
    [Header("Health")]
    public int maxhealth = 500;
    private int health;

    [Header("Movement")]
    public float normalSpeed = 5f;
    public float enragedSpeed = 8f;
    private float currentSpeed;

    [Header("Combat")]
    public GameObject minionPrefab;
    public int minionsPerSpawn = 3;
    public float minionSpawnRadius = 3f;

    [Header("Phases")]
    private enum BossPhase { Phase1, Phase2, Phase3 }
    private BossPhase currentPhase = BossPhase.Phase1;

    private Transform player;
    private Rigidbody2D rb;
    private HealthBarEnemy healthBarEnemy;
    private bool isEnraged = false;

    void Start()
    {
        health = maxhealth;
        currentSpeed = normalSpeed;
        rb = GetComponent<Rigidbody2D>();
        healthBarEnemy = GetComponentInChildren<HealthBarEnemy>();
        player = GameObject.FindGameObjectWithTag("Player").transform;

        if (healthBarEnemy != null)
        {
            healthBarEnemy.UpdateHealthBar(health, maxhealth);
        }
    }

    void Update()
    {
        if (player != null)
        {
            // Chase player
            Vector2 direction = (player.position - transform.position).normalized;
            rb.velocity = direction * currentSpeed;
        }
    }

    public void TakeDamage(int damage)
    {
        health -= damage;

        if (healthBarEnemy != null)
        {
            healthBarEnemy.UpdateHealthBar(health, maxhealth);
        }

        // Check for phase transitions
        float healthPercent = (float)health / maxhealth;

        if (healthPercent <= 0.66f && currentPhase == BossPhase.Phase1)
        {
            EnterPhase2();
        }
        else if (healthPercent <= 0.33f && currentPhase == BossPhase.Phase2)
        {
            EnterPhase3();
        }

        if (health <= 0)
        {
            Die();
        }
    }

    void EnterPhase2()
    {
        currentPhase = BossPhase.Phase2;
        Debug.Log("Boss entered Phase 2!");

        // Increase speed
        currentSpeed = normalSpeed * 1.3f;

        // Spawn minions
        SpawnMinions();

        // Change color to indicate phase (optional)
        GetComponent<SpriteRenderer>().color = Color.yellow;
    }

    void EnterPhase3()
    {
        currentPhase = BossPhase.Phase3;
        Debug.Log("Boss entered Phase 3 - ENRAGED!");

        // Maximum speed
        currentSpeed = enragedSpeed;

        // Spawn more minions
        SpawnMinions();
        SpawnMinions(); // Double spawn

        // Change color to red
        GetComponent<SpriteRenderer>().color = Color.red;

        // Start enraged mode (increased damage)
        isEnraged = true;
    }

    void SpawnMinions()
    {
        if (minionPrefab == null) return;

        for (int i = 0; i < minionsPerSpawn; i++)
        {
            // Random position around boss
            Vector2 randomOffset = Random.insideUnitCircle * minionSpawnRadius;
            Vector3 spawnPosition = transform.position + (Vector3)randomOffset;

            Instantiate(minionPrefab, spawnPosition, Quaternion.identity);
        }

        Debug.Log($"Boss spawned {minionsPerSpawn} minions!");
    }

    void OnCollisionEnter2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            PlayerHealth playerHealth = collision.gameObject.GetComponent<PlayerHealth>();
            if (playerHealth != null)
            {
                // Increased damage when enraged
                int damage = isEnraged ? 30 : 20;
                playerHealth.TakeDamage(damage);
            }
        }
    }

    void Die()
    {
        Debug.Log("Boss defeated!");
        StartCoroutine(LoadVictoryScene());
    }

    IEnumerator LoadVictoryScene()
    {
        yield return new WaitForSeconds(3f);
        SceneManager.LoadScene(2); // Victory scene
    }
}
```

**Step 2: Setup Boss in Unity**

1. Open Boss prefab: `Assets/Prefabs/Boss.prefab`
2. In Boss Script component:
   - Set Normal Speed: 5
   - Set Enraged Speed: 8
   - Assign Enemy1 prefab as Minion Prefab
   - Set Minions Per Spawn: 3
   - Set Minion Spawn Radius: 3

**Step 3: Test**

1. Play the game
2. Navigate to the boss room
3. Damage boss to 66% health - should turn yellow and spawn minions
4. Damage to 33% health - should turn red, speed up, spawn more minions

💡 **Enhancement Ideas:**
- Add different attack patterns per phase
- Teleportation in phase 3
- Area-of-effect attacks
- Shield mechanic that must be broken
- Visual effects for phase transitions

---

## Dungeon Generation Modifications

### Modification 3.1: Add Different Room Types

🎯 **Goal:** Create specialized rooms (treasure room, trap room, shop, etc.)

🔧 **Steps:**

**Step 1: Create Room Type Enum**

Create `/Assets/_Scripts/Map/RoomType.cs`:

```csharp
public enum RoomType
{
    Normal,
    Treasure,
    Trap,
    Shop,
    Boss,
    Start
}
```

**Step 2: Create RoomData Class**

Create `/Assets/_Scripts/Map/RoomData.cs`:

```csharp
using UnityEngine;

public class RoomData : MonoBehaviour
{
    public RoomType roomType = RoomType.Normal;
    public BoundsInt bounds;
    public Vector2Int center;

    // Room-specific properties
    public int coinMultiplier = 1;
    public int enemyMultiplier = 1;
    public bool hasBeenVisited = false;

    public void Initialize(BoundsInt roomBounds, RoomType type)
    {
        bounds = roomBounds;
        center = (Vector2Int)Vector3Int.RoundToInt(roomBounds.center);
        roomType = type;

        // Set properties based on room type
        switch (type)
        {
            case RoomType.Treasure:
                coinMultiplier = 3;
                enemyMultiplier = 0; // No enemies in treasure room
                break;

            case RoomType.Trap:
                coinMultiplier = 2;
                enemyMultiplier = 2; // Extra enemies
                break;

            case RoomType.Shop:
                coinMultiplier = 0;
                enemyMultiplier = 0;
                break;

            case RoomType.Boss:
                coinMultiplier = 0;
                enemyMultiplier = 0; // Boss only
                break;

            case RoomType.Start:
                coinMultiplier = 1;
                enemyMultiplier = 0; // Safe starting room
                break;

            default: // Normal
                coinMultiplier = 1;
                enemyMultiplier = 1;
                break;
        }
    }
}
```

**Step 3: Modify RoomFirstDungeonGenerator.cs**

Open `/Assets/_Scripts/Map/RoomFirstDungeonGenerator.cs`:

Add at top of class:
```csharp
using System.Collections.Generic;

private List<RoomData> roomDataList = new List<RoomData>();
```

Modify the generation method (around line 40):

```csharp
protected override void playRunProceduralGeneration()
{
    tilemapVisualizer.Clear();
    roomDataList.Clear(); // NEW

    // Generate room bounds
    List<BoundsInt> roomBounds = ProceduralGenerationAlgorithms
        .BinarySpacePartitioning(
            new BoundsInt((Vector3Int)startPosition,
            new Vector3Int(dungeonWidth, dungeonHeight, 0)),
            minRoomWidth, minRoomHeight
        );

    // NEW: Assign room types
    AssignRoomTypes(roomBounds);

    // ... rest of existing generation code ...

    // MODIFIED: Generate entities based on room type
    GenerateEntitiesWithRoomTypes();
}

// NEW METHOD: Assign types to rooms
void AssignRoomTypes(List<BoundsInt> roomBounds)
{
    for (int i = 0; i < roomBounds.Count; i++)
    {
        GameObject roomObj = new GameObject($"Room_{i}");
        roomObj.transform.parent = transform;

        RoomData roomData = roomObj.AddComponent<RoomData>();

        // Assign room types
        if (i == 0)
        {
            // First room is always start
            roomData.Initialize(roomBounds[i], RoomType.Start);
        }
        else if (i == roomBounds.Count - 1)
        {
            // Last room (farthest) is boss room
            roomData.Initialize(roomBounds[i], RoomType.Boss);
        }
        else
        {
            // Randomly assign special room types
            float random = Random.value;
            RoomType type;

            if (random < 0.15f)
                type = RoomType.Treasure;
            else if (random < 0.25f)
                type = RoomType.Trap;
            else if (random < 0.30f)
                type = RoomType.Shop;
            else
                type = RoomType.Normal;

            roomData.Initialize(roomBounds[i], type);
        }

        roomDataList.Add(roomData);
        Debug.Log($"Room {i}: {roomData.roomType}");
    }
}

// NEW METHOD: Generate based on room types
void GenerateEntitiesWithRoomTypes()
{
    foreach (RoomData room in roomDataList)
    {
        GenerateRoomEntities(room);
    }
}

void GenerateRoomEntities(RoomData room)
{
    switch (room.roomType)
    {
        case RoomType.Normal:
            GenerateCoinsForRoom(room.bounds, room.coinMultiplier);
            GenerateEnemiesForRoom(room.bounds, room.enemyMultiplier);
            GenerateHealthPotionsForRoom(room.bounds, 2);
            break;

        case RoomType.Treasure:
            GenerateCoinsForRoom(room.bounds, room.coinMultiplier);
            GenerateHealthPotionsForRoom(room.bounds, 5);
            // Spawn locked chest or special items
            break;

        case RoomType.Trap:
            GenerateCoinsForRoom(room.bounds, room.coinMultiplier);
            GenerateEnemiesForRoom(room.bounds, room.enemyMultiplier);
            // TODO: Add trap prefabs
            break;

        case RoomType.Shop:
            // TODO: Spawn shop NPC and purchasable items
            Debug.Log("Shop room - no enemies");
            break;

        case RoomType.Boss:
            // Boss already placed separately
            break;

        case RoomType.Start:
            // Safe room - maybe spawn extra health potions
            GenerateHealthPotionsForRoom(room.bounds, 3);
            break;
    }
}

// Helper methods
void GenerateCoinsForRoom(BoundsInt bounds, int multiplier)
{
    // Call existing coin generator with multiplier
    int coinCount = 5 * multiplier;
    // ... generate coins ...
}

void GenerateEnemiesForRoom(BoundsInt bounds, int multiplier)
{
    int enemyCount = 3 * multiplier;
    // ... generate enemies ...
}

void GenerateHealthPotionsForRoom(BoundsInt bounds, int count)
{
    // ... generate health potions ...
}
```

**Step 4: Visual Indicators for Room Types**

Create different colored overlays or floor tiles for different room types:

```csharp
// In RoomData.cs, add this method:
void Start()
{
    SetRoomVisuals();
}

void SetRoomVisuals()
{
    // Get tilemap
    Tilemap tilemap = GameObject.Find("Floor Tilemap").GetComponent<Tilemap>();

    // Apply colored overlay based on type
    Color overlayColor = Color.white;

    switch (roomType)
    {
        case RoomType.Treasure:
            overlayColor = new Color(1f, 1f, 0.5f, 0.3f); // Yellow tint
            break;
        case RoomType.Trap:
            overlayColor = new Color(1f, 0.5f, 0.5f, 0.3f); // Red tint
            break;
        case RoomType.Shop:
            overlayColor = new Color(0.5f, 1f, 0.5f, 0.3f); // Green tint
            break;
    }

    // Apply color to all tiles in room bounds
    for (int x = bounds.xMin; x < bounds.xMax; x++)
    {
        for (int y = bounds.yMin; y < bounds.yMax; y++)
        {
            Vector3Int pos = new Vector3Int(x, y, 0);
            if (tilemap.HasTile(pos))
            {
                tilemap.SetTileFlags(pos, TileFlags.None);
                tilemap.SetColor(pos, overlayColor);
            }
        }
    }
}
```

**Step 5: Test**

1. Play the game
2. Check Console for room type assignments
3. Notice different rooms have different entity distributions
4. Treasure rooms should have more coins, no enemies
5. Trap rooms should have more enemies

💡 **Enhancement Ideas:**
- Add doors that must be unlocked
- Mini-boss rooms
- Puzzle rooms with rewards
- Secret rooms behind breakable walls
- Room difficulty scaling based on distance from start

---

### Modification 3.2: Add Procedural Room Decorations

🎯 **Goal:** Add decorative elements that make rooms feel unique

🔧 **Steps:**

**Step 1: Create Decoration Prefabs**

Create simple decoration objects:

1. **Pillar:**
   - Empty GameObject
   - Sprite Renderer (pillar sprite)
   - Box Collider 2D (not trigger)
   - Tag: "Prop"

2. **Torch:**
   - Empty GameObject
   - Sprite Renderer (torch sprite)
   - Add Light2D component (if using URP)
   - Tag: "Decoration"

3. **Furniture:**
   - Tables, chairs, barrels, etc.
   - Similar setup to Pillar

**Step 2: Create DecorationGenerator.cs**

Create `/Assets/_Scripts/ProceduralGeneration/DecorationGenerator.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class DecorationGenerator : MonoBehaviour
{
    [Header("Decoration Prefabs")]
    public GameObject[] wallDecorations;  // Torches, banners
    public GameObject[] floorDecorations; // Pillars, furniture
    public GameObject[] cornerDecorations; // Special corner pieces

    [Header("Settings")]
    public float decorationDensity = 0.3f; // 0-1, chance per tile
    public float minSpacing = 2f;

    public void GenerateDecorations(List<RoomData> rooms, Tilemap floorTilemap)
    {
        foreach (RoomData room in rooms)
        {
            GenerateRoomDecorations(room, floorTilemap);
        }
    }

    void GenerateRoomDecorations(RoomData room, Tilemap floorTilemap)
    {
        BoundsInt bounds = room.bounds;

        // Place corner decorations
        PlaceCornerDecorations(bounds);

        // Place wall decorations along walls
        PlaceWallDecorations(bounds);

        // Place random floor decorations
        PlaceFloorDecorations(bounds, floorTilemap);
    }

    void PlaceCornerDecorations(BoundsInt bounds)
    {
        if (cornerDecorations.Length == 0) return;

        Vector2Int[] corners = new Vector2Int[]
        {
            new Vector2Int(bounds.xMin + 1, bounds.yMin + 1), // Bottom-left
            new Vector2Int(bounds.xMax - 1, bounds.yMin + 1), // Bottom-right
            new Vector2Int(bounds.xMin + 1, bounds.yMax - 1), // Top-left
            new Vector2Int(bounds.xMax - 1, bounds.yMax - 1)  // Top-right
        };

        foreach (Vector2Int corner in corners)
        {
            if (Random.value < 0.5f) // 50% chance per corner
            {
                GameObject decoration = Instantiate(
                    cornerDecorations[Random.Range(0, cornerDecorations.Length)],
                    (Vector2)corner,
                    Quaternion.identity
                );
                decoration.transform.parent = transform;
            }
        }
    }

    void PlaceWallDecorations(BoundsInt bounds)
    {
        if (wallDecorations.Length == 0) return;

        // Place along top and bottom walls
        for (int x = bounds.xMin + 2; x < bounds.xMax - 2; x += 3)
        {
            // Top wall
            if (Random.value < 0.4f)
            {
                PlaceDecoration(
                    wallDecorations,
                    new Vector2(x, bounds.yMax - 1)
                );
            }

            // Bottom wall
            if (Random.value < 0.4f)
            {
                PlaceDecoration(
                    wallDecorations,
                    new Vector2(x, bounds.yMin + 1)
                );
            }
        }

        // Place along left and right walls
        for (int y = bounds.yMin + 2; y < bounds.yMax - 2; y += 3)
        {
            // Left wall
            if (Random.value < 0.4f)
            {
                PlaceDecoration(
                    wallDecorations,
                    new Vector2(bounds.xMin + 1, y)
                );
            }

            // Right wall
            if (Random.value < 0.4f)
            {
                PlaceDecoration(
                    wallDecorations,
                    new Vector2(bounds.xMax - 1, y)
                );
            }
        }
    }

    void PlaceFloorDecorations(BoundsInt bounds, Tilemap floorTilemap)
    {
        if (floorDecorations.Length == 0) return;

        // Get all floor tiles in room (excluding edges)
        List<Vector2Int> floorTiles = new List<Vector2Int>();
        for (int x = bounds.xMin + 2; x < bounds.xMax - 2; x++)
        {
            for (int y = bounds.yMin + 2; y < bounds.yMax - 2; y++)
            {
                Vector3Int tilePos = new Vector3Int(x, y, 0);
                if (floorTilemap.HasTile(tilePos))
                {
                    floorTiles.Add(new Vector2Int(x, y));
                }
            }
        }

        // Place random decorations
        foreach (Vector2Int tile in floorTiles)
        {
            if (Random.value < decorationDensity)
            {
                // Check spacing
                if (IsValidDecorationPosition(tile))
                {
                    PlaceDecoration(floorDecorations, tile);
                }
            }
        }
    }

    bool IsValidDecorationPosition(Vector2Int position)
    {
        // Check if too close to other decorations
        Collider2D[] overlaps = Physics2D.OverlapCircleAll(
            position,
            minSpacing
        );

        foreach (Collider2D overlap in overlaps)
        {
            if (overlap.CompareTag("Prop") ||
                overlap.CompareTag("Decoration") ||
                overlap.CompareTag("Enemy") ||
                overlap.CompareTag("Player"))
            {
                return false;
            }
        }

        return true;
    }

    void PlaceDecoration(GameObject[] prefabs, Vector2 position)
    {
        GameObject decoration = Instantiate(
            prefabs[Random.Range(0, prefabs.Length)],
            position,
            Quaternion.identity
        );
        decoration.transform.parent = transform;
    }
}
```

**Step 3: Integrate with Dungeon Generator**

Modify `/Assets/_Scripts/Map/RoomFirstDungeonGenerator.cs`:

```csharp
[Header("Decoration")]
public DecorationGenerator decorationGenerator; // NEW

protected override void playRunProceduralGeneration()
{
    // ... existing generation code ...

    // NEW: Generate decorations after everything else
    if (decorationGenerator != null)
    {
        decorationGenerator.GenerateDecorations(roomDataList, floorTilemap);
    }
}
```

**Step 4: Setup in Unity**

1. Create decoration prefabs (pillars, torches, etc.)
2. Create empty GameObject as child of DungeonGenerator
3. Name it "DecorationGenerator"
4. Add DecorationGenerator component
5. Assign prefab arrays
6. Set Decoration Density: 0.2
7. Set Min Spacing: 2
8. In DungeonGenerator, assign this object to Decoration Generator field

**Step 5: Test**

Play the game - rooms should now have decorative elements!

💡 **Tips:**
- Different room types can have different decorations
- Add particle effects to torches
- Make some decorations destructible
- Hide secrets behind movable decorations

---

## Combat System Modifications

### Modification 4.1: Add Knockback to Player Attacks

🎯 **Goal:** Add satisfying knockback when bullets hit enemies

🔧 **Steps:**

**Step 1: Modify Bullet.cs**

Open `/Assets/_Scripts/Bullet.cs`:

```csharp
public class Bullet : MonoBehaviour
{
    public LayerMask whatIsEnemy;
    public int damage = 30;
    public float knockbackForce = 10f; // NEW
    public float knockbackDuration = 0.2f; // NEW

    void OnTriggerEnter2D(Collider2D hitInfo)
    {
        if (((1 << hitInfo.gameObject.layer) & whatIsEnemy) == 0)
            return;

        // Apply damage
        IDamageable damageable = hitInfo.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(damage);
        }

        // NEW: Apply knockback
        ApplyKnockback(hitInfo.gameObject);

        gameObject.SetActive(false);
    }

    void ApplyKnockback(GameObject target)
    {
        Rigidbody2D targetRb = target.GetComponent<Rigidbody2D>();
        if (targetRb != null)
        {
            // Calculate knockback direction (away from bullet)
            Vector2 knockbackDirection = (target.transform.position -
                                         transform.position).normalized;

            // Apply force
            targetRb.AddForce(knockbackDirection * knockbackForce,
                            ForceMode2D.Impulse);

            // Stop knockback after duration
            StartCoroutine(StopKnockback(targetRb, knockbackDuration));
        }
    }

    System.Collections.IEnumerator StopKnockback(Rigidbody2D rb, float delay)
    {
        yield return new WaitForSeconds(delay);

        if (rb != null)
        {
            // Gradually reduce velocity
            rb.velocity *= 0.5f;
        }
    }
}
```

**Step 2: Adjust Enemy Rigidbody**

For knockback to work properly, enemy Rigidbody2D needs these settings:

1. Select Enemy prefabs (Enemy1, Enemy2, Enemy3)
2. In Rigidbody2D component:
   - Body Type: Dynamic
   - Linear Drag: 5 (stops knockback naturally)
   - Angular Drag: 0.05
   - Gravity Scale: 0
   - Constraints: Freeze Rotation Z

**Step 3: Modify Enemy Script to Handle Knockback**

Open `/Assets/_Scripts/Enemy/EnemyScript.cs`:

```csharp
public class EnemyScript : MonoBehaviour, IDamageable
{
    // ... existing variables ...

    [Header("Knockback")]
    private bool isKnockedBack = false; // NEW
    private float knockbackEndTime = 0f; // NEW

    void Update()
    {
        // NEW: Don't pathfind while knocked back
        if (isKnockedBack)
        {
            if (Time.time >= knockbackEndTime)
            {
                isKnockedBack = false;
            }
            else
            {
                return; // Skip pathfinding
            }
        }

        // ... existing pathfinding code ...
    }

    public void TakeDamage(int damage)
    {
        health -= damage;

        // NEW: Enter knockback state
        isKnockedBack = true;
        knockbackEndTime = Time.time + 0.2f;

        if (healthBarEnemy != null)
        {
            healthBarEnemy.UpdateHealthBar(health, maxHealth);
        }

        if (health <= 0)
        {
            Die();
        }
    }

    // ... rest of code ...
}
```

**Step 4: Test**

1. Play the game
2. Shoot enemies - they should be pushed back
3. Adjust knockbackForce in Bullet prefab if too strong/weak

💡 **Enhancement Ideas:**
- Screen shake on impact
- Particle effects at hit point
- Different knockback for different weapons
- Enemies can be knocked into each other
- Knockback into walls causes stun

---

### Modification 4.2: Add Critical Hit System

🎯 **Goal:** Random chance for extra damage with visual feedback

🔧 **Steps:**

**Step 1: Create Critical Hit System**

Modify `/Assets/_Scripts/Bullet.cs`:

```csharp
using UnityEngine;
using TMPro;

public class Bullet : MonoBehaviour
{
    [Header("Damage")]
    public int damage = 30;
    public float critChance = 0.15f; // 15% chance
    public float critMultiplier = 2f; // 2x damage

    [Header("Visual Feedback")]
    public GameObject damageTextPrefab; // NEW: Floating damage numbers
    public Color normalDamageColor = Color.white;
    public Color critDamageColor = Color.yellow;

    void OnTriggerEnter2D(Collider2D hitInfo)
    {
        if (((1 << hitInfo.gameObject.layer) & whatIsEnemy) == 0)
            return;

        // Calculate damage
        int finalDamage = damage;
        bool isCritical = false;

        // Roll for critical hit
        if (Random.value < critChance)
        {
            finalDamage = Mathf.RoundToInt(damage * critMultiplier);
            isCritical = true;
            Debug.Log("CRITICAL HIT!");
        }

        // Apply damage
        IDamageable damageable = hitInfo.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(finalDamage);
        }

        // Show damage number
        ShowDamageNumber(hitInfo.transform.position, finalDamage, isCritical);

        // Apply knockback
        ApplyKnockback(hitInfo.gameObject, isCritical);

        gameObject.SetActive(false);
    }

    void ShowDamageNumber(Vector3 position, int damage, bool isCritical)
    {
        if (damageTextPrefab == null) return;

        // Spawn damage text above enemy
        Vector3 spawnPos = position + Vector3.up * 0.5f;
        GameObject textObj = Instantiate(damageTextPrefab, spawnPos, Quaternion.identity);

        // Set text and color
        TextMeshPro textMesh = textObj.GetComponent<TextMeshPro>();
        if (textMesh != null)
        {
            textMesh.text = damage.ToString();
            textMesh.color = isCritical ? critDamageColor : normalDamageColor;

            if (isCritical)
            {
                textMesh.fontSize = 6; // Larger for crits
                textMesh.text += "!";
            }
            else
            {
                textMesh.fontSize = 4;
            }
        }

        // Destroy after animation
        Destroy(textObj, 1f);
    }

    void ApplyKnockback(GameObject target, bool isCritical)
    {
        Rigidbody2D targetRb = target.GetComponent<Rigidbody2D>();
        if (targetRb != null)
        {
            Vector2 knockbackDirection = (target.transform.position -
                                         transform.position).normalized;

            // Extra knockback on critical hits
            float force = isCritical ? knockbackForce * 1.5f : knockbackForce;

            targetRb.AddForce(knockbackDirection * force, ForceMode2D.Impulse);
        }
    }
}
```

**Step 2: Create Damage Text Prefab**

1. Create Empty GameObject: "DamageText"
2. Add TextMeshPro component (not UI, use 3D)
3. Settings:
   - Font Size: 4
   - Alignment: Center
   - Sorting Layer: UI or above enemies
   - Order in Layer: 100
4. Add script for floating animation:

Create `/Assets/_Scripts/DamageText.cs`:
```csharp
using UnityEngine;
using TMPro;

public class DamageText : MonoBehaviour
{
    public float floatSpeed = 1f;
    public float fadeDuration = 1f;

    private TextMeshPro textMesh;
    private Color originalColor;
    private float timer = 0f;

    void Start()
    {
        textMesh = GetComponent<TextMeshPro>();
        if (textMesh != null)
        {
            originalColor = textMesh.color;
        }
    }

    void Update()
    {
        // Float upward
        transform.Translate(Vector3.up * floatSpeed * Time.deltaTime);

        // Fade out
        timer += Time.deltaTime;
        float alpha = Mathf.Lerp(1f, 0f, timer / fadeDuration);

        if (textMesh != null)
        {
            Color newColor = originalColor;
            newColor.a = alpha;
            textMesh.color = newColor;
        }

        // Destroy when fully faded
        if (timer >= fadeDuration)
        {
            Destroy(gameObject);
        }
    }
}
```

5. Add this script to DamageText prefab
6. Save as prefab

**Step 3: Assign Damage Text Prefab**

1. Select Bullet prefab
2. In Bullet component, assign DamageText prefab
3. Set Crit Chance: 0.15 (15%)
4. Set Crit Multiplier: 2.0 (double damage)
5. Set colors

**Step 4: Test**

Shoot enemies and watch for:
- Yellow damage numbers (critical hits)
- Larger numbers and exclamation mark
- Extra knockback on crits

💡 **Enhancement Ideas:**
- Sound effects for critical hits
- Screen shake on crit
- Crit chance increases with combo
- Lucky items that boost crit chance
- Different crit multipliers per weapon

---

## UI System Modifications

### Modification 5.1: Add Minimap System

🎯 **Goal:** Show a minimap of explored rooms

🔧 **Steps:**

**Step 1: Create Minimap Camera**

1. GameObject → Camera
2. Name it "MinimapCamera"
3. Settings:
   - Projection: Orthographic
   - Orthographic Size: 15 (adjust based on dungeon)
   - Position: (0, 0, 10)
   - Clear Flags: Solid Color
   - Background: Black
   - Culling Mask: Minimap layer only
   - Depth: 1 (render after main camera)
   - Viewport Rect: X: 0.75, Y: 0.75, W: 0.2, H: 0.2 (top-right corner)

**Step 2: Create Minimap Layer**

1. Edit → Project Settings → Tags and Layers
2. Add new layer: "Minimap"

**Step 3: Create Minimap Icons**

Create simple colored sprites for minimap representation:

1. **Room Icon** - Small square (white)
2. **Player Icon** - Green dot
3. **Enemy Icon** - Red dot
4. **Boss Icon** - Large red square
5. **Unexplored** - Gray square

**Step 4: Create MinimapManager Script**

Create `/Assets/_Scripts/UI/MinimapManager.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class MinimapManager : MonoBehaviour
{
    [Header("References")]
    public Transform player;
    public Camera minimapCamera;

    [Header("Icons")]
    public GameObject roomIconPrefab;
    public GameObject playerIconPrefab;
    public GameObject enemyIconPrefab;
    public GameObject bossIconPrefab;

    [Header("Settings")]
    public float iconScale = 0.3f;
    public float updateInterval = 0.5f; // Update enemy positions

    private Dictionary<GameObject, GameObject> entityIcons =
        new Dictionary<GameObject, GameObject>();
    private List<RoomData> rooms = new List<RoomData>();
    private GameObject playerIcon;
    private float nextUpdateTime = 0f;

    void Start()
    {
        // Create player icon
        if (player != null && playerIconPrefab != null)
        {
            playerIcon = Instantiate(playerIconPrefab, transform);
            playerIcon.transform.localScale = Vector3.one * iconScale;
            playerIcon.layer = LayerMask.NameToLayer("Minimap");
        }

        // Find all rooms
        rooms.AddRange(FindObjectsOfType<RoomData>());
        CreateRoomIcons();

        // Make minimap camera follow player
        if (minimapCamera != null && player != null)
        {
            Vector3 camPos = player.position;
            camPos.z = minimapCamera.transform.position.z;
            minimapCamera.transform.position = camPos;
        }
    }

    void Update()
    {
        // Update player icon position
        if (playerIcon != null && player != null)
        {
            Vector3 iconPos = player.position;
            iconPos.z = transform.position.z;
            playerIcon.transform.position = iconPos;
        }

        // Update camera to follow player
        if (minimapCamera != null && player != null)
        {
            Vector3 camPos = player.position;
            camPos.z = minimapCamera.transform.position.z;
            minimapCamera.transform.position = camPos;
        }

        // Periodically update enemy icons
        if (Time.time >= nextUpdateTime)
        {
            UpdateEntityIcons();
            nextUpdateTime = Time.time + updateInterval;
        }
    }

    void CreateRoomIcons()
    {
        if (roomIconPrefab == null) return;

        foreach (RoomData room in rooms)
        {
            GameObject icon = Instantiate(roomIconPrefab, transform);
            icon.transform.position = new Vector3(
                room.center.x,
                room.center.y,
                transform.position.z
            );
            icon.transform.localScale = new Vector3(
                room.bounds.size.x * 0.8f,
                room.bounds.size.y * 0.8f,
                1f
            );
            icon.layer = LayerMask.NameToLayer("Minimap");

            // Initially hidden (fog of war)
            icon.SetActive(false);

            // Store for revealing later
            entityIcons[room.gameObject] = icon;
        }
    }

    void UpdateEntityIcons()
    {
        // Reveal rooms near player
        foreach (RoomData room in rooms)
        {
            float distance = Vector2.Distance(
                player.position,
                room.center
            );

            // Reveal if player is close
            if (distance < 20f && entityIcons.ContainsKey(room.gameObject))
            {
                entityIcons[room.gameObject].SetActive(true);
                room.hasBeenVisited = true;
            }
        }

        // Update enemy icons
        GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
        foreach (GameObject enemy in enemies)
        {
            if (!entityIcons.ContainsKey(enemy))
            {
                // Create new icon
                GameObject icon = Instantiate(enemyIconPrefab, transform);
                icon.layer = LayerMask.NameToLayer("Minimap");
                icon.transform.localScale = Vector3.one * iconScale;
                entityIcons[enemy] = icon;
            }

            // Update position
            if (entityIcons.ContainsKey(enemy))
            {
                Vector3 iconPos = enemy.transform.position;
                iconPos.z = transform.position.z;
                entityIcons[enemy].transform.position = iconPos;
            }
        }

        // Update boss icon
        GameObject boss = GameObject.FindGameObjectWithTag("Boss");
        if (boss != null && !entityIcons.ContainsKey(boss))
        {
            GameObject icon = Instantiate(bossIconPrefab, transform);
            icon.layer = LayerMask.NameToLayer("Minimap");
            icon.transform.localScale = Vector3.one * iconScale;
            entityIcons[boss] = icon;
        }

        // Clean up destroyed enemies
        List<GameObject> toRemove = new List<GameObject>();
        foreach (var kvp in entityIcons)
        {
            if (kvp.Key == null)
            {
                Destroy(kvp.Value);
                toRemove.Add(kvp.Key);
            }
        }
        foreach (var key in toRemove)
        {
            entityIcons.Remove(key);
        }
    }
}
```

**Step 5: Create Icon Prefabs**

Simple sprite objects:
1. Create Empty GameObject
2. Add Sprite Renderer
3. Assign colored sprite
4. Set layer to "Minimap"
5. Save as prefab

**Step 6: Setup in Unity**

1. Add MinimapCamera to scene
2. Create empty GameObject "MinimapManager"
3. Add MinimapManager script
4. Assign all references
5. Test!

💡 **Enhancement Ideas:**
- Click minimap to ping location
- Show treasure chest icons
- Fog of war (unexplored rooms dark)
- Zoom in/out with mouse wheel
- Toggle minimap with key press

---

(Continued in next part due to length...)

## Item System Modifications

### Modification 6.1: Add Weapon Pickup System

🎯 **Goal:** Create collectible weapons scattered in the dungeon

🔧 **Steps:**

**Step 1: Create WeaponPickup Script**

Create `/Assets/_Scripts/WeaponPickup.cs`:

```csharp
using UnityEngine;

public class WeaponPickup : MonoBehaviour
{
    public WeaponData weaponData;
    public GameObject visualEffect;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            PlayerAimWeapon playerWeapon = other.GetComponentInChildren<PlayerAimWeapon>();

            if (playerWeapon != null && weaponData != null)
            {
                playerWeapon.AddWeapon(weaponData);
                Debug.Log($"Picked up {weaponData.weaponName}!");

                // Spawn visual effect
                if (visualEffect != null)
                {
                    Instantiate(visualEffect, transform.position, Quaternion.identity);
                }

                Destroy(gameObject);
            }
        }
    }

    void Update()
    {
        // Floating animation
        float bobHeight = Mathf.Sin(Time.time * 2f) * 0.2f;
        transform.position = new Vector3(
            transform.position.x,
            transform.position.y + bobHeight * Time.deltaTime,
            transform.position.z
        );

        // Rotation animation
        transform.Rotate(Vector3.forward * 50f * Time.deltaTime);
    }
}
```

**Step 2: Modify PlayerAimWeapon to Support Dynamic Weapons**

Add method to `/Assets/_Scripts/Player/PlayerAimWeapon.cs`:

```csharp
public void AddWeapon(WeaponData newWeapon)
{
    // Check if already have this weapon
    foreach (WeaponData weapon in weapons)
    {
        if (weapon == newWeapon)
        {
            Debug.Log("Already have this weapon!");
            return;
        }
    }

    // Add to weapons array
    List<WeaponData> weaponList = new List<WeaponData>(weapons);
    weaponList.Add(newWeapon);
    weapons = weaponList.ToArray();

    // Switch to new weapon
    SwitchWeapon(weapons.Length - 1);

    Debug.Log($"Added {newWeapon.weaponName} to inventory!");
}
```

**Step 3: Create Weapon Pickup Prefabs**

1. Create Empty GameObject "WeaponPickup_Shotgun"
2. Add Sprite Renderer (weapon icon)
3. Add Circle Collider 2D (Is Trigger: true)
4. Add WeaponPickup script
5. Assign WeaponData (the shotgun ScriptableObject you created earlier)
6. Save as prefab
7. Repeat for other weapons

**Step 4: Spawn in Treasure Rooms**

Modify room generation to spawn weapons:

In `/Assets/_Scripts/Map/RoomFirstDungeonGenerator.cs`:

```csharp
void GenerateRoomEntities(RoomData room)
{
    switch (room.roomType)
    {
        case RoomType.Treasure:
            GenerateCoinsForRoom(room.bounds, room.coinMultiplier);
            GenerateHealthPotionsForRoom(room.bounds, 5);
            GenerateWeaponPickup(room.bounds); // NEW
            break;
        // ... other cases ...
    }
}

void GenerateWeaponPickup(BoundsInt bounds)
{
    // Find center of room
    Vector2 centerPosition = new Vector2(
        (bounds.xMin + bounds.xMax) / 2f,
        (bounds.yMin + bounds.yMax) / 2f
    );

    // Spawn random weapon
    GameObject[] weaponPickups = // Assign in Inspector
    {
        shotgunPickupPrefab,
        riflePickupPrefab,
        // ... other weapons
    };

    if (weaponPickups.Length > 0)
    {
        GameObject pickup = weaponPickups[Random.Range(0, weaponPickups.Length)];
        Instantiate(pickup, centerPosition, Quaternion.identity);
    }
}
```

---

## Summary

This modding guide provides step-by-step instructions for modifying:
- ✅ Player systems (dash, stamina, weapons)
- ✅ Enemy systems (ranged enemies, boss phases)
- ✅ Dungeon generation (room types, decorations)
- ✅ Combat systems (knockback, critical hits)
- ✅ UI systems (minimap)
- ✅ Item systems (weapon pickups)

### Next Steps

1. **Practice:** Try each modification in order
2. **Experiment:** Combine multiple modifications
3. **Create:** Design your own unique features
4. **Share:** Contribute your improvements!

### Additional Resources

- `ARCHITECTURE.md` - Understand the codebase structure
- `QUICKSTART.md` - Setup and basic modifications
- Unity Documentation - https://docs.unity3d.com/
- C# Reference - https://docs.microsoft.com/en-us/dotnet/csharp/

Happy modding! 🎮✨
