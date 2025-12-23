--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// Rayfield
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

local Window = Rayfield:CreateWindow({
    Name = "Chatuba Xit",
    LoadingTitle = "Carregando",
    LoadingSubtitle = "Feito por Zny...",
    ConfigurationSaving = {Enabled = false},
    Theme = "Light"
})

--// Variáveis gerais
local FlyEnabled = false
local FlySpeed = 5
local BV, BG

local AimbotEnabled = false
local FovEnabled = false
local FovSize = 100
local DetectTeam = false
local IgnoreDead = false

local HitboxEnabled = false
local HitboxSize = 5

--// FOV Circle
local FovCircle = Drawing.new("Circle")
FovCircle.Color = Color3.fromRGB(255,255,255)
FovCircle.Thickness = 2
FovCircle.NumSides = 100
FovCircle.Filled = false
FovCircle.Visible = false

RunService.RenderStepped:Connect(function()
    FovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    FovCircle.Radius = FovSize
end)

--// Função alvo mais próximo
local function GetClosestPlayer()
    local closest, dist = nil, FovSize
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            if DetectTeam and plr.Team == LocalPlayer.Team then continue end
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if IgnoreDead and (not hum or hum.Health <= 0) then continue end

            local pos, onscreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
            if onscreen then
                local mag = (Vector2.new(pos.X,pos.Y) - FovCircle.Position).Magnitude
                if mag < dist then
                    dist = mag
                    closest = plr
                end
            end
        end
    end
    return closest
end

--// Aimbot loop
RunService.RenderStepped:Connect(function()
    if AimbotEnabled and FovEnabled then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

--// Hitbox
local function SetHitbox(state)
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            if state then
                head.Size = Vector3.new(HitboxSize,HitboxSize,HitboxSize)
                head.CanCollide = false
                head.Transparency = 0.5
            else
                head.Size = Vector3.new(2,1,1)
                head.Transparency = 0
            end
        end
    end
end

--// MAIN
local MainTab = Window:CreateTab("Main","home")

MainTab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Callback = function(v)
        FlyEnabled = v
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        local hrp = char.HumanoidRootPart

        if v then
            BV = Instance.new("BodyVelocity",hrp)
            BV.MaxForce = Vector3.new(1e9,1e9,1e9)
            BG = Instance.new("BodyGyro",hrp)
            BG.MaxTorque = Vector3.new(1e9,1e9,1e9)
        else
            if BV then BV:Destroy() end
            if BG then BG:Destroy() end
        end
    end
})

MainTab:CreateSlider({
    Name = "Velocidade do Fly",
    Range = {5,20},
    Increment = 1,
    CurrentValue = 5,
    Callback = function(v)
        FlySpeed = v
    end
})

RunService.RenderStepped:Connect(function()
    if FlyEnabled and BV and BG then
        BV.Velocity = Camera.CFrame.LookVector * FlySpeed * 5
        BG.CFrame = Camera.CFrame
    end
end)

--// COMBATE
local CombatTab = Window:CreateTab("Combate","target")

CombatTab:CreateToggle({
    Name = "Ativa Aimbot",
    CurrentValue = false,
    Callback = function(v)
        AimbotEnabled = v
    end
})

CombatTab:CreateSlider({
    Name = "Tamanho do Fov",
    Range = {40,250},
    Increment = 1,
    CurrentValue = 100,
    Callback = function(v)
        FovSize = v
    end
})

CombatTab:CreateToggle({
    Name = "Ativar Fov",
    CurrentValue = false,
    Callback = function(v)
        FovEnabled = v
        FovCircle.Visible = v
    end
})

CombatTab:CreateToggle({
    Name = "Detectar Time",
    CurrentValue = false,
    Callback = function(v)
        DetectTeam = v
    end
})

CombatTab:CreateToggle({
    Name = "Ignorar mortos",
    CurrentValue = false,
    Callback = function(v)
        IgnoreDead = v
    end
})

CombatTab:CreateInput({
    Name = "Tamanho Hitbox (5-20)",
    PlaceholderText = "Digite um número",
    Callback = function(v)
        local n = tonumber(v)
        if n and n >= 5 and n <= 20 then
            HitboxSize = n
        end
    end
})

CombatTab:CreateToggle({
    Name = "Ativar Hitbox",
    CurrentValue = false,
    Callback = function(v)
        HitboxEnabled = v
        SetHitbox(v)
    end
})

--// ESP
local EspTab = Window:CreateTab("Esp","eye")

local EspEnabled = false
local ShowDist = false
local ShowHP = false
local ShowName = false

EspTab:CreateToggle({
    Name = "Ativar esp",
    CurrentValue = false,
    Callback = function(v)
        EspEnabled = v
    end
})

EspTab:CreateToggle({
    Name = "Distância",
    CurrentValue = false,
    Callback = function(v)
        ShowDist = v
    end
})

EspTab:CreateToggle({
    Name = "HP",
    CurrentValue = false,
    Callback = function(v)
        ShowHP = v
    end
})

EspTab:CreateToggle({
    Name = "Name",
    CurrentValue = false,
    Callback = function(v)
        ShowName = v
    end
})