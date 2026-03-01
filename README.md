local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "ItemTrollGUI"
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 250, 0, 400)
Frame.Position = UDim2.new(0.1, 0, 0.1, 0)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "🎯 Troll Target Player"
Title.TextColor3 = Color3.new(1,1,1)
Title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 16

-- SLIDER UNTUK RADIUS
local radiusLabel = Instance.new("TextLabel", Frame)
radiusLabel.Position = UDim2.new(0, 10, 0, 35)
radiusLabel.Size = UDim2.new(0, 200, 0, 20)
radiusLabel.Text = "Radius: 5"
radiusLabel.TextColor3 = Color3.new(1,1,1)
radiusLabel.BackgroundTransparency = 1
radiusLabel.Font = Enum.Font.SourceSans
radiusLabel.TextSize = 14

local radius = 5
local radiusSlider = Instance.new("TextBox", Frame)
radiusSlider.Position = UDim2.new(0, 10, 0, 55)
radiusSlider.Size = UDim2.new(0, 100, 0, 25)
radiusSlider.Text = tostring(radius)
radiusSlider.BackgroundColor3 = Color3.fromRGB(60,60,60)
radiusSlider.TextColor3 = Color3.new(1,1,1)
radiusSlider.Font = Enum.Font.SourceSans
radiusSlider.TextSize = 14

radiusSlider.FocusLost:Connect(function()
	local value = tonumber(radiusSlider.Text)
	if value then
		radius = math.clamp(value, 1, 50)
		radiusLabel.Text = "Radius: " .. tostring(radius)
	end
end)

-- SLIDER UNTUK SPEED
local speedLabel = Instance.new("TextLabel", Frame)
speedLabel.Position = UDim2.new(0, 10, 0, 85)
speedLabel.Size = UDim2.new(0, 200, 0, 20)
speedLabel.Text = "Speed: 70"
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.BackgroundTransparency = 1
speedLabel.Font = Enum.Font.SourceSans
speedLabel.TextSize = 14

local speed = 70
local speedSlider = Instance.new("TextBox", Frame)
speedSlider.Position = UDim2.new(0, 10, 0, 105)
speedSlider.Size = UDim2.new(0, 100, 0, 25)
speedSlider.Text = tostring(speed)
speedSlider.BackgroundColor3 = Color3.fromRGB(60,60,60)
speedSlider.TextColor3 = Color3.new(1,1,1)
speedSlider.Font = Enum.Font.SourceSans
speedSlider.TextSize = 14

speedSlider.FocusLost:Connect(function()
	local value = tonumber(speedSlider.Text)
	if value then
		speed = math.clamp(value, 1, 300)
		speedLabel.Text = "Speed: " .. tostring(speed)
	end
end)

-- SCROLLING PLAYER LIST
local Scrolling = Instance.new("ScrollingFrame", Frame)
Scrolling.Position = UDim2.new(0, 0, 0, 140)
Scrolling.Size = UDim2.new(1, 0, 0, 210)
Scrolling.CanvasSize = UDim2.new(0, 0, 0, 0)
Scrolling.ScrollBarThickness = 6
Scrolling.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Scrolling.BorderSizePixel = 0

-- STOP BUTTON
local stopBtn = Instance.new("TextButton", Frame)
stopBtn.Size = UDim2.new(1, 0, 0, 30)
stopBtn.Position = UDim2.new(0, 0, 1, -30)
stopBtn.BackgroundColor3 = Color3.fromRGB(170, 50, 50)
stopBtn.TextColor3 = Color3.new(1, 1, 1)
stopBtn.Font = Enum.Font.SourceSansBold
stopBtn.Text = "🛑 Stop Troll"
stopBtn.TextSize = 14

-- VARIABEL
local trolling = false
local trollConnection

-- FUNGSI: Ambil part yang bisa digerakkan
local function getAllMovableParts()
	local parts = {}
	for _, obj in ipairs(Workspace:GetDescendants()) do
		if obj:IsA("Part") and obj.Anchored == false then
			table.insert(parts, obj)
		end
	end
	return parts
end

-- FUNGSI: Mulai Troll
local function startTroll(targetPlayer)
	if trolling then return end
	trolling = true

	local parts = getAllMovableParts()
	trollConnection = RunService.Heartbeat:Connect(function()
		if not (targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart")) then
			return
		end

		local pos = targetPlayer.Character.HumanoidRootPart.Position
		for _, part in ipairs(parts) do
			if part and part.Parent then
				local offset = Vector3.new(
					math.random(-radius, radius),
					math.random(1, radius),
					math.random(-radius, radius)
				)
				local goal = pos + offset
				local dir = (goal - part.Position).Unit
				local force = dir * speed
				part:ApplyImpulse(force * part:GetMass())
			end
		end
	end)
end

-- FUNGSI: Stop Troll
stopBtn.MouseButton1Click:Connect(function()
	trolling = false
	if trollConnection then
		trollConnection:Disconnect()
		trollConnection = nil
	end
end)

-- FUNGSI: Tampilkan List Pemain
local function updatePlayerList()
	Scrolling:ClearAllChildren()
	local y = 0

	for _, plr in ipairs(Players:GetPlayers()) do
		local btn = Instance.new("TextButton", Scrolling)
		btn.Size = UDim2.new(1, 0, 0, 30)
		btn.Position = UDim2.new(0, 0, 0, y)
		btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
		btn.TextColor3 = Color3.new(1, 1, 1)
		btn.Font = Enum.Font.SourceSans
		btn.TextSize = 14
		btn.Text = plr.Name

		btn.MouseButton1Click:Connect(function()
			startTroll(plr)
		end)

		y = y + 30
	end

	Scrolling.CanvasSize = UDim2.new(0, 0, 0, y)
end

Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)

updatePlayerList()
