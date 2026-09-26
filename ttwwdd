if getgenv and getgenv().__oh_v56 then
    warn("[Origin HUB] v56 already loaded -- rejoin before running it again")
    return
end
-- detect OLDER versions still loaded (you re-ran the script without a full rejoin). Their old fire
-- hooks stay active and can break bullet reg / cause fake-ammo shots no matter what THIS version
-- does -- only a fresh rejoin clears them. Warn loudly so it's obvious.
if getgenv then
    local stale = {}
    for v = 40, 55 do if getgenv()["__oh_v"..v] then stale[#stale+1] = "v"..v end end
    if #stale > 0 then
        warn("[Origin HUB] !!! OLD VERSION(S) STILL LOADED: " .. table.concat(stale, ", ") ..
             " -- their old hooks will break bullet reg. FULLY REJOIN (close & rejoin), do not just re-execute.")
        task.delay(3, function() pcall(function()
            game:GetService("StarterGui"):SetCore("SendNotification", { Title = "Origin HUB", Text = "Old version still loaded -- REJOIN to fix broken shots", Duration = 8 })
        end) end)
    end
end
if getgenv then getgenv().__oh_v56 = true end

local oldWarn = warn
warn = function(...)
    local msg = tostring((...))
    if msg:find("argument #1 expects a string") then return end
    oldWarn(...)
end

local Players  = game:GetService("Players")
local RS       = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")
local RunSvc   = game:GetService("RunService")
local UIS      = game:GetService("UserInputService")
local Http     = game:GetService("HttpService")
local VIM      = game:GetService("VirtualInputManager")
local SoundSvc = game:GetService("SoundService")
local LP       = Players.LocalPlayer
local Cam      = workspace.CurrentCamera

local function allHosts()
    local hosts = {}
    pcall(function() local h = gethui and gethui() if h then table.insert(hosts, h) end end)
    pcall(function() table.insert(hosts, game:GetService("CoreGui")) end)
    return hosts
end
local function getMainHost() local h = gethui and gethui() return h or game:GetService("CoreGui") end
local function snapshotAllHosts()
    local snap = {}
    for _, host in ipairs(allHosts()) do
        pcall(function() for _, c in ipairs(host:GetChildren()) do if c:IsA("ScreenGui") then snap[c] = true end end end)
    end
    return snap
end
local function captureNewGuisInto(snap, list)
    for _, host in ipairs(allHosts()) do
        pcall(function() for _, c in ipairs(host:GetChildren()) do if c:IsA("ScreenGui") and not snap[c] then table.insert(list, c) end end end)
    end
end

local AR2_CFG = {
    players  = { on=false, box=true, name=true, dist=true, health=true, weapon=false, maxDist=5000, tungGlobal=false },
    zombies  = { on=false, box=true, name=false, dist=true, health=true, maxDist=3000 },
    vehicles = { on=false, box=true, name=true, dist=true, moving=false, maxDist=3000 },
    horses   = { on=false, box=true, name=true, dist=true, maxDist=5000 },
    corpses  = { on=false, box=true, name=true, dist=true, maxDist=2000 },
    loot     = { on=false, box=true, name=true, dist=true, maxDist=500, highlight=true, keyword="", maxDraw=40, maxHighlight=20, tierFilter="All" },
    drops    = { on=false, box=true, name=true, dist=true, maxDist=1000, highlight=true },
    aim      = { on=false, hold=true, smooth=1, fov=250, maxDist=1500, showFov=false, mode360=false },
    silent   = { on=false, fov=250, fovDeg=30, showFov=false },
    trigger  = { on=false, holdRMB=false, fov=100, maxDist=300, delay=0.08, visibleOnly=false, mode360=false,
                 useSilent=false, autoHold=false, burst=1, burstGap=0.04, forceAds=false },
    weapon   = { on=false, hitbox=false, hitboxSize=50 },
    cars     = { noclip=false, noclipSpeed=90, noclipEjectOnDisable=false, fly=false, flySpeed=60,
                 platformOnDisable=true, speedMult=false, speedMultVal=2, enterRange=120, clipOff=false,
                 horseFly=false, horseFlySpeed=60,
                 horseSpeed=false, horseSpeedVal=120, horseJump=false, horseJumpVal=150, horseEnterRange=600 },
    wallbang = { on=false, zombies=true },
    esp      = { style="Corners", cornerLen=0.28, fillTint=false, thickness=1, textSize=13,
                 box3D=false, box3DZombies=true, box3DColor=Color3.fromRGB(255,70,70) },
    hitmark  = { on=false, volume=0.5, sound="Ping", customId="" },
    killfeed = { on=false },
    move     = { walkOn=false, walkSpeed=16 },
    tp       = { tpOffsetY=4, arrivalDist=4,
                 fallTime=6, settleTime=2, respawnTP=false, respawnDelay=1.5,
                 holdTime=3, retries=6,
                 clickTP=false, clickTPKey="T",
                 mapFake=false, deathSave=false,
                 mapRot=9 },
    misc     = {
        noclip=false, infJump=false,
        freeze=false, freezeAll=false, freezeRadius=300,
        rapidFire=false, rapidShots=2, rapidDelay=0.05,
        fullbright=false, noFog=false, setTime=false, timeValue=14, brightness=3,
        skybox="Default", removeShadows=false,
        removeTextures=false, removeFoliage=false, removeDoors=false, removeSunRays=false,
        removeWeather=false, forceStream=false,
        removeClouds=false, modifyClouds=false, cloudsColor=Color3.fromRGB(255,255,255), cloudsCover=0.5, cloudsDensity=0.5,
        customAmbient=false, ambientIndoor=Color3.fromRGB(127,127,127), ambientOutdoor=Color3.fromRGB(127,127,127),
        personalCharOn=false, personalCharMode="Color", personalCharHue=0, personalCharGray=100, personalCharMaterial="ForceField", personalCharRainbow=false,
        personalGunOn=false, personalGunMode="Color", personalGunHue=0, personalGunGray=100, personalGunMaterial="Neon", personalGunRainbow=false,
        distZoomEnabled=false, distZoomActive=false, normalDist=10, zoomedDist=3,
        fovZoom=false, zoomFOV=30,
        tungSelf=false, gunDebug=false, alwaysEquip=false, playerFly=false, playerFlySpeed=80,
        morphId="", morphHideSelf=false, morphPreset="Custom",
    },
    ui = {
        accent=Color3.fromRGB(255,140,30), accentHue=30,
        background=Color3.fromRGB(14,14,14), panel=Color3.fromRGB(20,20,20),
        transparency=0.15, cornerRadius=8, glow=true, compact=false, sizePreset="Default",
    },
    keybinds = {
        playerESP="None", zombieESP="None", vehicleESP="None", corpseESP="None", lootESP="None",
        dropESP="None",
        aimbot="None", triggerBot="None", noclip="None", infJump="None",
        silentAim="None", silentAim360="None",
        freezeZ="None", rapidFire="None",
        fullbright="None", noFog="None", personalChams="None",
        tungSelf="None", tungGlobal="None", distZoom="None", fovZoom="None",
        tpNearestP="None", tpNearestZ="None", tpNearestC="None",
        aimbot360="None", triggerBot360="None",
        carNoclip="None", carFly="None", carEnter="None", carPark="None", carRepair="None", carClip="None", horseFly="None", playerFly="None",
        horseESP="None", horseEnter="None", horseSpeed="None", horseJump="None",
        mapFake="None", deathTP="None", wallbang="None", spinBot="None",
        walkSpeed="None",
        weaponMods="None", forceAds="None", hitbox="None", alwaysEquip="None",
    },
}

-- shared hit feed: bumped on EVERY confirmed hit (silent aim on OR off) so the hitmarker and the
-- fake killfeed both fire regardless of which fire path was used.
AR2_HITFEED = { n = 0, name = "-", dmg = 0, killed = false, at = 0 }

local AR2_SETTINGS = { menuKey = "P" }
local AR2_UI_SIZES = { ["Tiny"]=0.65, ["Small"]=0.80, ["Default"]=1.00, ["Large"]=1.20, ["Huge"]=1.45, ["Colossal"]=1.75 }
local AR2_CONFIG_FILE = "origin_hub_twd_index.json"
local AR2_configList = {}

local function AR2_uiSizeToUDim2()
    local mult = AR2_UI_SIZES[AR2_CFG.ui.sizePreset or "Default"] or 1.0
    return UDim2.fromOffset(math.floor(600 * mult), math.floor(500 * mult))
end

local function encodeColors(v)
    if typeof(v) == "Color3" then return { __c3=true, r=v.R, g=v.G, b=v.B } end
    if type(v) == "table" then
        local out = {}
        for k, v2 in pairs(v) do if type(k)~="function" and type(k)~="userdata" then out[k]=encodeColors(v2) end end
        return out
    end
    return v
end
local function decodeColors(v)
    if type(v) == "table" then
        if v.__c3 then return Color3.new(v.r,v.g,v.b) end
        local out = {}
        for k, v2 in pairs(v) do out[k]=decodeColors(v2) end
        return out
    end
    return v
end
local function AR2_mergeInto(target, saved)
    if type(saved) ~= "table" then return end
    for key, current in pairs(target) do
        local incoming = saved[key]
        if type(current) == "table" then AR2_mergeInto(current, incoming)
        elseif incoming ~= nil and type(incoming) == type(current) then target[key]=incoming end
    end
end

local function AR2_basename(p) return tostring(p):match("[^/\\]+$") or tostring(p) end
local function AR2_cfgName(n)
    n = tostring(n or ""):gsub("[^%w%s_-]", ""):gsub("^%s+", ""):gsub("%s+$", "")
    return n
end
local function AR2_refreshConfigList()
    AR2_configList = {}
    if isfile and readfile then
        local exists = false
        pcall(function() exists = isfile(AR2_CONFIG_FILE) end)
        if exists then
            local ok, raw = pcall(readfile, AR2_CONFIG_FILE)
            if ok and raw and raw ~= "" then
                local decoded, data = pcall(Http.JSONDecode, Http, raw)
                if decoded and type(data) == "table" and type(data.configs) == "table" then
                    for _, n in ipairs(data.configs) do if type(n)=="string" and n~="" then table.insert(AR2_configList, n) end end
                end
            end
        end
    end
    if listfiles then
        local ok, files = pcall(listfiles, "")
        if ok and type(files) == "table" then
            local existing = {}
            for _, n in ipairs(AR2_configList) do existing[n] = true end
            for _, file in ipairs(files) do
                local base = AR2_basename(file)
                local name = base:match("origin_hub_twd_(.+)%.json$")
                if name and name ~= "index" and not existing[name] then table.insert(AR2_configList, name) existing[name] = true end
            end
        end
    end
    return AR2_configList
end
local function AR2_saveConfig(name)
    if not writefile then warn("[Origin HUB] executor has no writefile") return false end
    name = AR2_cfgName(name)
    if name == "" then return false end
    local file = "origin_hub_twd_" .. name .. ".json"
    local payload = { version = 54, cfg = AR2_CFG, settings = AR2_SETTINGS, sa = SA, tracer = AR2_tracer, spin = AR2_spin }
    local ok, encoded = pcall(Http.JSONEncode, Http, encodeColors(payload))
    if not ok then warn("[Origin HUB] config encode failed") return false end
    if not pcall(writefile, file, encoded) then warn("[Origin HUB] config write failed: " .. file) return false end
    local exists = false
    for _, n in ipairs(AR2_configList) do if n == name then exists = true break end end
    if not exists then AR2_configList[#AR2_configList + 1] = name end
    local idxOk, idxEncoded = pcall(Http.JSONEncode, Http, { configs = AR2_configList })
    if idxOk then pcall(writefile, AR2_CONFIG_FILE, idxEncoded) end
    AR2_refreshConfigList()
    return true
end
function AR2_applyConfig() end
local function AR2_loadConfigByName(name)
    name = AR2_cfgName(name)
    if name == "" or not (readfile and isfile) then return false end
    local file = "origin_hub_twd_" .. name .. ".json"
    local present = false
    pcall(function() present = isfile(file) end)
    if not present then warn("[Origin HUB] no such config: " .. name) return false end
    local ok, raw = pcall(readfile, file)
    if not ok or not raw then warn("[Origin HUB] could not read " .. file) return false end
    local decoded, data = pcall(Http.JSONDecode, Http, raw)
    if not decoded or type(data) ~= "table" then warn("[Origin HUB] config file is corrupt: " .. file) return false end
    local clean = decodeColors(data)
    AR2_mergeInto(AR2_CFG, clean.cfg)
    AR2_mergeInto(AR2_SETTINGS, clean.settings)
    if clean.sa     then AR2_mergeInto(SA, clean.sa) end
    if clean.tracer then AR2_mergeInto(AR2_tracer, clean.tracer) end
    if clean.spin   then AR2_mergeInto(AR2_spin, clean.spin) end
    AR2_applyConfig()
    return true
end
local function AR2_deleteConfig(name)
    name = AR2_cfgName(name)
    if name == "" then return false end
    pcall(function()
        local file = "origin_hub_twd_" .. name .. ".json"
        if isfile and isfile(file) and delfile then delfile(file) end
    end)
    for i, n in ipairs(AR2_configList) do if n == name then table.remove(AR2_configList, i) break end end
    local idxOk, idxEncoded = pcall(Http.JSONEncode, Http, { configs = AR2_configList })
    if idxOk then pcall(writefile, AR2_CONFIG_FILE, idxEncoded) end
    AR2_refreshConfigList()
    return true
end

local function AR2_normalizeKey(k)
    if not k or k == "" then return "None" end
    k = tostring(k):gsub("^%s+",""):gsub("%s+$","")
    if k == "" then return "None" end
    local low = k:lower()
    if low == "none" then return "None" end
    local aliases = {
        ["mouse1"]="MouseButton1", ["mouse2"]="MouseButton2", ["mouse3"]="MouseButton3",
        ["m1"]="MouseButton1", ["m2"]="MouseButton2", ["m3"]="MouseButton3",
        ["ctrl"]="LeftControl", ["control"]="LeftControl", ["shift"]="LeftShift",
        ["lshift"]="LeftShift", ["spacebar"]="Space", ["space"]="Space",
        ["esc"]="Escape", ["enter"]="Return",
    }
    if aliases[low] then return aliases[low] end
    if #k == 1 then local c = k:upper() if c >= "A" and c <= "Z" then return c end end
    local upper = k:upper()
    if upper:match("^F%d+$") then return upper end
    return k:sub(1,1):upper()..k:sub(2)
end
local function inputToKeyName(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then return "MouseButton1" end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then return "MouseButton2" end
    if input.UserInputType == Enum.UserInputType.MouseButton3 then return "MouseButton3" end
    if input.UserInputType == Enum.UserInputType.Keyboard then return input.KeyCode.Name end
    return nil
end

local function getZombieContainer()
    local ai = workspace:FindFirstChild("AI")
    if ai then local w = ai:FindFirstChild("Walkers") return w or ai end
    return nil
end
local VEHICLE_FOLDERS = { "Cars", "Vehicles", "Horses", "Mounts" }
local function getVehicleContainers()
    local out = {}
    for _, n in ipairs(VEHICLE_FOLDERS) do local f = workspace:FindFirstChild(n) if f then out[#out+1] = f end end
    return out
end
local function getVehicleContainer()
    for _, n in ipairs(VEHICLE_FOLDERS) do local f = workspace:FindFirstChild(n) if f then return f end end
    return nil
end

function AR2_readEntityHealth(model)
    if not model then return nil, nil end
    local hum = model:FindFirstChildWhichIsA("Humanoid")
    if hum and type(hum.MaxHealth) == "number" and hum.MaxHealth > 0 then return hum.Health, hum.MaxHealth end
    local anim = model:FindFirstChild("Animator")
    if anim then
        local hp = anim:FindFirstChild("Health")
        local mhp = anim:FindFirstChild("MaxHealth")
        if hp and hp:IsA("ValueBase") then
            local h = tonumber(hp.Value)
            local mh = (mhp and mhp:IsA("ValueBase") and tonumber(mhp.Value)) or 100
            if h then return h, math.max(mh, 1) end
        end
    end
    local bestHp, bestMhp
    for _, d in ipairs(model:GetDescendants()) do
        if d:IsA("ValueBase") then
            if d.Name == "Health" or d.Name == "HP" then local h = tonumber(d.Value) if h and (not bestHp or h > bestHp) then bestHp = h end
            elseif d.Name == "MaxHealth" or d.Name == "MaxHP" then local m = tonumber(d.Value) if m and (not bestMhp or m > bestMhp) then bestMhp = m end end
        end
    end
    if bestHp then return bestHp, bestMhp or 100 end
    return nil, nil
end

-- =============================================
-- DRAWING POOL (corner-box)
-- =============================================
local pool = {}
local function getDraw(i)
    if not pool[i] then
        local box = Drawing.new("Square")
        box.Thickness, box.Filled, box.Transparency = 1, false, 1
        box.Color, box.Visible = Color3.fromRGB(255,90,90), false
        local fill = Drawing.new("Square")
        fill.Thickness, fill.Filled, fill.Transparency = 1, true, 0.15
        fill.Color, fill.Visible = Color3.fromRGB(255,90,90), false
        local corners = {}
        for c = 1, 8 do
            local l = Drawing.new("Line")
            l.Thickness, l.Transparency, l.Visible = 1, 1, false
            l.Color = Color3.fromRGB(255,90,90)
            corners[c] = l
        end
        local nm = Drawing.new("Text") nm.Size, nm.Center, nm.Outline = 13, true, true nm.Color, nm.Visible = Color3.fromRGB(255,255,255), false
        local ds = Drawing.new("Text") ds.Size, ds.Center, ds.Outline = 12, true, true ds.Color, ds.Visible = Color3.fromRGB(200,200,200), false
        local hbBg = Drawing.new("Square") hbBg.Filled, hbBg.Color, hbBg.Transparency, hbBg.Visible = true, Color3.fromRGB(15,15,15), 1, false
        local hbFill = Drawing.new("Square") hbFill.Filled, hbFill.Color, hbFill.Transparency, hbFill.Visible = true, Color3.fromRGB(80,220,90), 1, false
        pool[i] = { box=box, fill=fill, corners=corners, name=nm, dist=ds, hbBg=hbBg, hbFill=hbFill }
    end
    return pool[i]
end
local function hideAll()
    for _, e in ipairs(pool) do
        e.box.Visible=false e.fill.Visible=false e.name.Visible=false e.dist.Visible=false
        e.hbBg.Visible=false e.hbFill.Visible=false
        for _, l in ipairs(e.corners) do l.Visible=false end
    end
end

local function drawEspBox(e, x, y, w, h, col, wantBox)
    if (not wantBox) or (not x) or (not y) or (not w) or (not h) then
        e.box.Visible=false e.fill.Visible=false
        for _, l in ipairs(e.corners) do l.Visible=false end
        return
    end
    local st = AR2_CFG.esp.style
    local th = AR2_CFG.esp.thickness or 1
    if AR2_CFG.esp.fillTint then
        e.fill.Position = Vector2.new(x, y) e.fill.Size = Vector2.new(w, h)
        e.fill.Color = col e.fill.Transparency = 0.12 e.fill.Visible = true
    else e.fill.Visible = false end
    if st == "Full" then
        e.box.Position = Vector2.new(x, y) e.box.Size = Vector2.new(w, h)
        e.box.Color = col e.box.Thickness = th e.box.Visible = true
        for _, l in ipairs(e.corners) do l.Visible=false end
    else
        e.box.Visible = false
        local len = math.floor(math.min(w, h) * (AR2_CFG.esp.cornerLen or 0.28))
        len = math.max(len, 4)
        local c = e.corners
        local function seg(idx, x1, y1, x2, y2)
            local l = c[idx]
            l.From = Vector2.new(x1, y1) l.To = Vector2.new(x2, y2)
            l.Color = col l.Thickness = th l.Visible = true
        end
        seg(1, x, y, x+len, y)          seg(2, x, y, x, y+len)
        seg(3, x+w, y, x+w-len, y)      seg(4, x+w, y, x+w, y+len)
        seg(5, x, y+h, x+len, y+h)      seg(6, x, y+h, x, y+h-len)
        seg(7, x+w, y+h, x+w-len, y+h)  seg(8, x+w, y+h, x+w, y+h-len)
    end
end

local function drawHealthBar(e, x, y, h, pct)
    pct = math.clamp(pct, 0, 1)
    local bw = 4
    local bx = x - bw - 3
    local fh = math.max(1, math.floor(h * pct))
    e.hbBg.Position = Vector2.new(bx, y) e.hbBg.Size = Vector2.new(bw, h)
    e.hbBg.Color = Color3.fromRGB(12,12,12) e.hbBg.Transparency = 0.6 e.hbBg.Visible = true
    local fc
    if pct > 0.5 then fc = Color3.fromRGB(math.floor(255*(1-(pct-0.5)*2)), 220, 60)
    else fc = Color3.fromRGB(230, math.floor(220*(pct*2)), 40) end
    e.hbFill.Position = Vector2.new(bx, y+h-fh) e.hbFill.Size = Vector2.new(bw, fh)
    e.hbFill.Color = fc e.hbFill.Transparency = 0 e.hbFill.Visible = true
end

local aimFovCircle = Drawing.new("Circle")
aimFovCircle.Thickness, aimFovCircle.Filled, aimFovCircle.Transparency = 1, false, 1
aimFovCircle.NumSides, aimFovCircle.Color, aimFovCircle.Visible = 64, Color3.fromRGB(255,90,90), false
local silentFovCircle = Drawing.new("Circle")
silentFovCircle.Thickness, silentFovCircle.Filled, silentFovCircle.Transparency = 1, false, 1
silentFovCircle.NumSides, silentFovCircle.Color, silentFovCircle.Visible = 64, Color3.fromRGB(255,120,220), false
local trigFovCircle = Drawing.new("Circle")
trigFovCircle.Thickness, trigFovCircle.Filled, trigFovCircle.Transparency = 1, false, 1
trigFovCircle.NumSides, trigFovCircle.Color, trigFovCircle.Visible = 64, Color3.fromRGB(80,220,90), false
local lockLine = Drawing.new("Line")
lockLine.Thickness, lockLine.Transparency, lockLine.Color, lockLine.Visible = 1, 1, Color3.fromRGB(255,90,90), false
local tpBeam = Drawing.new("Line")
tpBeam.Thickness, tpBeam.Transparency, tpBeam.Color, tpBeam.Visible = 2, 1, Color3.fromRGB(80,220,255), false

local function d3(a,b) return (a-b).Magnitude end

local function getPartBox(part)
    if not part or not part:IsA("BasePart") then return nil end
    local top, onTop = Cam:WorldToViewportPoint(part.Position + Vector3.new(0, part.Size.Y/2, 0))
    local bot, onBot = Cam:WorldToViewportPoint(part.Position - Vector3.new(0, part.Size.Y/2, 0))
    if not (onTop and onBot and top.Z > 0 and bot.Z > 0) then return nil end
    local h = math.abs(bot.Y - top.Y)
    if h < 3 or h > 2500 then return nil end
    return top.X - h/4, top.Y, h/2, h
end

local function sa4PlayerBox(char)
    local part = char:FindFirstChild("Head") or char:FindFirstChild("Head2")
               or char:FindFirstChild("UpperTorso") or char:FindFirstChild("HumanoidRootPart")
    if not part then return nil end
    local pos = part.Position
    local ts, tOn = Cam:WorldToViewportPoint(pos + Vector3.new(0, 2.2, 0))
    local bs, bOn = Cam:WorldToViewportPoint(pos - Vector3.new(0, 3.2, 0))
    if not (tOn and bOn) or ts.Z <= 0 then return nil end
    local h = math.abs(bs.Y - ts.Y)
    local w = h * 0.55
    return ts.X - w/2, ts.Y, w, h
end

local function getModelBox(model)
    if not model then return nil end
    local head = model:FindFirstChild("Head") or model:FindFirstChild("Head2")
    local root = model:FindFirstChild("HumanoidRootPart")
    if head and head:IsA("BasePart") and root and root:IsA("BasePart") then
        local topPos = head.Position + Vector3.new(0, head.Size.Y/2 + 0.4, 0)
        local botPos = Vector3.new(root.Position.X, root.Position.Y - 2.8, root.Position.Z)
        local top, onTop = Cam:WorldToViewportPoint(topPos)
        local bot, onBot = Cam:WorldToViewportPoint(botPos)
        if onTop and onBot and top.Z > 0 and bot.Z > 0 then
            local h = math.abs(bot.Y - top.Y)
            if h >= 3 and h <= 2500 then return top.X - h/4, top.Y, h/2, h end
        end
    end
    local prim = model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
    if not prim then return nil end
    return getPartBox(prim)
end

local function getWideModelBox(model)
    if not model then return nil end
    local ok, cf, size = pcall(function() return model:GetBoundingBox() end)
    if not ok or not size then return nil end
    local hx, hy, hz = size.X/2, size.Y/2, size.Z/2
    local offsets = {
        Vector3.new( hx, hy, hz), Vector3.new(-hx, hy, hz), Vector3.new( hx,-hy, hz), Vector3.new(-hx,-hy, hz),
        Vector3.new( hx, hy,-hz), Vector3.new(-hx, hy,-hz), Vector3.new( hx,-hy,-hz), Vector3.new(-hx,-hy,-hz),
    }
    local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
    local any = false
    for _, o in ipairs(offsets) do
        local s, on = Cam:WorldToViewportPoint((cf * CFrame.new(o)).Position)
        if on and s.Z > 0 then
            any = true
            if s.X < minX then minX = s.X end
            if s.Y < minY then minY = s.Y end
            if s.X > maxX then maxX = s.X end
            if s.Y > maxY then maxY = s.Y end
        end
    end
    if not any then return nil end
    local hgt = maxY - minY
    if hgt < 3 or hgt > 2500 then return nil end
    return minX, minY, maxX - minX, hgt
end

local function isVisible(model, part)
    if not part then return false end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = { LP.Character, model }
    return workspace:Raycast(Cam.CFrame.Position, part.Position - Cam.CFrame.Position, params) == nil
end

local function resolveTargetPart(char)
    if not char then return nil end
    return char:FindFirstChild("Head") or char:FindFirstChild("Head2")
end

-- =============================================
-- SILENT AIM + WALLBANG
-- Remote is re-found on a loop (fixes "must toggle at join before TP"):
-- retries until CommandV8 exists, then re-verifies in case it gets swapped.
-- =============================================
do
local WS = workspace

local CFG = {
    on = false, fovDeg = 25, mode360 = false,
    wDir = true, wEnd = true, wPart = true, wPos = true, wHuman = true,
    wTrim = true, wStop = true, stopValue = "Energy", wBuild = true,
    showFov = true, showLine = true, esp = true, espMaxDist = 800, alive = true,
}

local A = { shots = 0, hits = 0 }
local B = { shots = 0, hits = 0 }
local stuck, tried = 0, 0
local lastInfo, lastStop = "-", "?"
local log = {}
local function logLine(s) table.insert(log,1,s) if #log>7 then table.remove(log) end end

local hasDrawing = false
pcall(function() local d=Drawing.new("Line") pcall(function() d:Remove() end) hasDrawing=true end)

local function myHead()
    local c = LP.Character
    return c and (c:FindFirstChild("Head") or c:FindFirstChild("HumanoidRootPart"))
end

local candidates = {}
local target = { part = nil, pos = nil, name = "-", model = nil }

task.spawn(function()
    while CFG.alive do
        local head = myHead()
        local list = {}
        if head then
            local from = head.Position
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= LP and pl.Character then
                    local p = pl.Character:FindFirstChild("Head")
                           or pl.Character:FindFirstChild("UpperTorso")
                           or pl.Character:FindFirstChild("HumanoidRootPart")
                    local h = pl.Character:FindFirstChildOfClass("Humanoid")
                    if p and h and h.Health > 0 then
                        list[#list+1] = { model=pl.Character, part=p, hum=h, name=pl.Name, dist=(p.Position-from).Magnitude }
                    end
                end
            end
        end
        candidates = list
        if CFG.on and head then
            local from = head.Position
            if CFG.mode360 then
                -- 360: nearest player in ANY direction, no FOV cone
                local best, bestD = nil, math.huge
                for _, c in ipairs(candidates) do
                    local d = (c.part.Position - from).Magnitude
                    if d < bestD then bestD = d best = c end
                end
                if best then
                    target.part, target.pos = best.part, best.part.Position
                    target.model, target.name = best.model, best.name
                else target.part, target.pos, target.model, target.name = nil, nil, nil, "-" end
            else
                local look = WS.CurrentCamera.CFrame.LookVector
                local best, bestAng = nil, CFG.fovDeg
                for _, c in ipairs(candidates) do
                    local d = c.part.Position - from
                    if d.Magnitude > 1 then
                        local ang = math.deg(math.acos(math.clamp(d.Unit:Dot(look),-1,1)))
                        if ang < bestAng then bestAng, best = ang, c end
                    end
                end
                if best then
                    target.part, target.pos = best.part, best.part.Position
                    target.model, target.name = best.model, best.name
                else target.part, target.pos, target.model, target.name = nil, nil, nil, "-" end
            end
        else target.part, target.pos, target.model, target.name = nil, nil, nil, "-" end
        task.wait(0.05)
    end
end)

local function snap()
    local s = {}
    for _, c in ipairs(candidates) do s[#s+1] = { k=c.name, h=c.hum, v=c.hum.Health } end
    return s
end
local function confirm(before, bucket, tag)
    task.delay(0.30, function()
        for _, r in ipairs(before) do
            if r.h and r.h.Parent and r.h.Health < r.v - 0.5 then
                bucket.hits = bucket.hits + 1
                logLine(("HIT %s -%.0f [%s]"):format(r.k, r.v - r.h.Health, tag))
                return
            end
        end
        logLine(("no reg [%s]"):format(tag))
    end)
end

local function setV(tbl, key, value, failList)
    if tbl == nil then return false end
    tried = tried + 1
    local okW = pcall(function() tbl[key] = value end)
    if not okW then if failList then failList[#failList+1] = tostring(key) end return false end
    local okR, now = pcall(function() return tbl[key] end)
    if not okR then return false end
    local same
    if typeof(now) == "Vector3" and typeof(value) == "Vector3" then same = (now - value).Magnitude < 0.01
    else same = (now == value) end
    if same then stuck = stuck + 1 return true end
    if failList then failList[#failList+1] = tostring(key) end
    return false
end

local function forgePath(data, origin, hitPos, hitPart)
    local dir = (hitPos - origin)
    local dist = dir.Magnitude
    if dist < 0.01 then return "too close" end
    dir = dir.Unit
    local fails = {}
    if CFG.wDir then setV(data, "Direction", dir, fails) end
    local paths = data.Paths
    if type(paths) ~= "table" then return "no Paths" end
    local p1 = paths[1]
    if type(p1) ~= "table" then return "no Paths[1]" end
    lastStop = tostring(rawget(p1, "Stop") or p1.Stop)
    if CFG.wEnd then setV(p1, "EndPosition", hitPos, fails) setV(p1, "EndDistance", dist, fails) end
    local steps = p1.Steps
    if type(steps) ~= "table" then return "no Steps" end
    local s1 = steps[1]
    if type(s1) ~= "table" and CFG.wBuild then
        local made = { Part = hitPart, Position = hitPos, Normal = -dir, ExitNormal = dir,
                       Distance = dist, ExitDistance = dist + 0.6, Human = true, Material = "Plastic" }
        if pcall(function() steps[1] = made end) then
            local okR, back = pcall(function() return steps[1] end)
            if okR and type(back) == "table" then s1 = back end
        end
    end
    if type(s1) ~= "table" then return "no Steps[1]" end
    if CFG.wPart  then setV(s1, "Part", hitPart, fails) end
    if CFG.wHuman then setV(s1, "Human", true, fails) end
    if CFG.wPos then
        setV(s1, "Position", hitPos, fails) setV(s1, "Normal", -dir, fails) setV(s1, "ExitNormal", dir, fails)
        setV(s1, "Distance", dist, fails) setV(s1, "ExitDistance", dist + 0.6, fails) setV(s1, "Material", "Plastic", fails)
    end
    local trimmed = 0
    if CFG.wTrim then
        for i = 2, 12 do
            if rawget(steps, i) ~= nil or steps[i] ~= nil then
                if pcall(function() steps[i] = nil end) then
                    local okR, back = pcall(function() return steps[i] end)
                    if okR and back == nil then trimmed = trimmed + 1 end
                end
            end
        end
    end
    if CFG.wStop then setV(p1, "Stop", CFG.stopValue, fails) end
    return ("trim %d  fails: %s"):format(trimmed, #fails > 0 and table.concat(fails, ",", 1, math.min(#fails,5)) or "none")
end

-- (Weapon Mods removed.)

-- FORCE ADS (packet-level, no input injection): flips the shoot packet's own aiming
-- flag(s) to true so the server treats every shot as aimed. Only touches fields that
-- already exist on the packet, so it can't corrupt it. If none match, dump a Fire
-- packet and I'll add the exact key.
local ADS_FIELDS = { "Aimed","Aiming","ADS","Ads","IsAiming","Scoped","Aim","AimingDown","AimedIn","IsAds" }
local function applyForceADS(data)
    for _, k in ipairs(ADS_FIELDS) do
        if rawget(data, k) ~= nil then pcall(function() rawset(data, k, true) end) end
    end
    local st = rawget(data, "Settings") or rawget(data, "State")
    if type(st) == "table" then
        for _, k in ipairs(ADS_FIELDS) do if rawget(st, k) ~= nil then pcall(function() rawset(st, k, true) end) end end
    end
end

local CV8
local function findRemote()
    pcall(function()
        local cr = RS:FindFirstChild("CLIENT_REMOTES")
        local gr = cr and cr:FindFirstChild("GUN_REMOTES")
        local cv = gr and gr:FindFirstChild("CommandV8")
        if cv then CV8 = cv end
    end)
    return CV8 ~= nil
end
-- retry until found, then keep re-verifying (handles late join + remote swap on TP/spawn)
task.spawn(function()
    while not findRemote() do task.wait(0.25) end
    while true do
        task.wait(2)
        pcall(function()
            local cr = RS:FindFirstChild("CLIENT_REMOTES")
            local gr = cr and cr:FindFirstChild("GUN_REMOTES")
            local cv = gr and gr:FindFirstChild("CommandV8")
            if cv and cv ~= CV8 then CV8 = cv end
        end)
    end
end)
LP.CharacterAdded:Connect(function() task.wait(1) findRemote() end)

local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)
mt.__namecall = newcclosure(function(self, ...)
    if self ~= CV8 then return old(self, ...) end
    if getnamecallmethod() ~= "FireServer" then return old(self, ...) end
    local packed = table.pack(...)
    local data = packed[1]
    if type(data) ~= "table" or data.Kind ~= "Fire" then return old(self, table.unpack(packed,1,packed.n)) end
    AR2_lastFireAt = os.clock()   -- fire signal for the hitmarker/killfeed hit detector (safe global write)
    if AR2_CFG.trigger.forceAds then pcall(applyForceADS, data) end
    -- feed bullet tracers on EVERY real shot (independent of silent aim / lock-on).
    -- WRAPPED IN PCALL: this is the only pre-FireServer code that isn't protected, so a weird
    -- packet must NEVER be able to throw here and drop the shot (that would kill ALL bullet reg).
    -- Pure table reads only -- no namecalls (would corrupt getnamecallmethod()).
    if AR2_tracer and AR2_tracer.on and AR2_shotFeed and AR2_shotFeed.q then
        pcall(function()
            if typeof(data.Origin) ~= "Vector3" then return end
            local a, b = data.Origin, nil
            if CFG.on and target.pos then b = target.pos
            else
                local p1 = (type(data.Paths) == "table") and data.Paths[1]
                local ep = p1 and p1.EndPosition
                if typeof(ep) == "Vector3" then b = ep
                elseif typeof(data.Direction) == "Vector3" and data.Direction.Magnitude > 0 then b = a + data.Direction.Unit * 2000 end
            end
            if b then local q = AR2_shotFeed.q q[#q+1] = { a = a, b = b } if #q > 32 then table.remove(q, 1) end end
        end)
    end
    local willForge = CFG.on and target.pos and target.part and typeof(data.Origin) == "Vector3"
    if not CFG.on then
        B.shots = B.shots + 1
        local before = snap()
        local r = old(self, table.unpack(packed,1,packed.n))
        confirm(before, B, "BASELINE")
        return r
    end
    A.shots = A.shots + 1
    local tp, tpart, og = target.pos, target.part, data.Origin
    if not tp or not tpart or typeof(og) ~= "Vector3" then
        local before = snap()
        local r = old(self, table.unpack(packed,1,packed.n))
        confirm(before, A, "noTarget")
        return r
    end
    local okF, info = pcall(forgePath, data, og, tp, tpart)
    lastInfo = okF and tostring(info) or ("ERR " .. tostring(info))
    local before = snap()
    local r = old(self, table.unpack(packed,1,packed.n))
    confirm(before, A, "FORGE " .. target.name)
    return r
end)
setreadonly(mt, true)

SA = CFG
SA_target = target
function SA_status() return A.shots, A.hits, target.name or "-", lastInfo or "-" end
end

-- =============================================
-- NO RECOIL / NO CAMERA SHAKE / NO BLOOM
-- Scans the garbage collector for the game's recoil/shake/bloom tables by
-- signature and neutralizes them + noop-hooks their kick functions.
-- Guarded so executors without getgc/hookfunction just skip it.
-- =============================================
do
local hasGC   = type(getgc) == "function"
local hasHook = type(hookfunction) == "function"
local RecoilTables, ShakerTables, BloomTables = {}, {}, {}
local HookedFunctions = {}
local LastScan = 0
local RecoilHookNames  = { "Kick", "KickADS", "Update" }
local ShakerMethodNames = { "Shake", "ShakeOnce", "StartShake", "ShakeSustain" }
local RecoilBigTimes = { ImpactTime=true, WeaponImpactTime=true, FOVTime=true, KickTimer=true, ShakeTime=true }
local RecoilNumberKeys = {
    "Strength","ShotIndex","ShotCount","Pitch","Yaw","Roll","Back",
    "PitchVelocity","YawVelocity","RollVelocity","BackVelocity",
    "ImpactPitch","ImpactYaw","ImpactRoll","ImpactBack","ImpactTime",
    "WeaponImpactPitch","WeaponImpactYaw","WeaponImpactRoll","WeaponImpactBack","WeaponImpactUp","WeaponImpactTime",
    "VMPitch","VMYaw","VMRoll","VMBack","VMUp",
    "VMPitchVelocity","VMYawVelocity","VMRollVelocity","VMBackVelocity","VMUpVelocity",
    "FOVAmount","FOVTime","LastFOVOffset","CameraPitch","CameraYaw","CameraBack","FOVOffset","VMOffset",
    "CurrentKick","CurrentPunch","KickTimer","ShakeTime",
}
local function FindFunctions(value, names, out)
    local cur, seen = value, {}
    for _ = 1, 5 do
        if type(cur) ~= "table" or seen[cur] then break end
        seen[cur] = true
        for _, k in ipairs(names) do
            if out[k] == nil then local fn = rawget(cur, k) if type(fn) == "function" then out[k] = fn end end
        end
        cur = rawget(cur, "__index")
    end
end
local function HookNoop(fn)
    -- DISABLED: replacing the game's recoil functions broke GunClient ("index nil with
    -- CameraBack"). This is now a no-op so nothing in the script ever calls hookfunction.
    return
end
local function ScanRecoilObjects()
    if not hasGC then return end
    RecoilTables, ShakerTables, BloomTables = {}, {}, {}
    for _, v in ipairs(getgc(true)) do
        if type(v) == "table" then
            local a, b, c = rawget(v,"ShotIndex"), rawget(v,"PitchVelocity"), rawget(v,"YawVelocity")
            if type(a)=="number" and type(b)=="number" and type(c)=="number" then RecoilTables[#RecoilTables+1] = v end
            if rawget(v,"_camShakeInstances") ~= nil and rawget(v,"_removeInstances") ~= nil then ShakerTables[#ShakerTables+1] = v end
            local fa, afa = rawget(v,"FireAmount"), rawget(v,"ADSFireAmount")
            if type(fa)=="number" and type(afa)=="number" and type(rawget(v,"RecoverySpeed"))=="number" then BloomTables[#BloomTables+1] = v end
        end
    end
end
local function Neutralize()
    -- Zero the recoil fields every frame. We DELIBERATELY do not replace ("noop-hook") the
    -- game's Kick/Update/Shake functions any more -- one of them returns the camera object
    -- GunClient reads, so nil-ing it caused the "index nil with CameraBack" crash flood.
    -- Field-zeroing alone kills recoil for this game's value-driven recoil system.
    for _, v in ipairs(RecoilTables) do
        if type(v) == "table" then
            for _, k in ipairs(RecoilNumberKeys) do
                local val = rawget(v, k)
                if type(val) == "number" then
                    if RecoilBigTimes[k] then if val > 0 and val < 1e9 then rawset(v, k, 1e9) end
                    elseif val ~= 0 then rawset(v, k, 0) end
                end
            end
        end
    end
    -- camera shake: clear its instance lists (data only, no function replacement)
    for _, v in ipairs(ShakerTables) do
        local insts = rawget(v, "_camShakeInstances") if type(insts)=="table" then table.clear(insts) end
        local rems = rawget(v, "_removeInstances") if type(rems)=="table" then table.clear(rems) end
    end
    -- weapon bloom: zero the offset field
    for _, v in ipairs(BloomTables) do
        local off = rawget(v, "Offset") if type(off)=="number" and off ~= 0 then rawset(v, "Offset", 0) end
    end
end
-- No Recoil was part of Weapon Mods, which has been removed -- the scan/neutralize loop no
-- longer runs. (Scaffolding above is left inert; nothing calls it.)
end


-- =============================================
-- HIT DETECTOR  (feeds the hitmarker + killfeed). Watches the health of nearby humanoids --
-- PLAYERS **and ZOMBIES** -- and, when one drops health within ~0.5s of you firing, bumps the
-- shared hit feed. The old path only tracked players, so the hitmarker never fired on zombies
-- (which is what you actually shoot). Runs OUTSIDE the fire hook so its namecalls are safe.
-- =============================================
AR2_lastFireAt = AR2_lastFireAt or 0
do
    local prevH = setmetatable({}, { __mode = "k" })   -- weak keys: dead humanoids get GC'd
    task.spawn(function()
        while true do
            local firedRecently = (os.clock() - (AR2_lastFireAt or 0)) < 0.55
            local list = {}
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= LP and pl.Character then
                    local h = pl.Character:FindFirstChildOfClass("Humanoid")
                    if h then list[#list+1] = { hum = h, name = pl.Name } end
                end
            end
            local zc = getZombieContainer and getZombieContainer()
            if zc then
                for _, m in ipairs(zc:GetChildren()) do
                    if m:IsA("Model") then
                        local h = m:FindFirstChildOfClass("Humanoid")
                        if h then list[#list+1] = { hum = h, name = m.Name } end
                    end
                end
            end
            for _, e in ipairs(list) do
                local h = e.hum
                local prev = prevH[h]
                local cur = h.Health
                if prev ~= nil and cur < prev - 0.5 and firedRecently and AR2_HITFEED then
                    AR2_HITFEED.n = AR2_HITFEED.n + 1
                    AR2_HITFEED.name = e.name or "?"
                    AR2_HITFEED.dmg = prev - cur
                    AR2_HITFEED.killed = (cur <= 0)
                    AR2_HITFEED.at = os.clock()
                end
                prevH[h] = cur
            end
            task.wait(0.05)
        end
    end)
end

-- =============================================
-- HITMARKER  (selectable sound pack; sounds parented to a workspace part so
-- 3D positional playback works in games that mute SoundService hits)
-- =============================================
-- "Ping" is a BUILT-IN Roblox sound (rbxasset) -- it always loads and plays, unlike marketplace
-- rbxassetid sounds which can be moderated/fail to load (that's why the hitmarker was silent).
AR2_HITMARK_SOUNDS = {
    ["Ping"]     = "rbxasset://sounds/electronicpingshort.wav",
    ["Tick"]     = "rbxassetid://3744371342",
    ["Click"]    = "rbxassetid://186809061",
    ["Thud"]     = "rbxassetid://2648562756",
    ["Ding"]     = "rbxassetid://8394333801",
    ["Pop"]      = "rbxassetid://5178271747",
}
AR2_hitmark = AR2_CFG.hitmark
AR2_playHitSound = function() end
do
    -- host one copy on a workspace part (some games gag SoundService) and one on SoundService (2D).
    local holder = Instance.new("Part")
    holder.Name = "oh_hm_holder" holder.Anchored = true holder.CanCollide = false
    holder.Transparency = 1 holder.Size = Vector3.new(1,1,1)
    pcall(function() holder.Parent = workspace end)
    local snd = Instance.new("Sound")
    snd.Volume = 0.5 snd.RollOffMinDistance = 10 snd.RollOffMaxDistance = 100000
    pcall(function() snd.Parent = holder end)
    local snd2 = Instance.new("Sound")
    snd2.Volume = 0.5
    pcall(function() snd2.Parent = SoundSvc end)
    -- resolve a custom ID like the script you shared: a raw number -> rbxassetid://N, or a full
    -- rbxassetid:// / rbxasset:// URL passed through. If set, it overrides the dropdown pick.
    local function resolveId(input)
        if not input or input == "" then return nil end
        if input:find("rbxassetid://", 1, true) or input:find("rbxasset://", 1, true) then return input end
        local digits = input:gsub("%D", "")
        if digits ~= "" then return "rbxassetid://" .. digits end
        return nil
    end
    local function refreshSound()
        local id = resolveId(AR2_CFG.hitmark.customId) or AR2_HITMARK_SOUNDS[AR2_CFG.hitmark.sound] or AR2_HITMARK_SOUNDS["Ping"]
        if snd.SoundId ~= id then snd.SoundId = id end
        if snd2.SoundId ~= id then snd2.SoundId = id end
    end
    refreshSound()
    -- ONE reusable play fn (debounced), called by BOTH the health detector AND the game-hitmarker
    -- hook, so whichever notices the hit first makes the noise; debounce stops double-plays.
    local lastPlay = 0
    function AR2_playHitSound()
        if not AR2_CFG.hitmark.on then return end
        local now = os.clock()
        if now - lastPlay < 0.04 then return end
        lastPlay = now
        pcall(function()
            refreshSound()
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if hrp and holder.Parent then holder.CFrame = hrp.CFrame end
            local v = AR2_CFG.hitmark.volume or 0.5
            snd.Volume = v snd2.Volume = v
            snd.TimePosition = 0 snd2.TimePosition = 0
            snd:Play() snd2:Play()
        end)
    end
    local lastHits = AR2_HITFEED and AR2_HITFEED.n or 0
    task.spawn(function()
        while true do
            local hits = AR2_HITFEED and AR2_HITFEED.n or 0
            if hits > lastHits then lastHits = hits AR2_playHitSound() end
            task.wait(0.03)
        end
    end)
end

-- =============================================
-- GAME HITMARKER HOOK: play our sound the instant the GAME's own hitmarker fires -- the most
-- reliable trigger, since the game knows exactly when you land a hit (no ping/timing guess).
-- Best-effort: watches PlayerGui for a hitmarker-like GuiObject spawning or flashing visible.
-- =============================================
do
    local KW = { "hitmark", "hitmarker", "damageindicator", "hit_marker", "hitindicator", "hitfx" }
    local function looksLikeHM(obj)
        if not obj:IsA("GuiObject") then return false end
        local n = (obj.Name or ""):lower()
        for _, k in ipairs(KW) do if n:find(k, 1, true) then return true end end
        return false
    end
    local seen = setmetatable({}, { __mode = "k" })
    local function watch(obj, isNew)
        if seen[obj] or not looksLikeHM(obj) then return end
        seen[obj] = true
        if isNew then pcall(AR2_playHitSound) end   -- a freshly-spawned hitmarker == a hit
        pcall(function() obj:GetPropertyChangedSignal("Visible"):Connect(function() if obj.Visible then pcall(AR2_playHitSound) end end) end)
        if obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
            pcall(function() obj:GetPropertyChangedSignal("ImageTransparency"):Connect(function() if obj.ImageTransparency < 0.9 then pcall(AR2_playHitSound) end end) end)
        end
    end
    task.spawn(function()
        local PG = nil
        while not PG do PG = LP:FindFirstChildOfClass("PlayerGui") if not PG then task.wait(0.5) end end
        for _, d in ipairs(PG:GetDescendants()) do pcall(watch, d, false) end
        PG.DescendantAdded:Connect(function(d) pcall(watch, d, true) end)
        -- re-scan on respawn (PlayerGui children can get rebuilt)
        LP.CharacterAdded:Connect(function() task.wait(1) pcall(function() for _, d in ipairs(PG:GetDescendants()) do pcall(watch, d, false) end end) end)
    end)
end

-- =============================================
-- FAKE KILLFEED  (client-side "Killed <name>" popups on confirmed kills; watches the shared
-- hit feed so it works with silent aim on OR off. Scoped in a do-block = no main-chunk registers.)
-- =============================================
do
    local gui, layout, order = nil, nil, 0
    local function ensureGui()
        if gui and gui.Parent then return gui end
        gui = Instance.new("ScreenGui")
        gui.Name = "oh_killfeed" gui.ResetOnSpawn = false gui.IgnoreGuiInset = true gui.DisplayOrder = 99999
        pcall(function() gui.Parent = getMainHost() end)
        local holder = Instance.new("Frame")
        holder.Name = "list" holder.AnchorPoint = Vector2.new(1, 0)
        holder.Position = UDim2.new(1, -18, 0, 130) holder.Size = UDim2.new(0, 360, 0, 420)
        holder.BackgroundTransparency = 1 holder.Parent = gui
        layout = Instance.new("UIListLayout")
        layout.SortOrder = Enum.SortOrder.LayoutOrder layout.HorizontalAlignment = Enum.HorizontalAlignment.Right
        layout.Padding = UDim.new(0, 5) layout.Parent = holder
        return gui
    end
    local function push(verb, name, col)
        local g = ensureGui()
        local holder = g and g:FindFirstChild("list")
        if not holder then return end
        order = order + 1
        local lbl = Instance.new("TextLabel")
        lbl.LayoutOrder = -order lbl.AnchorPoint = Vector2.new(1, 0)
        lbl.Size = UDim2.new(0, 320, 0, 28) lbl.AutomaticSize = Enum.AutomaticSize.X
        lbl.BackgroundColor3 = Color3.fromRGB(14, 14, 16) lbl.BackgroundTransparency = 0.2 lbl.BorderSizePixel = 0
        lbl.Font = Enum.Font.GothamBold lbl.TextSize = 15 lbl.TextXAlignment = Enum.TextXAlignment.Right
        lbl.RichText = true lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        lbl.Text = ('<font color="%s">%s</font>  <font color="#ffffff">%s</font>'):format(col, verb, tostring(name))
        lbl.Parent = holder
        Instance.new("UICorner", lbl).CornerRadius = UDim.new(0, 6)
        local pad = Instance.new("UIPadding", lbl) pad.PaddingRight = UDim.new(0, 12) pad.PaddingLeft = UDim.new(0, 12)
        local stroke = Instance.new("UIStroke", lbl) stroke.Color = Color3.fromRGB(255, 80, 80) stroke.Thickness = 1 stroke.Transparency = 0.35
        task.delay(3.0, function()
            pcall(function()
                for i = 1, 12 do
                    if not lbl or not lbl.Parent then return end
                    local a = i / 12
                    lbl.TextTransparency = a lbl.BackgroundTransparency = 0.2 + a * 0.8 stroke.Transparency = 0.35 + a * 0.65
                    task.wait(0.03)
                end
                if lbl then lbl:Destroy() end
            end)
        end)
    end
    local seen = AR2_HITFEED and AR2_HITFEED.n or 0
    task.spawn(function()
        while true do
            local hf = AR2_HITFEED
            local n = hf and hf.n or 0
            if n > seen then
                local killed, nm = hf.killed, hf.name
                seen = n
                local kf = AR2_CFG.killfeed
                if kf and kf.on and killed then pcall(push, "☠ Killed", nm, "#ff5555") end
            end
            task.wait(0.05)
        end
    end)
end

-- =============================================
-- SPIN BOT
-- =============================================
AR2_spin = { on = false, speed = 20, tilt = false }
do
    local joint, baseC0, angle = nil, nil, 0
    local function findJoint()
        local ch = LP.Character
        if not ch then return nil end
        local lt = ch:FindFirstChild("LowerTorso")
        if lt then for _, d in ipairs(lt:GetChildren()) do if d:IsA("Motor6D") then return d end end end
        local hrp = ch:FindFirstChild("HumanoidRootPart")
        if hrp then local rj = hrp:FindFirstChild("RootJoint") if rj and rj:IsA("Motor6D") then return rj end end
        return nil
    end
    function AR2_setSpin(on)
        AR2_spin.on = on
        if not on then
            if joint and joint.Parent and baseC0 then pcall(function() joint.C0 = baseC0 end) end
            joint, baseC0 = nil, nil
        end
    end
    LP.CharacterAdded:Connect(function() joint, baseC0 = nil, nil end)
    pcall(function()
        RunSvc:BindToRenderStep("AR2_SpinBot", Enum.RenderPriority.Character.Value + 1, function(dt)
            if not AR2_spin.on then return end
            if not joint or not joint.Parent then joint = findJoint() baseC0 = joint and joint.C0 or nil end
            if not joint or not baseC0 then return end
            angle = (angle + AR2_spin.speed * dt) % (math.pi * 2)
            local cf = CFrame.Angles(0, angle, 0)
            if AR2_spin.tilt then cf = cf * CFrame.Angles(math.rad(25) * math.sin(angle * 2), 0, 0) end
            pcall(function() joint.C0 = baseC0 * cf end)
        end)
    end)
end

-- =============================================
-- BULLET TRACERS
-- =============================================
AR2_tracer = { on = false, life = 0.45, thickness = 2, color = Color3.fromRGB(120, 220, 255), style = "2D Line" }
AR2_shotFeed = AR2_shotFeed or { q = {} }
if not AR2_shotFeed.q then AR2_shotFeed.q = {} end
do
    local MAX = 24
    local lines, shots = {}, {}
    local nextSlot = 1
    for i = 1, MAX do
        local l = Drawing.new("Line")
        l.Thickness = 2 l.Color = AR2_tracer.color l.Transparency = 1 l.Visible = false
        lines[i] = l shots[i] = nil
    end
    -- 3D Part style: spawn a neon beam-part in the world along the shot and fade it out
    local function make3D(a, b)
        local d = (b - a).Magnitude
        if d < 0.1 then return end
        local p = Instance.new("Part")
        p.Anchored = true p.CanCollide = false p.CanQuery = false p.CanTouch = false
        p.Material = Enum.Material.Neon p.Color = AR2_tracer.color
        local w = math.max(0.05, (AR2_tracer.thickness or 2) * 0.06)
        p.Size = Vector3.new(w, w, d)
        p.CFrame = CFrame.lookAt(a, b) * CFrame.new(0, 0, -d/2)
        p.Parent = workspace
        task.spawn(function()
            local steps = 14
            for i = 1, steps do
                if not p.Parent then return end
                pcall(function() p.Transparency = i / steps end)
                task.wait((AR2_tracer.life or 0.45) / steps)
            end
            pcall(function() p:Destroy() end)
        end)
    end
    -- drain the shot feed (populated by the fire hook on EVERY real shot) and spawn a tracer
    -- for each. This no longer depends on silent aim / lock-on -- tracers fire whenever you shoot.
    task.spawn(function()
        while true do
            local q = AR2_shotFeed and AR2_shotFeed.q
            if q then
                if AR2_tracer.on then
                    while #q > 0 do
                        local s = table.remove(q, 1)
                        if s and s.a and s.b then
                            if AR2_tracer.style == "3D Part" then make3D(s.a, s.b)
                            else shots[nextSlot] = { a = s.a, b = s.b, born = os.clock() } nextSlot = nextSlot % MAX + 1 end
                        end
                    end
                else
                    for i = #q, 1, -1 do q[i] = nil end   -- discard buffered shots while off
                end
            end
            task.wait(0.02)
        end
    end)
    RunSvc.RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        local now = os.clock()
        for i = 1, MAX do
            local s, l = shots[i], lines[i]
            if s and AR2_tracer.on and AR2_tracer.style ~= "3D Part" then
                local age = now - s.born
                if age >= AR2_tracer.life then shots[i] = nil l.Visible = false
                else
                    local sa, oa = cam:WorldToViewportPoint(s.a)
                    local sb, ob = cam:WorldToViewportPoint(s.b)
                    if oa and ob and sa.Z > 0 and sb.Z > 0 then
                        l.From = Vector2.new(sa.X, sa.Y) l.To = Vector2.new(sb.X, sb.Y)
                        l.Color = AR2_tracer.color l.Thickness = AR2_tracer.thickness
                        l.Transparency = 1 - (age / AR2_tracer.life) l.Visible = true
                    else l.Visible = false end
                end
            elseif l.Visible then l.Visible = false end
        end
    end)
end

-- =============================================
-- TUNG
-- =============================================
local TAN = Color3.fromRGB(210, 163, 100)
local TUNG_BODY = { "Head","UpperTorso","LowerTorso","LeftUpperArm","LeftLowerArm","LeftHand","RightUpperArm","RightLowerArm","RightHand","LeftUpperLeg","LeftLowerLeg","LeftFoot","RightUpperLeg","RightLowerLeg","RightFoot" }
local tungOriginalParts = {}
local tungGlobalConns = {}
local tungSelfActive = false
local tungGlobalActive = false
local function tungWeld(p0,p1) local w = Instance.new("WeldConstraint") w.Part0=p0 w.Part1=p1 w.Parent=p1 end
local function tungMakePart(props)
    local p = Instance.new("Part")
    p.Name = props.name or "TungPart" p.Size = props.size or Vector3.new(1,1,1)
    p.Color = props.color or TAN p.Material = props.material or Enum.Material.SmoothPlastic
    p.Transparency = props.transparency or 0 p.Anchored=false p.CanCollide=false p.Massless=true p.CastShadow=false
    if props.shape then p.Shape = props.shape end
    p.Parent = props.parent p.CFrame = props.cframe or CFrame.new()
    return p
end
local function tungClear(char)
    if not char then return end
    for _, d in ipairs(char:GetDescendants()) do if d.Name:sub(1,4)=="Tung" then pcall(function() d:Destroy() end) end end
end
local function tungApply(char)
    if not char then return end
    tungClear(char)
    for _, name in ipairs(TUNG_BODY) do
        local part = char:FindFirstChild(name)
        if part and part:IsA("BasePart") then
            if char == LP.Character and not tungOriginalParts[part] then
                tungOriginalParts[part] = { Color=part.Color, Material=part.Material, Transparency=part.Transparency }
            end
            part.Material = Enum.Material.ForceField part.Color = TAN part.Transparency = 0.5
        end
    end
    local head = char:FindFirstChild("Head")
    local rightHand = char:FindFirstChild("RightHand") or char:FindFirstChild("Right Arm") or char:FindFirstChild("RightLowerArm")
    if head then
        local function makeEye(name, xOffset)
            local eye = tungMakePart({ name=name, shape=Enum.PartType.Ball, size=Vector3.new(0.3,0.3,0.3), color=Color3.fromRGB(255,255,255), parent=char, cframe=head.CFrame*CFrame.new(xOffset,0.22,-0.53) })
            local pupil = tungMakePart({ name=name.."Pupil", shape=Enum.PartType.Ball, size=Vector3.new(0.13,0.13,0.13), color=Color3.new(0,0,0), parent=char, cframe=eye.CFrame*CFrame.new(0,0,-0.12) })
            tungWeld(head,eye) tungWeld(eye,pupil)
        end
        makeEye("TungLeftEye",-0.22) makeEye("TungRightEye",0.22)
    end
    if rightHand then
        local hl,hw,bl,bw,th = 2.2,0.18,1.0,0.55,0.30
        local handle = tungMakePart({ name="TungBatHandle", size=Vector3.new(hw,hl,hw), color=Color3.fromRGB(88,52,12), material=Enum.Material.Wood, parent=char, cframe=rightHand.CFrame*CFrame.new(0,-(hl/2),0) })
        local barrel = tungMakePart({ name="TungBatBarrel", size=Vector3.new(bw,bl,bw), color=Color3.fromRGB(88,52,12), material=Enum.Material.Wood, parent=char, cframe=handle.CFrame*CFrame.new(0,-(hl/2+bl/2),0) })
        local tape = tungMakePart({ name="TungBatTape", size=Vector3.new(hw+0.06,th,hw+0.06), color=Color3.fromRGB(20,20,20), parent=char, cframe=handle.CFrame*CFrame.new(0,hl/2-th/2-0.1,0) })
        tungWeld(rightHand,handle) tungWeld(handle,barrel) tungWeld(handle,tape)
    end
end
local function tungRemove(char)
    tungClear(char)
    if char == LP.Character then
        for part, orig in pairs(tungOriginalParts) do if part and part.Parent then part.Color=orig.Color part.Material=orig.Material part.Transparency=orig.Transparency end end
        table.clear(tungOriginalParts)
    end
end
local function tungApplyGlobal(on)
    for _, conn in pairs(tungGlobalConns) do pcall(function() conn:Disconnect() end) end
    tungGlobalConns = {}
    if not on then for _, p in ipairs(Players:GetPlayers()) do if p ~= LP and p.Character then tungRemove(p.Character) end end return end
    for _, p in ipairs(Players:GetPlayers()) do if p ~= LP and p.Character then tungApply(p.Character) end end
    for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then tungGlobalConns[p] = p.CharacterAdded:Connect(function(c) task.wait(0.5) if tungGlobalActive then tungApply(c) end end) end end
    tungGlobalConns["added"] = Players.PlayerAdded:Connect(function(p) tungGlobalConns[p] = p.CharacterAdded:Connect(function(c) task.wait(0.5) if tungGlobalActive then tungApply(c) end end) end)
end
local function setTungSelf(on) tungSelfActive=on AR2_CFG.misc.tungSelf=on if on then tungApply(LP.Character) else tungRemove(LP.Character) end end
local function setTungGlobal(on) tungGlobalActive=on AR2_CFG.players.tungGlobal=on tungApplyGlobal(on) end
LP.CharacterAdded:Connect(function(c) task.wait(0.5) if tungSelfActive then tungApply(c) end end)

-- =============================================
-- PERSONAL CHAMS
-- =============================================
local PERSONAL_PARTS = { "Head","UpperTorso","LowerTorso","LeftUpperArm","LeftLowerArm","LeftHand","RightUpperArm","RightLowerArm","RightHand","LeftUpperLeg","LeftLowerLeg","LeftFoot","RightUpperLeg","RightLowerLeg","RightFoot" }
local personalCharOriginal = {}
local personalCharConn = nil
local personalGunOriginal = {}
local personalGunConn = nil
local gunHighlights = {}
local gunTick = 0
local function getPersonalColor(hue, gray, mode, rainbow)
    if rainbow then hue = (tick()*100) % 360 end
    if mode == "BlackWhite" then local g = gray/100 return Color3.fromRGB(g*255,g*255,g*255) end
    return Color3.fromHSV(hue/360, 1, 1)
end
local function restorePersonalChar()
    for part, orig in pairs(personalCharOriginal) do if part and part.Parent then pcall(function() part.Color=orig.Color part.Material=orig.Material part.Transparency=orig.Transparency end) end end
    personalCharOriginal = {}
end
local function applyPersonalChar()
    local char = LP.Character if not char then return end
    local mat = Enum.Material[AR2_CFG.misc.personalCharMaterial] or Enum.Material.ForceField
    local col = getPersonalColor(AR2_CFG.misc.personalCharHue, AR2_CFG.misc.personalCharGray, AR2_CFG.misc.personalCharMode, AR2_CFG.misc.personalCharRainbow)
    local trans = (mat == Enum.Material.ForceField) and 0.5 or 0
    for _, name in ipairs(PERSONAL_PARTS) do
        local part = char:FindFirstChild(name)
        if part and part:IsA("BasePart") then
            if not personalCharOriginal[part] then personalCharOriginal[part] = { Color=part.Color, Material=part.Material, Transparency=part.Transparency } end
            pcall(function() part.Material=mat end) pcall(function() part.Color=col end) pcall(function() part.Transparency=trans end)
        end
    end
end
local function setPersonalChar(enabled)
    AR2_CFG.misc.personalCharOn = enabled
    if personalCharConn then personalCharConn:Disconnect() personalCharConn = nil end
    if not enabled then restorePersonalChar() return end
    task.wait(0.3)
    applyPersonalChar()
    personalCharConn = RunSvc.RenderStepped:Connect(function()
        if not AR2_CFG.misc.personalCharOn then return end
        local char = LP.Character if not char then return end
        local wantMat = Enum.Material[AR2_CFG.misc.personalCharMaterial] or Enum.Material.ForceField
        local needsWork = false
        for _, name in ipairs(PERSONAL_PARTS) do
            local part = char:FindFirstChild(name)
            if part and part:IsA("BasePart") then
                if not personalCharOriginal[part] or part.Material ~= wantMat then needsWork = true break end
            end
        end
        if needsWork or AR2_CFG.misc.personalCharRainbow then applyPersonalChar() end
    end)
end
local function findLocalGun()
    local char = LP.Character if not char then return nil, "no char" end
    local equipped = char:FindFirstChild("Equipped")
    if equipped then for _, child in ipairs(equipped:GetChildren()) do if child:IsA("Model") or child:IsA("Tool") then return child, "Equipped/"..child.Name end end end
    for _, child in ipairs(char:GetChildren()) do if child:IsA("Tool") then return child, "char-tool/"..child.Name end end
    local cam = workspace.CurrentCamera
    if cam then
        for _, child in ipairs(cam:GetChildren()) do if child:IsA("Model") then return child, "camera/"..child.Name end end
        for _, child in ipairs(cam:GetDescendants()) do if child:IsA("Model") and child:FindFirstChildWhichIsA("BasePart") then return child, "camera-nested/"..child.Name end end
    end
    return nil, "not found"
end
local function restorePersonalGun()
    for part, orig in pairs(personalGunOriginal) do if part and part.Parent then pcall(function() part.Color=orig.Color part.Material=orig.Material end) end end
    personalGunOriginal = {}
    for gun, h in pairs(gunHighlights) do if h and h.Parent then pcall(function() h:Destroy() end) end end
    gunHighlights = {}
end
local function applyPersonalGun()
    local gun, why = findLocalGun()
    if not gun then return end
    local mat = Enum.Material[AR2_CFG.misc.personalGunMaterial] or Enum.Material.Neon
    local col = getPersonalColor(AR2_CFG.misc.personalGunHue, AR2_CFG.misc.personalGunGray, AR2_CFG.misc.personalGunMode, AR2_CFG.misc.personalGunRainbow)
    local cam = workspace.CurrentCamera
    local isViewModel = false
    if cam then local n = gun while n and n ~= game do if n == cam then isViewModel = true break end n = n.Parent end end
    for _, part in ipairs(gun:GetDescendants()) do
        if part:IsA("BasePart") then
            if not personalGunOriginal[part] then personalGunOriginal[part] = { Color = part.Color, Material = part.Material } end
            part.Material = mat part.Color = col
        end
    end
    if isViewModel then return end
    if not gunHighlights[gun] or not gunHighlights[gun].Parent then
        local h = Instance.new("Highlight")
        h.Name = "oh_gun_hl" h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.FillTransparency = 0.35 h.OutlineTransparency = 0 h.OutlineColor = Color3.new(1,1,1)
        h.Adornee = gun h.Parent = gun gunHighlights[gun] = h
    end
    local hl = gunHighlights[gun]
    if hl then hl.FillColor = col hl.Enabled = true end
end
local function setPersonalGun(enabled)
    AR2_CFG.misc.personalGunOn = enabled
    if personalGunConn then personalGunConn:Disconnect() personalGunConn = nil end
    if not enabled then restorePersonalGun() return end
    applyPersonalGun()
    personalGunConn = RunSvc.Heartbeat:Connect(function()
        if not AR2_CFG.misc.personalGunOn then return end
        local now = os.clock()
        if now - gunTick < 1/30 then return end
        gunTick = now applyPersonalGun()
    end)
end
LP.CharacterAdded:Connect(function(c)
    task.spawn(function()
        c:WaitForChild("HumanoidRootPart", 10)
        personalCharOriginal = {} personalGunOriginal = {}
        for _ = 1, 6 do
            task.wait(0.4)
            if AR2_CFG.misc.personalCharOn then pcall(applyPersonalChar) end
            if AR2_CFG.misc.personalGunOn  then pcall(applyPersonalGun)  end
        end
    end)
end)

-- =============================================
-- WALKSPEED
-- =============================================
AR2_setWalkSpeed = nil
do
    local conn = nil
    function AR2_setWalkSpeed(on)
        AR2_CFG.move.walkOn = on
        if conn then conn:Disconnect() conn = nil end
        if not on then
            local h = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
            if h then pcall(function() h.WalkSpeed = 16 end) end
            return
        end
        conn = RunSvc.Heartbeat:Connect(function()
            if not AR2_CFG.move.walkOn then return end
            local h = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
            if h and h.WalkSpeed ~= AR2_CFG.move.walkSpeed then
                pcall(function() h.WalkSpeed = AR2_CFG.move.walkSpeed end)
            end
        end)
    end
end

-- =============================================
-- AIM / TRIGGER TARGETING
-- =============================================
local function findAimTarget()
    local centre = Vector2.new(Cam.ViewportSize.X/2, Cam.ViewportSize.Y/2)
    local camPos = Cam.CFrame.Position
    local best, bestScore = nil, AR2_CFG.aim.fov
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LP and pl.Character then
            local hum = pl.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local part = resolveTargetPart(pl.Character)
                if part and d3(part.Position, camPos) <= AR2_CFG.aim.maxDist then
                    local sp, onScr = Cam:WorldToViewportPoint(part.Position)
                    if onScr and sp.Z > 0 then
                        local flat = (Vector2.new(sp.X, sp.Y) - centre).Magnitude
                        if flat < bestScore then bestScore, best = flat, part end
                    end
                end
            end
        end
    end
    return best
end
local function findTarget360()
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    local best, bestScore = nil, math.huge
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LP and pl.Character then
            local hum = pl.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local part = resolveTargetPart(pl.Character)
                if part then
                    local d = (part.Position - hrp.Position).Magnitude
                    local score = d / math.max(hum.Health, 1)
                    if score < bestScore then bestScore, best = score, part end
                end
            end
        end
    end
    return best
end
local function findTriggerTarget()
    local centre = Vector2.new(Cam.ViewportSize.X/2, Cam.ViewportSize.Y/2)
    local camPos = Cam.CFrame.Position
    local best, bestScore = nil, AR2_CFG.trigger.fov
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LP and pl.Character then
            local hum = pl.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local part = resolveTargetPart(pl.Character)
                if part and d3(part.Position, camPos) <= AR2_CFG.trigger.maxDist then
                    if not AR2_CFG.trigger.visibleOnly or isVisible(pl.Character, part) then
                        local sp, onScr = Cam:WorldToViewportPoint(part.Position)
                        if onScr and sp.Z > 0 then
                            local flat = (Vector2.new(sp.X, sp.Y) - centre).Magnitude
                            if flat < bestScore then bestScore, best = flat, part end
                        end
                    end
                end
            end
        end
    end
    return best
end
local function findTriggerTarget360() return findTarget360() end
local function toolReady()
    local char = LP.Character
    if not char then return false end
    local anim = char:FindFirstChildOfClass("Animator") or char:FindFirstChildOfClass("Humanoid")
    if not anim then return false end
    local hasTool = false
    for _, c in ipairs(char:GetChildren()) do if c:IsA("Tool") then hasTool = true break end end
    local eq = char:FindFirstChild("Equipped")
    if eq then for _, c in ipairs(eq:GetChildren()) do if c:IsA("Model") or c:IsA("Tool") then hasTool = true break end end end
    return hasTool
end
local trigLastShot, trigFiring, trigReleaseAt = 0, false, 0
local function trigRelease() pcall(function() VIM:SendMouseButtonEvent(0,0,0,false,game,1) end) trigFiring=false end
local function trigPress() pcall(function() VIM:SendMouseButtonEvent(0,0,0,true,game,1) end) trigFiring=true end

local rapidBusy = false
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if not AR2_CFG.misc.rapidFire then return end
    if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
    if rapidBusy then return end
    rapidBusy = true
    task.spawn(function()
        task.wait(0.05)
        for _ = 1, AR2_CFG.misc.rapidShots do
            task.wait(AR2_CFG.misc.rapidDelay)
            pcall(function()
                local cx = Cam.ViewportSize.X / 2
                local cy = Cam.ViewportSize.Y / 2
                VIM:SendMouseButtonEvent(cx, cy, 0, true, game, 1)
                task.wait(0.02)
                VIM:SendMouseButtonEvent(cx, cy, 0, false, game, 1)
            end)
        end
        rapidBusy = false
    end)
end)

-- =============================================
-- NOCLIP
-- =============================================
AR2_noclipConn = nil
function AR2_startNoclip()
    if AR2_noclipConn then return end
    AR2_CFG.misc.noclip = true
    AR2_noclipConn = task.spawn(function()
        while AR2_CFG.misc.noclip do
            pcall(function()
                local char = LP.Character
                if char then for _, part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then part.CanCollide = false end end end
            end)
            task.wait(0.1)
        end
        AR2_noclipConn = nil
    end)
end
function AR2_stopNoclip()
    AR2_CFG.misc.noclip = false
    AR2_noclipConn = nil
    pcall(function()
        local char = LP.Character
        if not char then return end
        for _, part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then part.CanCollide = true end end
        local h = char:FindFirstChildWhichIsA("Humanoid")
        if h then h.PlatformStand = false h.Sit = false end
    end)
end
function AR2_noclipActive() return AR2_CFG.misc.noclip == true end

UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if not AR2_CFG.misc.infJump then return end
    if input.KeyCode ~= Enum.KeyCode.Space then return end
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    task.spawn(function()
        if not hrp.Parent then return end
        hrp.Anchored = true hrp.CFrame = hrp.CFrame + Vector3.new(0,15,0) task.wait(0.03)
        if hrp and hrp.Parent then hrp.Anchored = false end
    end)
end)

-- =============================================
-- TELEPORT
-- =============================================
local TP = { active = false, phase = "idle" }

function AR2_tpBusy() return TP.active end
function AR2_tpPhase() return TP.phase end
local function tpRoot() local char = LP.Character return char and char:FindFirstChild("HumanoidRootPart"), char end
local function resolveDest(opts)
    if opts.player and opts.player.Parent then
        local ch = opts.player.Character
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if hrp then local off = hrp.CFrame.LookVector * 3 + Vector3.new(0, AR2_CFG.tp.tpOffsetY or 4, 0) return hrp.Position + off end
        return nil
    end
    if opts.part and opts.part.Parent then                 -- live-tracking a moving target (e.g. a horse)
        return opts.part.Position + Vector3.new(0, AR2_CFG.tp.tpOffsetY or 4, 0)
    end
    return opts.pos
end
function AR2_teleportStart(opts)
    opts = opts or {}
    if TP.active then return false end
    if not opts.player and not opts.pos and not opts.part then return false end
    local root, char = tpRoot()
    if not root or not char then return false end
    TP.active = true
    task.spawn(function()
        local ok, err = pcall(function()
            local hum = char:FindFirstChildWhichIsA("Humanoid")
            local wasNoclip = AR2_CFG.misc.noclip
            TP.phase = "dropping"
            AR2_CFG.misc.noclip = true AR2_startNoclip()
            if hum then hum.PlatformStand = true end
            tpBeam.Visible = false
            local fall = AR2_CFG.tp.fallTime or 2.5
            local t0 = os.clock()
            while os.clock() - t0 < fall do
                if not TP.active then break end
                local r = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                if not r then break end
                task.wait(0.05)
            end
            TP.phase = "arriving"
            local dest = resolveDest(opts)
            local hold = AR2_CFG.tp.holdTime or 0.8
            local arrived = false
            for attempt = 1, (AR2_CFG.tp.retries or 3) do
                if not TP.active then break end
                dest = resolveDest(opts) or dest
                if not dest then break end
                local h0 = os.clock()
                while os.clock() - h0 < hold do
                    if not TP.active then break end
                    local r = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                    if not r then break end
                    r.CFrame = CFrame.new(dest)
                    r.AssemblyLinearVelocity = Vector3.zero r.AssemblyAngularVelocity = Vector3.zero
                    task.wait()
                end
                local r = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                if r and dest and (r.Position - dest).Magnitude < 25 then arrived = true TP.phase = "arrived" break end
                TP.phase = ("retry %d"):format(attempt)
                task.wait(0.15)
            end
            if not arrived then TP.phase = "reanchored" end
            TP.phase = "settling"
            task.wait(AR2_CFG.tp.settleTime or 0.35)
            if not wasNoclip then AR2_stopNoclip() end
            local h2 = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
            if h2 then h2.PlatformStand = false h2.Sit = false end
        end)
        if not ok then warn("[Origin HUB] teleport error: " .. tostring(err)) end
        TP.active = false TP.phase = "idle"
        pcall(function() tpBeam.Visible = false end)
    end)
    return true
end
function AR2_teleportStop()
    TP.active = false TP.phase = "idle"
    pcall(function() tpBeam.Visible = false end)
    local h = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
    if h then h.PlatformStand = false h.Sit = false end
end
function AR2_teleportToPosition(vec) return AR2_teleportStart({ pos = vec }) end
function AR2_teleportToNearestZombie()
    local zc = getZombieContainer()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not zc or not hrp then return false end
    local bestRoot, bestD = nil, math.huge
    for _, m in ipairs(zc:GetChildren()) do
        if m:IsA("Model") then
            local r = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
            if r and r:IsA("BasePart") then local d = (r.Position - hrp.Position).Magnitude if d < bestD then bestD = d bestRoot = r end end
        end
    end
    if not bestRoot then return false end
    return AR2_teleportStart({ pos = bestRoot.Position + Vector3.new(0, 5, 0) })
end
function AR2_teleportToNearestCorpse()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local list = getCorpseList and getCorpseList() or {}
    local best, bestD = nil, math.huge
    for _, m in ipairs(list) do
        local r = m:FindFirstChild("HumanoidRootPart") or m:FindFirstChild("Head") or m:FindFirstChild("Head2")
        if r then local d = (r.Position - hrp.Position).Magnitude if d < bestD then bestD = d best = r end end
    end
    if not best then return false end
    return AR2_teleportStart({ pos = best.Position + Vector3.new(0, 5, 0) })
end
function AR2_teleportToNearestPlayer()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local best, bestD = nil, math.huge
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LP and pl.Character then
            local r = pl.Character:FindFirstChild("HumanoidRootPart")
            if r then local d = (r.Position - hrp.Position).Magnitude if d < bestD then bestD = d best = pl end end
        end
    end
    if not best then return false end
    return AR2_teleportStart({ player = best })
end
function AR2_teleportToPlayerName(name)
    local pl = Players:FindFirstChild(name)
    if not pl or pl == LP then return false end
    return AR2_teleportStart({ player = pl })
end

AR2_deathTP = { saved = nil }
local function AR2_notify(title, content, duration)
    local NL = getgenv and getgenv().OriginHUB_NL
    if NL then pcall(function() local Notif = NL:CreateNotification() Notif.new({ Title = title, Content = content, Duration = duration or 3 }) end) end
end
local function AR2_bindDeathTracker(char)
    if not char then return end
    task.spawn(function()
        local hum = char:FindFirstChildWhichIsA("Humanoid") or char:WaitForChild("Humanoid", 8)
        if not hum then return end
        hum.Died:Connect(function()
            if not AR2_CFG.tp.deathSave then return end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then AR2_deathTP.saved = hrp.Position AR2_notify("Origin HUB", "Death location saved", 3) end
        end)
    end)
end
LP.CharacterAdded:Connect(function(c) AR2_bindDeathTracker(c) end)
if LP.Character then AR2_bindDeathTracker(LP.Character) end
function AR2_teleportToDeath()
    if not AR2_deathTP.saved then AR2_notify("Origin HUB", "No death location saved", 3) return false end
    local pos = AR2_deathTP.saved AR2_deathTP.saved = nil
    AR2_notify("Origin HUB", "Flying to death location", 3)
    return AR2_teleportStart({ pos = pos + Vector3.new(0, AR2_CFG.tp.tpOffsetY or 4, 0) })
end
LP.CharacterAdded:Connect(function(c)
    if not AR2_CFG.tp.respawnTP then return end
    if not AR2_deathTP.saved then return end
    task.spawn(function()
        c:WaitForChild("HumanoidRootPart", 10)
        task.wait(AR2_CFG.tp.respawnDelay or 1.5)
        if not AR2_CFG.tp.respawnTP then return end
        local pos = AR2_deathTP.saved if not pos then return end
        AR2_deathTP.saved = nil
        AR2_notify("Origin HUB", "Respawn TP -- returning to death spot", 3)
        AR2_teleportStart({ pos = pos + Vector3.new(0, AR2_CFG.tp.tpOffsetY or 4, 0) })
    end)
end)
function AR2_clearDeathLocation() AR2_deathTP.saved = nil AR2_notify("Origin HUB", "Death location cleared", 2) end

-- =============================================
-- INTERACTIVE MAP  (2-point solve, 9deg default; send a Prison-center coord for exact)
-- =============================================
AR2_mapModule = (function()
    local MAP_IMAGE = "rbxassetid://102481571747859"
    local MAP_MIN_X, MAP_MAX_X = 9700, -9620
    local MAP_MIN_Z, MAP_MAX_Z = 8850, -9560
    local state = { gui=nil, img=nil, youDot=nil, conn=nil, dotPool={}, built=false }
    local function groundY(x, z)
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Exclude
        params.FilterDescendantsInstances = { LP.Character }
        local hit = workspace:Raycast(Vector3.new(x, 5000, z), Vector3.new(0, -12000, 0), params)
        return hit and hit.Position.Y or nil
    end
    local function clickToWorld(pu, pv)
        local rot = math.rad(AR2_CFG.tp.mapRot or 0)
        local cx, cy = pu - 0.5, pv - 0.5
        local cs, sn = math.cos(-rot), math.sin(-rot)
        local rx = cx * cs - cy * sn
        local ry = cx * sn + cy * cs
        local u, v = rx + 0.5, ry + 0.5
        local wx = MAP_MIN_X + u * (MAP_MAX_X - MAP_MIN_X)
        local wz = MAP_MIN_Z + v * (MAP_MAX_Z - MAP_MIN_Z)
        return wx, wz
    end
    local function worldToPanel(pos)
        local u = (pos.X - MAP_MIN_X) / (MAP_MAX_X - MAP_MIN_X)
        local v = (pos.Z - MAP_MIN_Z) / (MAP_MAX_Z - MAP_MIN_Z)
        local cx, cy = u - 0.5, v - 0.5
        local rot = math.rad(AR2_CFG.tp.mapRot or 0)
        local cs, sn = math.cos(rot), math.sin(rot)
        local rx = cx * cs - cy * sn
        local ry = cx * sn + cy * cs
        return 0.5 + rx, 0.5 + ry
    end
    local function build()
        if state.built and state.gui and state.gui.Parent then return end
        state.built = true
        local host = getMainHost()
        local gui = Instance.new("ScreenGui")
        gui.Name = "origin_hub_map" gui.ResetOnSpawn = false gui.IgnoreGuiInset = true gui.DisplayOrder = 999998 gui.Enabled = false gui.Parent = host
        state.gui = gui
        local dim = Instance.new("TextButton")
        dim.Size = UDim2.fromScale(1,1) dim.BackgroundColor3 = Color3.new(0,0,0) dim.BackgroundTransparency = 0.4 dim.AutoButtonColor = false dim.Text = "" dim.Parent = gui
        local W = 780
        local panel = Instance.new("Frame")
        panel.AnchorPoint = Vector2.new(0.5,0.5) panel.Position = UDim2.fromScale(0.5,0.5) panel.Size = UDim2.fromOffset(W, W+34)
        panel.BackgroundColor3 = Color3.fromRGB(15,16,20) panel.BorderSizePixel = 0 panel.Parent = gui
        local pc = Instance.new("UICorner"); pc.CornerRadius = UDim.new(0,8); pc.Parent = panel
        local title = Instance.new("TextLabel")
        title.BackgroundTransparency = 1 title.Position = UDim2.fromOffset(12,0) title.Size = UDim2.new(1,-80,0,34)
        title.Font = Enum.Font.GothamBold title.TextSize = 14 title.TextXAlignment = Enum.TextXAlignment.Left
        title.TextColor3 = Color3.fromRGB(225,225,235) title.Text = "Map -- click to TP" title.Parent = panel
        local close = Instance.new("TextButton")
        close.Size = UDim2.fromOffset(26,22) close.Position = UDim2.new(1,-32,0,6) close.BackgroundColor3 = Color3.fromRGB(40,42,52)
        close.Text = "X" close.Font = Enum.Font.GothamBold close.TextSize = 13 close.TextColor3 = Color3.fromRGB(230,120,120) close.BorderSizePixel = 0 close.Parent = panel
        local cc2 = Instance.new("UICorner"); cc2.CornerRadius = UDim.new(0,5); cc2.Parent = close
        local clip = Instance.new("Frame")
        clip.Position = UDim2.fromOffset(0,34) clip.Size = UDim2.fromOffset(W,W) clip.BackgroundColor3 = Color3.fromRGB(20,24,30) clip.BorderSizePixel = 0 clip.ClipsDescendants = true clip.Parent = panel
        local ic = Instance.new("UICorner"); ic.CornerRadius = UDim.new(0,6); ic.Parent = clip
        state.clip = clip
        local img = Instance.new("ImageLabel")
        img.AnchorPoint = Vector2.new(0.5,0.5) img.Position = UDim2.fromScale(0.5,0.5) img.Size = UDim2.fromScale(1.10,1.10)
        img.BackgroundTransparency = 1 img.Image = MAP_IMAGE img.ScaleType = Enum.ScaleType.Stretch img.BorderSizePixel = 0 img.ZIndex = 2 img.Parent = clip
        state.img = img
        local you = Instance.new("Frame")
        you.AnchorPoint = Vector2.new(0.5,0.5) you.Size = UDim2.fromOffset(9,9) you.BackgroundColor3 = Color3.fromRGB(255,220,60) you.BorderSizePixel = 0 you.ZIndex = 5 you.Parent = clip
        local yc = Instance.new("UICorner"); yc.CornerRadius = UDim.new(1,0); yc.Parent = you
        state.youDot = you
        dim.MouseButton1Click:Connect(function() AR2_mapModule.setVisible(false) end)
        close.MouseButton1Click:Connect(function() AR2_mapModule.setVisible(false) end)
        clip.InputBegan:Connect(function(input)
            if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
            local mouse = UIS:GetMouseLocation()
            local abs, sz = clip.AbsolutePosition, clip.AbsoluteSize
            if sz.X <= 0 or sz.Y <= 0 then return end
            local pu = (mouse.X - abs.X) / sz.X
            local pv = (mouse.Y - abs.Y) / sz.Y
            if pu < 0 or pu > 1 or pv < 0 or pv > 1 then return end
            local wx, wz = clickToWorld(pu, pv)
            local gy = groundY(wx, wz)
            AR2_teleportStart({ pos = Vector3.new(wx, (gy or 200) + 6, wz) })
            AR2_mapModule.setVisible(false)
        end)
    end
    local function refreshDots()
        if state.img then state.img.Rotation = AR2_CFG.tp.mapRot or 0 end
        local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if state.youDot and root then local pu, pv = worldToPanel(root.Position) state.youDot.Position = UDim2.fromScale(pu, pv) state.youDot.Visible = true
        elseif state.youDot then state.youDot.Visible = false end
        if not state.clip then return end
        local i = 0
        for _, pl in ipairs(Players:GetPlayers()) do
            if pl ~= LP and pl.Character then
                local r = pl.Character:FindFirstChild("HumanoidRootPart")
                if r then
                    i = i + 1
                    local d = state.dotPool[i]
                    if not d then
                        d = Instance.new("Frame")
                        d.AnchorPoint = Vector2.new(0.5,0.5) d.Size = UDim2.fromOffset(7,7) d.BackgroundColor3 = Color3.fromRGB(255,90,90) d.BorderSizePixel = 0 d.ZIndex = 4 d.Parent = state.clip
                        local dc2 = Instance.new("UICorner"); dc2.CornerRadius = UDim.new(1,0); dc2.Parent = d
                        state.dotPool[i] = d
                    end
                    local pu, pv = worldToPanel(r.Position)
                    d.Position = UDim2.fromScale(pu, pv) d.Visible = true
                end
            end
        end
        for j = i + 1, #state.dotPool do state.dotPool[j].Visible = false end
    end
    local M = {}
    function M.setVisible(on)
        if not state.gui or not state.gui.Parent then build() end
        if on == nil then on = not state.gui.Enabled end
        state.gui.Enabled = on
        AR2_CFG.tp.mapFake = on
        if on then
            refreshDots()
            if not state.conn then
                local acc = 0
                state.conn = RunSvc.RenderStepped:Connect(function(dt) acc = acc + dt if acc >= 0.1 then acc = 0 pcall(refreshDots) end end)
            end
        elseif state.conn then state.conn:Disconnect() state.conn = nil end
    end
    function M.destroy()
        if state.conn then state.conn:Disconnect() state.conn = nil end
        if state.gui then state.gui:Destroy() end
        state.gui, state.img, state.youDot, state.clip = nil, nil, nil, nil
        state.dotPool = {} state.built = false
    end
    return M
end)()
function AR2_toggleMapTP(on) AR2_mapModule.setVisible(on) end

local function getCursorWorldPos()
    local mouse = LP:GetMouse()
    if not mouse then return nil end
    local ray = mouse.UnitRay
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = { LP.Character }
    local result = workspace:Raycast(ray.Origin, ray.Direction * 5000, params)
    if result then return result.Position end
    return ray.Origin + ray.Direction * 500
end
local function doClickTeleport()
    if not AR2_CFG.tp.clickTP then return end
    if TP.active then return end
    local pos = getCursorWorldPos() if not pos then return end
    AR2_teleportStart({ pos = pos + Vector3.new(0, AR2_CFG.tp.tpOffsetY or 4, 0) })
end
local clickTPWasDown = false
RunSvc.Heartbeat:Connect(function()
    if not AR2_CFG.tp.clickTP then clickTPWasDown = false return end
    local keyName = AR2_normalizeKey(AR2_CFG.tp.clickTPKey or "T")
    local kc = Enum.KeyCode[keyName]
    if not kc then return end
    local isDown = UIS:IsKeyDown(kc)
    if isDown and not clickTPWasDown then clickTPWasDown = true doClickTeleport()
    elseif not isDown then clickTPWasDown = false end
end)

-- =============================================
-- FREEZE ZOMBIES
-- =============================================
-- Freeze uses a velocity + CFrame lock (NOT anchoring). Anchoring only freezes them on
-- your screen; a velocity lock replicates to the server for any zombie whose physics
-- assembly this client owns -- which is normally the ones near you -- so they actually
-- stop server-side. Truly server-authoritative AI can't be frozen from the client at all.
local frozenRoots = {}
local function freezeAllWalkers()
    local container = getZombieContainer() if not container then return end
    local origin = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    local originPos = origin and origin.Position or Vector3.zero
    local seen = {}
    for _, m in ipairs(container:GetChildren()) do
        if m:IsA("Model") then
            local root = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
            if root and root:IsA("BasePart") then
                local ok = true
                if not AR2_CFG.misc.freezeAll and origin then ok = (root.Position - originPos).Magnitude <= AR2_CFG.misc.freezeRadius end
                if ok then
                    seen[m] = true
                    local hum = m:FindFirstChildOfClass("Humanoid")
                    if not frozenRoots[m] then frozenRoots[m] = { cf = root.CFrame, hum = hum, walk = (hum and hum.WalkSpeed) or 16 } end
                    local d = frozenRoots[m]
                    pcall(function()
                        root.AssemblyLinearVelocity = Vector3.zero
                        root.AssemblyAngularVelocity = Vector3.zero
                        root.CFrame = d.cf
                    end)
                    if hum then pcall(function() hum.WalkSpeed = 0 hum.AutoRotate = false hum:Move(Vector3.zero, false) hum:ChangeState(Enum.HumanoidStateType.Physics) end) end
                end
            end
        end
    end
    for model, data in pairs(frozenRoots) do
        if not seen[model] or not model.Parent then
            if data.hum and data.hum.Parent then pcall(function() data.hum.WalkSpeed = data.walk data.hum.AutoRotate = true end) end
            frozenRoots[model] = nil
        end
    end
end
local function unfreezeAll()
    for _, data in pairs(frozenRoots) do if data.hum and data.hum.Parent then pcall(function() data.hum.WalkSpeed = data.walk data.hum.AutoRotate = true end) end end
    frozenRoots = {}
end
RunSvc.Heartbeat:Connect(function()
    if not AR2_CFG.misc.freeze then if next(frozenRoots) then unfreezeAll() end return end
    freezeAllWalkers()
end)

-- =============================================
-- CARS: seat-based noclip / fly (wheels+roof-park+repair) / collision-off / enter / speed
-- =============================================
local SEAT_STOP_PARENTS = { Cars=true, Vehicles=true, Horses=true, Mounts=true, Workspace=true, Map=true, Camera=true }
local function seatedModel()
    local char = LP.Character
    local hum = char and char:FindFirstChildWhichIsA("Humanoid")
    if not hum or not hum.SeatPart then return nil, nil, nil end
    local seat = hum.SeatPart
    local model = seat.Parent
    if not (model and model:IsA("Model")) then return nil, nil, nil end
    -- climb to the outermost vehicle model (handles nested horse rigs), but STOP the moment the
    -- parent is a container (Cars/Vehicles/Horses/Mounts folder OR model) or Workspace. Without
    -- this guard, if the container is a Model the climb grabbed EVERY vehicle and anchored them,
    -- which is why car fly broke. Now it can only ever grab the single vehicle you're in.
    local hops = 0
    while hops < 4 do
        local par = model.Parent
        if not par or not par:IsA("Model") or SEAT_STOP_PARENTS[par.Name] then break end
        model = par hops = hops + 1
    end
    local base = model.PrimaryPart or seat
    return model, base, hum
end

local cnActive, cnConn, cnModel, cnBase, cnParts, cnOffsets, cnSaved = false, nil, nil, nil, nil, nil, nil
local cnKeys = { W=false, A=false, S=false, D=false, Space=false, LeftControl=false }
local function stopModelVelocity(model)
    if not model or not model.Parent then return end
    for _, p in ipairs(model:GetDescendants()) do if p:IsA("BasePart") then pcall(function() p.AssemblyLinearVelocity = Vector3.zero p.AssemblyAngularVelocity = Vector3.zero end) end end
end
local function platformModel(model)
    if not model or not model.Parent then return end
    for _, p in ipairs(model:GetDescendants()) do
        if p:IsA("BasePart") then pcall(function() p.CanCollide = true p.Anchored = true end) end
    end
end
local function carNoclipStop()
    cnActive = false
    AR2_CFG.cars.noclip = false
    if cnConn then cnConn:Disconnect() cnConn = nil end
    if cnParts and cnSaved then
        if AR2_CFG.cars.platformOnDisable and cnModel then platformModel(cnModel)
        else for _, p in ipairs(cnParts) do pcall(function() if p and p.Parent then p.Anchored = cnSaved[p] or false end end) end end
    end
    if AR2_CFG.cars.noclipEjectOnDisable then
        local hum = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
        if hum and hum.Sit then hum.Sit = false pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end) end
    end
    if cnModel and not AR2_CFG.cars.platformOnDisable then stopModelVelocity(cnModel) end
    cnModel, cnBase, cnParts, cnOffsets, cnSaved = nil, nil, nil, nil, nil
    for k in pairs(cnKeys) do cnKeys[k] = false end
end
local function carNoclipStart()
    if cnConn then return true end
    local model, base, hum = seatedModel()
    if not model or not base then return false end
    if not hum or not hum.Sit then return false end
    local parts, offsets, saved = {}, {}, {}
    for _, p in ipairs(model:GetDescendants()) do
        if p:IsA("BasePart") then
            parts[#parts+1] = p
            saved[p] = p.Anchored
            offsets[p] = base.CFrame:Inverse() * p.CFrame
            p.Anchored = true
            p.AssemblyLinearVelocity = Vector3.zero p.AssemblyAngularVelocity = Vector3.zero
        end
    end
    if #parts == 0 then return false end
    cnModel, cnBase, cnParts, cnOffsets, cnSaved = model, base, parts, offsets, saved
    cnActive = true AR2_CFG.cars.noclip = true
    cnConn = RunSvc.Heartbeat:Connect(function(dt)
        if not cnActive or not cnModel or not cnModel.Parent or not cnBase or not cnBase.Parent then carNoclipStop() return end
        local h = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
        if not h or not h.Sit then carNoclipStop() return end
        local cam = workspace.CurrentCamera
        local move = Vector3.zero
        if cnKeys.W then move += cam.CFrame.LookVector end
        if cnKeys.S then move -= cam.CFrame.LookVector end
        if cnKeys.D then move += cam.CFrame.RightVector end
        if cnKeys.A then move -= cam.CFrame.RightVector end
        if cnKeys.Space then move += Vector3.new(0,1,0) end
        if cnKeys.LeftControl then move -= Vector3.new(0,1,0) end
        if move.Magnitude > 0 then
            local delta = move.Unit * (AR2_CFG.cars.noclipSpeed or 90) * dt
            local newCF = cnBase.CFrame + delta
            cnBase.CFrame = newCF
            for p, off in pairs(cnOffsets) do if p and p.Parent then p.CFrame = newCF * off end end
        end
    end)
    return true
end
local function carNoclipSetEnabled(on) if on then return carNoclipStart() else carNoclipStop() return true end end
UIS.InputBegan:Connect(function(input, gpe)
    if gpe or not cnActive then return end
    local k = input.KeyCode
    if k == Enum.KeyCode.W then cnKeys.W = true elseif k == Enum.KeyCode.A then cnKeys.A = true
    elseif k == Enum.KeyCode.S then cnKeys.S = true elseif k == Enum.KeyCode.D then cnKeys.D = true
    elseif k == Enum.KeyCode.Space then cnKeys.Space = true elseif k == Enum.KeyCode.LeftControl then cnKeys.LeftControl = true end
end)
UIS.InputEnded:Connect(function(input, gpe)
    if gpe then return end
    local k = input.KeyCode
    if k == Enum.KeyCode.W then cnKeys.W = false elseif k == Enum.KeyCode.A then cnKeys.A = false
    elseif k == Enum.KeyCode.S then cnKeys.S = false elseif k == Enum.KeyCode.D then cnKeys.D = false
    elseif k == Enum.KeyCode.Space then cnKeys.Space = false elseif k == Enum.KeyCode.LeftControl then cnKeys.LeftControl = false end
end)

-- CAR COLLISION OFF (drive through walls, no anchor-fly, wheels on ground)
AR2_setCarClip = nil
do
    local conn = nil
    function AR2_setCarClip(on)
        AR2_CFG.cars.clipOff = on
        if conn then conn:Disconnect() conn = nil end
        if not on then
            local m = (seatedModel())
            if m then for _, p in ipairs(m:GetDescendants()) do if p:IsA("BasePart") then pcall(function() p.CanCollide = true end) end end end
            return
        end
        conn = RunSvc.Heartbeat:Connect(function()
            if not AR2_CFG.cars.clipOff then return end
            local m = (seatedModel())
            if m then for _, p in ipairs(m:GetDescendants()) do if p:IsA("BasePart") and p.CanCollide then pcall(function() p.CanCollide = false end) end end end
        end)
    end
end

-- CAR FLY -- anchors whole model, drives every part by offset (wheels follow).
-- Roof-park hops you onto the top; repair snaps a bent car back to clean layout.
AR2_setFly = nil
AR2_parkVehicle = nil
AR2_repairVehicle = nil
do
    local flyConn = nil
    local flyModel, flyBase, flyOffsets = nil, nil, nil
    local repairSnap = {}
    local fk = { W=false, A=false, S=false, D=false, Space=false, LeftControl=false, LeftShift=false }

    local function grabModel()
        local model, base = seatedModel()
        if not model or not base then return false end
        if not repairSnap[model] then
            repairSnap[model] = {}
            for _, p in ipairs(model:GetDescendants()) do
                if p:IsA("BasePart") then repairSnap[model][p] = base.CFrame:Inverse() * p.CFrame end
            end
        end
        flyModel, flyBase, flyOffsets = model, base, {}
        for _, p in ipairs(model:GetDescendants()) do
            if p:IsA("BasePart") then
                flyOffsets[p] = base.CFrame:Inverse() * p.CFrame
                p.Anchored = true
                p.AssemblyLinearVelocity = Vector3.zero
                p.AssemblyAngularVelocity = Vector3.zero
            end
        end
        return true
    end
    local function releaseModel(keepFrozen)
        if flyModel and flyModel.Parent and not keepFrozen then
            for p in pairs(flyOffsets or {}) do if p and p.Parent then pcall(function() p.Anchored = false end) end end
        end
        flyModel, flyBase, flyOffsets = nil, nil, nil
    end

    UIS.InputBegan:Connect(function(input, gpe)
        if gpe or not AR2_CFG.cars.fly then return end
        local k = input.KeyCode
        if k == Enum.KeyCode.W then fk.W=true elseif k == Enum.KeyCode.A then fk.A=true
        elseif k == Enum.KeyCode.S then fk.S=true elseif k == Enum.KeyCode.D then fk.D=true
        elseif k == Enum.KeyCode.Space then fk.Space=true elseif k == Enum.KeyCode.LeftControl then fk.LeftControl=true
        elseif k == Enum.KeyCode.LeftShift then fk.LeftShift=true end
    end)
    UIS.InputEnded:Connect(function(input)
        local k = input.KeyCode
        if k == Enum.KeyCode.W then fk.W=false elseif k == Enum.KeyCode.A then fk.A=false
        elseif k == Enum.KeyCode.S then fk.S=false elseif k == Enum.KeyCode.D then fk.D=false
        elseif k == Enum.KeyCode.Space then fk.Space=false elseif k == Enum.KeyCode.LeftControl then fk.LeftControl=false
        elseif k == Enum.KeyCode.LeftShift then fk.LeftShift=false end
    end)

    function AR2_setFly(on)
        AR2_CFG.cars.fly = on
        if flyConn then flyConn:Disconnect() flyConn = nil end
        for key in pairs(fk) do fk[key] = false end
        if not on then releaseModel(false) return end
        flyConn = RunSvc.RenderStepped:Connect(function(dt)
            if not flyModel or not flyModel.Parent or not flyBase or not flyBase.Parent then
                if not grabModel() then return end
            end
            local hum = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
            local stillSeated = hum and hum.SeatPart and hum.SeatPart.Parent == flyModel
            local cam = workspace.CurrentCamera
            local move = Vector3.zero
            if stillSeated then
                if fk.W then move += cam.CFrame.LookVector end
                if fk.S then move -= cam.CFrame.LookVector end
                if fk.D then move += cam.CFrame.RightVector end
                if fk.A then move -= cam.CFrame.RightVector end
                if fk.Space then move += Vector3.new(0,1,0) end
                if fk.LeftControl then move -= Vector3.new(0,1,0) end
            end
            local mult = fk.LeftShift and 2 or 1
            local newCF = flyBase.CFrame
            if move.Magnitude > 0 then
                newCF = flyBase.CFrame + move.Unit * (AR2_CFG.cars.flySpeed or 60) * mult * dt
            end
            flyBase.CFrame = newCF
            for p, off in pairs(flyOffsets) do if p and p.Parent then p.CFrame = newCF * off end end
        end)
    end

    function AR2_parkVehicle()
        local model, base, hum = seatedModel()
        model = model or flyModel
        base = base or flyBase
        if not model then AR2_notify("Origin HUB", "Not in a vehicle", 2) return false end
        for _, p in ipairs(model:GetDescendants()) do
            if p:IsA("BasePart") then pcall(function() p.Anchored = true p.CanCollide = true
                p.AssemblyLinearVelocity = Vector3.zero p.AssemblyAngularVelocity = Vector3.zero end) end
        end
        local h = hum or (LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid"))
        if h then if h.Sit then h.Sit = false end pcall(function() h:ChangeState(Enum.HumanoidStateType.GettingUp) end) end
        AR2_CFG.cars.fly = false
        if flyConn then flyConn:Disconnect() flyConn = nil end
        task.spawn(function()
            local okC, cf, size = pcall(function() return model:GetBoundingBox() end)
            local center = (base and base.Position) or (okC and cf.Position) or Vector3.zero
            local topY = (okC and size) and (size.Y/2 + 4) or 8
            for _ = 1, 10 do
                task.wait(0.06)
                local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                if hrp then hrp.CFrame = CFrame.new(center + Vector3.new(0, topY, 0)) hrp.AssemblyLinearVelocity = Vector3.zero end
            end
        end)
        flyModel, flyBase, flyOffsets = nil, nil, nil
        AR2_notify("Origin HUB", "Vehicle parked -- on the roof", 3)
        return true
    end

    function AR2_repairVehicle()
        local model, base = seatedModel()
        model = model or flyModel
        base = base or flyBase
        if not model then
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local bestD = 60
                for _, container in ipairs(getVehicleContainers()) do
                    for _, m in ipairs(container:GetChildren()) do
                        if m:IsA("Model") then
                            local pr = m.PrimaryPart or m:FindFirstChild("DriveSeat")
                            if pr then local d = (pr.Position - hrp.Position).Magnitude if d < bestD then bestD = d model = m base = pr end end
                        end
                    end
                end
            end
        end
        if not model then AR2_notify("Origin HUB", "No vehicle to repair", 2) return false end
        base = base or model.PrimaryPart or model:FindFirstChild("DriveSeat") or model:FindFirstChildWhichIsA("BasePart")
        if not base then AR2_notify("Origin HUB", "Car has no base part", 2) return false end
        local snap = repairSnap[model]
        if not snap then
            AR2_notify("Origin HUB", "No clean snapshot -- snapping to current", 3)
            snap = {}
            for _, p in ipairs(model:GetDescendants()) do if p:IsA("BasePart") then snap[p] = base.CFrame:Inverse() * p.CFrame end end
            repairSnap[model] = snap
        end
        for p, off in pairs(snap) do
            if p and p.Parent then
                pcall(function()
                    p.CFrame = base.CFrame * off
                    p.AssemblyLinearVelocity = Vector3.zero
                    p.AssemblyAngularVelocity = Vector3.zero
                end)
            end
        end
        AR2_notify("Origin HUB", "Vehicle repaired", 2)
        return true
    end
end

function AR2_enterNearestVehicle()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    local hum = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
    if not hrp or not hum then return false end
    local bestSeat, bestD = nil, (AR2_CFG.cars.enterRange or 120)
    for _, container in ipairs(getVehicleContainers()) do
        for _, m in ipairs(container:GetChildren()) do
            if m:IsA("Model") then
                for _, d in ipairs(m:GetDescendants()) do
                    if d:IsA("VehicleSeat") or d:IsA("Seat") then
                        if not d.Occupant then
                            local dist = (d.Position - hrp.Position).Magnitude
                            if dist < bestD then bestD = dist bestSeat = d end
                        end
                    end
                end
            end
        end
    end
    if not bestSeat then AR2_notify("Origin HUB", "No empty seat in range", 2) return false end
    pcall(function() bestSeat:Sit(hum) end)
    return true
end

do
    RunSvc.Heartbeat:Connect(function(dt)
        if not AR2_CFG.cars.speedMult then return end
        local model, base, hum = seatedModel()
        if not model or not base or not hum or not hum.Sit then return end
        if cnActive or AR2_CFG.cars.fly then return end
        local vel = base.AssemblyLinearVelocity
        if vel.Magnitude > 2 then
            local extra = (AR2_CFG.cars.speedMultVal or 2) - 1
            base.AssemblyLinearVelocity = vel + vel.Unit * vel.Magnitude * extra * dt * 4
        end
    end)
end

-- =============================================
-- LOOT  (PhysicalLoot, tier from ItemStored)
-- =============================================
local AR2_LOOT_CACHE = { list = {}, at = 0, folder = nil }
local AR2_LOOT_REFRESH = 1.0
local function getLootablesFolder()
    if AR2_LOOT_CACHE.folder and AR2_LOOT_CACHE.folder.Parent then return AR2_LOOT_CACHE.folder end
    local f = workspace:FindFirstChild("PhysicalLoot") or workspace:FindFirstChild("Lootables") or workspace:FindFirstChild("Loot") or workspace:FindFirstChild("Items")
    AR2_LOOT_CACHE.folder = f
    return f
end
local function lootPrimary(model)
    if model:IsA("Model") then
        local named = model:FindFirstChild(model.Name)
        if named and named:IsA("BasePart") then return named end
        return model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
    elseif model:IsA("BasePart") then return model end
    return nil
end
local TIER_KEYWORDS = {
    Guns = { "rifle","pistol","shotgun","smg","ak","m4","glock","sniper","revolver","gun","carbine","launcher" },
    Ammo = { "ammo","round","mag","magazine","clip","shell" },
    Meds = { "bandage","medkit","med","syringe","pill","painkill","splint","morphine","adrenaline","health" },
    Food = { "food","chips","water","soda","can","snack","drink","meat","bread","apple" },
}
local function lootTier(itemName)
    local n = tostring(itemName or ""):lower()
    for tier, words in pairs(TIER_KEYWORDS) do
        for _, w in ipairs(words) do if n:find(w, 1, true) then return tier end end
    end
    return "Other"
end
local TIER_COLOR = {
    Guns = Color3.fromRGB(255,80,80), Ammo = Color3.fromRGB(255,160,40),
    Meds = Color3.fromRGB(80,220,120), Food = Color3.fromRGB(90,190,255),
    Other = Color3.fromRGB(255,200,40),
}
local function lootItemName(model)
    local a = model:GetAttribute("ItemStored")
    if a and tostring(a) ~= "" then return tostring(a) end
    return model.Name
end
local function refreshLootList()
    AR2_LOOT_CACHE.at = os.clock()
    local folder = getLootablesFolder()
    if not folder then AR2_LOOT_CACHE.list = {} return end
    local keyword = (AR2_CFG.loot.keyword or ""):lower()
    local filter = AR2_CFG.loot.tierFilter or "All"
    local list = {}
    local junk = { spawner=true, template=true, part=true }
    for _, model in ipairs(folder:GetChildren()) do
        if model:IsA("Model") or model:IsA("BasePart") then
            local rawName = model.Name:lower()
            if not junk[rawName] then
                local itemName = lootItemName(model)
                local tier = lootTier(itemName)
                local nameOk = (keyword == "" or itemName:lower():find(keyword, 1, true))
                local tierOk = (filter == "All" or tier == filter)
                if nameOk and tierOk then
                    local prim = lootPrimary(model)
                    if prim then list[#list+1] = { model = model, prim = prim, disp = itemName, tier = tier } end
                end
            end
        end
    end
    AR2_LOOT_CACHE.list = list
end
local function getLootList()
    local now = os.clock()
    if now - AR2_LOOT_CACHE.at >= AR2_LOOT_REFRESH then refreshLootList() end
    return AR2_LOOT_CACHE.list
end
local AR2_lootHighlights = {}
local AR2_lootHighlightFolder = nil
local function ensureLootHighlightFolder()
    if AR2_lootHighlightFolder and AR2_lootHighlightFolder.Parent then return end
    local host = (typeof(gethui) == "function" and gethui()) or game:GetService("CoreGui")
    AR2_lootHighlightFolder = Instance.new("Folder") AR2_lootHighlightFolder.Name = "oh_loot_hl" AR2_lootHighlightFolder.Parent = host
end
local function setLootHighlight(obj, color)
    if not obj then return end
    local h = AR2_lootHighlights[obj]
    if not h or not h.Parent then
        ensureLootHighlightFolder()
        h = Instance.new("Highlight") h.Name = "oh_loot_hl" h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.FillTransparency = 0.5 h.OutlineTransparency = 0 h.OutlineColor = Color3.new(1,1,1) h.Adornee = obj h.Parent = AR2_lootHighlightFolder
        AR2_lootHighlights[obj] = h
    end
    h.FillColor = color or Color3.fromRGB(255,200,40) h.Enabled = true
end
local function clearLootHighlights()
    for _, h in pairs(AR2_lootHighlights) do pcall(function() h:Destroy() end) end
    AR2_lootHighlights = {}
end

-- =============================================
-- ITEMDROPS
-- =============================================
local AR2_DROP_CACHE = { list = {}, at = 0 }
local function getDropsFolder() return workspace:FindFirstChild("ItemDrops") end
local function refreshDropList()
    AR2_DROP_CACHE.at = os.clock()
    local folder = getDropsFolder()
    if not folder then AR2_DROP_CACHE.list = {} return end
    local list = {}
    for _, m in ipairs(folder:GetChildren()) do
        local prim = (m:IsA("Model") and (m.PrimaryPart or m:FindFirstChildWhichIsA("BasePart"))) or (m:IsA("BasePart") and m)
        if prim then list[#list+1] = { model = m, prim = prim } end
    end
    AR2_DROP_CACHE.list = list
end
local function getDropList()
    local now = os.clock()
    if now - AR2_DROP_CACHE.at >= 1.0 then refreshDropList() end
    return AR2_DROP_CACHE.list
end

-- =============================================
-- CORPSES  (Loot_Corpse tag + Head2)
-- =============================================
local corpseCache = { list = {}, at = 0 }
local CORPSE_REFRESH = 1.0
local CORPSE_FOLDERS = { "Corpses","Characters","DeadBodies","Dead","Bodies","Graveyard" }
local function looksLikeBody(m)
    if m:FindFirstChild("Loot_Corpse") then return true end
    local head = m:FindFirstChild("Head") or m:FindFirstChild("Head2")
    if not head then return false end
    return (m:FindFirstChild("UpperTorso") or m:FindFirstChild("LowerTorso")
         or m:FindFirstChild("Torso") or m:FindFirstChild("HumanoidRootPart")) ~= nil
end
local function refreshCorpseCache()
    corpseCache.at = os.clock()
    local list, seen = {}, {}
    local myChar = LP.Character
    for _, folderName in ipairs(CORPSE_FOLDERS) do
        local f = workspace:FindFirstChild(folderName)
        if f then
            for _, m in ipairs(f:GetChildren()) do
                if m:IsA("Model") and not seen[m] and m ~= myChar and not Players:GetPlayerFromCharacter(m) and looksLikeBody(m) then
                    seen[m] = true list[#list+1] = m
                end
            end
        end
    end
    for _, m in ipairs(workspace:GetChildren()) do
        local hum = m:IsA("Model") and m:FindFirstChildOfClass("Humanoid")
        local dead = (not hum) or hum.Health <= 0
        if m:IsA("Model") and not seen[m] and m ~= myChar and not Players:GetPlayerFromCharacter(m) and dead and looksLikeBody(m) then
            seen[m] = true list[#list+1] = m
        end
    end
    corpseCache.list = list
end
local CORPSE_PARTS = {
    Head=true, Head2=true, UpperTorso=true, LowerTorso=true, Torso=true, HumanoidRootPart=true,
    LeftUpperArm=true, RightUpperArm=true, LeftLowerArm=true, RightLowerArm=true,
    LeftUpperLeg=true, RightUpperLeg=true, LeftLowerLeg=true, RightLowerLeg=true,
    LeftHand=true, RightHand=true, LeftFoot=true, RightFoot=true,
}
local function corpseBox(m)
    local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
    local any = false
    for _, p in ipairs(m:GetDescendants()) do
        if p:IsA("BasePart") and CORPSE_PARTS[p.Name] then
            local s, on = Cam:WorldToViewportPoint(p.Position)
            if on and s.Z > 0 then
                any = true
                if s.X < minX then minX = s.X end
                if s.Y < minY then minY = s.Y end
                if s.X > maxX then maxX = s.X end
                if s.Y > maxY then maxY = s.Y end
            end
        end
    end
    if not any then return nil end
    local pad = 6
    return minX - pad, minY - pad, (maxX - minX) + pad*2, (maxY - minY) + pad*2
end
function getCorpseList()
    local now = os.clock()
    if now - corpseCache.at >= CORPSE_REFRESH then refreshCorpseCache() end
    return corpseCache.list
end

-- =============================================
-- HITBOX EXPANDER  (resize enemy player heads client-side; the server trusts the
-- client-reported hit part, so a bigger head = easier hits. Reversible. Size to 10k.)
-- =============================================
local hbeSaved = {}
task.spawn(function()
    while true do
        if AR2_CFG.weapon.hitbox then
            local sz = AR2_CFG.weapon.hitboxSize or 50
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= LP and pl.Character then
                    local h = pl.Character:FindFirstChild("Head") or pl.Character:FindFirstChild("Head2")
                    local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                    if h and h:IsA("BasePart") and hum and hum.Health > 0 then
                        if hbeSaved[h] == nil then hbeSaved[h] = h.Size end
                        pcall(function() h.Size = Vector3.new(sz, sz, sz) h.Transparency = 0.55 h.CanCollide = false end)
                    end
                end
            end
        elseif next(hbeSaved) then
            for h, sz in pairs(hbeSaved) do if h and h.Parent then pcall(function() h.Size = sz h.Transparency = 0 end) end end
            hbeSaved = {}
        end
        task.wait(0.3)
    end
end)

-- =============================================
-- 3D BOX ESP  (draws a real 3D box around enemies in world space -- also shows the true
-- hitbox extent, so it grows with the Hitbox Expander. Toggle lives in ESP Style.)
-- =============================================
do
local box3dMap = {}   -- model -> SelectionBox
local function ohClear3D() for m, b in pairs(box3dMap) do pcall(function() b:Destroy() end) box3dMap[m] = nil end end
task.spawn(function()
    while true do
        local E = AR2_CFG.esp
        if E.box3D then
            local live = {}
            local myr = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            local from = myr and myr.Position or (workspace.CurrentCamera and workspace.CurrentCamera.CFrame.Position)
            local function adorn(model)
                if not model or not model.Parent then return end
                live[model] = true
                local b = box3dMap[model]
                if not b or not b.Parent then
                    b = Instance.new("SelectionBox")
                    b.Name = "OH_BOX3D"
                    b.LineThickness = 0.04
                    b.SurfaceTransparency = 1
                    b.Adornee = model
                    b.Parent = model
                    box3dMap[model] = b
                end
                pcall(function() b.Color3 = E.box3DColor or Color3.fromRGB(255,70,70) end)
            end
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= LP and pl.Character then
                    local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then adorn(pl.Character) end
                end
            end
            if E.box3DZombies then
                local cont = getZombieContainer()
                if cont and from then
                    local zs = {}
                    for _, m in ipairs(cont:GetChildren()) do
                        if m:IsA("Model") then
                            local hum = m:FindFirstChildOfClass("Humanoid")
                            local rp = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
                            if hum and hum.Health > 0 and rp then zs[#zs+1] = { m = m, d = (rp.Position - from).Magnitude } end
                        end
                    end
                    table.sort(zs, function(a,b) return a.d < b.d end)
                    for i = 1, math.min(#zs, 60) do adorn(zs[i].m) end
                end
            end
            for m, b in pairs(box3dMap) do if not live[m] then pcall(function() b:Destroy() end) box3dMap[m] = nil end end
        elseif next(box3dMap) then
            ohClear3D()
        end
        task.wait(0.3)
    end
end)
end

-- =============================================
-- HORSE FLY  (physics-based: BodyVelocity + BodyGyro on the seat's assembly). Works on ANY
-- seat -- horses AND cars -- because it just pushes whatever you're sitting on, no anchoring
-- or model-grabbing. Fly with WASD + Space/Ctrl, Shift = 2x. Toggle in the Cars tab.
-- =============================================
AR2_setHorseFly = nil
do
    local hfConn = nil
    local bv, bg, hfPart = nil, nil, nil
    local hk = { W=false, A=false, S=false, D=false, Space=false, LeftControl=false, LeftShift=false }
    local function seatBase()
        local char = LP.Character
        local hum = char and char:FindFirstChildWhichIsA("Humanoid")
        if not hum or not hum.SeatPart then return nil end
        local seat = hum.SeatPart
        local model = seat.Parent
        local part = (model and model:IsA("Model") and model.PrimaryPart) or seat
        return part
    end
    local function cleanup()
        if bv then pcall(function() bv:Destroy() end) bv = nil end
        if bg then pcall(function() bg:Destroy() end) bg = nil end
        hfPart = nil
    end
    function AR2_setHorseFly(on)
        AR2_CFG.cars.horseFly = on and true or false
        if hfConn then hfConn:Disconnect() hfConn = nil end
        for k in pairs(hk) do hk[k] = false end
        if not on then cleanup() return end
        hfConn = RunSvc.RenderStepped:Connect(function()
            local part = seatBase()
            if not part or not part.Parent then cleanup() return end
            if hfPart ~= part or not bv or not bv.Parent then
                cleanup()
                hfPart = part
                bv = Instance.new("BodyVelocity") bv.MaxForce = Vector3.new(1e9,1e9,1e9) bv.P = 12000 bv.Velocity = Vector3.zero bv.Parent = part
                bg = Instance.new("BodyGyro") bg.MaxTorque = Vector3.new(1e9,1e9,1e9) bg.P = 9000 bg.D = 500 bg.CFrame = Cam.CFrame bg.Parent = part
            end
            local cam = Cam.CFrame
            local move = Vector3.zero
            if hk.W then move = move + cam.LookVector end
            if hk.S then move = move - cam.LookVector end
            if hk.D then move = move + cam.RightVector end
            if hk.A then move = move - cam.RightVector end
            if hk.Space then move = move + Vector3.new(0,1,0) end
            if hk.LeftControl then move = move - Vector3.new(0,1,0) end
            local spd = (AR2_CFG.cars.horseFlySpeed or 60) * (hk.LeftShift and 2 or 1)
            bv.Velocity = (move.Magnitude > 0) and (move.Unit * spd) or Vector3.zero
            bg.CFrame = cam
        end)
    end
    UIS.InputBegan:Connect(function(input, gpe)
        if gpe or not AR2_CFG.cars.horseFly then return end
        local k = input.KeyCode
        if k == Enum.KeyCode.W then hk.W=true elseif k == Enum.KeyCode.A then hk.A=true
        elseif k == Enum.KeyCode.S then hk.S=true elseif k == Enum.KeyCode.D then hk.D=true
        elseif k == Enum.KeyCode.Space then hk.Space=true elseif k == Enum.KeyCode.LeftControl then hk.LeftControl=true
        elseif k == Enum.KeyCode.LeftShift then hk.LeftShift=true end
    end)
    UIS.InputEnded:Connect(function(input)
        local k = input.KeyCode
        if k == Enum.KeyCode.W then hk.W=false elseif k == Enum.KeyCode.A then hk.A=false
        elseif k == Enum.KeyCode.S then hk.S=false elseif k == Enum.KeyCode.D then hk.D=false
        elseif k == Enum.KeyCode.Space then hk.Space=false elseif k == Enum.KeyCode.LeftControl then hk.LeftControl=false
        elseif k == Enum.KeyCode.LeftShift then hk.LeftShift=false end
    end)
end

-- =============================================
-- HORSE FLY  (rides the mount): finds the nearest rideable NPC (horses live in AI.Wildlife)
-- and pushes ITS root with BodyVelocity -- like Inert's car fly. You control the mount while
-- riding, so this does NOT trip the player anti-teleport. WASD + Space/Ctrl, Shift = 2x.
-- =============================================
AR2_setPlayerFly = nil
do
    local pfConn = nil
    local pbv, pbg = nil, nil
    local pk = { W=false, A=false, S=false, D=false, Space=false, LeftControl=false, LeftShift=false }
    local function pfClean()
        if pbv then pcall(function() pbv:Destroy() end) pbv = nil end
        if pbg then pcall(function() pbg:Destroy() end) pbg = nil end
    end
    local mountPart = nil
    -- find the nearest rideable NPC root -- the horse you're on. Horses live in AI.Wildlife
    -- (from the mount dump). Pushing the MOUNT (which you control while riding) doesn't trip
    -- the player anti-teleport, unlike moving your own character.
    local function findMount(root)
        local best, bestD = nil, 16
        local ai = workspace:FindFirstChild("AI")
        local wl = ai and ai:FindFirstChild("Wildlife")
        if wl then
            for _, m in ipairs(wl:GetChildren()) do
                if m ~= LP.Character and not Players:GetPlayerFromCharacter(m) then
                    local rp = m:FindFirstChild("HumanoidRootPart") or (m:IsA("Model") and m.PrimaryPart)
                    if rp and rp:IsA("BasePart") and m:FindFirstChildOfClass("Humanoid") then
                        local d = (rp.Position - root.Position).Magnitude
                        if d < bestD then bestD, best = d, rp end
                    end
                end
            end
        end
        return best
    end
    function AR2_setPlayerFly(on)
        AR2_CFG.misc.playerFly = on and true or false
        if pfConn then pfConn:Disconnect() pfConn = nil end
        for k in pairs(pk) do pk[k] = false end
        mountPart = nil
        if not on then pfClean() return end
        pfConn = RunSvc.RenderStepped:Connect(function()
            local char = LP.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            if not root then return end
            -- (re)acquire the mount if we lost it or it wandered off
            if not mountPart or not mountPart.Parent or (mountPart.Position - root.Position).Magnitude > 30 then
                mountPart = findMount(root)
                if pbv then pfClean() end
            end
            if not mountPart then return end
            if pbv == nil or pbv.Parent ~= mountPart then
                pfClean()
                pbv = Instance.new("BodyVelocity") pbv.MaxForce = Vector3.new(1e9,1e9,1e9) pbv.P = 12000 pbv.Velocity = Vector3.zero pbv.Parent = mountPart
                pbg = Instance.new("BodyGyro")    pbg.MaxTorque = Vector3.new(1e9,1e9,1e9) pbg.P = 9000 pbg.D = 500 pbg.CFrame = Cam.CFrame pbg.Parent = mountPart
            end
            local cam = Cam.CFrame
            local move = Vector3.zero
            if pk.W then move = move + cam.LookVector end
            if pk.S then move = move - cam.LookVector end
            if pk.D then move = move + cam.RightVector end
            if pk.A then move = move - cam.RightVector end
            if pk.Space then move = move + Vector3.new(0,1,0) end
            if pk.LeftControl then move = move - Vector3.new(0,1,0) end
            local spd = (AR2_CFG.misc.playerFlySpeed or 80) * (pk.LeftShift and 2 or 1)
            pbv.Velocity = (move.Magnitude > 0) and (move.Unit * spd) or Vector3.zero
            pbg.CFrame = cam
        end)
    end
    UIS.InputBegan:Connect(function(input, gpe)
        if gpe or not AR2_CFG.misc.playerFly then return end
        local k = input.KeyCode
        if k == Enum.KeyCode.W then pk.W=true elseif k == Enum.KeyCode.A then pk.A=true
        elseif k == Enum.KeyCode.S then pk.S=true elseif k == Enum.KeyCode.D then pk.D=true
        elseif k == Enum.KeyCode.Space then pk.Space=true elseif k == Enum.KeyCode.LeftControl then pk.LeftControl=true
        elseif k == Enum.KeyCode.LeftShift then pk.LeftShift=true end
    end)
    UIS.InputEnded:Connect(function(input)
        local k = input.KeyCode
        if k == Enum.KeyCode.W then pk.W=false elseif k == Enum.KeyCode.A then pk.A=false
        elseif k == Enum.KeyCode.S then pk.S=false elseif k == Enum.KeyCode.D then pk.D=false
        elseif k == Enum.KeyCode.Space then pk.Space=false elseif k == Enum.KeyCode.LeftControl then pk.LeftControl=false
        elseif k == Enum.KeyCode.LeftShift then pk.LeftShift=false end
    end)
end

-- =============================================
-- ALWAYS EQUIP  (re-equips your gun when sitting in a car/horse unequips it, so you can
-- shoot while driving/riding). Reparents the last-held tool back from the backpack.
-- =============================================
local ohLastTool = nil
task.spawn(function()
    while true do
        if AR2_CFG.misc.alwaysEquip then
            pcall(function()
                local char = LP.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if char and hum and hum.Health > 0 then
                    local held = char:FindFirstChildOfClass("Tool")
                    if held then ohLastTool = held.Name
                    elseif ohLastTool then
                        local bp = LP:FindFirstChildOfClass("Backpack")
                        local tool = bp and bp:FindFirstChild(ohLastTool)
                        if tool and tool:IsA("Tool") then tool.Parent = char
                        elseif bp then
                            for _, t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then ohLastTool = t.Name t.Parent = char break end end
                        end
                    end
                end
            end)
        end
        task.wait(0.2)
    end
end)

-- =============================================
-- KEYBINDS
-- =============================================
local AR2_keybindActions = {}
local function AR2_registerKeybind(n, f) AR2_keybindActions[n] = f end
local function fireKeybind(keyName)
    for feature, key in pairs(AR2_CFG.keybinds) do
        if key and key ~= "None" and key ~= "" then
            if AR2_normalizeKey(key) == keyName then
                local action = AR2_keybindActions[feature]
                if action then pcall(action) end
            end
        end
    end
end
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    local p = inputToKeyName(input)
    if p then fireKeybind(p) end
end)
AR2_registerKeybind("playerESP", function() AR2_CFG.players.on = not AR2_CFG.players.on end)
AR2_registerKeybind("zombieESP", function() AR2_CFG.zombies.on = not AR2_CFG.zombies.on end)
AR2_registerKeybind("vehicleESP", function() AR2_CFG.vehicles.on = not AR2_CFG.vehicles.on end)
AR2_registerKeybind("corpseESP", function() AR2_CFG.corpses.on = not AR2_CFG.corpses.on end)
AR2_registerKeybind("lootESP", function() AR2_CFG.loot.on = not AR2_CFG.loot.on end)
AR2_registerKeybind("dropESP", function() AR2_CFG.drops.on = not AR2_CFG.drops.on end)
AR2_registerKeybind("aimbot", function() AR2_CFG.aim.on = not AR2_CFG.aim.on end)
AR2_registerKeybind("silentAim", function() SA.on = not SA.on end)
AR2_registerKeybind("silentAim360", function() SA.on = not SA.on SA.mode360 = not SA.mode360 end)
AR2_registerKeybind("aimbot360", function() AR2_CFG.aim.on = not AR2_CFG.aim.on AR2_CFG.aim.mode360 = not AR2_CFG.aim.mode360 end)
AR2_registerKeybind("triggerBot", function() AR2_CFG.trigger.on = not AR2_CFG.trigger.on end)
AR2_registerKeybind("triggerBot360", function() AR2_CFG.trigger.on = not AR2_CFG.trigger.on AR2_CFG.trigger.mode360 = not AR2_CFG.trigger.mode360 end)
AR2_registerKeybind("noclip", function() if AR2_CFG.misc.noclip then AR2_stopNoclip() else AR2_startNoclip() end end)
AR2_registerKeybind("infJump", function() AR2_CFG.misc.infJump = not AR2_CFG.misc.infJump end)
AR2_registerKeybind("freezeZ", function() AR2_CFG.misc.freeze = not AR2_CFG.misc.freeze end)
AR2_registerKeybind("rapidFire", function() AR2_CFG.misc.rapidFire = not AR2_CFG.misc.rapidFire end)
AR2_registerKeybind("fullbright", function() AR2_CFG.misc.fullbright = not AR2_CFG.misc.fullbright if AR2_CFG.misc.fullbright then pcall(applyFullbright) else pcall(restoreFullbright) end end)
AR2_registerKeybind("noFog", function() AR2_CFG.misc.noFog = not AR2_CFG.misc.noFog if AR2_CFG.misc.noFog then applyNoFog() else restoreFog() end end)
AR2_registerKeybind("personalChams", function() setPersonalChar(not AR2_CFG.misc.personalCharOn) end)
AR2_registerKeybind("tungSelf", function() setTungSelf(not tungSelfActive) end)
AR2_registerKeybind("tungGlobal", function() setTungGlobal(not tungGlobalActive) end)
AR2_registerKeybind("distZoom", function() setDistZoomEnabled(not AR2_CFG.misc.distZoomEnabled) end)
AR2_registerKeybind("fovZoom", function() setFOVZoom(not AR2_CFG.misc.fovZoom) end)
AR2_registerKeybind("carNoclip", function() carNoclipSetEnabled(not cnActive) end)
AR2_registerKeybind("carFly", function() AR2_setFly(not AR2_CFG.cars.fly) end)
AR2_registerKeybind("carEnter", function() AR2_enterNearestVehicle() end)
AR2_registerKeybind("carPark", function() AR2_parkVehicle() end)
AR2_registerKeybind("carRepair", function() AR2_repairVehicle() end)
AR2_registerKeybind("carClip", function() AR2_setCarClip(not AR2_CFG.cars.clipOff) end)
AR2_registerKeybind("walkSpeed", function() AR2_setWalkSpeed(not AR2_CFG.move.walkOn) end)
AR2_registerKeybind("tpNearestP", function() AR2_teleportToNearestPlayer() end)
AR2_registerKeybind("tpNearestZ", function() AR2_teleportToNearestZombie() end)
AR2_registerKeybind("tpNearestC", function() AR2_teleportToNearestCorpse() end)
AR2_registerKeybind("mapFake", function() AR2_toggleMapTP(nil) end)
AR2_registerKeybind("deathTP", function() AR2_teleportToDeath() end)
AR2_registerKeybind("wallbang", function() SA.wTrim = not SA.wTrim end)
AR2_registerKeybind("spinBot", function() AR2_setSpin(not AR2_spin.on) end)
AR2_registerKeybind("hitbox", function() AR2_CFG.weapon.hitbox = not AR2_CFG.weapon.hitbox end)
AR2_registerKeybind("alwaysEquip", function() AR2_CFG.misc.alwaysEquip = not AR2_CFG.misc.alwaysEquip end)
AR2_registerKeybind("horseFly", function() AR2_setHorseFly(not AR2_CFG.cars.horseFly) end)
AR2_registerKeybind("playerFly", function() AR2_setPlayerFly(not AR2_CFG.misc.playerFly) end)
AR2_registerKeybind("forceAds", function() AR2_CFG.trigger.forceAds = not AR2_CFG.trigger.forceAds end)
AR2_registerKeybind("horseESP", function() AR2_CFG.horses.on = not AR2_CFG.horses.on end)
AR2_registerKeybind("horseEnter", function() AR2_enterNearestHorse() end)
AR2_registerKeybind("horseSpeed", function() AR2_CFG.cars.horseSpeed = not AR2_CFG.cars.horseSpeed end)
AR2_registerKeybind("horseJump", function() AR2_CFG.cars.horseJump = not AR2_CFG.cars.horseJump end)

AR2_trigStatus = "off"
RunSvc.Heartbeat:Connect(function()
    local t = AR2_CFG.trigger
    if not t.on then AR2_trigStatus = "off" if trigFiring and os.clock() >= trigReleaseAt then trigRelease() end return end
    if trigFiring then if os.clock() >= trigReleaseAt then trigRelease() end return end
    if t.holdRMB and not UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then AR2_trigStatus = "waiting for RMB" return end
    local char = LP.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not hum or hum.Health <= 0 or not root then AR2_trigStatus = "no character" return end
    local target, how
    if t.useSilent and SA and SA.on and SA_target and SA_target.pos then
        local d = (SA_target.pos - root.Position).Magnitude
        if d <= t.maxDist then target, how = SA_target.pos, "silent" end
    end
    if not target then
        local p
        if t.mode360 then p = findTriggerTarget360() else p = findTriggerTarget() end
        if p then
            local pos = (typeof(p) == "Instance" and p.Position) or p
            if typeof(pos) == "Vector3" and (pos - root.Position).Magnitude <= t.maxDist then target, how = pos, t.mode360 and "360" or "fov" end
        end
    end
    if not target then AR2_trigStatus = "no target" return end
    if not toolReady() then AR2_trigStatus = "no weapon" return end
    local now = os.clock()
    if now - trigLastShot < t.delay then return end
    trigLastShot = now
    AR2_trigStatus = "FIRING (" .. tostring(how) .. ")"
    trigPress() trigReleaseAt = now + 0.015
    local n = math.floor(t.burst or 1)
    if n > 1 then
        task.spawn(function()
            for _ = 2, n do
                task.wait(t.burstGap or 0.04)
                if not AR2_CFG.trigger.on then break end
                pcall(trigPress) task.wait(0.015) pcall(trigRelease)
            end
        end)
    end
end)

RunSvc.RenderStepped:Connect(function()
    if AR2_CFG.aim.mode360 or AR2_CFG.trigger.mode360 then
        local target = findTarget360()
        if target then
            local sp, onScr = Cam:WorldToViewportPoint(target.Position)
            local centre = Vector2.new(Cam.ViewportSize.X/2, Cam.ViewportSize.Y/2)
            lockLine.From = centre
            if onScr and sp.Z > 0 then lockLine.To = Vector2.new(sp.X, sp.Y) else lockLine.To = centre + Vector2.new(0, -centre.Y * 0.9) end
            lockLine.Color = Color3.fromRGB(255,90,90) lockLine.Visible = true
        else lockLine.Visible = false end
    else lockLine.Visible = false end
end)

-- =============================================
-- HORSES  (ESP list + auto-enter + speed/jump mods). Horses are NPCs in AI.Wildlife.
-- =============================================
local AR2_HORSE_CACHE = { list = {}, at = 0 }
local function refreshHorseList()
    AR2_HORSE_CACHE.at = os.clock()
    local list = {}
    local ai = workspace:FindFirstChild("AI")
    local wl = ai and ai:FindFirstChild("Wildlife")
    if wl then
        for _, m in ipairs(wl:GetChildren()) do
            if m:IsA("Model") then
                local prim = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
                local hum = m:FindFirstChildOfClass("Humanoid")
                if prim and prim:IsA("BasePart") and hum then list[#list+1] = { model = m, prim = prim, hum = hum, name = m.Name } end
            end
        end
    end
    AR2_HORSE_CACHE.list = list
end
local function getHorseList()
    if os.clock() - AR2_HORSE_CACHE.at >= 1.0 then refreshHorseList() end
    return AR2_HORSE_CACHE.list
end
local function nearestHorse(range)
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local best, bestD = nil, range or 20
    for _, h in ipairs(getHorseList()) do
        if h.prim and h.prim.Parent then
            local d = (h.prim.Position - root.Position).Magnitude
            if d < bestD then bestD, best = d, h end
        end
    end
    return best
end
-- Nearest Horse teleport: uses the SAME underground drop-through method as Nearest Player
-- (via AR2_teleportStart, live-tracking the horse's root so a wandering horse is still caught),
-- then once it settles it fires the horse's prompt + taps E to actually mount.
function AR2_teleportToNearestHorse()
    refreshHorseList()
    local best = nearestHorse(1e9)          -- nearest horse at ANY distance, like Nearest Player
    if not best then AR2_notify("Origin HUB", "No horses found", 2) return false end
    local target = best.model
    local started = AR2_teleportStart({ part = best.prim })
    if not started then return false end
    AR2_notify("Origin HUB", "Going to nearest horse (" .. best.name .. ")", 3)
    task.spawn(function()
        for _ = 1, 120 do if not TP.active then break end task.wait(0.1) end   -- wait for TP to settle
        task.wait(0.25)
        if target and target.Parent then
            pcall(function() for _, d in ipairs(target:GetDescendants()) do if d:IsA("ProximityPrompt") and fireproximityprompt then fireproximityprompt(d) end end end)
            pcall(function() VIM:SendKeyEvent(true, Enum.KeyCode.E, false, game) task.wait(0.05) VIM:SendKeyEvent(false, Enum.KeyCode.E, false, game) end)
        end
    end)
    return true
end
AR2_enterNearestHorse = AR2_teleportToNearestHorse   -- back-compat alias (keybind)
-- speed / jump mods for the horse you're on (nearest within ~24 studs).
-- SPEED: the horse reads the RIDER's WalkSpeed (confirmed in-game -- the player-walkspeed trick
-- makes the horse gallop faster), so we drive the PLAYER's Humanoid.WalkSpeed here, just like the
-- Movement > Walk Speed feature but with a much larger cap. Gated to when a horse is nearby so it
-- only kicks in while mounted, and restored on dismount (unless the on-foot Walk Speed owns it).
-- JUMP: applied to the horse's own Humanoid (that already works). Scoped in a do-block = no
-- main-chunk registers.
do
    local boosted = false
    RunSvc.Heartbeat:Connect(function()
        local wantSpeed = AR2_CFG.cars.horseSpeed
        local wantJump  = AR2_CFG.cars.horseJump
        if not (wantSpeed or wantJump) then
            if boosted then
                local ph = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
                if ph and not AR2_CFG.move.walkOn then pcall(function() ph.WalkSpeed = 16 end) end
                boosted = false
            end
            return
        end
        pcall(function()
            local h = nearestHorse(24)          -- are we on / next to a horse?
            if wantSpeed then
                local ph = LP.Character and LP.Character:FindFirstChildWhichIsA("Humanoid")
                if ph then
                    if h then
                        -- the horse heavily damps the rider's WalkSpeed, so scale it up hard:
                        -- the 2000 slider maps to an effective WalkSpeed of up to 10000.
                        local tgt = (AR2_CFG.cars.horseSpeedVal or 120) * 5
                        if ph.WalkSpeed ~= tgt then ph.WalkSpeed = tgt end
                        boosted = true
                    elseif boosted and not AR2_CFG.move.walkOn then
                        pcall(function() ph.WalkSpeed = 16 end)  -- dismounted -> restore
                        boosted = false
                    end
                end
            end
            if wantJump and h and h.hum then
                pcall(function() h.hum.UseJumpPower = true end)
                pcall(function() h.hum.JumpPower = AR2_CFG.cars.horseJumpVal or 150 end)
                pcall(function() h.hum.JumpHeight = (AR2_CFG.cars.horseJumpVal or 150) / 10 end)
            end
        end)
    end)
end

-- =============================================
-- MAIN ESP RENDER LOOP
-- =============================================
RunSvc.RenderStepped:Connect(function()
    if AR2_CFG.aim.on and (not AR2_CFG.aim.hold or UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)) then
        pcall(function()
            local target = AR2_CFG.aim.mode360 and findTarget360() or findAimTarget()
            if target and target.Parent then
                local camPos = Cam.CFrame.Position
                local dir = (target.Position - camPos).Unit
                local smooth = 1 / math.max(1, AR2_CFG.aim.smooth or 1)
                Cam.CFrame = Cam.CFrame:Lerp(CFrame.new(camPos, camPos + dir), smooth)
            end
        end)
    end
    local centre = Vector2.new(Cam.ViewportSize.X/2, Cam.ViewportSize.Y/2)
    if AR2_CFG.aim.on and AR2_CFG.aim.showFov and not AR2_CFG.aim.mode360 then
        aimFovCircle.Position = centre aimFovCircle.Radius = AR2_CFG.aim.fov aimFovCircle.Color = Color3.fromRGB(255,90,90) aimFovCircle.Visible = true
    else aimFovCircle.Visible = false end
    if SA.on and AR2_CFG.silent.showFov and not SA.mode360 then
        local _halfH = Cam.ViewportSize.Y / 2
        local _camHalf = math.rad(Cam.FieldOfView / 2)
        local _r = (_camHalf > 0) and (_halfH * math.tan(math.rad(SA.fovDeg or 25)) / math.tan(_camHalf)) or 150
        silentFovCircle.Position = centre silentFovCircle.Radius = _r silentFovCircle.Color = Color3.fromRGB(255,120,220) silentFovCircle.Visible = true
    else silentFovCircle.Visible = false end
    if AR2_CFG.trigger.on and not AR2_CFG.trigger.mode360 then
        trigFovCircle.Position = centre trigFovCircle.Radius = AR2_CFG.trigger.fov trigFovCircle.Color = Color3.fromRGB(80,220,90) trigFovCircle.Visible = true
    else trigFovCircle.Visible = false end
    if AR2_CFG.misc.personalCharOn and AR2_CFG.misc.personalCharRainbow then pcall(applyPersonalChar) end
    if AR2_CFG.misc.personalGunOn and AR2_CFG.misc.personalGunRainbow then pcall(applyPersonalGun) end
    -- maintain fullbright / no-fog inline (the apply* functions are defined later in the
    -- file, so they are nil in this closure -- calling them was the "attempt to call nil")
    if AR2_CFG.misc.fullbright then
        Lighting.Brightness = AR2_CFG.misc.brightness or 3
        Lighting.GlobalShadows = false
        Lighting.Ambient = Color3.fromRGB(180,180,180) Lighting.OutdoorAmbient = Color3.fromRGB(200,200,200)
    end
    if AR2_CFG.misc.noFog then Lighting.FogEnd = 1e6 Lighting.FogStart = 0 end
    if AR2_CFG.misc.setTime then Lighting.ClockTime = AR2_CFG.misc.timeValue end
    if AR2_CFG.misc.removeShadows then Lighting.GlobalShadows = false end

    hideAll()
    local camPos = Cam.CFrame.Position
    local slot = 0
    local ts = AR2_CFG.esp.textSize or 13

    if AR2_CFG.players.on then
        local c = AR2_CFG.players
        for _, pl in ipairs(Players:GetPlayers()) do
            if pl ~= LP and pl.Character then
                local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then
                    local root = pl.Character:FindFirstChild("HumanoidRootPart")
                    local d = root and d3(root.Position, camPos) or math.huge
                    if d <= c.maxDist then
                        local x, y, w, h = sa4PlayerBox(pl.Character)
                        if x and y and w and h then
                            slot = slot + 1
                            local e = getDraw(slot)
                            local isT = (SA_target and SA_target.model == pl.Character)
                            local col = isT and Color3.fromRGB(255,80,80) or Color3.fromRGB(90,190,255)
                            drawEspBox(e, x, y, w, h, col, c.box)
                            local nmTxt = pl.Name
                            if c.weapon then local tool = pl.Character:FindFirstChildOfClass("Tool") nmTxt = nmTxt .. (tool and ("  [" .. tool.Name .. "]") or "  [unarmed]") end
                            e.name.Text = nmTxt e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                            local _hp = hum and math.floor(hum.Health) or -1
                            e.dist.Text = string.format("%dm  hp %d", math.floor(d), _hp) e.dist.Size = ts-1
                            e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = Color3.fromRGB(210,210,210)
                            if c.health then local hp, mx = AR2_readEntityHealth(pl.Character) if hp and mx and mx > 0 then drawHealthBar(e, x, y, h, hp/mx) end end
                        end
                    end
                end
            end
        end
    end

    if AR2_CFG.zombies.on then
        local c = AR2_CFG.zombies
        local container = getZombieContainer()
        if container then
            for _, m in ipairs(container:GetChildren()) do
                if m:IsA("Model") then
                    local hum = m:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local root = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
                        local d = root and d3(root.Position, camPos) or math.huge
                        if d <= c.maxDist then
                            local x, y, w, h = getModelBox(m)
                            if x and y and w and h then
                                slot = slot + 1
                                local e = getDraw(slot)
                                local col = Color3.fromRGB(150,255,120)
                                drawEspBox(e, x, y, w, h, col, c.box)
                                e.name.Text = m.Name e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                                e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = col
                                if c.health then local hp, mx = AR2_readEntityHealth(m) if hp and mx and mx > 0 then drawHealthBar(e, x, y, h, hp/mx) end end
                            end
                        end
                    end
                end
            end
        end
    end

    if AR2_CFG.vehicles.on then
        local c = AR2_CFG.vehicles
        for _, container in ipairs(getVehicleContainers()) do
            for _, m in ipairs(container:GetChildren()) do
                if m:IsA("Model") then
                    local prim = m.PrimaryPart or m:FindFirstChild("DriveSeat") or m:FindFirstChild("Base") or m:FindFirstChildWhichIsA("BasePart")
                    if prim then
                        local d = d3(prim.Position, camPos)
                        if d <= c.maxDist then
                            local x, y, w, h = getWideModelBox(m)
                            if not (x and y and w and h) then x, y, w, h = getModelBox(m) end
                            if not (x and y and w and h) then x, y, w, h = getPartBox(prim) end
                            if x and y and w and h then
                                slot = slot + 1
                                local e = getDraw(slot)
                                local col = Color3.fromRGB(90,200,255)
                                local vnm = m.Name
                                if c.moving then
                                    local vel = 0
                                    pcall(function() vel = prim.AssemblyLinearVelocity.Magnitude end)
                                    if vel > 3 then col = Color3.fromRGB(90,255,140) vnm = vnm .. "  [MOVING " .. math.floor(vel) .. "]"
                                    else vnm = vnm .. "  [parked]" end
                                end
                                drawEspBox(e, x, y, w, h, col, c.box)
                                e.name.Text = vnm e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                                e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = col
                            end
                        end
                    end
                end
            end
        end
    end

    if AR2_CFG.horses.on then
        local c = AR2_CFG.horses
        for _, h in ipairs(getHorseList()) do
            if h.prim and h.prim.Parent then
                local d = d3(h.prim.Position, camPos)
                if d <= c.maxDist then
                    local x, y, w, hh = getWideModelBox(h.model)
                    if not (x and y and w and hh) then x, y, w, hh = getModelBox(h.model) end
                    if not (x and y and w and hh) then x, y, w, hh = getPartBox(h.prim) end
                    if x and y and w and hh then
                        slot = slot + 1
                        local e = getDraw(slot)
                        local col = Color3.fromRGB(190,140,80)
                        drawEspBox(e, x, y, w, hh, col, c.box)
                        e.name.Text = h.name e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                        e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+hh+2) e.dist.Visible = c.dist e.dist.Color = col
                    end
                end
            end
        end
    end

    if AR2_CFG.corpses.on then
        local c = AR2_CFG.corpses
        for _, m in ipairs(getCorpseList()) do
            if m and m.Parent then
                local root = m:FindFirstChild("HumanoidRootPart") or m:FindFirstChild("Head") or m:FindFirstChild("Head2")
                if root and root:IsA("BasePart") then
                    local d = (root.Position - camPos).Magnitude
                    if d <= c.maxDist then
                        local x, y, w, h = corpseBox(m)
                        if x and y and w and h then
                            slot = slot + 1
                            local e = getDraw(slot)
                            local col = Color3.fromRGB(190,190,190)
                            drawEspBox(e, x, y, w, h, col, c.box)
                            e.name.Text = "Corpse" e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                            e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = col
                        end
                    end
                end
            end
        end
    end

    if AR2_CFG.drops.on then
        local c = AR2_CFG.drops
        for _, entry in ipairs(getDropList()) do
            if entry.prim and entry.prim.Parent then
                local d = (entry.prim.Position - camPos).Magnitude
                if d <= c.maxDist then
                    local x, y, w, h
                    if entry.model:IsA("Model") then x, y, w, h = getModelBox(entry.model) else x, y, w, h = getPartBox(entry.prim) end
                    if x and y and w and h then
                        slot = slot + 1
                        local e = getDraw(slot)
                        local col = Color3.fromRGB(255,110,255)
                        drawEspBox(e, x, y, w, h, col, c.box)
                        e.name.Text = entry.model.Name e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                        e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = col
                        if c.highlight then setLootHighlight(entry.model, col) end
                    end
                end
            end
        end
    end

    if AR2_CFG.loot.on then
        local c = AR2_CFG.loot
        local list = getLootList()
        local inRange = {}
        for _, item in ipairs(list) do
            if item.prim and item.prim.Parent then
                local d = (item.prim.Position - camPos).Magnitude
                if d <= c.maxDist then inRange[#inRange+1] = { item = item, d = d } end
            end
        end
        table.sort(inRange, function(a, b) return a.d < b.d end)
        local drawCap = c.maxDraw or 40
        local hlCap = c.maxHighlight or 20
        local seenHl = {}
        for i = 1, math.min(#inRange, drawCap) do
            local entry = inRange[i]
            local m = entry.item.model
            local prim = entry.item.prim
            local d = entry.d
            local col = TIER_COLOR[entry.item.tier] or TIER_COLOR.Other
            local x, y, w, h
            if m:IsA("Model") then x, y, w, h = getModelBox(m) else x, y, w, h = getPartBox(prim) end
            if x and y and w and h then
                slot = slot + 1
                local e = getDraw(slot)
                drawEspBox(e, x, y, w, h, col, c.box)
                e.name.Text = entry.item.disp e.name.Size = ts e.name.Position = Vector2.new(x+w/2, y-16) e.name.Visible = c.name e.name.Color = col
                e.dist.Text = string.format("%.0f", d) e.dist.Size = ts-1 e.dist.Position = Vector2.new(x+w/2, y+h+2) e.dist.Visible = c.dist e.dist.Color = col
            end
            if c.highlight and i <= hlCap then seenHl[m] = true setLootHighlight(m, col) end
        end
        if c.highlight then
            for obj in pairs(AR2_lootHighlights) do
                if not seenHl[obj] then local hl = AR2_lootHighlights[obj] if hl then hl:Destroy() end AR2_lootHighlights[obj] = nil end
            end
        else clearLootHighlights() end
    else clearLootHighlights() end
end)

-- =============================================
-- WORLD / LIGHTING  (+ weather + grass removal)
-- =============================================
local worldSaved = nil
local addedCC, addedBloom = false, false
local skyboxOriginal = { Ambient=Lighting.Ambient, OutdoorAmbient=Lighting.OutdoorAmbient, FogColor=Lighting.FogColor, FogStart=Lighting.FogStart, FogEnd=Lighting.FogEnd, ClockTime=Lighting.ClockTime }
local function saveWorld()
    if worldSaved then return end
    worldSaved = { Brightness=Lighting.Brightness, Ambient=Lighting.Ambient, OutdoorAmbient=Lighting.OutdoorAmbient,
        GlobalShadows=Lighting.GlobalShadows, ExposureCompensation=Lighting.ExposureCompensation,
        FogEnd=Lighting.FogEnd, FogStart=Lighting.FogStart, ClockTime=Lighting.ClockTime }
    local cc = Lighting:FindFirstChildOfClass("ColorCorrectionEffect")
    if cc then worldSaved.cc = {inst=cc, S=cc.Saturation, C=cc.Contrast, B=cc.Brightness, E=cc.Enabled} end
    local bl = Lighting:FindFirstChildOfClass("BloomEffect")
    if bl then worldSaved.bl = {inst=bl, I=bl.Intensity, S=bl.Size, T=bl.Threshold, E=bl.Enabled} end
    local air = Lighting:FindFirstChildOfClass("Atmosphere")
    if air then worldSaved.atmo = air.Density end
end
local function applyFullbright()
    saveWorld()
    Lighting.Brightness = AR2_CFG.misc.brightness or 3
    Lighting.Ambient = Color3.fromRGB(180,180,180) Lighting.OutdoorAmbient = Color3.fromRGB(200,200,200)
    Lighting.GlobalShadows = false Lighting.ExposureCompensation = 0.45
    local cc = worldSaved.cc and worldSaved.cc.inst or Lighting:FindFirstChildOfClass("ColorCorrectionEffect")
    if not cc or not cc.Parent then cc = Instance.new("ColorCorrectionEffect") cc.Name="oh_cc" cc.Parent=Lighting addedCC=true end
    cc.Saturation, cc.Contrast, cc.Brightness, cc.Enabled = 0.25, 0.10, 0.02, true
    local bl = worldSaved.bl and worldSaved.bl.inst or Lighting:FindFirstChildOfClass("BloomEffect")
    if not bl or not bl.Parent then bl = Instance.new("BloomEffect") bl.Name="oh_bloom" bl.Parent=Lighting addedBloom=true end
    bl.Intensity, bl.Size, bl.Threshold, bl.Enabled = 0.20, 22, 0.95, true
end
local function restoreFullbright()
    if not worldSaved then return end
    Lighting.Brightness = worldSaved.Brightness Lighting.Ambient = worldSaved.Ambient
    Lighting.OutdoorAmbient = worldSaved.OutdoorAmbient Lighting.GlobalShadows = worldSaved.GlobalShadows Lighting.ExposureCompensation = worldSaved.ExposureCompensation
    if addedCC then local cc = Lighting:FindFirstChild("oh_cc") if cc then cc:Destroy() end addedCC=false
    elseif worldSaved.cc and worldSaved.cc.inst and worldSaved.cc.inst.Parent then
        worldSaved.cc.inst.Saturation, worldSaved.cc.inst.Contrast, worldSaved.cc.inst.Brightness, worldSaved.cc.inst.Enabled = worldSaved.cc.S, worldSaved.cc.C, worldSaved.cc.B, worldSaved.cc.E end
    if addedBloom then local bl = Lighting:FindFirstChild("oh_bloom") if bl then bl:Destroy() end addedBloom=false
    elseif worldSaved.bl and worldSaved.bl.inst and worldSaved.bl.inst.Parent then
        worldSaved.bl.inst.Intensity, worldSaved.bl.inst.Size, worldSaved.bl.inst.Threshold, worldSaved.bl.inst.Enabled = worldSaved.bl.I, worldSaved.bl.S, worldSaved.bl.T, worldSaved.bl.E end
end
local function applyNoFog() saveWorld() Lighting.FogEnd = 1e6 Lighting.FogStart = 0 local air = Lighting:FindFirstChildOfClass("Atmosphere") if air then pcall(function() air.Density = 0 air.Haze = 0 air.Glare = 0 end) end end
local function restoreFog() if not worldSaved then return end Lighting.FogEnd = worldSaved.FogEnd Lighting.FogStart = worldSaved.FogStart local air = Lighting:FindFirstChildOfClass("Atmosphere") if air and worldSaved.atmo then air.Density = worldSaved.atmo end end

-- keep No Fog / Fullbright asserted WITHOUT flicker. The old 1s poll let the game's day/night
-- value show for up to a second before snapping back = flicker. Instead, re-assert EVERY FRAME
-- and ONLY write the values that are actually wrong -- the game's changes are corrected within one
-- frame and never visibly render. The heavy setup (effects/atmosphere) still happens once via the
-- toggle/config; this only maintains the cheap scalar values.
do
    local FB_AMB  = Color3.fromRGB(180,180,180)
    local FB_OAMB = Color3.fromRGB(200,200,200)
    local air = nil
    RunSvc.RenderStepped:Connect(function()
        if AR2_CFG.misc.noFog then
            if Lighting.FogEnd ~= 1e6 then Lighting.FogEnd = 1e6 end
            if Lighting.FogStart ~= 0 then Lighting.FogStart = 0 end
            -- the real haze in this game is an Atmosphere object -- zero its Density/Haze/Glare every
            -- frame (the game re-applies them, which is why No Fog "didn't work"). Re-find it if gone.
            if not air or not air.Parent then air = Lighting:FindFirstChildOfClass("Atmosphere") end
            if air then
                if air.Density ~= 0 then air.Density = 0 end
                if air.Haze ~= 0 then air.Haze = 0 end
                if air.Glare ~= 0 then air.Glare = 0 end
            end
        end
        if AR2_CFG.misc.fullbright then
            local b = AR2_CFG.misc.brightness or 3
            if Lighting.Brightness ~= b then Lighting.Brightness = b end
            if Lighting.GlobalShadows then Lighting.GlobalShadows = false end
            if Lighting.Ambient ~= FB_AMB then Lighting.Ambient = FB_AMB end
            if Lighting.OutdoorAmbient ~= FB_OAMB then Lighting.OutdoorAmbient = FB_OAMB end
        end
    end)
end

-- =============================================
-- MODEL MORPH  (troll): load any public model by asset ID and weld it to your character so you
-- "become" it (e.g. a Verity model). Uses the executor's game:GetObjects (InsertService fallback),
-- same idea as the custom-ID loading in the script you shared. NOTE: game:GetObjects builds the
-- model on the CLIENT, so YOU see the morph but other players usually won't (client-side only).
-- =============================================
AR2_morph = AR2_morph or { model = nil }
AR2_hideSaved = AR2_hideSaved or {}
function AR2_restoreHide()   -- put every part's original Transparency back
    for part, t in pairs(AR2_hideSaved) do if part and part.Parent then pcall(function() part.Transparency = t end) end end
    AR2_hideSaved = {}
end
function AR2_removeMorph()
    if AR2_morph.model then pcall(function() AR2_morph.model:Destroy() end) AR2_morph.model = nil end
end
function AR2_applyMorph()
    AR2_removeMorph()
    local id = tostring(AR2_CFG.misc.morphId or ""):gsub("%D", "")
    if id == "" then AR2_notify("Origin HUB", "Enter a model ID first", 2) return false end
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then AR2_notify("Origin HUB", "No character", 2) return false end
    -- weld to the TORSO, not the HRP: the Spin Bot rotates the torso via the RootJoint (the HRP
    -- itself never turns), and the torso also turns with normal movement -- so the morph follows both.
    local anchor = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso") or char:FindFirstChild("LowerTorso") or hrp
    local objs
    local ok = pcall(function() objs = game:GetObjects("rbxassetid://" .. id) end)
    if not ok or type(objs) ~= "table" or #objs == 0 then
        objs = nil
        pcall(function() local m = game:GetService("InsertService"):LoadAsset(tonumber(id)) if m then objs = m:GetChildren() end end)
    end
    if type(objs) ~= "table" or #objs == 0 then AR2_notify("Origin HUB", "Couldn't load model " .. id .. " (private/invalid?)", 4) return false end
    local model = objs[1]
    for _, o in ipairs(objs) do if o:IsA("Model") then model = o break end end
    model.Name = "oh_morph" model.Parent = workspace
    pcall(function() if model:IsA("Model") then model:PivotTo(hrp.CFrame) elseif model:IsA("BasePart") then model.CFrame = hrp.CFrame end end)
    -- strip scripts/humanoid and the model's own joints so every part pins rigidly to us (no flop)
    pcall(function() for _, d in ipairs(model:GetDescendants()) do
        if d:IsA("Script") or d:IsA("LocalScript") or d:IsA("Humanoid")
        or d:IsA("Weld") or d:IsA("WeldConstraint") or d:IsA("Motor6D") or d:IsA("Snap") or d:IsA("ManualWeld") then d:Destroy() end
    end end)
    local parts = {}
    if model:IsA("Model") then for _, d in ipairs(model:GetDescendants()) do if d:IsA("BasePart") then parts[#parts+1] = d end end
    elseif model:IsA("BasePart") then parts[1] = model end
    if #parts == 0 then pcall(function() model:Destroy() end) AR2_notify("Origin HUB", "That asset has no parts to weld", 3) return false end
    for _, p in ipairs(parts) do
        pcall(function()
            p.Anchored = false p.CanCollide = false p.Massless = true
            local w = Instance.new("Weld") w.Part0 = anchor w.Part1 = p
            w.C0 = anchor.CFrame:ToObjectSpace(p.CFrame) w.Parent = p
        end)
    end
    AR2_morph.model = model
    AR2_notify("Origin HUB", "Morph applied (" .. #parts .. " parts)", 3)
    return true
end
-- re-apply the morph after respawn so it isn't lost
LP.CharacterAdded:Connect(function()
    if AR2_morph.model then task.wait(1.5) pcall(AR2_applyMorph) end
end)
-- keep the player + equipped weapon FULLY invisible (locally) while Hide My Character is on. The
-- game resets transparency when you aim/zoom/equip a gun, so a one-shot hide gets undone -- we
-- re-assert LocalTransparencyModifier every frame on all body/weapon parts, decals and textures.
-- Hide My Character: EXACTLY like Character Chams, but sets Transparency = 1 (fully invisible) on
-- your character AND your held gun's parts (meshes, decals, textures). Saves each part's original
-- Transparency and restores it when turned off. Re-asserted each frame so equipping/aiming can't
-- un-hide you. NEVER touches the camera/viewmodel -- that's what broke shooting before.
do
    local wasHiding = false
    RunSvc.RenderStepped:Connect(function()
        local hide = AR2_CFG.misc.morphHideSelf and true or false
        if hide then
            local char = LP.Character
            if char then
                for _, p in ipairs(char:GetDescendants()) do
                    if p:IsA("BasePart") or p:IsA("Decal") or p:IsA("Texture") then
                        if AR2_hideSaved[p] == nil then AR2_hideSaved[p] = p.Transparency end
                        if p.Transparency ~= 1 then pcall(function() p.Transparency = 1 end) end
                    end
                end
            end
        elseif wasHiding then
            AR2_restoreHide()
        end
        wasHiding = hide
    end)
end

-- FORCE STREAM: fight StreamingEnabled culling so distant players/parts load in.
-- Cranks the client streaming + simulation radius and keeps requesting streaming around
-- the map. ONLY works if the game uses client-side instance streaming -- if the server
-- culls distant players on its end, nothing here can pull them in (they're never sent).
local streamConn = nil
local streamOrigRadius = nil
local function setForceStream(on)
    AR2_CFG.misc.forceStream = on and true or false
    if streamConn then streamConn:Disconnect() streamConn = nil end
    if not on then
        pcall(function() if streamOrigRadius then workspace.StreamingTargetRadius = streamOrigRadius end end)
        return
    end
    -- GENTLE: the old version cranked the radius to 100k and re-requested streaming every
    -- frame, which choked the client and fried ESP. Now: a moderate radius bump + a request
    -- only every few seconds. (Still can't beat server-side culling; this only helps client streaming.)
    pcall(function() if streamOrigRadius == nil then streamOrigRadius = workspace.StreamingTargetRadius end end)
    pcall(function() workspace.StreamingTargetRadius = math.max(streamOrigRadius or 0, 4000) end)
    local acc = 3
    streamConn = RunSvc.Heartbeat:Connect(function(dt)
        if not AR2_CFG.misc.forceStream then return end
        acc = acc + dt
        if acc < 3 then return end   -- throttle: request at most every 3s so it never floods
        acc = 0
        local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if hrp then pcall(function() LP:RequestStreamingAround(hrp.Position) end) end
    end)
end
local savedTextures = {}
local function toggleTextures(on)
    if on then for _, obj in ipairs(workspace:GetDescendants()) do if (obj:IsA("Texture") or obj:IsA("Decal")) and obj.Transparency < 1 and not savedTextures[obj] then savedTextures[obj] = obj.Transparency obj.Transparency = 1 end end
    else for obj, t in pairs(savedTextures) do if obj and obj.Parent then obj.Transparency = t end end savedTextures = {} end
end
-- REMOVE FOLIAGE: trees, bushes, plants, grass meshes AND terrain grass in one toggle.
-- Uses the game's own procedural-foliage tags (FoliageLocalOffset / OriginalSize) + its
-- foliage folders. DescendantAdded events hide streamed-in foliage the instant it appears,
-- and a re-assert loop re-hides anything the game turns back on -> no flicker.
local FOLIAGE_FOLDERS = { "GeneratedFoliage", "ClientProceduralFoliage", "ThumbnailProceduralFoliage" }
local FOLIAGE_KEYWORDS = { "grass","bush","shrub","fern","plant","clover","stick","leaf","leaves","seedling","weed","vine","flower","moss","bramble","reed",
    "tree","pine","oak","birch","palm","fronds","foliage","vegetation","spruce","cedar","willow","trunk" }
local savedFoliage = {}
local foliageConns = {}
local grassWasOn = nil
local function ohIsFoliagePart(part)
    if part:GetAttribute("FoliageLocalOffset") ~= nil then return true end
    if part:GetAttribute("OriginalSize") ~= nil then return true end
    local parent = part.Parent
    if parent then local nm = (parent.Name or ""):lower() for _, k in ipairs(FOLIAGE_KEYWORDS) do if nm:find(k,1,true) then return true end end end
    local sn = (part.Name or ""):lower() for _, k in ipairs(FOLIAGE_KEYWORDS) do if sn:find(k,1,true) then return true end end
    return false
end
local function ohHideFoliage(part)
    if savedFoliage[part] == nil then savedFoliage[part] = part.Transparency end
    pcall(function() part.Transparency = 1 part.LocalTransparencyModifier = 1 part.CanCollide = false part.CanQuery = false part.CanTouch = false end)
end
local function ohFoliageScan(fullScan)
    pcall(function()
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            if sethiddenproperty then pcall(sethiddenproperty, terrain, "Decoration", false) end
            for _, child in ipairs(terrain:GetChildren()) do
                if child:IsA("Attachment") or child:IsA("Model") or child:IsA("Folder") then
                    for _, part in ipairs(child:GetDescendants()) do if part:IsA("BasePart") then ohHideFoliage(part) end end
                elseif child:IsA("BasePart") and ohIsFoliagePart(child) then ohHideFoliage(child) end
            end
        end
        for _, name in ipairs(FOLIAGE_FOLDERS) do
            local folder = workspace:FindFirstChild(name)
            if folder then for _, part in ipairs(folder:GetDescendants()) do if part:IsA("BasePart") then ohHideFoliage(part) end end end
        end
        if fullScan then
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("BasePart") and savedFoliage[obj] == nil and ohIsFoliagePart(obj) then ohHideFoliage(obj) end
            end
        end
    end)
end
local function ohWatchFolder(inst)
    table.insert(foliageConns, inst.DescendantAdded:Connect(function(d)
        if AR2_CFG.misc.removeFoliage and d:IsA("BasePart") then task.defer(ohHideFoliage, d) end
    end))
end
local function toggleFoliage(on)
    if on then
        pcall(function() if grassWasOn == nil then grassWasOn = workspace.Terrain.Decoration end workspace.Terrain.Decoration = false end)
        ohFoliageScan(true)
        local terrain = workspace:FindFirstChildOfClass("Terrain") if terrain then ohWatchFolder(terrain) end
        for _, name in ipairs(FOLIAGE_FOLDERS) do local f = workspace:FindFirstChild(name) if f then ohWatchFolder(f) end end
    else
        for _, c in ipairs(foliageConns) do pcall(function() c:Disconnect() end) end foliageConns = {}
        for part, t in pairs(savedFoliage) do if part and part.Parent then pcall(function() part.Transparency = t part.LocalTransparencyModifier = 0 end) end end
        savedFoliage = {}
        pcall(function() if grassWasOn ~= nil then workspace.Terrain.Decoration = grassWasOn end end)
    end
end
-- re-assert transparency on tracked parts while enabled -> kills the "comes back for a frame" flicker
task.spawn(function()
    while true do
        task.wait(0.25)
        if AR2_CFG.misc.removeFoliage then
            for part in pairs(savedFoliage) do if part and part.Parent and part.Transparency ~= 1 then pcall(function() part.Transparency = 1 part.LocalTransparencyModifier = 1 end) end end
        end
    end
end)
local removedDoors = {}
local function toggleDoors(on)
    local mapE = workspace:FindFirstChild("Map") and workspace.Map:FindFirstChild("Elements")
    if not mapE then return end
    if on then local function scan(p) for _, cc in ipairs(p:GetChildren()) do if cc.Name == "Doors" then for _, dc in ipairs(cc:GetChildren()) do for _, part in ipairs(dc:GetChildren()) do if part.Name == "Door" then table.insert(removedDoors,{instance=part,parent=part.Parent}) part.Parent=nil end end end end scan(cc) end end scan(mapE)
    else for _, d in ipairs(removedDoors) do if d.instance and d.parent then d.instance.Parent = d.parent end end removedDoors = {} end
end
local sunRaysSaved = {}
local function toggleSunRays(on) for _, child in ipairs(Lighting:GetChildren()) do if child:IsA("SunRaysEffect") then if sunRaysSaved[child] == nil then sunRaysSaved[child] = child.Enabled end child.Enabled = not on end end end

-- WEATHER: LocalRain / LocalSnow models + any rain/snow ParticleEmitters
local weatherHidden = {}
local function toggleWeather(on)
    local names = { "LocalRain", "LocalSnow", "Rain", "Snow", "Weather", "Blizzard", "Storm" }
    if on then
        for _, n in ipairs(names) do
            local w = workspace:FindFirstChild(n)
            if w and weatherHidden[w] == nil then weatherHidden[w] = w.Parent pcall(function() w.Parent = nil end) end
        end
        for _, d in ipairs(workspace:GetDescendants()) do
            if d:IsA("ParticleEmitter") then
                local ln = d.Name:lower()
                if ln:find("rain") or ln:find("snow") or ln:find("weather") or ln:find("storm") or ln:find("blizzard") then
                    if weatherHidden[d] == nil then weatherHidden[d] = d.Enabled pcall(function() d.Enabled = false end) end
                end
            end
        end
    else
        for obj, val in pairs(weatherHidden) do
            if typeof(val) == "Instance" then pcall(function() obj.Parent = val end)
            elseif obj and obj.Parent then pcall(function() obj.Enabled = val end) end
        end
        weatherHidden = {}
    end
end

local cloudsInstance, cloudsConn = nil, nil
local function findClouds() local c = workspace:FindFirstChildOfClass("Clouds") if not c and workspace.Terrain then c = workspace.Terrain:FindFirstChildOfClass("Clouds") end if not c then c = Lighting:FindFirstChildOfClass("Clouds") end return c end
local function applyClouds()
    local c = cloudsInstance or findClouds() if not c then return end
    cloudsInstance = c
    pcall(function() c.Enabled = not AR2_CFG.misc.removeClouds end)
    if AR2_CFG.misc.modifyClouds and not AR2_CFG.misc.removeClouds then pcall(function() c.Color = AR2_CFG.misc.cloudsColor c.Cover = AR2_CFG.misc.cloudsCover c.Density = AR2_CFG.misc.cloudsDensity end) end
    if not cloudsConn then cloudsConn = RunSvc.RenderStepped:Connect(function() local ci = cloudsInstance if not ci or not ci.Parent then return end if AR2_CFG.misc.removeClouds then pcall(function() ci.Enabled = false end) return end if not AR2_CFG.misc.modifyClouds then return end pcall(function() ci.Color = AR2_CFG.misc.cloudsColor ci.Cover = AR2_CFG.misc.cloudsCover ci.Density = AR2_CFG.misc.cloudsDensity end) end) end
end
local ambientSaved, ambientConn = nil, nil
local function applyCustomAmbient()
    if ambientSaved == nil then ambientSaved = { Ambient = Lighting.Ambient, OutdoorAmbient = Lighting.OutdoorAmbient } end
    if AR2_CFG.misc.customAmbient then
        Lighting.Ambient = AR2_CFG.misc.ambientIndoor Lighting.OutdoorAmbient = AR2_CFG.misc.ambientOutdoor
        if not ambientConn then ambientConn = RunSvc.RenderStepped:Connect(function() if not AR2_CFG.misc.customAmbient then return end Lighting.Ambient = AR2_CFG.misc.ambientIndoor Lighting.OutdoorAmbient = AR2_CFG.misc.ambientOutdoor end) end
    else if ambientConn then ambientConn:Disconnect() ambientConn = nil end if ambientSaved then Lighting.Ambient = ambientSaved.Ambient Lighting.OutdoorAmbient = ambientSaved.OutdoorAmbient end end
end
local function applySkybox(name)
    if name == nil or name == "" then name = AR2_CFG.misc.skybox or "Default" end
    AR2_CFG.misc.skybox = name
    for _, child in ipairs(Lighting:GetChildren()) do if child:IsA("Sky") then child:Destroy() end end
    if name == "Default" then
        Lighting.Ambient = skyboxOriginal.Ambient Lighting.OutdoorAmbient = skyboxOriginal.OutdoorAmbient
        Lighting.FogColor = skyboxOriginal.FogColor
        if not AR2_CFG.misc.noFog then Lighting.FogStart = skyboxOriginal.FogStart Lighting.FogEnd = skyboxOriginal.FogEnd end
        if not AR2_CFG.misc.setTime then Lighting.ClockTime = skyboxOriginal.ClockTime end
    else
        local skyboxes = {
            ["Space"]={Bk="rbxassetid://159454299",Dn="rbxassetid://159454299",Ft="rbxassetid://159454299",Lf="rbxassetid://159454299",Rt="rbxassetid://159454299",Up="rbxassetid://159454299"},
            ["Galaxy"]={Bk="rbxassetid://149397692",Dn="rbxassetid://149397686",Ft="rbxassetid://149397697",Lf="rbxassetid://149397684",Rt="rbxassetid://149397688",Up="rbxassetid://149397702"},
            ["Galaxy 2"]={Bk="rbxassetid://155441936",Dn="rbxassetid://155441802",Ft="rbxassetid://155441818",Lf="rbxassetid://155441777",Rt="rbxassetid://155441874",Up="rbxassetid://155441905"},
            ["Galaxy 3"]={Bk="rbxassetid://135908594667929",Dn="rbxassetid://139584143501514",Ft="rbxassetid://92947876187368",Lf="rbxassetid://72493016739936",Rt="rbxassetid://81731245279712",Up="rbxassetid://88174897344210"},
            ["Saturne"]={Bk="rbxassetid://1898724755",Dn="rbxassetid://1898727189",Ft="rbxassetid://1898722814",Lf="rbxassetid://1898729298",Rt="rbxassetid://1898741025",Up="rbxassetid://1898736761"},
            ["Neptune"]={Bk="rbxassetid://218955819",Dn="rbxassetid://218953419",Ft="rbxassetid://218954524",Lf="rbxassetid://218958493",Rt="rbxassetid://218957134",Up="rbxassetid://218950090"},
            ["Redshift"]={Bk="rbxassetid://401664839",Dn="rbxassetid://401664862",Ft="rbxassetid://401664960",Lf="rbxassetid://401664881",Rt="rbxassetid://401664901",Up="rbxassetid://401664936"},
            ["Pink Daylights"]={Bk="rbxassetid://11555017034",Dn="rbxassetid://11555013415",Ft="rbxassetid://11555010145",Lf="rbxassetid://11555006545",Rt="rbxassetid://11555000712",Up="rbxassetid://11554996247"},
            ["Purple Night"]={Bk="rbxassetid://17279854976",Dn="rbxassetid://17279856318",Ft="rbxassetid://17279858447",Lf="rbxassetid://17279860360",Rt="rbxassetid://17279862234",Up="rbxassetid://17279864507"},
            ["Gray Night"]={Bk="rbxassetid://1618912481",Dn="rbxassetid://1618913943",Ft="rbxassetid://1618913244",Lf="rbxassetid://1618912849",Rt="rbxassetid://1618911568",Up="rbxassetid://1618913654"},
            ["Anime Sky"]={Bk="rbxassetid://18351376859",Dn="rbxassetid://18351374919",Ft="rbxassetid://18351376800",Lf="rbxassetid://18351376469",Rt="rbxassetid://18351376457",Up="rbxassetid://18351377189"},
        }
        local data = skyboxes[name]
        if data then
            if not AR2_CFG.misc.fullbright then Lighting.Ambient = Color3.new(0.5,0.5,0.5) Lighting.OutdoorAmbient = Color3.new(0.8,0.8,0.8) end
            if not AR2_CFG.misc.noFog then Lighting.FogColor = Color3.new(0.5,0.5,0.5) Lighting.FogStart = 0 Lighting.FogEnd = 1000 end
            if not AR2_CFG.misc.setTime then Lighting.ClockTime = 0 end
            local sky = Instance.new("Sky") sky.Name = name
            sky.SkyboxBk = data.Bk sky.SkyboxDn = data.Dn sky.SkyboxFt = data.Ft sky.SkyboxLf = data.Lf sky.SkyboxRt = data.Rt sky.SkyboxUp = data.Up
            sky.Parent = Lighting
        end
    end
end
local fovZoomConn, fovZoomOriginal = nil, nil
local function setFOVZoom(enabled)
    AR2_CFG.misc.fovZoom = enabled
    if fovZoomConn then fovZoomConn:Disconnect() fovZoomConn = nil end
    if fovZoomOriginal then Cam.FieldOfView = fovZoomOriginal fovZoomOriginal = nil end
    if not enabled then return end
    fovZoomConn = RunSvc.RenderStepped:Connect(function()
        local holding = UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
        if holding then fovZoomOriginal = fovZoomOriginal or Cam.FieldOfView Cam.FieldOfView = AR2_CFG.misc.zoomFOV
        elseif fovZoomOriginal then Cam.FieldOfView = fovZoomOriginal fovZoomOriginal = nil end
    end)
end
local distZoomConn = nil
local function currentTargetDist() if AR2_CFG.misc.distZoomActive then return AR2_CFG.misc.zoomedDist end return AR2_CFG.misc.normalDist end
local function setDistZoomEnabled(enabled)
    AR2_CFG.misc.distZoomEnabled = enabled
    if distZoomConn then distZoomConn:Disconnect() distZoomConn = nil end
    if not enabled then pcall(function() local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid") workspace.CurrentCamera.CameraSubject = hum workspace.CurrentCamera.CameraType = Enum.CameraType.Custom end) return end
    distZoomConn = RunSvc.RenderStepped:Connect(function()
        local char = LP.Character local root = char and char:FindFirstChild("HumanoidRootPart") local cam = workspace.CurrentCamera
        if not (root and cam) then return end
        local dist = currentTargetDist() local look = cam.CFrame.LookVector
        cam.CFrame = CFrame.new(root.Position + Vector3.new(0,2,0) - look * dist, root.Position + Vector3.new(0,2,0))
    end)
end
local function setDistZoomActive(active) AR2_CFG.misc.distZoomActive = active if active and not AR2_CFG.misc.distZoomEnabled then setDistZoomEnabled(true) end end

-- =============================================
-- APPLY CONFIG
-- =============================================
function AR2_applyConfig()
    local function try(fn, ...) if type(fn) ~= "function" then return end local ok, err = pcall(fn, ...) if not ok then warn("[Origin HUB] apply: " .. tostring(err)) end end
    try(function() if AR2_CFG.misc.noclip then AR2_startNoclip() else AR2_stopNoclip() end end)
    try(function() carNoclipSetEnabled(AR2_CFG.cars.noclip) end)
    try(function() AR2_setFly(AR2_CFG.cars.fly) end)
    try(function() AR2_setCarClip(AR2_CFG.cars.clipOff) end)
    try(function() AR2_setWalkSpeed(AR2_CFG.move.walkOn) end)
    try(function() applySkybox(AR2_CFG.misc.skybox or "Default") end)
    try(applyClouds) try(applyCustomAmbient)
    -- apply fog/fullbright CONDITIONALLY and AFTER skybox/clouds so they respect the config flag
    -- and aren't overwritten by an atmosphere/skybox change (that's why No Fog wasn't applying).
    try(function() if AR2_CFG.misc.fullbright then applyFullbright() else restoreFullbright() end end)
    try(function() if AR2_CFG.misc.noFog then applyNoFog() else restoreFog() end end)
    try(function() toggleTextures(AR2_CFG.misc.removeTextures) end)
    try(function() toggleFoliage(AR2_CFG.misc.removeFoliage) end)
    try(function() toggleDoors(AR2_CFG.misc.removeDoors) end)
    try(function() toggleSunRays(AR2_CFG.misc.removeSunRays) end)
    try(function() toggleWeather(AR2_CFG.misc.removeWeather) end)
    try(function() setForceStream(AR2_CFG.misc.forceStream) end)
    try(applyPersonalChar) try(applyPersonalGun)
    try(function() setTungSelf(AR2_CFG.misc.tungSelf) end)
    try(function() setTungGlobal(AR2_CFG.players.tungGlobal) end)
    try(function() setFOVZoom(AR2_CFG.misc.fovZoom) end)
    try(function() setDistZoomEnabled(AR2_CFG.misc.distZoomEnabled) end)
    try(function() setDistZoomActive(AR2_CFG.misc.distZoomActive) end)
    try(function() AR2_setSpin(AR2_spin.on) end)
    try(refreshCorpseCache) try(refreshLootList) try(refreshDropList)
end

-- =============================================
-- UI
-- =============================================
AR2_UI_ELEMENTS = {}
AR2_uiGuisToDestroy = AR2_uiGuisToDestroy or {}
AR2_uiWindowRef = nil
AR2_rebuildQueued = false
AR2_rebuildCount = 0
AR2_rebuildWindow = os.clock()
-- these UI helpers are declared as GLOBALS (no `local`) on purpose: the main chunk is at
-- the Luau 200-local ceiling, and globals consume no local registers. Only used by AR2_buildUI.
function AR2_queueRebuild(delaySecs) if AR2_rebuildQueued then return end AR2_rebuildQueued = true task.delay(delaySecs or 0.4, function() AR2_rebuildQueued = false AR2_buildUI() end) end

function addToggle(section, label, cfgTable, cfgKey, cb)
    return section:AddLabel(label):AddToggle({ Default = cfgTable[cfgKey] or false, Flag = "oh_"..label:gsub("[^%w]","_"), Callback = function(v) cfgTable[cfgKey]=v if cb then cb(v) end end })
end
function addSlider(section, label, cfgTable, cfgKey, min, max, step, decimals, cb)
    local def = cfgTable[cfgKey] if type(def) ~= "number" then def = min end def = math.clamp(def, min, max)
    return section:AddLabel(label):AddSlider({ Min=min, Max=max, Default=def, Rounding = decimals or ((step and step < 1) and 2 or 0), Flag = "oh_"..label:gsub("[^%w]","_"), Callback = function(v) cfgTable[cfgKey]=v if cb then cb(v) end end })
end
function addDropdown(section, label, options, cfgTable, cfgKey, cb)
    return section:AddLabel(label):AddDropdown({ Default = cfgTable[cfgKey], Values = options, Flag = "oh_"..label:gsub("[^%w]","_"), Callback = function(v) cfgTable[cfgKey]=v if cb then cb(v) end end })
end
function addColorPicker(section, label, cfgTable, cfgKey, cb)
    local ok, elem = pcall(function() return section:AddLabel(label):AddColorPicker({ Default = cfgTable[cfgKey], Flag = "oh_"..label:gsub("[^%w]","_"), Callback = function(v) cfgTable[cfgKey]=v if cb then cb(v) end end }) end)
    return ok and elem or nil
end
function addButton(section, label, cb) local ok, elem = pcall(function() return section:AddButton({ Name = label, Callback = cb }) end) return ok and elem or nil end

function promptText(title, current, callback)
    local host = getMainHost()
    local gui = Instance.new("ScreenGui") gui.Name = "oh_prompt" gui.ResetOnSpawn = false gui.Parent = host
    local frame = Instance.new("Frame") frame.Size = UDim2.fromOffset(300,100) frame.Position = UDim2.fromScale(0.5,0.5) frame.AnchorPoint = Vector2.new(0.5,0.5) frame.BackgroundColor3 = Color3.fromRGB(20,20,20) frame.BorderSizePixel = 0 frame.Parent = gui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0,8)
    local stroke = Instance.new("UIStroke", frame) stroke.Color = AR2_CFG.ui.accent stroke.Thickness = 1.5
    local lbl = Instance.new("TextLabel") lbl.Size = UDim2.new(1,-20,0,20) lbl.Position = UDim2.fromOffset(10,5) lbl.BackgroundTransparency = 1 lbl.Text = title lbl.TextColor3 = AR2_CFG.ui.accent lbl.Font = Enum.Font.GothamBold lbl.TextSize = 14 lbl.TextXAlignment = Enum.TextXAlignment.Center lbl.Parent = frame
    local input = Instance.new("TextBox") input.Size = UDim2.new(1,-40,0,30) input.Position = UDim2.fromOffset(20,30) input.BackgroundColor3 = Color3.fromRGB(35,35,35) input.TextColor3 = Color3.fromRGB(255,255,255) input.PlaceholderText = current or "Name" input.Text = current or "" input.Font = Enum.Font.SourceSans input.TextSize = 14 input.TextXAlignment = Enum.TextXAlignment.Center input.ClearTextOnFocus = false input.Parent = frame
    Instance.new("UICorner", input).CornerRadius = UDim.new(0,5)
    local okBtn = Instance.new("TextButton") okBtn.Size = UDim2.fromOffset(100,25) okBtn.Position = UDim2.fromOffset(55,65) okBtn.BackgroundColor3 = AR2_CFG.ui.accent okBtn.TextColor3 = Color3.fromRGB(0,0,0) okBtn.Text = "OK" okBtn.Font = Enum.Font.GothamBold okBtn.TextSize = 12 okBtn.Parent = frame
    Instance.new("UICorner", okBtn).CornerRadius = UDim.new(0,5)
    local cancel = Instance.new("TextButton") cancel.Size = UDim2.fromOffset(100,25) cancel.Position = UDim2.fromOffset(165,65) cancel.BackgroundColor3 = Color3.fromRGB(60,60,60) cancel.TextColor3 = Color3.fromRGB(255,255,255) cancel.Text = "Cancel" cancel.Font = Enum.Font.GothamBold cancel.TextSize = 12 cancel.Parent = frame
    Instance.new("UICorner", cancel).CornerRadius = UDim.new(0,5)
    local function submit() local v = input.Text gui:Destroy() if callback then callback(v) end end
    okBtn.MouseButton1Click:Connect(submit) cancel.MouseButton1Click:Connect(function() gui:Destroy() end)
    input.FocusLost:Connect(function(enter) if enter then submit() end end)
    input:CaptureFocus()
end

function AR2_buildUI()
    local now = os.clock()
    if now - AR2_rebuildWindow > 10 then AR2_rebuildWindow = now AR2_rebuildCount = 0 end
    AR2_rebuildCount = AR2_rebuildCount + 1
    if AR2_rebuildCount > 6 then warn("[Origin HUB] rebuild loop") return end
    for _, g in ipairs(AR2_uiGuisToDestroy) do pcall(function() if g and g.Parent then g:Destroy() end end) end
    AR2_uiGuisToDestroy = {}
    if AR2_uiWindowRef then
        pcall(function() if AR2_uiWindowRef.Destroy then AR2_uiWindowRef:Destroy() end end)
        pcall(function() if AR2_uiWindowRef.Unload then AR2_uiWindowRef:Unload() end end)
        AR2_uiWindowRef = nil
    end
    AR2_UI_ELEMENTS = {}
    local snapshot = snapshotAllHosts()
    local okNL, NL = pcall(function() return loadstring(game:HttpGet("https://raw.githubusercontent.com/DeviceHB21/mylib/refs/heads/main/source.lua"))() end)
    if not okNL or not NL then warn("[Origin HUB] NL library failed: "..tostring(NL)) return end
    if getgenv then getgenv().OriginHUB_NL = NL end
    local ORANGE = AR2_CFG.ui.accent
    local size = AR2_uiSizeToUDim2()
    pcall(function() NL.Theme = "Dark" end)
    pcall(function() if NL.SetAccent then NL:SetAccent(ORANGE) end end)
    local win
    local uiBuildOk, uiBuildErr = pcall(function()
        local Notif = NL:CreateNotification()
        win = NL:CreateWindow({ Name = "Origin HUB", Content = "TWD Online v56", Size = size, ConfigFolder = "OriginHUB_Configs", Enable3DRenderer = false, Keybind = AR2_SETTINGS.menuKey or "P", Theme = "Dark", Accent = ORANGE })
        if getgenv then getgenv().OriginHUB_Window = win end
        pcall(function() if win.SetAccent then win:SetAccent(ORANGE) end end)

        local espTab = win:AddTab({Icon="eye", Name="ESP"})
        local ps = espTab:AddSection({Name="Player ESP"})
        addToggle(ps,"Enabled",AR2_CFG.players,"on") addToggle(ps,"Box",AR2_CFG.players,"box") addToggle(ps,"Name",AR2_CFG.players,"name")
        addToggle(ps,"Distance",AR2_CFG.players,"dist") addToggle(ps,"Health Bar",AR2_CFG.players,"health")
        addToggle(ps,"Weapon Equipped",AR2_CFG.players,"weapon")
        addToggle(ps,"Tung Others",AR2_CFG.players,"tungGlobal",function(v) setTungGlobal(v) end)
        addSlider(ps,"Max Distance",AR2_CFG.players,"maxDist",100,5000,100)
        local zs = espTab:AddSection({Name="Zombie ESP", Position="right"})
        addToggle(zs,"Enabled",AR2_CFG.zombies,"on") addToggle(zs,"Box",AR2_CFG.zombies,"box") addToggle(zs,"Name",AR2_CFG.zombies,"name")
        addToggle(zs,"Distance",AR2_CFG.zombies,"dist") addToggle(zs,"Health Bar",AR2_CFG.zombies,"health") addSlider(zs,"Max Distance",AR2_CFG.zombies,"maxDist",100,5000,100)
        local vs = espTab:AddSection({Name="Vehicle ESP"})
        addToggle(vs,"Enabled",AR2_CFG.vehicles,"on") addToggle(vs,"Box",AR2_CFG.vehicles,"box") addToggle(vs,"Name",AR2_CFG.vehicles,"name")
        addToggle(vs,"Distance",AR2_CFG.vehicles,"dist") addToggle(vs,"Moving State",AR2_CFG.vehicles,"moving") addSlider(vs,"Max Distance",AR2_CFG.vehicles,"maxDist",100,5000,100)
        local hsE = espTab:AddSection({Name="Horse ESP"})
        addToggle(hsE,"Enabled",AR2_CFG.horses,"on") addToggle(hsE,"Box",AR2_CFG.horses,"box") addToggle(hsE,"Name",AR2_CFG.horses,"name")
        addToggle(hsE,"Distance",AR2_CFG.horses,"dist") addSlider(hsE,"Max Distance",AR2_CFG.horses,"maxDist",100,5000,100)
        local cs = espTab:AddSection({Name="Corpse ESP", Position="right"})
        addToggle(cs,"Enabled",AR2_CFG.corpses,"on") addToggle(cs,"Box",AR2_CFG.corpses,"box") addToggle(cs,"Name",AR2_CFG.corpses,"name")
        addToggle(cs,"Distance",AR2_CFG.corpses,"dist") addSlider(cs,"Max Distance",AR2_CFG.corpses,"maxDist",100,5000,100)
        local lsS = espTab:AddSection({Name="Loot ESP"})
        addToggle(lsS,"Enabled",AR2_CFG.loot,"on") addToggle(lsS,"Box",AR2_CFG.loot,"box") addToggle(lsS,"Name",AR2_CFG.loot,"name")
        addToggle(lsS,"Distance",AR2_CFG.loot,"dist") addToggle(lsS,"Highlight",AR2_CFG.loot,"highlight")
        addDropdown(lsS,"Tier",{"All","Guns","Ammo","Meds","Food","Other"},AR2_CFG.loot,"tierFilter",function() AR2_LOOT_CACHE.at=0 end)
        addSlider(lsS,"Max Distance",AR2_CFG.loot,"maxDist",50,3000,50)
        addSlider(lsS,"Max Draw",AR2_CFG.loot,"maxDraw",5,100,5,0) addSlider(lsS,"Max Highlight",AR2_CFG.loot,"maxHighlight",5,50,5,0)
        lsS:AddButton({ Name = "Keyword: " .. (AR2_CFG.loot.keyword ~= "" and AR2_CFG.loot.keyword or "(all)"), Callback = function() promptText("Loot keyword:", AR2_CFG.loot.keyword or "", function(v) AR2_CFG.loot.keyword = v or "" AR2_LOOT_CACHE.at = 0 AR2_queueRebuild(0.15) end) end })
        local ds = espTab:AddSection({Name="Item Drops", Position="right"})
        addToggle(ds,"Enabled",AR2_CFG.drops,"on") addToggle(ds,"Box",AR2_CFG.drops,"box") addToggle(ds,"Name",AR2_CFG.drops,"name")
        addToggle(ds,"Distance",AR2_CFG.drops,"dist") addToggle(ds,"Highlight",AR2_CFG.drops,"highlight") addSlider(ds,"Max Distance",AR2_CFG.drops,"maxDist",100,3000,100)
        local stS = espTab:AddSection({Name="ESP Style", Position="right"})
        addDropdown(stS,"Box Style",{"Corners","Full"},AR2_CFG.esp,"style")
        addToggle(stS,"Fill Tint",AR2_CFG.esp,"fillTint")
        addSlider(stS,"Corner Length",AR2_CFG.esp,"cornerLen",0.1,0.5,0.05,2)
        addSlider(stS,"Thickness",AR2_CFG.esp,"thickness",1,4,1,0)
        addSlider(stS,"Text Size",AR2_CFG.esp,"textSize",10,20,1,0)
        addToggle(stS,"3D Box (hitbox)",AR2_CFG.esp,"box3D")
        addToggle(stS,"3D Box on Zombies",AR2_CFG.esp,"box3DZombies")
        addColorPicker(stS,"3D Box Color",AR2_CFG.esp,"box3DColor")

        local tpTab = win:AddTab({Icon="send", Name="Teleport"})
        local tpQuick = tpTab:AddSection({Name="Quick Actions"})
        tpQuick:AddButton({Name="Nearest Player", Callback=function() AR2_teleportToNearestPlayer() end})
        tpQuick:AddButton({Name="Nearest Zombie", Callback=function() AR2_teleportToNearestZombie() end})
        tpQuick:AddButton({Name="Nearest Corpse", Callback=function() AR2_teleportToNearestCorpse() end})
        tpQuick:AddButton({Name="Nearest Horse", Callback=function() AR2_teleportToNearestHorse() end})
        tpQuick:AddButton({Name="Coordinates", Callback=function() promptText("Enter x,y,z:", "", function(v) if not v or v=="" then return end local x,y,z = v:match("^%s*(-?%d+%.?%d*)%s*,%s*(-?%d+%.?%d*)%s*,%s*(-?%d+%.?%d*)%s*$") if x and y and z then AR2_teleportToPosition(Vector3.new(tonumber(x),tonumber(y),tonumber(z))) end end) end})
        tpQuick:AddButton({Name="Stop", Callback=function() AR2_teleportStop() end})
        addSlider(tpQuick,"Vertical Offset",AR2_CFG.tp,"tpOffsetY",0,20,1,0)
        local mapS = tpTab:AddSection({Name="Interactive Map", Position="right"})
        addToggle(mapS,"Open Map",AR2_CFG.tp,"mapFake",function(v) AR2_toggleMapTP(v) end)
        addSlider(mapS,"Rotation (straighten)",AR2_CFG.tp,"mapRot",-30,30,0.5,1)
        mapS:AddButton({Name="Open Now", Callback=function() AR2_toggleMapTP(true) end})
        mapS:AddButton({Name="Close", Callback=function() AR2_toggleMapTP(false) end})
        local deathS = tpTab:AddSection({Name="Death Location"})
        addToggle(deathS,"Save on Death",AR2_CFG.tp,"deathSave") addToggle(deathS,"Auto TP on Respawn",AR2_CFG.tp,"respawnTP")
        deathS:AddButton({Name="TP to Death Location", Callback=function() AR2_teleportToDeath() end})
        deathS:AddButton({Name="Clear Saved", Callback=function() AR2_clearDeathLocation() end})
        local tpTune = tpTab:AddSection({Name="TP Method", Position="right"})
        addSlider(tpTune,"Fall Time (s)",AR2_CFG.tp,"fallTime",1,6,0.5,1) addSlider(tpTune,"Hold Time (s)",AR2_CFG.tp,"holdTime",0.2,3,0.1,1)
        addSlider(tpTune,"Retries",AR2_CFG.tp,"retries",1,6,1,0) addSlider(tpTune,"Settle Time (s)",AR2_CFG.tp,"settleTime",0.1,2,0.05,2)
        addToggle(tpTune,"Cursor TP (T key)",AR2_CFG.tp,"clickTP")
        tpTune:AddButton({Name="Cancel Teleport", Callback=function() AR2_teleportStop() end})

        local combatTab = win:AddTab({Icon="crosshair", Name="Combat"})
        local as = combatTab:AddSection({Name="Aimbot"})
        addToggle(as,"Enabled",AR2_CFG.aim,"on") addToggle(as,"360 Mode",AR2_CFG.aim,"mode360") addToggle(as,"Hold RMB",AR2_CFG.aim,"hold") addToggle(as,"Show FOV",AR2_CFG.aim,"showFov")
        addSlider(as,"Smoothness",AR2_CFG.aim,"smooth",1,10,1) addSlider(as,"FOV",AR2_CFG.aim,"fov",50,600,25) addSlider(as,"Max Distance",AR2_CFG.aim,"maxDist",100,3000,100)
        local silS = combatTab:AddSection({Name="Silent Aim", Position="right"})
        addToggle(silS,"Enabled",SA,"on") addToggle(silS,"360 Mode",SA,"mode360") addToggle(silS,"Show FOV",AR2_CFG.silent,"showFov") addSlider(silS,"FOV (degrees)",SA,"fovDeg",1,90,1,0)
        local silStatus = silS:AddLabel("idle")
        task.spawn(function() while true do pcall(function() local shots,hits,tname,note = SA_status() silStatus:SetText(("%d shots  %d hits  fov %d  %s [%s]"):format(shots,hits,SA.fovDeg,tname,note)) end) task.wait(0.4) end end)
        local wbS = combatTab:AddSection({Name="Wallbang"})
        addToggle(wbS,"Enabled (TRIM)",SA,"wTrim") addToggle(wbS,"Build empty step",SA,"wBuild")
        local hmS = combatTab:AddSection({Name="Hitmarker", Position="right"})
        addToggle(hmS,"Enabled",AR2_CFG.hitmark,"on") addSlider(hmS,"Volume",AR2_CFG.hitmark,"volume",0,1,0.05,2)
        addDropdown(hmS,"Sound",{"Ping","Tick","Click","Thud","Ding","Pop"},AR2_CFG.hitmark,"sound")
        hmS:AddButton({Name="Custom Sound ID: " .. (AR2_CFG.hitmark.customId ~= "" and AR2_CFG.hitmark.customId or "(off)"), Callback=function() promptText("Sound asset ID (blank = use dropdown):", AR2_CFG.hitmark.customId or "", function(v) AR2_CFG.hitmark.customId = tostring(v or ""):gsub("^%s+",""):gsub("%s+$","") AR2_queueRebuild(0.1) end) end})
        addToggle(hmS,"Fake Killfeed",AR2_CFG.killfeed,"on")
        local trS = combatTab:AddSection({Name="Bullet Tracers"})
        addToggle(trS,"Enabled",AR2_tracer,"on") addDropdown(trS,"Style",{"2D Line","3D Part"},AR2_tracer,"style") addSlider(trS,"Duration (s)",AR2_tracer,"life",0.1,2,0.05,2) addSlider(trS,"Thickness",AR2_tracer,"thickness",1,6,1,0) addColorPicker(trS,"Color",AR2_tracer,"color")
        local tsSec = combatTab:AddSection({Name="Trigger Bot", Position="right"})
        addToggle(tsSec,"Enabled",AR2_CFG.trigger,"on") addToggle(tsSec,"360 Mode",AR2_CFG.trigger,"mode360") addToggle(tsSec,"Force ADS",AR2_CFG.trigger,"forceAds")
        addToggle(tsSec,"Hold RMB",AR2_CFG.trigger,"holdRMB") addToggle(tsSec,"Visible Only",AR2_CFG.trigger,"visibleOnly")
        addSlider(tsSec,"Burst shots",AR2_CFG.trigger,"burst",1,6,1,0) addSlider(tsSec,"FOV",AR2_CFG.trigger,"fov",20,500,10) addSlider(tsSec,"Max Distance",AR2_CFG.trigger,"maxDist",50,3000,50) addSlider(tsSec,"Fire Delay",AR2_CFG.trigger,"delay",0.01,1,0.01,2)
        local trigStatusLbl = tsSec:AddLabel("off") task.spawn(function() while true do pcall(function() trigStatusLbl:SetText(tostring(AR2_trigStatus)) end) task.wait(0.3) end end)
        local rfs = combatTab:AddSection({Name="Rapid Fire"})
        addToggle(rfs,"Enabled",AR2_CFG.misc,"rapidFire") addSlider(rfs,"Extra Shots",AR2_CFG.misc,"rapidShots",1,6,1,0) addSlider(rfs,"Delay Between",AR2_CFG.misc,"rapidDelay",0.01,0.20,0.01,2)
        local wmS = combatTab:AddSection({Name="Hitbox Expander"})
        addToggle(wmS,"Hitbox Expander",AR2_CFG.weapon,"hitbox")
        addSlider(wmS,"Hitbox Size",AR2_CFG.weapon,"hitboxSize",5,2000,5,0)

        local moveTab = win:AddTab({Icon="move", Name="Movement"})
        local wsS = moveTab:AddSection({Name="Walk Speed"})
        addToggle(wsS,"Enabled",AR2_CFG.move,"walkOn",function(v) AR2_setWalkSpeed(v) end)
        addSlider(wsS,"Speed",AR2_CFG.move,"walkSpeed",16,1000,5,0)
        local nsS = moveTab:AddSection({Name="Noclip", Position="right"})
        addToggle(nsS,"Enabled",AR2_CFG.misc,"noclip",function(v) if v then AR2_startNoclip() else AR2_stopNoclip() end end)
        local ijs = moveTab:AddSection({Name="Inf Jump"})
        addToggle(ijs,"Enabled",AR2_CFG.misc,"infJump")
        local spS = moveTab:AddSection({Name="Spin Bot", Position="right"})
        addToggle(spS,"Enabled",AR2_spin,"on",function(v) AR2_setSpin(v) end) addToggle(spS,"Tilt",AR2_spin,"tilt") addSlider(spS,"Speed",AR2_spin,"speed",1,60,1,0)

        local carsTab = win:AddTab({Icon="car", Name="Cars"})
        local noclipCS = carsTab:AddSection({Name="Vehicle Noclip"})
        addToggle(noclipCS,"Enabled",AR2_CFG.cars,"noclip",function(v) carNoclipSetEnabled(v) end)
        addSlider(noclipCS,"Speed",AR2_CFG.cars,"noclipSpeed",30,1000,10,0)
        addToggle(noclipCS,"Platform on Disable",AR2_CFG.cars,"platformOnDisable")
        addToggle(noclipCS,"Eject on Disable",AR2_CFG.cars,"noclipEjectOnDisable")
        addToggle(noclipCS,"Collisions Off (drive thru walls)",AR2_CFG.cars,"clipOff",function(v) AR2_setCarClip(v) end)
        local flyS = carsTab:AddSection({Name="Vehicle Fly", Position="right"})
        addToggle(flyS,"Enabled",AR2_CFG.cars,"fly",function(v) AR2_setFly(v) end)
        addSlider(flyS,"Speed",AR2_CFG.cars,"flySpeed",20,1000,25,0)
        flyS:AddButton({Name="Park in Air (on roof)", Callback=function() AR2_parkVehicle() end})
        flyS:AddButton({Name="Repair / Reset Parts", Callback=function() AR2_repairVehicle() end})
        local hfS = carsTab:AddSection({Name="Vehicle Fly 2"})
        addToggle(hfS,"Enabled",AR2_CFG.cars,"horseFly",function(v) AR2_setHorseFly(v) end)
        addSlider(hfS,"Speed",AR2_CFG.cars,"horseFlySpeed",20,1000,25,0)
        local pfS = carsTab:AddSection({Name="Horse Fly (rides mount)"})
        addToggle(pfS,"Enabled",AR2_CFG.misc,"playerFly",function(v) AR2_setPlayerFly(v) end)
        addSlider(pfS,"Speed",AR2_CFG.misc,"playerFlySpeed",20,1000,25,0)
        local spdCS = carsTab:AddSection({Name="Vehicle Speed"})
        addToggle(spdCS,"Speed Multiplier",AR2_CFG.cars,"speedMult")
        addSlider(spdCS,"Multiplier",AR2_CFG.cars,"speedMultVal",1,5,0.5,1)
        local enterCS = carsTab:AddSection({Name="Auto Enter", Position="right"})
        enterCS:AddButton({Name="Enter Nearest Vehicle", Callback=function() AR2_enterNearestVehicle() end})
        addSlider(enterCS,"Range",AR2_CFG.cars,"enterRange",50,3000,50,0)
        local aeS = carsTab:AddSection({Name="Always Equip"})
        addToggle(aeS,"Shoot while in vehicle",AR2_CFG.misc,"alwaysEquip")
        local horseCS = carsTab:AddSection({Name="Horse", Position="right"})
        addToggle(horseCS,"Speed",AR2_CFG.cars,"horseSpeed")
        addSlider(horseCS,"Speed Value",AR2_CFG.cars,"horseSpeedVal",16,2000,10,0)
        addToggle(horseCS,"Jump",AR2_CFG.cars,"horseJump")
        addSlider(horseCS,"Jump Value",AR2_CFG.cars,"horseJumpVal",50,500,10,0)

        local worldTab = win:AddTab({Icon="globe", Name="World"})
        local lightS = worldTab:AddSection({Name="Lighting"})
        addToggle(lightS,"Fullbright",AR2_CFG.misc,"fullbright",function(v) if v then pcall(applyFullbright) else pcall(restoreFullbright) end end)
        addToggle(lightS,"No Fog",AR2_CFG.misc,"noFog",function(v) if v then applyNoFog() else restoreFog() end end)
        addToggle(lightS,"Set Time",AR2_CFG.misc,"setTime") addSlider(lightS,"Brightness",AR2_CFG.misc,"brightness",0,10,1,0) addSlider(lightS,"Time",AR2_CFG.misc,"timeValue",0,24,1,0)
        addDropdown(lightS,"Skybox",{"Default","Space","Galaxy","Galaxy 2","Galaxy 3","Saturne","Neptune","Redshift","Pink Daylights","Purple Night","Gray Night","Anime Sky"},AR2_CFG.misc,"skybox",function(v) applySkybox(v) end)
        addToggle(lightS,"Remove Shadows",AR2_CFG.misc,"removeShadows")
        local perfS = worldTab:AddSection({Name="Performance", Position="right"})
        addToggle(perfS,"Remove Textures",AR2_CFG.misc,"removeTextures",function(v) toggleTextures(v) end)
        addToggle(perfS,"Remove Foliage",AR2_CFG.misc,"removeFoliage",function(v) toggleFoliage(v) end)
        addToggle(perfS,"Remove Doors",AR2_CFG.misc,"removeDoors",function(v) toggleDoors(v) end)
        addToggle(perfS,"Remove Sun Rays",AR2_CFG.misc,"removeSunRays",function(v) toggleSunRays(v) end)
        addToggle(perfS,"Remove Weather (rain/snow)",AR2_CFG.misc,"removeWeather",function(v) toggleWeather(v) end)
        addToggle(perfS,"Force Stream (load distant)",AR2_CFG.misc,"forceStream",function(v) setForceStream(v) end)
        local cloudS = worldTab:AddSection({Name="Clouds"})
        addToggle(cloudS,"Remove Clouds",AR2_CFG.misc,"removeClouds",function() applyClouds() end)
        addToggle(cloudS,"Edit Clouds",AR2_CFG.misc,"modifyClouds",function() applyClouds() end)
        addColorPicker(cloudS,"Cloud Color",AR2_CFG.misc,"cloudsColor",function() applyClouds() end)
        addSlider(cloudS,"Cover",AR2_CFG.misc,"cloudsCover",0,1,0.05,2,function() applyClouds() end)
        addSlider(cloudS,"Density",AR2_CFG.misc,"cloudsDensity",0,1,0.05,2,function() applyClouds() end)

        local miscTab = win:AddTab({Icon="box", Name="Misc"})
        local fsS = miscTab:AddSection({Name="Freeze Zombies"})
        addToggle(fsS,"Enabled",AR2_CFG.misc,"freeze") addToggle(fsS,"Whole Server",AR2_CFG.misc,"freezeAll") addSlider(fsS,"Radius",AR2_CFG.misc,"freezeRadius",50,2000,50)
        local morphS = miscTab:AddSection({Name="Model Morph (troll)"})
        addDropdown(morphS,"Preset",{"Custom","Verity","Tung Tung Tung"},AR2_CFG.misc,"morphPreset",function(v)
            if v == "Verity" then AR2_CFG.misc.morphId = "140105640267431"
            elseif v == "Tung Tung Tung" then AR2_CFG.misc.morphId = "138151705692565" end
            AR2_queueRebuild(0.1)
        end)
        morphS:AddButton({Name="Set Model ID: " .. (AR2_CFG.misc.morphId ~= "" and AR2_CFG.misc.morphId or "(none)"), Callback=function() promptText("Model asset ID:", AR2_CFG.misc.morphId or "", function(v) AR2_CFG.misc.morphId = tostring(v or ""):gsub("%D","") AR2_CFG.misc.morphPreset = "Custom" AR2_queueRebuild(0.1) end) end})
        addToggle(morphS,"Hide My Character",AR2_CFG.misc,"morphHideSelf")
        morphS:AddButton({Name="Apply Morph", Callback=function() AR2_applyMorph() end})
        morphS:AddButton({Name="Remove Morph", Callback=function() AR2_removeMorph() end})
        local charS = miscTab:AddSection({Name="Character Chams", Position="right"})
        addToggle(charS,"Enabled",AR2_CFG.misc,"personalCharOn",function(v) setPersonalChar(v) end)
        addSlider(charS,"Hue",AR2_CFG.misc,"personalCharHue",0,360,15,0)
        addDropdown(charS,"Material",{"ForceField","Neon"},AR2_CFG.misc,"personalCharMaterial",function() if AR2_CFG.misc.personalCharOn then applyPersonalChar() end end)
        addToggle(charS,"Rainbow",AR2_CFG.misc,"personalCharRainbow")
        local gunS = miscTab:AddSection({Name="Gun Chams"})
        addToggle(gunS,"Enabled",AR2_CFG.misc,"personalGunOn",function(v) setPersonalGun(v) end)
        addSlider(gunS,"Hue",AR2_CFG.misc,"personalGunHue",0,360,15,0) addToggle(gunS,"Rainbow",AR2_CFG.misc,"personalGunRainbow")
        local camS = miscTab:AddSection({Name="Camera", Position="right"})
        addToggle(camS,"Distance Zoom",AR2_CFG.misc,"distZoomEnabled",function(v) setDistZoomEnabled(v) end)
        addSlider(camS,"Normal",AR2_CFG.misc,"normalDist",3,30,1,0) addSlider(camS,"Zoomed",AR2_CFG.misc,"zoomedDist",1,10,1,0)
        addToggle(camS,"FOV Zoom (RMB)",AR2_CFG.misc,"fovZoom",function(v) setFOVZoom(v) end) addSlider(camS,"Zoom FOV",AR2_CFG.misc,"zoomFOV",10,100,1,0)

        local kbTab = win:AddTab({Icon="keyboard", Name="Keybinds"})
        local function mkKB(section, label, id)
            local btn
            local function cur() return label .. "  ->  " .. (AR2_CFG.keybinds[id] or "None") end
            pcall(function() btn = section:AddButton({ Name = cur(), Callback = function() promptText("Keybind: "..label, AR2_CFG.keybinds[id], function(newKey) AR2_CFG.keybinds[id] = AR2_normalizeKey(newKey) if btn then pcall(function() if btn.SetName then btn:SetName(cur()) end end) pcall(function() if btn.SetText then btn:SetText(cur()) end end) end end) end }) end)
            return btn
        end
        local kbE = kbTab:AddSection({Name="ESP"})
        mkKB(kbE,"Player ESP","playerESP") mkKB(kbE,"Zombie ESP","zombieESP") mkKB(kbE,"Vehicle ESP","vehicleESP")
        mkKB(kbE,"Corpse ESP","corpseESP") mkKB(kbE,"Loot ESP","lootESP") mkKB(kbE,"Drop ESP","dropESP") mkKB(kbE,"Horse ESP","horseESP")
        local kbC = kbTab:AddSection({Name="Combat", Position="right"})
        mkKB(kbC,"Aimbot","aimbot") mkKB(kbC,"Aimbot 360","aimbot360") mkKB(kbC,"Silent Aim","silentAim") mkKB(kbC,"Silent Aim 360","silentAim360") mkKB(kbC,"Wallbang","wallbang")
        mkKB(kbC,"Trigger Bot","triggerBot") mkKB(kbC,"Rapid Fire","rapidFire") mkKB(kbC,"Freeze Zombies","freezeZ")
        mkKB(kbC,"Force ADS","forceAds") mkKB(kbC,"Hitbox Expander","hitbox")
        local kbMv = kbTab:AddSection({Name="Movement"})
        mkKB(kbMv,"Walk Speed","walkSpeed") mkKB(kbMv,"Noclip","noclip") mkKB(kbMv,"Inf Jump","infJump") mkKB(kbMv,"Spin Bot","spinBot")
        local kbCar = kbTab:AddSection({Name="Cars", Position="right"})
        mkKB(kbCar,"Vehicle Noclip","carNoclip") mkKB(kbCar,"Collisions Off","carClip") mkKB(kbCar,"Vehicle Fly","carFly") mkKB(kbCar,"Enter Nearest","carEnter") mkKB(kbCar,"Park Vehicle","carPark") mkKB(kbCar,"Repair Vehicle","carRepair") mkKB(kbCar,"Vehicle Fly 2","horseFly") mkKB(kbCar,"Horse Fly","playerFly") mkKB(kbCar,"Always Equip","alwaysEquip")
        mkKB(kbCar,"Horse Speed","horseSpeed") mkKB(kbCar,"Horse Jump","horseJump")
        local kbT = kbTab:AddSection({Name="Teleport"})
        mkKB(kbT,"TP Nearest Player","tpNearestP") mkKB(kbT,"TP Nearest Zombie","tpNearestZ") mkKB(kbT,"TP Nearest Corpse","tpNearestC")
        mkKB(kbT,"TP Nearest Horse","horseEnter") mkKB(kbT,"Interactive Map","mapFake") mkKB(kbT,"TP Death Location","deathTP")

        local setTab = win:AddTab({Icon="settings", Name="Settings"})
        local cfgS = setTab:AddSection({Name="Config Manager"})
        AR2_refreshConfigList()
        local cfgStatus = cfgS:AddLabel("ready")
        local function cfgSay(t) pcall(function() cfgStatus:SetText(t) end) task.delay(4, function() pcall(function() cfgStatus:SetText("ready") end) end) end
        addButton(cfgS,"Save Config",function() promptText("Config name:","",function(name) if not name or name=="" then return end if AR2_saveConfig(name) then cfgSay("saved: "..tostring(name)) AR2_queueRebuild(0.2) else cfgSay("SAVE FAILED") end end) end)
        addButton(cfgS,"Load Config",function() promptText("Load config name:","",function(name) if not name or name=="" then return end if AR2_loadConfigByName(name) then cfgSay("loaded: "..tostring(name)) AR2_queueRebuild(0.2) else cfgSay("LOAD FAILED") end end) end)
        addButton(cfgS,"Delete Config",function() promptText("Delete config name:","",function(name) if not name or name=="" then return end if AR2_deleteConfig(name) then cfgSay("deleted: "..tostring(name)) AR2_queueRebuild(0.2) else cfgSay("DELETE FAILED") end end) end)
        local sizeS = setTab:AddSection({Name="UI Size", Position="right"})
        addDropdown(sizeS,"Size Preset",{"Tiny","Small","Default","Large","Huge","Colossal"},AR2_CFG.ui,"sizePreset",function() AR2_queueRebuild(0.15) end)
        local uiS = setTab:AddSection({Name="UI Personalization"})
        addSlider(uiS,"Accent Hue",AR2_CFG.ui,"accentHue",0,360,15,0,function(v) AR2_CFG.ui.accent = Color3.fromHSV(math.clamp(v,0,360)/360,1,1) pcall(function() if NL.SetAccent then NL:SetAccent(AR2_CFG.ui.accent) end end) pcall(function() if win.SetAccent then win:SetAccent(AR2_CFG.ui.accent) end end) end)
        addColorPicker(uiS,"Accent Color",AR2_CFG.ui,"accent",function(v) pcall(function() if NL.SetAccent then NL:SetAccent(v) end end) pcall(function() if win.SetAccent then win:SetAccent(v) end end) end)
        local rS = setTab:AddSection({Name="Reset", Position="right"})
        addButton(rS,"Reset All Settings",function()
            for k,v in pairs(AR2_CFG) do if type(v)=="table" then for k2,v2 in pairs(v) do if type(v2)=="boolean" then v[k2]=false end end end end
            AR2_teleportStop() AR2_stopNoclip() unfreezeAll() pcall(restoreFullbright) pcall(restoreFog)
            restorePersonalChar() restorePersonalGun() setTungSelf(false) setTungGlobal(false)
            carNoclipStop() AR2_setFly(false) AR2_setHorseFly(false) AR2_setPlayerFly(false) AR2_setCarClip(false) AR2_setWalkSpeed(false) clearLootHighlights() AR2_deathTP.saved = nil AR2_toggleMapTP(false)
            toggleWeather(false) toggleFoliage(false) setForceStream(false) pcall(AR2_removeMorph)
            AR2_queueRebuild(0.2)
        end)
        Notif.new({Title="Origin HUB", Content="Loaded -- press P", Duration=5})
    end)
    if not uiBuildOk and uiBuildErr then warn("[Origin HUB] UI build failed: "..tostring(uiBuildErr)) end
    captureNewGuisInto(snapshot, AR2_uiGuisToDestroy)
    if win then
        AR2_uiWindowRef = win
        pcall(function() if win.UI and win.UI.Destroy then table.insert(AR2_uiGuisToDestroy, win.UI) end end)
        pcall(function() if win.ScreenGui and win.ScreenGui.Destroy then table.insert(AR2_uiGuisToDestroy, win.ScreenGui) end end)
    end
end

task.spawn(AR2_buildUI)
task.delay(2, function() if AR2_CFG.misc.skybox and AR2_CFG.misc.skybox ~= "Default" then pcall(applySkybox, AR2_CFG.misc.skybox) end end)
