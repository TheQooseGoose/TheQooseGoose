# Alexander Forsell

> Status: Living portfolio - Systems are *actively* under development.

Unreal Engine 5 gameplay programmer focused on building core gameplay systems.

This repository serves as a living portfolio and development log, showcasing work-in-progress systems such as weapons, AI, and dynamic world generation. Each section documents what is implemented, what is planned, known issues, and areas marked for refactoring or improvement.

All projects shown here are actively developed and intentionally presented as snapshots of the iteration process rather than finished products.

---

### Project Context

This project serves as a systems-focused demonstration of my ability to design and implement gameplay mechanics within a game engine. It exists to support and validate my independent study by providing concrete evidence of hands-on game development experience through working systems, documented design decisions, and problem-solving.
The primary focus is on gameplay logic and system architecture, including weapons, AI behavior and spawning, and dynamic world generation. Unreal Engine 5 is used as the chosen implementation environment, but the emphasis is placed on transferable design principles, such as state-driven logic or system extensibility, rather than engine-specific features. Visuals are intentionally deprioritized in favor of functionality. 

---

## Design Constraints
- Solo Development: All gameplay systems were designed and implemented independently, with external references used only for unfamiliar engine-specific concepts.
- Art Scope Separation: The project prioritizes gameplay systems and logic over original asset creation. Most visual assets (e.g., foliage and rocks) were sourced externally and modified as needed.

---

## Weapon System
**Video Demo:** (Needs video)

### Design Rationale
- Weapon behavior differences are handled through a single configurable parent weapon class rather than separate blueprints per weapon type. An enumerated firing mode (e.g., Semi-Auto, Full-Auto, Bolt-Action) defines the active behavior path, with logic branching resolved through a switch on the enumerator. This approach avoids blueprint proliferation and keeps core weapon logic centralized, while behavior-specific differences are isolated behind clearly defined state paths. New weapons are introduced as lightweight child classes of the base weapon blueprint, with behavior defined primarily through configuration rather than custom logic. Per-weapon setup therefore focuses on tuning variables such as "RPS" and "Damage" values while preserving a single, shared execution path for weapon logic.

### Done
- Parent weapon class (stores the main logic), child classes inherit the logic and use their own variables such as "damage," "magazineCapacity" and "RPS." Each weapon can behave differently just by changing variables.
- Completed separate paths to handle different behavior between full-auto, semi-auto, and bolt-action weapons.
- Spawning projectiles when the player *has* ammo
- Projectiles travel where the player is looking (if we disregard bullet-drop), regardless of using first-person or third-person camera.
- Replicated projectiles. (Fixed issue where client would watch the host fire two bullets in two different directions).
- Animation montages (weapon recoil) when firing and not out of ammo. Different montage for different positions. Ex: Prone firing vs. Stand firing
- AI dealt damage based on the struck bone in the skeletal mesh. Ex: A hit to a limb does half damage. A hit to the torso does normal damage. A hit to the head deals double damage.
- Prevent the player from discarding a full magazine. When discarding a non-empty magazine, the ammo is wasted.
- Updated animation montage section (weapon recoil animations): Uses fewer branches, is less clunky, and less complex. 

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
- Extra comments and cleanup would not hurt.
- Review server/client authority. Client should not have authority. Most things should be fine, but something might have slipped through the cracks.

### Tradeoffs & Alternatives
-

### Player Impact
-
  
---

- ## Basic AI
**Video Demo:** (Needs video)

### Design Rationale
- AI behavior prioritizes self-preservation as a core decision driver. Enemies continuously evaluate threat and positioning to decide when to seek cover, advance, or apply pressure, rather than remaining exposed until shot. Under active threat, the AI prioritizes defensive positioning to maintain survivability and, by extension, continued relevance in the encounter. When a player shifts their attention elsewhere, such as by responding to another oncoming threat, that window is treated as an opportunity to reposition, advance, and apply pressure safely. Thus, the AI avoids reckless, "stupid" aggression and instead favors behavior that mirrors believable combat decision-making, as opposed to static stand-and-shoot (without cover or concealment) exchanges.

### Done
- Patrol Behavior:
  - To always be relevant, patrols are anchored around the *nearest* player (Anchors can change).
  - Go to a randomly selected point within the bounds of the area anchored to the nearest player. Repeat until entering combat.
  - Projectile strikes to the limbs deal half damage. Projectile strikes to the torso deal normal damage. Projectile strikes to the head deal double damage.
- Becoming Alert:
  - Has different "alert" ranges depending on what the player in the vision cone is doing (Struggles to find prone players, is moderate at finding crouching players, easily sees standing or running players).
  - Can become alert by detecting a bullet from the player. The player does not need to get a hit or a kill to alert the enemy. The mere presence of a bullet will put the AI into the combat branch.
- Combat Behavior:
  - ... 

### Planned
- Self-preservation behavior. Actively seek cover when alert && the player is looking in their direction.
- Offensive Behavior. When alert && the player is *not* looking in their direction, advance toward the player.
- Squad Behavior. Officers are the brain of the squad and direct movement. Foot soldiers will be tied to a limited area surrounding their officer. When the officer dies, the movement restrictions are gone, and the squad should *theoretically* dissolve without order.
- Anti-crowding. When seeking cover, prevent AI from crowding into a single spot. If no nearby cover is available, crouch in place.
- Morale. If the AI squad takes too many casualties too quickly or loses their officer, squads will either be fearful and rout (and later despawn), or be invigorated and bayonet-charge.
- Behavioral differences based on enemy faction. One faction might have AI that is hyper-aggressive, whereas another faction might rely on stalking and ambushes.
- LOTS of VA. Keep the AI shouting orders.
- Formations??? Ex: March in column (Patrol), 'L' formation (combat)
- Out of ammo? Seek cover! -> Reload -> Continue (AI will not stand out in the open while it does not have ammunition with which to defend itself). 

### Known Bugs
- 

### Needs Updating
- Combat Behavior has no substance. Currently, it is "Go to 0.00, 0.00, 0.00."
- Experiment with different patrol ranges. Larger area = less-likely to encounter player.

### Tradeoffs & Alternatives
- If all the AI is actively seeking cover and running a lot of checks, the AI might be more expensive to run in bulk. In exchange for less "dumb" AI, there will *probably* have to be less AI for the sake of performance.
- Alternatively, the AI could be "dumber" and *not* seek cover or otherwise behave more recklessly, but that runs the risk of creating "another generic AI" that simply runs around and shoots without considering its own safety. If the AI does not try to preserve its own life, it feels less "real." 

### Player Impact
- Due to anchored patrol behavior, the AI always "knows" the location of the players and will patrol in that area. As such, there should never be a squad of AI patrolling several kilometers away from the nearest player. So, AI squads should always be relevant due to their proximity, and are more likely to encounter players during their patrols.
- AI that actively tries to keep itself alive, such as by using cover, as opposed to standing out in the open, is harder to perceive as being "dumb." Killing a self-preserving AI should feel like a satisfying minor-accomplishment for the player since the AI, despite having low-health (one or two shots to kill), is more challenging to hit while it is concealed behind cover.
- The AI knows that it has a ranged projectile weapon. To keep pressure, the AI will still advance on the players, but they will not be afraid to use their rifles, as opposed to charging from tens of meters away with their bayonets. 

---

## World Generation
**Video Demo:** (Needs video)

### Design Rationale
- World layout is driven by dynamic generation to prioritize replayability and prevent encounters from becoming predictable or mundane over time. Unlike static, purpose-built levels, dynamically generated environments vary terrain, structures, and sightlines between sessions, keeping positioning and threat assessment contextual rather than memorized. By keeping level generation dynamic, there is also long-term scalability, allowing new variety to be introduced through additional assets or the adjustment of parameters, as opposed to full level redesigns. 

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
- Foliage penetrates through everything on top of itself. Ex: Blades of grass penetrating floors or rocks.

### Needs Updating
- Towns and military bases spawn in perfect squares. Add irregularity.
- Forests (specifically around mission objectives) are perfect circles. Add irregularity.

### Tradeoffs & Alternatives
- 

### Player Impact
- 

---

## AI Spawner
**Video Demo:** (Too early in development for a video)

### Design Rationale
- 

### Done
- Spawned by server on game start.
- Knows the number of connected players.
- Ticks every 1.0 second to determine the AI cap (based on player-count), the existing AI, and however many AI are needed to reach the cap.

### Planned
- Flexibility? AI are meant to operate and spawn in squads of 10. 1 officer and 9 "grunts." The current setup will spawn *individual* AI once the current AI-count is beneath the cap, not a full squad. Perhaps make the cap flexible so that, if the squad cap is reached, but some squads are seriously understrength, spawn another squad anyway.
- Isolation punishment. If there is more than 1 player in the level, find all instances of players in which the nearest player to them is further than 1Km. Spawn AI squads at three points around that isolated player and converge on their last known location. Make sure this does not happen every tick!!!
- Prevent spawning <1Km to nearest player, prevent spawning >1.5Km to nearest player. (Players should never witness AI spawning. AI should not need to travel a gargantuan distance just to get within range of the nearest player).
- The more mission-objectives completed, the greater the squad spawn-cap? Naturally increase the spawn-cap with time?

### Known Bugs
-

### Needs Updating
-

### Tradeoffs & Alternatives
- 

### Player Impact
-

---
  
Email: alexforsell354@gmail.com
LinkedIn: https://www.linkedin.com/in/alex-forsell-a810ba364/
