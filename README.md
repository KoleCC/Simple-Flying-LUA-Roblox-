# Simple-Flying-LUA-Roblox-
This is an simple fly Lua created by KoleCC, And also pls comment to report bugs in my Lua creation. Basically This is how it's work:                                               • Press F To Toggle Fly (Press it 2 times and you unfly)                                                                                       • Have Fun

------------------------COPY THIS---------------------------

local player = game.Players.LocalPlayer
local userInput = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local character = player.Character or player.CharacterAdded:Wait()
local humanoid
local rootPart

local flyEnabled = false
local flySpeed = 35

local bodyVelocity = Instance.new("BodyVelocity")
bodyVelocity.MaxForce = Vector3.new(1000000, 1000000, 1000000)
bodyVelocity.Velocity = Vector3.zero

-- Double-tap detection
local lastFPressTime = 0
local doubleTapDelay = 0.3

-- Animation variables
local flyAnimation
local idleAnimation

local function createFlyAnimation()
	local anim = Instance.new("Animation")
	anim.AnimationId = "rbxassetid://507777686" -- Falling animation (you can change this)
	return anim
end

local function playAnimation(anim)
	if not humanoid then return end
	
	local animTrack = humanoid:LoadAnimation(anim)
	animTrack:Play()
	animTrack.Looped = true
	
	return animTrack
end

local function stopAnimation(animTrack)
	if animTrack then
		animTrack:Stop()
	end
end

local function setupCharacter()
	character = player.Character or player.CharacterAdded:Wait()
	humanoid = character:WaitForChild("Humanoid")
	rootPart = character:WaitForChild("HumanoidRootPart")

	bodyVelocity.Parent = rootPart
	
	-- Create fly animation
	flyAnimation = createFlyAnimation()
	idleAnimation = nil
end

setupCharacter()

player.CharacterAdded:Connect(function(newCharacter)
	character = newCharacter
	setupCharacter()
end)

local currentFlyAnimation = nil

local function enableFly()
	flyEnabled = true
	
	if humanoid and rootPart then
		humanoid.PlatformStand = true
		
		-- Play fly animation
		if flyAnimation then
			currentFlyAnimation = playAnimation(flyAnimation)
		end
		
		-- Tilt character forward slightly
		rootPart.CFrame = rootPart.CFrame * CFrame.Angles(math.rad(45), 0, 0)
		
		print("Flying enabled!")
	end
end

local function disableFly()
	flyEnabled = false
	
	if humanoid and rootPart then
		humanoid.PlatformStand = false
		bodyVelocity.Velocity = Vector3.zero
		
		-- Stop fly animation
		stopAnimation(currentFlyAnimation)
		currentFlyAnimation = nil
		
		-- Reset character rotation
		rootPart.CFrame = CFrame.new(rootPart.Position, rootPart.Position + humanoid.MoveDirection)
		
		print("Flying disabled!")
	end
end

local function updateFly()
	if not flyEnabled or not humanoid or not rootPart then return end

	local camera = workspace.CurrentCamera
	if not camera then return end

	local direction = Vector3.zero
	local forward = camera.CFrame.LookVector
	local right = camera.CFrame.RightVector

	forward = Vector3.new(forward.X, 0, forward.Z)
	right = Vector3.new(right.X, 0, right.Z)

	if forward.Magnitude > 0 then
		forward = forward.Unit
	end
	if right.Magnitude > 0 then
		right = right.Unit
	end

	if userInput:IsKeyDown(Enum.KeyCode.W) then
		direction += forward
	end
	if userInput:IsKeyDown(Enum.KeyCode.S) then
		direction -= forward
	end
	if userInput:IsKeyDown(Enum.KeyCode.A) then
		direction -= right
	end
	if userInput:IsKeyDown(Enum.KeyCode.D) then
		direction += right
	end
	if userInput:IsKeyDown(Enum.KeyCode.Space) then
		direction += Vector3.new(0, 1, 0)
	end
	if userInput:IsKeyDown(Enum.KeyCode.LeftControl) then
		direction += Vector3.new(0, -1, 0)
	end

	if direction.Magnitude > 0 then
		direction = direction.Unit * flySpeed
	end

	bodyVelocity.Velocity = direction
	
	-- Rotate character to face flight direction
	if direction.Magnitude > 0 then
		local lookDirection = direction.Unit
		rootPart.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + lookDirection)
	end
end

userInput.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end

	if input.KeyCode == Enum.KeyCode.F then
		local currentTime = tick()
		
		-- Check for double-tap
		if currentTime - lastFPressTime <= doubleTapDelay then
			-- Double-tap detected
			if flyEnabled then
				disableFly()
			end
			lastFPressTime = 0 -- Reset timer
		else
			-- Single tap
			if not flyEnabled then
				enableFly()
			end
			lastFPressTime = currentTime
		end
	end
end)

runService.RenderStepped:Connect(function()
	if flyEnabled then
		updateFly()
	end
end)