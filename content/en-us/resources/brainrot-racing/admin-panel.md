---
title: Admin Panel
comments:
description: Implement a comprehensive admin panel with moderation and gameplay control commands for Brainrot Racing.
prev: /resources/brainrot-racing/track-design
---

The admin panel provides game administrators and moderators with powerful tools to manage players, control game state, and enhance the experience. This guide covers implementing a secure, user-friendly admin interface with essential commands.

## Installation in Roblox Studio

To install the admin panel system in your game:

### Option 1: Manual Setup

1. **Create Folder Structure** in your place:

   ```
   ServerScriptService/
   ├── AdminService (ModuleScript)
   ├── AdminCommands/ (Folder)
   │   ├── GlobalMessageCommand (ModuleScript)
   │   ├── BanCommand (ModuleScript)
   │   ├── KickCommand (ModuleScript)
   │   ├── SetStatsCommand (ModuleScript)
   │   ├── MusicCommand (ModuleScript)
   │   ├── SpawnBrainrotCommand (ModuleScript)
   │   └── StealBrainrotCommand (ModuleScript)

   ReplicatedStorage/
   ├── Events/ (Folder)
   │   └── ShowGlobalMessage (RemoteEvent)

   StarterGui/
   └── AdminPanelUI (ScreenGui with LocalScript)
   ```

2. **Copy the code** from each section of this documentation into the corresponding scripts.

3. **Configure Admin UserIds** - The admin panel is pre-configured for UserIds:
   - `7392445200`
   - `10443874977`

   These users will have full Owner permissions automatically.

### Option 2: Download Pre-Made Model

Download the complete admin panel from the Roblox library:

1. Open Roblox Studio and your place file.
2. Navigate to the **Toolbox** (View > Toolbox).
3. Search for "Brainrot Racing Admin Panel" in Models.
4. Click to insert it into your game.
5. The model will automatically place scripts in the correct locations.

### Option 3: Import from File

If you have the admin panel as a `.rbxm` file:

1. In Roblox Studio, right-click on `Workspace`.
2. Select **Insert from File**.
3. Choose the `AdminPanel.rbxm` file.
4. Move the scripts to their correct locations as shown in the folder structure above.

### Verification

After installation, verify the setup:

1. **Test in Studio**: Click Play and press **F9** to open the Developer Console.
2. Check for any error messages related to admin scripts.
3. **Test Admin Access**: Join with one of the configured admin UserIds.
4. Press **;** (semicolon) or the configured hotkey to open the admin panel.
5. Test a command like Global Message to verify functionality.

## Admin Panel Overview

The admin panel includes:

- **Global Messaging** — Broadcast announcements to all players.
- **Player Moderation** — Ban, kick, and manage player access.
- **Game State Control** — Modify player stats (speed, money, stamina).
- **Music Control** — Play background music for all players.
- **Brainrot Management** — Spawn characters and transfer ownership.

## Admin Permission System

Implement role-based permissions to control admin access:

```lua
-- AdminService in ServerScriptService
local AdminService = {}
local Players = game:GetService("Players")

-- Define admin roles and permissions
local AdminRoles = {
    Owner = {
        Level = 100,
        Permissions = {"all"},
        UserIds = {7392445200, 10443874977},  -- Authorized admin UserIds
    },

    HeadAdmin = {
        Level = 75,
        Permissions = {
            "global_message", "ban", "kick", "mute",
            "set_speed", "set_money", "set_stamina",
            "play_music", "spawn_brainrot", "steal_brainrot",
        },
        UserIds = {},
    },

    Moderator = {
        Level = 50,
        Permissions = {
            "global_message", "kick", "mute",
            "spawn_brainrot",
        },
        UserIds = {},
    },
}

function AdminService.GetPlayerRole(player)
    for roleName, roleData in pairs(AdminRoles) do
        if table.find(roleData.UserIds, player.UserId) then
            return roleName, roleData
        end
    end
    return nil, nil
end

function AdminService.HasPermission(player, permission)
    local roleName, roleData = AdminService.GetPlayerRole(player)

    if not roleData then
        return false
    end

    -- Owner has all permissions
    if table.find(roleData.Permissions, "all") then
        return true
    end

    return table.find(roleData.Permissions, permission) ~= nil
end

return AdminService
```

## Admin Panel UI

Create a clean, accessible admin interface:

```lua
-- AdminPanelUI in StarterGui
local AdminPanel = {}
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer

function AdminPanel.CreateUI()
    -- Main admin panel frame
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AdminPanel"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- Main container
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 600, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui

    -- Add rounded corners
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame

    -- Title bar
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 50)
    titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame

    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(1, -20, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "🛡️ ADMIN PANEL"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar

    -- Close button
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0, 40, 0, 40)
    closeButton.Position = UDim2.new(1, -45, 0, 5)
    closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    closeButton.Text = "✕"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 20
    closeButton.Font = Enum.Font.GothamBold
    closeButton.Parent = titleBar

    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeButton

    closeButton.MouseButton1Click:Connect(function()
        screenGui.Enabled = false
    end)

    -- Commands container
    local commandsFrame = Instance.new("ScrollingFrame")
    commandsFrame.Name = "CommandsFrame"
    commandsFrame.Size = UDim2.new(1, -20, 1, -70)
    commandsFrame.Position = UDim2.new(0, 10, 0, 60)
    commandsFrame.BackgroundTransparency = 1
    commandsFrame.BorderSizePixel = 0
    commandsFrame.ScrollBarThickness = 6
    commandsFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    commandsFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    commandsFrame.Parent = mainFrame

    -- Layout for commands
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 10)
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Parent = commandsFrame

    screenGui.Parent = player.PlayerGui

    return screenGui, commandsFrame
end

return AdminPanel
```

## Global Message Command

Broadcast messages to all players with customizable styling:

```lua
-- GlobalMessageCommand
local GlobalMessage = {}
local Players = game:GetService("Players")

function GlobalMessage.Execute(adminPlayer, message, duration)
    duration = duration or 5

    -- Broadcast to all players
    for _, player in ipairs(Players:GetPlayers()) do
        GlobalMessage.ShowMessageToPlayer(player, message, duration, adminPlayer.DisplayName)
    end
end

function GlobalMessage.ShowMessageToPlayer(player, message, duration, senderDisplayName)
    -- Fire client event to show message
    local remoteEvent = game.ReplicatedStorage.Events.ShowGlobalMessage
    remoteEvent:FireClient(player, {
        Message = message,
        Duration = duration,
        Sender = senderDisplayName,
        Timestamp = os.date("%H:%M:%S"),
    })
end

-- Client-side display (in LocalScript)
function GlobalMessage.CreateMessageUI(messageData)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "GlobalMessage"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- Message frame
    local messageFrame = Instance.new("Frame")
    messageFrame.Name = "MessageFrame"
    messageFrame.Size = UDim2.new(0, 500, 0, 100)
    messageFrame.Position = UDim2.new(0.5, -250, 0, -120)
    messageFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    messageFrame.BorderSizePixel = 2
    messageFrame.BorderColor3 = Color3.fromRGB(255, 200, 50)
    messageFrame.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = messageFrame

    -- Sender label
    local senderLabel = Instance.new("TextLabel")
    senderLabel.Name = "Sender"
    senderLabel.Size = UDim2.new(1, -20, 0, 25)
    senderLabel.Position = UDim2.new(0, 10, 0, 5)
    senderLabel.BackgroundTransparency = 1
    senderLabel.Text = "📢 " .. messageData.Sender .. " • " .. messageData.Timestamp
    senderLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
    senderLabel.TextSize = 16
    senderLabel.Font = Enum.Font.GothamBold
    senderLabel.TextXAlignment = Enum.TextXAlignment.Left
    senderLabel.Parent = messageFrame

    -- Message text
    local messageLabel = Instance.new("TextLabel")
    messageLabel.Name = "Message"
    messageLabel.Size = UDim2.new(1, -20, 1, -35)
    messageLabel.Position = UDim2.new(0, 10, 0, 30)
    messageLabel.BackgroundTransparency = 1
    messageLabel.Text = messageData.Message
    messageLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    messageLabel.TextSize = 20
    messageLabel.Font = Enum.Font.Gotham
    messageLabel.TextWrapped = true
    messageLabel.TextYAlignment = Enum.TextYAlignment.Top
    messageLabel.Parent = messageFrame

    screenGui.Parent = game.Players.LocalPlayer.PlayerGui

    -- Animate in
    messageFrame:TweenPosition(
        UDim2.new(0.5, -250, 0, 20),
        Enum.EasingDirection.Out,
        Enum.EasingStyle.Back,
        0.5,
        true
    )

    -- Auto-dismiss after duration
    task.wait(messageData.Duration)

    messageFrame:TweenPosition(
        UDim2.new(0.5, -250, 0, -120),
        Enum.EasingDirection.In,
        Enum.EasingStyle.Back,
        0.3,
        true
    )

    task.wait(0.3)
    screenGui:Destroy()
end

return GlobalMessage
```

## Player Moderation Commands

### Ban Command

```lua
-- BanCommand in ServerScriptService
local BanCommand = {}
local DataStoreService = game:GetService("DataStoreService")
local banDataStore = DataStoreService:GetDataStore("BannedPlayers_v1")

function BanCommand.Execute(adminPlayer, targetPlayer, reason, duration)
    -- Validate permission
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "ban") then
        return false, "Insufficient permissions"
    end

    reason = reason or "No reason provided"
    duration = duration or 0  -- 0 = permanent

    local banData = {
        BannedBy = adminPlayer.Name,
        BannedById = adminPlayer.UserId,
        Reason = reason,
        Timestamp = os.time(),
        Duration = duration,
        ExpiresAt = duration > 0 and (os.time() + duration) or 0,
    }

    -- Save ban to DataStore
    local success, error = pcall(function()
        banDataStore:SetAsync(tostring(targetPlayer.UserId), banData)
    end)

    if success then
        -- Kick player immediately
        targetPlayer:Kick("You have been banned.\nReason: " .. reason)

        -- Log ban
        print(string.format("[BAN] %s banned %s for: %s",
            adminPlayer.Name, targetPlayer.Name, reason))

        return true, "Player banned successfully"
    else
        return false, "Failed to save ban: " .. tostring(error)
    end
end

function BanCommand.CheckBan(player)
    local success, banData = pcall(function()
        return banDataStore:GetAsync(tostring(player.UserId))
    end)

    if not success or not banData then
        return false
    end

    -- Check if ban expired
    if banData.ExpiresAt > 0 and os.time() > banData.ExpiresAt then
        -- Remove expired ban
        BanCommand.Unban(player.UserId)
        return false
    end

    return true, banData
end

return BanCommand
```

### Kick Command

```lua
-- KickCommand
local KickCommand = {}

function KickCommand.Execute(adminPlayer, targetPlayer, reason)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "kick") then
        return false, "Insufficient permissions"
    end

    reason = reason or "Kicked by admin"

    targetPlayer:Kick("You have been kicked.\nReason: " .. reason)

    print(string.format("[KICK] %s kicked %s for: %s",
        adminPlayer.Name, targetPlayer.Name, reason))

    return true, "Player kicked successfully"
end

return KickCommand
```

## Gameplay Control Commands

### Set Player Stats

```lua
-- SetStatsCommand
local SetStatsCommand = {}

function SetStatsCommand.SetSpeed(adminPlayer, targetPlayer, speed)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "set_speed") then
        return false, "Insufficient permissions"
    end

    speed = math.clamp(speed, 0, 200)

    local character = targetPlayer.Character
    if character then
        local stats = character:GetAttribute("Stats") or {}
        stats.Speed = speed
        character:SetAttribute("Stats", stats)

        print(string.format("[ADMIN] %s set %s's speed to %d",
            adminPlayer.Name, targetPlayer.Name, speed))

        return true, "Speed updated"
    end

    return false, "Character not found"
end

function SetStatsCommand.SetMoney(adminPlayer, targetPlayer, amount)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "set_money") then
        return false, "Insufficient permissions"
    end

    local dataService = require(game.ServerScriptService.DataService)
    local playerData = dataService.GetPlayerData(targetPlayer)

    playerData.Currency = math.max(0, amount)
    dataService.SavePlayerData(targetPlayer, playerData)

    -- Update UI
    game.ReplicatedStorage.Events.UpdateCurrency:FireClient(targetPlayer, playerData.Currency)

    print(string.format("[ADMIN] %s set %s's money to %d",
        adminPlayer.Name, targetPlayer.Name, amount))

    return true, "Money updated"
end

function SetStatsCommand.SetStamina(adminPlayer, targetPlayer, stamina)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "set_stamina") then
        return false, "Insufficient permissions"
    end

    stamina = math.clamp(stamina, 0, 100)

    local character = targetPlayer.Character
    if character and character:FindFirstChild("Stamina") then
        character.Stamina.Value = stamina

        print(string.format("[ADMIN] %s set %s's stamina to %d",
            adminPlayer.Name, targetPlayer.Name, stamina))

        return true, "Stamina updated"
    end

    return false, "Character not found"
end

return SetStatsCommand
```

## Music Control

```lua
-- MusicCommand
local MusicCommand = {}
local SoundService = game:GetService("SoundService")

function MusicCommand.PlayMusic(adminPlayer, assetId)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "play_music") then
        return false, "Insufficient permissions"
    end

    -- Stop current music
    for _, sound in ipairs(SoundService:GetChildren()) do
        if sound:IsA("Sound") and sound.Name == "AdminMusic" then
            sound:Stop()
            sound:Destroy()
        end
    end

    -- Create new sound
    local sound = Instance.new("Sound")
    sound.Name = "AdminMusic"
    sound.SoundId = "rbxassetid://" .. assetId
    sound.Volume = 0.5
    sound.Looped = true
    sound.Parent = SoundService

    sound:Play()

    print(string.format("[ADMIN] %s is now playing music: %s",
        adminPlayer.Name, assetId))

    return true, "Music playing"
end

function MusicCommand.StopMusic(adminPlayer)
    for _, sound in ipairs(SoundService:GetChildren()) do
        if sound:IsA("Sound") and sound.Name == "AdminMusic" then
            sound:Stop()
            sound:Destroy()
        end
    end

    return true, "Music stopped"
end

return MusicCommand
```

## Brainrot Management

### Spawn Brainrot

```lua
-- SpawnBrainrotCommand
local SpawnBrainrot = {}
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local InsertService = game:GetService("InsertService")

function SpawnBrainrot.Execute(adminPlayer, characterName, position)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "spawn_brainrot") then
        return false, "Insufficient permissions"
    end

    -- Check if character exists in game
    local character = ReplicatedStorage.Characters:FindFirstChild(characterName)

    if character then
        -- Spawn from game assets
        local newCharacter = character:Clone()
        newCharacter:PivotTo(CFrame.new(position))
        newCharacter.Parent = workspace

        print(string.format("[ADMIN] %s spawned %s",
            adminPlayer.Name, characterName))

        return true, "Brainrot spawned"
    else
        return false, "Character not found in game"
    end
end

function SpawnBrainrot.SpawnFromCatalog(adminPlayer, assetId, position)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "spawn_brainrot") then
        return false, "Insufficient permissions"
    end

    -- Attempt to load from Roblox catalog
    local success, model = pcall(function()
        return InsertService:LoadAsset(assetId)
    end)

    if success and model then
        local character = model:GetChildren()[1]
        if character then
            character:PivotTo(CFrame.new(position))
            character.Parent = workspace
            model:Destroy()

            print(string.format("[ADMIN] %s spawned catalog asset %s",
                adminPlayer.Name, assetId))

            return true, "Catalog brainrot spawned"
        end
    end

    return false, "Failed to load from catalog"
end

return SpawnBrainrot
```

### Steal Brainrot

```lua
-- StealBrainrotCommand
local StealBrainrot = {}

function StealBrainrot.Execute(adminPlayer, targetPlayer)
    local adminService = require(script.Parent.AdminService)
    if not adminService.HasPermission(adminPlayer, "steal_brainrot") then
        return false, "Insufficient permissions"
    end

    local dataService = require(game.ServerScriptService.DataService)

    -- Get both players' data
    local targetData = dataService.GetPlayerData(targetPlayer)
    local adminData = dataService.GetPlayerData(adminPlayer)

    -- Get target's current character
    local currentCharacter = targetPlayer:GetAttribute("SelectedCharacter")

    if not currentCharacter then
        return false, "Target has no character selected"
    end

    -- Transfer character
    if targetData.UnlockedCharacters[currentCharacter] then
        -- Add to admin
        adminData.UnlockedCharacters[currentCharacter] = true

        -- Remove from target (keep at least basic character)
        if currentCharacter ~= "BasicBrainrot" then
            targetData.UnlockedCharacters[currentCharacter] = nil
            targetPlayer:SetAttribute("SelectedCharacter", "BasicBrainrot")
        end

        -- Save data
        dataService.SavePlayerData(adminPlayer, adminData)
        dataService.SavePlayerData(targetPlayer, targetData)

        print(string.format("[ADMIN] %s stole %s from %s",
            adminPlayer.Name, currentCharacter, targetPlayer.Name))

        return true, "Brainrot stolen: " .. currentCharacter
    end

    return false, "Unable to steal character"
end

return StealBrainrot
```

## Admin Command Interface

Create command buttons in the UI:

```lua
-- CommandButton creator
function CreateCommandButton(parent, commandName, icon, callback)
    local button = Instance.new("TextButton")
    button.Name = commandName
    button.Size = UDim2.new(1, 0, 0, 45)
    button.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
    button.Text = ""
    button.Parent = parent

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button

    -- Icon and label
    local iconLabel = Instance.new("TextLabel")
    iconLabel.Size = UDim2.new(0, 40, 1, 0)
    iconLabel.BackgroundTransparency = 1
    iconLabel.Text = icon
    iconLabel.TextSize = 24
    iconLabel.Parent = button

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -50, 1, 0)
    nameLabel.Position = UDim2.new(0, 45, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = commandName
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 18
    nameLabel.Font = Enum.Font.GothamSemibold
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = button

    button.MouseButton1Click:Connect(callback)

    return button
end
```

## Security Best Practices

- Store admin user IDs server-side, never in client scripts.
- Validate all admin commands on the server.
- Log all admin actions for audit trails.
- Implement rate limiting to prevent command spam.
- Use RemoteEvents with proper security checks.
- Never trust client-provided player references for moderation.

## Conclusion

This admin panel provides comprehensive game management tools while maintaining security through role-based permissions. Customize the UI styling and commands to match your game's specific needs.
