local VirtualInputManager = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

-- Criar a interface de forma segura
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UniversalMobileKeyboardSupport"
screenGui.ResetOnSpawn = false

local success, err = pcall(function()
    screenGui.Parent = CoreGui
end)
if not success then
    screenGui.Parent = playerGui
end

-- Tabela para guardar os botões principais e facilitar o abrir/fechar
local gameplayButtons = {}

-- Configurações visuais de animação (Tweens)
local function applyButtonFeedback(button, isPressed)
    local targetColor = isPressed and Color3.fromRGB(0, 210, 255) or Color3.fromRGB(255, 255, 255)
    local targetBgTrans = isPressed and 0.2 or 0.5
    local targetStrokeColor = isPressed and Color3.fromRGB(0, 210, 255) or Color3.fromRGB(255, 255, 255)

    TweenService:Create(button, TweenInfo.new(0.1), {TextColor3 = targetColor, BackgroundTransparency = targetBgTrans}):Play()
    if button:FindFirstChild("UIStroke") then
        TweenService:Create(button.UIStroke, TweenInfo.new(0.1), {Color = targetStrokeColor}):Play()
    end
end

-- Função padrão para simular o pressionar e soltar das teclas
local function setupKeyButton(button, keyCode)
    button.MouseButton1Down:Connect(function()
        VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
        applyButtonFeedback(button, true)
    end)

    button.MouseButton1Up:Connect(function()
        VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
        applyButtonFeedback(button, false)
    end)

    button.MouseLeave:Connect(function()
        VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
        applyButtonFeedback(button, false)
    end)
end

-- Função estilizada para criar os botões (Ajustada para auto-escala)
local function createStyledButton(name, text, position, size, textSize, anchorPoint)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Size = size
    button.Position = position
    if anchorPoint then button.AnchorPoint = anchorPoint end
    
    button.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    button.BackgroundTransparency = 0.5 -- Estilo vidro fumê
    button.Text = text
    button.Font = Enum.Font.GothamBold
    button.TextSize = textSize
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Parent = screenGui
    
    -- Cantos arredondados modernos baseados no tamanho do botão
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 14) -- Mantém as bordas arredondadas perfeitas no celular
    uiCorner.Parent = button
    
    -- Borda fina e elegante
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Thickness = 1.5
    uiStroke.Color = Color3.fromRGB(255, 255, 255)
    uiStroke.Transparency = 0.4
    uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    uiStroke.Parent = button

    table.insert(gameplayButtons, button)
    return button
end

-- --- DESIGN AUTO-AJUSTÁVEL PARA DISPOSITIVOS MÓVEIS ---

-- Tamanhos físicos fixos em pixels (Ideais para dedos no celular, não mudam de formato)
local sizeWASD = UDim2.new(0, 65, 0, 65)
local sizeQE = UDim2.new(0, 52, 0, 52)

-- Botões WASD (Posicionados em porcentagem da tela para ficarem sempre nos cantos inferiores)
local btnA = createStyledButton("BtnA", "A", UDim2.new(0.05, 0, 0.72, 0), sizeWASD, 30, Vector2.new(0, 0.5))
local btnD = createStyledButton("BtnD", "D", UDim2.new(0.05, 80, 0.72, 0), sizeWASD, 30, Vector2.new(0, 0.5)) -- 80 pixels de espaço do botão A

local btnS = createStyledButton("BtnS", "S", UDim2.new(0.95, -80, 0.72, 0), sizeWASD, 30, Vector2.new(1, 0.5)) -- Alinhado à direita
local btnW = createStyledButton("BtnW", "W", UDim2.new(0.95, 0, 0.72, 0), sizeWASD, 30, Vector2.new(1, 0.5))

-- Botões Q e E (Alinhados perfeitamente no topo esquerdo usando pixels de margem fixa para não sobrepor o fundo)
local btnQ = createStyledButton("BtnQ", "Q", UDim2.new(0, 20, 0, 20), sizeQE, 22)
local btnE = createStyledButton("BtnE", "E", UDim2.new(0, 85, 0, 20), sizeQE, 22) -- Exatamente 65 pixels para a direita do Q

-- Ativando os comandos do teclado
setupKeyButton(btnW, Enum.KeyCode.W)
setupKeyButton(btnA, Enum.KeyCode.A)
setupKeyButton(btnS, Enum.KeyCode.S)
setupKeyButton(btnD, Enum.KeyCode.D)
setupKeyButton(btnQ, Enum.KeyCode.Q)
setupKeyButton(btnE, Enum.KeyCode.E)


-- --- DESIGN DO BOTÃO DE MINIMIZAR (Canto Superior Direito - Auto-posicionado) ---

local minimizerBtn = Instance.new("TextButton")
minimizerBtn.Name = "MinimizerBtn"
minimizerBtn.Size = UDim2.new(0, 38, 0, 38)
minimizerBtn.Position = UDim2.new(1, -20, 0, 20) -- Sempre a 20 pixels de distância das bordas do topo direito
minimizerBtn.AnchorPoint = Vector2.new(1, 0)
minimizerBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
minimizerBtn.BackgroundTransparency = 0.3
minimizerBtn.Text = "➖"
minimizerBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizerBtn.Font = Enum.Font.GothamBold
minimizerBtn.TextSize = 14
minimizerBtn.Parent = screenGui

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(1, 0)
minCorner.Parent = minimizerBtn

local minStroke = Instance.new("UIStroke")
minStroke.Thickness = 1.5
minStroke.Color = Color3.fromRGB(255, 60, 60)
minStroke.Parent = minimizerBtn

-- Lógica de minimizar/abrir suave
local menuAberto = true
minimizerBtn.MouseButton1Click:Connect(function()
    menuAberto = not menuAberto
    
    for _, btn in ipairs(gameplayButtons) do
        btn.Visible = menuAberto
    end
    
    if menuAberto then
        minimizerBtn.Text = "➖"
        TweenService:Create(minStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(255, 60, 60)}):Play()
    else
        minimizerBtn.Text = "➕"
        TweenService:Create(minStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(60, 255, 60)}):Play()
    end
end)
