---
title: Racing Mechanics
comments:
description: Deep dive into the physics and control systems that power Brainrot Racing gameplay.
prev: /resources/brainrot-racing/installation-and-setup
next: /resources/brainrot-racing/character-system
---

The racing mechanics in Brainrot Racing provide responsive controls and dynamic physics that create an engaging competitive experience. This guide explains the core systems and demonstrates how to customize them.

## Movement System

The character movement system uses a combination of [`BodyVelocity`](https://create.roblox.com/docs/reference/engine/classes/BodyVelocity) and [`BodyGyro`](https://create.roblox.com/docs/reference/engine/classes/BodyGyro) instances for precise control over character movement and rotation.

### Input Handling

The `CharacterController` script in `StarterCharacterScripts` processes player input:

```lua
-- CharacterController.lua
local UserInputService = game:GetService("UserInputService")
local character = script.Parent
local humanoid = character:WaitForChild("Humanoid")

-- Input states
local inputs = {
    forward = false,
    backward = false,
    left = false,
    right = false,
    boost = false,
}

-- Process keyboard input
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
        inputs.forward = true
    elseif input.KeyCode == Enum.KeyCode.S or input.KeyCode == Enum.KeyCode.Down then
        inputs.backward = true
    elseif input.KeyCode == Enum.KeyCode.A or input.KeyCode == Enum.KeyCode.Left then
        inputs.left = true
    elseif input.KeyCode == Enum.KeyCode.D or input.KeyCode == Enum.KeyCode.Right then
        inputs.right = true
    elseif input.KeyCode == Enum.KeyCode.LeftShift then
        inputs.boost = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
        inputs.forward = false
    -- Continue for other keys...
    end
end)
```

### Physics Calculation

The physics engine calculates velocity and rotation based on character stats and input:

```lua
-- PhysicsHandler ModuleScript
local PhysicsHandler = {}

function PhysicsHandler.UpdateMovement(character, inputs, stats, deltaTime)
    local bodyVelocity = character.PrimaryPart:FindFirstChild("BodyVelocity")
    local bodyGyro = character.PrimaryPart:FindFirstChild("BodyGyro")

    if not bodyVelocity or not bodyGyro then return end

    -- Calculate forward/backward movement
    local acceleration = stats.Acceleration
    local maxSpeed = stats.Speed

    if inputs.boost and character.BoostMeter.Value > 0 then
        maxSpeed = maxSpeed * stats.BoostMultiplier
        character.BoostMeter.Value = math.max(0, character.BoostMeter.Value - deltaTime * 20)
    end

    local targetVelocity = Vector3.zero
    if inputs.forward then
        targetVelocity = character.PrimaryPart.CFrame.LookVector * maxSpeed
    elseif inputs.backward then
        targetVelocity = -character.PrimaryPart.CFrame.LookVector * (maxSpeed * 0.5)
    end

    -- Smooth velocity transition
    local currentVelocity = bodyVelocity.Velocity
    bodyVelocity.Velocity = currentVelocity:Lerp(targetVelocity, acceleration * deltaTime)

    -- Calculate rotation
    local turnSpeed = stats.Handling * 2
    if inputs.left then
        bodyGyro.CFrame = bodyGyro.CFrame * CFrame.Angles(0, math.rad(turnSpeed * deltaTime), 0)
    elseif inputs.right then
        bodyGyro.CFrame = bodyGyro.CFrame * CFrame.Angles(0, -math.rad(turnSpeed * deltaTime), 0)
    end

    return bodyVelocity.Velocity
end

return PhysicsHandler
```

## Drift System

Drifting allows players to maintain speed through corners while building boost meter.

### Drift Detection

The system detects when a player is drifting based on steering input and velocity angle:

```lua
-- DriftSystem ModuleScript
local DriftSystem = {}

function DriftSystem.CheckDrift(character, inputs, velocity)
    local isDrifting = false
    local driftAngle = 0

    -- Calculate angle between velocity and facing direction
    if velocity.Magnitude > 10 then
        local velocityDirection = velocity.Unit
        local facingDirection = character.PrimaryPart.CFrame.LookVector

        driftAngle = math.deg(math.acos(velocityDirection:Dot(facingDirection)))

        -- Drifting occurs when turning while moving fast
        if (inputs.left or inputs.right) and driftAngle > 15 then
            isDrifting = true
        end
    end

    return isDrifting, driftAngle
end

function DriftSystem.ApplyDriftBonus(character, driftAngle, deltaTime)
    -- Build boost meter while drifting
    local boostGain = driftAngle * 0.1 * deltaTime
    character.BoostMeter.Value = math.min(100, character.BoostMeter.Value + boostGain)

    -- Visual feedback
    if character.BoostMeter.Value >= 100 then
        -- Trigger boost ready effect
        DriftSystem.ShowBoostReadyEffect(character)
    end
end

return DriftSystem
```

### Drift Effects

Visual effects enhance the drifting experience:

```lua
-- EffectsHandler in StarterCharacterScripts
local function CreateDriftParticles(character)
    local primaryPart = character.PrimaryPart

    -- Create tire smoke particles
    local leftSmoke = Instance.new("ParticleEmitter")
    leftSmoke.Name = "LeftDriftSmoke"
    leftSmoke.Texture = "rbxasset://textures/particles/smoke_main.dds"
    leftSmoke.Rate = 50
    leftSmoke.Lifetime = NumberRange.new(0.5, 1)
    leftSmoke.Speed = NumberRange.new(5, 10)
    leftSmoke.Color = ColorSequence.new(Color3.fromRGB(200, 200, 200))
    leftSmoke.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 1)
    })
    leftSmoke.Enabled = false
    leftSmoke.Parent = primaryPart

    -- Create right smoke (similar configuration)
    local rightSmoke = leftSmoke:Clone()
    rightSmoke.Name = "RightDriftSmoke"
    rightSmoke.Parent = primaryPart

    return leftSmoke, rightSmoke
end
```

## Boost System

The boost system provides burst speed when activated, consuming the boost meter accumulated through drifting.

### Boost Activation

```lua
-- BoostController in StarterCharacterScripts
local character = script.Parent
local boostMeter = character:WaitForChild("BoostMeter")

local boosting = false
local boostSoundEffect = character.PrimaryPart:WaitForChild("BoostSound")

function ActivateBoost()
    if boostMeter.Value >= 100 and not boosting then
        boosting = true
        boostSoundEffect:Play()

        -- Create boost visual effect
        local boostTrail = character.PrimaryPart:WaitForChild("BoostTrail")
        boostTrail.Enabled = true

        -- Boost will be consumed by PhysicsHandler
        wait(0.1)
        boostTrail.Enabled = false
        boosting = false
    end
end
```

### Boost Consumption

The boost meter depletes while boost is active, handled in the physics update loop:

```lua
-- In PhysicsHandler.UpdateMovement
if inputs.boost and character.BoostMeter.Value > 0 then
    maxSpeed = maxSpeed * stats.BoostMultiplier

    -- Consume boost over time
    local boostConsumptionRate = 20  -- Points per second
    character.BoostMeter.Value = math.max(0,
        character.BoostMeter.Value - deltaTime * boostConsumptionRate)
end
```

## Collision System

Collisions between characters and environment are handled to create dynamic racing interactions.

### Character Collisions

When characters collide, their velocities are affected based on weight and impact angle:

```lua
-- CollisionHandler ModuleScript
local CollisionHandler = {}

function CollisionHandler.HandleCharacterCollision(char1, char2)
    local stats1 = char1:GetAttribute("Stats")
    local stats2 = char2:GetAttribute("Stats")

    local velocity1 = char1.PrimaryPart.AssemblyLinearVelocity
    local velocity2 = char2.PrimaryPart.AssemblyLinearVelocity

    -- Calculate collision normal
    local direction = (char2.PrimaryPart.Position - char1.PrimaryPart.Position).Unit

    -- Apply impulse based on weight difference
    local mass1 = stats1.Weight
    local mass2 = stats2.Weight

    local impulse1 = direction * (mass2 / mass1) * 50
    local impulse2 = -direction * (mass1 / mass2) * 50

    -- Apply velocities
    char1.PrimaryPart.AssemblyLinearVelocity = velocity1 + impulse1
    char2.PrimaryPart.AssemblyLinearVelocity = velocity2 + impulse2

    -- Play collision sound
    CollisionHandler.PlayCollisionSound(char1, char2)
end

return CollisionHandler
```

### Wall Collisions

Colliding with track boundaries reduces speed and prevents clipping:

```lua
function CollisionHandler.HandleWallCollision(character, hitPart)
    -- Reduce velocity when hitting walls
    local velocity = character.PrimaryPart.AssemblyLinearVelocity
    local normal = hitPart.CFrame.UpVector

    -- Reflect velocity off wall with energy loss
    local reflectedVelocity = velocity - 2 * velocity:Dot(normal) * normal
    character.PrimaryPart.AssemblyLinearVelocity = reflectedVelocity * 0.5

    -- Reset boost if collision is severe
    if velocity.Magnitude > 40 then
        character.BoostMeter.Value = 0
    end
end
```

## Checkpoint System

Checkpoints track player progress through the race and ensure proper path following.

### Checkpoint Detection

Checkpoints use [`Region3`](https://create.roblox.com/docs/reference/engine/datatypes/Region3) or part touch detection:

```lua
-- CheckpointSystem ModuleScript
local CheckpointSystem = {}

function CheckpointSystem.InitializeCheckpoints(track)
    local checkpoints = track.Checkpoints:GetChildren()

    for i, checkpoint in ipairs(checkpoints) do
        checkpoint.Touched:Connect(function(hit)
            local character = hit.Parent
            if character and character:FindFirstChild("Humanoid") then
                CheckpointSystem.OnCheckpointReached(character, i)
            end
        end)
    end
end

function CheckpointSystem.OnCheckpointReached(character, checkpointIndex)
    local raceData = character:FindFirstChild("RaceData")
    if not raceData then return end

    local currentCheckpoint = raceData.CurrentCheckpoint.Value
    local expectedCheckpoint = currentCheckpoint + 1

    -- Check if player hit checkpoint in order
    if checkpointIndex == expectedCheckpoint then
        raceData.CurrentCheckpoint.Value = checkpointIndex

        -- Check if lap completed
        if checkpointIndex == 0 then  -- Back to start
            CheckpointSystem.CompleteLap(character)
        end
    elseif checkpointIndex < currentCheckpoint - 1 then
        -- Player is going backwards
        CheckpointSystem.ShowWrongWayWarning(character)
    end
end

return CheckpointSystem
```

### Lap Tracking

When players pass through all checkpoints and return to start:

```lua
function CheckpointSystem.CompleteLap(character)
    local raceData = character:FindFirstChild("RaceData")
    local lapTime = tick() - raceData.LapStartTime.Value

    -- Update lap count
    raceData.CurrentLap.Value = raceData.CurrentLap.Value + 1
    raceData.LapStartTime.Value = tick()

    -- Check for personal best
    if lapTime < raceData.BestLapTime.Value or raceData.BestLapTime.Value == 0 then
        raceData.BestLapTime.Value = lapTime
        CheckpointSystem.AwardBestLapBonus(character)
    end

    -- Check if race complete
    local totalLaps = game.ReplicatedStorage.Configurations.RaceConfiguration.LapCount
    if raceData.CurrentLap.Value > totalLaps then
        CheckpointSystem.FinishRace(character)
    end
end
```

## Customization Examples

### Adjusting Physics Feel

To make racing feel more arcade-style or simulation-style, adjust these parameters:

```lua
-- For arcade feel (easier handling)
stats.Acceleration = 12  -- Faster acceleration
stats.Handling = 10      -- Responsive turning
BoostMultiplier = 2.0    -- Powerful boost

-- For simulation feel (realistic physics)
stats.Acceleration = 6   -- Gradual acceleration
stats.Handling = 6       -- Slower turning
BoostMultiplier = 1.3    -- Subtle boost
```

### Creating Power Slide Mechanics

Add a power slide feature that rewards extended drifts:

```lua
function DriftSystem.UpdatePowerSlide(character, isDrifting, deltaTime)
    if not character:FindFirstChild("PowerSlideTimer") then
        local timer = Instance.new("NumberValue")
        timer.Name = "PowerSlideTimer"
        timer.Value = 0
        timer.Parent = character
    end

    local timer = character.PowerSlideTimer

    if isDrifting then
        timer.Value = timer.Value + deltaTime

        -- Bonus boost for extended drifts
        if timer.Value > 2 then
            character.BoostMeter.Value = math.min(100,
                character.BoostMeter.Value + deltaTime * 30)
        end
    else
        timer.Value = 0
    end
end
```

## Performance Optimization

For smooth gameplay with multiple racers:

- Use [`RunService.Heartbeat`](https://create.roblox.com/docs/reference/engine/classes/RunService#Heartbeat) for physics updates instead of loops.
- Implement spatial partitioning for collision detection.
- Use object pooling for particle effects.
- Optimize checkpoint detection with region-based systems rather than per-frame raycasts.

## Next Steps

Now that you understand the racing mechanics, explore the [Character System](./character-system.md) to learn how to create and balance unique raceable characters.
