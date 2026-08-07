--[[
    Saadx XIT - Aimbot Script for Sintonia RP
    Theme: Red
    Features: Aimbot, FOV Circle, Wall Check, ESP Box 2D, ESP Line
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações
local settings = {
    Aimbot = false,
    FOVCircle = false,
    WallCheck = false,
    ESPBox = false,
    ESPLine = false,
    FOV = 150,
    Smooth = 0.2
}

-- Variáveis da GUI
local screenGui
local mainFrame
local isDragging = false
local dragInput
local dragStart
local startPos
local isMinimized = false

-- Variáveis do ESP Box
local espBoxes = {}
local espBoxEnabled = false

-- Variáveis do ESP Line
local espLines = {}
local espLineEnabled = false

-- ==================== ESP BOX ====================
local function createESPBox(player)
    if espBoxes[player] then return end
    
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Thickness = 1.5
    box.Filled = false
    box.Transparency = 1
    
    local outline = Drawing.new("Square")
    outline.Visible = false
    outline.Color = Color3.fromRGB(200, 0, 0)
    outline.Thickness = 3
    outline.Filled = false
    outline.Transparency = 1
    
    espBoxes[player] = {
        box = box,
        outline = outline,
        character = nil,
        valid = false
    }
end

local function updateESPBox(player)
    local espData = espBoxes[player]
    if not espData then return end
    
    local character = player.Character
    if not character then
        espData.box.Visible = false
        espData.outline.Visible = false
        espData.valid = false
        return
    end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then
        espData.box.Visible = false
        espData.outline.Visible = false
        espData.valid = false
        return
    end
    
    local head = character:FindFirstChild("Head")
    local torso = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso")
    
    if not head or not torso then
        espData.box.Visible = false
        espData.outline.Visible = false
        espData.valid = false
        return
    end
    
    local headPos, headOnScreen = Camera:WorldToViewportPoint(head.Position)
    local torsoPos, torsoOnScreen = Camera:WorldToViewportPoint(torso.Position)
    
    if not headOnScreen or not torsoOnScreen then
        espData.box.Visible = false
        espData.outline.Visible = false
        espData.valid = false
        return
    end
    
    local height = math.abs(headPos.Y - torsoPos.Y) * 2.5
    local width = height * 0.5
    
    local centerX = headPos.X
    local centerY = (headPos.Y + torsoPos.Y) / 2
    
    local x1 = centerX - width / 2
    local y1 = centerY - height / 2
    local x2 = centerX + width / 2
    local y2 = centerY + height / 2
    
    local viewportSize = Camera.ViewportSize
    if x2 < 0 or x1 > viewportSize.X or y2 < 0 or y1 > viewportSize.Y then
        espData.box.Visible = false
        espData.outline.Visible = false
        espData.valid = false
        return
    end
    
    espData.box.Size = Vector2.new(width, height)
    espData.box.Position = Vector2.new(x1, y1)
    espData.box.Visible = true
    
    espData.outline.Size = Vector2.new(width + 2, height + 2)
    espData.outline.Position = Vector2.new(x1 - 1, y1 - 1)
    espData.outline.Visible = true
    
    espData.valid = true
    espData.character = character
end

local function removeESPBox(player)
    local espData = espBoxes[player]
    if espData then
        if espData.box then
            espData.box.Visible = false
            espData.box:Remove()
        end
        if espData.outline then
            espData.outline.Visible = false
            espData.outline:Remove()
        end
        espBoxes[player] = nil
    end
end

local function clearAllESPBoxes()
    for player, _ in pairs(espBoxes) do
        removeESPBox(player)
    end
    espBoxes = {}
end

-- ==================== ESP LINE ====================
local function createESPLine(player)
    if espLines[player] then return end
    
    local line = Drawing.new("Line")
    line.Visible = false
    line.Color = Color3.fromRGB(255, 0, 0)
    line.Thickness = 1.5
    line.Transparency = 1
    
    espLines[player] = {
        line = line,
        character = nil,
        valid = false
    }
end

local function updateESPLine(player)
    local espData = espLines[player]
    if not espData then return end
    
    local character = player.Character
    if not character then
        espData.line.Visible = false
        espData.valid = false
        return
    end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then
        espData.line.Visible = false
        espData.valid = false
        return
    end
    
    local targetPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Head")
    if not targetPart then
        espData.line.Visible = false
        espData.valid = false
        return
    end
    
    local targetPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
    
    if not onScreen then
        espData.line.Visible = false
        espData.valid = false
        return
    end
    
    -- Ponto central inferior da tela
    local viewportSize = Camera.ViewportSize
    local bottomCenter = Vector2.new(viewportSize.X / 2, viewportSize.Y)
    
    -- Atualizar linha
    espData.line.From = bottomCenter
    espData.line.To = Vector2.new(targetPos.X, targetPos.Y)
    espData.line.Visible = true
    espData.valid = true
    espData.character = character
end

local function removeESPLine(player)
    local espData = espLines[player]
    if espData then
        if espData.line then
            espData.line.Visible = false
            espData.line:Remove()
        end
        espLines[player] = nil
    end
end

local function clearAllESPLines()
    for player, _ in pairs(espLines) do
        removeESPLine(player)
    end
    espLines = {}
end

-- ==================== GERENCIADOR ESP ====================
local function setupESPForPlayer(player)
    if player == LocalPlayer then return end
    
    -- Box
    if settings.ESPBox then
        removeESPBox(player)
        createESPBox(player)
    end
    
    -- Line
    if settings.ESPLine then
        removeESPLine(player)
        createESPLine(player)
    end
end

local function setupAllPlayersESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            setupESPForPlayer(player)
        end
    end
end

local function toggleESPBox()
    espBoxEnabled = settings.ESPBox
    
    if espBoxEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                removeESPBox(player)
                createESPBox(player)
            end
        end
    else
        clearAllESPBoxes()
    end
end

local function toggleESPLine()
    espLineEnabled = settings.ESPLine
    
    if espLineEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                removeESPLine(player)
                createESPLine(player)
            end
        end
    else
        clearAllESPLines()
    end
end

local function updateAllESP()
    -- Atualizar Boxes
    if settings.ESPBox then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and not espBoxes[player] then
                createESPBox(player)
            end
        end
        
        for player, _ in pairs(espBoxes) do
            if not player or not player.Parent then
                removeESPBox(player)
            end
        end
        
        for player, _ in pairs(espBoxes) do
            updateESPBox(player)
        end
    else
        for _, espData in pairs(espBoxes) do
            espData.box.Visible = false
            espData.outline.Visible = false
        end
    end
    
    -- Atualizar Lines
    if settings.ESPLine then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and not espLines[player] then
                createESPLine(player)
            end
        end
        
        for player, _ in pairs(espLines) do
            if not player or not player.Parent then
                removeESPLine(player)
            end
        end
        
        for player, _ in pairs(espLines) do
            updateESPLine(player)
        end
    else
        for _, espData in pairs(espLines) do
            espData.line.Visible = false
        end
    end
end

-- ==================== AIMBOT ====================
local function getClosestPlayer()
    local closest = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()
    
    local playersList = {}
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if distance < settings.FOV then
                        table.insert(playersList, {
                            player = player,
                            distance = distance,
                            head = head
                        })
                    end
                end
            end
        end
    end
    
    table.sort(playersList, function(a, b)
        return a.distance < b.distance
    end)
    
    for _, data in ipairs(playersList) do
        if settings.WallCheck then
            local ray = Ray.new(Camera.CFrame.Position, (data.head.Position - Camera.CFrame.Position).unit * 1000)
            local hit, position = workspace:FindPartOnRay(ray, LocalPlayer.Character)
            if hit and hit:IsDescendantOf(data.player.Character) then
                return data.player
            end
        else
            return data.player
        end
    end
    
    return nil
end

local function aimbot()
    if not settings.Aimbot then return end
    
    local target = getClosestPlayer()
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local head = target.Character.Head
        local targetPos = head.Position
        local cameraPos = Camera.CFrame.Position
        
        local lookVector = (targetPos - cameraPos).unit
        local targetCFrame = CFrame.lookAt(cameraPos, cameraPos + lookVector)
        local currentCFrame = Camera.CFrame
        
        local lerpedCFrame = currentCFrame:Lerp(targetCFrame, settings.Smooth)
        Camera.CFrame = lerpedCFrame
    end
end

-- ==================== CRIAÇÃO DA GUI ====================
local function createGUI()
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "SaadxXITGui"
    screenGui.Parent = LocalPlayer.PlayerGui
    screenGui.ResetOnSpawn = false
    
    -- Janela principal
    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 260, 0, 290)
    mainFrame.Position = UDim2.new(0.5, -130, 0.5, -145)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 0, 0)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = Color3.fromRGB(200, 0, 0)
    mainFrame.Parent = screenGui
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 24)
    titleBar.BackgroundColor3 = Color3.fromRGB(40, 0, 0)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    -- Título com Discord
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -55, 1, 0)
    title.Position = UDim2.new(0, 5, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "Saadx XIT | discord.gg/GSWQ3PzTHj"
    title.TextColor3 = Color3.fromRGB(255, 0, 0)
    title.TextSize = 9
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Font = Enum.Font.GothamBold
    title.Parent = titleBar
    
    -- Botão Minimizar
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Name = "MinimizeBtn"
    minimizeBtn.Size = UDim2.new(0, 18, 1, 0)
    minimizeBtn.Position = UDim2.new(1, -42, 0, 0)
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Text = "-"
    minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeBtn.TextSize = 11
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.Parent = titleBar
    
    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseBtn"
    closeBtn.Size = UDim2.new(0, 18, 1, 0)
    closeBtn.Position = UDim2.new(1, -22, 0, 0)
    closeBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    closeBtn.BorderSizePixel = 0
    closeBtn.Text = "X"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 11
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = titleBar
    
    -- Frame de conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "ContentFrame"
    contentFrame.Size = UDim2.new(1, -14, 1, -36)
    contentFrame.Position = UDim2.new(0, 7, 0, 30)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Função para criar botões de toggle
    local function createToggle(parent, text, yPos, settingKey)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Name = text .. "Frame"
        toggleFrame.Size = UDim2.new(1, 0, 0, 24)
        toggleFrame.Position = UDim2.new(0, 0, 0, yPos)
        toggleFrame.BackgroundTransparency = 1
        toggleFrame.Parent = parent
        
        local label = Instance.new("TextLabel")
        label.Name = "Label"
        label.Size = UDim2.new(0.5, 0, 1, 0)
        label.Position = UDim2.new(0, 0, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = text
        label.TextColor3 = Color3.fromRGB(255, 180, 180)
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Font = Enum.Font.Gotham
        label.Parent = toggleFrame
        
        local toggleBtn = Instance.new("TextButton")
        toggleBtn.Name = "ToggleBtn"
        toggleBtn.Size = UDim2.new(0, 50, 1, -4)
        toggleBtn.Position = UDim2.new(0.78, 0, 0, 2)
        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
        toggleBtn.BorderSizePixel = 1
        toggleBtn.BorderColor3 = Color3.fromRGB(200, 0, 0)
        toggleBtn.Text = "OFF"
        toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggleBtn.TextSize = 11
        toggleBtn.Font = Enum.Font.GothamBold
        toggleBtn.Parent = toggleFrame
        
        toggleBtn.MouseButton1Click:Connect(function()
            settings[settingKey] = not settings[settingKey]
            toggleBtn.Text = settings[settingKey] and "ON" or "OFF"
            toggleBtn.BackgroundColor3 = settings[settingKey] and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(50, 0, 0)
            
            if settingKey == "ESPBox" then
                toggleESPBox()
            elseif settingKey == "ESPLine" then
                toggleESPLine()
            end
        end)
        
        return toggleBtn
    end
    
    -- Criar toggles (Aumentado para 5 itens)
    createToggle(contentFrame, "Aimbot", 2, "Aimbot")
    createToggle(contentFrame, "FOV Circle", 26, "FOVCircle")
    createToggle(contentFrame, "Wall Check", 50, "WallCheck")
    createToggle(contentFrame, "ESP Box", 74, "ESPBox")
    createToggle(contentFrame, "ESP Line", 98, "ESPLine")
    
    -- Slider FOV
    local fovSliderFrame = Instance.new("Frame")
    fovSliderFrame.Name = "FOVSliderFrame"
    fovSliderFrame.Size = UDim2.new(1, 0, 0, 38)
    fovSliderFrame.Position = UDim2.new(0, 0, 0, 124)
    fovSliderFrame.BackgroundTransparency = 1
    fovSliderFrame.Parent = contentFrame
    
    -- Label FOV
    local fovLabel = Instance.new("TextLabel")
    fovLabel.Name = "FOVLabel"
    fovLabel.Size = UDim2.new(0, 40, 0, 18)
    fovLabel.Position = UDim2.new(0, 0, 0, 0)
    fovLabel.BackgroundTransparency = 1
    fovLabel.Text = "FOV:"
    fovLabel.TextColor3 = Color3.fromRGB(255, 180, 180)
    fovLabel.TextSize = 12
    fovLabel.TextXAlignment = Enum.TextXAlignment.Left
    fovLabel.Font = Enum.Font.Gotham
    fovLabel.Parent = fovSliderFrame
    
    -- Barra do Slider FOV
    local fovSlider = Instance.new("Frame")
    fovSlider.Name = "FOVSlider"
    fovSlider.Size = UDim2.new(0, 130, 0, 4)
    fovSlider.Position = UDim2.new(0, 35, 0, 7)
    fovSlider.BackgroundColor3 = Color3.fromRGB(60, 0, 0)
    fovSlider.BorderSizePixel = 1
    fovSlider.BorderColor3 = Color3.fromRGB(200, 0, 0)
    fovSlider.Parent = fovSliderFrame
    
    local fovFill = Instance.new("Frame")
    fovFill.Name = "FOVFill"
    fovFill.Size = UDim2.new(settings.FOV / 500, 0, 1, 0)
    fovFill.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    fovFill.BorderSizePixel = 0
    fovFill.Parent = fovSlider
    
    -- Valor FOV
    local fovValue = Instance.new("TextLabel")
    fovValue.Name = "FOVValue"
    fovValue.Size = UDim2.new(0, 40, 0, 18)
    fovValue.Position = UDim2.new(0, 170, 0, 0)
    fovValue.BackgroundTransparency = 1
    fovValue.Text = "150"
    fovValue.TextColor3 = Color3.fromRGB(255, 180, 180)
    fovValue.TextSize = 12
    fovValue.TextXAlignment = Enum.TextXAlignment.Left
    fovValue.Font = Enum.Font.Gotham
    fovValue.Parent = fovSliderFrame
    
    -- Drag do Slider FOV
    local isDraggingFOV = false
    fovSlider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDraggingFOV = true
        end
    end)
    
    fovSlider.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDraggingFOV = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if isDraggingFOV and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mousePos = input.Position.X
            local sliderPos = fovSlider.AbsolutePosition.X
            local sliderSize = fovSlider.AbsoluteSize.X
            local newValue = math.clamp((mousePos - sliderPos) / sliderSize, 0, 1)
            settings.FOV = math.floor(newValue * 500)
            settings.FOV = math.clamp(settings.FOV, 1, 500)
            fovFill.Size = UDim2.new(settings.FOV / 500, 0, 1, 0)
            fovValue.Text = tostring(settings.FOV)
        end
    end)
    
    -- Slider Smooth
    local smoothSliderFrame = Instance.new("Frame")
    smoothSliderFrame.Name = "SmoothSliderFrame"
    smoothSliderFrame.Size = UDim2.new(1, 0, 0, 38)
    smoothSliderFrame.Position = UDim2.new(0, 0, 0, 164)
    smoothSliderFrame.BackgroundTransparency = 1
    smoothSliderFrame.Parent = contentFrame
    
    -- Label Smooth
    local smoothLabel = Instance.new("TextLabel")
    smoothLabel.Name = "SmoothLabel"
    smoothLabel.Size = UDim2.new(0, 55, 0, 18)
    smoothLabel.Position = UDim2.new(0, 0, 0, 0)
    smoothLabel.BackgroundTransparency = 1
    smoothLabel.Text = "Smooth:"
    smoothLabel.TextColor3 = Color3.fromRGB(255, 180, 180)
    smoothLabel.TextSize = 12
    smoothLabel.TextXAlignment = Enum.TextXAlignment.Left
    smoothLabel.Font = Enum.Font.Gotham
    smoothLabel.Parent = smoothSliderFrame
    
    -- Barra do Slider Smooth
    local smoothSlider = Instance.new("Frame")
    smoothSlider.Name = "SmoothSlider"
    smoothSlider.Size = UDim2.new(0, 130, 0, 4)
    smoothSlider.Position = UDim2.new(0, 50, 0, 7)
    smoothSlider.BackgroundColor3 = Color3.fromRGB(60, 0, 0)
    smoothSlider.BorderSizePixel = 1
    smoothSlider.BorderColor3 = Color3.fromRGB(200, 0, 0)
    smoothSlider.Parent = smoothSliderFrame
    
    local smoothFill = Instance.new("Frame")
    smoothFill.Name = "SmoothFill"
    smoothFill.Size = UDim2.new(settings.Smooth, 0, 1, 0)
    smoothFill.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    smoothFill.BorderSizePixel = 0
    smoothFill.Parent = smoothSlider
    
    -- Valor Smooth
    local smoothValue = Instance.new("TextLabel")
    smoothValue.Name = "SmoothValue"
    smoothValue.Size = UDim2.new(0, 40, 0, 18)
    smoothValue.Position = UDim2.new(0, 185, 0, 0)
    smoothValue.BackgroundTransparency = 1
    smoothValue.Text = "0.2"
    smoothValue.TextColor3 = Color3.fromRGB(255, 180, 180)
    smoothValue.TextSize = 12
    smoothValue.TextXAlignment = Enum.TextXAlignment.Left
    smoothValue.Font = Enum.Font.Gotham
    smoothValue.Parent = smoothSliderFrame
    
    -- Drag do Slider Smooth
    local isDraggingSmooth = false
    smoothSlider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDraggingSmooth = true
        end
    end)
    
    smoothSlider.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDraggingSmooth = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if isDraggingSmooth and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mousePos = input.Position.X
            local sliderPos = smoothSlider.AbsolutePosition.X
            local sliderSize = smoothSlider.AbsoluteSize.X
            local newValue = math.clamp((mousePos - sliderPos) / sliderSize, 0, 1)
            settings.Smooth = math.round(newValue * 10) / 10
            settings.Smooth = math.clamp(settings.Smooth, 0.1, 1)
            smoothFill.Size = UDim2.new(settings.Smooth, 0, 1, 0)
            smoothValue.Text = tostring(settings.Smooth)
        end
    end)
    
    -- Arrastar a janela
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    isDragging = false
                end
            end)
        end
    end)
    
    titleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and isDragging then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    -- Conexões dos botões
    minimizeBtn.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        contentFrame.Visible = not isMinimized
        mainFrame.Size = isMinimized and UDim2.new(0, 260, 0, 24) or UDim2.new(0, 260, 0, 290)
        minimizeBtn.Text = isMinimized and "+" or "-"
    end)
    
    closeBtn.MouseButton1Click:Connect(function()
        clearAllESPBoxes()
        clearAllESPLines()
        screenGui:Destroy()
    end)
end

-- ==================== EXECUÇÃO PRINCIPAL ====================

-- Círculo do FOV (centralizado na tela)
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = false
fovCircle.Color = Color3.fromRGB(255, 0, 0)
fovCircle.Thickness = 1.5
fovCircle.Filled = false
fovCircle.NumSides = 50
fovCircle.Radius = settings.FOV
fovCircle.Transparency = 0.8

-- Criar GUI
createGUI()

-- Configurar ESP inicial
if settings.ESPBox then
    espBoxEnabled = true
    setupAllPlayersESP()
else
    espBoxEnabled = false
end

if settings.ESPLine then
    espLineEnabled = true
    setupAllPlayersESP()
else
    espLineEnabled = false
end

-- Lidar com novos jogadores
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        setupESPForPlayer(player)
    end
end)

-- Lidar com remoção de jogadores
Players.PlayerRemoving:Connect(function(player)
    if player ~= LocalPlayer then
        removeESPBox(player)
        removeESPLine(player)
    end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
    -- Atualizar círculo do FOV
    if settings.FOVCircle then
        local viewportSize = Camera.ViewportSize
        fovCircle.Position = Vector2.new(viewportSize.X / 2, viewportSize.Y / 2)
        fovCircle.Radius = settings.FOV
        fovCircle.Visible = true
    else
        fovCircle.Visible = false
    end
    
    -- Aimbot
    if settings.Aimbot then
        aimbot()
    end
    
    -- Atualizar ESP
    updateAllESP()
end)

print("Saadx XIT Carregado com Sucesso!")
