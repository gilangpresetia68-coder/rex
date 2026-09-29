--[[
    ┌──────────────────────────────────────────────┐
    │  DELTA EXECUTOR  •  STEAL AN EGG  [FIXED]    │
    │  Theme : Green & White                       │
    │  Tabs  : Home | Script | ESP | Misc          │
    └──────────────────────────────────────────────┘
--]]

--========== SERVICES ==========--
local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local UserInputService  = game:GetService("UserInputService")
local TweenService      = game:GetService("TweenService")
local HttpService       = game:GetService("HttpService")
local TeleportService   = game:GetService("TeleportService")
local VirtualUser       = game:GetService("VirtualUser")
local StarterGui        = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer

--========== CONFIG ==========--
local CONFIG = {
    Green       = Color3.fromRGB(0, 175, 100),
    GreenDark   = Color3.fromRGB(0, 135, 75),
    GreenLight  = Color3.fromRGB(80, 220, 150),
    White       = Color3.fromRGB(255, 255, 255),
    Background  = Color3.fromRGB(245, 248, 246),
    TextDark    = Color3.fromRGB(35, 45, 40),
    TextMuted   = Color3.fromRGB(120, 130, 125),
    Font        = Enum.Font.GothamMedium,
    FontBold    = Enum.Font.GothamBold,
    Discord     = "https://discord.gg/XFu7NvmFj",
    WhatsApp1   = "https://whatsapp.com/channel/0029Vb8btgUEawdxyjZ8hF1J",
    WhatsApp2   = "https://whatsapp.com/channel/0029VbD6xItEgGfHYgbJrM3h",
    LogoId      = "rbxassetid://7828196911", -- GANTI dengan asset ID logomu
    PlaceId     = game.PlaceId,
    JobId       = game.JobId
}

--========== STATE ==========--
local State = {
    AutoFarm = false, AntiHitGuard = false, AntiHitBattle = false,
    AntiAfk = false, AntiTrap = false,
    AutoStealSecret = false, AutoStealEternal = false, AutoStealDivine = false,
    ESPEgg = false, ESPPlayer = false, ESPHitbox = false, ESPSkeleton = false,
    ESPColor = Color3.fromRGB(0, 255, 120),
    WalkSpeed = 16, JumpPower = 50, WebhookURL = ""
}

--========== HELPERS ==========--
local function GetGuiParent()
    local ok, hui = pcall(function() return gethui and gethui() end)
    if ok and hui then return hui end
    local ok2, cg = pcall(function() return game:GetService("CoreGui") end)
    if ok2 and cg then return cg end
    return LocalPlayer:WaitForChild("PlayerGui")
end

local function SafeParent(inst, parent)
    local ok = pcall(function() inst.Parent = parent end)
    if not ok or not inst.Parent then
        pcall(function() inst.Parent = LocalPlayer:WaitForChild("PlayerGui") end)
    end
end

local function Create(class, props, parent)
    local inst = Instance.new(class)
    for k, v in pairs(props or {}) do
        pcall(function() inst[k] = v end)
    end
    if parent then SafeParent(inst, parent) end
    return inst
end

local function Corner(inst, r)
    return Create("UICorner", { CornerRadius = UDim.new(0, r or 8) }, inst)
end

local function Stroke(inst, col, thick, trans)
    return Create("UIStroke", {
        Color = col or CONFIG.Green, Thickness = thick or 1,
        Transparency = trans or 0, ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    }, inst)
end

local function Dragify(frame, handle)
    handle = handle or frame
    local dragging, dragInput, startPos, startInput
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging, startPos, startInput = true, frame.Position, input.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local d = input.Position - startInput
            frame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

--========== ROOT GUI ==========--
local GUI_PARENT = GetGuiParent()

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaStealEgg_" .. tostring(math.random(1000, 9999))
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
SafeParent(ScreenGui, GUI_PARENT)

--========== MAIN WINDOW ==========--
local Main = Create("Frame", {
    Name = "Main", Size = UDim2.new(0, 560, 0, 380),
    Position = UDim2.new(0.5, -280, 0.5, -190),
    BackgroundColor3 = CONFIG.White, BorderSizePixel = 0, Active = true
}, ScreenGui)
Corner(Main, 10); Stroke(Main, CONFIG.Green, 1.5, 0)

local TitleBar = Create("Frame", {
    Name = "TitleBar", Size = UDim2.new(1, 0, 0, 38),
    BackgroundColor3 = CONFIG.Green, BorderSizePixel = 0
}, Main)
Corner(TitleBar, 10)
Create("Frame", {
    Size = UDim2.new(1, 0, 0, 10), Position = UDim2.new(0, 0, 1, -10),
    BackgroundColor3 = CONFIG.Green, BorderSizePixel = 0
}, TitleBar)

Create("TextLabel", {
    Size = UDim2.new(1, -120, 1, 0), Position = UDim2.new(0, 12, 0, 0),
    BackgroundTransparency = 1, Text = "  DELTA  •  Steal An Egg",
    TextColor3 = CONFIG.White, Font = CONFIG.FontBold, TextSize = 15,
    TextXAlignment = Enum.TextXAlignment.Left
}, TitleBar)

Dragify(Main, TitleBar)

local function MakeTitleBtn(text, xOff, cb)
    local b = Create("TextButton", {
        Size = UDim2.new(0, 28, 0, 26), Position = UDim2.new(1, xOff, 0, 6),
        BackgroundColor3 = CONFIG.GreenDark, Text = text,
        TextColor3 = CONFIG.White, Font = CONFIG.FontBold, TextSize = 16,
        BorderSizePixel = 0, AutoButtonColor = true
    }, TitleBar)
    Corner(b, 6)
    b.MouseButton1Click:Connect(cb)
end

MakeTitleBtn("✕", -36, function() Main.Visible = false end)
MakeTitleBtn("–", -70, function() Main.Visible = false end)

--========== SIDEBAR ==========--
local Sidebar = Create("Frame", {
    Size = UDim2.new(0, 130, 1, -50), Position = UDim2.new(0, 8, 0, 44),
    BackgroundColor3 = CONFIG.Background, BorderSizePixel = 0
}, Main)
Corner(Sidebar, 8)

local TabHolder = Create("Frame", {
    Size = UDim2.new(1, -12, 1, -12), Position = UDim2.new(0, 6, 0, 6),
    BackgroundTransparency = 1
}, Sidebar)
Create("UIListLayout", {
    Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder
}, TabHolder)

local Content = Create("Frame", {
    Size = UDim2.new(1, -150, 1, -50), Position = UDim2.new(0, 142, 0, 44),
    BackgroundColor3 = CONFIG.White, BorderSizePixel = 0
}, Main)

local Pages = {}

local function CreateTab(name)
    local btn = Create("TextButton", {
        Size = UDim2.new(1, 0, 0, 34), BackgroundColor3 = CONFIG.White,
        Text = "  " .. name, TextColor3 = CONFIG.TextDark, Font = CONFIG.Font,
        TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left,
        BorderSizePixel = 0, AutoButtonColor = false
    }, TabHolder)
    Corner(btn, 6); Stroke(btn, Color3.fromRGB(220, 228, 224), 1, 0)

    local page = Create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1,
        BorderSizePixel = 0, ScrollBarThickness = 4,
        ScrollBarImageColor3 = CONFIG.Green, CanvasSize = UDim2.new(0, 0, 0, 0),
        Visible = false
    }, Content)
    Create("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder }, page)
    Create("UIPadding", {
        PaddingTop = UDim.new(0, 4), PaddingBottom = UDim.new(0, 4)
    }, page)

    Pages[name] = { Button = btn, Page = page }
    btn.MouseButton1Click:Connect(function()
        for _, d in pairs(Pages) do
            d.Page.Visible = false
            d.Button.BackgroundColor3 = CONFIG.White
            d.Button.TextColor3 = CONFIG.TextDark
        end
        page.Visible = true
        btn.BackgroundColor3 = CONFIG.Green
        btn.TextColor3 = CONFIG.White
    end)
    return page
end

--========== UI COMPONENTS ==========--
local function AddButton(parent, text, cb)
    local b = Create("TextButton", {
        Size = UDim2.new(1, -10, 0, 34), BackgroundColor3 = CONFIG.Green,
        Text = text, TextColor3 = CONFIG.White, Font = CONFIG.FontBold,
        TextSize = 13, BorderSizePixel = 0, AutoButtonColor = true
    }, parent)
    Corner(b, 6)
    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = CONFIG.GreenDark }):Play()
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = CONFIG.Green }):Play()
    end)
    b.MouseButton1Click:Connect(function() pcall(cb) end)
    return b
end

local function AddToggle(parent, text, default, cb)
    local h = Create("Frame", {
        Size = UDim2.new(1, -10, 0, 34), BackgroundColor3 = CONFIG.Background,
        BorderSizePixel = 0
    }, parent)
    Corner(h, 6); Stroke(h, Color3.fromRGB(220, 228, 224), 1, 0)

    Create("TextLabel", {
        Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 12, 0, 0),
        BackgroundTransparency = 1, Text = text, TextColor3 = CONFIG.TextDark,
        Font = CONFIG.Font, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left
    }, h)

    local sw = Create("TextButton", {
        Size = UDim2.new(0, 44, 0, 22), Position = UDim2.new(1, -52, 0.5, -11),
        BackgroundColor3 = default and CONFIG.Green or Color3.fromRGB(200, 205, 203),
        Text = "", BorderSizePixel = 0, AutoButtonColor = false
    }, h)
    Corner(sw, 11)

    local knob = Create("Frame", {
        Size = UDim2.new(0, 18, 0, 18),
        Position = default and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9),
        BackgroundColor3 = CONFIG.White, BorderSizePixel = 0
    }, sw)
    Corner(knob, 9)

    local on = default
    local function set(v)
        on = v
        TweenService:Create(sw, TweenInfo.new(0.2), {
            BackgroundColor3 = v and CONFIG.Green or Color3.fromRGB(200, 205, 203)
        }):Play()
        TweenService:Create(knob, TweenInfo.new(0.2), {
            Position = v and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)
        }):Play()
        pcall(cb, v)
    end
    sw.MouseButton1Click:Connect(function() set(not on) end)
    return { Set = set }
end

local function AddTextBox(parent, placeholder, default, cb)
    local box = Create("TextBox", {
        Size = UDim2.new(1, -10, 0, 34), BackgroundColor3 = CONFIG.Background,
        Text = default or "", PlaceholderText = placeholder, TextColor3 = CONFIG.TextDark,
        PlaceholderColor3 = CONFIG.TextMuted, Font = CONFIG.Font, TextSize = 13,
        BorderSizePixel = 0, ClearTextOnFocus = false
    }, parent)
    Corner(box, 6); Stroke(box, Color3.fromRGB(220, 228, 224), 1, 0)
    Create("UIPadding", { PaddingLeft = UDim.new(0, 10) }, box)
    if cb then
        box.FocusLost:Connect(function() pcall(cb, box.Text) end)
    end
    return box
end

local function AddLabel(parent, text)
    Create("TextLabel", {
        Size = UDim2.new(1, -10, 0, 26), BackgroundTransparency = 1,
        Text = text, TextColor3 = CONFIG.TextMuted, Font = CONFIG.FontBold,
        TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left
    }, parent)
end

local function AddDivider(parent)
    Create("Frame", {
        Size = UDim2.new(1, -10, 0, 1),
        BackgroundColor3 = Color3.fromRGB(225, 232, 228), BorderSizePixel = 0
    }, parent)
end

--========== TABS ==========--
local HomePage   = CreateTab("🏠  Home")
local ScriptPage = CreateTab("⚙  Script")
local ESPPage    = CreateTab("👁  ESP")
local MiscPage   = CreateTab("⚡  Misc")

--========== HOME ==========--
AddLabel(HomePage, "COMMUNITY")
AddButton(HomePage, "💬  Join Discord", function()
    if setclipboard then setclipboard(CONFIG.Discord) end
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Discord", Text = "Link dicopy ke clipboard!", Duration = 4
        })
    end)
end)
AddButton(HomePage, "📢  Join Saluran WhatsApp #1", function()
    if setclipboard then setclipboard(CONFIG.WhatsApp1) end
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "WhatsApp", Text = "Link #1 dicopy!", Duration = 4
        })
    end)
end)
AddButton(HomePage, "📢  Join Saluran WhatsApp #2", function()
    if setclipboard then setclipboard(CONFIG.WhatsApp2) end
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "WhatsApp", Text = "Link #2 dicopy!", Duration = 4
        })
    end)
end)

AddDivider(HomePage)
AddLabel(HomePage, "SERVER")

AddButton(HomePage, "🔄  Server Hop", function()
    task.spawn(function()
        local ok, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(
                "https://games.roblox.com/v1/games/" .. CONFIG.PlaceId ..
                "/servers/Public?sortOrder=Asc&limit=100"))
        end)
        if ok and result and result.data then
            for _, srv in ipairs(result.data) do
                if srv.id ~= CONFIG.JobId and srv.playing < srv.maxPlayers then
                    TeleportService:TeleportToPlaceInstance(CONFIG.PlaceId, srv.id, LocalPlayer)
                    return
                end
            end
        end
    end)
end)

AddButton(HomePage, "🔒  Auto Join Private Server (Sepi)", function()
    task.spawn(function()
        local ok, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(
                "https://games.roblox.com/v1/games/" .. CONFIG.PlaceId ..
                "/servers/Public?sortOrder=Asc&limit=100"))
        end)
        if ok and result and result.data then
            local best, bestCount = nil, math.huge
            for _, srv in ipairs(result.data) do
                if srv.id ~= CONFIG.JobId and srv.playing < bestCount then
                    best, bestCount = srv.id, srv.playing
                end
            end
            if best then
                TeleportService:TeleportToPlaceInstance(CONFIG.PlaceId, best, LocalPlayer)
            end
        end
    end)
end)

--========== SCRIPT TAB ==========--
AddLabel(ScriptPage, "FARM")
AddToggle(ScriptPage, "🌾  Auto Farm", false, function(v) State.AutoFarm = v end)

AddDivider(ScriptPage)
AddLabel(ScriptPage, "PROTECTION")
AddToggle(ScriptPage, "🛡  Anti Hit Guard", false, function(v) State.AntiHitGuard = v end)
AddToggle(ScriptPage, "⚔  Anti Hit Battle", false, function(v) State.AntiHitBattle = v end)
AddToggle(ScriptPage, "💤  Anti AFK", false, function(v) State.AntiAfk = v end)
AddToggle(ScriptPage, "🕳  Anti Trap", false, function(v) State.AntiTrap = v end)

AddDivider(ScriptPage)
AddLabel(ScriptPage, "AUTO STEAL")
AddToggle(ScriptPage, "🔮  Auto Steal Secret", false, function(v) State.AutoStealSecret = v end)
AddToggle(ScriptPage, "✨  Auto Steal Eternal", false, function(v) State.AutoStealEternal = v end)
AddToggle(ScriptPage, "👑  Auto Steal Divine", false, function(v) State.AutoStealDivine = v end)

--========== ESP TAB ==========--
AddLabel(ESPPage, "ESP TOGGLE")
AddToggle(ESPPage, "🥚  ESP Egg", false, function(v) State.ESPEgg = v end)
AddToggle(ESPPage, "🧍  ESP Player", false, function(v) State.ESPPlayer = v end)
AddToggle(ESPPage, "📦  ESP Hitbox", false, function(v) State.ESPHitbox = v end)
AddToggle(ESPPage, "🦴  ESP Skeleton", false, function(v) State.ESPSkeleton = v end)

AddDivider(ESPPage)
AddLabel(ESPPage, "ESP COLOR (Hex)")
local colorBox = AddTextBox(ESPPage, "#00FF78", "#00FF78", function(txt)
    local hex = tostring(txt):gsub("#", "")
    local ok, color = pcall(function() return Color3.fromHex("#" .. hex) end)
    if ok then State.ESPColor = color end
end)
AddButton(ESPPage, "🎨  Apply Color", function()
    local hex = tostring(colorBox.Text):gsub("#", "")
    local ok, color = pcall(function() return Color3.fromHex("#" .. hex) end)
    if ok then State.ESPColor = color end
end)

--========== MISC TAB ==========--
AddLabel(MiscPage, "🏃 PLAYER SPEED  (1 – 30000)")
local speedBox = AddTextBox(MiscPage, "WalkSpeed (1-30000)", "16", nil)
AddButton(MiscPage, "✅  Apply Speed", function()
    local n = tonumber(speedBox.Text)
    if n and n >= 1 and n <= 30000 then
        State.WalkSpeed = n
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = n end
    else
        pcall(function()
            StarterGui:SetCore("SendNotification", {
                Title = "Speed", Text = "Nilai harus 1 - 30000!", Duration = 3
            })
        end)
    end
end)

AddLabel(MiscPage, "🦘 JUMP POWER  (1 – 5000)")
local jumpBox = AddTextBox(MiscPage, "JumpPower (1-5000)", "50", nil)
AddButton(MiscPage, "✅  Apply Jump", function()
    local n = tonumber(jumpBox.Text)
    if n and n >= 1 and n <= 5000 then
        State.JumpPower = n
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.UseJumpPower = true
            hum.JumpPower = n
        end
    else
        pcall(function()
            StarterGui:SetCore("SendNotification", {
                Title = "Jump", Text = "Nilai harus 1 - 5000!", Duration = 3
            })
        end)
    end
end)

AddDivider(MiscPage)
AddLabel(MiscPage, "🔗 WEBHOOK  (Discord)")
local webhookBox = AddTextBox(MiscPage, "Paste Webhook URL...", "", function(txt)
    State.WebhookURL = txt
end)

AddButton(MiscPage, "📤  Send Player Info ke Webhook", function()
    local url = (State.WebhookURL ~= "" and State.WebhookURL) or webhookBox.Text
    if not url or url == "" then
        pcall(function()
            StarterGui:SetCore("SendNotification", {
                Title = "Webhook", Text = "URL kosong!", Duration = 3
            })
        end)
        return
    end
    task.spawn(function()
        local payload = {
            username = "Delta Steal-An-Egg Logger",
            content = "**Player Info**",
            embeds = {{
                title = "🎮 Roblox Player",
                color = 175000, -- hijau
                fields = {
                    { name = "Username", value = LocalPlayer.Name, inline = true },
                    { name = "DisplayName", value = LocalPlayer.DisplayName, inline = true },
                    { name = "UserId", value = tostring(LocalPlayer.UserId), inline = true },
                    { name = "PlaceId", value = tostring(CONFIG.PlaceId), inline = true },
                    { name = "JobId", value = CONFIG.JobId, inline = false },
                    { name = "Executor", value = tostring(identifyexecutor and identifyexecutor() or "Unknown"), inline = true }
                },
                timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
            }}
        }
        local data = HttpService:JSONEncode(payload)
        local headers = { ["Content-Type"] = "application/json" }
        local sent = false

        -- Coba beberapa executor request functions
        if syn and syn.request then
            sent = pcall(function() syn.request({ Url = url, Method = "POST", Headers = headers, Body = data }) end)
        elseif http_request then
            sent = pcall(function() http_request({ Url = url, Method = "POST", Headers = headers, Body = data }) end)
        elseif request then
            sent = pcall(function() request({ Url = url, Method = "POST", Headers = headers, Body = data }) end)
        else
            sent = pcall(function() HttpService:PostAsync(url, data) end)
        end

        pcall(function()
            StarterGui:SetCore("SendNotification", {
                Title = "Webhook",
                Text = sent and "Berhasil dikirim ✓" or "Gagal kirim ❌",
                Duration = 4
            })
        end)
    end)
end)

-
