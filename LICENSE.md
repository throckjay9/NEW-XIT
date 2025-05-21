-- FFH4X DEV PANEL - GUI Arrastável Completo
local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local camera = workspace.CurrentCamera

-- Criar GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "FFH4X_GUI"
gui.ResetOnSpawn = false

-- Painel principal
local frame = Instance.new("Frame", gui)
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 280, 0, 400)
frame.Position = UDim2.new(0.5, -140, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Visible = false

-- Cabeçalho (barra de arraste)
local dragHeader = Instance.new("TextLabel", frame)
dragHeader.Name = "DragLabel"
dragHeader.Size = UDim2.new(1, 0, 0, 30)
dragHeader.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
dragHeader.Text = "FFH4X DEV PANEL"
dragHeader.TextColor3 = Color3.fromRGB(255, 255, 0)
dragHeader.Font = Enum.Font.GothamBold
dragHeader.TextSize = 16

-- Botão criador
local function createButton(text, index)
	local btn = Instance.new("TextButton", frame)
	btn.Name = text .. "Button"
	btn.Text = text
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, 40 + ((index - 1) * 35))
	btn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
	btn.TextColor3 = Color3.fromRGB(0, 255, 0)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	return btn
end

-- Criar botões
local buttonNames = {
	"Aimbot", "Teleport", "Speed", "Jump", "ESP", "Fly", "Gravity", "Heal"
}

local buttons = {}
for i, name in ipairs(buttonNames) do
	buttons[name] = createButton(name, i)
end

-- Abrir/fechar GUI com Q
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.Q then
		frame.Visible = not frame.Visible
	end
end)

-- Ativar modo arrastar com tecla E
local dragToggle = false
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then
		dragToggle = not dragToggle
	end
end)

-- Lógica de arrastar
local dragging, dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end

dragHeader.InputBegan:Connect(function(input)
	if dragToggle and input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

dragHeader.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)

UIS.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		update(input)
	end
end)

-- Funções dos botões
local function getHumanoid()
	return player.Character and player.Character:FindFirstChildOfClass("Humanoid")
end

local function getRoot()
	return player.Character and player.Character:FindFirstChild("HumanoidRootPart")
end

local function aimbot()
	local closest, dist = nil, math.huge
	for _, plr in pairs(game.Players:GetPlayers()) do
		if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
			local head = plr.Character.Head
			local d = (head.Position - player.Character.Head.Position).Magnitude
			if d < dist then
				dist = d
				closest = head
			end
		end
	end
	if closest then
		camera.CFrame = CFrame.new(camera.CFrame.Position, closest.Position)
	end
end

local function teleport()
	local hrp = getRoot()
	if hrp then
		hrp.CFrame = CFrame.new(150, 15, 150)
	end
end

local function speed()
	local hum = getHumanoid()
	if hum then hum.WalkSpeed = 100 end
end

local function jump()
	local hum = getHumanoid()
	if hum then hum.JumpPower = 150 end
end

local function esp()
	for _, plr in pairs(game.Players:GetPlayers()) do
		if plr ~= player and plr.Character and not plr.Character:FindFirstChild("FFH4X_ESP") then
			local hl = Instance.new("Highlight")
			hl.Name = "FFH4X_ESP"
			hl.FillColor = Color3.fromRGB(255, 0, 0)
			hl.OutlineColor = Color3.fromRGB(255, 255, 0)
			hl.Parent = plr.Character
		end
	end
end

local flying = false
local bv

local function fly()
	local root = getRoot()
	if not root then return end
	if not flying then
		bv = Instance.new("BodyVelocity")
		bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		bv.Velocity = Vector3.zero
		bv.Parent = root
		flying = true
	else
		if bv then bv:Destroy() end
		flying = false
	end
end

RS.RenderStepped:Connect(function()
	if flying and bv then
		local dir = Vector3.zero
		if UIS:IsKeyDown(Enum.KeyCode.W) then dir += camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
		bv.Velocity = dir * 50
	end
end)

local function gravity()
	workspace.Gravity = 50
end

local function heal()
	local hum = getHumanoid()
	if hum then hum.Health = hum.MaxHealth end
end

-- Conectar botões
buttons.Aimbot.MouseButton1Click:Connect(aimbot)
buttons.Teleport.MouseButton1Click:Connect(teleport)
buttons.Speed.MouseButton1Click:Connect(speed)
buttons.Jump.MouseButton1Click:Connect(jump)
buttons.ESP.MouseButton1Click:Connect(esp)
buttons.Fly.MouseButton1Click:Connect(fly)
buttons.Gravity.MouseButton1Click:Connect(gravity)
buttons.Heal.MouseButton1Click:Connect(heal)
