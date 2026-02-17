---
title: Character System
comments:
description: Learn how to create, balance, and customize playable characters in Brainrot Racing.
prev: /resources/brainrot-racing/racing-mechanics
next: /resources/brainrot-racing/reward-system
---

The character system in Brainrot Racing allows players to collect and race with unique characters, each with distinct attributes and visual designs. This guide explains how the system works and how to add new characters.

## Character Architecture

Each character consists of three main components:

- **Model** — The visual representation stored in `ReplicatedStorage/Characters`.
- **Stats** — Numerical attributes that affect racing performance.
- **Metadata** — Information about unlock requirements, rarity, and cosmetics.

### Character Model Structure

A character model must follow this hierarchy:

```
CharacterName (Model)
├── PrimaryPart (Part) - Main body with collision
├── BodyVelocity (BodyVelocity) - Movement control
├── BodyGyro (BodyGyro) - Rotation control
├── Humanoid (Humanoid) - Required for player control
├── BoostMeter (NumberValue) - Current boost charge
├── RaceData (Folder) - Race progress tracking
│   ├── CurrentLap (IntValue)
│   ├── CurrentCheckpoint (IntValue)
│   ├── Position (IntValue)
│   ├── LapStartTime (NumberValue)
│   └── BestLapTime (NumberValue)
└── Effects (Folder) - Visual effect attachments
    ├── BoostTrail (Trail)
    ├── DriftSmoke (ParticleEmitter)
    └── SpeedLines (ParticleEmitter)
```

### Required Components

Set up the essential components for proper functionality:

```lua
-- CharacterSetup ModuleScript
local CharacterSetup = {}

function CharacterSetup.PrepareCharacter(characterModel, stats)
    local primaryPart = characterModel.PrimaryPart

    -- Configure Humanoid for vehicle mode
    local humanoid = characterModel:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = 0  -- Disable walking
        humanoid.JumpPower = 0  -- Disable jumping
    end

    -- Create BodyVelocity for movement
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Name = "BodyVelocity"
    bodyVelocity.MaxForce = Vector3.new(50000, 50000, 50000)
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.Parent = primaryPart

    -- Create BodyGyro for rotation
    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.Name = "BodyGyro"
    bodyGyro.MaxTorque = Vector3.new(50000, 50000, 50000)
    bodyGyro.D = 500
    bodyGyro.P = 3000
    bodyGyro.Parent = primaryPart

    -- Initialize boost meter
    local boostMeter = Instance.new("NumberValue")
    boostMeter.Name = "BoostMeter"
    boostMeter.Value = 0
    boostMeter.Parent = characterModel

    -- Store stats as attributes
    for statName, value in pairs(stats) do
        characterModel:SetAttribute(statName, value)
    end

    return characterModel
end

return CharacterSetup
```

## Character Stats

Character attributes determine racing behavior and create diverse gameplay experiences.

### Core Attributes

Define these stats for each character:

```lua
-- Example character stats
local CharacterStats = {
    -- Speed: Maximum velocity (studs per second)
    -- Range: 50-80 for balanced gameplay
    Speed = 65,

    -- Acceleration: How quickly character reaches max speed
    -- Range: 5-10, higher values feel more responsive
    Acceleration = 7,

    -- Handling: Turn rate and drift control
    -- Range: 5-10, affects cornering ability
    Handling = 8,

    -- Weight: Collision impact factor
    -- Range: 3-7, heavier = less affected by collisions
    Weight = 5,

    -- BoostMultiplier: Speed multiplier during boost
    -- Range: 1.3-2.0, affects boost power
    BoostMultiplier = 1.6,

    -- UnlockCost: Brainrots required to unlock
    -- 0 for starter characters
    UnlockCost = 500,
}
```

### Stat Balancing

Balance characters by creating strengths and weaknesses:

```lua
-- Speed-focused character (high speed, low handling)
["SpeedDemon"] = {
    Speed = 78,
    Acceleration = 6,
    Handling = 5,
    Weight = 4,
    BoostMultiplier = 1.9,
    UnlockCost = 2000,
}

-- Handling-focused character (high control, moderate speed)
["DriftKing"] = {
    Speed = 62,
    Acceleration = 8,
    Handling = 10,
    Weight = 5,
    BoostMultiplier = 1.5,
    UnlockCost = 1500,
}

-- Balanced character (no extremes)
["AllRounder"] = {
    Speed = 68,
    Acceleration = 7,
    Handling = 7,
    Weight = 5,
    BoostMultiplier = 1.6,
    UnlockCost = 1000,
}

-- Tank character (heavy, durable)
["Juggernaut"] = {
    Speed = 58,
    Acceleration = 5,
    Handling = 6,
    Weight = 8,
    BoostMultiplier = 1.4,
    UnlockCost = 1800,
}
```

## Creating New Characters

Follow these steps to add a new character to your game:

### Step 1: Design the Model

Create the character model in Roblox Studio:

1. Build the character using parts, meshes, or imported models.
2. Set one part as the `PrimaryPart` (typically the main body).
3. Ensure all parts are anchored initially.
4. Add collision geometry appropriate for the character size.
5. Apply materials and textures for visual style.

### Step 2: Add Required Components

```lua
-- In a Script, add components to your character model
local character = workspace.NewCharacterModel

-- Add Humanoid
local humanoid = Instance.new("Humanoid")
humanoid.Parent = character

-- Configure parts
for _, part in ipairs(character:GetDescendants()) do
    if part:IsA("BasePart") then
        part.Anchored = false
        part.CanCollide = true
        part.CustomPhysicalProperties = PhysicalProperties.new(0.7, 0.3, 0.5)
    end
end

-- Create effects folder
local effectsFolder = Instance.new("Folder")
effectsFolder.Name = "Effects"
effectsFolder.Parent = character

-- Add boost trail
local boostTrail = Instance.new("Trail")
boostTrail.Name = "BoostTrail"
boostTrail.Lifetime = 0.5
boostTrail.Color = ColorSequence.new(Color3.fromRGB(255, 150, 0))
boostTrail.Enabled = false
boostTrail.Parent = effectsFolder
```

### Step 3: Configure Stats

Add character stats to the configuration:

```lua
-- In ReplicatedStorage/Configurations/CharacterStats
local CharacterStats = require(script)

CharacterStats["YourNewCharacter"] = {
    Speed = 70,
    Acceleration = 7,
    Handling = 8,
    Weight = 5,
    BoostMultiplier = 1.7,
    UnlockCost = 1200,

    -- Optional: Special abilities
    SpecialAbility = "DoubleBoost",  -- Custom ability identifier
    Rarity = "Epic",                  -- Common, Rare, Epic, Legendary
    Description = "A mysterious character with balanced stats and unique flair.",
}
```

### Step 4: Add Visual Effects

Customize the character's appearance during gameplay:

```lua
-- Effects configuration
function CharacterSetup.AddEffects(character, effectsConfig)
    local effects = character:FindFirstChild("Effects")
    if not effects then return end

    -- Drift smoke particles
    local driftSmoke = Instance.new("ParticleEmitter")
    driftSmoke.Name = "DriftSmoke"
    driftSmoke.Texture = "rbxasset://textures/particles/smoke_main.dds"
    driftSmoke.Rate = 40
    driftSmoke.Lifetime = NumberRange.new(0.4, 0.8)
    driftSmoke.Speed = NumberRange.new(5, 10)
    driftSmoke.Color = effectsConfig.SmokeColor or ColorSequence.new(Color3.fromRGB(150, 150, 150))
    driftSmoke.Enabled = false
    driftSmoke.Parent = character.PrimaryPart

    -- Speed lines for fast movement
    local speedLines = Instance.new("ParticleEmitter")
    speedLines.Name = "SpeedLines"
    speedLines.Texture = "rbxasset://textures/particles/sparkles_main.dds"
    speedLines.Rate = 20
    speedLines.Lifetime = NumberRange.new(0.2, 0.4)
    speedLines.Speed = NumberRange.new(20, 30)
    speedLines.Color = effectsConfig.SpeedColor or ColorSequence.new(Color3.fromRGB(255, 255, 255))
    speedLines.Enabled = false
    speedLines.Parent = character.PrimaryPart
end
```

## Character Selection

Players select characters before races through the lobby UI:

```lua
-- CharacterSelector ModuleScript
local CharacterSelector = {}
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

function CharacterSelector.LoadOwnedCharacters(player)
    -- Retrieve owned characters from player data
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    local ownedCharacters = {}
    for characterName, stats in pairs(ReplicatedStorage.Configurations.CharacterStats) do
        if playerData.UnlockedCharacters[characterName] then
            table.insert(ownedCharacters, {
                Name = characterName,
                Stats = stats,
                IsOwned = true,
            })
        else
            table.insert(ownedCharacters, {
                Name = characterName,
                Stats = stats,
                IsOwned = false,
                Cost = stats.UnlockCost,
            })
        end
    end

    return ownedCharacters
end

function CharacterSelector.SelectCharacter(player, characterName)
    local characters = ReplicatedStorage.Characters
    local characterModel = characters:FindFirstChild(characterName)

    if not characterModel then
        warn("Character not found:", characterName)
        return nil
    end

    -- Store selection
    player:SetAttribute("SelectedCharacter", characterName)

    return characterModel:Clone()
end

return CharacterSelector
```

## Special Abilities

Add unique abilities to differentiate characters:

### Ability System

```lua
-- AbilitySystem ModuleScript
local AbilitySystem = {}

-- Define abilities
local Abilities = {
    DoubleBoost = function(character)
        -- Consumes boost slower, lasts longer
        local normalConsumption = 20
        return normalConsumption * 0.5
    end,

    SpeedBurst = function(character)
        -- Instant speed boost without depleting meter
        local bodyVelocity = character.PrimaryPart.BodyVelocity
        local currentVelocity = bodyVelocity.Velocity
        bodyVelocity.Velocity = currentVelocity * 1.3
        wait(1)
        bodyVelocity.Velocity = currentVelocity
    end,

    ShieldBash = function(character)
        -- Temporary immunity to collisions
        character:SetAttribute("ShieldActive", true)
        wait(3)
        character:SetAttribute("ShieldActive", false)
    end,
}

function AbilitySystem.ActivateAbility(character, abilityName)
    local ability = Abilities[abilityName]
    if ability then
        ability(character)
    end
end

return AbilitySystem
```

### Cooldown Management

Prevent ability spam:

```lua
function AbilitySystem.CanUseAbility(character, abilityName)
    local lastUse = character:GetAttribute("LastAbilityUse_" .. abilityName) or 0
    local cooldown = 30  -- Seconds

    return (tick() - lastUse) >= cooldown
end

function AbilitySystem.UseAbility(character, abilityName)
    if not AbilitySystem.CanUseAbility(character, abilityName) then
        return false
    end

    AbilitySystem.ActivateAbility(character, abilityName)
    character:SetAttribute("LastAbilityUse_" .. abilityName, tick())
    return true
end
```

## Character Unlocking

Players unlock characters by spending earned currency:

```lua
-- UnlockSystem on Server
local UnlockSystem = {}

function UnlockSystem.UnlockCharacter(player, characterName)
    local dataService = require(script.Parent.DataService)
    local playerData = dataService.GetPlayerData(player)

    local stats = ReplicatedStorage.Configurations.CharacterStats[characterName]
    if not stats then
        return false, "Character not found"
    end

    -- Check if already owned
    if playerData.UnlockedCharacters[characterName] then
        return false, "Already owned"
    end

    -- Check currency
    if playerData.Currency < stats.UnlockCost then
        return false, "Insufficient brainrots"
    end

    -- Purchase character
    playerData.Currency = playerData.Currency - stats.UnlockCost
    playerData.UnlockedCharacters[characterName] = true

    -- Save data
    dataService.SavePlayerData(player, playerData)

    return true, "Character unlocked!"
end

return UnlockSystem
```

## Testing Characters

Balance test your characters:

1. **Speed Tests** — Time laps on standard tracks to compare performance.
2. **Handling Tests** — Navigate tight corners to evaluate control.
3. **Collision Tests** — Test weight differences in multi-player scenarios.
4. **Progression Tests** — Ensure unlock costs match power levels.

### Debug Tools

Create a testing interface:

```lua
-- Debug script to spawn and test characters
local function TestCharacter(characterName)
    local character = ReplicatedStorage.Characters[characterName]:Clone()
    local stats = ReplicatedStorage.Configurations.CharacterStats[characterName]

    CharacterSetup.PrepareCharacter(character, stats)
    character:PivotTo(workspace.TestSpawn.CFrame)
    character.Parent = workspace

    print("Testing", characterName)
    print("Speed:", stats.Speed)
    print("Handling:", stats.Handling)
    print("Weight:", stats.Weight)
end
```

## Best Practices

When designing characters:

- Create clear archetypes (speed, handling, tank, balanced).
- Ensure no character is strictly superior to others.
- Visual design should hint at character attributes (bulky = heavy, sleek = fast).
- Test with real players to identify balance issues.
- Start with fewer well-balanced characters rather than many similar ones.

## Next Steps

Learn how the [Reward System](./reward-system.md) works to understand how players earn currency to unlock your characters.
