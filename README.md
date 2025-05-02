--// SERVICES
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--// REQUIRE ABILITYCONTROLLER
local success, AbilityController = pcall(function()
	return require(ReplicatedStorage:WaitForChild("Controllers"):WaitForChild("AbilityController"))
end)

--// VARIABLES
local styleName = "NEL Bachira"
local cooldownEnabled = false
local awakeningEnabled = false
local lastUsed = 0
local originalAbilityCooldown = success and AbilityController.AbilityCooldown
local v179 = "Prodigy"  -- Giá trị Flow mặc định

--// MAIN GUI
local screenGui = Instance.new("ScreenGui", PlayerGui)
screenGui.Name = "UnifiedMenu"
screenGui.ResetOnSpawn = false

-- MAIN FRAME
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 480, 0, 500)
mainFrame.Position = UDim2.new(0.5, -240, 0.4, -250)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Active = true
mainFrame.Draggable = true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "🎮 Control Menu"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

-- RAINBOW TOGGLE BUTTON
local rainbowButton = Instance.new("TextButton", screenGui)
rainbowButton.Size = UDim2.new(0, 40, 0, 40)
rainbowButton.Position = UDim2.new(0, 20, 0.5, -20)
rainbowButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
rainbowButton.Text = "☰"
rainbowButton.TextColor3 = Color3.new(1, 1, 1)
rainbowButton.Font = Enum.Font.SourceSansBold
rainbowButton.TextSize = 24
rainbowButton.Draggable = true
rainbowButton.Active = true

-- FLOW PANEL
local flowFrame = Instance.new("Frame", mainFrame)
flowFrame.Size = UDim2.new(1, -20, 0, 60)
flowFrame.Position = UDim2.new(0, 10, 0, 40)
flowFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
flowFrame.BorderSizePixel = 0

local flowLabel = Instance.new("TextLabel", flowFrame)
flowLabel.Size = UDim2.new(1, 0, 0, 20)
flowLabel.BackgroundTransparency = 1
flowLabel.Text = "Current Flow: " .. v179
flowLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
flowLabel.Font = Enum.Font.SourceSans
flowLabel.TextSize = 14

local flowInputBox = Instance.new("TextBox", flowFrame)
flowInputBox.Size = UDim2.new(1, -6, 0, 30)
flowInputBox.Position = UDim2.new(0, 3, 0, 30)
flowInputBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
flowInputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
flowInputBox.Font = Enum.Font.SourceSans
flowInputBox.TextSize = 18
flowInputBox.PlaceholderText = "Nhập Flow"
flowInputBox.Text = v179

flowInputBox.FocusLost:Connect(function(enterPressed)
	if enterPressed and flowInputBox.Text ~= "" then
		v179 = flowInputBox.Text
		flowLabel.Text = "Current Flow: " .. v179
		local playerStats = LocalPlayer:FindFirstChild("PlayerStats")
		if playerStats and playerStats:FindFirstChild("Flow") then
			playerStats.Flow.Value = v179
		end
	end
end)

-- STYLE PANEL
local styleFrame = Instance.new("Frame", mainFrame)
styleFrame.Size = UDim2.new(1, -20, 0, 150)
styleFrame.Position = UDim2.new(0, 10, 0, 100)
styleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
styleFrame.BorderSizePixel = 0

local styleLabel = Instance.new("TextLabel", styleFrame)
styleLabel.Size = UDim2.new(1, 0, 0, 20)
styleLabel.Position = UDim2.new(0, 0, 1, -20)
styleLabel.BackgroundTransparency = 1
styleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
styleLabel.Font = Enum.Font.SourceSans
styleLabel.TextSize = 14
styleLabel.Text = "Đang áp dụng Style..."

local scroll = Instance.new("ScrollingFrame", styleFrame)
scroll.Size = UDim2.new(1, 0, 1, -20)
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.ScrollBarThickness = 4
scroll.BackgroundTransparency = 1

-- TEXTBOX NHẬP TAY
local styleBox = Instance.new("TextBox", scroll)
styleBox.Size = UDim2.new(1, -6, 0, 30)
styleBox.Position = UDim2.new(0, 3, 0, 0)
styleBox.PlaceholderText = "Nhập Style (VD: Blue Lock Neo)"
styleBox.Text = ""
styleBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
styleBox.TextColor3 = Color3.new(1, 1, 1)
styleBox.ClearTextOnFocus = false
styleBox.Font = Enum.Font.SourceSans
styleBox.TextSize = 14

styleBox.FocusLost:Connect(function(enterPressed)
	if enterPressed and styleBox.Text ~= "" then
		styleName = styleBox.Text
		styleLabel.Text = "Đã đổi Style thành: " .. styleName
	end
end)

local styles = {"NEL Isagi", "Kaiser", "Sae", "Don Lorenzo", "Kunigami", "Kurona"}
for i, name in ipairs(styles) do
	local btn = Instance.new("TextButton", scroll)
	btn.Size = UDim2.new(1, -6, 0, 30)
	btn.Position = UDim2.new(0, 3, 0, i * 32)
	btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSans
	btn.TextSize = 14
	btn.Text = name
	btn.MouseButton1Click:Connect(function()
		styleName = name
		styleLabel.Text = "Đã đổi Style thành: " .. name
	end)
end

scroll.CanvasSize = UDim2.new(0, 0, 0, (#styles + 1) * 32)

-- CẬP NHẬT STYLE LIÊN TỤC
task.spawn(function()
	while true do
		task.wait(1)
		local stats = LocalPlayer:FindFirstChild("PlayerStats")
		if stats and stats:FindFirstChild("Style") then
			stats.Style.Value = styleName
		end
	end
end)

-- COOLDOWN BUTTON
local cooldownButton = Instance.new("TextButton", mainFrame)
cooldownButton.Size = UDim2.new(1, -20, 0, 40)
cooldownButton.Position = UDim2.new(0, 10, 0, 280)
cooldownButton.Text = "Bật AbilityCooldown"
cooldownButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
cooldownButton.TextColor3 = Color3.new(1, 1, 1)
cooldownButton.Font = Enum.Font.SourceSansBold
cooldownButton.TextSize = 16

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
local awakeningButton = Instance.new("TextButton", mainFrame)
awakeningButton.Size = UDim2.new(1, -20, 0, 40)
awakeningButton.Position = UDim2.new(0, 10, 0, 330)
awakeningButton.Text = "Awakening: OFF"
awakeningButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
awakeningButton.TextColor3 = Color3.new(1, 1, 1)
awakeningButton.Font = Enum.Font.SourceSansBold
awakeningButton.TextSize = 16

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

-- RAINBOW EFFECT & TOGGLE
local hue = 0
RunService.RenderStepped:Connect(function()
	hue = (hue + 0.005) % 1
	local color = Color3.fromHSV(hue, 1, 1)
	rainbowButton.BackgroundColor3 = color
end)

rainbowButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)
