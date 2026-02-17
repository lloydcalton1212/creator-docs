---
title: Reward System
comments:
description: Configure the economy and progression mechanics that drive player engagement in Brainrot Racing.
prev: /resources/brainrot-racing/character-system
next: /resources/brainrot-racing/track-design
---

The reward system in Brainrot Racing creates a progression loop that motivates players to improve their racing skills and engage with the game regularly. This guide explains how to configure and customize the economy, unlocks, and achievement systems.

## Economy Overview

The game uses **Brainrots** as the primary currency, earned through racing performance and spent on unlocking new characters and cosmetics. The economy balances these factors:

- **Earning Rate** — How quickly players accumulate currency.
- **Unlock Costs** — Price of characters and items.
- **Reward Distribution** — Balance between skill and participation rewards.
- **Retention Incentives** — Daily bonuses and challenges.

### Currency System

The currency is managed server-side to prevent cheating:

```lua
-- DataService in ServerScriptService
local DataService = {}
local DataStoreService = game:GetService("DataStoreService")
local playerDataStore = DataStoreService:GetDataStore("PlayerData_v1")

-- Default player data structure
local function GetDefaultData()
    return {
        Currency = 0,
        UnlockedCharacters = {
            ["BasicBrainrot"] = true,  -- Starter character
        },
        Statistics = {
            RacesCompleted = 0,
            TotalWins = 0,
            BestLapTimes = {},
            TotalDistance = 0,
        },
        DailyProgress = {
            LastLoginDate = os.date("%Y-%m-%d"),
            DailyRacesCompleted = 0,
            ClaimedDailyReward = false,
        },
    }
end

function DataService.GetPlayerData(player)
    local success, data = pcall(function()
        return playerDataStore:GetAsync(player.UserId)
    end)

    if success and data then
        return data
    else
        return GetDefaultData()
    end
end

function DataService.SavePlayerData(player, data)
    local success = pcall(function()
        playerDataStore:SetAsync(player.UserId, data)
    end)

    return success
end

return DataService
```

## Race Rewards

Players earn currency based on race performance with multiple factors contributing to the final payout.

### Position-Based Rewards

The primary reward comes from finishing position:

```lua
-- RewardSystem ModuleScript
local RewardSystem = {}
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local economyConfig = require(ReplicatedStorage.Configurations.EconomySettings)

function RewardSystem.CalculatePositionReward(position, totalPlayers)
    local baseReward = economyConfig.PositionRewards[position] or 0

    -- Scale reward based on competition
    local competitionMultiplier = 1 + (totalPlayers - 2) * 0.1

    return math.floor(baseReward * competitionMultiplier)
end
```

### Performance Bonuses

Reward exceptional racing performance:

```lua
function RewardSystem.CalculatePerformanceBonuses(raceData)
    local bonuses = {}
    local totalBonus = 0

    -- Perfect lap bonus (no collisions)
    if raceData.CollisionsThisLap == 0 then
        local perfectLapBonus = economyConfig.BonusRewards.PerfectLap
        table.insert(bonuses, {Type = "Perfect Lap", Amount = perfectLapBonus})
        totalBonus = totalBonus + perfectLapBonus
    end

    -- New personal best
    if raceData.IsNewPersonalBest then
        local recordBonus = economyConfig.BonusRewards.NewRecord
        table.insert(bonuses, {Type = "New Record", Amount = recordBonus})
        totalBonus = totalBonus + recordBonus
    end

    -- Comeback victory (winning from 5th or lower)
    if raceData.FinalPosition == 1 and raceData.LowestPosition >= 5 then
        local comebackBonus = economyConfig.BonusRewards.Comeback
        table.insert(bonuses, {Type = "Comeback", Amount = comebackBonus})
        totalBonus = totalBonus + comebackBonus
    end

    -- First to use power-up
    if raceData.FirstPowerUpUser then
        local firstBloodBonus = economyConfig.BonusRewards.FirstBlood
        table.insert(bonuses, {Type = "First Strike", Amount = firstBloodBonus})
        totalBonus = totalBonus + firstBloodBonus
    end

    return bonuses, totalBonus
end
```

### Reward Distribution

Award rewards at race completion:

```lua
function RewardSystem.AwardRaceRewards(player, position, totalPlayers, raceData)
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    -- Calculate total reward
    local positionReward = RewardSystem.CalculatePositionReward(position, totalPlayers)
    local bonuses, bonusTotal = RewardSystem.CalculatePerformanceBonuses(raceData)

    -- Apply daily multiplier if applicable
    local multiplier = 1.0
    if not playerData.DailyProgress.ClaimedDailyReward then
        multiplier = economyConfig.DailyRewardMultiplier
        playerData.DailyProgress.ClaimedDailyReward = true
    end

    local totalReward = math.floor((positionReward + bonusTotal) * multiplier)

    -- Award currency
    playerData.Currency = playerData.Currency + totalReward

    -- Update statistics
    playerData.Statistics.RacesCompleted = playerData.Statistics.RacesCompleted + 1
    if position == 1 then
        playerData.Statistics.TotalWins = playerData.Statistics.TotalWins + 1
    end

    -- Save data
    dataService.SavePlayerData(player, playerData)

    -- Send reward notification to player
    RewardSystem.NotifyPlayer(player, totalReward, bonuses, multiplier)

    return totalReward
end
```

## Daily Challenges

Daily challenges provide additional goals and rewards:

### Challenge System

```lua
-- ChallengeSystem ModuleScript
local ChallengeSystem = {}

-- Define challenge types
local ChallengeTemplates = {
    {
        Id = "complete_races",
        Name = "Race Enthusiast",
        Description = "Complete 5 races",
        Goal = 5,
        Reward = 500,
        Check = function(playerData)
            return playerData.DailyProgress.DailyRacesCompleted >= 5
        end
    },
    {
        Id = "win_races",
        Name = "Victory Streak",
        Description = "Win 3 races",
        Goal = 3,
        Reward = 800,
        Check = function(playerData)
            return playerData.DailyProgress.WinsToday >= 3
        end
    },
    {
        Id = "perfect_laps",
        Name = "Flawless Driver",
        Description = "Complete 3 perfect laps",
        Goal = 3,
        Reward = 600,
        Check = function(playerData)
            return playerData.DailyProgress.PerfectLapsToday >= 3
        end
    },
    {
        Id = "distance_traveled",
        Name = "Marathon Runner",
        Description = "Travel 10000 studs",
        Goal = 10000,
        Reward = 400,
        Check = function(playerData)
            return playerData.DailyProgress.DistanceToday >= 10000
        end
    },
}

function ChallengeSystem.GenerateDailyChallenges()
    -- Select 3 random challenges for the day
    local shuffled = {}
    for _, template in ipairs(ChallengeTemplates) do
        table.insert(shuffled, template)
    end

    -- Shuffle array
    for i = #shuffled, 2, -1 do
        local j = math.random(i)
        shuffled[i], shuffled[j] = shuffled[j], shuffled[i]
    end

    -- Return first 3 challenges
    local dailyChallenges = {}
    for i = 1, math.min(3, #shuffled) do
        table.insert(dailyChallenges, shuffled[i])
    end

    return dailyChallenges
end

function ChallengeSystem.CheckChallengeCompletion(player, challengeId)
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    -- Find challenge template
    local challenge = nil
    for _, template in ipairs(ChallengeTemplates) do
        if template.Id == challengeId then
            challenge = template
            break
        end
    end

    if not challenge then return false end

    -- Check if already claimed today
    if playerData.DailyProgress.CompletedChallenges[challengeId] then
        return false
    end

    -- Check completion
    if challenge.Check(playerData) then
        -- Award reward
        playerData.Currency = playerData.Currency + challenge.Reward
        playerData.DailyProgress.CompletedChallenges[challengeId] = true

        dataService.SavePlayerData(player, playerData)

        -- Notify player
        ChallengeSystem.NotifyChallengeComplete(player, challenge)

        return true
    end

    return false
end

return ChallengeSystem
```

### Daily Reset

Reset daily progress at the start of each day:

```lua
function ChallengeSystem.CheckAndResetDaily(player)
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    local today = os.date("%Y-%m-%d")

    if playerData.DailyProgress.LastLoginDate ~= today then
        -- Reset daily progress
        playerData.DailyProgress = {
            LastLoginDate = today,
            DailyRacesCompleted = 0,
            WinsToday = 0,
            PerfectLapsToday = 0,
            DistanceToday = 0,
            ClaimedDailyReward = false,
            CompletedChallenges = {},
        }

        dataService.SavePlayerData(player, playerData)

        -- Generate new challenges
        local challenges = ChallengeSystem.GenerateDailyChallenges()
        return challenges
    end

    return nil
end
```

## Progression System

Track player advancement through levels and milestones:

```lua
-- ProgressionSystem ModuleScript
local ProgressionSystem = {}

function ProgressionSystem.CalculateLevel(totalRaces, totalWins)
    -- Simple level calculation based on races and wins
    local racePoints = totalRaces * 10
    local winPoints = totalWins * 50
    local totalPoints = racePoints + winPoints

    -- Level formula: Level = floor(sqrt(totalPoints / 100))
    local level = math.floor(math.sqrt(totalPoints / 100))

    return math.max(1, level)
end

function ProgressionSystem.GetLevelRewards(level)
    local rewards = {
        [5] = {Currency = 1000, UnlockCharacter = "SpeedDemon"},
        [10] = {Currency = 2000, UnlockTrack = "VolcanoCircuit"},
        [15] = {Currency = 3000, UnlockCharacter = "DriftKing"},
        [20] = {Currency = 5000, UnlockGameMode = "BattleRace"},
        [25] = {Currency = 7500, UnlockCharacter = "Juggernaut"},
    }

    return rewards[level]
end

function ProgressionSystem.CheckLevelUp(player)
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    local currentLevel = playerData.Statistics.Level or 1
    local newLevel = ProgressionSystem.CalculateLevel(
        playerData.Statistics.RacesCompleted,
        playerData.Statistics.TotalWins
    )

    if newLevel > currentLevel then
        -- Level up!
        playerData.Statistics.Level = newLevel

        -- Award level rewards
        local rewards = ProgressionSystem.GetLevelRewards(newLevel)
        if rewards then
            if rewards.Currency then
                playerData.Currency = playerData.Currency + rewards.Currency
            end
            if rewards.UnlockCharacter then
                playerData.UnlockedCharacters[rewards.UnlockCharacter] = true
            end

            ProgressionSystem.NotifyLevelUp(player, newLevel, rewards)
        end

        dataService.SavePlayerData(player, playerData)

        return true, newLevel
    end

    return false, currentLevel
end

return ProgressionSystem
```

## Achievement System

Reward long-term goals with achievements:

```lua
-- AchievementSystem ModuleScript
local AchievementSystem = {}

local Achievements = {
    {
        Id = "first_win",
        Name = "First Victory",
        Description = "Win your first race",
        Reward = 500,
        Icon = "rbxassetid://123456789",
        Check = function(stats) return stats.TotalWins >= 1 end
    },
    {
        Id = "speed_demon",
        Name = "Speed Demon",
        Description = "Complete a lap in under 60 seconds",
        Reward = 1000,
        Icon = "rbxassetid://123456790",
        Check = function(stats)
            for _, time in pairs(stats.BestLapTimes) do
                if time < 60 then return true end
            end
            return false
        end
    },
    {
        Id = "veteran_racer",
        Name = "Veteran Racer",
        Description = "Complete 100 races",
        Reward = 2000,
        Icon = "rbxassetid://123456791",
        Check = function(stats) return stats.RacesCompleted >= 100 end
    },
    {
        Id = "champion",
        Name = "Champion",
        Description = "Win 50 races",
        Reward = 5000,
        Icon = "rbxassetid://123456792",
        Check = function(stats) return stats.TotalWins >= 50 end
    },
}

function AchievementSystem.CheckAchievements(player)
    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(player)

    if not playerData.UnlockedAchievements then
        playerData.UnlockedAchievements = {}
    end

    local newAchievements = {}

    for _, achievement in ipairs(Achievements) do
        -- Skip if already unlocked
        if not playerData.UnlockedAchievements[achievement.Id] then
            -- Check if conditions met
            if achievement.Check(playerData.Statistics) then
                -- Unlock achievement
                playerData.UnlockedAchievements[achievement.Id] = true
                playerData.Currency = playerData.Currency + achievement.Reward

                table.insert(newAchievements, achievement)
            end
        end
    end

    if #newAchievements > 0 then
        dataService.SavePlayerData(player, playerData)

        -- Notify player
        for _, achievement in ipairs(newAchievements) do
            AchievementSystem.NotifyAchievement(player, achievement)
        end
    end

    return newAchievements
end

return AchievementSystem
```

## Economy Configuration

Fine-tune the economy in `ReplicatedStorage/Configurations/EconomySettings`:

```lua
local EconomySettings = {
    -- Position rewards (base values)
    PositionRewards = {
        [1] = 500,   -- 1st place
        [2] = 350,   -- 2nd place
        [3] = 250,   -- 3rd place
        [4] = 150,   -- 4th place
        [5] = 100,   -- 5th place
        [6] = 75,    -- 6th place
        [7] = 50,    -- 7th place
        [8] = 25,    -- 8th place
    },

    -- Bonus rewards
    BonusRewards = {
        PerfectLap = 100,
        NewRecord = 200,
        Comeback = 150,
        FirstBlood = 50,
    },

    -- Multipliers
    DailyRewardMultiplier = 2.0,  -- First race of the day
    WeekendMultiplier = 1.5,      -- Saturday and Sunday races

    -- Unlock costs (can be overridden per character)
    DefaultUnlockCosts = {
        Common = 500,
        Rare = 1500,
        Epic = 3000,
        Legendary = 5000,
    },
}

return EconomySettings
```

## Balancing the Economy

Consider these factors when tuning rewards:

### Earning Rate

Calculate how long it takes to unlock content:

```lua
-- Example: Time to unlock a 2000 cost character
-- Average race: 3rd place = 250 currency
-- With bonuses: ~300 currency per race
-- Races needed: 2000 / 300 = ~7 races
-- At 5 minutes per race: ~35 minutes

-- Adjust if this feels too fast or too slow
```

### Retention Metrics

Design rewards to encourage regular play:

- **Daily bonuses** — Incentivize daily logins.
- **Weekly challenges** — Provide long-term goals.
- **Seasonal events** — Special limited-time rewards.
- **Battle pass** — Progressive unlocks over a season.

### Monetization Balance

If implementing in-game purchases:

- Ensure free players can unlock content through gameplay.
- Purchases should accelerate, not replace, progression.
- Cosmetic items work well for premium purchases.
- Avoid pay-to-win mechanics that unbalance competitive play.

## Anti-Cheat Measures

Protect the economy from exploitation:

```lua
-- Server-side validation
function RewardSystem.ValidateRaceResult(player, reportedPosition, raceId)
    local raceManager = require(game.ServerScriptService.RaceManager)
    local actualPosition = raceManager.GetPlayerPosition(player, raceId)

    -- Verify position matches server records
    if actualPosition ~= reportedPosition then
        warn("Position mismatch for player:", player.Name)
        return false
    end

    -- Check for impossible times
    local lapTime = raceManager.GetLapTime(player, raceId)
    local minPossibleTime = raceManager.GetMinimumLapTime(raceId)

    if lapTime < minPossibleTime * 0.9 then  -- 10% tolerance
        warn("Suspicious lap time for player:", player.Name)
        return false
    end

    return true
end
```

## Next Steps

With the reward system configured, learn how to create custom racing tracks in the [Track Design](./track-design.md) guide.
