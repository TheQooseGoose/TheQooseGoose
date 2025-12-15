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

### Planned
- Ammo drops/pickups
- Bayonets/melee
- More weapons and weapon models

### Known Bugs
- Projectiles *have* collision and fully interact with characters and blueprints, but projectiles pass straight through HISMs (trees, rocks, etc) and the landscape.

### Needs Updating
- …
  
---

- ## Basic AI
**Video Demo:** (unlisted YouTube link)

### Design Rationale

### Done
- …

### Planned
- …

### Known Bugs
- …

### Needs Updating
- …

---

## World Generation
**Video Demo:** (unlisted YouTube link)

### Design Rationale

### Done
- …

### Planned
- …

### Known Bugs
- …

### Needs Updating
- …

---
  
Email: alexforsell354@gmail.com
LinkedIn: https://www.linkedin.com/in/alex-forsell-a810ba364/
