-- FFH4X DEV PANEL - Para testes no mapa GARENA FREE FIRE MAX🏆
-- Feito para uso seguro e local

local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local frame = script.Parent:WaitForChild("MainFrame")

-- Inicial
frame.Visible = false
local dragging = false
local dragToggle = false
local camera = workspace.CurrentCamera

-- Abrir/fechar GUI com Q
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.Q then
		frame.Visible = not frame.Visible
	end
end)

-- Ativar/desativar arrastar com F
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.F then
		dragToggle = not dragToggle
	end
end)

-- Drag GUI
local dragArea = frame:WaitForChild("DragLabel")
local function dragGUI(gui)
	local dragInput, mousePos, startPos

	gui.InputBegan:Connect(function(input)
		if dragToggle and input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			mousePos = input.Position
			startPos = gui.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	gui.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - mousePos
			gui.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end)
end

dragGUI(frame)

-- Função: Aimbot simulado (olhar para cabeça do inimigo mais próximo)
local function aimbot()
	local closestEnemy, closestDist = nil, math.huge
	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
			local head = plr.Character.Head
			local distance = (head.Position - player.Character.Head.Position).Magnitude
			if distance < closestDist then
				closestDist = distance
				closestEnemy = head
			end
		end
	end
	if closestEnemy then
		camera.CFrame = CFrame.new(camera.CFrame.Position, closestEnemy.Position)
	end
end

-- Teleporte
local function teleport()
	local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if hrp then
		hrp.CFrame = CFrame.new(150, 15, 150) -- ajuste conforme o mapa
	end
end

-- Velocidade
local function speed()
	local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
	if hum then hum.WalkSpeed = 100 end
end

-- Pulo
local function jump()
	local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
	if hum then hum.JumpPower = 150 end
end

-- ESP
local function esp()
	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= player and plr.Character and not plr.Character:FindFirstChild("FFH4X_ESP") then
			local hl = Instance.new("Highlight")
			hl.Name = "FFH4X_ESP"
			hl.FillColor = Color3.fromRGB(255, 0, 0)
			hl.OutlineColor = Color3.fromRGB(255, 255, 0)
			hl.Parent = plr.Character
		end
	end
end

-- Fly
local flying = false
local bv

local function toggleFly()
	local char = player.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return end
	if not flying then
		bv = Instance.new("BodyVelocity")
		bv.Velocity = Vector3.zero
		bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
		bv.Parent = char.HumanoidRootPart
		flying = true
	else
		flying = false
		if bv then bv:Destroy() end
	end
end

RS.RenderStepped:Connect(function()
	if flying then
		local direction = Vector3.zero
		if UIS:IsKeyDown(Enum.KeyCode.W) then direction += camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.S) then direction -= camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then direction += Vector3.new(0,1,0) end
		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then direction -= Vector3.new(0,1,0) end
		if bv then bv.Velocity = direction * 50 end
	end
end)

-- Gravidade local
local function gravity()
	workspace.Gravity = 50
end

-- Cura
local function heal()
	local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
	if hum then hum.Health = hum.MaxHealth end
end

-- Conectar botões
frame:WaitForChild("AimbotButton").MouseButton1Click:Connect(aimbot)
frame:WaitForChild("TeleportButton").MouseButton1Click:Connect(teleport)
frame:WaitForChild("SpeedButton").MouseButton1Click:Connect(speed)
frame:WaitForChild("JumpButton").MouseButton1Click:Connect(jump)
frame:WaitForChild("ESPButton").MouseButton1Click:Connect(esp)
frame:WaitForChild("FlyButton").MouseButton1Click:Connect(toggleFly)
frame:WaitForChild("GravityButton").MouseButton1Click:Connect(gravity)
frame:WaitForChild("HealthButton").MouseButton1Click:Connect(heal)
