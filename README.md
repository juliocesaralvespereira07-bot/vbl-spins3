--[
    Script: Volleyball Legends Hub - Lucky Spin & Coins V3
    Versão: 3.0.0
    Foco: Farm de Lucky Spins e Coins (UPDATE 78)
    Sem sistema de key
    Compatível com SolaraV3
]]

-- =============================================
-- SERVIÇOS
-- =============================================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local VirtualUser = game:GetService("VirtualUser")
local TeleportService = game:GetService("TeleportService")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- =============================================
-- CONFIGURAÇÕES
-- =============================================
local Config = {
    Theme = {
        Primary = Color3.fromRGB(122, 0, 0),
        Secondary = Color3.fromRGB(15, 15, 15),
        Accent = Color3.fromRGB(255, 50, 50),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(200, 200, 200),
    }
}

-- =============================================
-- VARIÁVEIS GLOBAIS
-- =============================================
local Hub = {}
local farmRunning = false
local farmDelay = 2
local antiAFKRunning = false
local currentTab = "Farm"
local coinsCollected = 0
local spinsCompleted = 0

-- =============================================
-- FUNÇÕES DE NOTIFICAÇÃO
-- =============================================
function Hub:Notify(text, duration)
    duration = duration or 3
    
    local notification = Instance.new("Frame")
    notification.Size = UDim2.new(0, 350, 0, 50)
    notification.Position = UDim2.new(0.5, 0, 0.5, 100)
    notification.AnchorPoint = Vector2.new(0.5, 0.5)
    notification.BackgroundColor3 = Config.Theme.Secondary
    notification.BackgroundTransparency = 1
    notification.BorderSizePixel = 0
    notification.Parent = CoreGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = notification
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Config.Theme.Primary
    stroke.Thickness = 1
    stroke.Parent = notification
    
    local label = Instance.new("TextLabel")
    label.Text = text
    label.TextColor3 = Config.Theme.Text
    label.TextSize = 14
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Font = Enum.Font.Gotham
    label.Parent = notification
    
    TweenService:Create(notification, TweenInfo.new(0.5), {
        Position = UDim2.new(0.5, 0, 0.5, 0),
        BackgroundTransparency = 0
    }):Play()
    
    task.wait(duration)
    
    TweenService:Create(notification, TweenInfo.new(0.5), {
        Position = UDim2.new(0.5, 0, 0.5, -100),
        BackgroundTransparency = 1
    }):Play()
    
    task.wait(0.5)
    notification:Destroy()
end

-- =============================================
-- FUNÇÕES DE BUSCA DO JOGO
-- =============================================

-- Função para encontrar o botão de Lucky Spin
function Hub:FindLuckySpinButton()
    local found = nil
    
    -- Procurar em toda a interface do jogo
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("TextButton") and v.Visible and v.Active then
            local text = v.Text or ""
            local name = v.Name or ""
            
            -- Verificar se é um botão de lucky spin
            if text:lower():find("lucky") or 
               text:lower():find("spin") or
               text:lower():find("girar") or
               text:lower():find("roda") or
               text:lower():find("sorte") or
               name:lower():find("spin") or
               name:lower():find("lucky") then
                
                -- Verificar se o botão está em um frame de lucky spin
                local parent = v.Parent
                if parent and parent:IsA("Frame") then
                    local parentName = parent.Name or ""
                    if parentName:lower():find("lucky") or 
                       parentName:lower():find("spin") or
                       parentName:lower():find("wheel") then
                        found = v
                        break
                    end
                end
            end
        end
        
        -- Procurar ImageButton também
        if not found and v:IsA("ImageButton") and v.Visible and v.Active then
            local name = v.Name or ""
            if name:lower():find("spin") or 
               name:lower():find("lucky") or
               name:lower():find("wheel") then
                found = v
                break
            end
        end
    end
    
    return found
end

-- Função para encontrar botão de coletar moedas
function Hub:FindCoinButton()
    local found = nil
    
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("TextButton") and v.Visible and v.Active then
            local text = v.Text or ""
            local name = v.Name or ""
            
            if text:lower():find("collect") or 
               text:lower():find("claim") or
               text:lower():find("coletar") or
               text:lower():find("receive") or
               text:lower():find("pegar") or
               name:lower():find("collect") or
               name:lower():find("claim") then
                found = v
                break
            end
        end
        
        if not found and v:IsA("ImageButton") and v.Visible and v.Active then
            local name = v.Name or ""
            if name:lower():find("collect") or 
               name:lower():find("claim") then
                found = v
                break
            end
        end
    end
    
    return found
end

-- Função para verificar se a roda de sorte está disponível
function Hub:IsWheelAvailable()
    -- Procurar por indicadores da roda
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("Frame") and v.Visible then
            local name = v.Name or ""
            if name:lower():find("wheel") or 
               name:lower():find("roda") or
               name:lower():find("spin") then
                -- Verificar se tem filhos visíveis
                for _, child in ipairs(v:GetChildren()) do
                    if child:IsA("TextButton") or child:IsA("ImageButton") then
                        if child.Visible and child.Active then
                            return true
                        end
                    end
                end
            end
        end
    end
    return false
end

-- =============================================
-- FUNÇÕES DO FARM
-- =============================================

function Hub:StartAutoFarm()
    farmRunning = true
    spinsCompleted = 0
    coinsCollected = 0
    
    Hub:Notify("🔄 Auto Farm iniciado!", 2)
    
    task.spawn(function()
        while farmRunning do
            task.wait(farmDelay)
            
            local success = pcall(function()
                -- 1. Primeiro, tentar coletar moedas
                local coinBtn = Hub:FindCoinButton()
                if coinBtn then
                    coinBtn:Click()
                    coinsCollected = coinsCollected + 1
                    Hub:Notify("💰 Moeda coletada! Total: " .. coinsCollected, 1.5)
                    task.wait(0.5)
                end
                
                -- 2. Verificar se a roda está disponível
                if Hub:IsWheelAvailable() then
                    -- 3. Encontrar e clicar no botão de lucky spin
                    local spinBtn = Hub:FindLuckySpinButton()
                    if spinBtn then
                        spinBtn:Click()
                        spinsCompleted = spinsCompleted + 1
                        Hub:Notify("🌀 Lucky Spin realizado! Total: " .. spinsCompleted, 1.5)
                        task.wait(1)
                    else
                        -- Tentar alternativas
                        local success2 = pcall(function()
                            -- Procurar por botões mais específicos
                            for _, v in ipairs(game:GetDescendants()) do
                                if v:IsA("TextButton") and v.Visible and v.Active then
                                    local text = v.Text or ""
                                    if text:find("SPIN") or text:find("GIRAR") then
                                        v:Click()
                                        spinsCompleted = spinsCompleted + 1
                                        Hub:Notify("🌀 Lucky Spin realizado! Total: " .. spinsCompleted, 1.5)
                                        task.wait(1)
                                        break
                                    end
                                end
                            end
                        end)
                    end
                end
            end)
            
            if not success then
                -- Se falhou, tentar novamente após um delay maior
                task.wait(2)
            end
        end
    end)
end

function Hub:StopAutoFarm()
    farmRunning = false
    Hub:Notify("⏹️ Auto Farm parado!", 2)
end

-- Anti-AFK
function Hub:StartAntiAFK()
    antiAFKRunning = true
    
    task.spawn(function()
        while antiAFKRunning do
            task.wait(60)
            
            pcall(function()
                -- Método 1: VirtualUser
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
            
            pcall(function()
                -- Método 2: Movimento do mouse
                local mouse = LocalPlayer:GetMouse()
                if mouse then
                    mouse.Move(Vector2.new(math.random(1, 100), math.random(1, 100)))
                end
            end)
            
            pcall(function()
                -- Método 3: Simular tecla
                local key = Enum.KeyCode[{"A", "W", "D", "S"}[math.random(1, 4)]]
                if key then
                    game:GetService("VirtualInputManager"):SendKeyEvent(true, key, false, game)
                    task.wait(0.1)
                    game:GetService("VirtualInputManager"):SendKeyEvent(false, key, false, game)
                end
            end)
        end
    end)
end

function Hub:StopAntiAFK()
    antiAFKRunning = false
end

-- =============================================
-- CRIAÇÃO DA INTERFACE
-- =============================================
function Hub:CreateInterface()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "VolleyballHub"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = CoreGui
    
    -- Janela Principal
    local mainWindow = Instance.new("Frame")
    mainWindow.Name = "MainWindow"
    mainWindow.Size = UDim2.new(0, 500, 0, 450)
    mainWindow.Position = UDim2.new(0.5, 0, 0.5, 0)
    mainWindow.AnchorPoint = Vector2.new(0.5, 0.5)
    mainWindow.BackgroundColor3 = Config.Theme.Secondary
    mainWindow.BorderSizePixel = 0
    mainWindow.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = mainWindow
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Config.Theme.Primary
    stroke.Thickness = 1
    stroke.Parent = mainWindow
    
    -- Barra de Título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 35)
    titleBar.BackgroundTransparency = 1
    titleBar.Parent = mainWindow
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Text = "🏐 Volleyball Hub V3"
    titleLabel.TextColor3 = Config.Theme.Text
    titleLabel.TextSize = 18
    titleLabel.BackgroundTransparency = 1
    titleLabel.Size = UDim2.new(0.6, 0, 1, 0)
    titleLabel.Position = UDim2.new(0.02, 0, 0, 0)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Botão Fechar
    local closeBtn = Instance.new("TextButton")
    closeBtn.Name = "CloseBtn"
    closeBtn.Size = UDim2.new(0, 30, 0, 25)
    closeBtn.Position = UDim2.new(0.95, 0, 0.5, 0)
    closeBtn.AnchorPoint = Vector2.new(0, 0.5)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Config.Theme.Text
    closeBtn.TextSize = 16
    closeBtn.BackgroundColor3 = Config.Theme.Primary
    closeBtn.BorderSizePixel = 0
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 5)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        mainWindow.Visible = not mainWindow.Visible
    end)
    
    -- Sistema de Arrastar
    local dragging = false
    local dragStart = nil
    local startPos = nil
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainWindow.Position
        end
    end)
    
    titleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
            local delta = input.Position - dragStart
            mainWindow.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
    
    titleBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    -- Área de Conteúdo
    local contentArea = Instance.new("Frame")
    contentArea.Name = "ContentArea"
    contentArea.Size = UDim2.new(1, -20, 1, -45)
    contentArea.Position = UDim2.new(0.5, 0, 0.5, 0)
    contentArea.AnchorPoint = Vector2.new(0.5, 0.5)
    contentArea.BackgroundTransparency = 1
    contentArea.Parent = mainWindow
    
    -- Barra de Abas
    local tabBar = Instance.new("Frame")
    tabBar.Name = "TabBar"
    tabBar.Size = UDim2.new(1, 0, 0, 35)
    tabBar.Position = UDim2.new(0, 0, 0, 0)
    tabBar.BackgroundTransparency = 1
    tabBar.Parent = contentArea
    
    local tabs = {
        {name = "Farm", icon = "🎯"},
        {name = "Stats", icon = "📊"},
        {name = "Settings", icon = "⚙️"},
        {name = "Credits", icon = "ℹ️"}
    }
    
    local tabButtons = {}
    
    for i, tab in ipairs(tabs) do
        local btn = Instance.new("TextButton")
        btn.Name = "Tab_" .. tab.name
        btn.Size = UDim2.new(0.25, -2, 1, -5)
        btn.Position = UDim2.new((i-1) * 0.25 + 0.005, 0, 0.5, 0)
        btn.AnchorPoint = Vector2.new(0, 0.5)
        btn.Text = tab.icon .. " " .. tab.name
        btn.TextColor3 = Config.Theme.Text
        btn.TextSize = 13
        btn.BackgroundColor3 = i == 1 and Config.Theme.Primary or Config.Theme.Secondary
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        btn.Parent = tabBar
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 5)
        btnCorner.Parent = btn
        
        tabButtons[tab.name] = btn
        
        btn.MouseButton1Click:Connect(function()
            Hub:SwitchTab(tab.name, tabButtons, contentArea)
        end)
    end
    
    -- Criar conteúdo das abas
    local farmTab = self:CreateFarmTab(contentArea)
    local statsTab = self:CreateStatsTab(contentArea)
    local settingsTab = self:CreateSettingsTab(contentArea)
    local creditsTab = self:CreateCreditsTab(contentArea)
    
    -- Mostrar primeira aba
    farmTab.Visible = true
    statsTab.Visible = false
    settingsTab.Visible = false
    creditsTab.Visible = false
    
    return screenGui
end

-- =============================================
-- FUNÇÃO PARA ALTERNAR ABAS
-- =============================================
function Hub:SwitchTab(tabName, tabButtons, contentArea)
    currentTab = tabName
    
    for name, btn in pairs(tabButtons) do
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = name == tabName and Config.Theme.Primary or Config.Theme.Secondary
        }):Play()
    end
    
    for _, child in ipairs(contentArea:GetChildren()) do
        if child.Name:find("Tab_") then
            child.Visible = child.Name == "Tab_" .. tabName
        end
    end
end

-- =============================================
-- CRIAR ABA FARM
-- =============================================
function Hub:CreateFarmTab(parent)
    local tab = Instance.new("Frame")
    tab.Name = "Tab_Farm"
    tab.Size = UDim2.new(1, 0, 1, -35)
    tab.Position = UDim2.new(0, 0, 0, 35)
    tab.BackgroundTransparency = 1
    tab.Visible = false
    tab.Parent = parent
    
    local y = 10
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Text = "🎯 Sistema de Farm"
    title.TextColor3 = Config.Theme.Text
    title.TextSize = 20
    title.BackgroundTransparency = 1
    title.Size = UDim2.new(1, 0, 0, 35)
    title.Position = UDim2.new(0.5, 0, 0, y)
    title.AnchorPoint = Vector2.new(0.5, 0)
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Center
    title.Parent = tab
    y = y + 40
    
    -- Auto Farm
    local farmFrame = Instance.new("Frame")
    farmFrame.Size = UDim2.new(0.9, 0, 0, 35)
    farmFrame.Position = UDim2.new(0.05, 0, 0, y)
    farmFrame.BackgroundTransparency = 1
    farmFrame.Parent = tab
    
    local farmLabel = Instance.new("TextLabel")
    farmLabel.Text = "🌀 Auto Farm (Lucky Spins + Coins)"
    farmLabel.TextColor3 = Config.Theme.Text
    farmLabel.TextSize = 14
    farmLabel.BackgroundTransparency = 1
    farmLabel.Size = UDim2.new(0.7, 0, 1, 0)
    farmLabel.Font = Enum.Font.Gotham
    farmLabel.TextXAlignment = Enum.TextXAlignment.Left
    farmLabel.Parent = farmFrame
    
    local farmToggle = Instance.new("Frame")
    farmToggle.Size = UDim2.new(0, 40, 0, 20)
    farmToggle.Position = UDim2.new(1, -45, 0.5, 0)
    farmToggle.AnchorPoint = Vector2.new(0, 0.5)
    farmToggle.BackgroundColor3 = Config.Theme.Secondary
    farmToggle.BorderSizePixel = 0
    farmToggle.Parent = farmFrame
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = farmToggle
    
    local farmIndicator = Instance.new("Frame")
    farmIndicator.Size = UDim2.new(0, 16, 0, 16)
    farmIndicator.Position = UDim2.new(0, 2, 0.5, 0)
    farmIndicator.AnchorPoint = Vector2.new(0, 0.5)
    farmIndicator.BackgroundColor3 = Config.Theme.Text
    farmIndicator.BorderSizePixel = 0
    farmIndicator.Parent = farmToggle
    
    local indicatorCorner = Instance.new("UICorner")
    indicatorCorner.CornerRadius = UDim.new(1, 0)
    indicatorCorner.Parent = farmIndicator
    
    local farmState = false
    
    farmToggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            farmState = not farmState
            local targetPos = farmState and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
            local targetColor = farmState and Config.Theme.Primary or Config.Theme.Secondary
            
            TweenService:Create(farmToggle, TweenInfo.new(0.3), {
                BackgroundColor3 = targetColor
            }):Play()
            
            TweenService:Create(farmIndicator, TweenInfo.new(0.3), {
                Position = targetPos
            }):Play()
            
            if farmState then
                Hub:StartAutoFarm()
            else
                Hub:StopAutoFarm()
            end
        end
    end)
    y = y + 45
    
    -- Delay
    local delayLabel = Instance.new("TextLabel")
    delayLabel.Text = "⏱️ Delay entre ações (segundos)"
    delayLabel.TextColor3 = Config.Theme.Text
    delayLabel.TextSize = 14
    delayLabel.BackgroundTransparency = 1
    delayLabel.Size = UDim2.new(0.9, 0, 0, 25)
    delayLabel.Position = UDim2.new(0.05, 0, 0, y)
    delayLabel.Font = Enum.Font.Gotham
    delayLabel.TextXAlignment = Enum.TextXAlignment.Left
    delayLabel.Parent = tab
    y = y + 25
    
    local delayValue = 2
    local delayDisplay = Instance.new("TextLabel")
    delayDisplay.Text = "2s"
    delayDisplay.TextColor3 = Config.Theme.Text
    delayDisplay.TextSize = 14
    delayDisplay.BackgroundTransparency = 1
    delayDisplay.Size = UDim2.new(0.1, 0, 0, 25)
    delayDisplay.Position = UDim2.new(0.85, 0, 0, y - 25)
    delayDisplay.Font = Enum.Font.Gotham
    delayDisplay.TextXAlignment = Enum.TextXAlignment.Right
    delayDisplay.Parent = tab
    
    local delaySlider = Instance.new("Frame")
    delaySlider.Size = UDim2.new(0.8, 0, 0, 4)
    delaySlider.Position = UDim2.new(0.05, 0, 0, y)
    delaySlider.BackgroundColor3 = Config.Theme.Secondary
    delaySlider.BorderSizePixel = 0
    delaySlider.Parent = tab
    
    local sliderCorner = Instance.new("UICorner")
    sliderCorner.CornerRadius = UDim.new(1, 0)
    sliderCorner.Parent = delaySlider
    
    local delayFill = Instance.new("Frame")
    delayFill.Size = UDim2.new(0.1, 0, 1, 0)
    delayFill.BackgroundColor3 = Config.Theme.Primary
    delayFill.BorderSizePixel = 0
    delayFill.Parent = delaySlider
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = delayFill
    
    local delayBtn = Instance.new("Frame")
    delayBtn.Size = UDim2.new(0, 16, 0, 16)
    delayBtn.Position = UDim2.new(0.1, -8, 0.5, 0)
    delayBtn.AnchorPoint = Vector2.new(0, 0.5)
    delayBtn.BackgroundColor3 = Config.Theme.Text
    delayBtn.BorderSizePixel = 0
    delayBtn.Parent = delaySlider
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(1, 0)
    btnCorner.Parent = delayBtn
    
    local function UpdateDelay(value)
        delayValue = math.clamp(value, 1, 10)
        local percent = (delayValue - 1) / 9
        delayFill.Size = UDim2.new(percent, 0, 1, 0)
        delayBtn.Position = UDim2.new(percent, -8, 0.5, 0)
        delayDisplay.Text = math.round(delayValue) .. "s"
        farmDelay = math.round(delayValue)
    end
    
    local isDragging = false
    
    delayBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = true
        end
    end)
    
    delayBtn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = false
        end
    end)
    
    delaySlider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local mousePos = input.Position.X
            local framePos = delaySlider.AbsolutePosition.X
            local frameWidth = delaySlider.AbsoluteSize.X
            local percent = math.clamp((mousePos - framePos) / frameWidth, 0, 1)
            UpdateDelay(1 + percent * 9)
        end
    end)
    
    delaySlider.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and isDragging then
            local mousePos = input.Position.X
            local framePos = delaySlider.AbsolutePosition.X
            local frameWidth = delaySlider.AbsoluteSize.X
            local percent = math.clamp((mousePos - framePos) / frameWidth, 0, 1)
            UpdateDelay(1 + percent * 9)
        end
    end)
    y = y + 30
    
    -- Anti-AFK
    local afkFrame = Instance.new("Frame")
    afkFrame.Size = UDim2.new(0.9, 0, 0, 35)
    afkFrame.Position = UDim2.new(0.05, 0, 0, y)
    afkFrame.BackgroundTransparency = 1
    afkFrame.Parent = tab
    
    local afkLabel = Instance.new("TextLabel")
    afkLabel.Text = "🛡️ Anti-AFK"
    afkLabel.TextColor3 = Config.Theme.Text
    afkLabel.TextSize = 14
    afkLabel.BackgroundTransparency = 1
    afkLabel.Size = UDim2.new(0.7, 0, 1, 0)
    afkLabel.Font = Enum.Font.Gotham
    afkLabel.TextXAlignment = Enum.TextXAlignment.Left
    afkLabel.Parent = afkFrame
    
    local afkToggle = Instance.new("Frame")
    afkToggle.Size = UDim2.new(0, 40, 0, 20)
    afkToggle.Position = UDim2.new(1, -45, 0.5, 0)
    afkToggle.AnchorPoint = Vector2.new(0, 0.5)
    afkToggle.BackgroundColor3 = Config.Theme.Secondary
    afkToggle.BorderSizePixel = 0
    afkToggle.Parent = afkFrame
    
    local afkCorner = Instance.new("UICorner")
    afkCorner.CornerRadius = UDim.new(1, 0)
    afkCorner.Parent = afkToggle
    
    local afkIndicator = Instance.new("Frame")
    afkIndicator.Size = UDim2.new(0, 16, 0, 16)
    afkIndicator.Position = UDim2.new(0, 2, 0.5, 0)
    afkIndicator.AnchorPoint = Vector2.new(0, 0.5)
    afkIndicator.BackgroundColor3 = Config.Theme.Text
    afkIndicator.BorderSizePixel = 0
    afkIndicator.Parent = afkToggle
    
    local afkIndicatorCorner = Instance.new("UICorner")
    afkIndicatorCorner.CornerRadius = UDim.new(1, 0)
    afkIndicatorCorner.Parent = afkIndicator
    
    local afkState = false
    
    afkToggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            afkState = not afkState
            local targetPos = afkState and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
            local targetColor = afkState and Config.Theme.Primary or Config.Theme.Secondary
            
            TweenService:Create(afkToggle, TweenInfo.new(0.3), {
                BackgroundColor3 = targetColor
            }):Play()
            
            TweenService:Create(afkIndicator, TweenInfo.new(0.3), {
                Position = targetPos
            }):Play()
            
            if afkState then
                Hub:StartAntiAFK()
                Hub:Notify("🛡️ Anti-AFK ativado!")
            else
                Hub:StopAntiAFK()
                Hub:Notify("⏹️ Anti-AFK desativado!")
            end
        end
    end)
    y = y + 45
    
    -- Botão Rejoin
    local rejoinBtn = Instance.new("TextButton")
    rejoinBtn.Size = UDim2.new(0.9, 0, 0, 35)
    rejoinBtn.Position = UDim2.new(0.05, 0, 0, y)
    rejoinBtn.Text = "🔄 Rejoin Server"
    rejoinBtn.TextColor3 = Config.Theme.Text
    rejoinBtn.TextSize = 14
    rejoinBtn.BackgroundColor3 = Config.Theme.Primary
    rejoinBtn.BorderSizePixel = 0
    rejoinBtn.Font = Enum.Font.GothamBold
    rejoinBtn.Parent = tab
    
    local rejoinCorner = Instance.new("UICorner")
    rejoinCorner.CornerRadius = UDim.new(0, 5)
    rejoinCorner.Parent = rejoinBtn
    
    rejoinBtn.MouseButton1Click:Connect(function()
        Hub:Notify("🔄 Reconectando...")
        task.wait(1)
        TeleportService:Teleport(game.PlaceId)
    end)
    
    return tab
end

-- =============================================
-- CRIAR ABA STATS
-- =============================================
function Hub:CreateStatsTab(parent)
    local tab = Instance.new("Frame")
    tab.Name = "Tab_Stats"
    tab.Size = UDim2.new(1, 0, 1, -35)
    tab.Position = UDim2.new(0, 0, 0, 35)
    tab.BackgroundTransparency = 1
    tab.Visible = false
    tab.Parent = parent
    
    local y = 30
    
    local title = Instance.new("TextLabel")
    title.Text = "📊 Estatísticas"
    title.TextColor3 = Config.Theme.Text
    title.TextSize = 20
    title.BackgroundTransparency = 1
    title.Size = UDim2.new(1, 0, 0, 35)
    title.Position = UDim2.new(0.5, 0, 0, y)
    title.AnchorPoint = Vector2.new(0.5, 0)
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Center
    title.Parent = tab
    y = y + 45
    
    -- Spins realizados
    local spinsLabel = Instance.new("TextLabel")
    spinsLabel.Text = "🔄 Spins realizados: 0"
    spinsLabel.TextColor3 = Config.Theme.Text
    spinsLabel.TextSize = 16
    spinsLabel.BackgroundTransparency = 1
    spinsLabel.Size = UDim2.new(0.9, 0, 0, 30)
    spinsLabel.Position = UDim2.new(0.05, 0, 0, y)
    spinsLabel.Font = Enum.Font.Gotham
    spinsLabel.TextXAlignment = Enum.TextXAlignment.Left
    spinsLabel.Parent = tab
    
    local spinsUpdate = function()
        spinsLabel.Text = "🔄 Spins realizados: " .. spinsCompleted
    end
    
    -- Moedas coletadas
    local coinsLabel = Instance.new("TextLabel")
    coinsLabel.Text = "💰 Moedas coletadas: 0"
    coinsLabel.TextColor3 = Config.Theme.Text
    coinsLabel.TextSize = 16
    coinsLabel.BackgroundTransparency = 1
    coinsLabel.Size = UDim2.new(0.9, 0, 0, 30)
    coinsLabel.Position = UDim2.new(0.05, 0, 0, y + 35)
    coinsLabel.Font = Enum.Font.Gotham
    coinsLabel.TextXAlignment = Enum.TextXAlignment.Left
    coinsLabel.Parent = tab
    
    local coinsUpdate = function()
        coinsLabel.Text = "💰 Moedas coletadas: " .. coinsCollected
    end
    
    -- Atualizar estatísticas periodicamente
    task.spawn(function()
        while true do
            task.wait(2)
            spinsUpdate()
            coinsUpdate()
        end
    end)
    
    -- Botão reset stats
    local resetBtn = Instance.new("TextButton")
    resetBtn.Size = UDim2.new(0.9, 0, 0, 35)
    resetBtn.Position = UDim2.new(0.05, 0, 0, y + 80)
    resetBtn.Text = "🔄 Resetar Estatísticas"
    resetBtn.TextColor3 = Config.Theme.Text
    resetBtn.TextSize = 14
    resetBtn.BackgroundColor3 = Config.Theme.Primary
