-- Carrega a biblioteca da interface (Rayfield UI Library)
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Cria a janela principal do Hub (Eclipse X Hub)
local Window = Rayfield:CreateWindow({
   Name = "Eclipse X Hub",
   LoadingTitle = "Carregando Eclipse X...",
   LoadingSubtitle = "Sistema de Alta Performance",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

-- ABAS
local AimbotTab = Window:CreateTab("Aimbot", 4483362458)
local MovementTab = Window:CreateTab("Movimento", 4483362458)
local ESPTab = Window:CreateTab("ESP & Visão", 4483362458)

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- CONFIGURAÇÕES GLOBAIS
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
    NoclipEnabled = false
}

getgenv().ESPConfig = {
    HighlightEnabled = false,
    BoxEnabled = false,
    SkeletonEnabled = false,
    Color = Color3.fromRGB(0, 255, 255),
    XRayEnabled = false,
    XRayTransparency = 0.5
}

-- CÍRCULO DO FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Transparency = 0.8
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(0, 255, 255)
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

--------------------------------------------------------------------------------
-- SELEÇÃO DE ALVO DE ALTA PRECISÃO
--------------------------------------------------------------------------------
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

--------------------------------------------------------------------------------
-- LOOP DO AIMBOT
--------------------------------------------------------------------------------
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

--------------------------------------------------------------------------------
-- SISTEMA DE NOCLIP LEVE (COM LIGA/DESLIGA FUNCIONAL)
--------------------------------------------------------------------------------
RunService.Stepped:Connect(function()
    local character = LocalPlayer.Character
    if character then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                if getgenv().MovementConfig.NoclipEnabled then
                    part.CanCollide = false
                else
                    if part.Name == "HumanoidRootPart" or part.Name == "Head" or part.Name == "UpperTorso" or part.Name == "Torso" or part.Name:find("Leg") or part.Name:find("Arm") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end
end)

--------------------------------------------------------------------------------
-- LOOP DE MOVIMENTO
--------------------------------------------------------------------------------
RunService.Heartbeat:Connect(function()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        local humanoid = character.Humanoid
        local hrp = character:FindFirstChild("HumanoidRootPart")
        
        -- Velocidade de Andar
        if getgenv().MovementConfig.SpeedEnabled then
            humanoid.WalkSpeed = getgenv().MovementConfig.SpeedValue
        end
        
        -- Pulo
        if getgenv().MovementConfig.JumpEnabled then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = getgenv().MovementConfig.JumpValue
        end

        -- Giro (Spin)
        if getgenv().MovementConfig.SpinEnabled and hrp then
            hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(getgenv().MovementConfig.SpinSpeed), 0)
        end
    end
end)

-- CONTROLES DA UI: AIMBOT
AimbotTab:CreateToggle({
   Name = "Ativar Aimbot",
   CurrentValue = false,
   Flag = "AimbotToggle",
   Callback = function(Value) getgenv().AimbotConfig.Enabled = Value end,
})

AimbotTab:CreateToggle({
   Name = "Focar Apenas na Cabeça",
   CurrentValue = true,
   Flag = "HeadPrioritizeToggle",
   Callback = function(Value) getgenv().AimbotConfig.PrioritizarCabeca = Value end,
})

AimbotTab:CreateToggle({
   Name = "Mostrar Círculo do FOV",
   CurrentValue = true,
   Flag = "FOVToggle",
   Callback = function(Value) getgenv().AimbotConfig.VisualizarFOV = Value end,
})

AimbotTab:CreateSlider({
   Name = "Tamanho do FOV",
   Range = {50, 600},
   Increment = 10,
   Suffix = " px",
   CurrentValue = 250,
   Flag = "FOVSlider",
   Callback = function(Value) getgenv().AimbotConfig.TamanhoFOV = Value end,
})

AimbotTab:CreateToggle({
   Name = "Limitar Por Range/Distância",
   CurrentValue = true,
   Flag = "RangeLimitToggle",
   Callback = function(Value) getgenv().AimbotConfig.RangeEnabled = Value end,
})

local RangeSlider = AimbotTab:CreateSlider({
   Name = "Alcance Máximo (Range)",
   Range = {50, 5000},
   Increment = 25,
   Suffix = " studs",
   CurrentValue = 500,
   Flag = "RangeSlider",
   Callback = function(Value) getgenv().AimbotConfig.MaxRange = Value end,
})

AimbotTab:CreateDropdown({
   Name = "Modo de Alcance Pré-definido",
   Options = {"Curto (300 studs)", "Médio (800 studs)", "Longo (1500 studs)", "Sem Limite (Ilimitado)"},
   CurrentOption = "Médio (800 studs)",
   Flag = "RangePresetDropdown",
   Callback = function(Option)
      if Option == "Curto (300 studs)" then
         getgenv().AimbotConfig.RangeEnabled = true
         getgenv().AimbotConfig.MaxRange = 300
         RangeSlider:Set(300)
      elseif Option == "Médio (800 studs)" then
         getgenv().AimbotConfig.RangeEnabled = true
         getgenv().AimbotConfig.MaxRange = 800
         RangeSlider:Set(800)
      elseif Option == "Longo (1500 studs)" then
         getgenv().AimbotConfig.RangeEnabled = true
         getgenv().AimbotConfig.MaxRange = 1500
         RangeSlider:Set(1500)
      elseif Option == "Sem Limite (Ilimitado)" then
         getgenv().AimbotConfig.RangeEnabled = false
      end
   end,
})

AimbotTab:CreateKeybind({
   Name = "Atalho: Ligar/Desligar Limite de Range",
   CurrentKeybind = "R",
   HoldToInteract = false,
   Flag = "RangeKeybind",
   Callback = function()
      getgenv().AimbotConfig.RangeEnabled = not getgenv().AimbotConfig.RangeEnabled
      Rayfield:Notify({
         Title = "Aimbot Range",
         Content = "Limite de distância: " .. (getgenv().AimbotConfig.RangeEnabled and "ATIVADO" or "DESATIVADO"),
         Duration = 2,
         Image = 4483362458,
      })
   end,
})

AimbotTab:CreateSlider({
   Name = "Suavidade (Smoothness)",
   Range = {1, 5},
   Increment = 0.1,
   Suffix = "",
   CurrentValue = 1.1,
   Flag = "SmoothnessSlider",
   Callback = function(Value) getgenv().AimbotConfig.Smoothness = Value end,
})

-- CONTROLES DA UI: MOVIMENTO
MovementTab:CreateToggle({
   Name = "Ativar Noclip (risco de ban dependendo do jogo😶👍)",
   CurrentValue = false,
   Flag = "NoclipToggle",
   Callback = function(Value)
      getgenv().MovementConfig.NoclipEnabled = Value
   end,
})

MovementTab:CreateToggle({
   Name = "Ativar Speed",
   CurrentValue = false,
   Flag = "SpeedToggle",
   Callback = function(Value)
      getgenv().MovementConfig.SpeedEnabled = Value
      if not Value and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
          LocalPlayer.Character.Humanoid.WalkSpeed = 16
      end
   end,
})

MovementTab:CreateSlider({
   Name = "Velocidade de Andar",
   Range = {16, 100},
   Increment = 1,
   Suffix = " spd",
   CurrentValue = 25,
   Flag = "SpeedSlider",
   Callback = function(Value) getgenv().MovementConfig.SpeedValue = Value end,
})

MovementTab:CreateToggle({
   Name = "Ativar JumpPower",
   CurrentValue = false,
   Flag = "JumpToggle",
   Callback = function(Value)
      getgenv().MovementConfig.JumpEnabled = Value
      if not Value and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
          LocalPlayer.Character.Humanoid.JumpPower = 50
      end
   end,
})

MovementTab:CreateSlider({
   Name = "Altura do Pulo",
   Range = {50, 300},
   Increment = 5,
   Suffix = " jmp",
   CurrentValue = 75,
   Flag = "JumpSlider",
   Callback = function(Value) getgenv().MovementConfig.JumpValue = Value end,
})

MovementTab:CreateToggle({
   Name = "Girar Personagem (Spin)",
   CurrentValue = false,
   Flag = "SpinToggle",
   Callback = function(Value) getgenv().MovementConfig.SpinEnabled = Value end,
})

MovementTab:CreateSlider({
   Name = "Velocidade do Giro",
   Range = {5, 100},
   Increment = 5,
   Suffix = "°",
   CurrentValue = 20,
   Flag = "SpinSpeedSlider",
   Callback = function(Value) getgenv().MovementConfig.SpinSpeed = Value end,
})

--------------------------------------------------------------------------------
-- SISTEMA DE ESP OTIMIZADO & X-RAY
--------------------------------------------------------------------------------
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
    if getgenv().ESPConfig.HighlightEnabled and character then
        local highlight = character:FindFirstChild("HighlightESP") or Instance.new("Highlight")
        highlight.Name = "HighlightESP"
        highlight.Adornee = character
        highlight.FillColor = getgenv().ESPConfig.Color
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.4
        highlight.Parent = character
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
        if getgenv().ESPConfig.HighlightEnabled then
            AplicarHighlight(char)
        end
    end)
end

for _, player in ipairs(Players:GetPlayers()) do ConfigurarJogador(player) end
Players.PlayerAdded:Connect(ConfigurarJogador)

-- CONTROLES DA UI: ESP & VISÃO
ESPTab:CreateToggle({
   Name = "Ativar ESP (Highlights)",
   CurrentValue = false,
   Flag = "ESPHighlightToggle",
   Callback = function(Value)
      getgenv().ESPConfig.HighlightEnabled = Value
      for _, player in ipairs(Players:GetPlayers()) do
          if player ~= LocalPlayer and player.Character then
              local char = player.Character
              if Value then
                  AplicarHighlight(char)
              else
                  local hl = char:FindFirstChild("HighlightESP")
                  if hl then hl:Destroy() end
              end
          end
      end
   end,
})

ESPTab:CreateToggle({
   Name = "Ativar ESP (Caixa / Box)",
   CurrentValue = false,
   Flag = "ESPBoxToggle",
   Callback = function(Value) 
      getgenv().ESPConfig.BoxEnabled = Value 
   end,
})

ESPTab:CreateToggle({
   Name = "Ativar ESP (Esqueleto / Skeleton)",
   CurrentValue = false,
   Flag = "ESPSpawnToggle",
   Callback = function(Value) 
      getgenv().ESPConfig.SkeletonEnabled = Value 
   end,
})

ESPTab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = Color3.fromRGB(0, 255, 255),
    Flag = "ESPColorPicker",
    Callback = function(Value)
        getgenv().ESPConfig.Color = Value
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local hl = player.Character:FindFirstChild("HighlightESP")
                if hl then hl.FillColor = Value end
            end
        end
    end,
})

ESPTab:CreateToggle({
   Name = "Ativar X-Ray (Visão Raio-X)",
   CurrentValue = false,
   Flag = "XRayToggle",
   Callback = function(Value)
      getgenv().ESPConfig.XRayEnabled = Value
      AlternarXRay(Value)
   end,
})

ESPTab:CreateSlider({
   Name = "Transparência do X-Ray",
   Range = {0.1, 0.9},
   Increment = 0.1,
   Suffix = " trans",
   CurrentValue = 0.5,
   Flag = "XRayTransparencySlider",
   Callback = function(Value)
      getgenv().ESPConfig.XRayTransparency = Value
      if getgenv().ESPConfig.XRayEnabled then
          AlternarXRay(true)
      end
   end,
})

Rayfield:Notify({
   Title = "Eclipse X Hub Atualizado!",
   Content = "Texto do Noclip atualizado com o aviso desejado.",
   Duration = 3,
   Image = 4483362458,
})
