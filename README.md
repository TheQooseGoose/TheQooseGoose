# Alexander Forsell

> Status: Living portfolio - Systems are *actively* under development.

Unreal Engine 5 gameplay programmer focused on building core gameplay systems.

This repository serves as a living portfolio and development log, showcasing work-in-progress systems such as weapons, AI, and dynamic world generation. Each section documents what is implemented, what is planned, known issues, and areas marked for refactoring or improvement.

All projects shown here are actively developed and intentionally presented as snapshots of the iteration process rather than finished products.

---

### Project Context

This project serves as a systems-focused demonstration of my ability to design and implement gameplay mechanics within a game engine. It exists to support and validate my independent study by providing concrete evidence of hands-on game development experience through working systems, documented design decisions, and problem-solving.
The primary focus is on gameplay logic and system architecture, such as weapons, AI behavior, and dynamic world generation. Unreal Engine 5 is used as the chosen implementation environment, but the emphasis is placed on transferable design principles, such as state-driven logic or system extensibility, rather than engine-specific features. Visuals are intentionally deprioritized in favor of functionality. 

---

## Design Constraints
- Solo Development: All gameplay systems were designed and implemented independently, with external references used only for unfamiliar engine-specific concepts.
- Art Scope Separation: The project prioritizes gameplay systems and logic over original asset creation. Most *visual* assets (e.g., foliage meshes) were sourced externally and modified as needed.

---

## Weapon System
**Video Demo:** https://youtu.be/E1dLsZp6jjw 

### Design Rationale
- Weapon behavior differences are handled through a single configurable parent weapon class rather than separate blueprints per weapon type. An enumerated firing mode (e.g., Semi-Auto, Full-Auto, Bolt-Action) defines the active behavior path, with logic branching resolved through a switch on the enumerator. This approach avoids blueprint proliferation and keeps core weapon logic centralized, while behavior-specific differences are isolated behind clearly defined state paths. New weapons are introduced as lightweight child classes of the base weapon blueprint, with behavior defined primarily through configuration rather than custom logic. Per-weapon setup, therefore, focuses on tuning variables such as "RPS" and "Damage" values while preserving a single, shared execution path for weapon logic.

### Done
- Parent weapon class (stores the main logic), child classes inherit the logic and use their own variables such as "damage," "magazineCapacity," and "RPS." Each weapon can behave differently just by changing variables.
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
- Review server/client authority. The client should not have authority. Most things should be fine, but something might have slipped through the cracks.

### Tradeoffs & Alternatives
- A weapon that behaves differently from the expected bolt-action/semi/full enumerators would need its own logic path added, such as burst-fire weapons. In exchange, however, weapons are straightforward to design, since a child of the base class just needs a few parameters to be modified, such as the rate of fire or magazine capacity.

### Player Impact
- 

### Screenshots
<details>
<summary><strong>Logic Reference (Blueprint)</strong></summary>

<br>

- This collection of screenshots does not include related logic for aiming/camera-zoom, the animation graph, or where player booleans that prevent weapon firing, such as "isInTransition?", are defined.
- Note: Some areas reflect ongoing iteration. Minor refactors (such as further function consolidation) have been identified but deferred in favor of validating system behavior and design direction.

- Player input is validated locally and forwarded to authoritative server events, with transition-state checks preventing invalid fire requests. (Ex: Prevent player from firing while moving TO or FROM prone).
- <img width="602" height="332" alt="IA_Shoot" src="https://github.com/user-attachments/assets/fa487fb9-36de-4562-a2e2-a5d687672e2e" />
- Weapon firing behavior is resolved server-side using a unified execution path, with fore mode determining whether shots are timer-driven (full auto) or single-execution (semi-auto/bolt-action).
- <img width="2048" height="623" alt="16c3d95d-c5c5-4c01-856b-fe5564e891cf" src="https://github.com/user-attachments/assets/9493722d-c783-4c32-89cb-426d3fed1f55" />
- Weapon firing is gated by server authority, validated ownership, and weapon state before any behavior-specific logic is executed. A centralized validation gate prevents invalid firing, after which, weapon behavior branches cleanly to the enumerated fire mode.
- <img width="2048" height="617" alt="b4cb53bd-f323-45ff-baa2-9a91bbd75cfc" src="https://github.com/user-attachments/assets/e3bcc387-f459-42a3-bc0c-6641ecba7455" />
- <img width="898" height="361" alt="b28746b6-98b0-4672-baff-144acf930cf6" src="https://github.com/user-attachments/assets/35fbe74f-3aa6-4996-8402-65db975f0e5c" />
- Semi-automatic fire executes a single validated shot by decrementing ammunition and spawning a projectile through the shared pipeline.
  - Note: Semi-auto and full-auto share the same core shot execution logic. I have not yet unified the two.
- <img width="764" height="190" alt="fe5926c7-fb80-4125-ba94-fcea60c0fd47" src="https://github.com/user-attachments/assets/779196ff-d9aa-4937-b2b3-0f1c43578abe" />
- Bolt-action fire enforces a manual cycling state, preventing additional shots until the weapon has been explicitly cycled.
- <img width="763" height="307" alt="be99a75e-8021-44b1-b5b2-38c4d587363e" src="https://github.com/user-attachments/assets/0596748d-0977-4335-8bc1-d3bc697178f5" />
- Projectile spawning logic determines the active camera context to ensure aim calculations reflect the player's actual viewpoint.
- <img width="1839" height="511" alt="image" src="https://github.com/user-attachments/assets/4cba70e0-1592-47b4-b483-f67064afdb64" />
- Third-person aiming uses a camera-based line trace to determine the intended target, ensuring projectiles align with the player's crosshair rather than weapon orientation.
  - Note: This logic is old and messy. Could use some cleanup.
  - If the line trace has no blocking hit, the "False" part of the branch has the projectile move toward the end of the line trace. This prevents a bug where, when the player fires into the air, the projectile travels in the wrong direction.
- <img width="2048" height="568" alt="image" src="https://github.com/user-attachments/assets/7fcce5f1-deb2-4635-96fa-45e0b96d663f" />
- Projectile lifetime is explicitly bounded to prevent lingering actors and unnecessary world clutter.
- <img width="601" height="305" alt="6947da1a-2228-4b74-ad99-95743cf42f26" src="https://github.com/user-attachments/assets/8c890749-5ec7-4c00-b323-3198d8540ad9" />
- Recoil animations are selected dynamically based on player posture and aiming state, ensuring correct visual feedback across movement states. 
- <img width="998" height="373" alt="d718643f-12af-499c-a3c0-ccee15ece332" src="https://github.com/user-attachments/assets/b856d37e-fc04-4017-a52b-0666af767d1f" />


</details>

  
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
- Due to anchored patrol behavior, the AI always "knows" the location of the players and will patrol in that area. As such, there should never be a squad of AI patrolling several kilometers away from the nearest player. So, AI squads should always be relevant due to their proximity and are more likely to encounter players during their patrols.
- AI that actively tries to keep itself alive, such as by using cover, as opposed to standing out in the open, is harder to perceive as being "dumb." Killing a self-preserving AI should feel like a satisfying minor accomplishment for the player since the AI, despite having low health (one or two shots to kill), is more challenging to hit while it is concealed behind cover.
- The AI knows that it has a ranged projectile weapon. To keep pressure, the AI will still advance on the players, but they will not be afraid to use their rifles, as opposed to charging from tens of meters away with their bayonets.

### Screenshots
<details>
<summary><strong>Logic Reference (Blueprint)</strong></summary>

<br>

- This collection of screenshots does not include related logic for...
- Note: 

</details>

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
- Universal Landscape Material. Spawns grass and flower meshes atop itself; no need to manually paint. The steeper the slope, the yellower (drier) the terrain.
- Completed logic for a simple "Capture and Hold" objective.
- Complex foliage materials. Using Runtime Virtual Textures, foliage like grass, bushes, and flowers inherit their colors from the landscape beneath it. Sloped, yellow hills will have yellow foliage.

### Planned
- Dynamic landscape generator. Rather than importing different heightmap scans of random fields in rural Canada, dynamically generate the landscape using the seed. Massive project, but it would make each level more dynamic.
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
- Alternatively, I could have purpose-built levels, which *are* generally better built than random generation. But that also means that, after enough playthroughs, the content gets old very fast, because nothing new ever happens, and gameplay stops feeling refreshing.

### Player Impact
- 

### Screenshots
<details>
<summary><strong>Logic Reference (Blueprint)</strong></summary>

<br>

- This collection of screenshots does not include related logic for defining the number of connected players and their respective controllers. Nor does it cover how the host and client handle world generation differently. Only the independent generators are shown.  

- The host joins the game first and initiates world generation. The first step, while the level is devoid of obstacles, is to spawn the key gameplay features, the mission objectives.
- <img width="454" height="368" alt="image" src="https://github.com/user-attachments/assets/d7fde572-d245-4f48-922b-1e2795571d0c" />
- The randomly generated seed is used to create an "RNGMaster," which is used to replicate the results of a "Random" node, since, when run independently on a device, a host, and a client will get different results. That number is then used to determine the random number (between 4 and 8) of objectives that will spawn in the level. Then, a loop will run for that 'random number' amount of times until all objectives have spawned in the level.
- <img width="947" height="407" alt="image" src="https://github.com/user-attachments/assets/375d2c47-9d8f-4810-83f1-71e90909431f" />
- Using the coordinates of two opposite corners of the level boundaries, using two "Random" nodes, generate a random coordinate.
  - Note: The coordinates are currently hard-defined since I only have one level. If I ever get more levels or new boundaries to experiment with, I am aware that I will need to be more creative.
- <img width="716" height="303" alt="image" src="https://github.com/user-attachments/assets/45a39f55-b9a1-48c9-8ef3-76f5c64cadcc" />
- Using the random coordinate, a line trace is conducted to determine the elevation of the terrain. This way, a mission objective is spawned ON the landscape, and not floating in the air, or worse yet, totally inaccessible underground. To settle the spawned mesh into the terrain a little, it is moved down by 15.0cm. 
- <img width="749" height="335" alt="image" src="https://github.com/user-attachments/assets/38db7ebc-4c1b-4891-9b46-c298504e2fdc" />
- Before the "SpawnActor" node is executed, a random objective is chosen from an array.
  - Note: At the time of writing, I only have a simple but functional "Capture and Hold" objective in the array, since it is the only semi-completed mission objective blueprint that I have.
- <img width="671" height="175" alt="image" src="https://github.com/user-attachments/assets/0a195c27-80b1-4b25-a2a2-979cec7f514d" />
- Now the level has between four and eight mission objectives for the players to complete. But, because the level is currently a barren landscape, we want to make the surroundings a little more interesting. So, using a "Switch on Int" and a random number, each objective has a chance to spawn one of three things around itself: If 0, a Town. If 1, a forest. If 2, a military base.
  - Note: The houses are all blueprints because they are planned to have features like "Health" so that they can collapse after taking too much damage. Since blueprints can be easily replicated, the houses can be spawned here, whereas the HISMs, as used by the forests and military base props (such as sandbags or pillboxes) cannot, and have their own blueprint for spawning. So, instead, the location of the mission objective is stored as a "Forest Objective Location" or a "Base Objective Location" to be used later. 
- <img width="908" height="407" alt="image" src="https://github.com/user-attachments/assets/4c2eb653-0edb-4b10-9028-bfaebee3db55" />
- If a town is to be generated around an objective, the first thing to do is define how big that town is going to be. Then, with a rough estimate of the area around a house (+added room for spacing), we can figure out roughly how many houses can be supported in that town. 
- <img width="911" height="304" alt="image" src="https://github.com/user-attachments/assets/3fafc48b-3625-4518-b3fc-ee09de2f46c4" />
- Now that we know how many houses the town can support, we need to use the selected area and figure out the boundaries of the town, based on the town center (the spawned mission objective). 
- <img width="659" height="284" alt="image" src="https://github.com/user-attachments/assets/d95a581d-91b1-4de2-ad62-0e38c0afe8a0" />
- Now we can start spawning houses. So, using the defined boundaries of the town, select a random location, and repeat for however many houses *can* (not necessarily "will") be supported in that area. 
- <img width="541" height="306" alt="image" src="https://github.com/user-attachments/assets/d6aee907-ea31-4275-8fa3-3f14b5ab0969" />
- Conduct a line trace to determine the elevation, and subtract 125.0 units from that point to settle the mesh into the ground a little. If the line trace does NOT hit the Landscape, then do NOT spawn the house. Houses should not spawn atop other houses or foliage. 
- <img width="774" height="397" alt="image" src="https://github.com/user-attachments/assets/cec1c344-eff0-4617-82c4-359eb9bfc7aa" />
- Once a house is successfully spawned, the location is stored in an array. This loop iterates through those locations and determines if the new coordinate to test is too close to a house that was just spawned. (Prevent houses from spawning too closely). 
- <img width="1001" height="232" alt="image" src="https://github.com/user-attachments/assets/8ae1ce57-3c0a-4689-94bd-eb9fb22a2c92" />
- Choose a random type of house, give it a random rotation, and spawn it in the selected and approved location.
- <img width="818" height="344" alt="image" src="https://github.com/user-attachments/assets/e43075b7-a919-460d-8f09-2f996b69e8a8" />
- Now that the house has been successfully spawned, we need to store its location in the array so that its location can be tested later as new houses spawn.
- <img width="293" height="227" alt="image" src="https://github.com/user-attachments/assets/60fe8e42-3c88-4bfb-a60b-2f40a6fd8370" />





</details>

---

## AI Spawner
**Video Demo:** (Too early in development for a video)

### Design Rationale
- In a multiplayer environment, players frequently diverge in spacing and intent. Some players cluster into a primary engagement zone, while others operate more independently as "lone wolves." A naive spawner that targets only the largest cluster of players risks leaving isolated players unchallenged, enabling objective completion with minimal resistance. To counter, the spawner will collect the current information of all AI squad officers and all players. Using the collected data, the spawner can determine which player is the *most* isolated from enemy AI and will use that player as the spawn-anchor. The next batch of AI will spawn (while abiding by the spawn rules) near that isolated player.

### Done
- Spawned by the server on game start.
- Knows the number of connected players.
- Ticks every 1.0 second to determine the AI cap (based on player-count), the existing AI, and however many AI are needed to reach the cap.
- Isolation analysis. When finding a spawn location for the AI, figure out which player is the most isolated (furthest from the nearest AI squad). Use that isolated player as the anchor to spawn the next batch of AI.
- 

### Planned
- Prevent spawning <1Km to nearest player, prevent spawning >1.5Km to nearest player. (Players should never witness AI spawning. AI should not need to travel a gargantuan distance just to get within range of the nearest player).
- The more mission objectives completed, the greater the squad spawn-cap? Naturally increase the spawn-cap with time?

### Known Bugs
- 

### Needs Updating
- Currently only tracks AI OFFICERS, not foot soldiers. When an Officer is killed, a new squad is spawned. If players prioritize killing Officers *only,* the game will continuously spawn batches of 9 foot soldiers for every 1 officer killed. Needs a counter for AI foot soldiers so that a new squad can only be spawned when the AI officer cap is below quota, and the AI soldier cap is 9 below quota. (A squad is 10 AI, 1 Officer, 9 soldiers). Only spawn those 10 AI when there is room to spawn those 10 AI.

### Tradeoffs & Alternatives
- 

### Player Impact
- 

### Screenshots
<details>
<summary><strong>Logic Reference (Blueprint)</strong></summary>

<br>

- This collection of screenshots does not include related logic for...
- Note: 

</details>

---
  
Email: alexforsell354@gmail.com
LinkedIn: https://www.linkedin.com/in/alex-forsell-a810ba364/
