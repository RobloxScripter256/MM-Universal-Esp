
-- LocalScript (StarterPlayerScripts)
-- ESP + Admin-style GUI Lines
-- Lines from top screen center to all OTHER player Heads
-- Name color = TeamColor (neutral = white)

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- CONFIG
local LINE_THICKNESS = 1
local OFFSET_Y = 40 -- pixels down from top center

--// Create ScreenGui once for toggle
local screenGui = LocalPlayer:FindFirstChild("PlayerGui"):FindFirstChild("ESP_GUI")
if not screenGui then
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESP_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = game.CoreGui
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

---------------------------------------------------------------------------------------------------
-- GUI for lines (CoreGui, admin ESP lines style)
local lineGui = Instance.new("ScreenGui")
lineGui.Name = "AdminHeadLines"
lineGui.IgnoreGuiInset = true
lineGui.DisplayOrder = 1000
lineGui.ResetOnSpawn = false
lineGui.Parent = game.CoreGui

local tracked = {} -- [player] = {line = Frame, head = BasePart?}

local function makeLine()
	local f = Instance.new("Frame")
	f.Size = UDim2.fromOffset(10, LINE_THICKNESS)
	f.AnchorPoint = Vector2.new(0.5, 0.5)
	f.BorderSizePixel = 0
	f.Visible = false
	f.Parent = lineGui
	return f
end

local function getColor(player)
	if player.Team and player.Team.TeamColor then
		return player.Team.TeamColor.Color
	else
		return Color3.new(1,1,1)
	end
end

local function updateHead(player, character)
	if tracked[player] then
		tracked[player].head = character:WaitForChild("Head", 5)
	end
end

local function trackPlayer(player)
	if player == LocalPlayer then return end
	if not tracked[player] then
		local line = makeLine()
		tracked[player] = {line = line, head = nil}
		line.BackgroundColor3 = getColor(player)

		player:GetPropertyChangedSignal("Team"):Connect(function()
			line.BackgroundColor3 = getColor(player)
		end)

		player.CharacterAdded:Connect(function(char)
			updateHead(player, char)
		end)
		player.CharacterRemoving:Connect(function()
			if tracked[player] then
				tracked[player].head = nil
			end
		end)
	end
	if player.Character then
		updateHead(player, player.Character)
	end
end

local function untrackPlayer(player)
	if tracked[player] then
		if tracked[player].line then
			tracked[player].line:Destroy()
		end
		tracked[player] = nil
	end
end

for _,p in ipairs(Players:GetPlayers()) do
	trackPlayer(p)
end
Players.PlayerAdded:Connect(trackPlayer)
Players.PlayerRemoving:Connect(untrackPlayer)
---------------------------------------------------------------------------------------------------

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
	nameTag.TextColor3 = getColor(player)
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
end

--// Loop update info
RunService.RenderStepped:Connect(function()
	-- update GUI lines
	local vpSize = Camera.ViewportSize
	local srcX = vpSize.X/2
	local srcY = OFFSET_Y

	for player,data in pairs(tracked) do
		local line = data.line
		local head = data.head
		if toggle and head and head.Parent then
			local pos, visible = Camera:WorldToViewportPoint(head.Position)
			if visible and pos.Z > 0 then
				local dx,dy = pos.X-srcX, pos.Y-srcY
				local dist = math.sqrt(dx*dx+dy*dy)
				local midX, midY = (pos.X+srcX)/2, (pos.Y+srcY)/2

				line.Visible = true
				line.Size = UDim2.fromOffset(dist, LINE_THICKNESS)
				line.Position = UDim2.fromOffset(midX, midY)
				line.Rotation = math.deg(math.atan2(dy,dx))
			else
				line.Visible = false
			end
		else
			line.Visible = false
		end
	end

	-- update ESP billboards
	for _,player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			createESP(player)
			
			local esp = player.Character.HumanoidRootPart:FindFirstChild("MainEsp")
			local side = player.Character.HumanoidRootPart:FindFirstChild("Side")
			
			if esp then esp.Enabled = toggle end
			if side then side.Enabled = toggle end
			
			if toggle and esp then
				local mainFrame = esp:FindFirstChild("MainFrame")
				if mainFrame then
					local nameTag = mainFrame:FindFirstChild("Name")
					local fTag = mainFrame:FindFirstChild("FlagName")
					local ITag = mainFrame:FindFirstChild("Info")
					
					if nameTag then
						nameTag.Text = string.format("%s (@%s)", player.DisplayName, player.Name)
						nameTag.TextColor3 = getColor(player)
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
								mt.Text = "[No LeaderStats Found]"
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
		toggleButton.Text = "Esp Lines:ON"
		toggleButton.TextColor3 = Color3.fromRGB(0,255,0)
	else
		toggleButton.Text = "Esp Lines:OFF"
		toggleButton.TextColor3 = Color3.fromRGB(255,0,0)
	end
end)

---- Aim-Bot

-- LocalScript (StarterPlayerScripts)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- Config (default)
local SWITCH_TIME = 2 -- seconds per target
local MAX_DISTANCE = 100 -- default distance (studs)
local AIM_ENABLED = false
local lastTarget
local RaycastParams = RaycastParams.new()
RaycastParams.FilterType = Enum.RaycastFilterType.Blacklist

-- Utility: safely parse number
local function toPositiveNumber(s, default)
    local n = tonumber(s)
    if not n or n <= 0 then return default end
    return n
end

-- === GUI ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimBotUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui
screenGui.ScreenInsets = Enum.ScreenInsets.None
screenGui.Parent = game.CoreGui


-- Frame (draggable)
local frame = Instance.new("Frame")
frame.Name = "AimbotFrame"
frame.Size = UDim2.new(0.15, 0, 0.34, 0) -- container size
frame.Position = UDim2.new(0.5, 0, 0.05, 0) -- user requested
frame.AnchorPoint = Vector2.new(0.5, 0)     -- anchor (0.5, 0)
frame.BackgroundTransparency = 0.35
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.Parent = screenGui

-- Enable dragging via UIDragDetector
local dragDetector = Instance.new("UIDragDetector")
dragDetector.Parent = frame

-- Distance TextBox
local distanceBox = Instance.new("TextBox")
distanceBox.Name = "DistanceBox"
distanceBox.Size = UDim2.new(0.68,0, 0.68, 0) -- user requested
distanceBox.Position = UDim2.new(0.5, 0, 0.05, 0)
distanceBox.AnchorPoint = Vector2.new(0.5, 0)
distanceBox.Text = tostring(MAX_DISTANCE)
distanceBox.PlaceholderText = "Aim-Bot Range"
distanceBox.ClearTextOnFocus = false
distanceBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
distanceBox.TextColor3 = Color3.fromRGB(255, 255, 255)
distanceBox.Font = Enum.Font.SourceSans
distanceBox.TextScaled = true
distanceBox.Parent = frame

local distance_aspect = Instance.new("UIAspectRatioConstraint")
distance_aspect.Parent = distanceBox
distance_aspect.AspectRatio = 1.7

-- Toggle Button (under textbox)
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleAimbot"
toggleButton.Size = UDim2.new(0.68, 0, 0.68, 0) -- user requested
toggleButton.Position = UDim2.new(0.5, 0, 0.9, 0) -- user requested
toggleButton.AnchorPoint = Vector2.new(0.5, 0.9)  -- user requested
toggleButton.Text = "AimBot: OFF"
toggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextScaled = true
toggleButton.Parent = frame

local toggle_aspect = Instance.new("UIAspectRatioConstraint")
toggle_aspect.Parent = toggleButton
toggle_aspect.AspectRatio = 1.7

-- Circle ImageLabel (visible only when AIM_ENABLED)
local circle = Instance.new("ImageLabel")
circle.Name = "Circle"
circle.Size = UDim2.new(0.5, 0, 0.5, 0) -- requested
circle.Position = UDim2.new(0.5, 0, 0.5, 0)
circle.AnchorPoint = Vector2.new(0.5, 0.5)
circle.BackgroundTransparency = 1
circle.Image = "rbxthumb://type=Asset&id=8789695013&w=420&h=420" -- leave blank or put your circle asset id here (e.g. "rbxassetid://12345")
-- initial color while OFF: red when aimbot on but not locked; we'll set when toggled
circle.Visible = false -- only visible when aim toggled on
circle.Parent = screenGui

local circle_aspect = Instance.new("UIAspectRatioConstraint")
circle_aspect.Parent = circle
circle_aspect.AspectRatio = 1

local function isVisible(targetHead)
    if not (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head")) then
        return false
    end

    -- Ignore local character
    RaycastParams.FilterDescendantsInstances = {LocalPlayer.Character}

    local origin = Camera.CFrame.Position
    local direction = (targetHead.Position - origin)
    local result = workspace:Raycast(origin, direction, RaycastParams)

    if result then
        -- Hit something, only valid if we hit the same head
        return result.Instance:IsDescendantOf(targetHead.Parent)
    end
    -- No hit = clear view
    return true
end

-- Helper to update UI colors/visibility
local function setAimbotUIEnabled(enabled)
    AIM_ENABLED = enabled
    if enabled then
        toggleButton.Text = "AimBot: ON"
        toggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        circle.Visible = true
        -- when enabled and not locked, circle should be red per your request
        circle.ImageColor3 = Color3.fromRGB(255, 0, 0)
    else
        toggleButton.Text = "AimBot: OFF"
        toggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        circle.Visible = false
        -- reset color (keeps tidy)
        circle.ImageColor3 = Color3.fromRGB(255, 255, 255)
    end
end

-- Toggle button click
toggleButton.MouseButton1Click:Connect(function()
    -- update MAX_DISTANCE from textbox first
    MAX_DISTANCE = toPositiveNumber(distanceBox.Text, MAX_DISTANCE)
    setAimbotUIEnabled(not AIM_ENABLED)
end)

-- When textbox loses focus, update distance
distanceBox.FocusLost:Connect(function(enterPressed)
    MAX_DISTANCE = toPositiveNumber(distanceBox.Text, MAX_DISTANCE)
    distanceBox.Text = tostring(MAX_DISTANCE)
end)

-- === Targeting logic ===

-- Check whether any other player has a Team assigned
local function teamsExist()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Team ~= nil then
            return true
        end
    end
    return false
end

-- Gather valid targets according to user's constraints
local function getTargets()
    local targets = {}
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return targets
    end
    local myPos = LocalPlayer.Character.HumanoidRootPart.Position
    local teamMode = teamsExist()

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
            local head = plr.Character:FindFirstChild("Head")
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and humanoid.Health > 0 and head and hrp then
                local distance = (hrp.Position - myPos).Magnitude
                if distance <= MAX_DISTANCE and isVisible(head) then
                    -- team filtering
                    if teamMode then
                        if plr.Team ~= LocalPlayer.Team then
                            table.insert(targets, {player = plr, distance = distance})
                        end
                    else
                        table.insert(targets, {player = plr, distance = distance})
                    end
                end
            end
        end
    end

    return targets
end

-- Pick a random target not equal to lastTarget (if possible)
local function pickRandomTarget()
    local others = getTargets()
    if #others == 0 then return nil end

    local pool = {}
    for _, t in ipairs(others) do
        table.insert(pool, t.player)
    end

    if #pool == 0 then return nil end

    local choice
    repeat
        choice = pool[math.random(1, #pool)]
    until choice ~= lastTarget or #pool == 1

    lastTarget = choice
    return choice
end

-- Set circle color: GREEN when locked, RED when unlocked (but only visible while AIM_ENABLED)
local function setCircleLocked(locked)
    if not AIM_ENABLED then
        circle.Visible = false
        return
    end
    circle.Visible = true
    if locked then
        circle.ImageColor3 = Color3.fromRGB(0, 255, 0) -- green when locked
    else
        circle.ImageColor3 = Color3.fromRGB(255, 0, 0) -- red when not locked
    end
end

-- Main loop: perform local camera aim lock
task.spawn(function()
    while true do
        if AIM_ENABLED then
            -- Always keep MAX_DISTANCE updated from the box (in case user typed but didn't unfocus)
            MAX_DISTANCE = toPositiveNumber(distanceBox.Text, MAX_DISTANCE)

            local target = pickRandomTarget()
            if target and target.Character and target.Character:FindFirstChild("Head") then
                local head = target.Character.Head
                local startTime = tick()
                local locked = false

                while tick() - startTime < SWITCH_TIME and AIM_ENABLED and target.Character and target.Character:FindFirstChild("Head") do
                    -- recheck alive + hrp + distance + team (in case of changes)
                    local humanoid = target.Character:FindFirstChildOfClass("Humanoid")
                    local hrp = target.Character:FindFirstChild("HumanoidRootPart")
                    if not (humanoid and humanoid.Health > 0 and hrp and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) then
                        break
                    end

                    local myPos = LocalPlayer.Character.HumanoidRootPart.Position
                    local distance = (hrp.Position - myPos).Magnitude
                    if distance <= MAX_DISTANCE and isVisible(head) then
                        -- team re-check
                        if teamsExist() and target.Team == LocalPlayer.Team then
                            break
                        end

                        -- lock camera locally
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
                        if not locked then
                            locked = true
                            setCircleLocked(true) -- green when locked
                        end
                    else
                        break
                    end

                    RunService.RenderStepped:Wait()
                end

                -- unlocked (either time expired or lost)
                setCircleLocked(false) -- red when unlocked
            else
                -- no valid target: show unlocked red circle while enabled
                setCircleLocked(false)
                RunService.RenderStepped:Wait()
            end
        else
            RunService.RenderStepped:Wait()
        end
    end
end)

-- Initialize UI state
distanceBox.Text = tostring(MAX_DISTANCE)
setAimbotUIEnabled(false)

-- Optional: clicking the circle will turn the aimbot off
circle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        setAimbotUIEnabled(false)
    end
end) 

print("Thanks for using my script, made by Dev_Smiley")
