
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
---- Aim-Bot (Merged with new UI)
-- LocalScript (StarterPlayerScripts)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- Config (default)
local SWITCH_TIME = 2 -- seconds per target
local MAX_DISTANCE = 100 -- default distance (studs)
local AIM_ENABLED = false
local lastTarget

-- Raycast params (visibility)
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Blacklist

-- Utility: safely parse number
local function toPositiveNumber(s, default)
    local n = tonumber(s)
    if not n or n <= 0 then return default end
    return n
end

-- === Prepare TestGui container ===
local testGui = PlayerGui:FindFirstChild("TestGui")
if not testGui then
    testGui = Instance.new("ScreenGui")
    testGui.Name = "AimBotGui"
    testGui.ResetOnSpawn = false
    testGui.Parent = game.CoreGui
testGui.ScreenInsets = Enum.ScreenInsets.None
end

-- === USER UI CONFIG (from your snippet) ===
local tfs = Enum.Font.ArialBold -- text font
local oicon = "rbxthumb://type=Asset&id=11552476835&w=420&h=420"
local cicon = "rbxthumb://type=Asset&id=11552476835&w=420&h=420"
local cron = 180
local oron = 0
local cpos = UDim2.new(0.5,0,1.087,0)
local opos = UDim2.new(0.75,0,0.5,0)

-- === BUILD NEW UI (MainFrame + TopBar + MiddleFrame + controls) ===

-- Main Frame (mf)
local mf = Instance.new("Frame")
mf.Name = "AimBotSettingsFrame"
mf.Parent = testGui
mf.BackgroundTransparency = 0.8
mf.BackgroundColor3 = Color3.new(0,0,0)
mf.Size = UDim2.new(0.32,0,0.32,0)
mf.Position = UDim2.new(0.5,0,0.5,0)
mf.AnchorPoint = Vector2.new(0.5,0.5)

local mfas = Instance.new("UIAspectRatioConstraint")
mfas.Parent = mf
mfas.AspectRatio = 1.6

local mfdg = Instance.new("UIDragDetector")
mfdg.Parent = mf

local mfStroke = Instance.new("UIStroke")
mfStroke.Parent = mf
mfStroke.Color = Color3.fromRGB(255,255,255)
mfStroke.Thickness = 0.8

-- TopBar
local tbf = Instance.new("Frame")
tbf.Parent = mf
tbf.BackgroundTransparency = 0.5
tbf.BackgroundColor3 = Color3.new(0,0,0)
tbf.Size = UDim2.new(1,0,0.18,0)
tbf.Position = UDim2.new(0.5,0,0,0)
tbf.AnchorPoint = Vector2.new(0.5,0)
tbf.BorderSizePixel = 0
tbf.Name = "TopBar"

local cBtn = Instance.new("ImageButton")
cBtn.Parent = tbf
cBtn.BackgroundTransparency = 0.8
cBtn.Image = oicon
cBtn.Rotation = oron
cBtn.BackgroundColor3 = Color3.new(0,0,0)
cBtn.Size = UDim2.new(0.8,0,0.8,0)
cBtn.Position = UDim2.new(0.9,0,0.5,0)
cBtn.AnchorPoint = Vector2.new(0.9,0.5)
cBtn.BorderSizePixel = 1
cBtn.Name = "ToggleHide"

local cBtnAspect = Instance.new("UIAspectRatioConstraint")
cBtnAspect.Parent = cBtn

local ttt = Instance.new("TextLabel")
ttt.Parent = tbf
ttt.BackgroundTransparency = 0.8
ttt.Text = "Universal Aimbot 🎯"
ttt.TextScaled = true
ttt.BackgroundColor3 = Color3.new(0,0,0)
ttt.Size = UDim2.new(0.7,0,0.7,0)
ttt.Position = UDim2.new(0.1,0,0.5,0)
ttt.Font = tfs
ttt.AnchorPoint = Vector2.new(0.1,0.5)
ttt.BorderSizePixel = 1
ttt.Name = "Title"
ttt.TextColor3 = Color3.fromRGB(255,255,255)

-- Middle Frame
local mmf = Instance.new("Frame")
mmf.Parent = mf
mmf.BackgroundTransparency = 0.9
mmf.BackgroundColor3 = Color3.new(0,0,0)
mmf.Size = UDim2.new(1,0,0.82,0)
mmf.Position = UDim2.new(0.5,0,0.92,0)
mmf.AnchorPoint = Vector2.new(0.5,0.9)
mmf.BorderSizePixel = 0
mmf.Name = "MiddleFrame"

-- Child labels
local t1 = Instance.new("TextLabel")
t1.Parent = mmf
t1.BackgroundTransparency = 0.8
t1.Text = "Target Player Name Or All"
t1.TextScaled = true
t1.BackgroundColor3 = Color3.new(0,0,0)
t1.Size = UDim2.new(0.35,0,0.2,0)
t1.Position = UDim2.new(0.1,0,0.11,0)
t1.Font = tfs
t1.AnchorPoint = Vector2.new(0.1,0.11)
t1.BorderSizePixel = 1
t1.Name = "TitleA"
t1.TextColor3 = Color3.fromRGB(255,255,255)

local t2 = Instance.new("TextLabel")
t2.Parent = mmf
t2.BackgroundTransparency = 0.8
t2.Text = "Aimbot Range"
t2.TextScaled = true
t2.BackgroundColor3 = Color3.new(0,0,0)
t2.Size = UDim2.new(0.35,0,0.2,0)
t2.Position = UDim2.new(0.1,0,0.5,0)
t2.Font = tfs
t2.AnchorPoint = Vector2.new(0.1,0.5)
t2.BorderSizePixel = 1
t2.Name = "TitleB"
t2.TextColor3 = Color3.fromRGB(255,255,255)

-- Aimbot toggle TextButton (apbtn)
local apbtn = Instance.new("TextButton")
apbtn.Parent = mmf
apbtn.BackgroundTransparency = 0.8
apbtn.Text = "Aimbot: Off"
apbtn.TextScaled = true
apbtn.BackgroundColor3 = Color3.new(0,0,0)
apbtn.Size = UDim2.new(0.7,0,0.219,0)
apbtn.Position = UDim2.new(0.5,0,0.9,0)
apbtn.Font = tfs
apbtn.AnchorPoint = Vector2.new(0.5,0.9)
apbtn.BorderSizePixel = 1
apbtn.Name = "ToggleAimBotBtn"
apbtn.TextColor3 = Color3.fromRGB(255,255,255)

-- Target TextBox (trbx)
local trbx = Instance.new("TextBox")
trbx.Parent = mmf
trbx.AnchorPoint = Vector2.new(0.1,0.108)
trbx.Size = UDim2.new(0.5,0,0.2,0)
trbx.Position = UDim2.new(0.512,0,0.108,0)
trbx.PlaceholderText = "Target Player Name Or All"
trbx.Text = ""
trbx.BackgroundTransparency = 0.8
trbx.TextColor3 = Color3.fromRGB(255,255,255)
trbx.BackgroundColor3 = Color3.new(0,0,0)
trbx.TextScaled = true
trbx.Font = tfs
trbx.Name = "TargetBox"

-- Range TextBox (trrbx)
local trrbx = Instance.new("TextBox")
trrbx.Parent = mmf
trrbx.AnchorPoint = Vector2.new(0.1,0.5)
trrbx.Size = UDim2.new(0.5,0,0.2,0)
trrbx.Position = UDim2.new(0.512,0,0.5,0)
trrbx.PlaceholderText = "Aimbot Range"
trrbx.Text = tostring(MAX_DISTANCE)
trrbx.BackgroundTransparency = 0.8
trrbx.TextColor3 = Color3.fromRGB(255,255,255)
trrbx.BackgroundColor3 = Color3.new(0,0,0)
trrbx.TextScaled = true
trrbx.Font = tfs
trrbx.Name = "RangeBox"

-- === CIRCLE (center screen) ===
local circle = Instance.new("ImageLabel")
circle.Name = "Circle"
circle.Size = UDim2.new(0.5, 0, 0.5, 0)
circle.Position = UDim2.new(0.5, 0, 0.5, 0)
circle.AnchorPoint = Vector2.new(0.5, 0.5)
circle.BackgroundTransparency = 1
circle.Image = "rbxthumb://type=Asset&id=8789695013&w=420&h=420"
circle.ImageColor3 = Color3.fromRGB(255, 0, 0)
circle.Visible = false
circle.Parent = testGui

local circle_aspect = Instance.new("UIAspectRatioConstraint")
circle_aspect.Parent = circle
circle_aspect.AspectRatio = 1

-- === UI behaviour: cBtn hide/show the main frame (mf) with tween and rotate cBtn ===
local hidden = false
cBtn.Rotation = oron
cBtn.Image = oicon

local function setFrameHidden(hide)
    hidden = hide
    local targetPos = hide and cpos or opos
    local targetRot = hide and cron or oron
    local tweenInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(mf, tweenInfo, {Position = targetPos}):Play()
    TweenService:Create(cBtn, tweenInfo, {Rotation = targetRot, ImageTransparency = hide and 1 or 0}):Play()
    cBtn.Image = hide and cicon or oicon
end

cBtn.MouseButton1Click:Connect(function()
    setFrameHidden(not hidden)
end)

-- === Toggle button handling (apbtn toggles aimbot) ===
local function setAimbotUIEnabled(enabled)
    AIM_ENABLED = enabled
    if enabled then
        apbtn.Text = "Aimbot: On"
        apbtn.BackgroundColor3 = Color3.fromRGB(0,170,0)
        circle.Visible = true
        circle.ImageColor3 = Color3.fromRGB(255,0,0) -- red until locked
    else
        apbtn.Text = "Aimbot: Off"
        apbtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
        circle.Visible = false
        circle.ImageColor3 = Color3.fromRGB(255,255,255)
        lastTarget = nil
    end
end

apbtn.MouseButton1Click:Connect(function()
    -- update MAX_DISTANCE from RangeBox
    MAX_DISTANCE = toPositiveNumber(trrbx.Text, MAX_DISTANCE)
    setAimbotUIEnabled(not AIM_ENABLED)
end)

trrbx.FocusLost:Connect(function()
    MAX_DISTANCE = toPositiveNumber(trrbx.Text, MAX_DISTANCE)
    trrbx.Text = tostring(MAX_DISTANCE)
end)

trbx.FocusLost:Connect(function()
    -- no extra action required; we will read trbx.Text in logic
end)

-- === RAYCAST / VISIBILITY setup ===
local function isVisible(targetHead)
    if not (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head")) then
        return false
    end

    rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
    local origin = Camera.CFrame.Position
    local dir = (targetHead.Position - origin)
    local res = workspace:Raycast(origin, dir, rayParams)
    if res then
        return res.Instance:IsDescendantOf(targetHead.Parent)
    end
    return true
end

-- === TEAM DETECTION ===
local function teamsExist()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Team ~= nil then
            return true
        end
    end
    return false
end

-- === TARGET COLLECTION ===
local function nameMatchesCandidate(nameQuery, player)
    if not nameQuery or nameQuery == "" then return true end
    if nameQuery:lower() == "all" then return true end
    local lowerQuery = nameQuery:lower()
    local pn = player.Name:lower()
    if pn == lowerQuery then return true end
    if pn:find(lowerQuery, 1, true) then return true end
    return false
end

local function getTargets()
    local targets = {}
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return targets
    end
    local myPos = LocalPlayer.Character.HumanoidRootPart.Position
    local teamMode = teamsExist()
    local query = trbx.Text or ""
    local useNameFilter = (query ~= "" and query:lower() ~= "all")

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
            local head = plr.Character:FindFirstChild("Head")
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and humanoid.Health > 0 and head and hrp then
                local distance = (hrp.Position - myPos).Magnitude
                if distance <= MAX_DISTANCE and isVisible(head) then
                    if teamMode then
                        if plr.Team ~= LocalPlayer.Team then
                            if not useNameFilter or nameMatchesCandidate(query, plr) then
                                table.insert(targets, {player = plr, distance = distance})
                            end
                        end
                    else
                        if not useNameFilter or nameMatchesCandidate(query, plr) then
                            table.insert(targets, {player = plr, distance = distance})
                        end
                    end
                end
            end
        end
    end

    return targets
end

-- === PICKING TARGET ===
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

-- Optionally closest picker
local function pickClosestTarget()
    local others = getTargets()
    if #others == 0 then return nil end
    table.sort(others, function(a,b) return a.distance < b.distance end)
    lastTarget = others[1].player
    return lastTarget
end

-- === CIRCLE STATE ===
local function setCircleLocked(locked)
    if not AIM_ENABLED then
        circle.Visible = false
        return
    end
    circle.Visible = true
    if locked then
        circle.ImageColor3 = Color3.fromRGB(0, 255, 0)
    else
        circle.ImageColor3 = Color3.fromRGB(255, 0, 0)
    end
end

-- === MAIN AIM LOOP ===
task.spawn(function()
    while true do
        if AIM_ENABLED then
            -- Always keep MAX_DISTANCE updated from the box (in case user typed but didn't unfocus)
            MAX_DISTANCE = toPositiveNumber(trrbx.Text, MAX_DISTANCE)

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
trrbx.Text = tostring(MAX_DISTANCE)
setAimbotUIEnabled(false)
setFrameHidden(false)

-- Optional: clicking the circle will turn the aimbot off
circle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        setAimbotUIEnabled(false)
    end
end)

print("Thanks for using my script, made by Dev_Smiley")
