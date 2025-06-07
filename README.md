-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis
local ESPEnabled = false
local InfiniteJumpEnabled = false
local SpeedEnabled = false
local SpeedValue = 50
local FlyEnabled = false
local NoclipEnabled = false
local MenuVisible = true
local ESPObjects = {}

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Position = UDim2.new(0.3, 0, 0.3, 0)
Frame.Size = UDim2.new(0, 400, 0, 550)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.Text = "🥝 KIWI XP V1"
Title.TextSize = 26
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Title.TextColor3 = Color3.new(1,1,1)
Title.BorderSizePixel = 0
Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 12)

local function createButton(text, posY)
    local btn = Instance.new("TextButton", Frame)
    btn.Position = UDim2.new(0.1, 0, posY, 0)
    btn.Size = UDim2.new(0.8, 0, 0, 45)
    btn.Text = text
    btn.TextSize = 20
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    
    -- Animação de hover
    btn.MouseEnter:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(100,100,100)
    end)
    btn.MouseLeave:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
    end)
    
    return btn
end

local ESPButton = createButton("Toggle ESP [OFF]", 0.12)
local JumpButton = createButton("Toggle Infinite Jump [OFF]", 0.22)
local SpeedButton = createButton("Toggle Speed [OFF]", 0.32)
local NoclipButton = createButton("Toggle Noclip [OFF]", 0.42)
local FlyButton = createButton("Toggle Fly [OFF]", 0.52)

-- Speed Slider
local SpeedSliderLabel = Instance.new("TextLabel", Frame)
SpeedSliderLabel.Position = UDim2.new(0.1, 0, 0.62, 0)
SpeedSliderLabel.Size = UDim2.new(0.8, 0, 0, 20)
SpeedSliderLabel.Text = "Speed Value: " .. SpeedValue
SpeedSliderLabel.TextSize = 18
SpeedSliderLabel.TextColor3 = Color3.new(1,1,1)
SpeedSliderLabel.BackgroundTransparency = 1

local SpeedSlider = Instance.new("TextBox", Frame)
SpeedSlider.Position = UDim2.new(0.1, 0, 0.67, 0)
SpeedSlider.Size = UDim2.new(0.8, 0, 0, 40)
SpeedSlider.Text = tostring(SpeedValue)
SpeedSlider.TextSize = 20
SpeedSlider.TextColor3 = Color3.new(1,1,1)
SpeedSlider.BackgroundColor3 = Color3.fromRGB(50,50,50)
Instance.new("UICorner", SpeedSlider).CornerRadius = UDim.new(0, 8)

-- Dropdown de TP
local TPDropdown = createButton("Teleport to Player ▼", 0.78)

local PlayerDropdownFrame = Instance.new("ScrollingFrame", Frame)
PlayerDropdownFrame.Position = UDim2.new(0.1, 0, 0.88, 0)
PlayerDropdownFrame.Size = UDim2.new(0.8, 0, 0, 100)
PlayerDropdownFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
PlayerDropdownFrame.Visible = false
PlayerDropdownFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
PlayerDropdownFrame.ScrollBarThickness = 6
Instance.new("UICorner", PlayerDropdownFrame).CornerRadius = UDim.new(0, 8)

local function updatePlayerDropdown()
    PlayerDropdownFrame:ClearAllChildren()
    local yPos = 0
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local btn = Instance.new("TextButton", PlayerDropdownFrame)
            btn.Position = UDim2.new(0, 0, 0, yPos)
            btn.Size = UDim2.new(1, 0, 0, 30)
            btn.Text = player.Name
            btn.TextSize = 18
            btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
            btn.TextColor3 = Color3.new(1,1,1)
            btn.MouseButton1Click:Connect(function()
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame + Vector3.new(0,5,0)
                end
            end)
            yPos = yPos + 32
        end
    end
    PlayerDropdownFrame.CanvasSize = UDim2.new(0, 0, 0, yPos)
end

-- ESP System
local function createESP(player)
    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Color = Color3.new(1, 0, 0)
    box.Filled = false
    box.Visible = false

    ESPObjects[player] = box

    RunService.RenderStepped:Connect(function()
        if ESPEnabled and player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local root = player.Character.HumanoidRootPart
            local pos, onscreen = Camera:WorldToViewportPoint(root.Position)

            if onscreen then
                local scale = math.clamp(2000 / (root.Position - Camera.CFrame.Position).Magnitude, 2, 300)
                box.Size = Vector2.new(40 * scale / 100, 60 * scale / 100)
                box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
                box.Visible = true
            else
                box.Visible = false
            end
        else
            box.Visible = false
        end
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        createESP(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        createESP(player)
    end)
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
    if InfiniteJumpEnabled then
        local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Speed loop + Fly + Noclip
RunService.RenderStepped:Connect(function()
    if SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = SpeedValue
    elseif LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end

    if NoclipEnabled and LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end

    if FlyEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.Velocity = Vector3.new(0, 50, 0)
    end
end)

-- Botões
ESPButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    ESPButton.Text = "Toggle ESP [" .. (ESPEnabled and "ON" or "OFF") .. "]"
end)

JumpButton.MouseButton1Click:Connect(function()
    InfiniteJumpEnabled = not InfiniteJumpEnabled
    JumpButton.Text = "Toggle Infinite Jump [" .. (InfiniteJumpEnabled and "ON" or "OFF") .. "]"
end)

SpeedButton.MouseButton1Click:Connect(function()
    SpeedEnabled = not SpeedEnabled
    SpeedButton.Text = "Toggle Speed [" .. (SpeedEnabled and "ON" or "OFF") .. "]"
end)

NoclipButton.MouseButton1Click:Connect(function()
    NoclipEnabled = not NoclipEnabled
    NoclipButton.Text = "Toggle Noclip [" .. (NoclipEnabled and "ON" or "OFF") .. "]"
end)

FlyButton.MouseButton1Click:Connect(function()
    FlyEnabled = not FlyEnabled
    FlyButton.Text = "Toggle Fly [" .. (FlyEnabled and "ON" or "OFF") .. "]"
end)

SpeedSlider.FocusLost:Connect(function()
    local val = tonumber(SpeedSlider.Text)
    if val and val >= 16 and val <= 500 then
        SpeedValue = val
        SpeedSliderLabel.Text = "Speed Value: " .. SpeedValue
    else
        SpeedSlider.Text = tostring(SpeedValue)
    end
end)

TPDropdown.MouseButton1Click:Connect(function()
    PlayerDropdownFrame.Visible = not PlayerDropdownFrame.Visible
    updatePlayerDropdown()
end)

-- Toggle menu com tecla F
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.F then
        MenuVisible = not MenuVisible
        Frame.Visible = MenuVisible
    end
end)
