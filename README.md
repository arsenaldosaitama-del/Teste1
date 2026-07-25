--[[
    Saadx Xit Premium
    Versão: Aimbot - Mira Centralizada na Cabeça
]]

local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local camera = game.Workspace.CurrentCamera
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")

-- ====== CONFIGURAÇÕES ======
local settings = {
    aimbot = {
        enabled = false,
        fov = 275,
        smoothness = 0.2,
        aimMode = "Head",
        wallCheck = false,
        showFOV = true
    },
    esp = {
        enabled = false,
        highlight = true,
        name = true,
        distance = true
    },
    misc = {
        antiLag = false
    },
    ui = {
        visible = true,
        toggleKey = "F",
        minimized = false
    }
}

-- ====== CORES ======
local colors = {
    bg = Color3.fromRGB(15, 15, 20),
    primary = Color3.fromRGB(120, 50, 200),
    secondary = Color3.fromRGB(80, 30, 150),
    accent = Color3.fromRGB(180, 100, 255),
    text = Color3.fromRGB(230, 230, 255),
    border = Color3.fromRGB(60, 20, 100),
    fov = Color3.fromRGB(180, 80, 255),
    header = Color3.fromRGB(30, 30, 40),
    highlight = Color3.fromRGB(120, 50, 200)
}

-- ====== FOV CIRCLE ======
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = settings.aimbot.fov
fovCircle.Thickness = 2
fovCircle.Color = colors.fov
fovCircle.Filled = false
fovCircle.Visible = false
fovCircle.Position = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

-- ====== HIGHLIGHTS ======
local highlightObjects = {}

local function createHighlight(playerObj)
    if not highlightObjects[playerObj] then
        local highlight = Instance.new("Highlight")
        highlight.Name = "SaadxHighlight"
        highlight.FillColor = colors.highlight
        highlight.FillTransparency = 0.3
        highlight.OutlineColor = colors.highlight
        highlight.OutlineTransparency = 0.1
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Adornee = playerObj.Character
        highlight.Parent = playerObj.Character
        highlightObjects[playerObj] = highlight
    end
end

local function removeHighlight(playerObj)
    if highlightObjects[playerObj] then
        highlightObjects[playerObj]:Destroy()
        highlightObjects[playerObj] = nil
    end
end

local function updateHighlights()
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player then
            if v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
                if settings.esp.enabled and settings.esp.highlight then
                    createHighlight(v)
                else
                    removeHighlight(v)
                end
            else
                removeHighlight(v)
            end
        end
    end
end

-- ====== ANTI LAG ======
local antiLagObjects = {}

local function applyAntiLag()
    if settings.misc.antiLag then
        for _, v in pairs(game:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
                v.Enabled = false
                table.insert(antiLagObjects, v)
            end
            if v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 1
                table.insert(antiLagObjects, v)
            end
            if v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") then
                v.Enabled = false
                table.insert(antiLagObjects, v)
            end
            if v:IsA("Terrain") then
                v.WaterWaveSize = 0
                v.WaterReflectance = 0
                v.WaterTransparency = 1
            end
        end
        
        game.Lighting.GlobalShadows = false
        game.Lighting.Brightness = 1
        game.Lighting.Ambient = Color3.fromRGB(255, 255, 255)
        game.Lighting.ClockTime = 12
        game.Lighting.GeographicLatitude = 0
        
        print("⚡ Anti Lag ATIVADO!")
    else
        for _, obj in pairs(antiLagObjects) do
            if obj and obj.Parent then
                if obj:IsA("ParticleEmitter") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                    obj.Enabled = true
                end
                if obj:IsA("Decal") or obj:IsA("Texture") then
                    obj.Transparency = 0
                end
                if obj:IsA("BloomEffect") or obj:IsA("BlurEffect") or obj:IsA("SunRaysEffect") or obj:IsA("ColorCorrectionEffect") then
                    obj.Enabled = true
                end
            end
        end
        antiLagObjects = {}
        
        game.Lighting.GlobalShadows = true
        game.Lighting.Brightness = 2
        game.Lighting.Ambient = Color3.fromRGB(127, 127, 127)
        
        print("⚡ Anti Lag DESATIVADO!")
    end
end

-- ====== GET CLOSEST PLAYER ======
local function getClosestPlayer()
    local closest = nil
    local bestScore = math.huge
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local camPos = camera.CFrame.Position
    
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            local aimPartName = settings.aimbot.aimMode == "Head" and "Head" or "UpperTorso"
            local aimPart = v.Character:FindFirstChild(aimPartName)
            
            if not aimPart and settings.aimbot.aimMode == "Chest" then
                aimPart = v.Character:FindFirstChild("Torso")
            end
            
            if not aimPart then
                aimPart = v.Character:FindFirstChild("Head")
                if not aimPart then continue end
            end
            
            if settings.aimbot.wallCheck then
                local ray = Ray.new(camPos, (aimPart.Position - camPos).Unit * 1000)
                local hit = game.Workspace:FindPartOnRay(ray, player.Character)
                if hit and hit.Parent ~= v.Character then continue end
            end
            
            local pos, onScreen = camera:WorldToScreenPoint(aimPart.Position)
            if onScreen and pos.Z > 0 then
                local screenDist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                if screenDist < settings.aimbot.fov then
                    local root = v.Character:FindFirstChild("HumanoidRootPart")
                    local physicalDist = root and (camPos - root.Position).Magnitude or math.huge
                    
                    if physicalDist < bestScore then
                        bestScore = physicalDist
                        closest = v
                    end
                end
            end
        end
    end
    return closest
end

-- ====== AIMBOT (MIRA CENTRALIZADA NA CABEÇA) ======
local function aimbot()
    if not settings.aimbot.enabled then return end
    local target = getClosestPlayer()
    if not target then return end
    
    local aimPartName = settings.aimbot.aimMode == "Head" and "Head" or "UpperTorso"
    local aimPart = target.Character:FindFirstChild(aimPartName)
    
    if not aimPart and settings.aimbot.aimMode == "Chest" then
        aimPart = target.Character:FindFirstChild("Torso")
    end
    
    if not aimPart then
        aimPart = target.Character:FindFirstChild("Head")
        if not aimPart then return end
    end
    
    local camPos = camera.CFrame.Position
    local aimPos = aimPart.Position
    
    -- AJUSTE: Centraliza na cabeça (sem offset, mira no centro exato)
    if settings.aimbot.aimMode == "Head" then
        aimPos = aimPos + Vector3.new(0, 0, 0) -- SEM OFFSET, CENTRO EXATO
    else
        aimPos = aimPos + Vector3.new(0, 0, 0)
    end
    
    local direction = (aimPos - camPos).Unit
    local targetCF = CFrame.lookAt(camPos, camPos + direction)
    local currentCF = camera.CFrame
    local smoothValue = 1 - settings.aimbot.smoothness
    
    camera.CFrame = currentCF:Lerp(targetCF, smoothValue)
end

-- ====== ESP TEXTOS ======
local espTexts = {}

local function updateESPTexts()
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player then
            if v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
                local torso = v.Character:FindFirstChild("UpperTorso") or v.Character:FindFirstChild("Torso")
                local root = v.Character:FindFirstChild("HumanoidRootPart")
                
                if torso and root then
                    local torsoPos, torsoOnScreen = camera:WorldToScreenPoint(torso.Position)
                    local rootPos, rootOnScreen = camera:WorldToScreenPoint(root.Position)
                    
                    if torsoOnScreen and rootOnScreen and torsoPos.Z > 0 and rootPos.Z > 0 then
                        if not espTexts[v] then
                            espTexts[v] = {
                                name = Drawing.new("Text"),
                                distance = Drawing.new("Text")
                            }
                            local esp = espTexts[v]
                            esp.name.Color = colors.accent
                            esp.name.Size = 14
                            esp.name.Center = true
                            esp.name.Outline = true
                            esp.name.OutlineColor = Color3.fromRGB(0,0,0)
                            esp.name.Visible = false
                            esp.distance.Color = Color3.fromRGB(200,200,200)
                            esp.distance.Size = 12
                            esp.distance.Center = true
                            esp.distance.Outline = true
                            esp.distance.OutlineColor = Color3.fromRGB(0,0,0)
                            esp.distance.Visible = false
                        end
                        
                        local esp = espTexts[v]
                        
                        if settings.esp.enabled and settings.esp.name then
                            esp.name.Text = v.Name
                            esp.name.Position = Vector2.new(torsoPos.X, torsoPos.Y - 25)
                            esp.name.Visible = true
                        else
                            esp.name.Visible = false
                        end
                        
                        if settings.esp.enabled and settings.esp.distance then
                            local distance = (camera.CFrame.Position - root.Position).Magnitude / 10
                            esp.distance.Text = string.format("%.1fm", distance)
                            esp.distance.Position = Vector2.new(torsoPos.X, torsoPos.Y + 25)
                            esp.distance.Visible = true
                        else
                            esp.distance.Visible = false
                        end
                    else
                        if espTexts[v] then
                            espTexts[v].name.Visible = false
                            espTexts[v].distance.Visible = false
                        end
                    end
                end
            else
                if espTexts[v] then
                    espTexts[v].name.Visible = false
                    espTexts[v].distance.Visible = false
                end
                removeHighlight(v)
            end
        end
    end
end

-- ====== LIMPEZA QUANDO JOGADOR SAI ======
game.Players.PlayerRemoving:Connect(function(v)
    removeHighlight(v)
    if espTexts[v] then
        espTexts[v].name:Destroy()
        espTexts[v].distance:Destroy()
        espTexts[v] = nil
    end
end)

-- ====== INTERFACE ======
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 420)
mainFrame.Position = UDim2.new(0.02, 0, 0.5, -210)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = colors.border
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui
mainFrame.Visible = settings.ui.visible

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- ====== HEADER ======
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 40)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = colors.header
header.BackgroundTransparency = 0.5
header.Parent = mainFrame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 8)
headerCorner.Parent = header

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 1, 0)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Saadx Xit"
title.TextColor3 = colors.text
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Center
title.Font = Enum.Font.GothamBold
title.Parent = header

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 1, -6)
minimizeBtn.Position = UDim2.new(1, -70, 0, 3)
minimizeBtn.BackgroundColor3 = colors.secondary
minimizeBtn.Text = "▼"
minimizeBtn.TextColor3 = colors.text
minimizeBtn.TextSize = 16
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.Parent = header

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 4)
minCorner.Parent = minimizeBtn

minimizeBtn.MouseButton1Click:Connect(function()
    settings.ui.minimized = not settings.ui.minimized
    minimizeBtn.Text = settings.ui.minimized and "▶" or "▼"
    if settings.ui.minimized then
        mainFrame.Size = UDim2.new(0, 280, 0, 40)
        for _, child in pairs(mainFrame:GetChildren()) do
            if child ~= header then child.Visible = false end
        end
        header.Visible = true
        minimizeBtn.Visible = true
        closeBtn.Visible = true
    else
        mainFrame.Size = UDim2.new(0, 280, 0, 420)
        for _, child in pairs(mainFrame:GetChildren()) do
            child.Visible = true
        end
        header.Visible = true
        minimizeBtn.Visible = true
        closeBtn.Visible = true
    end
end)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 1, -6)
closeBtn.Position = UDim2.new(1, -35, 0, 3)
closeBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
closeBtn.Text = "✕"
closeBtn.TextColor3 = colors.text
closeBtn.TextSize = 16
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = header

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
    settings.ui.visible = false
    mainFrame.Visible = false
end)

-- ====== CONTEÚDO ======
local content = Instance.new("Frame")
content.Size = UDim2.new(1, -20, 1, -50)
content.Position = UDim2.new(0, 10, 0, 45)
content.BackgroundTransparency = 1
content.Parent = mainFrame

-- ====== TABS ======
local tabFrame = Instance.new("Frame")
tabFrame.Size = UDim2.new(1, 0, 0, 26)
tabFrame.Position = UDim2.new(0, 0, 0, 0)
tabFrame.BackgroundTransparency = 1
tabFrame.Parent = content

local tabs = {"PvP", "Player", "Misc"}
local selectedTab = 1

local function criarAbas()
    for i, tabName in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 74, 1, -4)
        btn.Position = UDim2.new((i-1) * 0.33, 2, 0, 2)
        btn.BackgroundColor3 = i == 1 and colors.primary or colors.secondary
        btn.Text = tabName
        btn.TextColor3 = colors.text
        btn.TextSize = 12
        btn.Font = Enum.Font.Gotham
        btn.Parent = tabFrame
        btn.Name = "Tab" .. i
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = btn
        
        btn.MouseButton1Click:Connect(function()
            selectedTab = i
            for j = 1, 3 do
                local b = tabFrame:FindFirstChild("Tab" .. j)
                if b then
                    b.BackgroundColor3 = j == i and colors.primary or colors.secondary
                end
            end
            updateContent()
        end)
    end
end
criarAbas()

-- ====== CONTEÚDO DAS TABS ======
local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -35)
contentContainer.Position = UDim2.new(0, 0, 0, 32)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = content

-- ====== FUNÇÕES DE CRIAÇÃO DA UI ======
function createToggle(text, y, callback, initialValue)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 25)
    frame.Position = UDim2.new(0, 0, 0, y)
    frame.BackgroundTransparency = 1
    frame.Parent = contentContainer
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 22, 0, 22)
    btn.Position = UDim2.new(0, 0, 0, 0)
    btn.BackgroundColor3 = initialValue and colors.accent or colors.secondary
    btn.Text = initialValue and "✓" or ""
    btn.TextColor3 = colors.text
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = frame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 4)
    btnCorner.Parent = btn
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -30, 1, 0)
    label.Position = UDim2.new(0, 28, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = colors.text
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.Parent = frame
    
    btn.MouseButton1Click:Connect(function()
        local newValue = btn.Text == ""
        btn.Text = newValue and "✓" or ""
        btn.BackgroundColor3 = newValue and colors.accent or colors.secondary
        callback()
    end)
    return btn
end

function createSlider(text, y, defaultValue, min, max, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 50)
    frame.Position = UDim2.new(0, 0, 0, y)
    frame.BackgroundTransparency = 1
    frame.Parent = contentContainer
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 20)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = colors.text
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    label.Parent = frame
    
    local valueDisplay = Instance.new("TextLabel")
    valueDisplay.Size = UDim2.new(0, 60, 0, 20)
    valueDisplay.Position = UDim2.new(1, -60, 0, 0)
    valueDisplay.BackgroundTransparency = 1
    valueDisplay.Text = tostring(defaultValue)
    valueDisplay.TextColor3 = colors.accent
    valueDisplay.TextSize = 13
    valueDisplay.TextXAlignment = Enum.TextXAlignment.Right
    valueDisplay.Font = Enum.Font.GothamBold
    valueDisplay.Parent = frame
    
    local slider = Instance.new("Frame")
    slider.Size = UDim2.new(1, 0, 0, 6)
    slider.Position = UDim2.new(0, 0, 0, 28)
    slider.BackgroundColor3 = colors.secondary
    slider.Parent = frame
    
    local sliderCorner = Instance.new("UICorner")
    sliderCorner.CornerRadius = UDim.new(0, 3)
    sliderCorner.Parent = slider
    
    local fill = Instance.new("Frame")
    local valuePercent = (defaultValue - min) / (max - min)
    fill.Size = UDim2.new(valuePercent, 0, 1, 0)
    fill.BackgroundColor3 = colors.accent
    fill.Parent = slider
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 3)
    fillCorner.Parent = fill
    
    local dragging = false
    local function updateSlider(mouseX)
        local pos = mouseX - slider.AbsolutePosition.X
        local width = slider.AbsoluteSize.X
        if width <= 0 then return end
        local value = math.clamp(pos / width, 0, 1)
        local finalValue = math.floor(value * (max - min) + min)
        fill.Size = UDim2.new(value, 0, 1, 0)
        valueDisplay.Text = tostring(finalValue)
        callback(finalValue)
    end
    
    slider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            updateSlider(input.Position.X)
        end
    end)
    userInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    userInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            updateSlider(input.Position.X)
        end
    end)
end

-- ====== ATUALIZAR CONTEÚDO DAS ABAS ======
function updateContent()
    for _, child in pairs(contentContainer:GetChildren()) do
        child:Destroy()
    end

    if selectedTab == 1 then -- PVP (AIMBOT)
        createToggle("Ativar Aimbot", 0, function()
            settings.aimbot.enabled = not settings.aimbot.enabled
            fovCircle.Visible = settings.aimbot.showFOV and settings.aimbot.enabled
        end, settings.aimbot.enabled)

        createToggle("Mirar no Tronco", 30, function()
            if settings.aimbot.aimMode == "Head" then
                settings.aimbot.aimMode = "Chest"
                print("🔽 Mirando no TRONCO!")
            else
                settings.aimbot.aimMode = "Head"
                print("🔼 Mirando na CABEÇA!")
            end
        end, settings.aimbot.aimMode == "Chest")

        createToggle("Wall Check", 60, function()
            settings.aimbot.wallCheck = not settings.aimbot.wallCheck
        end, settings.aimbot.wallCheck)

        createToggle("Mostrar FOV", 90, function()
            settings.aimbot.showFOV = not settings.aimbot.showFOV
            fovCircle.Visible = settings.aimbot.showFOV and settings.aimbot.enabled
        end, settings.aimbot.showFOV)

        createSlider("Tamanho do FOV", 120, 275, 30, 400, function(value)
            settings.aimbot.fov = value
            fovCircle.Radius = value
        end)

        createSlider("Suavização (Smooth)", 170, 2, 1, 10, function(value)
            settings.aimbot.smoothness = value / 10
        end)

    elseif selectedTab == 2 then -- PLAYER (ESP)
        createToggle("Ativar ESP", 0, function()
            settings.esp.enabled = not settings.esp.enabled
            updateHighlights()
        end, settings.esp.enabled)
        
        createToggle("Highlight (Glow)", 30, function()
            settings.esp.highlight = not settings.esp.highlight
            updateHighlights()
        end, settings.esp.highlight)
        
        createToggle("Nome", 60, function()
            settings.esp.name = not settings.esp.name
        end, settings.esp.name)
        
        createToggle("Distância", 90, function()
            settings.esp.distance = not settings.esp.distance
        end, settings.esp.distance)

    elseif selectedTab == 3 then -- MISC
        createToggle("Anti Lag", 0, function()
            settings.misc.antiLag = not settings.misc.antiLag
            applyAntiLag()
        end, settings.misc.antiLag)
    end
end

updateContent()

-- ====== KEYBIND ======
userInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode[settings.ui.toggleKey] and input.UserInputType == Enum.UserInputType.Keyboard then
        settings.ui.visible = not settings.ui.visible
        mainFrame.Visible = settings.ui.visible
    end
end)

-- ====== LOOP PRINCIPAL ======
runService.RenderStepped:Connect(function()
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    fovCircle.Position = center
    fovCircle.Radius = settings.aimbot.fov
    fovCircle.Visible = settings.aimbot.showFOV and settings.aimbot.enabled
    
    if settings.aimbot.enabled then aimbot() end
    if settings.esp.enabled then 
        updateHighlights()
        updateESPTexts()
    else
        for _, v in pairs(game.Players:GetPlayers()) do
            removeHighlight(v)
        end
    end
end)

player.CharacterAdded:Connect(function()
    wait(0.5)
    camera = game.Workspace.CurrentCamera
end)

print("⚡ Saadx Xit Premium carregado!")
print("🔑 Pressione F para abrir/fechar")
print("🎯 Mira CENTRALIZADA na cabeça!")
