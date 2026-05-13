-- [[ YEW MACRO OVERCLOCK + FPS BOOST ]]
-- Mobile Stable Extreme Version

local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")

-- 删除旧 UI
pcall(function()
    CoreGui.YEW_MACRO:Destroy()
end)

-- =========================
-- FPS BOOST
-- =========================

pcall(function()
    settings().Rendering.QualityLevel =
        Enum.QualityLevel.Level01
end)

pcall(function()

    local Lighting = game:GetService("Lighting")

    Lighting.GlobalShadows = false
    Lighting.FogEnd = 999999
    Lighting.Brightness = 1

    for _,v in ipairs(Lighting:GetChildren()) do

        if v:IsA("BloomEffect")
        or v:IsA("BlurEffect")
        or v:IsA("SunRaysEffect")
        or v:IsA("ColorCorrectionEffect")
        or v:IsA("DepthOfFieldEffect") then

            v.Enabled = false

        end
    end
end)

-- 删除特效
local function removeEffects(v)

    pcall(function()

        if v:IsA("ParticleEmitter")
        or v:IsA("Trail")
        or v:IsA("Beam")
        or v:IsA("Sparkles") then

            v.Enabled = false
            v:Destroy()

        end

        if v:IsA("Highlight") then
            v.Enabled = false
        end

        if v:IsA("Explosion") then
            v:Destroy()
        end

        if v:IsA("PointLight")
        or v:IsA("SpotLight")
        or v:IsA("SurfaceLight") then

            v.Enabled = false

        end

        -- 删除盾牌
        if v:IsA("Part")
        or v:IsA("MeshPart") then

            local n = string.lower(v.Name)

            if string.find(n,"shield")
            or string.find(n,"bubble")
            or string.find(n,"sphere")
            or string.find(n,"parry")
            or string.find(n,"block") then

                v.Transparency = 1
                v.Material = Enum.Material.SmoothPlastic

            end
        end

    end)
end

for _,v in ipairs(Workspace:GetDescendants()) do
    removeEffects(v)
end

Workspace.DescendantAdded:Connect(function(v)
    task.defer(function()
        removeEffects(v)
    end)
end)

-- =========================
-- UI
-- =========================

local gui = Instance.new("ScreenGui")
gui.Name = "YEW_MACRO"
gui.ResetOnSpawn = false
gui.Parent = CoreGui

local frame = Instance.new("Frame")
frame.Parent = gui

frame.Size = UDim2.new(0,180,0,100)
frame.Position = UDim2.new(0,10,0,10)
frame.BackgroundColor3 = Color3.new(0,0,0)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local stroke = Instance.new("UIStroke")
stroke.Parent = frame
stroke.Thickness = 3

local button = Instance.new("TextButton")
button.Parent = frame

button.Size = UDim2.new(1,-20,0,45)
button.Position = UDim2.new(0,10,0,10)

button.BackgroundColor3 = Color3.new(0,0,0)
button.BorderSizePixel = 0

button.Font = Enum.Font.SourceSansBold
button.TextSize = 18
button.Text = "HOOK MACRO"
button.TextColor3 = Color3.new(1,1,1)

local statsLabel = Instance.new("TextLabel")
statsLabel.Parent = frame

statsLabel.Size = UDim2.new(1,0,0,20)
statsLabel.Position = UDim2.new(0,0,1,-22)

statsLabel.BackgroundTransparency = 1
statsLabel.Font = Enum.Font.SourceSansBold
statsLabel.TextSize = 16
statsLabel.TextColor3 = Color3.new(1,1,1)

-- 彩虹 UI
task.spawn(function()

    local h = 0

    while task.wait(0.02) do

        h += 0.008

        if h >= 1 then
            h = 0
        end

        local c = Color3.fromHSV(h,0.85,1)

        stroke.Color = c
        button.TextColor3 = c
        statsLabel.TextColor3 = c

    end
end)

-- FPS / PING
local fps = 60

RunService.RenderStepped:Connect(function(dt)
    fps = math.floor(1/dt)
end)

task.spawn(function()

    while task.wait(1) do

        local ping = 0

        pcall(function()

            ping = math.floor(
                Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
            )

        end)

        statsLabel.Text =
            "FPS: "..fps.." | Ping: "..ping

    end
end)

-- =========================
-- MACRO
-- =========================

local activated = false

local remote = nil
local f_raw = nil

local v1,v2,v3,v4,v5,v6,v7

local isHooked = false

-- 更强速度
local spamThreads = 7
local waitTime = 0.001

local function spamRemote()

    if remote and f_raw then

        pcall(function()

            f_raw(
                remote,
                v1,v2,v3,v4,v5,v6,v7
            )

        end)
    end
end

local function steppedSpam()

    for i = 1, spamThreads do

        task.spawn(function()

            while activated do

                spamRemote()

                task.wait(waitTime)

            end
        end)
    end
end

local function toggleSpam()

    activated = not activated

    if activated then

        button.Text = "STOP MACRO"

        steppedSpam()

        print("YEW MACRO ENABLED")

    else

        button.Text = "START MACRO"

        print("YEW MACRO DISABLED")

    end
end

-- Hook
local function hookMetatable()

    local mt = getrawmetatable(game)
    local oldIndex = mt.__index

    setreadonly(mt,false)

    mt.__index = newcclosure(function(self,key)

        if key == "FireServer" then

            return function(obj,...)

                local args = {...}

                if #args == 7
                and typeof(args[4]) == "CFrame" then

                    remote = obj
                    f_raw = obj.FireServer

                    v1,v2,v3,v4,v5,v6,v7 =
                        args[1],
                        args[2],
                        args[3],
                        args[4],
                        args[5],
                        args[6],
                        args[7]

                    button.Text = "READY"

                    print("YEW MACRO HOOKED")

                end

                return oldIndex(self,key)(obj,...)

            end
        end

        return oldIndex(self,key)

    end)

    setreadonly(mt,true)

    isHooked = true
end

button.MouseButton1Click:Connect(function()

    if not isHooked then

        hookMetatable()

        button.Text = "HOOKED"

        task.wait(0.5)

        button.Text = "START MACRO"

        return
    end

    toggleSpam()

end)

print("YEW MACRO EXTREME LOADED")
print("Mobile Stable Overclock")
