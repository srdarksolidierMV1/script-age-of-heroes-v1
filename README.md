--[[
    Age of Heroes - age modd v1
    Versão: 1.0.0
    Desenvolvedor: Srdarksolidier
    Descrição: Interface premium com Fast Punch e Player Teleport
--]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Variáveis de configuração
local CONFIG = {
    UI_NAME = "Age of Heroes Premium",
    FAST_PUNCH_ENABLED = false,
    FAST_PUNCH_SPEED = 0.05, -- Velocidade entre socos em segundos
    STAMINA_THRESHOLD = 10, -- Stamina mínima para continuar
    TELEPORT_DISTANCE = 5, -- Distância segura para teleporte
    ANIMATION_SPEED = 0.3, -- Velocidade das animações
}

-- Sistema de Notificações
local NotificationSystem = {}
NotificationSystem.__index = NotificationSystem

function NotificationSystem.new()
    local self = setmetatable({}, NotificationSystem)
    self.activeNotifications = {}
    return self
end

function NotificationSystem:Show(message, duration)
    duration = duration or 2
    
    local screenGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not screenGui then return end
    
    local notification = Instance.new("Frame")
    notification.Name = "Notification"
    notification.Size = UDim2.new(0, 300, 0, 50)
    notification.Position = UDim2.new(0.5, -150, 0, -60)
    notification.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    notification.BackgroundTransparency = 0.1
    notification.BorderSizePixel = 0
    notification.ClipsDescendants = true
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = notification
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(100, 100, 100)
    stroke.Thickness = 1
    stroke.Transparency = 0.5
    stroke.Parent = notification
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -20, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = message
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = notification
    
    notification.Parent = screenGui
    
    -- Animação de entrada
    local enterTween = TweenService:Create(notification, 
        TweenInfo.new(CONFIG.ANIMATION_SPEED, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Position = UDim2.new(0.5, -150, 0, 20)}
    )
    enterTween:Play()
    
    -- Remover após duração
    task.wait(duration)
    
    local exitTween = TweenService:Create(notification, 
        TweenInfo.new(CONFIG.ANIMATION_SPEED, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Position = UDim2.new(0.5, -150, 0, -60)}
    )
    exitTween:Play()
    exitTween.Completed:Connect(function()
        notification:Destroy()
    end)
end

-- Criar UI Principal
local function createMainUI()
    -- ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AgeOfHeroesUI"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Frame Principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 500, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -250, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Corner Principal
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 16)
    mainCorner.Parent = mainFrame
    
    -- Sombra
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 20, 1, 20)
    shadow.Position = UDim2.new(0, -10, 0, -10)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6014261993"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.ImageTransparency = 0.6
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(49, 49, 450, 450)
    shadow.ZIndex = -1
    shadow.Parent = mainFrame
    
    -- Barra de Título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 45)
    titleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 16)
    titleCorner.Parent = titleBar
    
    -- Corrigir cantos superiores
    local titleBottom = Instance.new("Frame")
    titleBottom.Size = UDim2.new(1, 0, 0.5, 0)
    titleBottom.Position = UDim2.new(0, 0, 0.5, 0)
    titleBottom.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    titleBottom.BorderSizePixel = 0
    titleBottom.Parent = titleBar
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0.6, 0, 1, 0)
    titleLabel.Position = UDim2.new(0, 20, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = CONFIG.UI_NAME
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 16
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Botão Minimizar
    local minimizeButton = Instance.new("TextButton")
    minimizeButton.Name = "MinimizeButton"
    minimizeButton.Size = UDim2.new(0, 30, 0, 30)
    minimizeButton.Position = UDim2.new(1, -75, 0, 7)
    minimizeButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    minimizeButton.Text = "—"
    minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeButton.Font = Enum.Font.GothamBold
    minimizeButton.TextSize = 18
    minimizeButton.BorderSizePixel = 0
    minimizeButton.Parent = titleBar
    
    local minCorner = Instance.new("UICorner")
    minCorner.CornerRadius = UDim.new(0, 8)
    minCorner.Parent = minimizeButton
    
    -- Botão Fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -35, 0, 7)
    closeButton.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
    closeButton.Text = "✕"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 18
    closeButton.BorderSizePixel = 0
    closeButton.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeButton
    
    -- Container de Conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "ContentFrame"
    contentFrame.Size = UDim2.new(1, 0, 1, -45)
    contentFrame.Position = UDim2.new(0, 0, 0, 45)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Barra de Navegação
    local navBar = Instance.new("Frame")
    navBar.Name = "NavBar"
    navBar.Size = UDim2.new(0, 150, 1, 0)
    navBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    navBar.BorderSizePixel = 0
    navBar.Parent = contentFrame
    
    local navCorner = Instance.new("UICorner")
    navCorner.CornerRadius = UDim.new(0, 16)
    navCorner.Parent = navBar
    
    -- Corrigir cantos da navbar
    local navRight = Instance.new("Frame")
    navRight.Size = UDim2.new(0.5, 0, 1, 0)
    navRight.Position = UDim2.new(0.5, 0, 0, 0)
    navRight.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    navRight.BorderSizePixel = 0
    navRight.Parent = navBar
    
    -- Área de Conteúdo Principal
    local mainContent = Instance.new("Frame")
    mainContent.Name = "MainContent"
    mainContent.Size = UDim2.new(1, -160, 1, -10)
    mainContent.Position = UDim2.new(0, 155, 0, 5)
    mainContent.BackgroundTransparency = 1
    mainContent.Parent = contentFrame
    
    -- Botões de Navegação
    local function createNavButton(text, position)
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, -20, 0, 40)
        button.Position = UDim2.new(0, 10, 0, position)
        button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        button.Text = text
        button.TextColor3 = Color3.fromRGB(200, 200, 200)
        button.Font = Enum.Font.GothamMedium
        button.TextSize = 14
        button.BorderSizePixel = 0
        button.Parent = navBar
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = button
        
        return button
    end
    
    -- Páginas
    local pages = {}
    
    -- Página Fast Punch
    local fastPunchPage = Instance.new("Frame")
    fastPunchPage.Name = "FastPunchPage"
    fastPunchPage.Size = UDim2.new(1, 0, 1, 0)
    fastPunchPage.BackgroundTransparency = 1
    fastPunchPage.Visible = true
    fastPunchPage.Parent = mainContent
    pages["FastPunch"] = fastPunchPage
    
    -- Título da página
    local fpTitle = Instance.new("TextLabel")
    fpTitle.Size = UDim2.new(1, 0, 0, 30)
    fpTitle.Position = UDim2.new(0, 0, 0, 10)
    fpTitle.BackgroundTransparency = 1
    fpTitle.Text = "⚡ Fast Punch System"
    fpTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    fpTitle.Font = Enum.Font.GothamBold
    fpTitle.TextSize = 18
    fpTitle.TextXAlignment = Enum.TextXAlignment.Center
    fpTitle.Parent = fastPunchPage
    
    -- Descrição
    local fpDesc = Instance.new("TextLabel")
    fpDesc.Size = UDim2.new(1, -20, 0, 40)
    fpDesc.Position = UDim2.new(0, 10, 0, 50)
    fpDesc.BackgroundTransparency = 1
    fpDesc.Text = "Ataque rápido como uma metralhadora! Remove o cooldown entre socos."
    fpDesc.TextColor3 = Color3.fromRGB(150, 150, 150)
    fpDesc.Font = Enum.Font.GothamMedium
    fpDesc.TextSize = 12
    fpDesc.TextWrapped = true
    fpDesc.TextXAlignment = Enum.TextXAlignment.Center
    fpDesc.Parent = fastPunchPage
    
    -- Botão ON/OFF
    local fpToggle = Instance.new("TextButton")
    fpToggle.Name = "FastPunchToggle"
    fpToggle.Size = UDim2.new(0.4, 0, 0, 50)
    fpToggle.Position = UDim2.new(0.3, 0, 0, 110)
    fpToggle.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
    fpToggle.Text = "OFF"
    fpToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    fpToggle.Font = Enum.Font.GothamBold
    fpToggle.TextSize = 20
    fpToggle.BorderSizePixel = 0
    fpToggle.Parent = fastPunchPage
    
    local fpToggleCorner = Instance.new("UICorner")
    fpToggleCorner.CornerRadius = UDim.new(0, 12)
    fpToggleCorner.Parent = fpToggle
    
    -- Status da Stamina
    local staminaLabel = Instance.new("TextLabel")
    staminaLabel.Size = UDim2.new(1, -20, 0, 30)
    staminaLabel.Position = UDim2.new(0, 10, 0, 180)
    staminaLabel.BackgroundTransparency = 1
    staminaLabel.Text = "Stamina: --"
    staminaLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    staminaLabel.Font = Enum.Font.GothamMedium
    staminaLabel.TextSize = 14
    staminaLabel.TextXAlignment = Enum.TextXAlignment.Center
    staminaLabel.Parent = fastPunchPage
    
    -- Página Player Teleport
    local teleportPage = Instance.new("Frame")
    teleportPage.Name = "TeleportPage"
    teleportPage.Size = UDim2.new(1, 0, 1, 0)
    teleportPage.BackgroundTransparency = 1
    teleportPage.Visible = false
    teleportPage.Parent = mainContent
    pages["Teleport"] = teleportPage
    
    -- Título da página
    local tpTitle = Instance.new("TextLabel")
    tpTitle.Size = UDim2.new(1, 0, 0, 30)
    tpTitle.Position = UDim2.new(0, 0, 0, 10)
    tpTitle.BackgroundTransparency = 1
    tpTitle.Text = "🌍 Player Teleport"
    tpTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    tpTitle.Font = Enum.Font.GothamBold
    tpTitle.TextSize = 18
    tpTitle.TextXAlignment = Enum.TextXAlignment.Center
    tpTitle.Parent = teleportPage
    
    -- Lista de Jogadores
    local playerList = Instance.new("ScrollingFrame")
    playerList.Name = "PlayerList"
    playerList.Size = UDim2.new(1, -20, 1, -60)
    playerList.Position = UDim2.new(0, 10, 0, 50)
    playerList.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    playerList.BorderSizePixel = 0
    playerList.ScrollBarThickness = 6
    playerList.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
    playerList.Parent = teleportPage
    
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 12)
    listCorner.Parent = playerList
    
    local listLayout = Instance.new("UIListLayout")
    listLayout.Padding = UDim.new(0, 5)
    listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    listLayout.SortOrder = Enum.SortOrder.LayoutOrder
    listLayout.Parent = playerList
    
    -- Botões de Navegação
    local fastPunchBtn = createNavButton("⚡ Fast Punch", 20)
    local teleportBtn = createNavButton("🌍 Teleport", 70)
    
    -- Sistema de arrastar
    local dragging = false
    local dragStart = nil
    local startPos = nil
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or 
           input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(
                startPos.X.Scale, 
                startPos.X.Offset + delta.X,
                startPos.Y.Scale, 
                startPos.Y.Offset + delta.Y
            )
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    -- Navegação entre páginas
    local function switchPage(pageName)
        for name, page in pairs(pages) do
            page.Visible = (name == pageName)
        end
        
        -- Destacar botão ativo
        if pageName == "FastPunch" then
            fastPunchBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
            teleportBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        else
            fastPunchBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            teleportBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        end
    end
    
    fastPunchBtn.MouseButton1Click:Connect(function()
        switchPage("FastPunch")
    end)
    
    teleportBtn.MouseButton1Click:Connect(function()
        switchPage("Teleport")
    end)
    
    -- Minimizar/Restaurar
    local isMinimized = false
    local originalSize = mainFrame.Size
    
    minimizeButton.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        
        if isMinimized then
            local tween = TweenService:Create(mainFrame, 
                TweenInfo.new(CONFIG.ANIMATION_SPEED, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Size = UDim2.new(0, 500, 0, 45)}
            )
            tween:Play()
            minimizeButton.Text = "□"
        else
            local tween = TweenService:Create(mainFrame, 
                TweenInfo.new(CONFIG.ANIMATION_SPEED, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Size = originalSize}
            )
            tween:Play()
            minimizeButton.Text = "—"
        end
    end)
    
    -- Fechar
    closeButton.MouseButton1Click:Connect(function()
        local tween = TweenService:Create(mainFrame, 
            TweenInfo.new(CONFIG.ANIMATION_SPEED, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
            {Size = UDim2.new(0, 0, 0, 0)}
        )
        tween:Play()
        tween.Completed:Connect(function()
            screenGui:Destroy()
        end)
    end)
    
    return {
        screenGui = screenGui,
        mainFrame = mainFrame,
        fpToggle = fpToggle,
        staminaLabel = staminaLabel,
        playerList = playerList,
        notificationSystem = NotificationSystem.new(),
        pages = pages
    }
end

-- Sistema Fast Punch
local function setupFastPunch(ui)
    local connection = nil
    
    local function getStamina()
        -- Tenta obter a stamina do personagem
        local character = LocalPlayer.Character
        if not character then return 0 end
        
        -- Procura por um atributo ou valor de stamina
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid then return 0 end
        
        -- Verifica se existe um sistema de stamina
        local stamina = character:GetAttribute("Stamina")
        if stamina then
            return stamina
        end
        
        -- Fallback: usa a saúde como indicador
        return humanoid.Health
    end
    
    local function fastPunchLoop()
        while CONFIG.FAST_PUNCH_ENABLED do
            local stamina = getStamina()
            
            -- Atualiza o label de stamina
            ui.staminaLabel.Text = "Stamina: " .. math.floor(stamina)
            
            -- Verifica se tem stamina suficiente
            if stamina < CONFIG.STAMINA_THRESHOLD then
                ui.staminaLabel.TextColor3 = Color3.fromRGB(231, 76, 60)
                task.wait(0.5) -- Aguarda um pouco antes de verificar novamente
                continue
            end
            
            ui.staminaLabel.TextColor3 = Color3.fromRGB(46, 204, 113)
            
            -- Simula um soco
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChild("Humanoid")
                if humanoid then
                    -- Tenta ativar a animação de soco
                    local animator = humanoid:FindFirstChild("Animator")
                    if animator then
                        -- Simula ação de soco (depende da implementação do jogo)
                        -- Aqui você pode adicionar a lógica específica do Age of Heroes
                        pcall(function()
                            -- Exemplo: dispara evento remoto ou usa ferramenta
                            local tool = character:FindFirstChildOfClass("Tool")
                            if tool and tool:FindFirstChild("RemoteEvent") then
                                tool.RemoteEvent:FireServer()
                            end
                        end)
                    end
                end
            end
            
            task.wait(CONFIG.FAST_PUNCH_SPEED)
        end
    end
    
    ui.fpToggle.MouseButton1Click:Connect(function()
        CONFIG.FAST_PUNCH_ENABLED = not CONFIG.FAST_PUNCH_ENABLED
        
        if CONFIG.FAST_PUNCH_ENABLED then
            ui.fpToggle.Text = "ON"
            ui.fpToggle.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
            ui.notificationSystem:Show("⚡ Fast Punch ATIVADO!")
            
            -- Inicia o loop
            task.spawn(fastPunchLoop)
        else
            ui.fpToggle.Text = "OFF"
            ui.fpToggle.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
            ui.notificationSystem:Show("⚡ Fast Punch DESATIVADO!")
            ui.staminaLabel.Text = "Stamina: --"
            ui.staminaLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
    end)
end

-- Sistema de Teleporte
local function setupTeleportSystem(ui)
    local function updatePlayerList()
        -- Limpa a lista atual
        for _, child in pairs(ui.playerList:GetChildren()) do