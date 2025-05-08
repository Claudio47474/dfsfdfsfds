-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- CONFIGS
_G.espEnabled = false
_G.nameEnabled = false
_G.boxEnabled = false
_G.distanceEnabled = false
_G.healthEnabled = false
_G.teamCheck = false
_G.aimbotEnabled = false
_G.linesEnabled = false
_G.espLinesEnabled = false

local aimbotKey = Enum.UserInputType.MouseButton2
local currentFOV = 150
local maxDistance = 1000

-- GUI PANEL
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "TestPanel"

local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 230, 0, 390)
panel.Position = UDim2.new(0, 50, 0, 50)
panel.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true

local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1, 0, 0, 35)
title.Text = "ESP / AIMBOT PANEL"
title.BackgroundColor3 = Color3.fromRGB(60, 0, 90)
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 16

-- MINIMIZE/CLOSE
local close = Instance.new("TextButton", panel)
close.Position = UDim2.new(1, -30, 0, 5)
close.Size = UDim2.new(0, 25, 0, 25)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(100, 0, 100)
close.TextColor3 = Color3.new(1, 1, 1)
close.MouseButton1Click:Connect(function() panel.Visible = false end)

-- TOGGLES
local toggles = {}

local function createToggle(text, y, globalVar)
	local btn = Instance.new("TextButton", panel)
	btn.Position = UDim2.new(0, 10, 0, y)
	btn.Size = UDim2.new(0, 210, 0, 25)
	btn.BackgroundColor3 = Color3.fromRGB(40, 0, 60)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Text = "[OFF] " .. text
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 14

	btn.MouseButton1Click:Connect(function()
		_G[globalVar] = not _G[globalVar]
		btn.Text = (_G[globalVar] and "[ON] " or "[OFF] ") .. text
	end)

	toggles[globalVar] = btn
end

-- ADD TOGGLE BUTTONS
createToggle("ESP", 50, "espEnabled")
createToggle("Box", 80, "boxEnabled")
createToggle("Nome", 110, "nameEnabled")
createToggle("Distância", 140, "distanceEnabled")
createToggle("Vida", 170, "healthEnabled")
createToggle("Team Check", 200, "teamCheck")
createToggle("Aimbot Cabeça", 230, "aimbotEnabled")
createToggle("ESP Lines", 260, "espLinesEnabled")
createToggle("FOV +", 290, "increaseFOV")
createToggle("FOV -", 320, "decreaseFOV")

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 0, 255)
fovCircle.Thickness = 1
fovCircle.NumSides = 100
fovCircle.Filled = false
fovCircle.Visible = false

RunService.RenderStepped:Connect(function()
	fovCircle.Visible = _G.aimbotEnabled
	fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
	fovCircle.Radius = currentFOV

	if _G.increaseFOV then
		currentFOV = math.clamp(currentFOV + 2, 10, 500)
	end
	if _G.decreaseFOV then
		currentFOV = math.clamp(currentFOV - 2, 10, 500)
	end
end)

-- Drawing Cleanup
local drawings = {}

local function clearDrawings()
	for _, v in pairs(drawings) do
		for _, obj in pairs(v) do
			if typeof(obj) == "table" then
				for _, part in pairs(obj) do part:Remove() end
			else
				obj:Remove()
			end
		end
	end
	drawings = {}
end

-- ESP + LINES
RunService.RenderStepped:Connect(function()
	if not _G.espEnabled then clearDrawings() return end
	clearDrawings()

	local localChar = LocalPlayer.Character
	if not (localChar and localChar:FindFirstChild("HumanoidRootPart")) then return end
	local localPos = localChar.HumanoidRootPart.Position

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			if _G.teamCheck and player.Team == LocalPlayer.Team then continue end

			local char = player.Character
			local hrp = char:FindFirstChild("HumanoidRootPart")
			local hum = char:FindFirstChild("Humanoid")
			if not (hrp and hum) then continue end

			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			local dist = (hrp.Position - localPos).Magnitude

			if onScreen and dist < maxDistance then
				local elements = {}

				-- Box
				if _G.boxEnabled then
					local box = Drawing.new("Square")
					box.Size = Vector2.new(60, 80)
					box.Position = Vector2.new(pos.X - 30, pos.Y - 80)
					box.Color = Color3.fromRGB(255, 0, 255)
					box.Thickness = 1
					box.Filled = false
					box.Visible = true
					elements.Box = box
				end

				-- Name + Distance
				if _G.nameEnabled or _G.distanceEnabled then
					local text = Drawing.new("Text")
					text.Text = player.Name
					if _G.distanceEnabled then
						text.Text = string.format("%s | %.0fm", player.Name, dist)
					end
					text.Position = Vector2.new(pos.X - (#text.Text * 3), pos.Y - 85)
					text.Color = Color3.new(1, 1, 1)
					text.Size = 14
					text.Outline = true
					text.Visible = true
					elements.Text = text
				end

				-- Line from local player to target
				if _G.espLinesEnabled then
					local from3D = localChar.HumanoidRootPart.Position
					local fromScreen, v1 = Camera:WorldToViewportPoint(from3D)
					local toScreen, v2 = Camera:WorldToViewportPoint(hrp.Position)
					if v1 and v2 then
						local line = Drawing.new("Line")
						line.From = Vector2.new(fromScreen.X, fromScreen.Y)
						line.To = Vector2.new(toScreen.X, toScreen.Y)
						line.Color = Color3.fromRGB(255, 255, 0)
						line.Thickness = 1
						line.Visible = true
						elements.Line = line
					end
				end

				table.insert(drawings, {elements})
			end
		end
	end
end)

-- AIMBOT
RunService.RenderStepped:Connect(function()
	if not _G.aimbotEnabled then return end
	if not UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then return end

	local closest, distance = nil, currentFOV

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			if _G.teamCheck and player.Team == LocalPlayer.Team then continue end
			local hrp = player.Character.HumanoidRootPart
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			if onScreen then
				local mousePos = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
				local targetPos = Vector2.new(pos.X, pos.Y)
				local diff = (mousePos - targetPos).Magnitude
				if diff < distance then
					distance = diff
					closest = player
				end
			end
		end
	end

	if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
		local target = closest.Character.HumanoidRootPart.Position
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, target)
	end
end)
