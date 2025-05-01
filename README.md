local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Tạo GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AwakeningToggleGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

-- Tạo khung chính có thể kéo
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 200, 0, 80)
mainFrame.Position = UDim2.new(0, 20, 0, 100)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true -- Cho phép kéo
mainFrame.Parent = screenGui

-- Tạo nút bật/tắt
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(1, -20, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 20)
toggleButton.Text = "Awakening: OFF"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 20
toggleButton.Parent = mainFrame

-- Tiêu đề
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 20)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "⚡ Awakening Control"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.SourceSans
title.TextSize = 16
title.Parent = mainFrame

-- Biến kiểm soát trạng thái
local awakeningEnabled = false
local lastUsed = 0

-- Load AbilityController
local success, AbilityController = pcall(function()
	return require(ReplicatedStorage:WaitForChild("Controllers"):WaitForChild("AbilityController"))
end)

-- Loop sử dụng Awakening khi bật
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

-- Sự kiện nút toggle
toggleButton.MouseButton1Click:Connect(function()
	local stats = LocalPlayer:FindFirstChild("PlayerStats")
	if not stats or not stats:FindFirstChild("InAwakening") then return end

	awakeningEnabled = not awakeningEnabled

	if awakeningEnabled then
		toggleButton.Text = "Awakening: ON"
		toggleButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
	else
		stats.InAwakening.Value = false
		toggleButton.Text = "Awakening: OFF"
		toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
	end
end)
