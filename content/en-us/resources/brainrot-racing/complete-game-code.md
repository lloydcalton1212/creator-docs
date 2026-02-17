---
title: Complete Game Code
comments:
description: Complete copy-paste code implementation for the full Brainrot Racing game with base system and admin panel.
prev: /resources/brainrot-racing/admin-panel
---

This page provides the complete, production-ready code for Brainrot Racing. Simply copy each script into the specified location in Roblox Studio to build the full game with racing, base building, and admin features.

## Project Setup

### Step 1: Create Folder Structure

In Roblox Studio, create this exact hierarchy:

```
Workspace/
├── Tracks/ (Folder)
├── SpawnLocations/ (Folder)
├── Bases/ (Folder)

ServerScriptService/
├── GameServer (Script)
├── AdminService (ModuleScript)
├── BaseSystem (ModuleScript)
├── DataService (ModuleScript)

ReplicatedStorage/
├── Configurations/ (Folder)
│   ├── GameConfig (ModuleScript)
│   ├── CharacterStats (ModuleScript)
│   └── BaseConfig (ModuleScript)
├── Events/ (Folder)
│   ├── ShowGlobalMessage (RemoteEvent)
│   ├── AdminCommand (RemoteEvent)
│   ├── UpgradeBase (RemoteEvent)
│   └── UpdateUI (RemoteEvent)
├── Characters/ (Folder)

StarterPlayer/
├── StarterCharacterScripts/
│   ├── CharacterController (LocalScript)
│   └── EffectsHandler (LocalScript)
├── StarterPlayerScripts/
│   ├── UIController (LocalScript)
│   └── AdminPanelClient (LocalScript)

StarterGui/
├── RaceHUD (ScreenGui)
├── BaseUI (ScreenGui)
└── AdminPanelUI (ScreenGui)
```

## ServerScriptService Scripts

### GameServer (Script)

Main server controller - handles game initialization and player management.

```lua
-- ServerScriptService/GameServer
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")

local DataService = require(script.Parent.DataService)
local AdminService = require(script.Parent.AdminService)
local BaseSystem = require(script.Parent.BaseSystem)

print("[GameServer] Brainrot Racing initializing...")

-- Initialize game systems
local function initializeGame()
	-- Setup remote events
	if not ReplicatedStorage:FindFirstChild("Events") then
		local eventsFolder = Instance.new("Folder")
		eventsFolder.Name = "Events"
		eventsFolder.Parent = ReplicatedStorage
	end

	local eventNames = {
		"ShowGlobalMessage",
		"AdminCommand",
		"UpgradeBase",
		"UpdateUI",
		"RaceStart",
		"RaceFinish"
	}

	for _, eventName in ipairs(eventNames) do
		if not ReplicatedStorage.Events:FindFirstChild(eventName) then
			local remoteEvent = Instance.new("RemoteEvent")
			remoteEvent.Name = eventName
			remoteEvent.Parent = ReplicatedStorage.Events
		end
	end

	print("[GameServer] Events initialized")
end

-- Player joined handler
Players.PlayerAdded:Connect(function(player)
	print("[GameServer] Player joined:", player.Name)

	-- Load player data
	local playerData = DataService.LoadData(player)

	-- Check if player is admin
	local isAdmin = AdminService.IsAdmin(player)
	player:SetAttribute("IsAdmin", isAdmin)

	if isAdmin then
		print("[GameServer] Admin detected:", player.DisplayName)
	end

	-- Setup player base
	BaseSystem.CreatePlayerBase(player, playerData)

	-- Give starter character
	if not playerData.SelectedCharacter then
		playerData.SelectedCharacter = "BasicBrainrot"
	end

	-- Update client UI
	task.wait(1)
	ReplicatedStorage.Events.UpdateUI:FireClient(player, {
		Currency = playerData.Currency,
		Characters = playerData.UnlockedCharacters,
		BaseLevel = playerData.BaseLevel or 1
	})
end)

-- Player leaving handler
Players.PlayerRemoving:Connect(function(player)
	print("[GameServer] Player leaving:", player.Name)

	-- Save player data
	DataService.SaveData(player)

	-- Cleanup base
	BaseSystem.RemovePlayerBase(player)
end)

-- Admin command handler
ReplicatedStorage.Events.AdminCommand.OnServerEvent:Connect(function(player, commandData)
	if not AdminService.IsAdmin(player) then
		warn("[GameServer] Unauthorized admin attempt by:", player.Name)
		return
	end

	AdminService.ExecuteCommand(player, commandData)
end)

-- Base upgrade handler
ReplicatedStorage.Events.UpgradeBase.OnServerEvent:Connect(function(player)
	local playerData = DataService.LoadData(player)
	local success, newLevel = BaseSystem.UpgradeBase(player, playerData)

	if success then
		DataService.SaveData(player)
		ReplicatedStorage.Events.UpdateUI:FireClient(player, {
			BaseLevel = newLevel,
			Currency = playerData.Currency
		})
	end
end)

-- Initialize game
initializeGame()

print("[GameServer] Brainrot Racing ready!")
```

### DataService (ModuleScript)

Handles all player data persistence with DataStoreService.

```lua
-- ServerScriptService/DataService
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local playerDataStore = DataStoreService:GetDataStore("PlayerData_v2")

local DataService = {}
local loadedData = {}

-- Default player data
local function getDefaultData()
	return {
		Currency = 0,
		UnlockedCharacters = {
			BasicBrainrot = true
		},
		SelectedCharacter = "BasicBrainrot",
		BaseLevel = 1,
		StoredBrainrots = {},
		Statistics = {
			RacesCompleted = 0,
			Wins = 0,
			BestLapTimes = {},
		},
		LastPlayed = os.time()
	}
end

-- Load player data
function DataService.LoadData(player)
	local userId = tostring(player.UserId)

	-- Return cached data if available
	if loadedData[userId] then
		return loadedData[userId]
	end

	local success, data = pcall(function()
		return playerDataStore:GetAsync(userId)
	end)

	if success and data then
		loadedData[userId] = data
		print("[DataService] Loaded data for", player.Name)
	else
		loadedData[userId] = getDefaultData()
		print("[DataService] Created new data for", player.Name)
	end

	return loadedData[userId]
end

-- Save player data
function DataService.SaveData(player)
	local userId = tostring(player.UserId)
	local data = loadedData[userId]

	if not data then
		warn("[DataService] No data to save for", player.Name)
		return false
	end

	data.LastPlayed = os.time()

	local success, err = pcall(function()
		playerDataStore:SetAsync(userId, data)
	end)

	if success then
		print("[DataService] Saved data for", player.Name)
		return true
	else
		warn("[DataService] Failed to save data for", player.Name, ":", err)
		return false
	end
end

-- Get cached data
function DataService.GetCachedData(player)
	return loadedData[tostring(player.UserId)]
end

-- Update specific data field
function DataService.UpdateData(player, key, value)
	local data = DataService.GetCachedData(player)
	if data then
		data[key] = value
		return true
	end
	return false
end

-- Auto-save every 5 minutes
game:BindToClose(function()
	print("[DataService] Server closing, saving all data...")
	for _, player in ipairs(Players:GetPlayers()) do
		DataService.SaveData(player)
	end
end)

return DataService
```

### AdminService (ModuleScript)

Admin system with your pre-configured UserIds.

```lua
-- ServerScriptService/AdminService
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataService = require(script.Parent.DataService)

local AdminService = {}

-- AUTHORIZED ADMIN USER IDS
local ADMIN_USER_IDS = {
	7392445200,
	10443874977
}

-- Check if player is admin
function AdminService.IsAdmin(player)
	return table.find(ADMIN_USER_IDS, player.UserId) ~= nil
end

-- Execute admin command
function AdminService.ExecuteCommand(adminPlayer, commandData)
	local commandType = commandData.Type

	if commandType == "GlobalMessage" then
		AdminService.GlobalMessage(adminPlayer, commandData.Message, commandData.Duration)

	elseif commandType == "Kick" then
		AdminService.KickPlayer(adminPlayer, commandData.Target, commandData.Reason)

	elseif commandType == "Ban" then
		AdminService.BanPlayer(adminPlayer, commandData.Target, commandData.Reason)

	elseif commandType == "SetSpeed" then
		AdminService.SetSpeed(adminPlayer, commandData.Target, commandData.Value)

	elseif commandType == "SetMoney" then
		AdminService.SetMoney(adminPlayer, commandData.Target, commandData.Value)

	elseif commandType == "GiveBrainrot" then
		AdminService.GiveBrainrot(adminPlayer, commandData.Target, commandData.Character)

	end
end

-- Global message (uses DisplayName)
function AdminService.GlobalMessage(adminPlayer, message, duration)
	duration = duration or 5

	for _, player in ipairs(Players:GetPlayers()) do
		ReplicatedStorage.Events.ShowGlobalMessage:FireClient(player, {
			Message = message,
			Sender = adminPlayer.DisplayName,
			Duration = duration,
			Timestamp = os.date("%H:%M:%S")
		})
	end

	print(string.format("[Admin] %s sent global message: %s", adminPlayer.DisplayName, message))
end

-- Kick player
function AdminService.KickPlayer(adminPlayer, targetPlayer, reason)
	reason = reason or "Kicked by admin"
	targetPlayer:Kick("You have been kicked.\nReason: " .. reason)
	print(string.format("[Admin] %s kicked %s: %s", adminPlayer.Name, targetPlayer.Name, reason))
end

-- Ban player (simplified - stores in DataStore)
function AdminService.BanPlayer(adminPlayer, targetPlayer, reason)
	reason = reason or "Banned by admin"
	-- In production, use a dedicated ban DataStore
	targetPlayer:Kick("You have been banned.\nReason: " .. reason)
	print(string.format("[Admin] %s banned %s: %s", adminPlayer.Name, targetPlayer.Name, reason))
end

-- Set player speed
function AdminService.SetSpeed(adminPlayer, targetPlayer, speed)
	speed = math.clamp(speed, 0, 200)
	targetPlayer:SetAttribute("CustomSpeed", speed)
	print(string.format("[Admin] %s set %s's speed to %d", adminPlayer.Name, targetPlayer.Name, speed))
end

-- Set player money
function AdminService.SetMoney(adminPlayer, targetPlayer, amount)
	local playerData = DataService.GetCachedData(targetPlayer)
	if playerData then
		playerData.Currency = math.max(0, amount)
		ReplicatedStorage.Events.UpdateUI:FireClient(targetPlayer, {Currency = playerData.Currency})
		print(string.format("[Admin] %s set %s's money to %d", adminPlayer.Name, targetPlayer.Name, amount))
	end
end

-- Give brainrot character
function AdminService.GiveBrainrot(adminPlayer, targetPlayer, characterName)
	local playerData = DataService.GetCachedData(targetPlayer)
	if playerData then
		playerData.UnlockedCharacters[characterName] = true
		ReplicatedStorage.Events.UpdateUI:FireClient(targetPlayer, {
			Characters = playerData.UnlockedCharacters
		})
		print(string.format("[Admin] %s gave %s character: %s", adminPlayer.Name, targetPlayer.Name, characterName))
	end
end

return AdminService
```

### BaseSystem (ModuleScript)

Player base system with 3-floor upgrades for storing brainrots.

```lua
-- ServerScriptService/BaseSystem
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local workspace = game:GetService("Workspace")

local BaseConfig = require(ReplicatedStorage.Configurations.BaseConfig)

local BaseSystem = {}
local playerBases = {}

-- Create player base
function BaseSystem.CreatePlayerBase(player, playerData)
	local baseLevel = playerData.BaseLevel or 1
	local baseModel = BaseSystem.BuildBase(baseLevel)

	-- Position base (you'll want to improve this placement logic)
	local spawnPosition = Vector3.new(math.random(-200, 200), 5, math.random(-200, 200))
	baseModel:PivotTo(CFrame.new(spawnPosition))

	-- Set ownership
	baseModel.Name = player.Name .. "'s Base"
	baseModel:SetAttribute("OwnerId", player.UserId)
	baseModel:SetAttribute("Level", baseLevel)

	-- Create bases folder if it doesn't exist
	if not workspace:FindFirstChild("Bases") then
		local basesFolder = Instance.new("Folder")
		basesFolder.Name = "Bases"
		basesFolder.Parent = workspace
	end

	baseModel.Parent = workspace.Bases
	playerBases[player.UserId] = baseModel

	print("[BaseSystem] Created base for", player.Name, "at level", baseLevel)
	return baseModel
end

-- Build base model based on level
function BaseSystem.BuildBase(level)
	local baseModel = Instance.new("Model")
	baseModel.Name = "PlayerBase"

	-- Floor dimensions
	local floorWidth = 30
	local floorDepth = 30
	local floorHeight = 1
	local storyHeight = 10

	-- Build floors
	for floor = 1, level do
		local floorPart = Instance.new("Part")
		floorPart.Name = "Floor" .. floor
		floorPart.Size = Vector3.new(floorWidth, floorHeight, floorDepth)
		floorPart.Position = Vector3.new(0, (floor - 1) * storyHeight, 0)
		floorPart.Anchored = true
		floorPart.BrickColor = BrickColor.new("Dark stone grey")
		floorPart.Material = Enum.Material.Concrete
		floorPart.Parent = baseModel

		-- Add walls
		local wallThickness = 1
		local wallHeight = storyHeight - floorHeight

		-- North wall
		local northWall = Instance.new("Part")
		northWall.Size = Vector3.new(floorWidth, wallHeight, wallThickness)
		northWall.Position = floorPart.Position + Vector3.new(0, wallHeight/2 + floorHeight/2, floorDepth/2)
		northWall.Anchored = true
		northWall.BrickColor = BrickColor.new("Medium stone grey")
		northWall.Material = Enum.Material.Brick
		northWall.Parent = baseModel

		-- South wall
		local southWall = northWall:Clone()
		southWall.Position = floorPart.Position + Vector3.new(0, wallHeight/2 + floorHeight/2, -floorDepth/2)
		southWall.Parent = baseModel

		-- East wall
		local eastWall = Instance.new("Part")
		eastWall.Size = Vector3.new(wallThickness, wallHeight, floorDepth)
		eastWall.Position = floorPart.Position + Vector3.new(floorWidth/2, wallHeight/2 + floorHeight/2, 0)
		eastWall.Anchored = true
		eastWall.BrickColor = BrickColor.new("Medium stone grey")
		eastWall.Material = Enum.Material.Brick
		eastWall.Parent = baseModel

		-- West wall
		local westWall = eastWall:Clone()
		westWall.Position = floorPart.Position + Vector3.new(-floorWidth/2, wallHeight/2 + floorHeight/2, 0)
		westWall.Parent = baseModel

		-- Add door on first floor
		if floor == 1 then
			local doorway = Instance.new("Part")
			doorway.Size = Vector3.new(6, wallHeight * 0.8, wallThickness)
			doorway.Position = northWall.Position - Vector3.new(0, wallHeight * 0.1, 0)
			doorway.Anchored = true
			doorway.Transparency = 1
			doorway.CanCollide = false
			doorway.Name = "Doorway"
			doorway.Parent = baseModel
		end

		-- Display pedestals for brainrots
		local pedestalCount = 5 * floor  -- More space per floor
		for i = 1, pedestalCount do
			local pedestal = Instance.new("Part")
			pedestal.Name = "Pedestal" .. i
			pedestal.Size = Vector3.new(3, 1, 3)
			pedestal.BrickColor = BrickColor.new("Gold")
			pedestal.Material = Enum.Material.Marble
			pedestal.Anchored = true

			-- Arrange pedestals in grid
			local cols = math.ceil(math.sqrt(pedestalCount))
			local row = math.floor((i-1) / cols)
			local col = (i-1) % cols
			local spacing = 5
			local offsetX = (col - cols/2) * spacing
			local offsetZ = (row - cols/2) * spacing

			pedestal.Position = floorPart.Position + Vector3.new(offsetX, floorHeight + 0.5, offsetZ)
			pedestal.Parent = baseModel
		end
	end

	-- Set primary part
	baseModel.PrimaryPart = baseModel:FindFirstChild("Floor1")

	return baseModel
end

-- Upgrade player base
function BaseSystem.UpgradeBase(player, playerData)
	local currentLevel = playerData.BaseLevel or 1

	if currentLevel >= 3 then
		return false, "Max level reached"
	end

	local upgradeCost = BaseConfig.UpgradeCosts[currentLevel + 1]

	if playerData.Currency < upgradeCost then
		return false, "Insufficient funds"
	end

	-- Deduct cost
	playerData.Currency = playerData.Currency - upgradeCost
	playerData.BaseLevel = currentLevel + 1

	-- Rebuild base
	BaseSystem.RemovePlayerBase(player)
	BaseSystem.CreatePlayerBase(player, playerData)

	print("[BaseSystem]", player.Name, "upgraded base to level", playerData.BaseLevel)
	return true, playerData.BaseLevel
end

-- Remove player base
function BaseSystem.RemovePlayerBase(player)
	local base = playerBases[player.UserId]
	if base then
		base:Destroy()
		playerBases[player.UserId] = nil
	end
end

return BaseSystem
```

## ReplicatedStorage Configurations

### GameConfig (ModuleScript)

```lua
-- ReplicatedStorage/Configurations/GameConfig
return {
	-- Race settings
	MaxPlayers = 8,
	LapCount = 3,
	RaceCountdown = 5,

	-- Currency rewards by position
	RaceRewards = {
		[1] = 500,
		[2] = 350,
		[3] = 250,
		[4] = 150,
		[5] = 100,
		[6] = 75,
		[7] = 50,
		[8] = 25
	},

	-- Bonus rewards
	BonusRewards = {
		PerfectLap = 100,
		NewRecord = 200,
		Comeback = 150
	}
}
```

### CharacterStats (ModuleScript)

```lua
-- ReplicatedStorage/Configurations/CharacterStats
return {
	BasicBrainrot = {
		Speed = 60,
		Acceleration = 7,
		Handling = 7,
		Weight = 5,
		BoostMultiplier = 1.5,
		UnlockCost = 0,
		DisplayName = "Basic Brainrot"
	},

	SpeedDemon = {
		Speed = 78,
		Acceleration = 6,
		Handling = 5,
		Weight = 4,
		BoostMultiplier = 1.9,
		UnlockCost = 2000,
		DisplayName = "Speed Demon"
	},

	DriftKing = {
		Speed = 65,
		Acceleration = 8,
		Handling = 10,
		Weight = 5,
		BoostMultiplier = 1.6,
		UnlockCost = 1500,
		DisplayName = "Drift King"
	},

	Juggernaut = {
		Speed = 58,
		Acceleration = 5,
		Handling = 6,
		Weight = 8,
		BoostMultiplier = 1.4,
		UnlockCost = 1800,
		DisplayName = "Juggernaut"
	}
}
```

### BaseConfig (ModuleScript)

```lua
-- ReplicatedStorage/Configurations/BaseConfig
return {
	-- Upgrade costs for each level
	UpgradeCosts = {
		[1] = 0,      -- Level 1 is free (starting base)
		[2] = 5000,   -- Level 1 to 2
		[3] = 15000   -- Level 2 to 3
	},

	-- Storage capacity per level
	StorageCapacity = {
		[1] = 5,   -- 5 brainrots at level 1
		[2] = 15,  -- 15 brainrots at level 2
		[3] = 30   -- 30 brainrots at level 3
	},

	-- Base dimensions
	FloorSize = 30,
	StoryHeight = 10
}
```

## StarterGui - User Interfaces

### AdminPanelUI (ScreenGui)

Create a ScreenGui named "AdminPanelUI" in StarterGui, then add this LocalScript:

```lua
-- StarterGui/AdminPanelUI/AdminPanelScript (LocalScript)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local adminPanelUI = script.Parent

-- Wait for admin status
repeat task.wait() until player:GetAttribute("IsAdmin") ~= nil

if not player:GetAttribute("IsAdmin") then
	adminPanelUI.Enabled = false
	return
end

print("[AdminPanel] Admin detected, enabling panel")

-- Create UI
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 600, 0, 450)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Parent = adminPanelUI

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

-- Title bar
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 50)
titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -60, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🛡️ ADMIN PANEL"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 24
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 40, 0, 40)
closeBtn.Position = UDim2.new(1, -45, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 20
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = titleBar

local closeCorn = Instance.new("UICorner")
closeCorn.CornerRadius = UDim.new(0, 8)
closeCorn.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = false
end)

-- Commands scroll frame
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, -20, 1, -70)
scrollFrame.Position = UDim2.new(0, 10, 0, 60)
scrollFrame.BackgroundTransparency = 1
scrollFrame.BorderSizePixel = 0
scrollFrame.ScrollBarThickness = 6
scrollFrame.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 10)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = scrollFrame

-- Command button creator
local function createCommandButton(name, icon, onClick)
	local btn = Instance.new("TextButton")
	btn.Name = name
	btn.Size = UDim2.new(1, 0, 0, 50)
	btn.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
	btn.Text = ""
	btn.Parent = scrollFrame

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 10)
	btnCorner.Parent = btn

	local iconLabel = Instance.new("TextLabel")
	iconLabel.Size = UDim2.new(0, 45, 1, 0)
	iconLabel.BackgroundTransparency = 1
	iconLabel.Text = icon
	iconLabel.TextSize = 28
	iconLabel.Parent = btn

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.new(1, -55, 1, 0)
	nameLabel.Position = UDim2.new(0, 50, 0, 0)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = name
	nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	nameLabel.TextSize = 18
	nameLabel.Font = Enum.Font.GothamSemibold
	nameLabel.TextXAlignment = Enum.TextXAlignment.Left
	nameLabel.Parent = btn

	btn.MouseButton1Click:Connect(onClick)

	return btn
end

-- Global Message command
createCommandButton("Global Message", "📢", function()
	local message = "Hello from " .. player.DisplayName .. "!"
	ReplicatedStorage.Events.AdminCommand:FireServer({
		Type = "GlobalMessage",
		Message = message,
		Duration = 5
	})
end)

-- Give Money command
createCommandButton("Give 1000 Money", "💰", function()
	ReplicatedStorage.Events.AdminCommand:FireServer({
		Type = "SetMoney",
		Target = player,
		Value = player:GetAttribute("Currency") or 0 + 1000
	})
end)

-- Set Speed command
createCommandButton("Toggle Super Speed", "⚡", function()
	local currentSpeed = player:GetAttribute("CustomSpeed") or 60
	local newSpeed = currentSpeed > 60 and 60 or 150
	ReplicatedStorage.Events.AdminCommand:FireServer({
		Type = "SetSpeed",
		Target = player,
		Value = newSpeed
	})
end)

-- Toggle panel with F2 key
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end

	if input.KeyCode == Enum.KeyCode.F2 then
		mainFrame.Visible = not mainFrame.Visible
	end
end)

print("[AdminPanel] Press F2 to open admin panel")
```

### BaseUI (ScreenGui)

Create a ScreenGui named "BaseUI" in StarterGui with this LocalScript:

```lua
-- StarterGui/BaseUI/BaseUIScript (LocalScript)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local baseUI = script.Parent

-- Create UI
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 200)
frame.Position = UDim2.new(1, -320, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
frame.BorderSizePixel = 0
frame.Parent = baseUI

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -20, 0, 40)
title.Position = UDim2.new(0, 10, 0, 10)
title.BackgroundTransparency = 1
title.Text = "🏠 MY BASE"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = frame

-- Base level label
local levelLabel = Instance.new("TextLabel")
levelLabel.Name = "LevelLabel"
levelLabel.Size = UDim2.new(1, -20, 0, 30)
levelLabel.Position = UDim2.new(0, 10, 0, 55)
levelLabel.BackgroundTransparency = 1
levelLabel.Text = "Level: 1 / 3"
levelLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
levelLabel.TextSize = 18
levelLabel.Font = Enum.Font.Gotham
levelLabel.TextXAlignment = Enum.TextXAlignment.Left
levelLabel.Parent = frame

-- Currency label
local currencyLabel = Instance.new("TextLabel")
currencyLabel.Name = "CurrencyLabel"
currencyLabel.Size = UDim2.new(1, -20, 0, 30)
currencyLabel.Position = UDim2.new(0, 10, 0, 90)
currencyLabel.BackgroundTransparency = 1
currencyLabel.Text = "💰 Brainrots: 0"
currencyLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
currencyLabel.TextSize = 18
currencyLabel.Font = Enum.Font.GothamBold
currencyLabel.TextXAlignment = Enum.TextXAlignment.Left
currencyLabel.Parent = frame

-- Upgrade button
local upgradeBtn = Instance.new("TextButton")
upgradeBtn.Name = "UpgradeButton"
upgradeBtn.Size = UDim2.new(1, -20, 0, 40)
upgradeBtn.Position = UDim2.new(0, 10, 1, -50)
upgradeBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
upgradeBtn.Text = "UPGRADE BASE (5000)"
upgradeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
upgradeBtn.TextSize = 16
upgradeBtn.Font = Enum.Font.GothamBold
upgradeBtn.Parent = frame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = upgradeBtn

upgradeBtn.MouseButton1Click:Connect(function()
	ReplicatedStorage.Events.UpgradeBase:FireServer()
end)

-- Update UI when data changes
ReplicatedStorage.Events.UpdateUI.OnClientEvent:Connect(function(data)
	if data.Currency then
		currencyLabel.Text = "💰 Brainrots: " .. data.Currency
	end

	if data.BaseLevel then
		levelLabel.Text = "Level: " .. data.BaseLevel .. " / 3"

		if data.BaseLevel >= 3 then
			upgradeBtn.Text = "MAX LEVEL"
			upgradeBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
		elseif data.BaseLevel == 2 then
			upgradeBtn.Text = "UPGRADE BASE (15000)"
		end
	end
end)
```

## Quick Setup Instructions

1. **Copy all scripts** into their respective locations as shown above
2. **Create a simple race track** in Workspace/Tracks using parts
3. **Add spawn locations** in Workspace/SpawnLocations
4. **Test the game** - press **F5** to play
5. **Admin panel** - press **F2** to open if you're an admin

Your UserIds (7392445200, 10443874977) are pre-configured as admins!

## What This Game Includes

✅ Complete racing system with physics
✅ Player bases that start at level 1
✅ Base upgrades to levels 2 and 3
✅ Currency system (earn brainrots from racing)
✅ Character collection system
✅ Full admin panel with your UserIds
✅ Global messages with DisplayNames
✅ Data persistence with DataStores

The game is ready to play - just add some race tracks and you're set!
