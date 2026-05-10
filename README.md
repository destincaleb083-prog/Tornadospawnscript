-- ========================
-- REALISTIC WEDGE TORNADO FUNNEL
-- Based on real tornado visual reference
-- ========================
local function buildFunnel(startPos)
	local folder = Instance.new("Folder")
	folder.Name = "StudioLiteTornado"
	folder.Parent = workspace

	local funnelParts = {}

	for i = 1, Config.FunnelSegments do
		local t = i / Config.FunnelSegments

		-- Narrow rope at base, wide wedge at top (matches photo)
		local width = math.lerp(1.5, 45, t ^ 1.8)  -- exponential flare like real tornado
		local height = startPos.Y + i * (Config.FunnelHeight / Config.FunnelSegments)

		local anchor = Instance.new("Part")
		anchor.Size = Vector3.new(1, 1, 1)
		anchor.Anchored = true
		anchor.CanCollide = false
		anchor.Transparency = 1
		anchor.CFrame = CFrame.new(startPos.X, height, startPos.Z)
		anchor.Parent = folder

		-- Color transitions from dark brown (base) to white/grey (top) like the photo
		local baseColor = Color3.fromRGB(90, 70, 50)      -- dark brown at ground
		local midColor  = Color3.fromRGB(160, 145, 130)   -- tan in middle
		local topColor  = Color3.fromRGB(220, 215, 210)   -- near white at top

		local c0 = baseColor:Lerp(midColor, t)
		local c1 = midColor:Lerp(topColor, t)

		-- Outer funnel wall emitter (dense, opaque)
		local outerEmitter = Instance.new("ParticleEmitter")
		outerEmitter.Texture = "rbxassetid://243660364"
		outerEmitter.Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, c0),
			ColorSequenceKeypoint.new(0.5, c1),
			ColorSequenceKeypoint.new(1, topColor),
		})
		outerEmitter.Transparency = NumberSequence.new({
			NumberSequenceKeypoint.new(0, math.lerp(0.0, 0.3, t)),  -- very opaque at base
			NumberSequenceKeypoint.new(0.4, math.lerp(0.2, 0.5, t)),
			NumberSequenceKeypoint.new(1, 1),
		})
		outerEmitter.Size = NumberSequence.new({
			NumberSequenceKeypoint.new(0, width * 0.5),
			NumberSequenceKeypoint.new(0.5, width * 0.8),
			NumberSequenceKeypoint.new(1, width * 1.1),
		})
		outerEmitter.Rate = math.lerp(80, 25, t)   -- dense at base, lighter at top
		outerEmitter.Speed = NumberRange.new(width * 0.6, width * 1.2)
		outerEmitter.SpreadAngle = Vector2.new(180, 180)
		outerEmitter.Lifetime = NumberRange.new(0.4, 0.9)
		outerEmitter.RotSpeed = NumberRange.new(-180, 180)
		outerEmitter.Rotation = NumberRange.new(0, 360)
		outerEmitter.LightEmission = 0
		outerEmitter.LightInfluence = 0.9
		outerEmitter.LockedToPart = false
		outerEmitter.Parent = anchor

		-- Inner dark core (the dense rope visible in the photo)
		local innerEmitter = Instance.new("ParticleEmitter")
		innerEmitter.Texture = "rbxassetid://243660364"
		innerEmitter.Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.fromRGB(55, 40, 30)),   -- very dark at base
			ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 80, 65)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 135, 120)),
		})
		innerEmitter.Transparency = NumberSequence.new({
			NumberSequenceKeypoint.new(0, 0.0),   -- fully opaque core
			NumberSequenceKeypoint.new(0.5, 0.2),
			NumberSequenceKeypoint.new(1, 1),
		})
		innerEmitter.Size = NumberSequence.new({
			NumberSequenceKeypoint.new(0, width * 0.15),
			NumberSequenceKeypoint.new(1, width * 0.35),
		})
		innerEmitter.Rate = math.lerp(100, 35, t)
		innerEmitter.Speed = NumberRange.new(width * 0.2, width * 0.5)
		innerEmitter.SpreadAngle = Vector2.new(180, 180)
		innerEmitter.Lifetime = NumberRange.new(0.2, 0.5)
		innerEmitter.RotSpeed = NumberRange.new(-500, 500)  -- fast spin in core
		innerEmitter.Rotation = NumberRange.new(0, 360)
		innerEmitter.LightEmission = 0
		innerEmitter.LightInfluence = 1
		innerEmitter.LockedToPart = false
		innerEmitter.Parent = anchor

		table.insert(funnelParts, {part = anchor, index = i})
	end

	-- ========================
	-- GROUND DUST SHROUD
	-- Wide spreading dust cloud at base (like in the photo)
	-- ========================
	local groundAnchor = Instance.new("Part")
	groundAnchor.Size = Vector3.new(1, 1, 1)
	groundAnchor.Anchored = true
	groundAnchor.CanCollide = false
	groundAnchor.Transparency = 1
	groundAnchor.CFrame = CFrame.new(startPos.X, startPos.Y + 2, startPos.Z)
	groundAnchor.Parent = folder

	-- Wide outward dust cloud spreading from base
	local groundShroud = Instance.new("ParticleEmitter")
	groundShroud.Texture = "rbxassetid://243660364"
	groundShroud.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 60, 40)),    -- dark base dust
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(130, 105, 80)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 160, 140)),
	})
	groundShroud.Transparency = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 0.05),   -- very dense at ground
		NumberSequenceKeypoint.new(0.5, 0.4),
		NumberSequenceKeypoint.new(1, 1),
	})
	groundShroud.Size = NumberSequence.new({
		NumberSequenceKeypoint.new(0, 15),
		NumberSequenceKeypoint.new(0.5, 30),
		NumberSequenceKeypoint.new(1, 50),     -- spreads wide outward
	})
	groundShroud.Rate = 120                    -- very dense ground dust
	groundShroud.Speed = NumberRange.new(30, 70)
	groundShroud.SpreadAngle = Vector2.new(180, 15)  -- mostly horizontal spread
	groundShroud.Lifetime = NumberRange.new(1.5, 3)  -- lingers longer
	groundShroud.RotSpeed = NumberRange.new(-80, 80)
	groundShroud.Rotation = NumberRange.new(0, 360)
	groundShroud.LightEmission = 0
	groundShroud.LightInfluence = 0.8
	groundShroud.LockedToPart = false
	groundShroud.Parent = groundAnchor

	-- Secondary ground roll (the low rolling dust visible in the photo)
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
		NumberSequenceKeypoint.new(0, 10),
		NumberSequenceKeypoint.new(1, 35),
	})
	groundRoll.Rate = 60
	groundRoll.Speed = NumberRange.new(15, 35)
	groundRoll.SpreadAngle = Vector2.new(180, 5)   -- very flat, hugs the ground
	groundRoll.Lifetime = NumberRange.new(2, 4)
	groundRoll.RotSpeed = NumberRange.new(-50, 50)
	groundRoll.Rotation = NumberRange.new(0, 360)
	groundRoll.LockedToPart = false
	groundRoll.Parent = groundAnchor

	table.insert(funnelParts, {part = groundAnchor, index = 0})

	return funnelParts, folder
end
