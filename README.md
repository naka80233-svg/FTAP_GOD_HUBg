--[[ 
    Fling Things & Brainrot - Black Edition Ultimate (Network Dominion)
    Version: 18.0 (Rayfield UI Edition - ANTI-DETECTION & HIGH-SPEED)
    
    [UPDATED] 
    - UI: Rayfield Library (V2)
    - Patch: EconomyMath Unknown mutation bypass
]]

-- ==========================================
-- 1. グローバル・イニシャライザ & システム依存
-- ==========================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

-- [システム・物理最適化設定]
pcall(function()
    settings().Physics.AllowSleep = false
    settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
    settings().Physics.ForceRealtimePhysics = true
    sethiddenproperty(workspace, "InterpolationThrottling", Enum.InterpolationThrottlingMode.Disabled)
end)

-- [Rayfield UI ライブラリの読み込み]
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- ==========================================
-- 2. システム・ステート・マトリックス
-- ==========================================
local States = {
    TargetPlayerName = "",
    WalkSpeed = 100,
    JumpPower = 150,
    InfiniteJump = true,
    Noclip = false,
    FlyEnabled = false,
    FlySpeed = 200,
    
    LagIntensity = 20000,
    MassLagEnabled = false,
    MassKickMode = false,
    NetworkOverdrive = true,
    FlingStrength = 800000,
    SimulationRadiusMax = 10000,
    NetworkOwnershipRadius = 6000,
    
    AutoFarm = false,
    BrainrotFarm = false,
    TsunamiEvade = false,
    ItemCaptureRange = 6000,
    
    Auras = {
        Fling = false,
        Orbit = false,
        MegaLag = false
    },
    
    Protections = {
        AntiFling = true,
        AntiMutation = true -- エラー回避フラグ
    }
}

-- ==========================================
-- 3. 物理演算コア & バイパスロジック
-- ==========================================
local function MasterNetworkDominion()
    if not States.NetworkOverdrive then return end
    pcall(function()
        LocalPlayer.MaximumSimulationRadius = States.SimulationRadiusMax
        sethiddenproperty(LocalPlayer, "SimulationRadius", States.SimulationRadiusMax)
        
        local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            local playerPos = root.Position
            local region = Region3.new(
                playerPos - Vector3.new(States.NetworkOwnershipRadius, 1000, States.NetworkOwnershipRadius),
                playerPos + Vector3.new(States.NetworkOwnershipRadius, 1000, States.NetworkOwnershipRadius)
            )
            
            for _, obj in pairs(Workspace:FindPartsInRegion3(region, nil, 3000)) do
                if not obj.Anchored and not obj:IsDescendantOf(LocalPlayer.Character) and not obj:IsDescendantOf(Players) then
                    sethiddenproperty(obj, "NetworkOwnershipRule", Enum.NetworkOwnership.Manual)
                    -- EconomyMath検知を避けるための微細な振動付与
                    obj.Velocity = obj.Velocity + Vector3.new(math.random(-10,10)/100, 0.05, math.random(-10,10)/100)
                end
            end
        end
    end)
end

-- ==========================================
-- 4. UI構築 (Rayfield Library)
-- ==========================================
local Window = Rayfield:CreateWindow({
    Name = "FTAP Dominion V18 - Rayfield Edition",
    LoadingTitle = "Dominion System Initializing...",
    LoadingSubtitle = "by Black Edition",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "DominionRayfield",
        FileName = "Config"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvite",
        RememberJoins = true
    },
    KeySystem = false
})

-- [Main Tab]
local MainTab = Window:CreateTab("Player", 4483345998)
MainTab:CreateSection("Locomotion")

MainTab:CreateSlider({
    Name = "WalkSpeed",
    Info = "プレイヤーの移動速度",
    Range = {16, 1000},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 100,
    Flag = "WalkSpeedSlider",
    Callback = function(Value) States.WalkSpeed = Value end,
})

MainTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = true,
    Flag = "InfJump",
    Callback = function(Value) States.InfiniteJump = Value end,
})

MainTab:CreateToggle({
    Name = "Noclip (Phase)",
    CurrentValue = false,
    Flag = "NoclipToggle",
    Callback = function(Value) States.Noclip = Value end,
})

MainTab:CreateToggle({
    Name = "Anti-Mutation Bypass",
    Info = "EconomyMathエラーを回避します",
    CurrentValue = true,
    Flag = "AntiMutation",
    Callback = function(Value) States.Protections.AntiMutation = Value end,
})

-- [Dominion Tab]
local DomTab = Window:CreateTab("Dominion", 4483345998)
DomTab:CreateSection("Network Interference")

DomTab:CreateToggle({
    Name = "Mass Kick Protocol",
    CurrentValue = false,
    Flag = "MassKick",
    Callback = function(Value) States.MassKickMode = Value end,
})

DomTab:CreateSlider({
    Name = "Fling Strength",
    Range = {1000, 10000000},
    Increment = 100000,
    Suffix = "Power",
    CurrentValue = 800000,
    Flag = "FlingPower",
    Callback = function(Value) States.FlingStrength = Value end,
})

-- [Utility Tab]
local UtilTab = Window:CreateTab("Utility", 4483345998)
UtilTab:CreateSection("Auras & Siphon")

UtilTab:CreateToggle({
    Name = "Fling Aura (Hyper)",
    CurrentValue = false,
    Flag = "FlingAura",
    Callback = function(Value) States.Auras.Fling = Value end,
})

UtilTab:CreateToggle({
    Name = "Auto Siphon (Anti-Detect)",
    Info = "周囲のアイテムを自動回収",
    CurrentValue = false,
    Flag = "AutoSiphon",
    Callback = function(Value) States.AutoFarm = Value end,
})

-- ==========================================
-- 5. 高速 Heartbeat ロジック
-- ==========================================
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChild("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root or not hum then return end

    hum.WalkSpeed = States.WalkSpeed
    hum.JumpPower = States.JumpPower

    MasterNetworkDominion()

    -- 高速自動略奪 (Detection Bypass 適用)
    if States.AutoFarm then
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and not obj.Anchored and not obj:IsDescendantOf(char) and not obj:IsDescendantOf(Players) then
                if (root.Position - obj.Position).Magnitude < States.ItemCaptureRange then
                    local offset = States.Protections.AntiMutation and Vector3.new(math.random(-5,5)/10, 0, math.random(-5,5)/10) or Vector3.new(0,0,0)
                    obj.Velocity = Vector3.new(0, 0, 0)
                    obj.CFrame = root.CFrame * CFrame.new(offset)
                end
            end
        end
    end

    -- プレイヤー干渉
    for _, other in pairs(Players:GetPlayers()) do
        if other ~= LocalPlayer and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
            local oRoot = other.Character.HumanoidRootPart
            local oDist = (root.Position - oRoot.Position).Magnitude
            
            if States.MassKickMode then
                oRoot.Velocity = Vector3.new(0, States.FlingStrength, 0)
                oRoot.CFrame = CFrame.new(oRoot.Position.X, 25000, oRoot.Position.Z)
            end

            if oDist < 120 and States.Auras.Fling then
                root.Velocity = Vector3.new(States.FlingStrength, States.FlingStrength, States.FlingStrength)
                root.RotVelocity = Vector3.new(States.FlingStrength, States.FlingStrength, States.FlingStrength)
            end
        end
    end

    if States.Noclip then
        for _, p in pairs(char:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if States.InfiniteJump and LocalPlayer.Character then
        local h = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

Rayfield:Notify({
    Title = "Dominion V18 Loaded",
    Content = "Rayfield UI Edition: Anti-Mutation Patch Active",
    Duration = 5,
    Image = 4483345998,
    Actions = {
        Ignore = {
            Name = "Okay",
            Callback = function() print("User acknowledged") end
        },
    },
})
