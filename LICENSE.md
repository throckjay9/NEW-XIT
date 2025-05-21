-- FFH4X DEV PANEL - GUI funcional e completo
local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local camera = workspace.CurrentCamera

-- Criar GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "FFH4X_GUI"
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame", screenGui)
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 300, 0, 400)
frame.Position = UDim2.new(0.5, -150, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Visible = false

-- Drag label
local dragLabel = Instance.new("TextLabel", frame)
dragLabel.Name = "DragLabel"
dragLabel.Size = UDim2.new(1, 0, 0, 30)
dragLabel.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
dragLabel.Text = "FFH4X DEV PANEL"
dragLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
dragLabel.Font = Enum.Font.GothamBold
dragLabel.TextSize = 16

-- Botão Generator
local function createButton(name, y, text)
	local btn = Instance.new("TextButton", frame)
	btn.Name = name
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, 40 + (y * 35))
	btn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
	btn.TextColor3 = Color3.fromRGB(0, 255, 0)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.Text = text
	return btn
end

-- Criar botões
local buttons = {
	{"AimbotButton", "Aimbot"},
	{"TeleportButton", "Teleport"},
	{"SpeedButton", "Speed"},
	{"JumpButton", "Jump"},
	{"ESPButton", "ESP"},
	{"FlyButton", "Fly"},
	{"GravityButton", "Gravity"},
	{"HealthButton", "Heal"}
}

for i, b in ipairs(buttons) do
	createButton(b[1], i - 1, b[2])
end

-- Teclas
local dragToggle = false
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.Q then
		frame.Visible = not frame.Visible
	elseif input.KeyCode == Enum.KeyCode.E then
		dragToggle = not dragToggle
	end
end)

-- Drag funcional
local dragging = false
local dragInput, dragStart, startPos

local function update(input)
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end

dragLabel.InputBegan:Connect(function(input)
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

dragLabel.InputChanged:Connect(function(input)
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
frame.AimbotButton.MouseButton1Click:Connect(aimbot)
frame.TeleportButton.MouseButton1Click:Connect(teleport)
frame.SpeedButton.MouseButton1Click:Connect(speed)
frame.JumpButton.MouseButton1Click:Connect(jump)
frame.ESPButton.MouseButton1Click:Connect(esp)
frame.FlyButton.MouseButton1Click:Connect(fly)
frame.GravityButton.MouseButton1Click:Connect(gravity)
frame.HealthButton.MouseButton1Click:Connect(heal)
