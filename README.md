-- ProbeSystem.lua
-- Twistex Probe: Earns money, tracks wind speed, has 1000 HP
-- Place in ServerScriptService

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

-- ========================
-- REMOTE EVENTS
-- ========================
local probeDataEvent = Instance.new("RemoteEvent")
probeDataEvent.Name = "ProbeData"
probeDataEvent.Parent = ReplicatedStorage

-- ========================
-- LEADERBOARD SETUP
-- Money stat for each player
-- ========================
Players.PlayerAdded:Connect(function(player)
	local leaderstats = Instance.new("Folder")
	leaderstats.Name = "leaderstats"
	leaderstats.Parent = player

	local money = Instance.new("IntValue")
	money.Name = "Money"
	money.Value = 0
	money.Parent = leaderstats
end)

-- ========================
-- PROBE SETTINGS
-- ========================
local ProbeConfig = {
	MaxHealth = 1000,
	MoneyPerSecond = 10,       -- Base money per second when near tornado
	MoneyMultiplierMax = 5,    -- Max multiplier at closest range
	EarnRadius = 150,          -- Distance from tornado to start earning
	DamageRadius = 40,         -- Distance where probe takes damage
	DamagePerSecond = 50,      -- Damage per second at closest range
	WindSpeedMax = 300,        -- Max wind speed in mph at damage radius
	WindSpeedBase = 50,        -- Wind speed at earn radius edge
	UpdateRate = 0.1,          -- How often probe updates (seconds)
}

-- ========================
-- WIND SPEED CALCULATOR
-- Based on real tornado wind speed formulas
-- EF Scale approximation
-- ========================
local function calculateWindSpeed(distFromTornado)
	if distFromTornado > ProbeConfig.EarnRadius then
		return 0
	end

	-- Inverse relationship: closer = faster wind
	local t = 1 - (distFromTornado / ProbeConfig.EarnRadius)
	local windSpeed = math.lerp(
		ProbeConfig.WindSpeedBase,
		ProbeConfig.WindSpeedMax,
		t ^ 1.5  -- exponential increase closer to core
	)

	return math.floor(windSpeed)
end

-- ========================
-- EF SCALE RATING
-- ========================
local function getEFRating(windSpeed)
	if windSpeed < 65 then
		return "No Rating", Color3.fromRGB(150, 150, 150)
	elseif windSpeed < 86 then
		return "EF0", Color3.fromRGB(100, 200, 100)
	elseif windSpeed < 110 then
		return "EF1", Color3.fromRGB(200, 200, 50)
	elseif windSpeed < 135 then
		return "EF2", Color3.fromRGB(230, 150, 50)
	elseif windSpeed < 165 then
		return "EF3", Color3.fromRGB(220, 80, 50)
	elseif windSpeed < 200 then
		return "EF4", Color3.fromRGB(180, 30, 30)
	else
		return "EF5", Color3.fromRGB(120, 0, 180)
	end
end

-- ========================
-- PROBE BUILDER
-- Creates the probe model programmatically
-- ========================
local function createProbe(position, owner)
	local probeModel = Instance.new("Model")
	probeModel.Name = "TwistexProbe_" .. owner.Name
	probeModel.Parent = workspace

	-- Main body (low wedge shape)
	local body = Instance.new("Part")
	body.Name = "PrimaryPart"
	body.Size = Vector3.new(6, 1.5, 10)
	body.Color = Color3.fromRGB(255, 140, 0)  -- orange like real Twistex probe
	body.Material = Enum.Material.SmoothPlastic
	body.CFrame = CFrame.new(position)
	body.Anchored = true
	body.Parent = probeModel
	probeModel.PrimaryPart = body

	-- Sensor dome on top
	local dome = Instance.new("Part")
	dome.Size = Vector3.new(2, 2, 2)
	dome.Shape = Enum.PartType.Ball
	dome.Color = Color3.fromRGB(200, 200, 200)
	dome.Material = Enum.Material.SmoothPlastic
	dome.CFrame = body.CFrame * CFrame.new(0, 1.5, 0)
	dome.Anchored = true
	dome.Parent = probeModel

	-- Antenna
	local antenna = Instance.new("Part")
	antenna.Size = Vector3.new(0.2, 3, 0.2)
	antenna.Color = Color3.fromRGB(50, 50, 50)
	antenna.Material = Enum.Material.SmoothPlastic
	antenna.CFrame = body.CFrame * CFrame.new(0, 3, 0)
	antenna.Anchored = true
	antenna.Parent = probeModel

	-- Front wedge bumper
	local bumper = Instance.new("Part")
	bumper.Size = Vector3.new(6, 1, 2)
	bumper.Color = Color3.fromRGB(80, 80, 80)
	bumper.Material = Enum.Material.SmoothPlastic
	bumper.CFrame = body.CFrame * CFrame.new(0, -0.3, -5.5)
	bumper.Anchored = true
	bumper.Parent = probeModel

	-- Left wheel
	local wheelL = Instance.new("Part")
	wheelL.Size = Vector3.new(0.5, 2, 2)
	wheelL.Shape = Enum.PartType.Cylinder
	wheelL.Color = Color3.fromRGB(30, 30, 30)
	wheelL.Material = Enum.Material.SmoothPlastic
	wheelL.CFrame = body.CFrame * CFrame.new(-3.2, -1, 0)
	wheelL.Anchored = true
	wheelL.Parent = probeModel

	-- Right wheel
	local wheelR = Instance.new("Part")
	wheelR.Size = Vector3.new(0.5, 2, 2)
	wheelR.Shape = Enum.PartType.Cylinder
	wheelR.Color = Color3.fromRGB(30, 30, 30)
	wheelR.Material = Enum.Material.SmoothPlastic
	wheelR.CFrame = body.CFrame * CFrame.new(3.2, -1, 0)
	wheelR.Anchored = true
	wheelR.Parent = probeModel

	-- ========================
	-- HEALTH BAR GUI (above probe)
	-- ========================
	local billboardGui = Instance.new("BillboardGui")
	billboardGui.Size = UDim2.new(0, 300, 0, 160)
	billboardGui.StudsOffset = Vector3.new(0, 6, 0)
	billboardGui.AlwaysOnTop = false
	billboardGui.Adornee = body
	billboardGui.Parent = body

	-- Background frame
	local bgFrame = Instance.new("Frame")
	bgFrame.Size = UDim2.new(1, 0, 1, 0)
	bgFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	bgFrame.BackgroundTransparency = 0.3
	bgFrame.BorderSizePixel = 0
	bgFrame.Parent = billboardGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = bgFrame

	-- Probe title
	local titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, 0, 0.2, 0)
	titleLabel.Position = UDim2.new(0, 0, 0, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = "🌪️ TWISTEX PROBE"
	titleLabel.TextColor3 = Color3.fromRGB(255, 140, 0)
	titleLabel.TextScaled = true
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.Parent = bgFrame

	-- Owner label
	local ownerLabel = Instance.new("TextLabel")
	ownerLabel.Size = UDim2.new(1, 0, 0.15, 0)
	ownerLabel.Position = UDim2.new(0, 0, 0.18, 0)
	ownerLabel.BackgroundTransparency = 1
	ownerLabel.Text = "Owner: " .. owner.Name
	ownerLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
	ownerLabel.TextScaled = true
	ownerLabel.Font = Enum.Font.Gotham
	ownerLabel.Parent = bgFrame

	-- Health bar background
	local healthBg = Instance.new("Frame")
	healthBg.Size = UDim2.new(0.9, 0, 0.12, 0)
	healthBg.Position = UDim2.new(0.05, 0, 0.35, 0)
	healthBg.BackgroundColor3 = Color3.fromRGB(60, 0, 0)
	healthBg.BorderSizePixel = 0
	healthBg.Parent = bgFrame

	local healthCorner = Instance.new("UICorner")
	healthCorner.CornerRadius = UDim.new(0, 4)
	healthCorner.Parent = healthBg

	-- Health bar fill
	local healthFill = Instance.new("Frame")
	healthFill.Name = "HealthFill"
	healthFill.Size = UDim2.new(1, 0, 1, 0)
	healthFill.BackgroundColor3 = Color3.fromRGB(0, 200, 80)
	healthFill.BorderSizePixel = 0
	healthFill.Parent = healthBg

	local healthFillCorner = Instance.new("UICorner")
	healthFillCorner.CornerRadius = UDim.new(0, 4)
	healthFillCorner.Parent = healthFill

	-- Health text
	local healthLabel = Instance.new("TextLabel")
	healthLabel.Name = "HealthLabel"
	healthLabel.Size = UDim2.new(1, 0, 0.15, 0)
	healthLabel.Position = UDim2.new(0, 0, 0.48, 0)
	healthLabel.BackgroundTransparency = 1
	healthLabel.Text = "HP: 1000 / 1000"
	healthLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	healthLabel.TextScaled = true
	healthLabel.Font = Enum.Font.GothamBold
	healthLabel.Parent = bgFrame

	-- Wind speed label
	local windLabel = Instance.new("TextLabel")
	windLabel.Name = "WindLabel"
	windLabel.Size = UDim2.new(1, 0, 0.15, 0)
	windLabel.Position = UDim2.new(0, 0, 0.63, 0)
	windLabel.BackgroundTransparency = 1
	windLabel.Text = "💨 Wind: 0 mph"
	windLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
	windLabel.TextScaled = true
	windLabel.Font = Enum.Font.GothamBold
	windLabel.Parent = bgFrame

	-- EF rating label
	local efLabel = Instance.new("TextLabel")
	efLabel.Name = "EFLabel"
	efLabel.Size = UDim2.new(1, 0, 0.15, 0)
	efLabel.Position = UDim2.new(0, 0, 0.78, 0)
	efLabel.BackgroundTransparency = 1
	efLabel.Text = "Rating: No Rating"
	efLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
	efLabel.TextScaled = true
	efLabel.Font = Enum.Font.GothamBold
	efLabel.Parent = bgFrame

	-- Money earned label
	local moneyLabel = Instance.new("TextLabel")
	moneyLabel.Name = "MoneyLabel"
	moneyLabel.Size = UDim2.new(1, 0, 0.15, 0)
	moneyLabel.Position = UDim2.new(0, 0, 0.93, 0)  
	moneyLabel.BackgroundTransparency = 1
	moneyLabel.Text = "💰 Earned: $0"
	moneyLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
	moneyLabel.TextScaled = true
	moneyLabel.Font = Enum.Font.GothamBold
	moneyLabel.Parent = bgFrame

	return probeModel, body, billboardGui, {
		healthFill = healthFill,
		healthLabel = healthLabel,
		windLabel = windLabel,
		efLabel = efLabel,
		moneyLabel = moneyLabel,
	}
end

-- ========================
-- PROBE SYSTEM RUNNER
-- ========================
local function runProbe(probeModel, body, guiRefs, owner, getTornadoPos)
	local health = ProbeConfig.MaxHealth
	local totalEarned = 0
	local alive = true
	local timer = 0

	local connection
	connection = RunService.Heartbeat:Connect(function(dt)
		if not alive then return end
		timer += dt

		if timer < ProbeConfig.UpdateRate then return end
		timer = 0

		-- Get tornado position
		local tornadoPos = getTornadoPos()
		if not tornadoPos then return end

		local probePos = body.Position
		local dist = (Vector3.new(tornadoPos.X, probePos.Y, tornadoPos.Z) - probePos).Magnitude

		-- ========================
		-- WIND SPEED CALCULATION
		-- ========================
		local windSpeed = calculateWindSpeed(dist)
		local efRating, efColor = getEFRating(windSpeed)

		-- ========================
		-- EARN MONEY
		-- ========================
		if dist <= ProbeConfig.EarnRadius and windSpeed > 0 then
			local moneyMultiplier = math.lerp(1, ProbeConfig.MoneyMultiplierMax,
				1 - (dist / ProbeConfig.EarnRadius)
			)
			local earned = ProbeConfig.MoneyPerSecond * moneyMultiplier * ProbeConfig.UpdateRate
			totalEarned += earned

			-- Give money to owner
			local leaderstats = owner:FindFirstChild("leaderstats")
			if leaderstats then
				local money = leaderstats:FindFirstChild("Money")
				if money then
					money.Value += math.floor(earned)
				end
			end
		end

		-- ========================
		-- TAKE DAMAGE
		-- ========================
		if dist <= ProbeConfig.DamageRadius then
			local damageScale = 1 - (dist / ProbeConfig.DamageRadius)
			local damage = ProbeConfig.DamagePerSecond * damageScale * ProbeConfig.UpdateRate
			health = math.max(0, health - damage)

			-- Flash body red when taking damage
			body.Color = Color3.fromRGB(255, 50, 50)
			task.delay(0.1, function()
				if body and body.Parent then
					body.Color = Color3.fromRGB(255, 140, 0)
				end
			end)
		end

		-- ========================
		-- UPDATE GUI
		-- ========================
		local healthPercent = health / ProbeConfig.MaxHealth

		-- Health bar color (green → yellow → red)
		local barColor
		if healthPercent > 0.6 then
			barColor = Color3.fromRGB(0, 200, 80)
		elseif healthPercent > 0.3 then
			barColor = Color3.fromRGB(230, 180, 0)
		else
			barColor = Color3.fromRGB(220, 40, 40)
		end

		guiRefs.healthFill.Size = UDim2.new(healthPercent, 0, 1, 0)
		guiRefs.healthFill.BackgroundColor3 = barColor
		guiRefs.healthLabel.Text = string.format("HP: %d / %d", math.floor(health), ProbeConfig.MaxHealth)
		guiRefs.windLabel.Text = string.format("💨 Wind: %d mph", windSpeed)
		guiRefs.efLabel.Text = "Rating: " .. efRating
		guiRefs.efLabel.TextColor3 = efColor
		guiRefs.moneyLabel.Text = string.format("💰 Earned: $%d", math.floor(totalEarned))

		-- ========================
		-- PROBE DESTROYED
		-- ========================
		if health <= 0 then
			alive = false
			connection:Disconnect()

			-- Explosion effect
			local explosion = Instance.new("Explosion")
			explosion.Position = body.Position
			explosion.BlastRadius = 10
			explosion.BlastPressure = 0
			explosion.ExplosionType = Enum.ExplosionType.NoCraters
			explosion.Parent = workspace

			-- Notify owner
			if owner and owner.Parent then
				local msg = Instance.new("Message")
				msg.Text = "💥 Your Twistex Probe has been destroyed! Total earned: $" .. math.floor(totalEarned)
				msg.Parent = owner
				game:GetService("Debris"):AddItem(msg, 5)
			end

			task.wait(0.5)
			probeModel:Destroy()
		end
	end)
end

-- ========================
-- PROBE SPAWNER
-- Call this to deploy a probe for a player
-- ========================
local activeTornadoPosition = nil  -- Updated by tornado script

-- Expose tornado position so probe can track it
local function setTornadoPosition(pos)
	activeTornadoPosition = pos
end

local function getTornadoPosition()
	return activeTornadoPosition
end

-- Export for tornado script to call
_G.SetTornadoPosition = setTornadoPosition

-- Deploy probe when player touches a "DeployPad" or via command
local function deployProbe(player, position)
	-- Remove any existing probe for this player
	for _, obj in workspace:GetChildren() do
		if obj.Name == "TwistexProbe_" .. player.Name then
			obj:Destroy()
		end
	end

	local probeModel, body, billboardGui, guiRefs = createProbe(
		position or Vector3.new(0, 3, 30),
		player
	)

	runProbe(probeModel, body, guiRefs, player, getTornadoPosition)
end

-- ========================
-- DEPLOY TRIGGER
-- Players can deploy by pressing "E" near a deploy zone
-- Or auto-deploy for testing
-- ========================

-- Auto deploy for all players when they join (for testing)
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		task.wait(2)
		-- Deploy probe 30 studs in front of spawn
		local hrp = character:WaitForChild("HumanoidRootPart")
		local deployPos = hrp.Position + hrp.CFrame.LookVector * 15 + Vector3.new(0, 0, 0)
		deployProbe(player, deployPos)
	end)
end)
