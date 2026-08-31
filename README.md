-- thelastcoowner | Anti Drop + Anti Die/Reset
-- Combined & cleaned

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local spoofedVelocity = Vector3.zero
local speedValue = 59
local antiDropEnabled = false

-- ====================== VELOCITY SPOOF ======================
local oldIndex
oldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
	if not checkcaller() and (key == "AssemblyLinearVelocity" or key == "Velocity") then
		if typeof(self) == "Instance" and self:IsA("BasePart") and self.Name == "HumanoidRootPart" and self:IsDescendantOf(LocalPlayer.Character) then
			return spoofedVelocity
		end
	end
	return oldIndex(self, key)
end))

local oldNewIndex
oldNewIndex = hookmetamethod(game, "__newindex", newcclosure(function(self, key, value)
	if not checkcaller() and (key == "AssemblyLinearVelocity" or key == "Velocity") then
		if typeof(self) == "Instance" and self:IsA("BasePart") and self.Name == "HumanoidRootPart" and self:IsDescendantOf(LocalPlayer.Character) then
			spoofedVelocity = value
			return
		end
	end
	return oldNewIndex(self, key, value)
end))

local function applyVelocitySpeed(speed)
	if not antiDropEnabled then return end

	local char = LocalPlayer.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if not char or not hum or not root or hum.Health <= 0 then return end

	local dir = hum.MoveDirection
	if dir.Magnitude > 0.05 then
		pcall(function()
			if root.SetNetworkOwner then
				root:SetNetworkOwner(LocalPlayer)
			end
		end)
		local unit = dir.Unit
		spoofedVelocity = Vector3.new(unit.X * 16, root.AssemblyLinearVelocity.Y, unit.Z * 16)
		root.AssemblyLinearVelocity = Vector3.new(unit.X * speed, root.AssemblyLinearVelocity.Y, unit.Z * speed)
	else
		spoofedVelocity = Vector3.new(0, root.AssemblyLinearVelocity.Y, 0)
	end
end

RunService.PreSimulation:Connect(function()
	applyVelocitySpeed(speedValue)
end)

-- ====================== ANTI DIE / ANTI RESET ======================
local function setupAntiDie(char)
	local humanoid = char:WaitForChild("Humanoid", 5)
	if not humanoid then return end

	-- Block death / reset
	humanoid:GetPropertyChangedSignal("Health"):Connect(function()
		if antiDropEnabled and humanoid.Health < humanoid.MaxHealth then
			humanoid.Health = humanoid.MaxHealth
		end
	end)

	humanoid.Died:Connect(function()
		if antiDropEnabled then
			task.wait()
			humanoid.Health = humanoid.MaxHealth
		end
	end)

	-- Prevent reset button from working while enabled
	pcall(function()
		local StarterGui = game:GetService("StarterGui")
		StarterGui:SetCore("ResetButtonCallback", false)
	end)
end

if LocalPlayer.Character then
	setupAntiDie(LocalPlayer.Character)
end

LocalPlayer.CharacterAdded:Connect(function(char)
	setupAntiDie(char)
	if antiDropEnabled then
		pcall(function()
			game:GetService("StarterGui"):SetCore("ResetButtonCallback", false)
		end)
	end
end)

-- ====================== GUI ======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "thelastcoowner"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.ClipsDescendants = true
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.Size = UDim2.new(0, 260, 0, 120)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(255, 255, 255)
MainStroke.Thickness = 1.4
MainStroke.Parent = MainFrame

-- Topbar
local Topbar = Instance.new("Frame")
Topbar.Name = "Topbar"
Topbar.Size = UDim2.new(1, 0, 0, 36)
Topbar.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Topbar.BorderSizePixel = 0
Topbar.Parent = MainFrame

local TopbarCorner = Instance.new("UICorner")
TopbarCorner.CornerRadius = UDim.new(0, 10)
TopbarCorner.Parent = Topbar

local TopbarExtension = Instance.new("Frame")
TopbarExtension.Position = UDim2.new(0, 0, 1, -10)
TopbarExtension.Size = UDim2.new(1, 0, 0, 10)
TopbarExtension.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
TopbarExtension.BorderSizePixel = 0
TopbarExtension.Parent = Topbar

local Title = Instance.new("TextLabel")
Title.Position = UDim2.new(0, 14, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.Text = "thelastcoowner"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 14
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Topbar

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.AnchorPoint = Vector2.new(1, 0.5)
MinimizeButton.Position = UDim2.new(1, -12, 0.5, 0)
MinimizeButton.Size = UDim2.new(0, 24, 0, 24)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MinimizeButton.Text = "—"
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.TextSize = 12
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.AutoButtonColor = false
MinimizeButton.Parent = Topbar

local MinimizeCorner = Instance.new("UICorner")
MinimizeCorner.CornerRadius = UDim.new(0, 6)
MinimizeCorner.Parent = MinimizeButton

-- Anti Drop Button Frame
local AntiDropFrame = Instance.new("Frame")
AntiDropFrame.Position = UDim2.new(0, 14, 0, 50)
AntiDropFrame.Size = UDim2.new(1, -28, 0, 48)
AntiDropFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
AntiDropFrame.BorderSizePixel = 0
AntiDropFrame.Parent = MainFrame

local AntiDropCorner = Instance.new("UICorner")
AntiDropCorner.CornerRadius = UDim.new(0, 8)
AntiDropCorner.Parent = AntiDropFrame

local AntiDropStroke = Instance.new("UIStroke")
AntiDropStroke.Color = Color3.fromRGB(255, 255, 255)
AntiDropStroke.Thickness = 1.2
AntiDropStroke.Parent = AntiDropFrame

local AntiDropLabel = Instance.new("TextLabel")
AntiDropLabel.AnchorPoint = Vector2.new(0, 0.5)
AntiDropLabel.Position = UDim2.new(0, 14, 0.5, 0)
AntiDropLabel.Size = UDim2.new(0.65, 0, 1, 0)
AntiDropLabel.BackgroundTransparency = 1
AntiDropLabel.Text = "Anti Drop"
AntiDropLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
AntiDropLabel.TextSize = 13
AntiDropLabel.Font = Enum.Font.GothamBold
AntiDropLabel.TextXAlignment = Enum.TextXAlignment.Left
AntiDropLabel.Parent = AntiDropFrame

local AntiDropToggle = Instance.new("TextButton")
AntiDropToggle.AnchorPoint = Vector2.new(1, 0.5)
AntiDropToggle.Position = UDim2.new(1, -14, 0.5, 0)
AntiDropToggle.Size = UDim2.new(0, 44, 0, 24)
AntiDropToggle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
AntiDropToggle.Text = ""
AntiDropToggle.AutoButtonColor = false
AntiDropToggle.Parent = AntiDropFrame

local AntiDropToggleCorner = Instance.new("UICorner")
AntiDropToggleCorner.CornerRadius = UDim.new(1, 0)
AntiDropToggleCorner.Parent = AntiDropToggle

local AntiDropKnob = Instance.new("Frame")
AntiDropKnob.AnchorPoint = Vector2.new(0, 0.5)
AntiDropKnob.Position = UDim2.new(0, 24, 0.5, 0)
AntiDropKnob.Size = UDim2.new(0, 16, 0, 16)
AntiDropKnob.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
AntiDropKnob.BorderSizePixel = 0
AntiDropKnob.Parent = AntiDropToggle

local AntiDropKnobCorner = Instance.new("UICorner")
AntiDropKnobCorner.CornerRadius = UDim.new(1, 0)
AntiDropKnobCorner.Parent = AntiDropKnob

-- Toggle Logic
local function setToggleState(enabled)
	local tweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

	if enabled then
		TweenService:Create(AntiDropToggle, tweenInfo, {BackgroundColor3 = Color3.fromRGB(40, 40, 40)}):Play()
		TweenService:Create(AntiDropKnob, tweenInfo, {
			Position = UDim2.new(0, 4, 0.5, 0),
			BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		}):Play()
	else
		TweenService:Create(AntiDropToggle, tweenInfo, {BackgroundColor3 = Color3.fromRGB(255, 255, 255)}):Play()
		TweenService:Create(AntiDropKnob, tweenInfo, {
			Position = UDim2.new(0, 24, 0.5, 0),
			BackgroundColor3 = Color3.fromRGB(18, 18, 18)
		}):Play()
	end
end

AntiDropToggle.MouseButton1Click:Connect(function()
	antiDropEnabled = not antiDropEnabled
	setToggleState(antiDropEnabled)

	if antiDropEnabled then
		pcall(function()
			game:GetService("StarterGui"):SetCore("ResetButtonCallback", false)
		end)
	else
		pcall(function()
			game:GetService("StarterGui"):SetCore("ResetButtonCallback", true)
		end)
	end
end)

-- Minimize
MinimizeButton.MouseButton1Click:Connect(function()
	MainFrame.Visible = not MainFrame.Visible
end)

-- Dragging
local dragging, dragStart, startPos
Topbar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
	end
end)

Topbar.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)
