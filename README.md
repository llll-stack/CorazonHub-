-- ==============================================================================
-- CORAZON HUB - COM EFEITO DE LUZ NO TÍTULO
-- ==============================================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Stats = game:GetService("Stats")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- =========================================
-- ESTADOS GLOBAIS
-- =========================================

local ATIVO = false
local DISTANCIA = 100
local DISTANCIA_MAX = 1000
local VELOCIDADE = 150
local VELOCIDADE_MIN = 50
local VELOCIDADE_MAX = 1000
local ALTURA_ABAIXO = 15
local MINIMIZADO = false
local NPCS_PUXADOS = {}

-- Kill Aura
local KILL_AURA_ATIVO = false
local AUTO_V3_ATIVO = false
local AUTO_V4_ATIVO = false
local AUTO_BUSO_ATIVO = false
local ATTACK_DISTANCIA = 2500
local ATTACK_DELAY = 0.01
local autoV3Task = nil
local autoV4Task = nil
local autoBusoTask = nil

local estados = {
	WalkSpeed = false, JumpPower = false, InfJump = false,
	Fly = false, NoClip = false, AntiKb = false, AntiRagdoll = false
}
local configs = { WalkSpeedVal = 16, JumpPowerVal = 50 }

-- =========================================
-- SISTEMA DE TEMAS
-- =========================================
local TEMAS = {
	{
		nome = "Padrão (Rosa)",
		bg1 = Color3.fromRGB(18, 16, 26),
		bg2 = Color3.fromRGB(20, 18, 25),
		card = Color3.fromRGB(26, 22, 36),
		stroke = Color3.fromRGB(140, 35, 75),
		primaria = Color3.fromRGB(255, 100, 145),
		botao = Color3.fromRGB(125, 35, 70),
		botaoOff = Color3.fromRGB(180, 40, 60),
		texto = Color3.fromRGB(230, 220, 245),
		textoFraco = Color3.fromRGB(150, 150, 160),
		sidebarAtiva = Color3.fromRGB(125, 35, 70),
		sidebarInativa = Color3.fromRGB(28, 24, 38)
	},
	{
		nome = "Preto e Branco",
		bg1 = Color3.fromRGB(10, 10, 10),
		bg2 = Color3.fromRGB(15, 15, 15),
		card = Color3.fromRGB(25, 25, 25),
		stroke = Color3.fromRGB(220, 220, 220),
		primaria = Color3.fromRGB(255, 255, 255),
		botao = Color3.fromRGB(50, 50, 50),
		botaoOff = Color3.fromRGB(80, 80, 80),
		texto = Color3.fromRGB(240, 240, 240),
		textoFraco = Color3.fromRGB(150, 150, 150),
		sidebarAtiva = Color3.fromRGB(60, 60, 60),
		sidebarInativa = Color3.fromRGB(22, 22, 22)
	},
	{
		nome = "Roxo e Rosa",
		bg1 = Color3.fromRGB(22, 10, 35),
		bg2 = Color3.fromRGB(28, 12, 45),
		card = Color3.fromRGB(40, 18, 60),
		stroke = Color3.fromRGB(200, 80, 255),
		primaria = Color3.fromRGB(255, 120, 200),
		botao = Color3.fromRGB(140, 50, 200),
		botaoOff = Color3.fromRGB(180, 60, 220),
		texto = Color3.fromRGB(245, 220, 255),
		textoFraco = Color3.fromRGB(180, 150, 220),
		sidebarAtiva = Color3.fromRGB(140, 50, 200),
		sidebarInativa = Color3.fromRGB(35, 18, 55)
	},
	{
		nome = "Verde e Verde Escuro",
		bg1 = Color3.fromRGB(8, 20, 12),
		bg2 = Color3.fromRGB(10, 28, 16),
		card = Color3.fromRGB(18, 40, 24),
		stroke = Color3.fromRGB(50, 180, 80),
		primaria = Color3.fromRGB(120, 255, 150),
		botao = Color3.fromRGB(30, 130, 60),
		botaoOff = Color3.fromRGB(60, 160, 80),
		texto = Color3.fromRGB(220, 255, 230),
		textoFraco = Color3.fromRGB(130, 190, 150),
		sidebarAtiva = Color3.fromRGB(30, 130, 60),
		sidebarInativa = Color3.fromRGB(15, 35, 22)
	},
	{
		nome = "Azul e Azul Escuro",
		bg1 = Color3.fromRGB(8, 15, 30),
		bg2 = Color3.fromRGB(10, 20, 40),
		card = Color3.fromRGB(18, 30, 55),
		stroke = Color3.fromRGB(50, 120, 220),
		primaria = Color3.fromRGB(120, 180, 255),
		botao = Color3.fromRGB(30, 80, 180),
		botaoOff = Color3.fromRGB(50, 100, 220),
		texto = Color3.fromRGB(220, 235, 255),
		textoFraco = Color3.fromRGB(130, 160, 200),
		sidebarAtiva = Color3.fromRGB(30, 80, 180),
		sidebarInativa = Color3.fromRGB(15, 28, 50)
	}
}
local TEMA_ATUAL = 1

local antigo = playerGui:FindFirstChild("CorazonHub")
if antigo then antigo:Destroy() end

-- =========================================
-- GUI PRINCIPAL
-- =========================================

local gui = Instance.new("ScreenGui")
gui.Name = "CorazonHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(0, 0)
main.Position = UDim2.new(0.5, 0, 0.5, 0)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = TEMAS[TEMA_ATUAL].bg1
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.ClipsDescendants = true
main.Parent = gui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = main

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = TEMAS[TEMA_ATUAL].stroke
mainStroke.Thickness = 1.5
mainStroke.Parent = main

-- Animação de abertura
TweenService:Create(main, TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
	Size = UDim2.fromOffset(600, 400),
	Position = UDim2.new(0.5, -300, 0.5, -200)
}):Play()
main.AnchorPoint = Vector2.new(0, 0)

-- Glow pulsante
task.spawn(function()
	while main.Parent do
		TweenService:Create(mainStroke, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
			Transparency = 0.5, Thickness = 2
		}):Play()
		task.wait(2)
		if not main.Parent then break end
		TweenService:Create(mainStroke, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {
			Transparency = 0, Thickness = 1.5
		}):Play()
		task.wait(2)
	end
end)

-- =========================================
-- BARRA DE TÍTULO
-- =========================================

local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 45)
topBar.BackgroundColor3 = TEMAS[TEMA_ATUAL].bg2
topBar.BorderSizePixel = 0
topBar.Parent = main

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

-- =========================================
-- TÍTULO COM EFEITO DE LUZ PASSANDO
-- =========================================

local titleContainer = Instance.new("Frame")
titleContainer.Size = UDim2.fromOffset(220, 30)
titleContainer.Position = UDim2.fromOffset(15, 8)
titleContainer.BackgroundTransparency = 1
titleContainer.Parent = topBar

local titleLayout = Instance.new("UIListLayout")
titleLayout.FillDirection = Enum.FillDirection.Horizontal
titleLayout.SortOrder = Enum.SortOrder.LayoutOrder
titleLayout.Padding = UDim.new(0, 1)
titleLayout.VerticalAlignment = Enum.VerticalAlignment.Center
titleLayout.Parent = titleContainer

local LETRAS = {"C","O","R","A","Z","O","N"," ","H","U","B"}
local labelsLetras = {}

for i, letra in ipairs(LETRAS) do
	local lbl = Instance.new("TextLabel")
	lbl.Size = UDim2.fromOffset(letra == " " and 6 or 14, 26)
	lbl.BackgroundTransparency = 1
	lbl.Text = letra
	lbl.TextColor3 = TEMAS[TEMA_ATUAL].primaria
	lbl.TextSize = 16
	lbl.Font = Enum.Font.GothamBlack
	lbl.LayoutOrder = i
	lbl.Parent = titleContainer

	table.insert(labelsLetras, lbl)
end

-- Animação: onda de luz passando pelas letras
task.spawn(function()
	local idx = 1
	while titleContainer.Parent do
		local lbl = labelsLetras[idx]
		if lbl and lbl.Parent then
			-- Acende a letra (fica branca + cresce)
			TweenService:Create(lbl, TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
				TextColor3 = Color3.fromRGB(255, 255, 255),
				TextSize = 22
			}):Play()

			-- Volta ao normal
			task.delay(0.18, function()
				if lbl and lbl.Parent then
					TweenService:Create(lbl, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
						TextColor3 = TEMAS[TEMA_ATUAL].primaria,
						TextSize = 16
					}):Play()
				end
			end)
		end

		idx = idx + 1
		if idx > #labelsLetras then
			idx = 1
			task.wait(0.6) -- pausa antes de reiniciar a onda
		end
		task.wait(0.08) -- velocidade da onda
	end
end)

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.fromOffset(28, 28)
minimizeBtn.Position = UDim2.new(1, -68, 0, 8)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
minimizeBtn.Text = "—"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.TextSize = 14
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.AutoButtonColor = false
minimizeBtn.Parent = topBar
Instance.new("UICorner", minimizeBtn).CornerRadius = UDim.new(0, 8)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.fromOffset(28, 28)
closeBtn.Position = UDim2.new(1, -34, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(90, 30, 50)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 180, 195)
closeBtn.TextSize = 14
closeBtn.Font = Enum.Font.GothamBold
closeBtn.AutoButtonColor = false
closeBtn.Parent = topBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

-- Hover suave botões de topo
for _, b in ipairs({minimizeBtn, closeBtn}) do
	b.MouseEnter:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.15), { Size = UDim2.fromOffset(30, 30) }):Play()
	end)
	b.MouseLeave:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.15), { Size = UDim2.fromOffset(28, 28) }):Play()
	end)
end

closeBtn.Activated:Connect(function()
	ATIVO = false
	KILL_AURA_ATIVO = false
	AUTO_V3_ATIVO = false
	AUTO_V4_ATIVO = false
	AUTO_BUSO_ATIVO = false
	NPCS_PUXADOS = {}
	TweenService:Create(main, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
		Size = UDim2.fromOffset(0, 0)
	}):Play()
	task.wait(0.25)
	gui:Destroy()
end)

-- =========================================
-- SIDEBAR
-- =========================================

local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 150, 1, -45)
sidebar.Position = UDim2.fromOffset(0, 45)
sidebar.BackgroundTransparency = 1
sidebar.Parent = main

local botoesSidebar = {}

local function criarBotaoSidebar(texto, icone, posY)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 36)
	btn.Position = UDim2.fromOffset(10, posY)
	btn.BackgroundColor3 = TEMAS[TEMA_ATUAL].sidebarInativa
	btn.BorderSizePixel = 0
	btn.AutoButtonColor = false
	btn.Text = "   " .. icone .. "  " .. texto
	btn.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
	btn.TextSize = 12
	btn.Font = Enum.Font.GothamMedium
	btn.TextXAlignment = Enum.TextXAlignment.Left
	btn.Parent = sidebar
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

	local indicator = Instance.new("Frame")
	indicator.Size = UDim2.new(0, 3, 0, 0)
	indicator.Position = UDim2.new(0, 0, 0.5, 0)
	indicator.AnchorPoint = Vector2.new(0, 0.5)
	indicator.BackgroundColor3 = TEMAS[TEMA_ATUAL].primaria
	indicator.BorderSizePixel = 0
	indicator.Parent = btn
	Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)

	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.15), {
			BackgroundColor3 = Color3.new(
				math.min(TEMAS[TEMA_ATUAL].sidebarInativa.R + 0.1, 1),
				math.min(TEMAS[TEMA_ATUAL].sidebarInativa.G + 0.1, 1),
				math.min(TEMAS[TEMA_ATUAL].sidebarInativa.B + 0.1, 1)
			)
		}):Play()
	end)
	btn.MouseLeave:Connect(function()
		if btn.TextColor3 ~= Color3.new(1,1,1) then
			TweenService:Create(btn, TweenInfo.new(0.15), { BackgroundColor3 = TEMAS[TEMA_ATUAL].sidebarInativa }):Play()
		end
	end)

	table.insert(botoesSidebar, {btn = btn, indicator = indicator})
	return btn, indicator
end

local btnMainTab, indMain = criarBotaoSidebar("Main", "🏠", 10)
local btnPlayerTab, indPlayer = criarBotaoSidebar("Player", "👤", 52)
local btnMiscTab, indMisc = criarBotaoSidebar("Misc", "⚙️", 94)

-- =========================================
-- CONTAINER
-- =========================================

local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, -160, 1, -55)
contentContainer.Position = UDim2.fromOffset(155, 50)
contentContainer.BackgroundTransparency = 1
contentContainer.ClipsDescendants = true
contentContainer.Parent = main

local tabMainFrame = Instance.new("ScrollingFrame")
tabMainFrame.Size = UDim2.new(1, 0, 1, 0)
tabMainFrame.BackgroundTransparency = 1
tabMainFrame.BorderSizePixel = 0
tabMainFrame.CanvasSize = UDim2.new(0, 0, 0, 500)
tabMainFrame.ScrollBarThickness = 3
tabMainFrame.ScrollBarImageColor3 = TEMAS[TEMA_ATUAL].stroke
tabMainFrame.Visible = true
tabMainFrame.Parent = contentContainer

local tabPlayerFrame = Instance.new("ScrollingFrame")
tabPlayerFrame.Size = UDim2.new(1, 0, 1, 0)
tabPlayerFrame.BackgroundTransparency = 1
tabPlayerFrame.BorderSizePixel = 0
tabPlayerFrame.CanvasSize = UDim2.new(0, 0, 0, 520)
tabPlayerFrame.ScrollBarThickness = 3
tabPlayerFrame.ScrollBarImageColor3 = TEMAS[TEMA_ATUAL].stroke
tabPlayerFrame.Visible = false
tabPlayerFrame.Parent = contentContainer

local tabMiscFrame = Instance.new("ScrollingFrame")
tabMiscFrame.Size = UDim2.new(1, 0, 1, 0)
tabMiscFrame.BackgroundTransparency = 1
tabMiscFrame.BorderSizePixel = 0
tabMiscFrame.CanvasSize = UDim2.new(0, 0, 0, 700)
tabMiscFrame.ScrollBarThickness = 3
tabMiscFrame.ScrollBarImageColor3 = TEMAS[TEMA_ATUAL].stroke
tabMiscFrame.Visible = false
tabMiscFrame.Parent = contentContainer

local function mudarAba(ativa)
	local abas = {
		Main = { frame = tabMainFrame, btn = btnMainTab, ind = indMain },
		Player = { frame = tabPlayerFrame, btn = btnPlayerTab, ind = indPlayer },
		Misc = { frame = tabMiscFrame, btn = btnMiscTab, ind = indMisc }
	}
	for nome, d in pairs(abas) do
		local sel = (nome == ativa)
		d.frame.Visible = sel
		if sel then
			d.frame.Position = UDim2.fromOffset(15, 0)
			TweenService:Create(d.frame, TweenInfo.new(0.25, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
				Position = UDim2.fromOffset(0, 0)
			}):Play()
			TweenService:Create(d.btn, TweenInfo.new(0.2), {
				BackgroundColor3 = TEMAS[TEMA_ATUAL].sidebarAtiva,
				TextColor3 = Color3.new(1, 1, 1)
			}):Play()
			TweenService:Create(d.ind, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
				Size = UDim2.new(0, 3, 0, 22)
			}):Play()
		else
			TweenService:Create(d.btn, TweenInfo.new(0.2), {
				BackgroundColor3 = TEMAS[TEMA_ATUAL].sidebarInativa,
				TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
			}):Play()
			TweenService:Create(d.ind, TweenInfo.new(0.2), { Size = UDim2.new(0, 3, 0, 0) }):Play()
		end
	end
end

btnMainTab.Activated:Connect(function() mudarAba("Main") end)
btnPlayerTab.Activated:Connect(function() mudarAba("Player") end)
btnMiscTab.Activated:Connect(function() mudarAba("Misc") end)

-- Estado inicial
btnMainTab.BackgroundColor3 = TEMAS[TEMA_ATUAL].sidebarAtiva
btnMainTab.TextColor3 = Color3.new(1,1,1)
indMain.Size = UDim2.new(0, 3, 0, 22)

-- =========================================
-- ABA MAIN
-- =========================================

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -15, 0, 22)
status.Position = UDim2.fromOffset(0, 0)
status.BackgroundTransparency = 1
status.Text = "● SISTEMA DESATIVADO"
status.TextColor3 = Color3.fromRGB(255, 80, 110)
status.TextSize = 11
status.Font = Enum.Font.GothamBold
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = tabMainFrame

local pullButton = Instance.new("TextButton")
pullButton.Size = UDim2.new(1, -15, 0, 45)
pullButton.Position = UDim2.fromOffset(0, 26)
pullButton.BackgroundColor3 = TEMAS[TEMA_ATUAL].botao
pullButton.Text = "🧲 PUXAR NPCs • [B]"
pullButton.TextColor3 = Color3.new(1, 1, 1)
pullButton.TextSize = 13
pullButton.Font = Enum.Font.GothamBold
pullButton.AutoButtonColor = false
pullButton.Parent = tabMainFrame
Instance.new("UICorner", pullButton).CornerRadius = UDim.new(0, 8)

pullButton.MouseEnter:Connect(function()
	TweenService:Create(pullButton, TweenInfo.new(0.15), {
		BackgroundColor3 = Color3.new(
			math.min(TEMAS[TEMA_ATUAL].botao.R + 0.15, 1),
			math.min(TEMAS[TEMA_ATUAL].botao.G + 0.15, 1),
			math.min(TEMAS[TEMA_ATUAL].botao.B + 0.15, 1)
		)
	}):Play()
end)
pullButton.MouseLeave:Connect(function()
	TweenService:Create(pullButton, TweenInfo.new(0.15), { BackgroundColor3 = TEMAS[TEMA_ATUAL].botao }):Play()
end)

-- KILL AURA toggle
local kaCard = Instance.new("Frame")
kaCard.Size = UDim2.new(1, -15, 0, 52)
kaCard.Position = UDim2.fromOffset(0, 78)
kaCard.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
kaCard.BorderSizePixel = 0
kaCard.Parent = tabMainFrame
Instance.new("UICorner", kaCard).CornerRadius = UDim.new(0, 8)

local kaTitle = Instance.new("TextLabel")
kaTitle.Size = UDim2.fromOffset(200, 18)
kaTitle.Position = UDim2.fromOffset(12, 8)
kaTitle.BackgroundTransparency = 1
kaTitle.Text = "🗡️ KILL AURA"
kaTitle.TextColor3 = TEMAS[TEMA_ATUAL].texto
kaTitle.TextSize = 12
kaTitle.Font = Enum.Font.GothamBold
kaTitle.TextXAlignment = Enum.TextXAlignment.Left
kaTitle.Parent = kaCard

local kaDesc = Instance.new("TextLabel")
kaDesc.Size = UDim2.fromOffset(230, 16)
kaDesc.Position = UDim2.fromOffset(12, 28)
kaDesc.BackgroundTransparency = 1
kaDesc.Text = "Ataca mobs/players próximos"
kaDesc.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
kaDesc.TextSize = 10
kaDesc.Font = Enum.Font.GothamMedium
kaDesc.TextXAlignment = Enum.TextXAlignment.Left
kaDesc.Parent = kaCard

local kaToggle = Instance.new("TextButton")
kaToggle.Size = UDim2.fromOffset(50, 24)
kaToggle.Position = UDim2.new(1, -62, 0.5, -12)
kaToggle.BackgroundColor3 = TEMAS[TEMA_ATUAL].botaoOff
kaToggle.Text = "OFF"
kaToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
kaToggle.TextSize = 11
kaToggle.Font = Enum.Font.GothamBold
kaToggle.AutoButtonColor = false
kaToggle.Parent = kaCard
Instance.new("UICorner", kaToggle).CornerRadius = UDim.new(0, 6)

-- Sub-opções KA
local kaSubFrame = Instance.new("Frame")
kaSubFrame.Size = UDim2.new(1, -15, 0, 90)
kaSubFrame.Position = UDim2.fromOffset(0, 134)
kaSubFrame.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
kaSubFrame.BorderSizePixel = 0
kaSubFrame.Visible = false
kaSubFrame.Parent = tabMainFrame
Instance.new("UICorner", kaSubFrame).CornerRadius = UDim.new(0, 8)

local subToggles = {}
local function criarSubToggle(posY, titulo, desc, callback)
	local card = Instance.new("Frame")
	card.Size = UDim2.new(1, -16, 0, 28)
	card.Position = UDim2.fromOffset(8, posY)
	card.BackgroundTransparency = 1
	card.Parent = kaSubFrame

	local lbl = Instance.new("TextLabel")
	lbl.Size = UDim2.fromOffset(180, 18)
	lbl.Position = UDim2.fromOffset(6, 5)
	lbl.BackgroundTransparency = 1
	lbl.Text = titulo
	lbl.TextColor3 = TEMAS[TEMA_ATUAL].texto
	lbl.TextSize = 11
	lbl.Font = Enum.Font.GothamBold
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.Parent = card

	local descLbl = Instance.new("TextLabel")
	descLbl.Size = UDim2.fromOffset(160, 12)
	descLbl.Position = UDim2.fromOffset(6, 20)
	descLbl.BackgroundTransparency = 1
	descLbl.Text = desc
	descLbl.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
	descLbl.TextSize = 9
	descLbl.Font = Enum.Font.GothamMedium
	descLbl.TextXAlignment = Enum.TextXAlignment.Left
	descLbl.Parent = card

	local tgl = Instance.new("TextButton")
	tgl.Size = UDim2.fromOffset(46, 22)
	tgl.Position = UDim2.new(1, -54, 0.5, -11)
	tgl.BackgroundColor3 = TEMAS[TEMA_ATUAL].botaoOff
	tgl.Text = "OFF"
	tgl.TextColor3 = Color3.fromRGB(255, 255, 255)
	tgl.TextSize = 10
	tgl.Font = Enum.Font.GothamBold
	tgl.AutoButtonColor = false
	tgl.Parent = card
	Instance.new("UICorner", tgl).CornerRadius = UDim.new(0, 6)

	tgl.Activated:Connect(function()
		local ativo = tgl.Text == "OFF"
		tgl.Text = ativo and "ON" or "OFF"
		tgl.BackgroundColor3 = ativo and Color3.fromRGB(50, 205, 50) or TEMAS[TEMA_ATUAL].botaoOff
		callback(ativo)
	end)

	table.insert(subToggles, {lbl = lbl, descLbl = descLbl, tgl = tgl})
end

criarSubToggle(2, "Auto V3", "Ativa habilidade V3", function(v)
	AUTO_V3_ATIVO = v
	if autoV3Task then task.cancel(autoV3Task); autoV3Task = nil end
	if v then
		autoV3Task = task.spawn(function()
			while AUTO_V3_ATIVO do
				pcall(function()
					local R = ReplicatedStorage:FindFirstChild("Remotes")
					local C = R and R:FindFirstChild("CommE")
					if C then C:FireServer("ActivateAbility") end
				end)
				task.wait(1)
			end
		end)
	end
end)

criarSubToggle(32, "Auto V4", "Ativa Awakening", function(v)
	AUTO_V4_ATIVO = v
	if autoV4Task then task.cancel(autoV4Task); autoV4Task = nil end
	if v then
		autoV4Task = task.spawn(function()
			while AUTO_V4_ATIVO do
				pcall(function()
					local char = player.Character
					local hum = char and char:FindFirstChildOfClass("Humanoid")
					if hum and hum.Health > 0 then
						local bp = player:FindFirstChild("Backpack")
						local awk = bp and bp:FindFirstChild("Awakening")
						local rf = awk and awk:FindFirstChild("RemoteFunction")
						if rf then rf:InvokeServer(true) end
					end
				end)
				task.wait(1)
			end
		end)
	end
end)

criarSubToggle(62, "Auto Buso", "Ativa Haki", function(v)
	AUTO_BUSO_ATIVO = v
	if autoBusoTask then task.cancel(autoBusoTask); autoBusoTask = nil end
	if v then
		autoBusoTask = task.spawn(function()
			while AUTO_BUSO_ATIVO do
				pcall(function()
					local char = player.Character
					local hum = char and char:FindFirstChildOfClass("Humanoid")
					if hum and hum.Health > 0 then
						local R = ReplicatedStorage:FindFirstChild("Remotes")
						local C = R and R:FindFirstChild("CommE")
						if C then C:FireServer("ActivateAbility") end
					end
				end)
				task.wait(0.5)
			end
		end)
	end
end)

-- Sliders
local distanceTitle = Instance.new("TextLabel")
distanceTitle.Size = UDim2.new(0, 180, 0, 18)
distanceTitle.Position = UDim2.fromOffset(0, 235)
distanceTitle.BackgroundTransparency = 1
distanceTitle.Text = "RAIO DE ATRAÇÃO"
distanceTitle.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
distanceTitle.TextSize = 10
distanceTitle.Font = Enum.Font.GothamBold
distanceTitle.TextXAlignment = Enum.TextXAlignment.Left
distanceTitle.Parent = tabMainFrame

local distanceValue = Instance.new("TextLabel")
distanceValue.Size = UDim2.new(0, 100, 0, 20)
distanceValue.Position = UDim2.new(1, -115, 0, 233)
distanceValue.BackgroundTransparency = 1
distanceValue.Text = tostring(DISTANCIA)
distanceValue.TextColor3 = TEMAS[TEMA_ATUAL].primaria
distanceValue.TextSize = 14
distanceValue.Font = Enum.Font.GothamBold
distanceValue.TextXAlignment = Enum.TextXAlignment.Right
distanceValue.Parent = tabMainFrame

local bar = Instance.new("Frame")
bar.Size = UDim2.new(1, -15, 0, 5)
bar.Position = UDim2.fromOffset(0, 257)
bar.BackgroundColor3 = Color3.fromRGB(38, 38, 48)
bar.BorderSizePixel = 0
bar.Parent = tabMainFrame

local fill = Instance.new("Frame")
fill.Size = UDim2.new(DISTANCIA/DISTANCIA_MAX, 0, 1, 0)
fill.BackgroundColor3 = TEMAS[TEMA_ATUAL].botao
fill.BorderSizePixel = 0
fill.Parent = bar

local minusDistance = Instance.new("TextButton")
minusDistance.Size = UDim2.new(0, 38, 0, 28)
minusDistance.Position = UDim2.fromOffset(0, 269)
minusDistance.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
minusDistance.Text = "−"
minusDistance.TextColor3 = Color3.new(1, 1, 1)
minusDistance.TextSize = 16
minusDistance.Font = Enum.Font.GothamBold
minusDistance.AutoButtonColor = false
minusDistance.Parent = tabMainFrame
Instance.new("UICorner", minusDistance).CornerRadius = UDim.new(0, 6)

local plusDistance = Instance.new("TextButton")
plusDistance.Size = UDim2.new(0, 38, 0, 28)
plusDistance.Position = UDim2.new(1, -53, 0, 269)
plusDistance.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
plusDistance.Text = "+"
plusDistance.TextColor3 = Color3.new(1, 1, 1)
plusDistance.TextSize = 16
plusDistance.Font = Enum.Font.GothamBold
plusDistance.AutoButtonColor = false
plusDistance.Parent = tabMainFrame
Instance.new("UICorner", plusDistance).CornerRadius = UDim.new(0, 6)

local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(0, 180, 0, 18)
speedTitle.Position = UDim2.fromOffset(0, 310)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "VELOCIDADE DO PUXÃO"
speedTitle.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
speedTitle.TextSize = 10
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextXAlignment = Enum.TextXAlignment.Left
speedTitle.Parent = tabMainFrame

local speedValue = Instance.new("TextLabel")
speedValue.Size = UDim2.new(0, 100, 0, 20)
speedValue.Position = UDim2.new(1, -115, 0, 308)
speedValue.BackgroundTransparency = 1
speedValue.Text = tostring(VELOCIDADE)
speedValue.TextColor3 = TEMAS[TEMA_ATUAL].primaria
speedValue.TextSize = 14
speedValue.Font = Enum.Font.GothamBold
speedValue.TextXAlignment = Enum.TextXAlignment.Right
speedValue.Parent = tabMainFrame

local speedBar = Instance.new("Frame")
speedBar.Size = UDim2.new(1, -15, 0, 5)
speedBar.Position = UDim2.fromOffset(0, 332)
speedBar.BackgroundColor3 = Color3.fromRGB(38, 38, 48)
speedBar.BorderSizePixel = 0
speedBar.Parent = tabMainFrame

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new(VELOCIDADE / VELOCIDADE_MAX, 0, 1, 0)
speedFill.BackgroundColor3 = TEMAS[TEMA_ATUAL].botao
speedFill.BorderSizePixel = 0
speedFill.Parent = speedBar

local minusSpeed = Instance.new("TextButton")
minusSpeed.Size = UDim2.new(0, 38, 0, 28)
minusSpeed.Position = UDim2.fromOffset(0, 344)
minusSpeed.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
minusSpeed.Text = "−"
minusSpeed.TextColor3 = Color3.new(1, 1, 1)
minusSpeed.TextSize = 16
minusSpeed.Font = Enum.Font.GothamBold
minusSpeed.AutoButtonColor = false
minusSpeed.Parent = tabMainFrame
Instance.new("UICorner", minusSpeed).CornerRadius = UDim.new(0, 6)

local plusSpeed = Instance.new("TextButton")
plusSpeed.Size = UDim2.new(0, 38, 0, 28)
plusSpeed.Position = UDim2.new(1, -53, 0, 344)
plusSpeed.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
plusSpeed.Text = "+"
plusSpeed.TextColor3 = Color3.new(1, 1, 1)
plusSpeed.TextSize = 16
plusSpeed.Font = Enum.Font.GothamBold
plusSpeed.AutoButtonColor = false
plusSpeed.Parent = tabMainFrame
Instance.new("UICorner", plusSpeed).CornerRadius = UDim.new(0, 6)

for _, b in ipairs({minusDistance, plusDistance, minusSpeed, plusSpeed}) do
	b.MouseEnter:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = TEMAS[TEMA_ATUAL].primaria }):Play()
	end)
	b.MouseLeave:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.15), { BackgroundColor3 = TEMAS[TEMA_ATUAL].card }):Play()
	end)
end

-- Controle puxar
local function atualizarHub()
	if ATIVO then
		status.Text = "● SISTEMA ATIVADO"
		status.TextColor3 = Color3.fromRGB(80, 255, 140)
		pullButton.Text = "🧲 PUXANDO NPCs • [B]"
		pullButton.BackgroundColor3 = Color3.fromRGB(35, 145, 80)
	else
		status.Text = "● SISTEMA DESATIVADO"
		status.TextColor3 = Color3.fromRGB(255, 80, 110)
		pullButton.Text = "🧲 PUXAR NPCs • [B]"
		pullButton.BackgroundColor3 = TEMAS[TEMA_ATUAL].botao
	end
end

local function alternar()
	ATIVO = not ATIVO
	if not ATIVO then NPCS_PUXADOS = {} end
	atualizarHub()
	TweenService:Create(pullButton, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {
		Size = UDim2.new(1, -5, 0, 50)
	}):Play()
	task.wait(0.1)
	TweenService:Create(pullButton, TweenInfo.new(0.15, Enum.EasingStyle.Back), {
		Size = UDim2.new(1, -15, 0, 45)
	}):Play()
end

pullButton.MouseButton1Click:Connect(alternar)

UserInputService.InputBegan:Connect(function(input, processado)
	if processado then return end
	if input.KeyCode == Enum.KeyCode.B then alternar() end
end)

minusDistance.MouseButton1Click:Connect(function()
	DISTANCIA = math.max(50, DISTANCIA - 50)
	distanceValue.Text = tostring(DISTANCIA)
	TweenService:Create(fill, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {
		Size = UDim2.new(DISTANCIA / DISTANCIA_MAX, 0, 1, 0)
	}):Play()
end)

plusDistance.MouseButton1Click:Connect(function()
	DISTANCIA = math.min(DISTANCIA_MAX, DISTANCIA + 50)
	distanceValue.Text = tostring(DISTANCIA)
	TweenService:Create(fill, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {
		Size = UDim2.new(DISTANCIA / DISTANCIA_MAX, 0, 1, 0)
	}):Play()
end)

minusSpeed.MouseButton1Click:Connect(function()
	VELOCIDADE = math.max(VELOCIDADE_MIN, VELOCIDADE - 50)
	speedValue.Text = tostring(VELOCIDADE)
	TweenService:Create(speedFill, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {
		Size = UDim2.new(VELOCIDADE / VELOCIDADE_MAX, 0, 1, 0)
	}):Play()
end)

plusSpeed.MouseButton1Click:Connect(function()
	VELOCIDADE = math.min(VELOCIDADE_MAX, VELOCIDADE + 50)
	speedValue.Text = tostring(VELOCIDADE)
	TweenService:Create(speedFill, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {
		Size = UDim2.new(VELOCIDADE / VELOCIDADE_MAX, 0, 1, 0)
	}):Play()
end)

-- Puxar NPC
local function prepararNPC(npc)
	for _, obj in ipairs(npc:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.CanCollide = false
			obj.CanTouch = true
			obj.CanQuery = true
			obj.AssemblyAngularVelocity = Vector3.zero
		end
	end
	local hum = npc:FindFirstChildOfClass("Humanoid")
	if hum then hum.PlatformStand = false end
end

local function puxarNPCs()
	local char = player.Character
	if not char then return end
	local pr = char:FindFirstChild("HumanoidRootPart")
	if not pr then return end
	for _, npc in ipairs(workspace:GetDescendants()) do
		if npc:IsA("Model") and npc ~= char then
			local hum = npc:FindFirstChildOfClass("Humanoid")
			local root = npc:FindFirstChild("HumanoidRootPart")
			if hum and root and hum.Health > 0 and not Players:GetPlayerFromCharacter(npc) then
				if (root.Position - pr.Position).Magnitude <= DISTANCIA then
					NPCS_PUXADOS[npc] = true
					prepararNPC(npc)
				end
			end
		end
	end
	for npc in pairs(NPCS_PUXADOS) do
		if not npc or not npc.Parent then
			NPCS_PUXADOS[npc] = nil
		else
			local hum = npc:FindFirstChildOfClass("Humanoid")
			local root = npc:FindFirstChild("HumanoidRootPart")
			if not hum or not root or hum.Health <= 0 then
				NPCS_PUXADOS[npc] = nil
			else
				prepararNPC(npc)
				local alvo = pr.Position - Vector3.new(0, ALTURA_ABAIXO, 0)
				local pos = root.Position
				local d = (alvo - pos).Magnitude
				if d > 0.5 then
					local passo = math.min(VELOCIDADE * 0.016, d)
					root.CFrame = CFrame.new(pos:Lerp(alvo, math.clamp(passo / math.max(d, 0.001), 0, 1)))
				else
					root.CFrame = CFrame.new(alvo)
				end
				root.AssemblyLinearVelocity = Vector3.zero
				root.AssemblyAngularVelocity = Vector3.zero
			end
		end
	end
end

RunService.Heartbeat:Connect(function()
	if ATIVO then puxarNPCs() end
end)

-- =========================================
-- KILL AURA - LÓGICA
-- =========================================

local function SafeWaitForChild(parent, childName, timeout)
	timeout = timeout or 10
	local ok, res = pcall(function() return parent:WaitForChild(childName, timeout) end)
	if ok then return res end
	return nil
end

local function IsAlive(character)
	if not character then return false end
	local hum = character:FindFirstChildOfClass("Humanoid")
	return hum and hum.Health and hum.Health > 0
end

local function GetRandomValidPart(target)
	if not target then return nil end
	local allParts = target:GetDescendants()
	local valid = {}
	local hrp = target:FindFirstChild("HumanoidRootPart")
	local boneParts = hrp and hrp.Parent and hrp.Parent:GetDescendants() or {}
	for _, part in ipairs(allParts) do
		if part:IsA("BasePart") and part.CanCollide and table.find(boneParts, part) then
			table.insert(valid, part)
		end
	end
	return #valid > 0 and valid[math.random(1, #valid)] or target:FindFirstChild("HumanoidRootPart")
end

local function CheckAndGetCoreComponents()
	local R = SafeWaitForChild(ReplicatedStorage, "Remotes", 1)
	local M = SafeWaitForChild(ReplicatedStorage, "Modules", 1)
	local N = M and SafeWaitForChild(M, "Net", 1) or nil
	local RA = N and SafeWaitForChild(N, "RE/RegisterAttack", 1) or nil
	local RH = N and SafeWaitForChild(N, "RE/RegisterHit", 1) or nil
	local E = SafeWaitForChild(Workspace, "Enemies", 1)
	return R, N, RA, RH, E
end

local FastAttack = {
	Distance = ATTACK_DISTANCIA,
	attackMobs = true,
	attackPlayers = true,
	IsRunning = false
}

local function ProcessEnemies(OthersEnemies, Folder)
	if not Folder or not FastAttack.attackMobs then return nil end
	local BasePart = nil
	for _, Enemy in ipairs(Folder:GetChildren()) do
		if Enemy == player.Character or not IsAlive(Enemy) then continue end
		local fp = GetRandomValidPart(Enemy)
		if fp and (player.Character and player.Character:FindFirstChild("HumanoidRootPart") and (player.Character.HumanoidRootPart.Position - fp.Position).Magnitude < FastAttack.Distance) then
			table.insert(OthersEnemies, {Enemy, fp})
			BasePart = fp
		end
	end
	return BasePart
end

local function ProcessRealPlayers(OthersEnemies)
	if not FastAttack.attackPlayers then return nil end
	local BasePart = nil
	for _, OP in ipairs(Players:GetPlayers()) do
		if OP == player then continue end
		local OC = OP.Character
		if not IsAlive(OC) then continue end
		local fp = GetRandomValidPart(OC)
		if fp and (player.Character and player.Character:FindFirstChild("HumanoidRootPart") and (player.Character.HumanoidRootPart.Position - fp.Position).Magnitude < FastAttack.Distance) then
			table.insert(OthersEnemies, {OC, fp})
			BasePart = fp
		end
	end
	return BasePart
end

local function Attack(BasePart, OthersEnemies)
	local _, _, RA, RH, _ = CheckAndGetCoreComponents()
	if not (BasePart and OthersEnemies and #OthersEnemies > 0 and RA and RH) then return end
	RA:FireServer(ATTACK_DELAY)
	RH:FireServer(BasePart, OthersEnemies)
end

local function AttackNearest()
	if not FastAttack.IsRunning then return end
	local _, _, _, _, E = CheckAndGetCoreComponents()
	local OE = {}
	local P1 = ProcessEnemies(OE, E)
	local P2 = ProcessRealPlayers(OE)
	if #OE > 0 then Attack(P1 or P2, OE) end
end

local function BladeHits()
	if not FastAttack.IsRunning then return end
	local eq = player.Character and IsAlive(player.Character) and player.Character:FindFirstChildOfClass("Tool")
	if eq and eq.ToolTip ~= "Gun" then AttackNearest() end
end

task.spawn(function()
	while true do
		task.wait(ATTACK_DELAY)
		if FastAttack.IsRunning then BladeHits() else task.wait() end
	end
end)

kaToggle.Activated:Connect(function()
	KILL_AURA_ATIVO = not KILL_AURA_ATIVO
	FastAttack.IsRunning = KILL_AURA_ATIVO
	if KILL_AURA_ATIVO then
		kaToggle.Text = "ON"
		kaToggle.BackgroundColor3 = Color3.fromRGB(50, 205, 50)
		kaSubFrame.Visible = true
		kaSubFrame.Size = UDim2.new(1, -15, 0, 0)
		TweenService:Create(kaSubFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quint), {
			Size = UDim2.new(1, -15, 0, 90)
		}):Play()
	else
		kaToggle.Text = "OFF"
		kaToggle.BackgroundColor3 = TEMAS[TEMA_ATUAL].botaoOff
		AUTO_V3_ATIVO = false; AUTO_V4_ATIVO = false; AUTO_BUSO_ATIVO = false
		if autoV3Task then task.cancel(autoV3Task); autoV3Task = nil end
		if autoV4Task then task.cancel(autoV4Task); autoV4Task = nil end
		if autoBusoTask then task.cancel(autoBusoTask); autoBusoTask = nil end
		TweenService:Create(kaSubFrame, TweenInfo.new(0.2), { Size = UDim2.new(1, -15, 0, 0) }):Play()
		task.wait(0.2)
		kaSubFrame.Visible = false
	end
end)

-- =========================================
-- ABA PLAYER
-- =========================================

local playerSecTitle = Instance.new("TextLabel")
playerSecTitle.Size = UDim2.new(1, 0, 0, 20)
playerSecTitle.Position = UDim2.fromOffset(0, 0)
playerSecTitle.BackgroundTransparency = 1
playerSecTitle.Text = "Configurações do Jogador"
playerSecTitle.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
playerSecTitle.TextSize = 11
playerSecTitle.Font = Enum.Font.GothamBold
playerSecTitle.TextXAlignment = Enum.TextXAlignment.Left
playerSecTitle.Parent = tabPlayerFrame

local togglesPlayer = {}

local function criarToggleEstilo(pai, posY, titulo, desc, chaveEstado, callbackCustom)
	local card = Instance.new("Frame")
	card.Size = UDim2.new(1, -15, 0, 48)
	card.Position = UDim2.fromOffset(0, posY)
	card.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
	card.BorderSizePixel = 0
	card.Parent = pai
	Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)

	local lblTitle = Instance.new("TextLabel")
	lblTitle.Size = UDim2.fromOffset(230, 18)
	lblTitle.Position = UDim2.fromOffset(12, 6)
	lblTitle.BackgroundTransparency = 1
	lblTitle.Text = titulo
	lblTitle.TextColor3 = TEMAS[TEMA_ATUAL].texto
	lblTitle.TextSize = 12
	lblTitle.Font = Enum.Font.GothamBold
	lblTitle.TextXAlignment = Enum.TextXAlignment.Left
	lblTitle.Parent = card

	local lblDesc = Instance.new("TextLabel")
	lblDesc.Size = UDim2.fromOffset(230, 16)
	lblDesc.Position = UDim2.fromOffset(12, 24)
	lblDesc.BackgroundTransparency = 1
	lblDesc.Text = desc
	lblDesc.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
	lblDesc.TextSize = 10
	lblDesc.Font = Enum.Font.GothamMedium
	lblDesc.TextXAlignment = Enum.TextXAlignment.Left
	lblDesc.Parent = card

	local tgl = Instance.new("TextButton")
	tgl.Size = UDim2.fromOffset(50, 24)
	tgl.Position = UDim2.new(1, -62, 0.5, -12)
	tgl.BackgroundColor3 = estados[chaveEstado] and Color3.fromRGB(50, 205, 50) or TEMAS[TEMA_ATUAL].botaoOff
	tgl.Text = estados[chaveEstado] and "ON" or "OFF"
	tgl.TextColor3 = Color3.fromRGB(255, 255, 255)
	tgl.TextSize = 11
	tgl.Font = Enum.Font.GothamBold
	tgl.AutoButtonColor = false
	tgl.Parent = card
	Instance.new("UICorner", tgl).CornerRadius = UDim.new(0, 6)

	tgl.Activated:Connect(function()
		estados[chaveEstado] = not estados[chaveEstado]
		local ativo = estados[chaveEstado]
		tgl.Text = ativo and "ON" or "OFF"
		tgl.BackgroundColor3 = ativo and Color3.fromRGB(50, 205, 50) or TEMAS[TEMA_ATUAL].botaoOff
		if callbackCustom then callbackCustom(ativo) end
	end)

	table.insert(togglesPlayer, {card = card, lblTitle = lblTitle, lblDesc = lblDesc, tgl = tgl})
end

criarToggleEstilo(tabPlayerFrame, 25, "WalkSpeed", "Altera a velocidade do personagem", "WalkSpeed", function(val)
	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	if hum then hum.WalkSpeed = val and configs.WalkSpeedVal or 16 end
end)

criarToggleEstilo(tabPlayerFrame, 80, "JumpPower", "Altera a altura do pulo", "JumpPower", function(val)
	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	if hum then
		hum.UseJumpPower = true
		hum.JumpPower = val and configs.JumpPowerVal or 50
	end
end)

criarToggleEstilo(tabPlayerFrame, 135, "Infinite Jump", "Pula continuamente no ar", "InfJump", function(val)
	estados.InfJump = val
end)

UserInputService.JumpRequest:Connect(function()
	if estados.InfJump then
		local char = player.Character
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end)

-- =========================================
-- ABA MISC - INFO DO PLAYER + TEMAS
-- =========================================

local miscTitle = Instance.new("TextLabel")
miscTitle.Size = UDim2.new(1, 0, 0, 20)
miscTitle.Position = UDim2.fromOffset(0, 0)
miscTitle.BackgroundTransparency = 1
miscTitle.Text = "📊 INFORMAÇÕES"
miscTitle.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
miscTitle.TextSize = 11
miscTitle.Font = Enum.Font.GothamBold
miscTitle.TextXAlignment = Enum.TextXAlignment.Left
miscTitle.Parent = tabMiscFrame

local infoCards = {}

local function criarInfoCard(posY, icone, titulo)
	local card = Instance.new("Frame")
	card.Size = UDim2.new(1, -15, 0, 42)
	card.Position = UDim2.fromOffset(0, posY)
	card.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
	card.BorderSizePixel = 0
	card.Parent = tabMiscFrame
	Instance.new("UICorner", card).CornerRadius = UDim.new(0, 8)

	local iconLbl = Instance.new("TextLabel")
	iconLbl.Size = UDim2.fromOffset(35, 35)
	iconLbl.Position = UDim2.fromOffset(8, 4)
	iconLbl.BackgroundTransparency = 1
	iconLbl.Text = icone
	iconLbl.TextSize = 20
	iconLbl.Font = Enum.Font.GothamBold
	iconLbl.Parent = card

	local titleLbl = Instance.new("TextLabel")
	titleLbl.Size = UDim2.fromOffset(120, 14)
	titleLbl.Position = UDim2.fromOffset(48, 6)
	titleLbl.BackgroundTransparency = 1
	titleLbl.Text = titulo
	titleLbl.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
	titleLbl.TextSize = 9
	titleLbl.Font = Enum.Font.GothamBold
	titleLbl.TextXAlignment = Enum.TextXAlignment.Left
	titleLbl.Parent = card

	local valueLbl = Instance.new("TextLabel")
	valueLbl.Size = UDim2.new(1, -55, 0, 20)
	valueLbl.Position = UDim2.fromOffset(48, 18)
	valueLbl.BackgroundTransparency = 1
	valueLbl.Text = "..."
	valueLbl.TextColor3 = TEMAS[TEMA_ATUAL].primaria
	valueLbl.TextSize = 12
	valueLbl.Font = Enum.Font.GothamBold
	valueLbl.TextXAlignment = Enum.TextXAlignment.Left
	valueLbl.TextTruncate = Enum.TextTruncate.AtEnd
	valueLbl.Parent = card

	card.MouseEnter:Connect(function()
		TweenService:Create(card, TweenInfo.new(0.15), {
			BackgroundColor3 = Color3.new(
				math.min(TEMAS[TEMA_ATUAL].card.R + 0.05, 1),
				math.min(TEMAS[TEMA_ATUAL].card.G + 0.05, 1),
				math.min(TEMAS[TEMA_ATUAL].card.B + 0.05, 1)
			)
		}):Play()
	end)
	card.MouseLeave:Connect(function()
		TweenService:Create(card, TweenInfo.new(0.15), { BackgroundColor3 = TEMAS[TEMA_ATUAL].card }):Play()
	end)

	table.insert(infoCards, {card = card, titleLbl = titleLbl, valueLbl = valueLbl})
	return valueLbl
end

local fpsLbl = criarInfoCard(25, "🎮", "FPS")
local timeLbl = criarInfoCard(72, "⏱️", "TEMPO NO SERVIDOR")
local playersLbl = criarInfoCard(119, "👥", "JOGADORES NO SERVIDOR")
local jobLbl = criarInfoCard(166, "🆔", "JOB ID")

task.spawn(function()
	local startTime = tick()
	local frameCount = 0
	local lastFrameTime = tick()
	local fps = 60

	RunService.RenderStepped:Connect(function()
		frameCount = frameCount + 1
		local now = tick()
		if now - lastFrameTime >= 1 then
			fps = frameCount / (now - lastFrameTime)
			frameCount = 0
			lastFrameTime = now
		end
	end)

	while tabMiscFrame.Parent do
		fpsLbl.Text = string.format("%d FPS", math.floor(fps + 0.5))

		local elapsed = tick() - startTime
		local h = math.floor(elapsed / 3600)
		local m = math.floor((elapsed % 3600) / 60)
		local s = math.floor(elapsed % 60)
		timeLbl.Text = string.format("%02d:%02d:%02d", h, m, s)

		local total = #Players:GetPlayers()
		playersLbl.Text = string.format("%d / %d", total, Players.MaxPlayers or 0)

		local jobId = game.JobId
		if jobId == "" then jobId = "Servidor Privado" end
		jobLbl.Text = jobId

		task.wait(0.5)
	end
end)

-- =========================================
-- PERSONALIZAÇÃO DE TEMA
-- =========================================

local themeTitle = Instance.new("TextLabel")
themeTitle.Size = UDim2.new(1, 0, 0, 20)
themeTitle.Position = UDim2.fromOffset(0, 218)
themeTitle.BackgroundTransparency = 1
themeTitle.Text = "🎨 PERSONALIZAÇÃO DO HUB"
themeTitle.TextColor3 = TEMAS[TEMA_ATUAL].textoFraco
themeTitle.TextSize = 11
themeTitle.Font = Enum.Font.GothamBold
themeTitle.TextXAlignment = Enum.TextXAlignment.Left
themeTitle.Parent = tabMiscFrame

local themeButtons = {}

local function aplicarTema(index)
	TEMA_ATUAL = index
	local T = TEMAS[index]

	TweenService:Create(main, TweenInfo.new(0.35, Enum.EasingStyle.Quad), {
		BackgroundColor3 = T.bg1
	}):Play()
	TweenService:Create(topBar, TweenInfo.new(0.35, Enum.EasingStyle.Quad), {
		BackgroundColor3 = T.bg2
	}):Play()
	TweenService:Create(mainStroke, TweenInfo.new(0.35), {
		Color = T.stroke
	}):Play()

	-- Atualiza cor das letras do título
	for _, lbl in ipairs(labelsLetras) do
		TweenService:Create(lbl, TweenInfo.new(0.35), {
			TextColor3 = T.primaria
		}):Play()
	end

	status.TextColor3 = ATIVO and Color3.fromRGB(80,255,140) or Color3.fromRGB(255, 80, 110)

	if not ATIVO then
		pullButton.BackgroundColor3 = T.botao
	end

	kaCard.BackgroundColor3 = T.card
	kaTitle.TextColor3 = T.texto
	kaDesc.TextColor3 = T.textoFraco
	if not KILL_AURA_ATIVO then
		kaToggle.BackgroundColor3 = T.botaoOff
	end
	kaSubFrame.BackgroundColor3 = T.card

	for _, st in ipairs(subToggles) do
		st.lbl.TextColor3 = T.texto
		st.descLbl.TextColor3 = T.textoFraco
		if st.tgl.Text == "OFF" then
			st.tgl.BackgroundColor3 = T.botaoOff
		end
	end

	distanceTitle.TextColor3 = T.textoFraco
	distanceValue.TextColor3 = T.primaria
	fill.BackgroundColor3 = T.botao
	minusDistance.BackgroundColor3 = T.card
	plusDistance.BackgroundColor3 = T.card

	speedTitle.TextColor3 = T.textoFraco
	speedValue.TextColor3 = T.primaria
	speedFill.BackgroundColor3 = T.botao
	minusSpeed.BackgroundColor3 = T.card
	plusSpeed.BackgroundColor3 = T.card

	for _, b in ipairs(botoesSidebar) do
		if b.btn.TextColor3 == Color3.new(1,1,1) then
			b.btn.BackgroundColor3 = T.sidebarAtiva
		else
			b.btn.BackgroundColor3 = T.sidebarInativa
			b.btn.TextColor3 = T.textoFraco
		end
		b.indicator.BackgroundColor3 = T.primaria
	end

	for _, tp in ipairs(togglesPlayer) do
		tp.card.BackgroundColor3 = T.card
		tp.lblTitle.TextColor3 = T.texto
		tp.lblDesc.TextColor3 = T.textoFraco
		if tp.tgl.Text == "OFF" then
			tp.tgl.BackgroundColor3 = T.botaoOff
		end
	end

	miscTitle.TextColor3 = T.textoFraco
	themeTitle.TextColor3 = T.textoFraco
	for _, ic in ipairs(infoCards) do
		ic.card.BackgroundColor3 = T.card
		ic.titleLbl.TextColor3 = T.textoFraco
		ic.valueLbl.TextColor3 = T.primaria
	end

	for i, tb in ipairs(themeButtons) do
		if i == index - 1 then
			tb.stroke.Transparency = 0
			tb.stroke.Thickness = 2
			tb.stroke.Color = T.primaria
		else
			tb.stroke.Transparency = 0.7
			tb.stroke.Thickness = 1
		end
	end

	tabMainFrame.ScrollBarImageColor3 = T.stroke
	tabPlayerFrame.ScrollBarImageColor3 = T.stroke
	tabMiscFrame.ScrollBarImageColor3 = T.stroke
end

local themeColors = {
	{ Color3.fromRGB(10, 10, 10), Color3.fromRGB(255, 255, 255) },
	{ Color3.fromRGB(140, 50, 200), Color3.fromRGB(255, 120, 200) },
	{ Color3.fromRGB(30, 130, 60), Color3.fromRGB(120, 255, 150) },
	{ Color3.fromRGB(30, 80, 180), Color3.fromRGB(120, 180, 255) }
}

local themeNames = {
	"Preto e Branco",
	"Roxo e Rosa",
	"Verde e Verde Escuro",
	"Azul e Azul Escuro"
}

for i, nome in ipairs(themeNames) do
	local rowY = 245 + (i - 1) * 42

	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -15, 0, 36)
	btn.Position = UDim2.fromOffset(0, rowY)
	btn.BackgroundColor3 = TEMAS[TEMA_ATUAL].card
	btn.BorderSizePixel = 0
	btn.AutoButtonColor = false
	btn.Text = ""
	btn.Parent = tabMiscFrame
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

	local strokeBtn = Instance.new("UIStroke")
	strokeBtn.Color = TEMAS[TEMA_ATUAL].primaria
	strokeBtn.Thickness = 1
	strokeBtn.Transparency = (i == 1) and 0 or 0.7
	strokeBtn.Parent = btn

	local c1 = Instance.new("Frame")
	c1.Size = UDim2.fromOffset(22, 22)
	c1.Position = UDim2.fromOffset(10, 7)
	c1.BackgroundColor3 = themeColors[i][1]
	c1.BorderSizePixel = 0
	c1.Parent = btn
	Instance.new("UICorner", c1).CornerRadius = UDim.new(1, 0)

	local c2 = Instance.new("Frame")
	c2.Size = UDim2.fromOffset(22, 22)
	c2.Position = UDim2.fromOffset(28, 7)
	c2.BackgroundColor3 = themeColors[i][2]
	c2.BorderSizePixel = 0
	c2.Parent = btn
	Instance.new("UICorner", c2).CornerRadius = UDim.new(1, 0)

	local lbl = Instance.new("TextLabel")
	lbl.Size = UDim2.new(1, -80, 1, 0)
	lbl.Position = UDim2.fromOffset(60, 0)
	lbl.BackgroundTransparency = 1
	lbl.Text = nome
	lbl.TextColor3 = TEMAS[TEMA_ATUAL].texto
	lbl.TextSize = 12
	lbl.Font = Enum.Font.GothamBold
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.Parent = btn

	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.15), {
			BackgroundColor3 = Color3.new(
				math.min(TEMAS[TEMA_ATUAL].card.R + 0.08, 1),
				math.min(TEMAS[TEMA_ATUAL].card.G + 0.08, 1),
				math.min(TEMAS[TEMA_ATUAL].card.B + 0.08, 1)
			)
		}):Play()
		TweenService:Create(btn, TweenInfo.new(0.15), {
			Size = UDim2.new(1, -10, 0, 36)
		}):Play()
	end)
	btn.MouseLeave:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.15), {
			BackgroundColor3 = TEMAS[TEMA_ATUAL].card
		}):Play()
		TweenService:Create(btn, TweenInfo.new(0.15), {
			Size = UDim2.new(1, -15, 0, 36)
		}):Play()
	end)

	btn.Activated:Connect(function()
		local themeIndex = i + 1
		aplicarTema(themeIndex)

		TweenService:Create(btn, TweenInfo.new(0.08), {
			Size = UDim2.new(1, -20, 0, 34)
		}):Play()
		task.wait(0.08)
		TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Back), {
			Size = UDim2.new(1, -15, 0, 36)
		}):Play()
	end)

	table.insert(themeButtons, {btn = btn, stroke = strokeBtn, lbl = lbl})
end

-- =========================================
-- MINIMIZAR
-- =========================================

minimizeBtn.Activated:Connect(function()
	MINIMIZADO = not MINIMIZADO
	for _, child in ipairs(main:GetChildren()) do
		if child ~= topBar and child ~= mainCorner and child ~= mainStroke then
			child.Visible = not MINIMIZADO
		end
	end
	local alvo = MINIMIZADO and UDim2.fromOffset(600, 45) or UDim2.fromOffset(600, 400)
	TweenService:Create(main, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
		Size = alvo
	}):Play()
end)

atualizarHub()
