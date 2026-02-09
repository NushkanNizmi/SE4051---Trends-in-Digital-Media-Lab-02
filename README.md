# Lab 02- Building an AI-Responsive Interactive Environment in Unity

## Overview
In this lab, you will build a small Unity scene where the environment behaves like a “smart” system:
- It detects when a player enters a zone
- It changes lighting and plays an alert sound
- An NPC reacts (approaches the player) when the player is nearby

This lab is fully self-study. Follow the steps in order.

## Duration
2 Hours

## What you will have at the end (Final Output)
When you press Play:
- You can move a Player using WASD
- When the Player enters a Detection Zone:
  - The light changes color
  - An alert sound plays (if you assign a clip)
- An NPC starts moving toward the Player when within a chosen distance
- You can enable debug logs to verify triggers and distances

## Requirements
- Unity Hub
- Unity 2022.3 LTS
- A new “3D Core” project
- No external packages

## Folder Setup (Recommended)
Inside the Project window, create folders:
- Assets/Scripts
- Assets/Audio (optional)

---

# Part A — Create the Project and Scene (15 minutes)

## Step A1 — Create a new Unity project
1. Open Unity Hub → New Project
2. Select: 3D Core
3. Name: `AI_Responsive_Lab`
4. Create

## Step A2 — Create the ground
1. Hierarchy → Right click → 3D Object → Plane
2. Rename to: `Ground`
3. Reset Transform (Inspector gear icon → Reset)

## Step A3 — Create the Player
1. Hierarchy → Right click → 3D Object → Capsule
2. Rename: `Player`
3. Set Position:
   - X = 0, Y = 1, Z = 0
4. Tag the Player:
   - Select Player → Inspector → Tag → Add Tag…
   - Click + → create new tag: `Player`
   - Select Player again → Tag = `Player`

## Step A4 — Create the NPC
1. Hierarchy → Right click → 3D Object → Cube
2. Rename: `NPC`
3. Set Position:
   - X = 0, Y = 1, Z = 8

## Step A5 — Create a “Detection Zone”
1. Hierarchy → Right click → Create Empty
2. Rename: `DetectionZone`
3. Set Position:
   - X = 0, Y = 1, Z = 4
4. Add a Box Collider:
   - Select DetectionZone → Add Component → Box Collider
   - Tick: Is Trigger
5. Resize the zone collider:
   - Box Collider → Size:
     - X = 6, Y = 2, Z = 6

Checkpoint:
- You should see Player, NPC, and a trigger zone (in Scene view you’ll see the collider outline).

---

# Part B — Player Movement (20 minutes)

## Step B1 — Create Player Movement script
1. Project → Assets/Scripts → Right click → Create → C# Script
2. Name: `PlayerMover`
3. Double click and replace the code with:

```csharp
using UnityEngine;

public class PlayerMover : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed = 5f;

    [Header("Debug")]
    public bool enableDebug = false;

    void Update()
    {
        float h = Input.GetAxis("Horizontal"); // A/D or Left/Right
        float v = Input.GetAxis("Vertical");   // W/S or Up/Down

        Vector3 move = new Vector3(h, 0f, v) * moveSpeed * Time.deltaTime;
        transform.Translate(move, Space.World);

        if (enableDebug && (Mathf.Abs(h) > 0.01f || Mathf.Abs(v) > 0.01f))
        {
            Debug.Log($"Player position: {transform.position}");
        }
    }
}
```
## Step B2 — Attach Movement Script to Player

1. Select `Player`
2. Drag the `PlayerMover` script onto the Player in the Inspector  
   (or use **Add Component → PlayerMover**)

### Checkpoint
- Press **Play**
- Use **W A S D** keys to move the Player on the ground

If the player does not move:
- Click the **Game** window once (keyboard input goes to the active window)

---

# Part C — Smart Zone Detection (AI-like detection) (25 minutes)

## Step C1 — Create an Alert Light

1. Hierarchy → Right Click → **Light → Directional Light**
2. Rename it: `AlertLight`

3. Set Rotation (optional, any is fine)

---

## Step C2 — Create Detector Script

1. `Assets/Scripts` → Right Click → **Create → C# Script**
2. Name it:`ZoneDetector`


3. Open the script and replace everything with the following code:

```csharp
using UnityEngine;

public class ZoneDetector : MonoBehaviour
{
    [Header("References")]
    public Light alertLight;
    public AudioSource alertAudio;

    [Header("Light Colors")]
    public Color normalColor = Color.white;
    public Color alertColor = Color.red;

    [Header("Debug")]
    public bool enableDebug = true;

    void Start()
    {
        if (alertLight != null)
        {
            alertLight.color = normalColor;
        }
    }

    void OnTriggerEnter(Collider other)
    {
        if (!other.CompareTag("Player")) return;

        if (enableDebug) Debug.Log("Player entered DetectionZone");

        if (alertLight != null)
        {
            alertLight.color = alertColor;
        }

        if (alertAudio != null)
        {
            if (!alertAudio.isPlaying)
                alertAudio.Play();
        }
    }

    void OnTriggerExit(Collider other)
    {
        if (!other.CompareTag("Player")) return;

        if (enableDebug) Debug.Log("Player exited DetectionZone");

        if (alertLight != null)
        {
            alertLight.color = normalColor;
        }

        if (alertAudio != null)
        {
            if (alertAudio.isPlaying)
                alertAudio.Stop();
        }
    }
}
```
## Step C3 — Add AudioSource (Optional but Recommended)

1. Hierarchy → Right Click → **Audio → Audio Source**
2. Rename it: `AlertSound`

3. Select `AlertSound` → Inspector:
   - Untick **Play On Awake**

4. Add an audio clip:
   - If you have a short `.wav` or `.mp3` file, drag it into: `Assets/Audio`
   - Then drag the clip into: `AlertSound → AudioClip`


If you do not have a sound clip:
- You can continue without sound. The lab will still function correctly.

---

## Step C4 — Attach ZoneDetector to DetectionZone

1. Select `DetectionZone`
2. Click **Add Component → ZoneDetector**
3. Assign references:
   - Drag `AlertLight` into the **Alert Light** slot
   - Drag `AlertSound` into the **Alert Audio** slot (if created)

---

### Checkpoint

- Press **Play**
- Move the Player into the zone
- The light should turn **red**
- The Console should display: `Player entered DetectionZone`

- Move out of the zone:
  - Light returns to **white**
  - Exit log appears in the Console


# Part D — NPC Reaction (Simple “AI” Behavior) (30 Minutes)

## Step D1 — Create NPC Behavior Script

1. Navigate to:`Assets/Scripts`

2. Right Click → **Create → C# Script**

3. Name the script: `SimpleNPC`


4. Open the script and replace everything with the following code:

```csharp
using UnityEngine;

public class SimpleNPC : MonoBehaviour
{
    [Header("References")]
    public Transform player;

    [Header("Behavior")]
    public float chaseDistance = 6f;
    public float moveSpeed = 2f;

    [Header("Debug")]
    public bool enableDebug = false;

    void Update()
    {
        if (player == null) return;

        float distance = Vector3.Distance(transform.position, player.position);

        if (enableDebug)
        {
            Debug.Log($"NPC distance to player: {distance:F2}");
        }

        if (distance <= chaseDistance)
        {
            // Look at player (only rotate on Y axis)
            Vector3 lookPos = new Vector3(player.position.x, transform.position.y, player.position.z);
            transform.LookAt(lookPos);

            // Move toward player
            transform.Translate(Vector3.forward * moveSpeed * Time.deltaTime);
        }
    }
}
```

## Step D2 — Attach SimpleNPC to NPC

1. Select `NPC`
2. Click **Add Component → SimpleNPC**
3. Assign the reference:
   - Drag `Player` (from the Hierarchy) into the **Player** field in the Inspector

---

### Checkpoint

- Press **Play**
- Move closer to the NPC
- When you are within `chaseDistance`, the NPC should start moving toward you

---

### If the NPC Does Not Move

Check the following:

- Confirm the **Player** reference is assigned in the NPC Inspector
- Make sure the Player is actually moving into range
- Increase `chaseDistance` to **10** and test again
- Ensure there are no errors shown in the Unity Console


# Part E — Final Integration Tasks (20 Minutes)

## Task E1 — Modify Zone Behavior

Update the light behavior inside the `ZoneDetector` component.

Steps:

- Select `DetectionZone`
- In the Inspector → Locate the **ZoneDetector** component
- Change the following values:
  - `alertColor` → Set to **Yellow** or **Blue**
  - `normalColor` → Set to **White**

Press **Play** and test the changes.

---

## Task E2 — Add a Second Detection Zone

1. Duplicate the existing zone: `Select DetectionZone → Ctrl + D`

2. Rename it: `DetectionZone_2`

3. Move it to a different location:

Example: `Position → Z = 12`


---

### Create a Second Alert Light

1. Duplicate the existing light: `Select AlertLight → Ctrl + D`

2. Rename it: `AlertLight2`

---

### Assign the New Light

- Select `DetectionZone_2`
- In the **ZoneDetector** component:
  - Drag `AlertLight2` into the **Alert Light** field
  - Change `alertColor` to a different color than the first zone

---

### Checkpoint

When you press **Play**:

- Enter Zone 1 → Light changes to Color A  
- Enter Zone 2 → Light changes to Color B  

You should now have **two zones with two different behaviors**.

---

## Task E3 — Adjust NPC Safe Distance

Select the `NPC`.

In the **SimpleNPC** component:

Set: `moveSpeed = 1.5` | ` chaseDistance = 8`


Press **Play** and observe:

- The NPC detects you from farther away
- The movement is slower and more controlled

Observe how small parameter changes affect system behavior.

# Debug and Troubleshooting (Read if Something Fails)

## Trigger Does Not Fire

Check the following:

- `DetectionZone` has a **Box Collider** and **Is Trigger** is ticked
- `Player` has a **Collider** (Capsule has one by default)
- At least one object has a **Rigidbody** (recommended on the Player)

### Add Rigidbody to Player (Recommended)

1. Select `Player`
2. **Add Component → Rigidbody**
3. Keep **Use Gravity** enabled (default)
4. Freeze rotations to avoid tipping:
   - Rigidbody → Constraints → Tick:
     - **Freeze Rotation X**
     - **Freeze Rotation Z**

---

## Console Shows No Messages

- Ensure `enableDebug` in `ZoneDetector` is set to **true**
- Ensure the Console is visible:
  - Window → General → Console

---

## Light Does Not Change

- Ensure `AlertLight` is assigned in the `ZoneDetector` Inspector
- Ensure you created a **Directional Light** named `AlertLight`

---

## NPC Does Not Move

- Ensure `player` is assigned in the `SimpleNPC` Inspector
- Increase `chaseDistance` (example: 10) and test again
- Make sure you are pressing **Play** and moving the Player closer to the NPC

---

# Submission (What to Submit)

Submit the Unity project including: 
`Assets/`
`Packages/`
`ProjectSettings/`
`.gitignore`


---

# Reflection Questions (Submit)

1. What does the Detection Zone simulate in a “smart environment”?
2. How does the NPC decide when to move?
3. What changes made the environment feel more responsive?
4. If you had real AI, what would you improve in this scene?






