-- Updated GUI improved overall kill aura

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

-- Cleanup previous instances
if CoreGui:FindFirstChild("FavelaMenuKillGUI") then
    CoreGui.FavelaMenuKillGUI:Destroy()
end

if CoreGui:FindFirstChild("FavelaTargetHighlight") then
    CoreGui.FavelaTargetHighlight:Destroy()
end

-- Configuration State
local Config = {
    Damage = 920,
    KillAura = false,
    KillAuraRange = 100,
    LoopKillTarget = false,
    LoopDelay = 0.2,
    WhitelistFriends = true,
    TriggerKey = Enum.KeyCode.F,
    KillAllKey = Enum.KeyCode.K,
    ToggleMenuKey = Enum.KeyCode.RightControl
}

-- Target Visualizer Highlight
local TargetHighlight = Instance.new("Highlight")
TargetHighlight.Name = "FavelaTargetHighlight"
TargetHighlight.FillColor = Color3.fromRGB(255, 50, 50)
TargetHighlight.FillTransparency = 0.5
TargetHighlight.OutlineColor = Color3.fromRGB(255, 255, 255)
TargetHighlight.OutlineTransparency = 0
TargetHighlight.Enabled = false
TargetHighlight.Parent = CoreGui

-- Get a team object by name (partial match)
local function getTeamByName(partialName)
    if not partialName or partialName == "" then return nil end
    local lower = string.lower(partialName)
    for _, team in ipairs(Players:GetTeams()) do
        if string.find(string.lower(team.Name), lower) then
            return team
        end
    end
    return nil
end

-- Check if a target string refers to a team
local function isTeamName(text)
    if not text or text == "" then return false end
    local lower = string.lower(text)
    for _, team in ipairs(Players:GetTeams()) do
        if string.find(string.lower(team.Name), lower) then
            return true
        end
    end
    return false
end

-- Get all player characters on a team
local function getPlayersOnTeam(team)
    local chars = {}
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Team == team and pl.Character and pl ~= LocalPlayer then
            table.insert(chars, pl.Character)
        end
    end
    return chars
end

-- Target & Weapon Resolution
local function getNil(name, class)
    if getnilinstances then
        for _, v in next, getnilinstances() do
            if v.ClassName == class and string.find(string.lower(v.Name), string.lower(name)) then
                return v
            end
        end
    end
    return nil
end

local function isFriend(player)
    if not Config.WhitelistFriends then return false end
    local ok, res = pcall(function()
        return LocalPlayer:IsFriendsWith(player.UserId)
    end)
    return ok and res
end

local function findWeapon(partialName)
    local char = LocalPlayer.Character
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    local locales = {char, backpack}

    if not partialName or partialName == "" then
        if char and char:FindFirstChildOfClass("Tool") then
            return char:FindFirstChildOfClass("Tool")
        end
        if backpack and backpack:FindFirstChildOfClass("Tool") then
            return backpack:FindFirstChildOfClass("Tool")
        end
        return nil
    end

    local partialNameLower = string.lower(partialName)
    for _, locale in ipairs(locales) do
        if locale then
            for _, item in ipairs(locale:GetChildren()) do
                if item:IsA("Tool") and string.find(string.lower(item.Name), partialNameLower) then
                    return item
                end
            end
        end
    end
    return nil
end

local function findTarget(partialName)
    if not partialName or partialName == "" then return nil end
    local partialNameLower = string.lower(partialName)
    
    for _, player in ipairs(Players:GetPlayers()) do
        if string.find(string.lower(player.Name), partialNameLower) or string.find(string.lower(player.DisplayName), partialNameLower) then
            if player.Character then return player.Character end
        end
    end
    
    for _, v in ipairs(workspace:GetChildren()) do
        if v:IsA("Model") and v:FindFirstChild("Humanoid") and string.find(string.lower(v.Name), partialNameLower) then
            return v
        end
    end
    
    return getNil(partialName, "Model")
end

local function getNearestEnemy(maxDist)
    local myChar = LocalPlayer.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end

    local nearest, shortestDist = nil, maxDist or math.huge
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and not isFriend(pl) then
            local root = pl.Character:FindFirstChild("HumanoidRootPart")
            local hum = pl.Character:FindFirstChildOfClass("Humanoid")
            if root and hum and hum.Health > 0 then
                local dist = (myRoot.Position - root.Position).Magnitude
                if dist < shortestDist then
                    shortestDist = dist
                    nearest = pl.Character
                end
            end
        end
    end
    return nearest
end

-- Core Remote Execution
local function sendDamage(targetChar, weaponTool, damageAmount)
    if not targetChar or not weaponTool then return false end
    local coms = ReplicatedStorage:FindFirstChild("Coms")
    if not coms then return false end

    local args = {
        [1] = "damage",
        [2] = damageAmount or Config.Damage,
        [3] = weaponTool,
        [4] = targetChar
    }
    
    local ok = pcall(function()
        coms:FireServer(unpack(args))
    end)
    return ok
end
  
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeathNoteOnCrack"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 280, 0, 440)
MainFrame.Position = UDim2.new(0.5, -140, 0.5, -220)
MainFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(255, 75, 75)
MainStroke.Thickness = 1.5
MainStroke.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 38)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local TitleCover = Instance.new("Frame")
TitleCover.Size = UDim2.new(1, 0, 0, 10)
TitleCover.Position = UDim2.new(0, 0, 1, -10)
TitleCover.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
TitleCover.BorderSizePixel = 0
TitleCover.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -45, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Favela Kill Suite"
TitleLabel.TextColor3 = Color3.fromRGB(255, 85, 85)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -32, 0, 6)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 12
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    TargetHighlight:Destroy()
end)

-- Content Scroll Frame
local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -16, 1, -80)
Content.Position = UDim2.new(0, 8, 0, 44)
Content.BackgroundTransparency = 1
Content.BorderSizePixel = 0
Content.ScrollBarThickness = 3
Content.ScrollBarImageColor3 = Color3.fromRGB(255, 80, 80)
Content.Parent = MainFrame

local UIList = Instance.new("UIListLayout")
UIList.Padding = UDim.new(0, 6)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Parent = Content

UIList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Content.CanvasSize = UDim2.new(0, 0, 0, UIList.AbsoluteContentSize.Y + 10)
end)

-- Status Label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -16, 0, 24)
StatusLabel.Position = UDim2.new(0, 8, 1, -30)
StatusLabel.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
StatusLabel.Text = "Ready"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 11
StatusLabel.Parent = MainFrame

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 6)
StatusCorner.Parent = StatusLabel

local function setStatus(text, color)
    StatusLabel.Text = text
    StatusLabel.TextColor3 = color or Color3.fromRGB(180, 180, 190)
end

-- Helper Builders
local function createTextBox(placeholder, defaultText)
    local Box = Instance.new("TextBox")
    Box.Size = UDim2.new(1, -6, 0, 32)
    Box.BackgroundColor3 = Color3.fromRGB(32, 32, 40)
    Box.BorderSizePixel = 0
    Box.PlaceholderText = placeholder
    Box.PlaceholderColor3 = Color3.fromRGB(130, 130, 140)
    Box.Text = defaultText or ""
    Box.TextColor3 = Color3.fromRGB(255, 255, 255)
    Box.Font = Enum.Font.Gotham
    Box.TextSize = 12
    Box.ClearTextOnFocus = false
    Box.Parent = Content

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Box
    return Box
end

local function createButton(text, bgColor, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 32)
    Btn.BackgroundColor3 = bgColor or Color3.fromRGB(40, 40, 50)
    Btn.BorderSizePixel = 0
    Btn.Text = text
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 12
    Btn.Parent = Content

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Btn

    Btn.MouseButton1Click:Connect(callback)
    return Btn
end

local function createToggle(text, initial, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, -6, 0, 32)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    Frame.BorderSizePixel = 0
    Frame.Parent = Content

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -55, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(220, 220, 230)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 44, 0, 20)
    Btn.Position = UDim2.new(1, -50, 0.5, -10)
    Btn.BackgroundColor3 = initial and Color3.fromRGB(255, 75, 75) or Color3.fromRGB(55, 55, 65)
    Btn.Text = initial and "ON" or "OFF"
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 10
    Btn.Parent = Frame

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = Btn

    local state = initial
    Btn.MouseButton1Click:Connect(function()
        state = not state
        Btn.Text = state and "ON" or "OFF"
        TweenService:Create(Btn, TweenInfo.new(0.15), {
            BackgroundColor3 = state and Color3.fromRGB(255, 75, 75) or Color3.fromRGB(55, 55, 65)
        }):Play()
        callback(state)
    end)
    return Frame
end

local function createSlider(text, min, max, defaultVal, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, -6, 0, 48)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    Frame.BorderSizePixel = 0
    Frame.Parent = Content

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -20, 0, 20)
    Label.Position = UDim2.new(0, 10, 0, 4)
    Label.BackgroundTransparency = 1
    Label.Text = text .. ": " .. tostring(defaultVal)
    Label.TextColor3 = Color3.fromRGB(220, 220, 230)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local SliderBack = Instance.new("Frame")
    SliderBack.Size = UDim2.new(1, -20, 0, 5)
    SliderBack.Position = UDim2.new(0, 10, 0, 30)
    SliderBack.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
    SliderBack.BorderSizePixel = 0
    SliderBack.Parent = Frame

    local BackCorner = Instance.new("UICorner")
    BackCorner.CornerRadius = UDim.new(1, 0)
    BackCorner.Parent = SliderBack

    local SliderFill = Instance.new("Frame")
    local startPercent = (defaultVal - min) / (max - min)
    SliderFill.Size = UDim2.new(math.clamp(startPercent, 0, 1), 0, 1, 0)
    SliderFill.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderBack

    local FillCorner = Instance.new("UICorner")
    FillCorner.CornerRadius = UDim.new(1, 0)
    FillCorner.Parent = SliderFill

    local Trigger = Instance.new("TextButton")
    Trigger.Size = UDim2.new(1, 0, 1, 0)
    Trigger.BackgroundTransparency = 1
    Trigger.Text = ""
    Trigger.Parent = SliderBack

    local function update(input)
        local pos = input.Position.X - SliderBack.AbsolutePosition.X
        local pct = math.clamp(pos / SliderBack.AbsoluteSize.X, 0, 1)
        SliderFill.Size = UDim2.new(pct, 0, 1, 0)
        
        local val = min + (max - min) * pct
        if max <= 5 then
            val = math.round(val * 100) / 100
        else
            val = math.round(val)
        end
        
        Label.Text = text .. ": " .. tostring(val)
        callback(val)
    end

    local dragging = false
    Trigger.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            update(input)
        end
    end)

    Trigger.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            update(input)
        end
    end)
    return Frame
end

local function createKeybind(text, defaultKey, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, -6, 0, 32)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
    Frame.BorderSizePixel = 0
    Frame.Parent = Content

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 6)
    Corner.Parent = Frame

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -95, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(220, 220, 230)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = Frame

    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 80, 0, 20)
    Btn.Position = UDim2.new(1, -86, 0.5, -10)
    Btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    Btn.Text = defaultKey.Name
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = Enum.Font.GothamBold
    Btn.TextSize = 10
    Btn.Parent = Frame

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = Btn

    local listening = false
    Btn.MouseButton1Click:Connect(function()
        if listening then return end
        listening = true
        Btn.Text = "..."
        Btn.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
        
        local conn
        conn = UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                conn:Disconnect()
                listening = false
                Btn.Text = input.KeyCode.Name
                Btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
                callback(input.KeyCode)
            end
        end)
    end)
    return Frame
end

local WeaponBox = createTextBox("Weapon Name (blank = auto-detect)", "")
local TargetBox = createTextBox("Name, 'all', or team name", "")
local DamageBox = createTextBox("Damage Amount", tostring(Config.Damage))

DamageBox.FocusLost:Connect(function()
    local num = tonumber(DamageBox.Text)
    if num then
        Config.Damage = num
        setStatus("Damage set to: " .. num, Color3.fromRGB(100, 220, 255))
    else
        DamageBox.Text = tostring(Config.Damage)
    end
end)

local function triggerKill()
    local wName = WeaponBox.Text
    local tName = TargetBox.Text
    local dmg = tonumber(DamageBox.Text) or Config.Damage

    local weapon = findWeapon(wName)
    if not weapon then
        setStatus("Weapon not found! Equip one or type name.", Color3.fromRGB(255, 80, 80))
        return
    end

    -- Handle "all" — everyone except friends
    if string.lower(tName) == "all" then
        local count = 0
        for _, pl in ipairs(Players:GetPlayers()) do
            if pl ~= LocalPlayer and pl.Character and not isFriend(pl) then
                if sendDamage(pl.Character, weapon, dmg) then
                    count = count + 1
                end
            end
        end
        setStatus("Hit " .. count .. " players with " .. weapon.Name, Color3.fromRGB(80, 255, 120))
        return
    end

    -- Handle team names — hit everyone on that team
    local team = getTeamByName(tName)
    if team then
        local chars = getPlayersOnTeam(team)
        if #chars == 0 then
            setStatus("No players found on team '" .. team.Name .. "'!", Color3.fromRGB(255, 200, 80))
            return
        end
        local count = 0
        for _, char in ipairs(chars) do
            if sendDamage(char, weapon, dmg) then
                count = count + 1
            end
        end
        setStatus("Hit " .. count .. " players on team " .. team.Name, Color3.fromRGB(80, 255, 120))
        return
    end

    -- Handle single target by name
    local target = findTarget(tName)
    if not target then
        setStatus("Target not found! Try a name, 'all', or team name.", Color3.fromRGB(255, 80, 80))
        return
    end

    local ok = sendDamage(target, weapon, dmg)
    if ok then
        setStatus("Killed " .. target.Name .. "!", Color3.fromRGB(80, 255, 120))
    else
        setStatus("Failed to fire remote.", Color3.fromRGB(255, 80, 80))
    end
end

local function killNearest()
    local wName = WeaponBox.Text
    local dmg = tonumber(DamageBox.Text) or Config.Damage
    local weapon = findWeapon(wName)
    if not weapon then
        setStatus("Weapon not found!", Color3.fromRGB(255, 80, 80))
        return
    end

    local enemy = getNearestEnemy(Config.KillAuraRange)
    if enemy then
        local ok = sendDamage(enemy, weapon, dmg)
        if ok then
            setStatus("Hit nearest: " .. enemy.Name, Color3.fromRGB(80, 255, 120))
        end
    else
        setStatus("No enemy in range!", Color3.fromRGB(255, 200, 80))
    end
end

-- ★ UPDATED: Kill All now also accepts team targeting
local function killAllPlayers()
    local wName = WeaponBox.Text
    local dmg = tonumber(DamageBox.Text) or Config.Damage
    local weapon = findWeapon(wName)
    if not weapon then
        setStatus("Weapon not found!", Color3.fromRGB(255, 80, 80))
        return
    end

    local tName = TargetBox.Text

    -- If target box has a team name, only kill that team
    if tName and tName ~= "" then
        local team = getTeamByName(tName)
        if team then
            local chars = getPlayersOnTeam(team)
            local count = 0
            for _, char in ipairs(chars) do
                if sendDamage(char, weapon, dmg) then
                    count = count + 1
                end
            end
            setStatus("Wiped " .. count .. " on team " .. team.Name .. "!", Color3.fromRGB(80, 255, 120))
            return
        end
    end

    -- Otherwise kill ALL non-friend players
    local count = 0
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and not isFriend(pl) then
            if sendDamage(pl.Character, weapon, dmg) then
                count = count + 1
            end
        end
    end
    setStatus("Wiped " .. count .. " players!", Color3.fromRGB(80, 255, 120))
end

createButton("FIRE DAMAGE [Trigger Key]", Color3.fromRGB(210, 50, 50), triggerKill)
createButton("KILL NEAREST ENEMY", Color3.fromRGB(180, 70, 40), killNearest)
createButton("KILL ALL PLAYERS [Kill All Key]", Color3.fromRGB(160, 40, 40), killAllPlayers)

createToggle("Kill Aura (Auto Near)", false, function(st)
    Config.KillAura = st
end)

createSlider("Kill Aura Range", 10, 1000, Config.KillAuraRange, function(val)
    Config.KillAuraRange = val
end)

createToggle("Loop Kill Target", false, function(st)
    Config.LoopKillTarget = st
end)

createSlider("Attack Loop Delay", 0.05, 1, Config.LoopDelay, function(val)
    Config.LoopDelay = val
end)

createToggle("Whitelist Friends", true, function(st)
    Config.WhitelistFriends = st
end)

createKeybind("Fire Keybind", Config.TriggerKey, function(key)
    Config.TriggerKey = key
end)

createKeybind("Kill All Keybind", Config.KillAllKey, function(key)
    Config.KillAllKey = key
end)

createKeybind("Toggle Menu Key", Config.ToggleMenuKey, function(key)
    Config.ToggleMenuKey = key
end)

-- ★ UPDATED: Target Visualizer now supports team highlight
RunService.RenderStepped:Connect(function()
    local targetChar = nil
    
    if TargetBox and TargetBox.Text ~= "" and string.lower(TargetBox.Text) ~= "all" then
        -- Check if it's a team name first
        local team = getTeamByName(TargetBox.Text)
        if team then
            -- Highlight the nearest enemy on that team
            local nearest, shortest = nil, math.huge
            local myChar = LocalPlayer.Character
            local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
            if myRoot then
                for _, pl in ipairs(Players:GetPlayers()) do
                    if pl.Team == team and pl.Character and pl ~= LocalPlayer and not isFriend(pl) then
                        local root = pl.Character:FindFirstChild("HumanoidRootPart")
                        local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                        if root and hum and hum.Health > 0 then
                            local dist = (myRoot.Position - root.Position).Magnitude
                            if dist < shortest then
                                shortest = dist
                                nearest = pl.Character
                            end
                        end
                    end
                end
            end
            targetChar = nearest
        else
            targetChar = findTarget(TargetBox.Text)
        end
    elseif Config.KillAura then
        targetChar = getNearestEnemy(Config.KillAuraRange)
    end

    if targetChar and targetChar:FindFirstChild("HumanoidRootPart") then
        TargetHighlight.Adornee = targetChar
        TargetHighlight.Enabled = true
    else
        TargetHighlight.Enabled = false
        TargetHighlight.Adornee = nil
    end
end)

task.spawn(function()
    while true do
        if Config.KillAura then
            local weapon = findWeapon(WeaponBox.Text)
            if weapon then
                local enemy = getNearestEnemy(Config.KillAuraRange)
                if enemy then
                    sendDamage(enemy, weapon, Config.Damage)
                end
            end
        end

        if Config.LoopKillTarget and TargetBox.Text ~= "" and string.lower(TargetBox.Text) ~= "all" then
            local weapon = findWeapon(WeaponBox.Text)
            
            -- Check if it's a team name
            local team = getTeamByName(TargetBox.Text)
            if team then
                -- Loop: hit all players on that team
                local chars = getPlayersOnTeam(team)
                for _, char in ipairs(chars) do
                    if weapon then
                        sendDamage(char, weapon, Config.Damage)
                    end
                end
            else
                -- Single target loop
                local target = findTarget(TargetBox.Text)
                if weapon and target then
                    sendDamage(target, weapon, Config.Damage)
                end
            end
        end
        task.wait(Config.LoopDelay)
    end
end)

-- Keybind Listeners
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Config.TriggerKey then
        triggerKill()
    elseif input.KeyCode == Config.KillAllKey then
        killAllPlayers()
    elseif input.KeyCode == Config.ToggleMenuKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Notification
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "Favela Kill Suite",
    Text = "Menu loaded! Type 'all', a name, or a team name (e.g. 'Contractor')",
    Duration = 5
})
