# Alexander Forsell

> Status: Living portfolio - Systems are *actively* under development.

Unreal Engine 5 gameplay programmer focused on building core gameplay systems.

This repository serves as a living portfolio and development log, showcasing work-in-progress systems such as weapons, AI, and dynamic world generation. Each section documents what is implemented, what is planned, known issues, and areas marked for refactoring or improvement.

All projects shown here are actively developed and intentionally presented as snapshots of the iteration process rather than finished products.

---

## Design Constraints
- Solo Development: All gameplay systems were designed and implemented independently, with external references used only for unfamiliar engine-specific concepts.
- Art Scope Separation: The project prioritizes gameplay systems and logic over original asset creation. Most visual assets (e.g., foliage and rocks) were sourced externally and modified as needed.

---

## Weapon System
**Video Demo:** (unlisted YouTube link)

### Design Rationale
- Different weapons will have different behaviors. A Semi-Auto rifle is quite simple because it is a simple matter of clicking the mouse and spawning a projectile while there is still ammo. A full-auto rifle is hold-and-release, but I also need different values for RPS (Rounds per Second) so that each full-auto weapon has different rates of fire. And a bolt-action rifle requires manual cycling, so, after a shot, if the weapon is not out of ammo, the 'r' key is used not to "reload" the weapon, but to cycle the next round into the chamber. 

### Done
- Parent weapon class (stores the main logic), child classes inherit the logic and use their own variables such as "damage," "magazineCapacity" and "RPS." Each weapon can behave differently just by changing variables.
- Completed separate paths to handle different behavior between full-auto, semi-auto, and bolt-action weapons.
- Spawning projectiles when the player *has* ammo
- Projectiles travel where the player is looking (if we disregard bullet-drop), regardless of using first-person or third-person camera.
- Replicated projectiles. (Fixed issue where client would watch the host fire two bullets in two different directions).
- Animation montages (weapon recoil) when firing and not out of ammo. Different montage for different positions. Ex: Prone firing vs. Stand firing
- AI dealt damage based on the struck bone in the skeletal mesh. Ex: A hit to a limb does half damage. A hit to the torso does normal damage. A hit to the head deals double damage.
- Prevent player from discarding a full magazine. When discaring a non-empty magazine, the ammo is wasted.

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
- Animation Montage section. Uses a lot of branches; could be done WAY more efficiently.
- Extra comments and cleanup would not hurt.
  
---

- ## Basic AI
**Video Demo:** (unlisted YouTube link)

### Design Rationale
- Maybe I do not have enough variety in my games, but I have never encountered a game where the AI has some shred of self-preservation. I want my AI to have realistic behavior, which means that the AI should prioritize *living*, so that one day, if it is lucky, it can go back home to see its AI wife and AI children. If the AI wants to get a kill or put pressure on the player, its best chance at doing so would be to stay alive. So, rather than stand right in the open without a lick of cover, as I see in many AI, I want *my* AI to actively seek cover so it can avoid becoming what doctors would call a red paste. If the AI is threatened or actively being shot at, then the AI should seek cover and *maybe* fire their weapon from their safe(r) position. Otherwise, if the player is not facing the AI, that is an opportune moment to safely apply pressure by advancing and firing. 

### Done
- Patrol Behavior:
  - To always be relevant, patrols are anchored around the *nearest* (anchors can change) player.
  - Go to a randomly selected point within the bounds of the area anchored to the nearest player. Repeat until entering combat.
- Becoming Alert:
  - Has different "alert" ranges depending on what the player in the vision cone is doing (Struggles to find prone players, is moderate at finding crouching players, easily sees standing or running players).
  - Can become alert by detecting a bullet from the player. The player does not need to get a hit or a kill to alert the enemy. The mere presence of a bullet will put the AI into the combat branch.
- Combat Behavior:
  - ... 

### Planned
- Self-preservation behavior. Actively seek cover when alert && the player is looking in their direction.
- Offensive Behavior. When alert && the player is *not* looking in their direction, advance toward the player.
- Squad Behavior. Officers are the brain of the squad and direct movement. Foot soldiers will be tied to a limited area surrounding their officer. When the officer dies, the movement restrictions are gone, and the squad should *theoretically* dissolve without order.
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
- Games with purpose-built levels *do* generally have better layouts, I agree. *However,* they also suffer because they can get old. Once you play the same level a few times over, it can get quite boring. And in competitive games, if nothing ever changes, there might be some unconsidered geometry that provides an exploitable location. For example, climbing on top of a building or a ledge that was never meant to be climbed. If everything is always dynamically generated, then there is always going to be something different to keep the game fresh and interesting. And unlike prebuilt levels, if I want more variety, I can always go back and expand an array of houses or tree types, and I can mess around with variables to get different results as needed. Dynamic worlds are always expandable with new content, such as new models, in a way that purpose-built levels are not. 

### Done
- (0) Sharing the world-seed with the client upon client login
  - (0.5) If not the host, wait to receive server-data. Do NOT start the spawners UNTIL all major BPs have been successfully received. Fixes replication bug where a tree (or something) might exist in one place for the host, but exist in another place for the client. 
- (1) Mission-Objective Spawning. Place a random number of mission objectives in random locations. Store those locations.
- (2) Using the stored locations, randomly choose between spawning one of these *around* a mission objective: Town, Forest, Military Base, Nothing (empty field).
- (3) 2D Noise, Seed-Based Forest Generator. (Randomly populates landscape with clusters of trees and bushes).
- (4) Seed-Based House Spawner. (Places a random number of houses in random locations). 
- (5) Seed-Based Foliage Generator. (Places trees, bushes, and rocks in random locations and in a random density).
- Universal Landscape Material. Spawns grass and flower meshes atop itself, no need to manually paint. The steeper the slope, the yellower (drier) the terrain.
- Completed logic for a simple "Capture and Hold" objective.
- Complex foliage materials. Using Runtime Virtual Textures, foliage like grass, bushes, and flowers inherit their colors from the landscape beneath it. Sloped, yellow hills will have yellow foliage.

### Planned
- Dynamic landscape generator. Rather than importing different heightmap-scans of random fields in rural Canada, dynamically generate the landscape using the seed. Massive project, but it would make each level more dynamic.
- Rivers. It would be nice to have mission objectives to seize, defend, or demolish bridges. 
- More mission objectives, static or otherwise (Such as "Kill this Officer," "Destroy this Convoy," "Steal this," "Sabotage that," "Destroy these," "Liberate Those.")
- More house models
- Mission objective models
- Dynamic "cities" (Larger towns with slightly larger building models)?
- *Somehow* figure out how to make dynamic roads/paths that connect random towns/houses.

### Known Bugs
- Foliage penetrates through things like rocks, houses/floors, and characters, despite my attempts to fix.

### Needs Updating
- Towns and military bases spawn in perfect squares. Add irregularity.
- Forests (around mission objectives) are perfect circles. Add irregularity.

---
  
Email: alexforsell354@gmail.com
LinkedIn: https://www.linkedin.com/in/alex-forsell-a810ba364/
