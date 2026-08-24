-- ============================================================
-- TSB VISUAL PACK - FINAL ALL IN ONE
-- ============================================================
-- KORBLOX + HEADLESS
-- HUNTER M1 BLUE EFFECT
-- REMOVE SKILL / DASH PARTICLES
-- DASH TRAIL TETAP ADA
-- FORCE SKYBOX
-- REMOVE TERRAIN CLOUD
-- AUTO RESPAWN
-- CLIENT VISUAL ONLY
-- ============================================================

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local Terrain = Workspace:FindFirstChildOfClass("Terrain")

-- ============================================================
-- SETTINGS
-- ============================================================

local KORBLOX_ID = 139607718

local BLUE = Color3.fromRGB(0, 85, 190)

local HAND_SIZE = 0.42
local TRAIL_WIDTH = 0.5

local REMOVE_SMOKE = true
local REMOVE_PARTICLES = true

local FORCE_SKYBOX = true
local REMOVE_CLOUDS = true

-- ============================================================
-- HEADLESS
-- ============================================================

local function applyHeadless(char)

    local head = char:WaitForChild("Head", 5)
    if not head then
        return
    end

    head.Transparency = 1
    head.CanCollide = false

    for _, v in ipairs(head:GetChildren()) do
        if v:IsA("Decal") then
            v:Destroy()
        end
    end
end

-- ============================================================
-- KORBLOX
-- ============================================================

local function applyKorblox(char)

    local rightLeg = char:WaitForChild("Right Leg", 5)
    if not rightLeg then
        return
    end

    rightLeg.Transparency = 1
    rightLeg.CanCollide = false

    -- Hapus Korblox lama jika ada
    local old = char:FindFirstChild("KorbloxClient")
    if old then
        old:Destroy()
    end

    local success, korblox = pcall(function()
        return game:GetObjects(
            "rbxassetid://" .. KORBLOX_ID
        )[1]
    end)

    if not success or not korblox then
        return
    end

    korblox.Name = "KorbloxClient"
    korblox.Parent = char

    local handle =
        korblox:FindFirstChild("Handle")
        or korblox:FindFirstChildWhichIsA("BasePart", true)

    if not handle then
        korblox:Destroy()
        return
    end

    for _, v in ipairs(handle:GetChildren()) do
        if v:IsA("Weld") or v:IsA("Motor6D") then
            v:Destroy()
        end
    end

    local weld = Instance.new("Motor6D")
    weld.Name = "KorbloxWeld"
    weld.Part0 = rightLeg
    weld.Part1 = handle
    weld.C0 = CFrame.new(0, 0.8, 0)
    weld.Parent = rightLeg

    for _, part in ipairs(korblox:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
            part.Massless = true
        end
    end
end

-- ============================================================
-- APPLY AVATAR
-- ============================================================

local function applyAvatar(char)

    local humanoid = char:FindFirstChildOfClass("Humanoid")

    if humanoid and humanoid.RigType ~= Enum.HumanoidRigType.R6 then
        warn("TSB Visual Pack: R6 required.")
        return
    end

    task.wait(0.3)

    applyHeadless(char)
    applyKorblox(char)

    -- Anti-reset
    task.spawn(function()

        while char.Parent do

            task.wait(0.25)

            local head = char:FindFirstChild("Head")

            if head then
                head.Transparency = 1
                head.CanCollide = false

                for _, v in ipairs(head:GetChildren()) do
                    if v:IsA("Decal") then
                        v:Destroy()
                    end
                end
            end

            local leg = char:FindFirstChild("Right Leg")

            if leg then
                leg.Transparency = 1
                leg.CanCollide = false
            end

        end

    end)
end

-- ============================================================
-- RESPAWN
-- ============================================================

if player.Character then
    task.spawn(function()
        applyAvatar(player.Character)
    end)
end

player.CharacterAdded:Connect(function(char)

    task.wait(0.5)

    applyAvatar(char)

end)

-- ============================================================
-- REMOVE SMOKE / SKILL PARTICLES
-- TRAIL TIDAK DIHAPUS
-- HUNTER PARTICLES DIKECUALIKAN
-- ============================================================

local function isHunterObject(obj)

    if not obj then
        return false
    end

    if obj:GetAttribute("Character") == "Hunter" then
        return true
    end

    local value = obj:FindFirstChild("Character")

    if value
        and value:IsA("StringValue")
        and value.Value == "Hunter" then

        return true
    end

    return false
end

local function findHunterOwner(obj)

    local current = obj

    while current and current ~= Workspace do

        if isHunterObject(current) then
            return current
        end

        current = current.Parent

    end

    return nil
end

local function clearEffect(obj)

    if REMOVE_SMOKE and obj:IsA("Smoke") then
        obj:Destroy()
        return
    end

    if REMOVE_PARTICLES and obj:IsA("ParticleEmitter") then

        -- Jangan hapus particle Hunter
        if findHunterOwner(obj) then
            return
        end

        obj:Destroy()
    end
end

for _, obj in ipairs(Workspace:GetDescendants()) do
    clearEffect(obj)
end

Workspace.DescendantAdded:Connect(function(obj)

    task.defer(function()

        if obj.Parent then
            clearEffect(obj)
        end

    end)

end)

-- ============================================================
-- HUNTER M1 BLUE EFFECT
-- ============================================================

local function applyHunterEffect(obj)

    if not obj or not obj.Parent then
        return
    end

    -- TRAIL
    if obj:IsA("Trail") then

        pcall(function()

            obj.Color = ColorSequence.new(BLUE)

            obj.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.08),
                NumberSequenceKeypoint.new(0.7, 0.35),
                NumberSequenceKeypoint.new(1, 1)
            })

            obj.Texture = ""
            obj.TextureLength = 0

            obj.LightEmission = 0.8
            obj.WidthScale = NumberSequence.new(TRAIL_WIDTH)
            obj.FaceCamera = true

        end)

    -- PARTICLE
    elseif obj:IsA("ParticleEmitter") then

        pcall(function()

            obj.Color = ColorSequence.new(BLUE)

            obj.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.12),
                NumberSequenceKeypoint.new(0.6, 0.4),
                NumberSequenceKeypoint.new(1, 1)
            })

            obj.Size = NumberSequence.new(HAND_SIZE)

            obj.LightEmission = 0.75
            obj.Rotation = NumberRange.new(0, 360)
            obj.RotSpeed = NumberRange.new(-30, 30)
            obj.Lifetime = NumberRange.new(0.25, 0.35)

            obj.Rate = math.min(obj.Rate, 65)

        end)

    end
end

local function checkHunterEffect(obj)

    if not obj:IsA("Trail")
        and not obj:IsA("ParticleEmitter") then
        return
    end

    if findHunterOwner(obj) then
        applyHunterEffect(obj)
    end
end

local function scanHunterEffects()

    for _, obj in ipairs(Workspace:GetDescendants()) do
        checkHunterEffect(obj)
    end

end

Workspace.DescendantAdded:Connect(function(obj)

    if obj:IsA("Trail")
        or obj:IsA("ParticleEmitter") then

        task.defer(function()

            if obj.Parent then
                checkHunterEffect(obj)
            end

        end)

    elseif obj.Name == "Character" then

        task.delay(0.1, scanHunterEffects)

    end

end)

Players.PlayerAdded:Connect(function(plr)

    plr.CharacterAdded:Connect(function(char)

        task.wait(0.5)

        for _, obj in ipairs(char:GetDescendants()) do
            checkHunterEffect(obj)
        end

        task.delay(1, scanHunterEffects)

    end)

end)

task.spawn(function()

    task.wait(1)

    scanHunterEffects()

    while task.wait(1) do
        scanHunterEffects()
    end

end)

-- ============================================================
-- FORCE SKYBOX
-- ============================================================

local skyIds = {

    Bk = "rbxassetid://92959017845968",
    Ft = "rbxassetid://129304841254693",
    Lf = "rbxassetid://129249062260004",
    Rt = "rbxassetid://117319232583147",
    Up = "rbxassetid://121193772599100",
    Dn = "rbxassetid://115022734343595"

}

local function applySkybox()

    if not FORCE_SKYBOX then
        return
    end

    for _, obj in ipairs(Lighting:GetChildren()) do

        if obj:IsA("Sky") then
            obj:Destroy()
        end

    end

    local sky = Instance.new("Sky")

    sky.Name = "TSB_ClientSky"

    sky.SkyboxBk = skyIds.Bk
    sky.SkyboxFt = skyIds.Ft
    sky.SkyboxLf = skyIds.Lf
    sky.SkyboxRt = skyIds.Rt
    sky.SkyboxUp = skyIds.Up
    sky.SkyboxDn = skyIds.Dn

    sky.Parent = Lighting
end

applySkybox()

-- ============================================================
-- SKYBOX ANTI-RESET
-- ============================================================

Lighting.ChildAdded:Connect(function(obj)

    if not FORCE_SKYBOX then
        return
    end

    if obj:IsA("Sky")
        and obj.Name ~= "TSB_ClientSky" then

        task.defer(function()
            if obj.Parent then
                obj:Destroy()
            end

            if not Lighting:FindFirstChild("TSB_ClientSky") then
                applySkybox()
            end
        end)

    end

end)

-- ============================================================
-- REMOVE TERRAIN CLOUDS
-- ============================================================

local function removeClouds()

    if not REMOVE_CLOUDS then
        return
    end

    Terrain = Workspace:FindFirstChildOfClass("Terrain")

    if not Terrain then
        return
    end

    for _, obj in ipairs(Terrain:GetChildren()) do

        if obj:IsA("Clouds") then

            obj.Enabled = false
            obj.Cover = 0
            obj.Density = 0

        end

    end
end

removeClouds()

task.spawn(function()

    while REMOVE_CLOUDS do

        task.wait(0.5)

        removeClouds()

    end

end)

-- ============================================================
-- FINAL
-- ============================================================

print("🔥 TSB VISUAL PACK FINAL LOADED")
print("✅ Headless")
print("✅ Korblox")
print("✅ Hunter M1 Blue")
print("✅ Skill/Dash Smoke Removed")
print("✅ Dash Trail Preserved")
print("✅ Force Skybox")
print("✅ Terrain Clouds Removed")
print("✅ Auto Respawn")
