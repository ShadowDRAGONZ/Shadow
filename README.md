-- SERVICES
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local mouse = player:GetMouse()
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local TS = game:GetService("TweenService")

-- VARIABLES
local flying = false
local speed = 100
local holdingKeys = {}
local keyBinds = {
    ToggleFly = Enum.KeyCode.F,
    IncreaseSpeed = Enum.KeyCode.Equals,
    DecreaseSpeed = Enum.KeyCode.Minus,
    TeleportForward = Enum.KeyCode.T,
    ToggleMenu = Enum.KeyCode.M
}
local bodyVelocity, bodyGyro

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "FlySystem"

-- Notification
local notif = Instance.new("TextLabel", gui)
notif.Size = UDim2.new(0.6, 0, 0, 40)
notif.Position = UDim2.new(0.5, 0, 0.1, 0)
notif.AnchorPoint = Vector2.new(0.5, 0.5)
notif.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
notif.BackgroundTransparency = 0.2
notif.Text = "✅ Fly Script Loaded!"
notif.TextColor3 = Color3.new(1, 1, 1)
notif.Font = Enum.Font.GothamBold
notif.TextScaled = true
Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 8)

TS:Create(notif, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out, 0, false, 3), {
    TextTransparency = 1,
    BackgroundTransparency = 1
}):Play()
task.delay(4, function()
    notif:Destroy()
end)

-- MAIN MENU
local menu = Instance.new("Frame", gui)
menu.Size = UDim2.new(0, 340, 0, 370)
menu.Position = UDim2.new(0.5, -170, 1, 0)
menu.BackgroundColor3 = Color3.fromRGB(33, 33, 33)
menu.Visible = false
Instance.new("UICorner", menu).CornerRadius = UDim.new(0, 10)

local tabFrame = Instance.new("Frame", menu)
tabFrame.Size = UDim2.new(1, 0, 0, 40)
tabFrame.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
Instance.new("UICorner", tabFrame).CornerRadius = UDim.new(0, 10)

local tab1 = Instance.new("TextButton", tabFrame)
tab1.Text = "📜 Menu"
tab1.Size = UDim2.new(0.5, 0, 1, 0)
tab1.BackgroundTransparency = 1
tab1.Font = Enum.Font.GothamBold
tab1.TextColor3 = Color3.new(0, 0, 0)
tab1.TextScaled = true

local tab2 = Instance.new("TextButton", tabFrame)
tab2.Text = "🎮 Keybinds"
tab2.Size = UDim2.new(0.5, 0, 1, 0)
tab2.Position = UDim2.new(0.5, 0, 0, 0)
tab2.BackgroundTransparency = 1
tab2.Font = Enum.Font.GothamBold
tab2.TextColor3 = Color3.new(0, 0, 0)
tab2.TextScaled = true

local page1 = Instance.new("Frame", menu)
page1.Size = UDim2.new(1, 0, 1, -40)
page1.Position = UDim2.new(0, 0, 0, 40)
page1.BackgroundTransparency = 1

local instruction = Instance.new("TextLabel", page1)
instruction.Size = UDim2.new(1, -20, 0, 80)
instruction.Position = UDim2.new(0, 10, 0.05, 0)
instruction.BackgroundTransparency = 1
instruction.Text = "WASD để di chuyển\nQ/E lên xuống\n" ..
    "F: Bật/Tắt bay\n+/-: Tăng/Giảm tốc độ\nT: Teleport về hướng nhìn\nM: Menu"
instruction.TextColor3 = Color3.new(1, 1, 1)
instruction.TextScaled = true
instruction.TextWrapped = true
instruction.Font = Enum.Font.Gotham

local speedLabel = Instance.new("TextLabel", page1)
speedLabel.Size = UDim2.new(1, -20, 0, 40)
speedLabel.Position = UDim2.new(0, 10, 0.35, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Tốc độ hiện tại: " .. speed
speedLabel.TextColor3 = Color3.new(1, 1, 1)
speedLabel.TextScaled = true
speedLabel.Font = Enum.Font.Gotham

local plusBtn = Instance.new("TextButton", page1)
plusBtn.Text = "+"
plusBtn.Size = UDim2.new(0, 100, 0, 50)
plusBtn.Position = UDim2.new(0.1, 0, 0.6, 0)
plusBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
plusBtn.TextColor3 = Color3.new(1, 1, 1)
plusBtn.TextScaled = true
plusBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 8)

local minusBtn = Instance.new("TextButton", page1)
minusBtn.Text = "-"
minusBtn.Size = UDim2.new(0, 100, 0, 50)
minusBtn.Position = UDim2.new(0.6, 0, 0.6, 0)
minusBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
minusBtn.TextColor3 = Color3.new(1, 1, 1)
minusBtn.TextScaled = true
minusBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", minusBtn).CornerRadius = UDim.new(0, 8)

-- TAB 2 - KEYBIND SETTINGS
local page2 = page1:Clone()
page2.Parent = menu
page2.Visible = false
for _, child in ipairs(page2:GetChildren()) do child:Destroy() end

local yPos = 0.05
for name, key in pairs(keyBinds) do
    local label = Instance.new("TextLabel", page2)
    label.Text = "Phím cho " .. name .. ":"
    label.Size = UDim2.new(0.5, -10, 0, 30)
    label.Position = UDim2.new(0, 10, yPos, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.Gotham
    label.TextScaled = true

    local input = Instance.new("TextBox", page2)
    input.Text = key.Name
    input.Size = UDim2.new(0.4, 0, 0, 30)
    input.Position = UDim2.new(0.5, 0, yPos, 0)
    input.Font = Enum.Font.Gotham
    input.TextScaled = true
    input.ClearTextOnFocus = false
    input.TextColor3 = Color3.new(0, 0, 0)
    input.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Instance.new("UICorner", input).CornerRadius = UDim.new(0, 6)

    input.FocusLost:Connect(function()
        local newKey = Enum.KeyCode[input.Text]
        if newKey then
            keyBinds[name] = newKey
        else
            input.Text = keyBinds[name].Name
        end
    end)
    yPos += 0.1
end

-- TAB SWITCH
tab1.MouseButton1Click:Connect(function()
    page1.Visible = true
    page2.Visible = false
end)
tab2.MouseButton1Click:Connect(function()
    page1.Visible = false
    page2.Visible = true
end)

-- MENU DRAGGABLE
local dragging, offset
tabFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        offset = input.Position - menu.Position
    end
end)
tabFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        menu.Position = UDim2.new(0, input.Position.X - offset.X, 0, input.Position.Y - offset.Y)
    end
end)

-- FLY FUNCTION
local function startFlying()
    if flying then return end
    flying = true
    local root = character:WaitForChild("HumanoidRootPart")

    bodyVelocity = Instance.new("BodyVelocity", root)
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.zero

    bodyGyro = Instance.new("BodyGyro", root)
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.CFrame = root.CFrame
end

local function stopFlying()
    flying = false
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
end

-- TELEPORT
local function teleportForward()
    local root = character:FindFirstChild("HumanoidRootPart")
    if root then
        root.CFrame = CFrame.new(root.Position + root.CFrame.LookVector * 10)
    end
end

-- KEYBIND ACTIONS
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    local key = input.KeyCode
    holdingKeys[key] = true

    if key == keyBinds.ToggleFly then
        if flying then stopFlying() else startFlying() end
    elseif key == keyBinds.IncreaseSpeed then
        speed = math.clamp(speed + 10, 10, 1000)
    elseif key == keyBinds.DecreaseSpeed then
        speed = math.clamp(speed - 10, 10, 1000)
    elseif key == keyBinds.ToggleMenu then
        menu.Visible = not menu.Visible
        if menu.Visible then
            menu.Position = UDim2.new(0.5, -170, 1, 0)
            TS:Create(menu, TweenInfo.new(0.5), { Position = UDim2.new(0.5, -170, 0.5, -185) }):Play()
        else
            TS:Create(menu, TweenInfo.new(0.5), { Position = UDim2.new(0.5, -170, 1, 0) }):Play()
        end
    elseif key == keyBinds.TeleportForward then
        teleportForward()
    end
end)

UIS.InputEnded:Connect(function(input)
    holdingKeys[input.KeyCode] = false
end)

-- MOVEMENT
RS.Heartbeat:Connect(function()
    if flying then
        local root = character:FindFirstChild("HumanoidRootPart")
        if not root then return end

        local move = Vector3.zero
        if holdingKeys[Enum.KeyCode.W] then move += root.CFrame.LookVector * speed end
        if holdingKeys[Enum.KeyCode.S] then move -= root.CFrame.LookVector * speed end
        if holdingKeys[Enum.KeyCode.A] then move -= root.CFrame.RightVector * speed end
        if holdingKeys[Enum.KeyCode.D] then move += root.CFrame.RightVector * speed end
        if holdingKeys[Enum.KeyCode.Q] then move += Vector3.new(0, speed, 0) end
        if holdingKeys[Enum.KeyCode.E] then move -= Vector3.new(0, speed, 0) end

        bodyVelocity.Velocity = move
        bodyGyro.CFrame = root.CFrame
    end

    speedLabel.Text = "Tốc độ hiện tại: " .. tostring(speed)
end)

-- RESET ON DEATH
humanoid.Died:Connect(stopFlying)
