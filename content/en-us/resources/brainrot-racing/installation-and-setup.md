---
title: Installation and Setup
comments:
description: Learn how to install and configure the Brainrot Racing game kit.
prev: /resources/brainrot-racing/index
next: /resources/brainrot-racing/racing-mechanics
---

This guide walks through installing the Brainrot Racing game kit and understanding the project structure to help you get started quickly.

## Installation

To install Brainrot Racing:

1. Navigate to the [Brainrot Racing model page](https://www.roblox.com/library/) on the Roblox website.
2. Click the **Get** button to add the model to your inventory.
3. In Roblox Studio, open the **Toolbox** (View tab > Toolbox).
4. Select **Inventory** from the Toolbox menu.
5. Find **Brainrot Racing** in your models and insert it into your place.

The model contains all necessary assets, scripts, and configurations to run a complete racing game.

## Project Structure

The Brainrot Racing project is organized into several key locations within the Roblox hierarchy:

### Workspace

Contains the physical game environment:

- **Tracks** — Folder containing all race track models and components.
- **StartingGrid** — Spawn points where racers begin each race.
- **PowerUpSpawns** — Locations where power-ups appear during races.
- **TrackPieces** — Modular components for building custom tracks.

### ReplicatedStorage

Contains shared resources accessible by both client and server:

- **Characters** — Models and configurations for all playable characters.
- **PowerUps** — Power-up effects and visual assets.
- **Configurations** — Game settings including:
  - `RaceConfiguration` — Lap counts, time limits, scoring rules.
  - `CharacterStats` — Attributes for each character.
  - `EconomySettings` — Currency rewards and unlock costs.
- **Modules** — Shared utility scripts:
  - `RaceManager` — Core racing logic and state management.
  - `PhysicsHandler` — Vehicle physics calculations.
  - `CheckpointSystem` — Lap tracking and position detection.

### ServerScriptService

Server-side scripts that handle authoritative game logic:

- **RaceServer** — Main server controller that:
  - Manages race initialization and completion.
  - Validates player positions and lap counts.
  - Handles collision detection and power-up activation.
  - Distributes rewards after races.
- **MatchmakingService** — Handles lobby system and player grouping.
- **DataService** — Manages player data persistence:
  - Owned characters and unlocks.
  - Best lap times and statistics.
  - Currency balances.

### StarterPlayer

Contains player initialization scripts:

- **StarterCharacterScripts** — Scripts that run when a player character spawns:
  - `CharacterController` — Handles input and movement.
  - `BoostController` — Manages boost meter and activation.
  - `EffectsHandler` — Client-side visual and sound effects.
- **StarterPlayerScripts** — Scripts for the player interface:
  - `RaceUI` — Updates position, lap count, and timer displays.
  - `CharacterSelector` — Character selection menu in lobby.
  - `ShopUI` — Interface for purchasing new characters.

### StarterGui

User interface elements:

- **RaceHUD** — In-race display showing:
  - Current position and lap count.
  - Boost meter and active power-ups.
  - Mini-map and checkpoint indicators.
  - Race timer and countdown.
- **LobbyUI** — Pre-race interface for:
  - Character selection.
  - Track voting.
  - Ready system and player list.
- **ShopUI** — Economy interface for unlocking content.

## Configuration

Before running your first race, configure the game settings to match your desired experience.

### Race Settings

Open `ReplicatedStorage/Configurations/RaceConfiguration` and adjust:

```lua
-- Race Configuration
local RaceConfig = {
    -- Race parameters
    LapCount = 3,                    -- Number of laps per race
    MaxPlayers = 8,                  -- Maximum racers per match
    MinPlayers = 2,                  -- Minimum to start race
    CountdownTime = 5,               -- Seconds before race starts
    RaceTimeLimit = 300,             -- Maximum race duration in seconds

    -- Checkpoint settings
    CheckpointRadius = 20,           -- Detection range for checkpoints
    WrongWayThreshold = 2,           -- Checkpoints missed before warning

    -- Power-up settings
    PowerUpSpawnRate = 10,           -- Seconds between power-up spawns
    MaxActivePowerUps = 6,           -- Maximum power-ups on track
}

return RaceConfig
```

### Character Balance

Edit `ReplicatedStorage/Configurations/CharacterStats` to balance character attributes:

```lua
-- Character Stats Configuration
local CharacterStats = {
    ["BasicBrainrot"] = {
        Speed = 60,              -- Top speed
        Acceleration = 8,        -- Acceleration rate
        Handling = 7,            -- Turning responsiveness
        Weight = 5,              -- Collision mass
        BoostMultiplier = 1.5,   -- Speed multiplier when boosting
        UnlockCost = 0,          -- Starting character (free)
    },

    ["SpeedDemon"] = {
        Speed = 75,
        Acceleration = 6,
        Handling = 5,
        Weight = 4,
        BoostMultiplier = 1.8,
        UnlockCost = 1000,       -- Costs 1000 brainrots to unlock
    },

    ["DriftKing"] = {
        Speed = 65,
        Acceleration = 7,
        Handling = 9,
        Weight = 5,
        BoostMultiplier = 1.6,
        UnlockCost = 1500,
    },

    -- Add more characters here
}

return CharacterStats
```

### Economy Tuning

Configure rewards in `ReplicatedStorage/Configurations/EconomySettings`:

```lua
-- Economy Configuration
local EconomyConfig = {
    -- Position-based rewards (per race)
    PositionRewards = {
        [1] = 500,   -- First place
        [2] = 350,   -- Second place
        [3] = 250,   -- Third place
        [4] = 150,   -- Fourth place
        [5] = 100,   -- Fifth place
        [6] = 75,    -- Sixth place
        [7] = 50,    -- Seventh place
        [8] = 25,    -- Eighth place
    },

    -- Bonus rewards
    BonusRewards = {
        PerfectLap = 100,        -- Completing a lap without collision
        FirstBlood = 50,         -- First player to use power-up
        Comeback = 150,          -- Win from 5th place or lower
        NewRecord = 200,         -- Set new personal best lap time
    },

    -- Daily objectives
    DailyRewardMultiplier = 2.0,  -- Multiplier for first daily race
}

return EconomyConfig
```

## Testing Your Setup

After configuration, test the game:

1. Click **Test** > **Start** in Roblox Studio to launch a local test server.
2. Use multiple clients (File > Test > Players = 2 or more) to test multiplayer functionality.
3. Verify that:
   - Characters spawn correctly at the starting grid.
   - Controls respond properly (WASD or arrow keys for movement, Shift for boost).
   - Checkpoints detect player passage accurately.
   - Lap counting increments correctly.
   - The race ends when a player completes all laps.

## Common Issues

### Characters Don't Spawn

**Solution**: Ensure `StartingGrid` contains properly positioned `SpawnLocation` parts with the `Neutral` property set to `true`.

### Checkpoints Not Detecting

**Solution**: Verify that checkpoint parts have `CanCollide` set to `false` and contain a `Script` or `ModuleScript` linked to the checkpoint system.

### Controls Unresponsive

**Solution**: Check that `StarterPlayer/StarterCharacterScripts/CharacterController` is present and that the character model has a `Humanoid` configured for vehicle mode.

### Power-Ups Not Spawning

**Solution**: Confirm `PowerUpSpawns` folder contains properly configured spawn locations and that `RaceConfiguration.PowerUpSpawnRate` is set to a positive value.

## Next Steps

Now that your game is installed and configured, proceed to [Racing Mechanics](./racing-mechanics.md) to understand how the core racing systems work and how to customize them for your game.
