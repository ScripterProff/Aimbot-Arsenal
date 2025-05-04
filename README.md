-- Vision Hub - Arsenal
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer
local Mouse = LP:GetMouse()
local UIS = game:GetService("UserInputService")

local ScreenGui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
ScreenGui.Name = "VisionHub"

local Settings = {
    Aimbot = false,
    ESP = false,
    FOV = 100,
    AimPart = "Head",
    TeamCheck = true,
    GameMode = "Arsenal"
}

-- UI
local main = Instance.new("Frame", ScreenGui)
main.Size = UDim2.new(0, 250, 0, 320)
main.Position = UDim2.new(0.05, 0, 0.1, 0)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
main.BorderSizePixel = 0
main.Visible = true
main.Active = true
main.Draggable = true

local title = Instance.new("TextLabel", main)
title.Text = "Vision Hub - Arsenal"
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local function createButton(text, y, callback)
    local button = Instance.new("TextButton", main)
    button.Size = UDim2.new(0.9, 0, 0, 30)
    button.Position = UDim2.new(0.05, 0, 0, y)
    button.Text = text
    button.TextColor3 = Color3.new(1, 1, 1)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.Font = Enum.Font.Gotham
    button.TextSize = 14
    button.MouseButton1Click:Connect(callback)
    return button
end

createButton("Toggle Aimbot", 40, function()
    Settings.Aimbot = not Settings.Aimbot
end)

createButton("Toggle ESP", 80, function()
    Settings.ESP = not Settings.ESP
end)

local minimize = createButton("Minimizar", 270, function()
    main.Visible = false
end)

local openBtn = Instance.new("TextButton", ScreenGui)
openBtn.Size = UDim2.new(0, 100, 0, 30)
openBtn.Position = UDim2.new(0.05, 0, 0.05, 0)
openBtn.Text = "Abrir Hub"
openBtn.TextColor3 = Color3.new(1, 1, 1)
openBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
openBtn.Font = Enum.Font.Gotham
openBtn.TextSize = 14
openBtn.Visible = false
openBtn.MouseButton1Click:Connect(function()
    main.Visible = true
    openBtn.Visible = false
end)

minimize.MouseButton1Click:Connect(function()
    openBtn.Visible = true
end)

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = true
fovCircle.Radius = Settings.FOV
fovCircle.Thickness = 1.5
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Filled = false

-- Get Closest Player
local function getClosest()
    local closest, dist = nil, Settings.FOV
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character and plr.Character:FindFirstChild(Settings.AimPart) then
            if not Settings.TeamCheck or plr.Team ~= LP.Team then
                local pos, onScreen = Camera:WorldToViewportPoint(plr.Character[Settings.AimPart].Position)
                if onScreen then
                    local mag = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                    if mag < dist then
                        dist = mag
                        closest = plr
                    end
                end
            end
        end
    end
    return closest
end

-- Aimbot
RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    if Settings.Aimbot and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild(Settings.AimPart) then
            local aimPos = target.Character[Settings.AimPart].Position
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, aimPos), 0.4)
        end
    end
end)

-- Wallbang Shoot
local function ShootAt(target)
    if target and target.Character and target.Character:FindFirstChild(Settings.AimPart) then
        if Settings.GameMode == "Arsenal" then
            local gun = LP.Backpack:FindFirstChild("Gun") or LP.Character:FindFirstChild("Gun")
            if gun then
                local aimPos = target.Character[Settings.AimPart].Position
                local direction = (aimPos - Camera.CFrame.Position).Unit

                local bullet = Instance.new("Part")
                bullet.Size = Vector3.new(0.2, 0.2, 0.2)
                bullet.Shape = Enum.PartType.Ball
                bullet.Material = Enum.Material.Neon
                bullet.BrickColor = BrickColor.new("Bright yellow")
                bullet.Anchored = false
                bullet.CanCollide = false
                bullet.CFrame = Camera.CFrame
                bullet.Velocity = direction * 500
                bullet.Parent = workspace

                game.Debris:AddItem(bullet, 2)

                task.delay(0.1, function()
                    bullet.Position = aimPos
                end)
            end
        end
    end
end

UIS.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local target = getClosest()
        ShootAt(target)
    end
end)

-- ESP Box com Team Check
local espBoxes = {}

local function CreateBox()
    local box = Drawing.new("Square")
    box.Thickness = 1.5
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Filled = false
    box.Visible = false
    return box
end

RunService.RenderStepped:Connect(function()
    if not Settings.ESP then
        for _, b in pairs(espBoxes) do b.Visible = false end
        return
    end

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if not Settings.TeamCheck or plr.Team ~= LP.Team then
                local hrp = plr.Character.HumanoidRootPart
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                if onScreen then
                    local sizeX = (Camera:WorldToViewportPoint(hrp.Position + Vector3.new(2, 3, 0)).X
                                  - Camera:WorldToViewportPoint(hrp.Position - Vector3.new(2, 3, 0)).X)

                    local box = espBoxes[plr] or CreateBox()
                    espBoxes[plr] = box

                    box.Size = Vector2.new(sizeX, sizeX * 1.5)
                    box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
                    box.Visible = true
                end
            elseif espBoxes[plr] then
                espBoxes[plr].Visible = false
            end
        elseif espBoxes[plr] then
            espBoxes[plr].Visible = false
        end
    end
end)
