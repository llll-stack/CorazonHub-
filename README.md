local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- ==============================
-- CONFIGURAÇÕES
-- ==============================

local ATIVO = false

local DISTANCIA = 100
local DISTANCIA_MAX = 1000

local VELOCIDADE = 150
local VELOCIDADE_MIN = 50
local VELOCIDADE_MAX = 1000

local ALTURA_ABAIXO = 15

local MINIMIZADO = false

local NPCS_PUXADOS = {}

-- ==============================
-- GUI
-- ==============================

local gui = Instance.new("ScreenGui")
gui.Name = "CorazonHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0,350,0,380)
main.Position = UDim2.new(0.5,-175,0.5,-190)
main.BackgroundColor3 = Color3.fromRGB(13,13,18)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0,14)
mainCorner.Parent = main

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(140,35,75)
stroke.Thickness = 1.5
stroke.Transparency = 0.15
stroke.Parent = main

-- ==============================
-- HEADER
-- ==============================

local header = Instance.new("Frame")
header.Size = UDim2.new(1,0,0,65)
header.BackgroundColor3 = Color3.fromRGB(20,18,25)
header.BorderSizePixel = 0
header.Parent = main

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0,14)
headerCorner.Parent = header

local icon = Instance.new("TextLabel")
icon.Size = UDim2.new(0,45,0,45)
icon.Position = UDim2.new(0,12,0,10)
icon.BackgroundColor3 = Color3.fromRGB(125,35,70)
icon.Text = "C"
icon.TextColor3 = Color3.new(1,1,1)
icon.TextSize = 24
icon.Font = Enum.Font.GothamBold
icon.Parent = header

local iconCorner = Instance.new("UICorner")
iconCorner.CornerRadius = UDim.new(0,12)
iconCorner.Parent = icon

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0,180,0,25)
title.Position = UDim2.new(0,68,0,10)
title.BackgroundTransparency = 1
title.Text = "CORAZON HUB"
title.TextColor3 = Color3.new(1,1,1)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(0,180,0,20)
subtitle.Position = UDim2.new(0,69,0,34)
subtitle.BackgroundTransparency = 1
subtitle.Text = "NPC CONTROL"
subtitle.TextColor3 = Color3.fromRGB(145,145,155)
subtitle.TextSize = 11
subtitle.Font = Enum.Font.GothamMedium
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = header

-- ==============================
-- MINIMIZAR
-- ==============================

local minimize = Instance.new("TextButton")
minimize.Size = UDim2.new(0,30,0,30)
minimize.Position = UDim2.new(1,-72,0,17)
minimize.BackgroundColor3 = Color3.fromRGB(40,40,48)
minimize.Text = "—"
minimize.TextColor3 = Color3.new(1,1,1)
minimize.TextSize = 18
minimize.Font = Enum.Font.GothamBold
minimize.Parent = header

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0,8)
minCorner.Parent = minimize

-- ==============================
-- FECHAR
-- ==============================

local close = Instance.new("TextButton")
close.Size = UDim2.new(0,30,0,30)
close.Position = UDim2.new(1,-37,0,17)
close.BackgroundColor3 = Color3.fromRGB(90,30,50)
close.Text = "×"
close.TextColor3 = Color3.fromRGB(255,180,195)
close.TextSize = 20
close.Font = Enum.Font.GothamBold
close.Parent = header

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0,8)
closeCorner.Parent = close

-- ==============================
-- STATUS
-- ==============================

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1,-40,0,25)
status.Position = UDim2.new(0,20,0,82)
status.BackgroundTransparency = 1
status.Text = "● SISTEMA DESATIVADO"
status.TextColor3 = Color3.fromRGB(255,80,110)
status.TextSize = 12
status.Font = Enum.Font.GothamBold
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = main

-- ==============================
-- BOTÃO PUXAR
-- ==============================

local pullButton = Instance.new("TextButton")
pullButton.Size = UDim2.new(1,-40,0,52)
pullButton.Position = UDim2.new(0,20,0,112)
pullButton.BackgroundColor3 = Color3.fromRGB(125,35,70)
pullButton.Text = "🧲 PUXAR NPCs • [B]"
pullButton.TextColor3 = Color3.new(1,1,1)
pullButton.TextSize = 15
pullButton.Font = Enum.Font.GothamBold
pullButton.Parent = main

local pullCorner = Instance.new("UICorner")
pullCorner.CornerRadius = UDim.new(0,10)
pullCorner.Parent = pullButton

-- ==============================
-- RAIO
-- ==============================

local distanceTitle = Instance.new("TextLabel")
distanceTitle.Size = UDim2.new(0,180,0,20)
distanceTitle.Position = UDim2.new(0,20,0,178)
distanceTitle.BackgroundTransparency = 1
distanceTitle.Text = "RAIO DE ATRAÇÃO"
distanceTitle.TextColor3 = Color3.fromRGB(150,150,160)
distanceTitle.TextSize = 11
distanceTitle.Font = Enum.Font.GothamBold
distanceTitle.TextXAlignment = Enum.TextXAlignment.Left
distanceTitle.Parent = main

local distanceValue = Instance.new("TextLabel")
distanceValue.Size = UDim2.new(0,100,0,25)
distanceValue.Position = UDim2.new(1,-120,0,174)
distanceValue.BackgroundTransparency = 1
distanceValue.Text = tostring(DISTANCIA)
distanceValue.TextColor3 = Color3.fromRGB(255,100,145)
distanceValue.TextSize = 16
distanceValue.Font = Enum.Font.GothamBold
distanceValue.TextXAlignment = Enum.TextXAlignment.Right
distanceValue.Parent = main

local bar = Instance.new("Frame")
bar.Size = UDim2.new(1,-40,0,6)
bar.Position = UDim2.new(0,20,0,205)
bar.BackgroundColor3 = Color3.fromRGB(38,38,48)
bar.BorderSizePixel = 0
bar.Parent = main

local fill = Instance.new("Frame")
fill.Size = UDim2.new(DISTANCIA/DISTANCIA_MAX,0,1,0)
fill.BackgroundColor3 = Color3.fromRGB(180,45,95)
fill.BorderSizePixel = 0
fill.Parent = bar

local minusDistance = Instance.new("TextButton")
minusDistance.Size = UDim2.new(0,42,0,32)
minusDistance.Position = UDim2.new(0,20,0,222)
minusDistance.BackgroundColor3 = Color3.fromRGB(35,35,43)
minusDistance.Text = "−"
minusDistance.TextColor3 = Color3.new(1,1,1)
minusDistance.TextSize = 20
minusDistance.Font = Enum.Font.GothamBold
minusDistance.Parent = main

local plusDistance = Instance.new("TextButton")
plusDistance.Size = UDim2.new(0,42,0,32)
plusDistance.Position = UDim2.new(1,-62,0,222)
plusDistance.BackgroundColor3 = Color3.fromRGB(35,35,43)
plusDistance.Text = "+"
plusDistance.TextColor3 = Color3.new(1,1,1)
plusDistance.TextSize = 20
plusDistance.Font = Enum.Font.GothamBold
plusDistance.Parent = main

-- ==============================
-- VELOCIDADE
-- ==============================

local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(0,180,0,20)
speedTitle.Position = UDim2.new(0,20,0,265)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "VELOCIDADE DO PUXÃO"
speedTitle.TextColor3 = Color3.fromRGB(150,150,160)
speedTitle.TextSize = 11
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextXAlignment = Enum.TextXAlignment.Left
speedTitle.Parent = main

local speedValue = Instance.new("TextLabel")
speedValue.Size = UDim2.new(0,100,0,25)
speedValue.Position = UDim2.new(1,-120,0,261)
speedValue.BackgroundTransparency = 1
speedValue.Text = tostring(VELOCIDADE)
speedValue.TextColor3 = Color3.fromRGB(255,100,145)
speedValue.TextSize = 16
speedValue.Font = Enum.Font.GothamBold
speedValue.TextXAlignment = Enum.TextXAlignment.Right
speedValue.Parent = main

local speedBar = Instance.new("Frame")
speedBar.Size = UDim2.new(1,-40,0,6)
speedBar.Position = UDim2.new(0,20,0,292)
speedBar.BackgroundColor3 = Color3.fromRGB(38,38,48)
speedBar.BorderSizePixel = 0
speedBar.Parent = main

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new(
    VELOCIDADE / VELOCIDADE_MAX,
    0,
    1,
    0
)
speedFill.BackgroundColor3 = Color3.fromRGB(180,45,95)
speedFill.BorderSizePixel = 0
speedFill.Parent = speedBar

local minusSpeed = Instance.new("TextButton")
minusSpeed.Size = UDim2.new(0,42,0,32)
minusSpeed.Position = UDim2.new(0,20,0,310)
minusSpeed.BackgroundColor3 = Color3.fromRGB(35,35,43)
minusSpeed.Text = "−"
minusSpeed.TextColor3 = Color3.new(1,1,1)
minusSpeed.TextSize = 20
minusSpeed.Font = Enum.Font.GothamBold
minusSpeed.Parent = main

local plusSpeed = Instance.new("TextButton")
plusSpeed.Size = UDim2.new(0,42,0,32)
plusSpeed.Position = UDim2.new(1,-62,0,310)
plusSpeed.BackgroundColor3 = Color3.fromRGB(35,35,43)
plusSpeed.Text = "+"
plusSpeed.TextColor3 = Color3.new(1,1,1)
plusSpeed.TextSize = 20
plusSpeed.Font = Enum.Font.GothamBold
plusSpeed.Parent = main

-- ==============================
-- ATUALIZAR HUB
-- ==============================

local function atualizarHub()

    if ATIVO then

        status.Text = "● SISTEMA ATIVADO"
        status.TextColor3 = Color3.fromRGB(80,255,140)

        pullButton.Text = "🧲 PUXANDO NPCs • [B]"
        pullButton.BackgroundColor3 = Color3.fromRGB(35,145,80)

    else

        status.Text = "● SISTEMA DESATIVADO"
        status.TextColor3 = Color3.fromRGB(255,80,110)

        pullButton.Text = "🧲 PUXAR NPCs • [B]"
        pullButton.BackgroundColor3 = Color3.fromRGB(125,35,70)

    end
end

-- ==============================
-- ATIVAR
-- ==============================

local function alternar()

    ATIVO = not ATIVO

    if not ATIVO then
        NPCS_PUXADOS = {}
    end

    atualizarHub()
end

pullButton.MouseButton1Click:Connect(alternar)

UserInputService.InputBegan:Connect(function(input, processado)

    if processado then
        return
    end

    if input.KeyCode == Enum.KeyCode.B then
        alternar()
    end
end)

-- ==============================
-- DISTÂNCIA
-- ==============================

minusDistance.MouseButton1Click:Connect(function()

    DISTANCIA = math.max(
        50,
        DISTANCIA - 50
    )

    distanceValue.Text = tostring(DISTANCIA)

    fill.Size = UDim2.new(
        DISTANCIA / DISTANCIA_MAX,
        0,
        1,
        0
    )
end)

plusDistance.MouseButton1Click:Connect(function()

    DISTANCIA = math.min(
        DISTANCIA_MAX,
        DISTANCIA + 50
    )

    distanceValue.Text = tostring(DISTANCIA)

    fill.Size = UDim2.new(
        DISTANCIA / DISTANCIA_MAX,
        0,
        1,
        0
    )
end)

-- ==============================
-- VELOCIDADE
-- ==============================

minusSpeed.MouseButton1Click:Connect(function()

    VELOCIDADE = math.max(
        VELOCIDADE_MIN,
        VELOCIDADE - 50
    )

    speedValue.Text = tostring(VELOCIDADE)

    speedFill.Size = UDim2.new(
        VELOCIDADE / VELOCIDADE_MAX,
        0,
        1,
        0
    )
end)

plusSpeed.MouseButton1Click:Connect(function()

    VELOCIDADE = math.min(
        VELOCIDADE_MAX,
        VELOCIDADE + 50
    )

    speedValue.Text = tostring(VELOCIDADE)

    speedFill.Size = UDim2.new(
        VELOCIDADE / VELOCIDADE_MAX,
        0,
        1,
        0
    )
end)

-- ==============================
-- PREPARAR NPC
-- ==============================

local function prepararNPC(npc)

    for _,obj in ipairs(npc:GetDescendants()) do

        if obj:IsA("BasePart") then

            obj.CanCollide = false
            obj.CanTouch = false

            obj.AssemblyAngularVelocity = Vector3.zero
        end
    end
end

-- ==============================
-- PUXAR NPC
-- ==============================

local function puxarNPCs()

    local character = player.Character

    if not character then
        return
    end

    local playerRoot =
        character:FindFirstChild("HumanoidRootPart")

    if not playerRoot then
        return
    end

    -- 15 STUDS ABAIXO
    local alvo =
        playerRoot.Position -
        Vector3.new(0,ALTURA_ABAIXO,0)

    -- ==========================
    -- CAPTURAR NPCs
    -- ==========================

    for _,npc in ipairs(workspace:GetDescendants()) do

        if npc:IsA("Model")
            and npc ~= character then

            local humanoid =
                npc:FindFirstChildOfClass("Humanoid")

            local root =
                npc:FindFirstChild("HumanoidRootPart")

            if humanoid
                and root
                and humanoid.Health > 0
                and not Players:GetPlayerFromCharacter(npc) then

                local distancia =
                    (root.Position - playerRoot.Position).Magnitude

                if distancia <= DISTANCIA then

                    NPCS_PUXADOS[npc] = true

                    prepararNPC(npc)
                end
            end
        end
    end

    -- ==========================
    -- CONTROLAR NPCs
    -- ==========================

    for npc,_ in pairs(NPCS_PUXADOS) do

        if not npc or not npc.Parent then

            NPCS_PUXADOS[npc] = nil

        else

            local humanoid =
                npc:FindFirstChildOfClass("Humanoid")

            local root =
                npc:FindFirstChild("HumanoidRootPart")

            if not humanoid
                or not root
                or humanoid.Health <= 0 then

                NPCS_PUXADOS[npc] = nil

            else

                prepararNPC(npc)

                -- Alvo atualizado a cada frame
                local alvoAtual =
                    playerRoot.Position -
                    Vector3.new(0,ALTURA_ABAIXO,0)

                local posicaoAtual =
                    root.Position

                local distancia =
                    (alvoAtual - posicaoAtual).Magnitude

                if distancia > 0.5 then

                    -- Movimento suave e controlado
                    local passo =
                        math.min(
                            VELOCIDADE * 0.016,
                            distancia
                        )

                    local novaPosicao =
                        posicaoAtual:Lerp(
                            alvoAtual,
                            math.clamp(
                                passo / math.max(distancia,0.001),
                                0,
                                1
                            )
                        )

                    root.CFrame =
                        CFrame.new(
                            novaPosicao
                        )

                else

                    root.CFrame =
                        CFrame.new(alvoAtual)

                end

                -- Impede a física de fazer o NPC girar
                root.AssemblyLinearVelocity =
                    Vector3.zero

                root.AssemblyAngularVelocity =
                    Vector3.zero
            end
        end
    end
end

-- ==============================
-- LOOP
-- ==============================

RunService.Heartbeat:Connect(function()

    if ATIVO then
        puxarNPCs()
    end

end)

-- ==============================
-- MINIMIZAR
-- ==============================

minimize.MouseButton1Click:Connect(function()

    MINIMIZADO = not MINIMIZADO

    if MINIMIZADO then

        main.Size = UDim2.new(0,350,0,65)

        minimize.Text = "+"

        status.Visible = false
        pullButton.Visible = false

        distanceTitle.Visible = false
        distanceValue.Visible = false
        bar.Visible = false

        minusDistance.Visible = false
        plusDistance.Visible = false

        speedTitle.Visible = false
        speedValue.Visible = false
        speedBar.Visible = false

        minusSpeed.Visible = false
        plusSpeed.Visible = false

    else

        main.Size = UDim2.new(0,350,0,380)

        minimize.Text = "—"

        status.Visible = true
        pullButton.Visible = true

        distanceTitle.Visible = true
        distanceValue.Visible = true
        bar.Visible = true

        minusDistance.Visible = true
        plusDistance.Visible = true

        speedTitle.Visible = true
        speedValue.Visible = true
        speedBar.Visible = true

        minusSpeed.Visible = true
        plusSpeed.Visible = true
    end
end)

-- ==============================
-- FECHAR
-- ==============================

close.MouseButton1Click:Connect(function()

    ATIVO = false
    NPCS_PUXADOS = {}

    gui:Destroy()
end)

-- ==============================
-- EFEITOS
-- ==============================

local function efeito(botao)

    local tamanhoOriginal = botao.Size

    botao.MouseEnter:Connect(function()

        TweenService:Create(
            botao,
            TweenInfo.new(0.12),
            {
                Size =
                    tamanhoOriginal +
                    UDim2.new(0,4,0,2)
            }
        ):Play()

    end)

    botao.MouseLeave:Connect(function()

        TweenService:Create(
            botao,
            TweenInfo.new(0.12),
            {
                Size = tamanhoOriginal
            }
        ):Play()

    end)
end

efeito(pullButton)
efeito(minusDistance)
efeito(plusDistance)
efeito(minusSpeed)
efeito(plusSpeed)
efeito(minimize)
efeito(close)

atualizarHub()
