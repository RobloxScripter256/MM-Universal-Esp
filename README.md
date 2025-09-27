local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

--// Create ScreenGui once
local screenGui = LocalPlayer:FindFirstChild("PlayerGui"):FindFirstChild("ESP_GUI")
if not screenGui then
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESP_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 120, 0, 40)
frame.Position = UDim2.new(0.05, 0, 0.2, 0)
frame.BackgroundTransparency = 0.5
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
frame.Active = true
frame.Draggable = false
frame.Parent = screenGui

-- Draggable detector
local drag = Instance.new("UIDragDetector", frame)

-- Aspect ratio
local aspect = Instance.new("UIAspectRatioConstraint")
aspect.Parent = frame

-- Toggle button
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0.8, 0, 0.8, 0)
toggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
toggleButton.Text = "OFF"
toggleButton.BackgroundTransparency = 0.5
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
toggleButton.TextColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.Parent = frame

local toggle = false

--// Function to create the ESP for a character
local function createESP(player)
	if player == LocalPlayer then return end
	if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
	if player.Character.HumanoidRootPart:FindFirstChild("MainEsp") then return end
	
	-- Main Billboard
	local m = Instance.new("BillboardGui")
	m.AlwaysOnTop = true
	m.Brightness = 1
	m.Size = UDim2.new(0, 100, 0, 28)
	m.ExtentsOffsetWorldSpace = Vector3.new(0, 3.8, 0)
	m.MaxDistance = math.huge
	m.ResetOnSpawn = false
	m.Name = "MainEsp"
	m.Parent = player.Character.HumanoidRootPart
	
	-- Main frame
	local mainbg = Instance.new("Frame")
	mainbg.Name = "MainFrame"
	mainbg.Parent = m
	mainbg.Size = UDim2.new(1, 0, 1, 0)
	mainbg.BackgroundTransparency = 1
	
	-- NameTag
	local nameTag = Instance.new("TextLabel")
	nameTag.Name = "Name"
	nameTag.Parent = mainbg
	nameTag.FontFace = Font.fromEnum(Enum.Font.ArialBold)
	nameTag.Size = UDim2.new(2,0,0.33,0)
	nameTag.AnchorPoint = Vector2.new(0.5,0.5)
	nameTag.TextScaled = true
	nameTag.Position = UDim2.new(0.5, 0, 0.5, 0)
	nameTag.BackgroundTransparency = 1
	nameTag.TextColor3 = Color3.fromRGB(255,255,255)
	local s1 = Instance.new("UIStroke", nameTag)
	s1.Thickness = 0.8
	s1.Color = Color3.new(0,0,0)
	
	-- UID / Flag
	local fTag = Instance.new("TextLabel")
	fTag.Name = "FlagName"
	fTag.Parent = mainbg
	fTag.Size = UDim2.new(1,0,0.34,0)
	fTag.FontFace = Font.fromEnum(Enum.Font.ArialBold)
	fTag.AnchorPoint = Vector2.new(0.5,0)
	fTag.Position = UDim2.new(0.5,0,0,0)
	fTag.BackgroundTransparency = 1
	fTag.TextColor3 = Color3.fromRGB(255, 191, 191)
	fTag.TextScaled = true
	local s2 = Instance.new("UIStroke", fTag)
	s2.Thickness = 0.8
	s2.Color = Color3.new(0,0,0)
	
	-- Info / Distance
	local ITag = Instance.new("TextLabel")
	ITag.Name = "Info"
	ITag.Parent = mainbg
	ITag.FontFace = Font.fromEnum(Enum.Font.ArialBold)
	ITag.Size = UDim2.new(1.2,0,0.34,0)
	ITag.AnchorPoint = Vector2.new(0.5,1)
	ITag.Position = UDim2.new(0.5,0,1,0)
	ITag.BackgroundTransparency = 1
	ITag.TextScaled = true
	ITag.TextColor3 = Color3.fromRGB(110, 164, 255)
	local s3 = Instance.new("UIStroke", ITag)
	s3.Thickness = 0.8
	s3.Color = Color3.new(0,0,0)
	
	-- Side
	local sb = Instance.new("BillboardGui")
	sb.Size = UDim2.new(4.2, 0, 1.3, 0)
	sb.Parent = player.Character.HumanoidRootPart
	sb.Brightness = 2
	sb.AlwaysOnTop = true
	sb.ResetOnSpawn = false
	sb.Name = "Side"
	sb.ExtentsOffsetWorldSpace = Vector3.new(-3.5,0,0)
	
	local sbg = Instance.new("Frame")
	sbg.Name = "MainFrame"
	sbg.Parent = sb
	sbg.Size = UDim2.new(1, 0, 1, 0)
	sbg.BackgroundTransparency = 1
	
	local hT = Instance.new("TextLabel")
	hT.Name = "Health"
	hT.Parent = sbg
	hT.Size = UDim2.new(1,0,0.33,0)
	hT.AnchorPoint = Vector2.new(0,0)
	hT.FontFace = Font.fromEnum(Enum.Font.ArialBold)
	hT.TextScaled = true
	hT.TextXAlignment = Enum.TextXAlignment.Left
	hT.Position = UDim2.new(0,0,0,0)
	hT.BackgroundTransparency = 1
	hT.TextColor3 = Color3.fromRGB(255, 0, 0)
	local s4 = Instance.new("UIStroke", hT)
	s4.Thickness = 0.8
	s4.Color = Color3.new(0,0,0)
	
	local mt = Instance.new("TextLabel")
	mt.Name = "Info"
	mt.Parent = sbg
	mt.Size = UDim2.new(1,0,0.35,0)
	mt.AnchorPoint = Vector2.new(0,0.5)
	mt.TextScaled = true
	mt.Position = UDim2.new(0,0,0.5,0)
	mt.TextXAlignment = Enum.TextXAlignment.Left
	mt.FontFace = Font.fromEnum(Enum.Font.ArialBold)
	mt.BackgroundTransparency = 1
	mt.TextColor3 = Color3.fromRGB(191, 255, 127)
	local s5 = Instance.new("UIStroke", mt)
	s5.Thickness = 0.8
	s5.Color = Color3.new(0,0,0)

-- Box
	local bb = Instance.new("BillboardGui")
	bb.Size = UDim2.new(0, 42, 0, 42)
	bb.Parent = player.Character.HumanoidRootPart
	bb.Brightness = math.huge
	bb.AlwaysOnTop = true
	bb.ResetOnSpawn = false
	bb.Name = "Box"

	local bicon = Instance.new("ImageLabel")
	bicon.Parent = bb
	bicon.Name = "Icon"
	bicon.Image = "rbxthumb://type=Asset&id=81228165635365&w=420&h=420"
	bicon.Size = UDim2.new(1, 0, 1, 0)
	bicon.BackgroundTransparency = 1
	
	-- Beam (connect RootParts)
	if not player.Character:FindFirstChild("HeadLine") then
		local att1 = Instance.new("Attachment", player.Character.HumanoidRootPart)
		local att2 = Instance.new("Attachment", LocalPlayer.Character and LocalPlayer.Character:WaitForChild("HumanoidRootPart"))
		local beam = Instance.new("Beam")
		beam.Name = "HeadLine"
		beam.Attachment0 = att1
		beam.Attachment1 = att2
		beam.Width0 = 0.1
		beam.Width1 = 0.1
		beam.Color = ColorSequence.new(Color3.fromRGB(0, 255, 0))
		beam.Parent = player.Character
	end
end

--// Loop update info
RunService.RenderStepped:Connect(function()
	for _,player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			createESP(player)
			
			local esp = player.Character.HumanoidRootPart:FindFirstChild("MainEsp")
			local side = player.Character.HumanoidRootPart:FindFirstChild("Side")
			local line = player.Character:FindFirstChild("HeadLine")
			
			-- Toggle handling
			if esp then esp.Enabled = toggle end
			if side then side.Enabled = toggle end
			if line then line.Enabled = toggle end
			
			if toggle and esp then
				local mainFrame = esp:FindFirstChild("MainFrame")
				if mainFrame then
					local nameTag = mainFrame:FindFirstChild("Name")
					local fTag = mainFrame:FindFirstChild("FlagName")
					local ITag = mainFrame:FindFirstChild("Info")
					
					if nameTag then
						nameTag.Text = string.format("%s (@%s)", player.DisplayName, player.Name)
					end
					
					if fTag then
						fTag.Text = "[UID: " .. tostring(player.UserId) .. "]"
					end
					
					if ITag then
						local distance = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart"))
							and math.floor((player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude) or 0
						local toolName = "None"
if player.Character then
    for _,tool in ipairs(player.Character:GetChildren()) do
        if tool:IsA("Tool") then
            toolName = tool.Name
            break
        end
    end
end
ITag.Text = "[Distance: "..distance.." s/s , "..toolName.."]"
					end
				end
				
				if side then
					local sbg = side:FindFirstChild("MainFrame")
					if sbg then
						local hT = sbg:FindFirstChild("Health")
						local mt = sbg:FindFirstChild("Info")
						
						local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
						if hT and humanoid then
							local hp = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
							hT.Text = "[Health: "..hp.."%]"
						end
						
						if mt then
							local ls = player:FindFirstChild("leaderstats")
							if ls then
								local str = {}
								for _,v in pairs(ls:GetChildren()) do
									if v:IsA("IntValue") or v:IsA("NumberValue") then
										table.insert(str, v.Name..": "..v.Value)
									end
								end
								mt.Text = "["..table.concat(str, " , ").."]"
							else
								mt.Text = "[Nil]"
							end
						end
					end
				end
			end
		end
	end
end)

--// Handle respawns
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		task.wait(1)
		createESP(player)
	end)
end)

-- Toggle button logic
toggleButton.MouseButton1Click:Connect(function()
	toggle = not toggle
	if toggle then
		toggleButton.Text = "ON"
		toggleButton.TextColor3 = Color3.fromRGB(0,255,0)
	else
		toggleButton.Text = "OFF"
		toggleButton.TextColor3 = Color3.fromRGB(255,0,0)
	end
end)
