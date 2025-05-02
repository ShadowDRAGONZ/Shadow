-- Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Require AbilityController
local success, AbilityController = pcall(function()
    return require(ReplicatedStorage:WaitForChild("Controllers"):WaitForChild("AbilityController"))
end)

-- Variables
local styleName = "NEL Bachira"
local cooldownEnabled = false
local awakeningEnabled = false
local lastUsed = 0
local originalAbilityCooldown = success and AbilityController.AbilityCooldown

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UnifiedMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 320, 0, 220)
mainFrame.Position = UDim2.new(0.5, -160, 0.4, -110)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "🎮 Control Menu"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.Parent = mainFrame

-- STYLE SECTION
local styleBox = Instance.new("TextBox")
styleBox.Size = UDim2.new(1, -20, 0, 40)
styleBox.Position = UDim2.new(0, 10, 0, 40)
styleBox.PlaceholderText = "Nhập Style (VD: NEL Bachira)"
styleBox.Text = ""
styleBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
styleBox.TextColor3 = Color3.fromRGB(255, 255, 255)
styleBox.ClearTextOnFocus = false
styleBox.Parent = mainFrame

local styleLabel = Instance.new("TextLabel")
styleLabel.Size = UDim2.new(1, -20, 0, 20)
styleLabel.Position = UDim2.new(0, 10, 0, 85)
styleLabel.BackgroundTransparency = 1
styleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
styleLabel.Font = Enum.Font.SourceSans
styleLabel.TextSize = 14
styleLabel.Text = "Đang áp dụng Style..."
styleLabel.Parent = mainFrame

styleBox.FocusLost:Connect(function(enterPressed)
    if enterPressed and styleBox.Text ~= "" then
        styleName = styleBox.Text
        styleLabel.Text = "Đã đổi Style thành: " .. styleName
    end
end)

spawn(function()
    while true do
        task.wait(1)
        local stats = LocalPlayer:FindFirstChild("PlayerStats")
        if stats and stats:FindFirstChild("Style") then
            stats.Style.Value = styleName
        end
    end
end)

-- COOLDOWN BUTTON
local cooldownButton = Instance.new("TextButton")
cooldownButton.Size = UDim2.new(1, -20, 0, 40)
cooldownButton.Position = UDim2.new(0, 10, 0, 115)
cooldownButton.Text = "Bật AbilityCooldown"
cooldownButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
cooldownButton.TextColor3 = Color3.new(1, 1, 1)
cooldownButton.Font = Enum.Font.SourceSansBold
cooldownButton.TextSize = 16
cooldownButton.Parent = mainFrame

cooldownButton.MouseButton1Click:Connect(function()
    if not success then return end
    if cooldownEnabled then
        AbilityController.AbilityCooldown = originalAbilityCooldown
        cooldownButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        cooldownButton.Text = "Bật AbilityCooldown"
    else
        AbilityController.AbilityCooldown = function(s, n, ...) return originalAbilityCooldown(s, n, 0, ...) end
        cooldownButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        cooldownButton.Text = "Tắt AbilityCooldown"
    end
    cooldownEnabled = not cooldownEnabled
end)

-- AWAKENING BUTTON
local awakeningButton = Instance.new("TextButton")
awakeningButton.Size = UDim2.new(1, -20, 0, 40)
awakeningButton.Position = UDim2.new(0, 10, 0, 165)
awakeningButton.Text = "Awakening: OFF"
awakeningButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
awakeningButton.TextColor3 = Color3.new(1, 1, 1)
awakeningButton.Font = Enum.Font.SourceSansBold
awakeningButton.TextSize = 16
awakeningButton.Parent = mainFrame

awakeningButton.MouseButton1Click:Connect(function()
    local stats = LocalPlayer:FindFirstChild("PlayerStats")
    if not stats or not stats:FindFirstChild("InAwakening") then return end

    awakeningEnabled = not awakeningEnabled
    if awakeningEnabled then
        awakeningButton.Text = "Awakening: ON"
        awakeningButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    else
        stats.InAwakening.Value = false
        awakeningButton.Text = "Awakening: OFF"
        awakeningButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    end
end)

RunService.RenderStepped:Connect(function()
    if awakeningEnabled and tick() - lastUsed >= 1 then
        local stats = LocalPlayer:FindFirstChild("PlayerStats")
        if stats and stats:FindFirstChild("InAwakening") and success and AbilityController then
            stats.InAwakening.Value = true
            AbilityController:UseAbility("Awakening")
            lastUsed = tick()
        end
    end
end)

