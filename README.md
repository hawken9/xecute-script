-- Full Xecute Script [Mobile-Compatible, Auto Farm, ESP, Teleport, Theme Toggle, etc.]

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local TweenSpeed = 250

local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "XecuteUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0.4, 0, 0.5, 0)
frame.Position = UDim2.new(0.3, 0, 0.25, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)

local tabBar = Instance.new("Frame", frame)
tabBar.Size = UDim2.new(1, 0, 0.1, 0)
tabBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local tabs, buttons, frames = {"Main", "Teleports", "Misc"}, {}, {}
for i, name in pairs(tabs) do
    local btn = Instance.new("TextButton", tabBar)
    btn.Size = UDim2.new(1 / #tabs, 0, 1, 0)
    btn.Position = UDim2.new((i - 1) / #tabs, 0, 0, 0)
    btn.Text = name
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    buttons[name] = btn

    local tab = Instance.new("Frame", frame)
    tab.Size = UDim2.new(1, 0, 0.9, 0)
    tab.Position = UDim2.new(0, 0, 0.1, 0)
    tab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    tab.Visible = (i == 1)
    frames[name] = tab

    btn.MouseButton1Click:Connect(function()
        for _, f in pairs(frames) do f.Visible = false end
        tab.Visible = true
    end)
end

-- Aura Effect
local function auraFX()
    local fx = Instance.new("ParticleEmitter", LocalPlayer.Character:WaitForChild("HumanoidRootPart"))
    fx.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0))
    }
    fx.Size = NumberSequence.new(2)
    fx.LightEmission = 0.7
    fx.Rate = 25
    fx.Lifetime = NumberRange.new(1)
    fx.Texture = "rbxassetid://48374994"
end
if LocalPlayer.Character then auraFX() end
LocalPlayer.CharacterAdded:Connect(function() task.wait(1) auraFX() end)

-- Closest Enemy
local function getClosestEnemy()
    local closest, dist = nil, math.huge
    for _, v in pairs(workspace.Enemies:GetChildren()) do
        if v:FindFirstChild("HumanoidRootPart") and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            local d = (LocalPlayer.Character.HumanoidRootPart.Position - v.HumanoidRootPart.Position).Magnitude
            if d < dist then
                dist = d
                closest = v
            end
        end
    end
    return closest
end

-- Tween Movement
local function tweenTo(pos)
    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if root then
        local dist = (root.Position - pos).Magnitude
        local t = TweenService:Create(root, TweenInfo.new(dist / TweenSpeed, Enum.EasingStyle.Linear), {CFrame = CFrame.new(pos)})
        t:Play()
    end
end

-- Auto Farm
local autoFarm = false
local farmBtn = Instance.new("TextButton", frames["Main"])
farmBtn.Size = UDim2.new(0.9, 0, 0.08, 0)
farmBtn.Position = UDim2.new(0.05, 0, 0.05, 0)
farmBtn.Text = "Auto Farm [OFF]"
farmBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
farmBtn.TextColor3 = Color3.new(1, 1, 1)
farmBtn.TextScaled = true
farmBtn.MouseButton1Click:Connect(function()
    autoFarm = not autoFarm
    farmBtn.Text = "Auto Farm [" .. (autoFarm and "ON" or "OFF") .. "]"
end)

task.spawn(function()
    while task.wait(0.1) do
        if autoFarm and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local enemy = getClosestEnemy()
            if enemy then
                tweenTo(enemy.HumanoidRootPart.Position + Vector3.new(0, 10, 0))
                task.wait(0.3)
                for i = 1, 5 do mouse1click() end
            end
        end
    end
end)

-- Fruit Notifier
workspace.DescendantAdded:Connect(function(desc)
    if desc:IsA("Tool") and desc.Name:lower():find("fruit") then
        tweenTo(desc.Position)
    end
end)

-- Teleports
local TPLocations = {
    ["Hydra Island"] = Vector3.new(5220, 50, -1200),
    ["Great Tree"] = Vector3.new(3850, 300, -700),
    ["Castle on the Sea"] = Vector3.new(-5050, 300, -3150),
    ["Floating Turtle"] = Vector3.new(-11500, 400, -7500),
    ["Haunted Castle"] = Vector3.new(-9500, 250, 6400)
}
local ypos = 0.05
for name, pos in pairs(TPLocations) do
    local b = Instance.new("TextButton", frames["Teleports"])
    b.Size = UDim2.new(0.9, 0, 0.08, 0)
    b.Position = UDim2.new(0.05, 0, ypos, 0)
    b.Text = name
    b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    b.TextColor3 = Color3.new(1,1,1)
    b.TextScaled = true
    b.MouseButton1Click:Connect(function() tweenTo(pos) end)
    ypos = ypos + 0.09
end

-- ESP Toggle
local esp = false
local espBtn = Instance.new("TextButton", frames["Misc"])
espBtn.Size = UDim2.new(0.9, 0, 0.08, 0)
espBtn.Position = UDim2.new(0.05, 0, 0.05, 0)
espBtn.Text = "Toggle ESP"
espBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
espBtn.TextColor3 = Color3.new(1,1,1)
espBtn.TextScaled = true
espBtn.MouseButton1Click:Connect(function()
    esp = not esp
    if esp then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local tag = Instance.new("BillboardGui", p.Character)
                tag.Size = UDim2.new(0, 100, 0, 40)
                tag.Adornee = p.Character.HumanoidRootPart
                tag.AlwaysOnTop = true
                local label = Instance.new("TextLabel", tag)
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.Text = p.Name
                label.TextColor3 = Color3.fromRGB(255, 0, 0)
                label.TextScaled = true
            end
        end
    end
end)

-- Tween Speed Selector
local speeds = {175, 200, 250, 300}
local current = 3
local dropdown = Instance.new("TextButton", frames["Misc"])
dropdown.Size = UDim2.new(0.9, 0, 0.08, 0)
dropdown.Position = UDim2.new(0.05, 0, 0.15, 0)
dropdown.Text = "Tween Speed: 250"
dropdown.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
dropdown.TextColor3 = Color3.new(1, 1, 1)
dropdown.TextScaled = true
dropdown.MouseButton1Click:Connect(function()
    current = current + 1
    if current > #speeds then current = 1 end
    TweenSpeed = speeds[current]
    dropdown.Text = "Tween Speed: " .. TweenSpeed
end)

-- Theme Toggle
local dark = true
local themeBtn = Instance.new("TextButton", frames["Misc"])
themeBtn.Size = UDim2.new(0.9, 0, 0.08, 0)
themeBtn.Position = UDim2.new(0.05, 0, 0.25, 0)
themeBtn.Text = "Toggle Theme"
themeBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
themeBtn.TextColor3 = Color3.new(1, 1, 1)
themeBtn.TextScaled = true
themeBtn.MouseButton1Click:Connect(function()
    dark = not dark
    frame.BackgroundColor3 = dark and Color3.fromRGB(20, 20, 20) or Color3.fromRGB(240, 240, 240)
end)

-- Keybind for UI (PC)
UIS.InputBegan:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.Semicolon then
        frame.Visible = not frame.Visible
    end
end)
