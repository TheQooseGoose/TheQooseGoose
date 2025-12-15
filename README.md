# Alexander Forsell

> Status: Living portfolio - Systems are *actively* under development and iteration.

Unreal Engine 5 gameplay programmer focused on building core gameplay systems.

This repository serves as a living portfolio and development log, showcasing work-in-progress systems such as weapons, AI, and dynamic world generation. Each section documents what is implemented, what is planned, known issues, and areas marked for refactoring or improvement.

All projects shown here are actively developed and intentionally presented as snapshots of the iteration process rather than finished products.

---

## Weapon System
**Video Demo:** (unlisted YouTube link)

### Design Rationale
-

### Done
- Parent weapon class (stores the main logic), child classes inherit the logic and use their own variables such as "damage," "magazineCapacity" and "RPS." Each weapon can behave differently just by changing variables.
- Completed separate paths to handle different behavior between full-auto, semi-auto, and bolt-action weapons.
- Spawning projectiles when the player *has* ammo
- Projectiles travel where the player is looking (if we disregard bullet-drop), regardless of using first-person or third-person camera.
- Replicated projectiles. (Fixed issue where client would watch the host fire two bullets in two different directions).
- Animation montages (weapon recoil) when firing and not out of ammo. Different montage for different positions. Ex: Prone firing vs. Stand firing
- AI dealt damage based on the struck bone in the skeletal mesh. Ex: A hit to a limb does half damage. A hit to the torso does normal damage. A hit to the head deals double damage.

### Planned
- Ammo drops/pickups
- Bayonets/melee
- More weapons and weapon MODELS
- Reload animation montages
- Sound (Muzzle blast, reloading, projectile-whistle), VA for characters ("I need ammo!", etc)
- Particles (Muzzle blast, shell-eject)
- Aim offset (reduction in accuracy) when moving or sprinting

### Known Bugs
- Projectiles *have* collision and fully interact with characters and blueprints, but projectiles pass straight through HISMs (trees, rocks, etc) and the landscape.

### Needs Updating
- Animation Montage section. Uses a lot of branches, could be done WAY more efficiently.
- Extra comments and cleanup would not hurt.
  
---

- ## Basic AI
**Video Demo:** (unlisted YouTube link)

### Design Rationale

### Done
- Patrol Behavior:
  - To always be relevant, patrols are anchored around the *nearest* (anchors can change) player.
  - Go to a randomly selected point within the bounds of the area anchored to the nearest player. Repeat until entering combat.
- Becoming Alert:
  - Has different "alert" ranges depending on what the player in the vision-cone is doing (Struggles to find prone players, moderate at finding crouching players, easily sees standing or running players).
  - Can become alert by detecing a bullet from the player. Player does not need to get a hit or a kill to alert the enemy. The mere presence of a bullet will put the AI into the combat branch.
- Combat Behavior:
  - ... 

### Planned
- Self-preservation behavior. Actively seek cover when alert && the player is looking in their direction.
- Offensive Behavior. When alert && the player is *not* looking in their direction, advance toward the player.
- Squad Behavior. Officers are the brain of the squad and direct movement. Foot soldiers will be tied to a limited area surrounding their officer. When the officer dies, the movement restrictions are gone and the squad should *theoretically* dissolve without order.
- Anti-crowding. When seeking cover, prevent AI from crowding into a single spot. If no nearby cover is available, crouch-in-place.
- Morale. If the AI squad takes too many casualties too quickly or loses their officer, squads will either be fearful and rout (and later despawn), or be invigorated and bayonet-charge.
- Behavioral differences based on enemy faction. One faction might have AI that is hyper-aggressive, whereas another faction might rely on stalking and ambushes.
- LOTS of VA. (Keep the AI shouting. War is loud and chaotic).
- AI Spawner tied to the player. If there are 2 players, there are two spawners. Spawn AI ~1Km from the anchored player and do not spawn near a *different* player. 

### Known Bugs
- 

### Needs Updating
- Combat Behavior has no substance. Currently, it is just "Go to 0.00, 0.00, 0.00."
- Experiment with different patrol ranges. Larger area = less-likely to encounter player.

---

## World Generation
**Video Demo:** (unlisted YouTube link)

### Design Rationale

### Done
- (0) Sharing the world-seed with the client upon client-login
  - (0.5) If not the host, wait to receive server-data. Do NOT start the spawners UNTIL all major BPs have been successfully received). Fixes replication bug where a tree (or something) might exist in one place for the host, but exist in another place for the client. 
- (1) Mission-Objective Spawning. Place a random number of mission objectives in random locations. Store those locations.
- (2) Using the stored locations, randomly choose between spawning one of these *around* a mission objective: Town, Forest, Military Base, Nothing (empty field).
- (3) 2D Noise, Seed-Based Forest Generator. (Randomly populates landscape with clusters of trees and bushes).
- (4) Seed-Based House Spawner. (Places a random number of houses in random locations). 
- (5) Seed-Based Foliage Generator. (Places trees, bushes and rocks in random locations and in a random density).
- Universal Landscape Material. Spawns grass and flower meshes atop itself, no need to manually paint. The steeper the slope, the yellower (dryer) the terrain.
- Completed logic for a simple "Capture and Hold" objective.
- Complex foliage materials. Using Runtime Virtual Textures, foliage like grass, bushes, and flowers inherit their colors from the landscape beneath it. Sloped, yellow hills will have yellow foliage.

### Planned
- Dynamic landscape generator. Rather than importing different heightmap-scans of random fields in rural Canada, dynamically generate the landscape using the seed. Massive project, but it would make each level more dynamic.
- Rivers. It would be nice to have mission objectives to seize, defend, or demolish bridges. 
- More mission objectives, static or otherwise (Such as "Kill this Officer," "Destroy this Convoy," "Steal this," "Sabotage that," "Destroy these," "Liberate Those.")
- More house models
- Mission objective models
- Dynamic cities?
- *Somehow* figure out how to make dynamic roads/paths that connect random towns/houses.

### Known Bugs
- Foliage penetrates through things like rocks, houses/floors, and characters.

### Needs Updating
- Towns and military bases spawn in perfect squares. Add irregularity.
- Forests (around mission objectives) are perfect circles. Add irregularity.

---
  
Email: alexforsell354@gmail.com
LinkedIn: https://www.linkedin.com/in/alex-forsell-a810ba364/
