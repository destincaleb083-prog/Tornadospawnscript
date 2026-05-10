-- StudioLiteTornado.lua
-- Full script: Thick particles, storm cloud, screen shake, sounds, warning
-- Place in ServerScriptService

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ========================
-- SETTINGS
-- ========================
local Config = {
	SpawnHeight = 350,
	TouchdownTime = 6,
	MoveSpeed = 12,
	WanderStrength = 45,
	NoiseScale = 0.2,
	LifeTime = 75,
	PullRadius = 70,
	DamageRadius = 18,
	DamageAmount = 6,
	FunnelSegments = 18,
	FunnelHeight = 180,
	DebrisCount = 22,
	WarningTime = 10,
	MapCenterX = 0,
	MapCenterZ = 0,
}

-- ========================
-- REMOTE EVENT (Screen Shake)
-- ========================
local shakeEvent = Instance.new("RemoteEvent")
shakeEvent.Name = "TornadoShake"
shakeEvent.Parent = ReplicatedStorage

-- ========================
-- WARNING SYSTEM
-- ========================
local function playWarning(position)
	local warnPart = Instance.new("Part")
	warnPart.Anchored = true
	warnPart.CanCollide = false
	warnPart.Transparency = 1
	warnPart.CFrame = CFrame.new(position)
	warnPart.Parent = workspace

	local siren = Instance.new("Sound")
	siren.SoundId = "rbxassetid://9120985791"
	siren.Volume = 1.5
	siren.RollOffMaxDistance = 1000
	siren.Looped = true
	siren.Parent = warnPart
	siren:Play()

	for _, player in Players:GetPlayers() do
		local msg = Instance.new("Message")
		msg.Text = "⚠️ TORNADO WARNING! A tornado is approaching the baseplate! ⚠️"
		msg.Parent = player
		Debris:AddItem(msg, 6)
	end

	task.delay(Config.WarningTime, function()
		TweenService:Create(siren, TweenInfo.new(2), {Volume = 0}):Play()
		task.wait(2)
		warnPart:Destroy()
	end)
end

-- ========================
-- SOUND SYSTEM
-- ========================
local function setupSounds(root)
	local sounds = {}

	local roar = Instance.new("Sound")
	roar.SoundId = "rbxassetid://1048498328"
	roar.Volume = 0
	roar.RollOffMaxDistance = 400
	roar.RollOffMinDistance = 10
	roar.Looped = true
	roar.Parent = root
	roar:Play()
	sounds.roar = roar

	local wind = Instance.new("Sound")
	wind.SoundId = "rbxassetid://6042053626"
	wind.Volume = 0
	wind.RollOffMaxDistance = 300
	wind.RollOffMinDistance = 5
	wind.Looped = true
	wind.Parent = root
	wind:Play()
	sounds.wind = wind

	local rumble = Instance.new("Sound")
	rumble.SoundId = "rbxassetid://5800804592"
	rumble.Volume = 0
	rumble.RollOffMaxDistance = 250
	rumble.RollOffMinDistance = 10
	rumble.Looped = true
	rumble.Parent = root
	rumble:Play()
	sounds.rumble = rumble

	local touchdown = Instance.new("Sound")
	touchdown.SoundId = "rbxassetid://4612425531"
	touchdown.Volume = 2
	touchdown.RollOffMaxDistance = 700
	touchdown.Parent = root
	sounds.touchdown = touchdown

	local playerHit = Instance.new("Sound")
	playerHit.SoundId = "rbxassetid://5221820205"
	playerHit.Volume = 1
	playerHit.RollOffMaxDistance = 120
	playerHit.Parent = root
	sounds.playerHit = playerHit

	return sounds
end

-- ========================
-- STORM CLOUD BUILDER
-- ========================
local function buildStormCloud(position, folder)
	local cloudParts = {}

	-- Main dark storm cloud mass
	local cloudLayerData = {
		{radius = 120, height = 20,  count = 10, color = Color3.fromRGB(45, 45, 55),  trans = 0.15, size = 55},
		{radius = 90,  height = 35,  count = 8,  color = Color3.fromRGB(55, 55, 65),  trans = 0.2,  size = 45},
		{radius = 60,  height = 50,  count = 6,  color = Color3.fromRGB(65, 65, 75),  trans = 0.25, size = 38},
		{radius = 30,  height = 60,  count = 4,  color = Color3.fromRGB(75, 75, 85),  trans = 0.3,  size = 30},
	}

	for _, layer in cloudLayerData do
		for i = 1, layer.count do
			local angle = (i / layer.count) * math.pi * 2
			local cx = position.X + math.cos(angle) * layer.radius
			local cz = position.Z + math.sin(angle) * layer.radius
			local cy = position.Y + Config.FunnelHeight + layer.height

			local cloudPart = Instance.new("Part")
			cloudPart.Size = Vector3.new(layer.size, layer.size * 0.5, layer.size)
			cloudPart.Shape = Enum.PartType.Ball
			cloudPart.Anchored = true
			cloudPart.CanCollide = false
			cloudPart.CastShadow = false
			cloudPart.Color = layer.color
			cloudPart.Material = Enum.Material.SmoothPlastic
			cloudPart.Transparency = layer.trans
			cloudPart.CFrame = CFrame.new(cx, cy, cz)
			cloudPart.Parent = folder

			-- Cloud particle emitter for wispy effect
			local cloudEmitter = Instance.new("ParticleEmitter")
			cloudEmitter.Texture = "rbxassetid://243660364"
			cloudEmitter.Color = ColorSequence.new({
				ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 40, 50)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 70, 80)),
			})
			cloudEmitter.Transparency = NumberSequence.new({
				NumberSequenceKeypoint.new(0, 0.3),
				NumberSequenceKeypoint.new(0.5, 0.6),
				NumberSequenceKeypoint.new(1, 1),
			})
			cloudEmitter.Size = NumberSequence.new({
				NumberSequenceKeypoint.new(0, layer.size * 0.4),
				NumberSequenceKeypoint.new(1, layer.size * 0.8),
			})
			cloudEmitter.Rate = 15
			cloudEmitter.Speed = NumberRange.new(5, 15)
			cloudEmitter.SpreadAngle = Vector2.new(180, 180)
			cloudEmitter.Lifetime = NumberRange.new(1, 2.5)
			cloudEmitter.RotSpeed = NumberRange.new(-30, 30)
			cloudEmitter.Rotation = NumberRange.new(0, 360)
			cloudEmitter.LightEmission = 0
			cloudEmitter.LightInfluence = 0.9
			cloudEmitter.Parent = cloudPart

			table.insert(cloudParts, {part = cloudPart, baseAngle = angle, radius = layer.radius, height = layer.height})
		end
	end

	-- Central dark funnel connection (where funnel meets cloud)
	local connectorAnchor = Instance.new("Part")
	connectorAnchor.Size = Vector3.new(1, 1, 1)
	connectorAnchor.Anchored = true
	connectorAnchor.CanCollide = false
	connectorAnchor.Transparency = 1
	connectorAnchor.CFrame = CFrame.new(position.X, position.Y + Config.FunnelHeight, position.Z)
	connectorAnchor.Parent = folder

	local connectorEmitter = Instance.new("ParticleEmitter")
	connectorEmitter.Texture = "rbxassetid://243660364"
	connectorEmitter.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 60)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 80, 95)),
	})
	connectorEmitter.Transparency = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 0.1),
		NumberSequenceKeypoint.new(0.5, 0.4),
		NumberSequenceKeypoint.new(1, 1),
	})
	connectorEmitter.Size = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 40),
		NumberSequenceKeypoint.new(1, 80),
	})
	connectorEmitter.Rate = 60
	connectorEmitter.Speed = NumberRange.new(10, 25)
	connectorEmitter.SpreadAngle = Vector2.new(180, 40)
	connectorEmitter.Lifetime = NumberRange.new(1, 2)
	connectorEmitter.RotSpeed = NumberRange.new(-60, 60)
	connectorEmitter.Rotation = NumberRange.new(0, 360)
	connectorEmitter.Parent = connectorAnchor

	table.insert(cloudParts, {part = connectorAnchor, baseAngle = 0, radius = 0, height = Config.FunnelHeight})

	return cloudParts
end

-- ========================
-- FUNNEL BUILDER (Thick Particles)
-- ========================
local function buildFunnel(startPos)
	local folder = Instance.new("Folder")
	folder.Name = "StudioLiteTornado"
	folder.Parent = workspace

	local funnelParts = {}

	for i = 1, Config.FunnelSegments do
		local t = i / Config.FunnelSegments
		local width = math.lerp(8, 55, t ^ 1.2)
		local height = startPos.Y + i * (Config.FunnelHeight / Config.FunnelSegments)

		local anchor = Instance.new("Part")
		anchor.Size = Vector3.new(1, 1, 1)
		anchor.Anchored = true
		anchor.CanCollide = false
		anchor.Transparency = 1
		anchor.CFrame = CFrame.new(startPos.X, height, startPos.Z)
		anchor.Parent = folder

		local baseColor = Color3.fromRGB(90, 70, 50)
		local midColor  = Color3.fromRGB(160, 145, 130)
		local topColor  = Color3.fromRGB(220, 215, 210)
		local c0 = baseColor:Lerp(midColor, t)
		local c1 = midColor:Lerp(topColor, t)

		-- Outer emitter
		local outerEmitter = Instance.new("ParticleEmitter")
		outerEmitter.Texture = "rbxassetid://243660364"
		outerEmitter.Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, c0),
			ColorSequenceKeypoint.new(0.5, c1),
			ColorSequenceKeypoint.new(1, topColor),
		})
		outerEmitter.Transparency = NumberSequence.new({
			NumberSequenceKeypoint.new(0, math.lerp(0.0, 0.3, t)),
			NumberSequenceKeypoint.new(0.4, math.lerp(0.2, 0.5, t)),
			NumberSequenceKeypoint.new(1, 1),
		})
		outerEmitter.Size = NumberSequence.new({
			NumberSequenceKeypoint.new(0, width * 1.2),
			NumberSequenceKeypoint.new(0.5, width * 1.6),
			NumberSequenceKeypoint.new(1, width * 2.0),
		})
		outerEmitter.Rate = math.lerp(200, 80, t)
		outerEmitter.Speed = NumberRange.new(width * 0.6, width * 1.2)
		outerEmitter.SpreadAngle = Vector2.new(180, 180)
		outerEmitter.Lifetime = NumberRange.new(0.4, 0.9)
		outerEmitter.RotSpeed = NumberRange.new(-180, 180)
		outerEmitter.Rotation = NumberRange.new(0, 360)
		outerEmitter.LightEmission = 0
		outerEmitter.LightInfluence = 0.9
		outerEmitter.LockedToPart = false
		outerEmitter.Parent = anchor

		-- Inner core emitter
		local innerEmitter = Instance.new("ParticleEmitter")
		innerEmitter.Texture = "rbxassetid://243660364"
		innerEmitter.Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.fromRGB(55, 40, 30)),
			ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 80, 65)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 135, 120)),
		})
		innerEmitter.Transparency = NumberSequence.new({
			NumberSequenceKeypoint.new(0, 0.0),
			NumberSequenceKeypoint.new(0.5, 0.2),
			NumberSequenceKeypoint.new(1, 1),
		})
		innerEmitter.Size = NumberSequence.new({
			NumberSequenceKeypoint.new(0, width * 0.6),
			NumberSequenceKeypoint.new(1, width * 0.9),
		})
		innerEmitter.Rate = math.lerp(250, 100, t)
		innerEmitter.Speed = NumberRange.new(width * 0.2, width * 0.5)
		innerEmitter.SpreadAngle = Vector2.new(180, 180)
		innerEmitter.Lifetime = NumberRange.new(0.2, 0.5)
		innerEmitter.RotSpeed = NumberRange.new(-500, 500)
		innerEmitter.Rotation = NumberRange.new(0, 360)
		innerEmitter.LightEmission = 0
		innerEmitter.LightInfluence = 1
		innerEmitter.LockedToPart = false
		innerEmitter.Parent = anchor

		table.insert(funnelParts, {part = anchor, index = i})
	end

	-- Ground dust shroud
	local groundAnchor = Instance.new("Part")
	groundAnchor.Size = Vector3.new(1, 1, 1)
	groundAnchor.Anchored = true
	groundAnchor.CanCollide = false
	groundAnchor.Transparency = 1
	groundAnchor.CFrame = CFrame.new(startPos.X, startPos.Y + 2, startPos.Z)
	groundAnchor.Parent = folder

	local groundShroud = Instance.new("ParticleEmitter")
	groundShroud.Texture = "rbxassetid://243660364"
	groundShroud.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 60, 40)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(130, 105, 80)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 160, 140)),
	})
	groundShroud.Transparency = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 0.05),
		NumberSequenceKeypoint.new(0.5, 0.4),
		NumberSequenceKeypoint.new(1, 1),
	})
	groundShroud.Size = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 35),
		NumberSequenceKeypoint.new(0.5, 60),
		NumberSequenceKeypoint.new(1, 90),
	})
	groundShroud.Rate = 300
	groundShroud.Speed = NumberRange.new(30, 70)
	groundShroud.SpreadAngle = Vector2.new(180, 15)
	groundShroud.Lifetime = NumberRange.new(1.5, 3)
	groundShroud.RotSpeed = NumberRange.new(-80, 80)
	groundShroud.Rotation = NumberRange.new(0, 360)
	groundShroud.LightEmission = 0
	groundShroud.LightInfluence = 0.8
	groundShroud.LockedToPart = false
	groundShroud.Parent = groundAnchor

	-- Ground roll
	local groundRoll = Instance.new("ParticleEmitter")
	groundRoll.Texture = "rbxassetid://243660364"
	groundRoll.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 75, 55)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(160, 140, 115)),
	})
	groundRoll.Transparency = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 0.1),
		NumberSequenceKeypoint.new(0.7, 0.6),
		NumberSequenceKeypoint.new(1, 1),
	})
	groundRoll.Size = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 25),
		NumberSequenceKeypoint.new(1, 65),
	})
	groundRoll.Rate = 150
	groundRoll.Speed = NumberRange.new(15, 35)
	groundRoll.SpreadAngle = Vector2.new(180, 5)
	groundRoll.Lifetime = NumberRange.new(2, 4)
	groundRoll.RotSpeed = NumberRange.new(-50, 50)
	groundRoll.Rotation = NumberRange.new(0, 360)
	groundRoll.LockedToPart = false
	groundRoll.Parent = groundAnchor

	table.insert(funnelParts, {part = groundAnchor, index = 0})

	-- Build storm cloud at top
	local cloudParts = buildStormCloud(startPos, folder)

	return funnelParts, folder, cloudParts
end

-- ========================
-- DEBRIS SYSTEM
-- ========================
local function spawnDebris(startPos)
	local debrisList = {}
	local partColors = {
		Color3.fromRGB(196, 40, 28),
		Color3.fromRGB(13, 105, 172),
		Color3.fromRGB(245, 205, 48),
		Color3.fromRGB(39, 70, 45),
		Color3.fromRGB(196, 196, 196),
		Color3.fromRGB(255, 255, 255),
	}

	for i = 1, Config.DebrisCount do
		local debris = Instance.new("Part")
		local s = math.random(1, 4)
		debris.Size = Vector3.new(s, s, s)
		debris.Color = partColors[math.random(1, #partColors)]
		debris.Material = Enum.Material.SmoothPlastic
		debris.Anchored = true
		debris.CanCollide = false
		debris.CastShadow = false

		local angle = math.random() * math.pi * 2
		local radius = math.random(10, 45)
		local height = math.random(5, Config.FunnelHeight - 15)

		debris.CFrame = CFrame.new(
			startPos.X + math.cos(angle) * radius,
			startPos.Y + height,
			startPos.Z + math.sin(angle) * radius
		)
		debris.Parent = workspace

		table.insert(debrisList, {
			part = debris,
			angle = angle,
			radius = radius,
			height = height,
			orbitSpeed = math.random(2, 6) + math.random()
		})
	end

	return debrisList
end

-- ========================
-- MAIN TORNADO SPAWNER
-- ========================
local function spawnTornado()
	local spawnX = Config.MapCenterX
	local spawnZ = Config.MapCenterZ
	local groundY = 3

	local spawnPos = Vector3.new(spawnX, groundY, spawnZ)

	playWarning(Vector3.new(spawnX, groundY + 10, spawnZ))
	task.wait(Config.WarningTime)

	-- Root anchor
	local root = Instance.new("Part")
	root.Size = Vector3.new(1, 1, 1)
	root.Anchored = true
	root.CanCollide = false
	root.Transparency = 1
	root.CFrame = CFrame.new(spawnX, Config.SpawnHeight, spawnZ)
	root.Parent = workspace

	local funnelParts, funnelFolder, cloudParts = buildFunnel(spawnPos)
	local debrisList = spawnDebris(spawnPos)
	local sounds = setupSounds(root)

	-- Touchdown tween
	local touchdownComplete = false

	local touchdownTween = TweenService:Create(root, TweenInfo.new(
		Config.TouchdownTime,
		Enum.EasingStyle.Quad,
		Enum.EasingDirection.In
	), {CFrame = CFrame.new(spawnX, groundY, spawnZ)})
	touchdownTween:Play()

	TweenService:Create(sounds.roar, TweenInfo.new(Config.TouchdownTime), {Volume = 1.3}):Play()
	TweenService:Create(sounds.wind, TweenInfo.new(Config.TouchdownTime), {Volume = 0.9}):Play()
	TweenService:Create(sounds.rumble, TweenInfo.new(Config.TouchdownTime * 0.5), {Volume = 0.7}):Play()

	touchdownTween.Completed:Connect(function()
		touchdownComplete = true
		sounds.touchdown:Play()

		for _, player in Players:GetPlayers() do
			local msg = Instance.new("Message")
			msg.Text = "🌪️ TORNADO HAS TOUCHED DOWN ON THE BASEPLATE!"
			msg.Parent = player
			Debris:AddItem(msg, 4)
		end
	end)

	-- Main loop
	local elapsed = 0
	local noiseOffset = math.random(0, 1000)
	local currentPos = Vector3.new(spawnX, groundY, spawnZ)
	local mapBound = 500
	local cloudRotation = 0

	local connection
	connection = RunService.Heartbeat:Connect(function(dt)
		elapsed += dt
		cloudRotation += dt * 0.3  -- slow cloud rotation

		-- Despawn
		if elapsed >= Config.LifeTime then
			connection:Disconnect()
			TweenService:Create(sounds.roar, TweenInfo.new(3), {Volume = 0}):Play()
			TweenService:Create(sounds.wind, TweenInfo.new(3), {Volume = 0}):Play()
			TweenService:Create(sounds.rumble, TweenInfo.new(3), {Volume = 0}):Play()

			task.wait(1)
			for _, player in Players:GetPlayers() do
				local msg = Instance.new("Message")
				msg.Text = "✅ The tornado has dissipated. The baseplate is safe."
				msg.Parent = player
				Debris:AddItem(msg, 4)
			end

			task.wait(2)
			funnelFolder:Destroy()
			root:Destroy()
			for _, d in debrisList do
				if d.part and d.part.Parent then
					d.part:Destroy()
				end
			end
			return
		end

		-- Unpredictable movement
		local noiseX = math.noise(elapsed * Config.NoiseScale + noiseOffset, 0) * Config.WanderStrength
		local noiseZ = math.noise(0, elapsed * Config.NoiseScale + noiseOffset) * Config.WanderStrength
		local moveDir = Vector3.new(noiseX, 0, noiseZ).Unit
		local nextPos = currentPos + moveDir * Config.MoveSpeed * dt

		-- Bounce off map bounds
		if math.abs(nextPos.X) > mapBound then
			nextPos = Vector3.new(-nextPos.X * 0.5, nextPos.Y, nextPos.Z)
		end
		if math.abs(nextPos.Z) > mapBound then
			nextPos = Vector3.new(nextPos.X, nextPos.Y, -nextPos.Z * 0.5)
		end

		currentPos = nextPos
		local rootY = touchdownComplete and groundY or root.CFrame.Y
		root.CFrame = CFrame.new(currentPos.X, rootY, currentPos.Z)

		-- Update funnel segments
		for _, seg in funnelParts do
			seg.part.CFrame = CFrame.new(
				currentPos.X,
				rootY + seg.index * (Config.FunnelHeight / Config.FunnelSegments),
				currentPos.Z
			)
		end

		-- Update storm cloud (rotates slowly around tornado center)
		for _, cloud in cloudParts do
			if cloud.radius > 0 then
				local newAngle = cloud.baseAngle + cloudRotation
				cloud.part.CFrame = CFrame.new(
					currentPos.X + math.cos(newAngle) * cloud.radius,
					rootY + cloud.height,
					currentPos.Z + math.sin(newAngle) * cloud.radius
				)
			else
				-- Connector stays centered
				cloud.part.CFrame = CFrame.new(
					currentPos.X,
					rootY + cloud.height,
					currentPos.Z
				)
			end
		end

		-- Orbit debris
		for _, d in debrisList do
			d.angle += d.orbitSpeed * dt
			d.part.CFrame = CFrame.new(
				currentPos.X + math.cos(d.angle) * d.radius,
				rootY + d.height,
				currentPos.Z + math.sin(d.angle) * d.radius
			)
		end

		-- Player interactions
		for _, player in Players:GetPlayers() do
			local char = player.Character
			if not char then continue end
			local hrp = char:FindFirstChild("HumanoidRootPart")
			local hum = char:FindFirstChild("Humanoid")
			if not hrp or not hum then continue end

			local flat = Vector3.new(currentPos.X, hrp.Position.Y, currentPos.Z)
			local dist = (hrp.Position - flat).Magnitude

			-- Pull
			if dist < Config.PullRadius and dist > Config.DamageRadius then
				local pullDir = (flat - hrp.Position).Unit
				local strength = (1 - dist / Config.PullRadius) * 55
				hrp.AssemblyLinearVelocity = hrp.AssemblyLinearVelocity + pullDir * strength * dt
			end

			-- Damage and launch
			if dist < Config.DamageRadius then
				hum:TakeDamage(Config.DamageAmount * dt)
				local throwDir = Vector3.new(
					math.random(-1, 1), 2, math.random(-1, 1)
				).Unit
				hrp.AssemblyLinearVelocity = throwDir * 75
				if not sounds.playerHit.IsPlaying then
					sounds.playerHit:Play()
				end
			end

			-- Screen shake
			if dist < Config.PullRadius * 1.5 then
				local shakeIntensity = math.clamp(
					1 - dist / (Config.PullRadius * 1.5), 0, 1
				)
				shakeEvent:FireClient(player, shakeIntensity)
			end

			-- Volume scale
			local vol = math.clamp(1 - dist / (Config.PullRadius * 1.5), 0.15, 1)
			sounds.roar.Volume = 1.3 * vol
		end
	end)
end

-- ========================
-- START
-- ========================
task.wait(5)
spawnTornado()
