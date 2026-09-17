--[[
    Nebula Hub - Murder Mystery 2 Module
    Returns a table consumed by the loader.
--]]

local module = {
    Name = "Murder Mystery 2",
    Theme = "Blood",
    OnThemeChange = function(theme) end,
}

--// Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

--// Shared state across features
local State = {
    AutoShoot = false,
    InfiniteRange = false,
    SilentAim = false,
    ESPCoins = false,
    ESPGun = false,
    ESPRole = false,
    AutoPickup = false,
    NoClip = false,
    Fullbright = false,
    WalkSpeed = 16,
    JumpPower = 50,
    CoinGrabRange = 10,
}

local espFolder

--// Helpers
local function getCharacter()
    return LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
end

local function getRoot()
    local c = getCharacter()
    return c:FindFirstChild("HumanoidRootPart"), c:FindFirstChildOfClass("Humanoid")
end

local function getRole(plr)
    local c = plr.Character
    if not c then return nil end
    if c:FindFirstChild("Knife") then return "Murderer" end
    if c:FindFirstChild("Gun") or (c:FindFirstChild("Backpack") and c.Backpack:FindFirstChild("Gun")) then return "Sheriff" end
    if c:FindFirstChild("Knife") then return "Murderer" end
    -- also check Backpack
    local bp = plr:FindFirstChildOfClass("Backpack")
    if bp then
        if bp:FindFirstChild("Knife") then return "Murderer" end
        if bp:FindFirstChild("Gun") then return "Sheriff" end
    end
    return "Innocent"
end

local function clearFolder()
    if espFolder then espFolder:Destroy() end
    espFolder = Instance.new("Folder")
    espFolder.Name = "NebulaESP"
    espFolder.Parent = game.CoreGui
end

local function makeESP(adornee, color, text)
    local bb = Instance.new("BillboardGui")
    bb.Size = UDim2.new(0, 100, 0, 40)
    bb.AlwaysOnTop = true
    bb.StudsOffset = Vector3.new(0, 3, 0)
    bb.Adornee = adornee
    bb.Parent = espFolder

    local hl = Instance.new("Highlight")
    hl.FillColor = color
    hl.OutlineColor = color
    hl.FillTransparency = 0.6
    hl.OutlineTransparency = 0.2
    hl.Adornee = adornee
    hl.Parent = espFolder

    if text then
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, 0, 1, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = color
        lbl.TextStrokeTransparency = 0
        lbl.Font = Enum.Font.GothamBold
        lbl.TextScaled = true
        lbl.Parent = bb
    end
    return bb
end

--// Feature: ESP
local function updateESP()
    clearFolder()
    if not (State.ESPCoins or State.ESPGun or State.ESPRole) then return end

    -- Coins
    if State.ESPCoins then
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj.Name == "Coin" and obj:IsA("BasePart") then
                makeESP(obj, Color3.fromRGB(255, 215, 0), "💰")
            end
        end
    end

    -- Gun
    if State.ESPGun then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local c = plr.Character
                local gun = c:FindFirstChild("Gun") or (c:FindFirstChild("Backpack") and c.Backpack:FindFirstChild("Gun"))
                if gun then
                    makeESP(c, Color3.fromRGB(0, 150, 255), "🔫 Sheriff")
                end
            end
        end
    end

    -- Role
    if State.ESPRole then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local role = getRole(plr)
                local col = role == "Murderer" and Color3.fromRGB(255,0,0)
                    or role == "Sheriff" and Color3.fromRGB(0,150,255)
                    or Color3.fromRGB(0,255,0)
                makeESP(plr.Character, col, role)
            end
        end
    end
end

--// Feature: Auto Shoot / Silent Aim
local function getNearestMurderer()
    local closest, dist = nil, math.huge
    local myRoot = getRoot()
    if not myRoot then return nil end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            if getRole(plr) == "Murderer" then
                local r = plr.Character:FindFirstChild("HumanoidRootPart")
                if r then
                    local d = (r.Position - myRoot.Position).Magnitude
                    if d < dist then closest, dist = plr, d end
                end
            end
        end
    end
    return closest
end

--// Main loop
local lastESP = 0
RunService.Heartbeat:Connect(function()
    -- ESP refresh (throttled)
    if tick() - lastESP > 1 then
        lastESP = tick()
        pcall(updateESP)
    end

    -- Silent Aim / Auto Shoot
    if State.SilentAim or State.AutoShoot then
        local target = getNearestMurderer()
        if target and target.Character then
            local myChar = LocalPlayer.Character
            local gun = myChar and (myChar:FindFirstChild("Gun") or (LocalPlayer.Backpack and LocalPlayer.Backpack:FindFirstChild("Gun")))
            if gun then
                local targetHead = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("HumanoidRootPart")
                if targetHead then
                    -- Point the gun's handle at target
                    local handle = gun:FindFirstChild("Handle")
                    if handle and handle:IsA("BasePart") then
                        handle.CFrame = CFrame.new(handle.Position, targetHead.Position)
                    end
                    -- Fire
                    if State.AutoShoot then
                        pcall(function() gun:Activate() end)
                    end
                end
            end
        end
    end

    -- Auto Pickup Coins
    if State.AutoPickup then
        local myRoot = getRoot()
        if myRoot then
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj.Name == "Coin" and obj:IsA("BasePart") then
                    if (obj.Position - myRoot.Position).Magnitude < State.CoinGrabRange then
                        pcall(function() obj.CFrame = myRoot.CFrame end)
                    end
                end
            end
        end
    end

    -- WalkSpeed / JumpPower
    local _, hum = getRoot()
    if hum then
        hum.WalkSpeed = State.WalkSpeed
        hum.JumpPower = State.JumpPower
    end

    -- NoClip
    if State.NoClip and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end

    -- Fullbright
    if State.Fullbright then
        local lighting = game:GetService("Lighting")
        lighting.Brightness = 3
        lighting.ClockTime = 14
        lighting.FogEnd = 1e6
        lighting.GlobalShadows = false
    end
end)

--// Build UI (called by loader)
function module.BuildUI()
    local addButton = module.AddButton
    local addToggle = module.AddToggle

    -- Main
    addButton("Main", "💥  Kill All (Touch)", function()
        local myRoot = getRoot()
        if not myRoot then return end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local r = plr.Character:FindFirstChild("HumanoidRootPart")
                if r then r.CFrame = myRoot.CFrame end
            end
        end
    end)

    addButton("Main", "🎯  Teleport to Random Player", function()
        local myRoot = getRoot()
        if not myRoot then return end
        local others = {}
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                table.insert(others, plr)
            end
        end
        if #others > 0 then
            local t = others[math.random(1, #others)]
            myRoot.CFrame = t.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
        end
    end)

    addButton("Main", "🛡  Give Self Gun (Sheriff)", function()
        -- Non-replicating, only works if you have server-side; placeholder
        local plr = LocalPlayer
        local bp = plr:FindFirstChildOfClass("Backpack")
        if bp then
            local gun = game:GetService("ReplicatedStorage")
            -- MM2's Gun template location may vary — try common paths
            local folder = gun:FindFirstChild("Gun") or (gun:FindFirstChild("Weapons") and gun.Weapons:FindFirstChild("Gun"))
            if folder then
                local clone = folder:Clone()
                clone.Parent = bp
            end
        end
    end)

    addToggle("Main", "🪙  Auto Pickup Coins", State.AutoPickup, function(v)
        State.AutoPickup = v
    end)

    -- Combat
    addToggle("Combat", "🔫  Silent Aim (Murderer)", State.SilentAim, function(v)
        State.SilentAim = v
    end)

    addToggle("Combat", "💥  Auto Shoot", State.AutoShoot, function(v)
        State.AutoShoot = v
    end)

    addToggle("Combat", "🎯  Infinite Range", State.InfiniteRange, function(v)
        State.InfiniteRange = v
        -- extend gun range if equipped
        local char = LocalPlayer.Character
        if char then
            for _, tool in ipairs(char:GetChildren()) do
                if tool:IsA("Tool") and tool:FindFirstChild("Handle") then
                    -- Hook RemoteEvent calls — varies by game
                end
            end
        end
    end)

    addButton("Combat", "💀  Instant Kill Nearest", function()
        local target = nil
        local myRoot = getRoot()
        if not myRoot then return end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local r = plr.Character:FindFirstChild("HumanoidRootPart")
                local h = plr.Character:FindFirstChildOfClass("Humanoid")
                if r and h then
                    if (r.Position - myRoot.Position).Magnitude < 15 then
                        h.Health = 0
                    end
                end
            end
        end
    end)

    -- Visual
    addToggle("Visual", "🪙  ESP Coins", State.ESPCoins, function(v) State.ESPCoins = v end)
    addToggle("Visual", "🔫  ESP Gun",   State.ESPGun,   function(v) State.ESPGun = v end)
    addToggle("Visual", "🎭  ESP Role",  State.ESPRole,  function(v) State.ESPRole = v end)
    addToggle("Visual", "💡  Fullbright",State.Fullbright,function(v) State.Fullbright = v end)

    -- Misc
    addToggle("Misc", "👻  NoClip", State.NoClip, function(v) State.NoClip = v end)

    addButton("Misc", "🏃  Speed: 16", function()
        State.WalkSpeed = 16
    end)
    addButton("Misc", "🏃  Speed: 32", function()
        State.WalkSpeed = 32
    end)
    addButton("Misc", "🏃  Speed: 60", function()
        State.WalkSpeed = 60
    end)
    addButton("Misc", "🦘  Jump: 100", function()
        State.JumpPower = 100
    end)
    addButton("Misc", "🔄  Reset Character", function()
        LocalPlayer.Character:BreakJoints()
    end)
end

return module
