-- Конфигурация
local AIM_KEY = Enum.KeyCode.LeftShift -- Клавиша для плавного аима
local TOGGLE_KEY = Enum.KeyCode.RightControl -- Клавиша для открытия/закрытия меню
local AIM_SMOOTHNESS = 0.2 -- Плавность аима (чем меньше, тем плавнее)
local AUTO_AIM = true -- Автоматическая фиксация на цели без удержания клавиши

-- Переменные
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local GuiService = game:GetService("GuiService")

local target = nil
local isAiming = false
local isMenuOpen = false
local targetPart = "Head" -- По умолчанию цель - голова
local playerList = {}
local selectedPlayer = nil
local isAutoAimEnabled = true -- Автоаим включен по умолчанию

-- Создаем GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimBotUI"
screenGui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0.35, 0, 0.55, 0)
frame.Position = UDim2.new(0.05, 0, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BackgroundTransparency = 0.5
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0.07, 0)
title.Text = "AimBot Menu"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.Parent = frame

local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0.2, 0, 0.1, 0)
openButton.Position = UDim2.new(0.05, 0, 0.05, 0)
openButton.Text = "Open"
openButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.Parent = screenGui

-- Панель выбора части тела
local targetPartLabel = Instance.new("TextLabel")
targetPartLabel.Size = UDim2.new(1, 0, 0.07, 0)
targetPartLabel.Position = UDim2.new(0, 0, 0.1, 0)
targetPartLabel.Text = "Target Part: " .. targetPart
targetPartLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
targetPartLabel.BackgroundTransparency = 1
targetPartLabel.Parent = frame

local headButton = Instance.new("TextButton")
headButton.Size = UDim2.new(0.8, 0, 0.07, 0)
headButton.Position = UDim2.new(0.1, 0, 0.18, 0)
headButton.Text = "Head"
headButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
headButton.TextColor3 = Color3.fromRGB(255, 255, 255)
headButton.Parent = frame

local torsoButton = Instance.new("TextButton")
torsoButton.Size = UDim2.new(0.8, 0, 0.07, 0)
torsoButton.Position = UDim2.new(0.1, 0, 0.26, 0)
torsoButton.Text = "Torso"
torsoButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
torsoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
torsoButton.Parent = frame

local humanoidRootPartButton = Instance.new("TextButton")
humanoidRootPartButton.Size = UDim2.new(0.8, 0, 0.07, 0)
humanoidRootPartButton.Position = UDim2.new(0.1, 0, 0.34, 0)
humanoidRootPartButton.Text = "HumanoidRootPart"
humanoidRootPartButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
humanoidRootPartButton.TextColor3 = Color3.fromRGB(255, 255, 255)
humanoidRootPartButton.Parent = frame

-- Панель выбора игрока
local playerSelectionLabel = Instance.new("TextLabel")
playerSelectionLabel.Size = UDim2.new(1, 0, 0.07, 0)
playerSelectionLabel.Position = UDim2.new(0, 0, 0.42, 0)
playerSelectionLabel.Text = "Selected: AUTO"
playerSelectionLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
playerSelectionLabel.BackgroundTransparency = 1
playerSelectionLabel.Parent = frame

local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(0.9, 0, 0.25, 0)
playerListFrame.Position = UDim2.new(0.05, 0, 0.5, 0)
playerListFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
playerListFrame.BackgroundTransparency = 0.5
playerListFrame.BorderSizePixel = 0
playerListFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
playerListFrame.ScrollBarThickness = 5
playerListFrame.Parent = frame

-- Настройка автоаима
local autoAimLabel = Instance.new("TextLabel")
autoAimLabel.Size = UDim2.new(1, 0, 0.07, 0)
autoAimLabel.Position = UDim2.new(0, 0, 0.76, 0)
autoAimLabel.Text = "Auto Aim: ON"
autoAimLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
autoAimLabel.BackgroundTransparency = 1
autoAimLabel.Parent = frame

local toggleAutoAimButton = Instance.new("TextButton")
toggleAutoAimButton.Size = UDim2.new(0.8, 0, 0.07, 0)
toggleAutoAimButton.Position = UDim2.new(0.1, 0, 0.84, 0)
toggleAutoAimButton.Text = "Toggle Auto Aim"
toggleAutoAimButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
toggleAutoAimButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleAutoAimButton.Parent = frame

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0.8, 0, 0.07, 0)
closeButton.Position = UDim2.new(0.1, 0, 0.92, 0)
closeButton.Text = "Close"
closeButton.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Parent = frame

-- Функции
local function updatePlayerList()
    -- Очищаем старый список
    for _, child in ipairs(playerListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- Обновляем список игроков
    playerList = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(playerList, player)
            
            local playerButton = Instance.new("TextButton")
            playerButton.Size = UDim2.new(1, 0, 0, 30)
            playerButton.Position = UDim2.new(0, 0, 0, #playerList * 32 - 32)
            playerButton.Text = player.Name
            playerButton.BackgroundColor3 = selectedPlayer == player and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(60, 60, 60)
            playerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            playerButton.AutoButtonColor = true
            playerButton.Parent = playerListFrame
            
            playerButton.MouseButton1Click:Connect(function()
                selectedPlayer = player
                playerSelectionLabel.Text = "Selected: " .. player.Name
                updatePlayerList() -- Обновляем список для подсветки выбранного игрока
                target = selectedPlayer -- Немедленно обновляем цель
            end)
        end
    end
    
    -- Добавляем опцию "Автовыбор"
    local autoSelectButton = Instance.new("TextButton")
    autoSelectButton.Size = UDim2.new(1, 0, 0, 30)
    autoSelectButton.Position = UDim2.new(0, 0, 0, #playerList * 32)
    autoSelectButton.Text = "AUTO SELECT"
    autoSelectButton.BackgroundColor3 = not selectedPlayer and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(60, 60, 60)
    autoSelectButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    autoSelectButton.AutoButtonColor = true
    autoSelectButton.Parent = playerListFrame
    
    autoSelectButton.MouseButton1Click:Connect(function()
        selectedPlayer = nil
        playerSelectionLabel.Text = "Selected: AUTO"
        updatePlayerList()
        target = nil -- Сбрасываем цель для авто выбора
    end)
end

local function isValidTarget(player)
    return player and player.Character and player.Character:FindFirstChild(targetPart) and 
           player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0
end

local function findTarget()
    if selectedPlayer and isValidTarget(selectedPlayer) then
        return selectedPlayer
    end
    
    -- Если нет выбранного игрока или он невалиден, ищем ближайшего
    if not LocalPlayer or not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then 
        return nil 
    end
    
    local closestPlayer = nil
    local shortestDistance = math.huge
    local localPosition = LocalPlayer.Character.HumanoidRootPart.Position
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isValidTarget(player) then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                local distance = (humanoidRootPart.Position - localPosition).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    
    return closestPlayer
end

local function aimAtTarget()
    if not target or not isValidTarget(target) then 
        target = findTarget()
        if not target then return end
    end
    
    local targetCharacter = target.Character
    local targetPosition = targetCharacter:FindFirstChild(targetPart).Position
    
    local cameraCFrame = Camera.CFrame
    local cameraPosition = cameraCFrame.Position
    
    if isAiming or isAutoAimEnabled then
        -- Плавный аим при удержании клавиши или автоаиме
        local direction = (targetPosition - cameraPosition).Unit
        Camera.CFrame = CFrame.new(cameraPosition, cameraPosition + direction:Lerp((targetPosition - cameraPosition).Unit, AIM_SMOOTHNESS))
    else
        -- Мгновенный аим
        Camera.CFrame = CFrame.new(cameraPosition, targetPosition)
    end
end

-- Обработчики событий
openButton.MouseButton1Click:Connect(function()
    isMenuOpen = not isMenuOpen
    frame.Visible = isMenuOpen
    openButton.Visible = not isMenuOpen
    updatePlayerList()
end)

closeButton.MouseButton1Click:Connect(function()
    isMenuOpen = false
    frame.Visible = false
    openButton.Visible = true
end)

headButton.MouseButton1Click:Connect(function()
    targetPart = "Head"
    targetPartLabel.Text = "Target Part: " .. targetPart
end)

torsoButton.MouseButton1Click:Connect(function()
    targetPart = "UpperTorso"
    targetPartLabel.Text = "Target Part: " .. targetPart
end)

humanoidRootPartButton.MouseButton1Click:Connect(function()
    targetPart = "HumanoidRootPart"
    targetPartLabel.Text = "Target Part: " .. targetPart
end)

toggleAutoAimButton.MouseButton1Click:Connect(function()
    isAutoAimEnabled = not isAutoAimEnabled
    autoAimLabel.Text = "Auto Aim: " .. (isAutoAimEnabled and "ON" or "OFF")
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == AIM_KEY then
        isAiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == AIM_KEY then
        isAiming = false
    end
end)

-- Авто-обновление списка игроков
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        updatePlayerList()
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if player == selectedPlayer then
        selectedPlayer = nil
        playerSelectionLabel.Text = "Selected: AUTO"
    end
    updatePlayerList()
end)

-- Основной цикл
RunService.RenderStepped:Connect(function()
    if isAutoAimEnabled or isAiming then
        aimAtTarget()
    end
end)

-- Настройка для мобильных устройств
if UserInputService.TouchEnabled then
    openButton.Size = UDim2.new(0.15, 0, 0.08, 0)
    openButton.Position = UDim2.new(0.02, 0, 0.02, 0)
    openButton.TextSize = 14
    
    frame.Size = UDim2.new(0.9, 0, 0.8, 0)
    frame.Position = UDim2.new(0.05, 0, 0.1, 0)
    
    GuiService:SetGlobalGuiInset(0, 0, 0, 0)
end

-- Инициализация
updatePlayerList()
-- [Предыдущий код остается без изменений до самого конца]

-- Инициализация
updatePlayerList()

-- Настройка для мобильных устройств
if UserInputService.TouchEnabled then
    openButton.Size = UDim2.new(0.15, 0, 0.08, 0)
    openButton.Position = UDim2.new(0.02, 0, 0.02, 0)
    openButton.TextSize = 14
    
    frame.Size = UDim2.new(0.9, 0, 0.8, 0)
    frame.Position = UDim2.new(0.05, 0, 0.1, 0)
    
    GuiService:SetGlobalGuiInset(0, 0, 0, 0)
end

-- Добавляем теги в интерфейс
local tagsLabel = Instance.new("TextLabel")
tagsLabel.Size = UDim2.new(1, 0, 0.05, 0)
tagsLabel.Position = UDim2.new(0, 0, 0.95, 0)
tagsLabel.Text = "cyku_cyku @privet12380"
tagsLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
tagsLabel.BackgroundTransparency = 1
tagsLabel.TextSize = 14
tagsLabel.Font = Enum.Font.SourceSans
tagsLabel.Parent = screenGui
