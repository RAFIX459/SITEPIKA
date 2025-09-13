-- Rafi_X Hub

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "Rafi_XHub"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 300)
frame.Position = UDim2.new(0.5, -125, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Parent = screenGui

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.Text = "Rafi_X Hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Parent = frame

-- Botão fechar
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
closeButton.Text = "X"
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 18
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Parent = frame

-- Botão abrir
local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0, 100, 0, 40)
openButton.Position = UDim2.new(0, 10, 0, 10)
openButton.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
openButton.Text = "Abrir GUI"
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 18
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.Visible = false
openButton.Parent = screenGui

-- Função fechar/abrir
closeButton.MouseButton1Click:Connect(function()
	frame.Visible = false
	openButton.Visible = true
end)
openButton.MouseButton1Click:Connect(function()
	frame.Visible = true
	openButton.Visible = false
end)

-- Botões principais
local function createButton(name, pos)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, -20, 0, 40)
	button.Position = UDim2.new(0, 10, 0, pos)
	button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.GothamBold
	button.TextSize = 18
	button.Text = name
	button.Parent = frame
	return button
end

local jumpButton = createButton("Super Pulo", 50)
local flyButton = createButton("Flutuar", 100)
local speedButton = createButton("Velocidade", 150)
local fpsButton = createButton("Remover Texturas", 200)

-- Contador de FPS
local fpsLabel = Instance.new("TextLabel")
fpsLabel.Size = UDim2.new(0, 120, 0, 30)
fpsLabel.Position = UDim2.new(1, -130, 0, 10)
fpsLabel.BackgroundTransparency = 1
fpsLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
fpsLabel.Font = Enum.Font.GothamBold
fpsLabel.TextSize = 18
fpsLabel.Text = "FPS: 0"
fpsLabel.Parent = screenGui

local lastUpdate = tick()
local frames = 0
RunService.RenderStepped:Connect(function()
	frames += 1
	if tick() - lastUpdate >= 1 then
		fpsLabel.Text = "FPS: " .. frames
		frames = 0
		lastUpdate = tick()
	end
end)

-- ================== FUNÇÕES ==================

-- Super Pulo (toggle)
local superJumpActive = false
local superJumpConnection

jumpButton.MouseButton1Click:Connect(function()
	superJumpActive = not superJumpActive
	if superJumpActive then
		jumpButton.Text = "Super Pulo Ativo"
		superJumpConnection = UserInputService.JumpRequest:Connect(function()
			local bv = Instance.new("BodyVelocity")
			bv.Velocity = Vector3.new(0, 200, 0)
			bv.MaxForce = Vector3.new(0, math.huge, 0)
			bv.Parent = rootPart
			Debris:AddItem(bv, 0.2)
		end)
	else
		jumpButton.Text = "Super Pulo"
		if superJumpConnection then
			superJumpConnection:Disconnect()
			superJumpConnection = nil
		end
	end
end)

-- Flutuar (ficar parado no ar)
local flying = false
flyButton.MouseButton1Click:Connect(function()
	flying = not flying
	if flying then
		flyButton.Text = "Flutuar Ativo"
		local bv = Instance.new("BodyVelocity")
		bv.Name = "FlyForce"
		bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
		bv.Velocity = Vector3.new(0, 0, 0) -- parado no ar
		bv.Parent = rootPart
	else
		flyButton.Text = "Flutuar"
		if rootPart:FindFirstChild("FlyForce") then
			rootPart.FlyForce:Destroy()
		end
	end
end)

-- Velocidade segura (toggle)
local speedActive = false
local speedBodyVelocity

speedButton.MouseButton1Click:Connect(function()
	speedActive = not speedActive
	if speedActive then
		speedButton.Text = "Velocidade Ativa"
		if not speedBodyVelocity then
			speedBodyVelocity = Instance.new("BodyVelocity")
			speedBodyVelocity.MaxForce = Vector3.new(math.huge, 0, math.huge)
			speedBodyVelocity.Velocity = Vector3.new(0, 0, 0)
			speedBodyVelocity.Parent = rootPart
		end
	else
		speedButton.Text = "Velocidade"
		if speedBodyVelocity then
			speedBodyVelocity:Destroy()
			speedBodyVelocity = nil
		end
	end
end)

RunService.RenderStepped:Connect(function()
	if speedActive and speedBodyVelocity then
		local dir = humanoid.MoveDirection
		speedBodyVelocity.Velocity = dir * 150
	end
end)

-- Remover Texturas (toggle)
local texturesRemoved = false
local removedObjects = {}

fpsButton.MouseButton1Click:Connect(function()
	texturesRemoved = not texturesRemoved
	if texturesRemoved then
		fpsButton.Text = "Texturas Removidas"
		for _, v in pairs(workspace:GetDescendants()) do
			if v:IsA("Texture") or v:IsA("Decal") then
				table.insert(removedObjects, {obj = v, parent = v.Parent})
				v.Parent = nil
			end
		end
	else
		fpsButton.Text = "Remover Texturas"
		for _, data in ipairs(removedObjects) do
			if data.obj and data.parent then
				data.obj.Parent = data.parent
			end
		end
		removedObjects = {}
	end
end)
