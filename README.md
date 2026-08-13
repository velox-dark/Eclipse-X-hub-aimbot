-- ========================================================
-- CONFIGURAÇÃO DA LOGO FLUTUANTE & AUTO-EXECUTE
-- ========================================================
local LogoID = "rbxassetid://96312022457470" 

-- Limpeza de UI anterior se existir
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local UserInputService = game:GetService("UserInputService")
local SoundService = game:GetService("SoundService")

if CoreGui:FindFirstChild("EclipseXHubModern") then CoreGui.EclipseXHubModern:Destroy() end
if CoreGui:FindFirstChild("EclipseXHubToggle") then CoreGui.EclipseXHubToggle:Destroy() end

-- Criando a UI Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "EclipseXHubModern"
ScreenGui.Parent = CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 540, 0, 320)
MainFrame.Position = UDim2.new(0.5, -270, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true 

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- Barra Superior (Header)
local Header = Instance.new("Frame")
Header.Parent = MainFrame
Header.Size = UDim2.new(1, 0, 0, 40)
Header.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
Header.BorderSizePixel = 0

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 10)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel")
Title.Parent = Header
Title.Size = UDim2.new(0, 180, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "Eclipse X Hub"
Title.TextColor3 = Color3.fromRGB(180, 100, 255)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.BackgroundTransparency = 1

-- Container das Abas
local TabBar = Instance.new("Frame")
TabBar.Parent = Header
TabBar.Size = UDim2.new(1, -190, 1, 0)
TabBar.Position = UDim2.new(0, 180, 0, 0)
TabBar.BackgroundTransparency = 1

local TabListLayout = Instance.new("UIListLayout")
TabListLayout.Parent = TabBar
TabListLayout.FillDirection = Enum.FillDirection.Horizontal
TabListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
TabListLayout.VerticalAlignment = Enum.VerticalAlignment.Center
TabListLayout.Padding = UDim.new(0, 6)

-- Container dos Conteúdos
local ContentContainer = Instance.new("Frame")
ContentContainer.Parent = MainFrame
ContentContainer.Size = UDim2.new(1, -20, 1, -55)
ContentContainer.Position = UDim2.new(0, 10, 0, 48)
ContentContainer.BackgroundTransparency = 1

local Tabs = {}

local function CreateTab(name)
    local tabBtn = Instance.new("TextButton")
    tabBtn.Parent = TabBar
    tabBtn.Size = UDim2.new(0, 65, 0, 28)
    tabBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
    tabBtn.Text = name
    tabBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    tabBtn.Font = Enum.Font.GothamMedium
    tabBtn.TextSize = 10
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = tabBtn

    local page = Instance.new("ScrollingFrame")
    page.Parent = ContentContainer
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = Color3.fromRGB(180, 100, 255)

    local pageGrid = Instance.new("UIGridLayout")
    pageGrid.Parent = page
    pageGrid.CellSize = UDim2.new(0, 250, 0, 42)
    pageGrid.CellPadding = UDim2.new(0, 10, 0, 10)

    tabBtn.MouseButton1Click:Connect(function()
        for _, t in pairs(Tabs) do
            t.Page.Visible = false
            t.Btn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
            t.Btn.TextColor3 = Color3.fromRGB(200, 200, 200)
        end
        page.Visible = true
        tabBtn.BackgroundColor3 = Color3.fromRGB(140, 60, 230)
        tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    end)

    local tabData = {Btn = tabBtn, Page = page}
    table.insert(Tabs, tabData)

    if #Tabs == 1 then
        page.Visible = true
        tabBtn.BackgroundColor3 = Color3.fromRGB(140, 60, 230)
        tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    end

    return page
end

--------------------------------------------------------------------------------
-- CRIADORES DE ELEMENTOS INTERATIVOS DA UI
--------------------------------------------------------------------------------

-- Botão Simples
local function AddButton(page, text, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = page
    btn.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(240, 240, 240)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 13
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    local stroke = Instance.new("UIStroke")
    stroke.Parent = btn
    stroke.Color = Color3.fromRGB(45, 45, 60)
    stroke.Thickness = 1

    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Toggle com Suporte a Subtexto / Descrição
local function AddToggle(page, text, initialState, callback, subtext)
    local state = initialState or false

    local toggleFrame = Instance.new("Frame")
    toggleFrame.Parent = page
    toggleFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 36)

    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 6)
    frameCorner.Parent = toggleFrame

    local stroke = Instance.new("UIStroke")
    stroke.Parent = toggleFrame
    stroke.Color = Color3.fromRGB(45, 45, 60)
    stroke.Thickness = 1

    local label = Instance.new("TextLabel")
    label.Parent = toggleFrame
    label.Size = subtext and UDim2.new(1, -50, 0, 20) or UDim2.new(1, -50, 1, 0)
    label.Position = subtext and UDim2.new(0, 10, 0, 4) or UDim2.new(0, 10, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(240, 240, 240)
    label.Font = Enum.Font.Gotham
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1

    if subtext then
        local subLabel = Instance.new("TextLabel")
        subLabel.Parent = toggleFrame
        subLabel.Size = UDim2.new(1, -50, 0, 14)
        subLabel.Position = UDim2.new(0, 10, 0, 22)
        subLabel.Text = subtext
        subLabel.TextColor3 = Color3.fromRGB(220, 120, 120)
        subLabel.Font = Enum.Font.Gotham
        subLabel.TextSize = 8
        subLabel.TextXAlignment = Enum.TextXAlignment.Left
        subLabel.BackgroundTransparency = 1
    end

    local indicator = Instance.new("Frame")
    indicator.Parent = toggleFrame
    indicator.Size = UDim2.new(0, 32, 0, 18)
    indicator.Position = UDim2.new(1, -40, 0.5, -9)
    indicator.BackgroundColor3 = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 65)

    local indCorner = Instance.new("UICorner")
    indCorner.CornerRadius = UDim.new(1, 0)
    indCorner.Parent = indicator

    local dot = Instance.new("Frame")
    dot.Parent = indicator
    dot.Size = UDim2.new(0, 14, 0, 14)
    dot.Position = state and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)
    dot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(1, 0)
    dotCorner.Parent = dot

    local btnClick = Instance.new("TextButton")
    btnClick.Parent = toggleFrame
    btnClick.Size = UDim2.new(1, 0, 1, 0)
    btnClick.BackgroundTransparency = 1
    btnClick.Text = ""

    local function updateVisual(newState)
        state = newState
        local targetColor = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 65)
        local targetPos = state and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)

        TweenService:Create(indicator, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundColor3 = targetColor}):Play()
        TweenService:Create(dot, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos}):Play()
    end

    btnClick.MouseButton1Click:Connect(function()
        state = not state
        updateVisual(state)
        callback(state)
    end)
end

-- Entrada de Texto (Input por Teclado)
local function AddInput(page, text, defaultVal, callback)
    local inputFrame = Instance.new("Frame")
    inputFrame.Parent = page
    inputFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 36)

    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 6)
    frameCorner.Parent = inputFrame

    local stroke = Instance.new("UIStroke")
    stroke.Parent = inputFrame
    stroke.Color = Color3.fromRGB(45, 45, 60)
    stroke.Thickness = 1

    local label = Instance.new("TextLabel")
    label.Parent = inputFrame
    label.Size = UDim2.new(0.6, -10, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(240, 240, 240)
    label.Font = Enum.Font.Gotham
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1

    local textBox = Instance.new("TextBox")
    textBox.Parent = inputFrame
    textBox.Size = UDim2.new(0.38, -10, 0.7, 0)
    textBox.Position = UDim2.new(0.62, 0, 0.15, 0)
    textBox.BackgroundColor3 = Color3.fromRGB(20, 20, 26)
    textBox.Text = tostring(defaultVal)
    textBox.TextColor3 = Color3.fromRGB(180, 100, 255)
    textBox.Font = Enum.Font.GothamBold
    textBox.TextSize = 13
    textBox.ClearTextOnFocus = false

    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 4)
    boxCorner.Parent = textBox

    local boxStroke = Instance.new("UIStroke")
    boxStroke.Parent = textBox
    boxStroke.Color = Color3.fromRGB(60, 60, 80)
    boxStroke.Thickness = 1

    textBox.FocusLost:Connect(function()
        local num = tonumber(textBox.Text)
        if num then
            callback(num)
        else
            textBox.Text = tostring(defaultVal)
        end
    end)
end

-- ========================================================
-- BOTÃO FLUTUANTE (70x70)
-- ========================================================
local ToggleGui = Instance.new("ScreenGui")
ToggleGui.Name = "EclipseXHubToggle"
ToggleGui.Parent = CoreGui

local MinButton = Instance.new("ImageButton")
MinButton.Parent = ToggleGui
MinButton.Size = UDim2.new(0, 70, 0, 70)
MinButton.Position = UDim2.new(0, 15, 0, 120)
MinButton.Image = LogoID
MinButton.Draggable = true
MinButton.BackgroundColor3 = Color3.fromRGB(18, 18, 22)

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(1, 0)
MinCorner.Parent = MinButton

local MinStroke = Instance.new("UIStroke")
MinStroke.Parent = MinButton
MinStroke.Color = Color3.fromRGB(180, 100, 255)
MinStroke.Thickness = 2

MinButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- ========================================================
-- CRIAÇÃO DAS ABAS
-- ========================================================
local AimbotTab = CreateTab("Aimbot")
local MovementTab = CreateTab("Movimento")
local ESPTab = CreateTab("ESP & Visão")
local MusicTab = CreateTab("Música")
local ConfigTab = CreateTab("Config")

-- ========================================================
-- SISTEMA DE ÁUDIO / MÚSICA
-- ========================================================
local bgMusic = Instance.new("Sound")
bgMusic.Name = "EclipseHubMusic"
bgMusic.SoundId = "rbxassetid://110919391228823"
bgMusic.Volume = 0.5
bgMusic.Looped = true
bgMusic.Parent = SoundService

-- ========================================================
-- SERVIÇOS E CONFIGURAÇÕES GLOBAIS
-- ========================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

getgenv().AimbotConfig = {
    Enabled = false,
    VisualizarFOV = true,
    TamanhoFOV = 250,
    MaxRange = 500,
    RangeEnabled = true,
    Smoothness = 1.1,
    PredictionAmount = 0.05,
    PrioritizarCabeca = true
}

getgenv().MovementConfig = {
    SpeedEnabled = false,
    SpeedValue = 25,
    JumpEnabled = false,
    JumpValue = 75,
    SpinEnabled = false,
    SpinSpeed = 20,
    NoclipEnabled = false,
    WallClimbEnabled = false
}

getgenv().ESPConfig = {
    HighlightEnabled = false,
    BoxEnabled = false,
    SkeletonEnabled = false,
    Color = Color3.fromRGB(0, 255, 255),
    XRayEnabled = false,
    XRayTransparency = 0.5
}

getgenv().SystemConfig = {
    AutoExecute = false
}

-- CÍRCULO DO FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Transparency = 0.8
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(180, 100, 255)
FOVCircle.Filled = false

RunService.RenderStepped:Connect(function()
    if getgenv().AimbotConfig.Enabled and getgenv().AimbotConfig.VisualizarFOV then
        FOVCircle.Visible = true
        FOVCircle.Radius = getgenv().AimbotConfig.TamanhoFOV
        FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    else
        FOVCircle.Visible = false
    end
end)

-- ========================================================
-- SISTEMA DE AUTO EXECUTE & CONFIGS
-- ========================================================
local queueOnTeleport = (syn and syn.queue_on_teleport) or queue_on_teleport or (fluxus and fluxus.queue_on_teleport)

local function ConfigurarAutoExecute(status)
    getgenv().SystemConfig.AutoExecute = status
    if status and queueOnTeleport then
        local scriptCode = [[
            repeat task.wait() until game:IsLoaded()
            loadstring(game:HttpGet('https://raw.githubusercontent.com/sua-url-aqui/script.lua'))()
        ]]
        queueOnTeleport(scriptCode)
    end
end

LocalPlayer.OnTeleport:Connect(function(State)
    if getgenv().SystemConfig.AutoExecute and queueOnTeleport then
        local scriptCode = "loadstring(game:HttpGet('https://raw.githubusercontent.com/sua-url-aqui/script.lua'))()"
        queueOnTeleport(scriptCode)
    end
end)

-- ========================================================
-- SISTEMA DE NOCLIP (ULTRA LEVE & OTIMIZADO)
-- ========================================================
RunService.Stepped:Connect(function()
    if getgenv().MovementConfig.NoclipEnabled then
        local character = LocalPlayer.Character
        if character then
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") and part.CanCollide then
                    part.CanCollide = false
                end
            end
        end
    end
end)

-- ========================================================
-- SISTEMA DE ESCALAR PAREDES (WALL CLIMB / SPIDER)
-- ========================================================
RunService.RenderStepped:Connect(function()
    if getgenv().MovementConfig.WallClimbEnabled then
        local character = LocalPlayer.Character
        if character then
            local hrp = character:FindFirstChild("HumanoidRootPart")
            local hum = character:FindFirstChildOfClass("Humanoid")

            if hrp and hum then
                local raycastParams = RaycastParams.new()
                raycastParams.FilterDescendantsInstances = {character}
                raycastParams.FilterType = Enum.RaycastFilterType.Exclude

                -- Raycast para detectar parede à frente do personagem
                local rayResult = Workspace:Raycast(hrp.Position, hrp.CFrame.LookVector * 2.5, raycastParams)

                if rayResult then
                    -- Se houver parede próxima e o jogador estiver tentando subir/pular
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) or hum.MoveDirection.Magnitude > 0 then
                        hrp.Velocity = Vector3.new(hrp.Velocity.X, 35, hrp.Velocity.Z)
                    end
                end
            end
        end
    end
end)

-- ========================================================
-- SISTEMA DE AIMBOT
-- ========================================================
local TargetAtual = nil

local function ObterAlvoMaisProximo()
    local menorDistancia = math.huge
    local alvoEscolhido = nil
    local centroTela = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    local meuChar = LocalPlayer.Character
    local meuHrp = meuChar and meuChar:FindFirstChild("HumanoidRootPart")

    if TargetAtual and TargetAtual.Parent then
        local char = TargetAtual.Parent
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 then
            local posTela, naTela = Camera:WorldToViewportPoint(TargetAtual.Position)
            local distMouse = (Vector2.new(posTela.X, posTela.Y) - centroTela).Magnitude

            if naTela and distMouse <= getgenv().AimbotConfig.TamanhoFOV then
                return TargetAtual
            end
        end
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character and character:FindFirstChildOfClass("Humanoid") and character:FindFirstChildOfClass("Humanoid").Health > 0 then
                
                local partesParaTestar = {}
                if getgenv().AimbotConfig.PrioritizarCabeca and character:FindFirstChild("Head") then
                    table.insert(partesParaTestar, character.Head)
                end
                if character:FindFirstChild("HumanoidRootPart") then
                    table.insert(partesParaTestar, character.HumanoidRootPart)
                end

                for _, parteAlvo in ipairs(partesParaTestar) do
                    if getgenv().AimbotConfig.RangeEnabled and meuHrp then
                        local distanciaMundo = (meuHrp.Position - parteAlvo.Position).Magnitude
                        if distanciaMundo > getgenv().AimbotConfig.MaxRange then
                            continue
                        end
                    end

                    local posicaoTela, naTela = Camera:WorldToViewportPoint(parteAlvo.Position)
                    if naTela then
                        local distanciaMouse = (Vector2.new(posicaoTela.X, posicaoTela.Y) - centroTela).Magnitude
                        
                        if distanciaMouse < getgenv().AimbotConfig.TamanhoFOV and distanciaMouse < menorDistancia then
                            menorDistancia = distanciaMouse
                            alvoEscolhido = parteAlvo
                            break
                        end
                    end
                end
            end
        end
    end

    TargetAtual = alvoEscolhido
    return alvoEscolhido
end

RunService.RenderStepped:Connect(function()
    if getgenv().AimbotConfig.Enabled then
        local alvo = ObterAlvoMaisProximo()
        if alvo then
            local posicaoFutura = alvo.Position
            local hrp = alvo.Parent:FindFirstChild("HumanoidRootPart")
            
            if hrp and hrp:IsA("BasePart") then
                posicaoFutura = alvo.Position + (hrp.AssemblyLinearVelocity * getgenv().AimbotConfig.PredictionAmount)
            end

            local cframeAtual = Camera.CFrame
            local cframeDestino = CFrame.new(Camera.CFrame.Position, posicaoFutura)
            
            if getgenv().AimbotConfig.Smoothness <= 1 then
                Camera.CFrame = cframeDestino
            else
                Camera.CFrame = cframeAtual:Lerp(cframeDestino, 1 / getgenv().AimbotConfig.Smoothness)
            end
        end
    else
        TargetAtual = nil
    end
end)

-- ========================================================
-- SISTEMA DE MOVIMENTO
-- ========================================================
RunService.Heartbeat:Connect(function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        local humanoid = character.Humanoid
        local hrp = character:FindFirstChild("HumanoidRootPart")
        
        if getgenv().MovementConfig.SpeedEnabled then
            humanoid.WalkSpeed = getgenv().MovementConfig.SpeedValue
        end
        
        if getgenv().MovementConfig.JumpEnabled then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = getgenv().MovementConfig.JumpValue
        end

        if getgenv().MovementConfig.SpinEnabled and hrp then
            hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(getgenv().MovementConfig.SpinSpeed), 0)
        end
    end
end)

-- ========================================================
-- SISTEMA DE ESP & X-RAY
-- ========================================================
local ESP_Cache = {}

local BonesR15 = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
}

local BonesR6 = {
    {"Head", "Torso"},
    {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
    {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
}

local function CriarCacheESP(player)
    if ESP_Cache[player] then return end

    local box = Drawing.new("Square")
    box.Visible = false
    box.Thickness = 1.5
    box.Filled = false

    local skeleton = {}
    for i = 1, #BonesR15 do
        local line = Drawing.new("Line")
        line.Visible = false
        line.Thickness = 1.5
        skeleton[i] = line
    end

    ESP_Cache[player] = { Box = box, Skeleton = skeleton }
end

local function LimparCacheESP(player)
    local data = ESP_Cache[player]
    if data then
        data.Box:Remove()
        for i = 1, #data.Skeleton do
            data.Skeleton[i]:Remove()
        end
        ESP_Cache[player] = nil
    end
end

local function OcultarDesenhosESP(esp)
    if esp then
        esp.Box.Visible = false
        for i = 1, #esp.Skeleton do
            esp.Skeleton[i].Visible = false
        end
    end
end

local function AplicarHighlight(character)
    if character then
        local highlight = character:FindFirstChild("HighlightESP") or Instance.new("Highlight")
        highlight.Name = "HighlightESP"
        highlight.Adornee = character
        highlight.FillColor = getgenv().ESPConfig.Color
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.4
        highlight.Enabled = getgenv().ESPConfig.HighlightEnabled
        highlight.Parent = character
    end
end

local function AtualizarCoresHighlight()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hl = player.Character:FindFirstChild("HighlightESP")
            if hl then
                hl.FillColor = getgenv().ESPConfig.Color
            end
        end
    end
end

local function AlternarXRay(estado)
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and not v:IsDescendantOf(LocalPlayer.Character) then
            local eJogador = false
            for _, player in ipairs(Players:GetPlayers()) do
                if player.Character and v:IsDescendantOf(player.Character) then
                    eJogador = true
                    break
                end
            end

            if not eJogador then
                if estado then
                    if not v:FindFirstChild("TransparenciaOriginal") then
                        local valorOriginal = Instance.new("NumberValue")
                        valorOriginal.Name = "TransparenciaOriginal"
                        valorOriginal.Value = v.LocalTransparencyModifier
                        valorOriginal.Parent = v
                    end
                    v.LocalTransparencyModifier = getgenv().ESPConfig.XRayTransparency
                else
                    if v:FindFirstChild("TransparenciaOriginal") then
                        v.LocalTransparencyModifier = v.TransparenciaOriginal.Value
                        v.TransparenciaOriginal:Destroy()
                    else
                        v.LocalTransparencyModifier = 0
                    end
                end
            end
        end
    end
end

RunService.RenderStepped:Connect(function()
    local boxActive = getgenv().ESPConfig.BoxEnabled
    local skelActive = getgenv().ESPConfig.SkeletonEnabled
    local mainColor = getgenv().ESPConfig.Color

    for player, esp in pairs(ESP_Cache) do
        local char = player.Character
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        local isAlive = humanoid and humanoid.Health > 0
        local hrp = isAlive and char:FindFirstChild("HumanoidRootPart")

        if isAlive and hrp then
            local hrpPos, naTela = Camera:WorldToViewportPoint(hrp.Position)

            if boxActive and naTela then
                local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
                local scale = 1000 / dist
                local width = math.clamp(scale * 2.5, 10, 300)
                local height = math.clamp(scale * 4.5, 15, 500)

                esp.Box.Size = Vector2.new(width, height)
                esp.Box.Position = Vector2.new(hrpPos.X - width * 0.5, hrpPos.Y - height * 0.5)
                esp.Box.Color = mainColor
                esp.Box.Visible = true
            else
                esp.Box.Visible = false
            end

            if skelActive and naTela then
                local isR15 = humanoid.RigType == Enum.HumanoidRigType.R15
                local bones = isR15 and BonesR15 or BonesR6
                local skeletonLines = esp.Skeleton

                for i = 1, #skeletonLines do
                    local line = skeletonLines[i]
                    local pair = bones[i]

                    if pair then
                        local partA = char:FindFirstChild(pair[1])
                        local partB = char:FindFirstChild(pair[2])

                        if partA and partB then
                            local posA, visA = Camera:WorldToViewportPoint(partA.Position)
                            local posB, visB = Camera:WorldToViewportPoint(partB.Position)

                            if visA and visB then
                                line.From = Vector2.new(posA.X, posA.Y)
                                line.To = Vector2.new(posB.X, posB.Y)
                                line.Color = mainColor
                                line.Visible = true
                            else
                                line.Visible = false
                            end
                        else
                            line.Visible = false
                        end
                    else
                        line.Visible = false
                    end
                end
            else
                for i = 1, #esp.Skeleton do esp.Skeleton[i].Visible = false end
            end
        else
            OcultarDesenhosESP(esp)
        end
    end
end)

Players.PlayerRemoving:Connect(LimparCacheESP)

local function ConfigurarJogador(player)
    if player == LocalPlayer then return end
    CriarCacheESP(player)
    
    player.CharacterRemoving:Connect(function()
        if ESP_Cache[player] then
            OcultarDesenhosESP(ESP_Cache[player])
        end
    end)

    player.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        AplicarHighlight(char)
    end)
end

for _, player in ipairs(Players:GetPlayers()) do ConfigurarJogador(player) end
Players.PlayerAdded:Connect(ConfigurarJogador)

-- ========================================================
-- POPOVOANDO AS ABAS COM INPUTS E TOGGLES
-- ========================================================

-- TAB: AIMBOT
AddToggle(AimbotTab, "Ativar Aimbot", getgenv().AimbotConfig.Enabled, function(val)
    getgenv().AimbotConfig.Enabled = val
end)

AddToggle(AimbotTab, "Foco na Cabeça", getgenv().AimbotConfig.PrioritizarCabeca, function(val)
    getgenv().AimbotConfig.PrioritizarCabeca = val
end)

AddToggle(AimbotTab, "Mostrar FOV", getgenv().AimbotConfig.VisualizarFOV, function(val)
    getgenv().AimbotConfig.VisualizarFOV = val
end)

AddInput(AimbotTab, "Tamanho FOV (px)", getgenv().AimbotConfig.TamanhoFOV, function(val)
    getgenv().AimbotConfig.TamanhoFOV = val
end)

AddToggle(AimbotTab, "Limitar Distância", getgenv().AimbotConfig.RangeEnabled, function(val)
    getgenv().AimbotConfig.RangeEnabled = val
end)

AddInput(AimbotTab, "Distância Máx (studs)", getgenv().AimbotConfig.MaxRange, function(val)
    getgenv().AimbotConfig.MaxRange = val
end)

-- TAB: MOVIMENTO
AddToggle(MovementTab, "Ativar Speed", getgenv().MovementConfig.SpeedEnabled, function(val)
    getgenv().MovementConfig.SpeedEnabled = val
    if not val and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end
end)

AddInput(MovementTab, "Valor da Speed", getgenv().MovementConfig.SpeedValue, function(val)
    getgenv().MovementConfig.SpeedValue = val
end)

AddToggle(MovementTab, "Ativar JumpPower", getgenv().MovementConfig.JumpEnabled, function(val)
    getgenv().MovementConfig.JumpEnabled = val
    if not val and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.JumpPower = 50
    end
end)

AddInput(MovementTab, "Altura do Pulo", getgenv().MovementConfig.JumpValue, function(val)
    getgenv().MovementConfig.JumpValue = val
end)

AddToggle(MovementTab, "Girar Personagem", getgenv().MovementConfig.SpinEnabled, function(val)
    getgenv().MovementConfig.SpinEnabled = val
end)

AddInput(MovementTab, "Velocidade Giro", getgenv().MovementConfig.SpinSpeed, function(val)
    getgenv().MovementConfig.SpinSpeed = val
end)

AddToggle(MovementTab, "Noclip", getgenv().MovementConfig.NoclipEnabled, function(val)
    getgenv().MovementConfig.NoclipEnabled = val
end, "(Risco de ban permanente dependendo do jogo😶👍)")

-- ADICIONANDO ESCALAR PAREDES NA ABA DE MOVIMENTO COM A DESCRIÇÃO SOLICITADA
AddToggle(MovementTab, "Escalar Paredes", getgenv().MovementConfig.WallClimbEnabled, function(val)
    getgenv().MovementConfig.WallClimbEnabled = val
end, "(Pesso que tome cuidado quando usar 😶👍)")

-- TAB: ESP & VISÃO
AddToggle(ESPTab, "ESP Highlight", getgenv().ESPConfig.HighlightEnabled, function(val)
    getgenv().ESPConfig.HighlightEnabled = val
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            AplicarHighlight(player.Character)
        end
    end
end)

AddToggle(ESPTab, "ESP Caixa (Box)", getgenv().ESPConfig.BoxEnabled, function(val)
    getgenv().ESPConfig.BoxEnabled = val
end)

AddToggle(ESPTab, "ESP Esqueleto", getgenv().ESPConfig.SkeletonEnabled, function(val)
    getgenv().ESPConfig.SkeletonEnabled = val
end)

-- SELEÇÃO DE CORES DO ESP
local coresESP = {
    {"Ciano", Color3.fromRGB(0, 255, 255)},
    {"Verde", Color3.fromRGB(0, 255, 100)},
    {"Vermelho", Color3.fromRGB(255, 50, 50)},
    {"Amarelo", Color3.fromRGB(255, 255, 0)},
    {"Roxo", Color3.fromRGB(180, 100, 255)},
    {"Branco", Color3.fromRGB(255, 255, 255)}
}

local indiceCor = 1
AddButton(ESPTab, "Cor ESP: Ciano", function()
    indiceCor = indiceCor + 1
    if indiceCor > #coresESP then indiceCor = 1 end
    
    local corEscolhida = coresESP[indiceCor]
    getgenv().ESPConfig.Color = corEscolhida[2]
    AtualizarCoresHighlight()
    
    for _, btn in ipairs(ESPTab:GetChildren()) do
        if btn:IsA("TextButton") and string.find(btn.Text, "Cor ESP:") then
            btn.Text = "Cor ESP: " .. corEscolhida[1]
        end
    end
end)

AddToggle(ESPTab, "Visão Raio-X", getgenv().ESPConfig.XRayEnabled, function(val)
    getgenv().ESPConfig.XRayEnabled = val
    AlternarXRay(val)
end)

AddInput(ESPTab, "Visibilidade XRay", getgenv().ESPConfig.XRayTransparency, function(val)
    local valorFormatado = math.clamp(val, 0.05, 0.95)
    getgenv().ESPConfig.XRayTransparency = valorFormatado
    if getgenv().ESPConfig.XRayEnabled then
        AlternarXRay(true)
    end
end)

-- TAB: MÚSICA
AddToggle(MusicTab, "Ai Đưa Em Về", false, function(val)
    if val then
        bgMusic:Play()
    else
        bgMusic:Stop()
    end
end)

-- TAB: CONFIGURAÇÕES & AUTO EXECUTE
AddToggle(ConfigTab, "Auto Executar", getgenv().SystemConfig.AutoExecute, function(val)
    ConfigurarAutoExecute(val)
end)

AddButton(ConfigTab, "Reexecutar Script", function()
    MainFrame.Visible = false
    task.wait(0.2)
    loadstring(game:HttpGet('https://raw.githubusercontent.com/sua-url-aqui/script.lua'))()
end)
