local Fluent = loadstring(game:HttpGet('https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua'))()
local SaveManager = loadstring(game:HttpGet('https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua'))()
local InterfaceManager = loadstring(game:HttpGet('https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua'))()

-- Services
local TweenService = game:GetService('TweenService')
local RunService = game:GetService('RunService')
local Players = game:GetService('Players')
local VirtualInputManager = game:GetService('VirtualInputManager')

local player = Players.LocalPlayer
local gravityNormal = workspace.Gravity

-- Script Variables
local isRunning = false
local currentTween = nil
local speed = 1000
local wsValue = 16
local jpValue = 50
local waterPlatforms = {}

local destinations = {
    CFrame.new(-43.6134491, 62.1137619, 672.744934, -0.999842644, -0.00183729955, 0.017645346, 0, 0.994622767, 0.103564225, -0.0177407414, 0.103547923, -0.994466245),
    CFrame.new(-60.1504707, 97.4659729, 8767.91406, -0.99889338, 0.000705028593, 0.0470264405, 0, 0.999887645, -0.0149902813, -0.047031723, -0.0149736926, -0.998781145),
    CFrame.new(-54.331871, -345.398346, 9488.60645, -0.98221302, 0, 0.187770084, 0, 1, 0, -0.187770084, 0, -0.98221302),
}

local Window = Fluent:CreateWindow({
    Title = 'Ceverz Hub',
    SubTitle = '',
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = 'Dark',
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = 'Main', Icon = 'home' }),
    Farm = Window:AddTab({ Title = 'Farm', Icon = 'shrub' }),
    Misc = Window:AddTab({ Title = 'Misc', Icon = 'layers' }),
    Settings = Window:AddTab({ Title = 'Settings', Icon = 'settings' })
}

-- [ MAIN TAB ] --
Tabs.Main:AddParagraph({
    Title = 'Game: Build a Boat for Treasure',
    Content = ''
})

Tabs.Main:AddButton({
    Title = 'Join Ceverz Discord',
    Callback = function() end
})

local AntiAfkConnection
local AntiAfkToggle = Tabs.Main:AddToggle('AntiAfk', {
    Title = 'Anti-AFK', 
    Default = false,
    Description = 'Spins and walks your character slightly'
})

AntiAfkToggle:OnChanged(function()
    if AntiAfkToggle.Value then
        AntiAfkConnection = RunService.Heartbeat:Connect(function()
            local Character = player.Character
            local Humanoid = Character and Character:FindFirstChild('Humanoid')
            local RootPart = Character and Character:FindFirstChild('HumanoidRootPart')
            
            if RootPart and Humanoid then
                RootPart.CFrame = RootPart.CFrame * CFrame.Angles(0, math.rad(5), 0)
                if tick() % 2 < 1 then
                    Humanoid:Move(Vector3.new(0, 0, 0.1), true)
                else
                    Humanoid:Move(Vector3.new(0, 0, -0.1), true)
                end
            end
        end)
    else
        if AntiAfkConnection then
            AntiAfkConnection:Disconnect()
            AntiAfkConnection = nil
        end
    end
end)

-- [ FARM TAB ] --
local function moveTo(targetCFrame, setGravity)
    local root = player.Character and player.Character:FindFirstChild('HumanoidRootPart')
    if not root then return end

    local reached = false
    workspace.Gravity = setGravity and gravityNormal or 0

    while isRunning and not reached do
        local currentRoot = player.Character and player.Character:FindFirstChild('HumanoidRootPart')
        if not currentRoot then break end
        
        local distance = (currentRoot.Position - targetCFrame.Position).Magnitude
        if distance < 5 then 
            reached = true 
            break 
        end

        local duration = distance / math.max(speed, 1)
        
        if currentTween then currentTween:Cancel() end
        currentTween = TweenService:Create(currentRoot, TweenInfo.new(duration, Enum.EasingStyle.Linear), {CFrame = targetCFrame})
        currentTween:Play()
        
        task.wait(0.1)
    end
    
    if currentTween then currentTween:Cancel() end
end

local function autoFarmLoop()
    while isRunning do
        local char = player.Character or player.CharacterAdded:Wait()
        local root = char:WaitForChild('HumanoidRootPart', 5)
        if not root then task.wait(1) continue end

        for i, cf in ipairs(destinations) do
            if not isRunning then break end
            moveTo(cf, i == #destinations)
        end

        if isRunning then
            task.wait(2)
            if player.Character and player.Character:FindFirstChild('Humanoid') then
                player.Character.Humanoid.Health = 0
            end
            player.CharacterAdded:Wait()
            task.wait(1)
        end
    end
end

local AutoFarmToggle = Tabs.Farm:AddToggle('AutoFarmBeta', {
    Title = 'AutoFarm (beta)',
    Default = false,
    Description = 'Finishes the game in under 30 seconds, then repeats.'
})

AutoFarmToggle:OnChanged(function()
    isRunning = AutoFarmToggle.Value
    if not isRunning then
        workspace.Gravity = gravityNormal
        if currentTween then
            currentTween:Cancel()
        end
    else
        task.spawn(autoFarmLoop)
    end
end)

Tabs.Farm:AddSlider('AFSpeed', {
    Title = 'AFSpeed',
    Description = 'Speed for AutoFarm',
    Default = 1000,
    Min = 1,
    Max = 1000,
    Rounding = 0,
    Callback = function(Value)
        speed = Value
    end
})

-- [ MISC TAB ] --
local WSToggle = Tabs.Misc:AddToggle('WSToggle', {Title = 'WalkSpeed', Default = false})
WSToggle:OnChanged(function()
    if not WSToggle.Value then
        if player.Character and player.Character:FindFirstChild('Humanoid') then
            player.Character.Humanoid.WalkSpeed = 16
        end
    end
end)

RunService.Heartbeat:Connect(function()
    if WSToggle.Value and player.Character and player.Character:FindFirstChild('Humanoid') then
        player.Character.Humanoid.WalkSpeed = wsValue
    end
end)

Tabs.Misc:AddSlider('WSSlider', {
    Title = 'WalkSpeed Slider',
    Default = 16,
    Min = 0,
    Max = 1000,
    Rounding = 0,
    Callback = function(Value) wsValue = Value end
})

Tabs.Misc:AddParagraph({Title = '--------------------------------------', Content = ''})

local JPToggle = Tabs.Misc:AddToggle('JPToggle', {Title = 'JumpPower', Default = false})
JPToggle:OnChanged(function()
    if not JPToggle.Value then
        if player.Character and player.Character:FindFirstChild('Humanoid') then
            player.Character.Humanoid.JumpPower = 50
        end
    end
end)

RunService.Heartbeat:Connect(function()
    if JPToggle.Value and player.Character and player.Character:FindFirstChild('Humanoid') then
        player.Character.Humanoid.UseJumpPower = true
        player.Character.Humanoid.JumpPower = jpValue
    end
end)

Tabs.Misc:AddSlider('JPSlider', {
    Title = 'JumpPower Slider',
    Default = 50,
    Min = 0,
    Max = 1000,
    Rounding = 0,
    Callback = function(Value) jpValue = Value end
})

Tabs.Misc:AddParagraph({Title = '---------------------------------------', Content = ''})

local NoDamageToggle = Tabs.Misc:AddToggle('NoDamage', {Title = 'NoDamage', Default = false})
NoDamageToggle:OnChanged(function()
    if NoDamageToggle.Value then
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj.Name == 'Water' and obj:IsA('BasePart') then
                local platform = Instance.new('Part')
                platform.Name = 'CeverzWaterGuard'
                platform.Size = Vector3.new(obj.Size.X, 1, obj.Size.Z)
                platform.CFrame = obj.CFrame * CFrame.new(0, (obj.Size.Y / 2) + 0.5, 0)
                platform.Anchored = true
                platform.Material = Enum.Material.Glass
                platform.Transparency = 0.7
                platform.CanCollide = true
                platform.Parent = workspace
                table.insert(waterPlatforms, platform)
            end
        end
    else
        for _, platform in pairs(waterPlatforms) do
            if platform then platform:Destroy() end
        end
        waterPlatforms = {}
    end
end)

-- Background Anti-Disconnect (K-press)
task.spawn(function()
    while true do
        task.wait(10)
        if isRunning then
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.K, false, game)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.K, false, game)
        end
    end
end)

-- [ SETTINGS TAB ] --
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})
InterfaceManager:SetFolder('CeverzHub')
SaveManager:SetFolder('CeverzHub/configs')

InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

Window:SelectTab(1)

Fluent:Notify({
    Title = 'Ceverz Hub',
    Content = 'Custom Hub Loaded Successfully',
    Duration = 8
})

SaveManager:LoadAutoloadConfig()
