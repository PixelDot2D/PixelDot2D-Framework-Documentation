# Core Overview

The **Core** module is the standalone foundation of the PixelDot2D Framework. It is designed to be reused across any 2D game project, regardless of genre.

Core does not contain game-specific features like character controllers, weapons, or combat. Instead, it provides the universal building blocks every game needs, including:

- **Input Handling:** Capturing and routing player inputs.
- **Generic 2D Physics Movement:** Driving Rigidbody2D velocity and forces without knowing what object is moving (whether it is a player, enemy, moving platform, or projectile).
- **Collision & Sensors:** Detecting ground, walls, and environmental triggers.
- **Animation:** Controlling sprite animations and transitions.
- **Saving & Persistence:** Storing and loading game state data.

## Standalone Architecture

Core lives in its own isolated assembly and has **zero dependencies** on any other part of the framework. Every other sub-library (such as Modular Character, Combat, Items, and Shared) depends on Core, never the reverse.

Because of this structure:
- You can delete every other folder in the framework and keep only **Core**—your project will still compile and run cleanly.
- **Core does not include standalone demo scenes.** Instead, the other sub-libraries act as live, production-ready demonstrations of how a single Core foundation can power everything from complex character movement to multi-layered combat systems.
