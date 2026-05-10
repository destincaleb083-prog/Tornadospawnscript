-- Create RemoteEvent for screen shake
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local shakeEvent = Instance.new("RemoteEvent")
shakeEvent.Name = "TornadoShake"
shakeEvent.Parent = ReplicatedStorage
