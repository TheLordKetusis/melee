local services = setmetatable({},{__index=
	function(self,service)
		self = game:GetService(service) or nil
		return self
	end,
})

local StarterGui = services.StarterGui
local UserInputService = services.UserInputService
local repl = services.ReplicatedStorage
local players = services.Players
local RunService= services.RunService

function notif(title,text)
	StarterGui:SetCore("SendNotification", {
		Title = title;
		Text = text
	})
end

local camera = workspace.CurrentCamera
local ambient = Color3.new(80/255,80/255,80/255)
local brightness = 2
local plr = players.LocalPlayer
local Mouse = plr:GetMouse()

local CamUtil = loadstring(game:HttpGet("https://raw.githubusercontent.com/Minhtuan0312/lego-scripts/main/c", true))()
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/Minhtuan0312/lego-scripts/main/ui-library", true))()
local Window = Library:CreateWindow("cool tools")

Library.flags['connections'] = {}

function UpdateLighting()
	if not Library.flags['fullbright'] then
		game.Lighting.OutdoorAmbient = ambient;
		game.Lighting.Brightness = 2
		if game.Lighting:FindFirstChild("NightVisionEffect") then
			game.Lighting:FindFirstChild("NightVisionEffect").Enabled = false;
		end
		game.Lighting.Ambient = ambient;
		return false
	end;
	if game.Lighting:FindFirstChild("NightVisionEffect") then
		game.Lighting:FindFirstChild("NightVisionEffect").Enabled = true;
	end
	game.Lighting.NightVisionEffect.TintColor = Color3.new(0.5, 1, 0.5);
	game.Lighting.Ambient = Color3.new(0.75, 0.75, 0.75);
	game.Lighting.OutdoorAmbient = Color3.new(0.75, 0.75, 0.75);
	game.Lighting.Brightness = 1;
	return true
end;

Window:AddToggle({
	text = "FullBright",
	flag = "fullbright",
	state = false,
	callback = function(a)
		return a
	end
})

Window:AddToggle({
	text = "Clean body",
	flag = "cleanbody",
	state = false,
	callback = function(a)
		if Library.flags['cleanbody'] then
			Library.flags["connections"]['cleanbody'] = RunService.RenderStepped:Connect(function()
				for _,v in pairs(game.Workspace.IgnoreList:GetChildren()) do
					if v:IsA("Model") then
						v:Destroy()
					end
				end
			end)
		else
			Library.flags["connections"]['cleanbody']:Disconnect()
		end
	end
})

function WaitForChildOfClass(parent,className,timeOut)
	local waitTime = 0
	local warned = false
	repeat
		local obj = parent:FindFirstChildOfClass(className)
		if obj then 
			return obj 
		else
			waitTime = waitTime + RunService.RenderStepped:Wait()
			if timeOut then 
				if waitTime > timeOut then return nil end
			elseif not warned then
				if waitTime > 5 then 
					warn("Infinite yield possible waiting on",parent:GetFullName() .. ":FindFirstChildOfClass(\"".. className .. "\")")
					warned = true
				end
			end
		end
	until false
end

Window:AddToggle({
	text = "Particle hater",
	flag = "cleanparticle",
	state = false,
	callback = function(a)
		if Library.flags['cleanparticle'] then
			Library.flags["connections"]['cleanparticle'] = game.Workspace.IgnoreList.ChildAdded:Connect(function(new)
				if new:IsA("Part") then
					if WaitForChildOfClass(new,"ParticleEmitter",5) then
						new:ClearAllChildren()
						new.Size = Vector3.new(new.Size.X,2,new.Size.Z)
						new.Color = Color3.new(170,0,0)
						new.Material = Enum.Material.Neon
						new.Transparency = .4
					end
				end
			end)
		else
			Library.flags["connections"]['cleanparticle']:Disconnect()
		end
	end
})

local melees = {}
local Head

for _, item in pairs(repl.Tools:GetChildren()) do
	if item:FindFirstChildWhichIsA("LocalScript") and item:FindFirstChildWhichIsA("LocalScript").Name == "MeleeScript" then
		melees[item.Name] = item.Name
	end
end

local function WTS(Object)
	local ObjectVector = camera:WorldToScreenPoint(Object.Position)
	return Vector2.new(ObjectVector.X, ObjectVector.Y)
end

local function PositionToRay(Origin, Position)
	return Ray.new(Origin, (Position - Origin).Unit * 600)
end

local function GetClosestZombieFromCursor()
	local ClosestDistance = math.huge
	local zombies = workspace:FindFirstChild("Zombies")
	for _, v in pairs(zombies:GetChildren()) do
		if v ~= nil and v:FindFirstChild("Head") and v:FindFirstChild("Zombie") then
			if v.Zombie.Health ~= 0 then
				local Magnitude = (WTS(v.Head) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
				if Magnitude <= ClosestDistance then
					Head = v.Head
					ClosestDistance = Magnitude
				end
			end
		end
	end
end

local remotemelee = repl:WaitForChild("RemoteEvents"):WaitForChild("RemoteMelee");
local remotefiremelee = repl:WaitForChild("RemoteEvents"):WaitForChild("RemoteFireMelee");
local RemoteCritMelee = repl:WaitForChild("RemoteEvents"):WaitForChild("RemoteCritMelee");

local oldNameCall;
oldNameCall = hookmetamethod(game, "__namecall", function(...)
	local Self, Args = (...), ({select(2, ...)})
	if getnamecallmethod() == "Kick" and Self == plr then 
		warn("Kick bypassed")
		notif("epic","Bypassed game's kick attempt.")
		return
	elseif Library.flags['silentaim'] and getnamecallmethod() == "FindPartOnRayWithIgnoreList" and Head then
		Args[1] = PositionToRay(camera.CFrame.Position, Head.Position)
		return oldNameCall(Self, unpack(Args))
	end
	return oldNameCall(...)
end)

Window:AddToggle({
	text = "Silent aim",
	flag = "silentaim",
	state = false,
	callback = function(a)
		if Library.flags['silentaim'] then
			notif("epic","Silent aim: ON.")
			Library.flags["connections"]['Silent Aim'] = RunService.RenderStepped:Connect(function()
				GetClosestZombieFromCursor()
			end)
		else
			notif("epic","Silent aim: OFF.")
			Head = nil
			Library.flags["connections"]['Silent Aim']:Disconnect()
		end
	end
})

function meleeAura()
	local zombies = workspace:FindFirstChild("Zombies")
	local toDeal = {}
	local meleestoDeal = {"Melee"}
	for _, item in pairs(plr:FindFirstChildWhichIsA("Backpack"):GetChildren()) do
		if melees[item.Name] then
			table.insert(meleestoDeal,item)
		end
	end
	for _, v in pairs(zombies:GetChildren()) do
		if v and v:FindFirstChild("Head") and v:FindFirstChild("Zombie") then
			if v.Zombie.Health ~= 0 then
				local mod = {
					Critical = true,
					StunCrits = true,
					FireCrits = true,
					AltDamage = true,
					OverDamage = true,
					UnderStun = true,
					OverStun = true,
				}
				table.insert(toDeal,{ v.Torso, mod})
			end
		end
	end
	for _, melee in pairs(meleestoDeal) do
		RemoteCritMelee:FireServer(melee, false, {
			WalkSpeed = math.huge
		});
		remotefiremelee:FireServer(melee, toDeal);
	end
end

Window:AddToggle({
	text = "Melee aura",
	flag = "meleeaura",
	state = false,
	callback = function(a)
		if Library.flags['meleeaura'] then
			notif("epic","Melee aura: ON.")
			Library.flags["connections"]['Melee aura'] = RunService.RenderStepped:Connect(function()
				meleeAura()
			end)
		else
			notif("epic","Melee aura: OFF.")
			Library.flags["connections"]['Melee aura']:Disconnect()
		end
	end
})

local esp = Library:CreateWindow("cool esp")
local esp_target = esp:AddFolder('target')

local function destroyEsps(specific)
	local function doDestroy(parent)
		for i, target in pairs(parent) do
			if target.Line then
				target.Line:Remove()
				target.Line = nil
			end
			if target.Boxbbg then
				target.Boxbbg:Destroy()
				target.Boxbbg = nil
			end
			if target.nametagbbg then
				target.nametagbbg:Destroy()
				target.nametagbbg = nil
			end
		end
	end
	if not specific then
		for _, parent in pairs(Library.flags['tracer_']) do
			doDestroy(parent)
		end
	else
		specific(Library.flags['tracer_'][specific])
	end
end

local zombie = esp_target:AddFolder('zombie')
zombie:AddColor({
	text = 'color', 
	flag = 'zombie_color',
	color = Color3.fromRGB(255, 255, 255),
	callback = function(v)
		return v
	end
})

Library.flags['tracer_'] = {}
zombie:AddToggle({
	text = "Zombies",
	flag = "esptarget_zombie",
	state = false,
	callback = function(v)
		local function setupTracer(zombie)
			if not Library.flags['tracer_']['zombies'] then
				Library.flags['tracer_']['zombies'] = {}
			end
			
			local funcs = {}
			
			funcs.target = zombie
			funcs.nametag = function()
				local target_character = funcs.target
				if Library.flags["esptype_nametag"] and target_character and target_character:IsDescendantOf(workspace) and target_character.Parent and target_character:FindFirstChild("Zombie") and target_character:FindFirstChild("Head") and target_character:FindFirstChild("Torso") and target_character.Zombie.Health ~= 0 then
					if not funcs.nametagbbg then
						funcs.nametagbbg = Instance.new("BillboardGui", zombie.Head)
						funcs.nametagbbg.AlwaysOnTop = true
						funcs.nametagbbg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
						funcs.nametagbbg.Adornee = zombie.Head
						funcs.nametagbbg.Size = UDim2.new(2, 1, 0)
						funcs.nametagbbg.StudsOffset = Vector3.new(0, 1.7, 0)
						funcs.nametagbbg.MaxDistance = math.huge

						local TextLabel = Instance.new("TextLabel", funcs.nametagbbg)
						TextLabel.Text = funcs.target.Name
						TextLabel.Font = Enum.Font.RobotoMono
						TextLabel.Size = UDim2.new(0, 150, 0, 20)
						TextLabel.Position = UDim2.new(0.5, -75, 1, -15)
						TextLabel.TextColor3 = Library.flags['zombie_color']
						TextLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
						TextLabel.TextStrokeTransparency = 0.2
						TextLabel.TextSize = 22
						TextLabel.ZIndex = 5
						TextLabel.BackgroundTransparency = 1
					end
					
					funcs.nametagbbg.TextLabel.TextColor3 = Library.flags['zombie_color']

					return true
				else
					if funcs.nametagbbg then
						funcs.nametagbbg:Destroy()
						funcs.nametagbbg = nil
					end
					if Library.flags["esptype_nametag"] then
						return true
					else
						return false
					end
				end
			end
			funcs.box = function()
				local target_character = funcs.target
				if Library.flags["esptype_box"] and target_character and target_character:IsDescendantOf(workspace) and target_character.Parent and target_character:FindFirstChild("Zombie") and target_character:FindFirstChild("Head") and target_character:FindFirstChild("Torso") and target_character.Zombie.Health ~= 0 then
					if not funcs.Boxbbg then
						funcs.Boxbbg = Instance.new("BillboardGui", zombie)
						funcs.Boxbbg.AlwaysOnTop = true
						funcs.Boxbbg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
						funcs.Boxbbg.Adornee = zombie.Torso
						funcs.Boxbbg.Size = UDim2.new(5, 1, 5)
						funcs.Boxbbg.MaxDistance = math.huge

						local a,b,c,d = Instance.new("Frame", funcs.Boxbbg), Instance.new("Frame", funcs.Boxbbg), Instance.new("Frame", funcs.Boxbbg), Instance.new("Frame", funcs.Boxbbg)
						a.Size = UDim2.new(0, 1, 1, 0)
						a.Position = UDim2.new(0, 0, 0, 0)
						b.Size = UDim2.new(0, 1, 1, 0)
						b.Position = UDim2.new(1, -1, 0, 0)
						c.Size = UDim2.new(1, 0, 0, 1)
						c.Position = UDim2.new(0, 0, 0, 0)
						d.Size = UDim2.new(1, 0, 0, 1)
						d.Position = UDim2.new(0, 0, 1, -1)
						for i, v in pairs(funcs.Boxbbg:GetChildren()) do
							v.BackgroundColor3 = Library.flags['zombie_color']
							v.BorderColor3 = Color3.new(0,0,0)
							v.BorderSizePixel = .4
						end
					end
					
					for i, v in pairs(funcs.Boxbbg:GetChildren()) do
						v.BackgroundColor3 = Library.flags['zombie_color']
						v.BorderColor3 = Color3.new(0,0,0)
						v.BorderSizePixel = .4
					end
					
					return true
				else
					if funcs.Boxbbg then
						funcs.Boxbbg:Destroy()
						funcs.Boxbbg = nil
					end
					if Library.flags["esptype_box"] then
						return true
					else
						return false
					end
				end
			end
			funcs.tracer = function()
				local target_character = funcs.target
				if Library.flags["esptype_tracer"] and target_character and target_character:IsDescendantOf(workspace) and target_character.Parent and target_character:FindFirstChild("Zombie") and target_character:FindFirstChild("Head") and target_character:FindFirstChild("Torso") and target_character.Zombie.Health ~= 0 then
					if not funcs.Line then
						funcs.Line = Drawing.new("Line")
					end
					local HumanoidRootPart_Position, HumanoidRootPart_Size = target_character.Torso.CFrame, target_character.Torso.Size * 1
					local Vector, OnScreen = camera:WorldToViewportPoint(HumanoidRootPart_Position * CFrame.new(0, -HumanoidRootPart_Size.Y, 0).p)
					funcs.Line.Thickness = 1.15666
					funcs.Line.Transparency = .3
					funcs.Line.Color = Library.flags['zombie_color']
					funcs.Line.From = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y)
					funcs.Line.To = Vector2.new(Vector.X, Vector.Y)
					if OnScreen then
						funcs.Line.Visible = true
					else
						funcs.Line:Remove()
						funcs.Line = nil
					end
					return true
				else
					if funcs.Line then
						funcs.Line:Remove()
						funcs.Line = nil
					end
					if Library.flags["esptype_tracer"] then
						return true
					else
						return false
					end
				end
			end
			
			table.insert(Library.flags['tracer_']['zombies'],setmetatable({},{__index=funcs}))
		end
		if Library.flags["esptarget_zombie"] then
			local zombies = workspace:FindFirstChild("Zombies")
			if zombies then

				Library.flags['connections']['esptarget_zombie'] = zombies.ChildAdded:Connect(function(zombie)
					zombie.ChildAdded:Wait()
					wait(.2)
					if zombie and zombie:FindFirstChild("Zombie") and zombie:FindFirstChild("Head") and zombie.Zombie.Health > 0 then
						setupTracer(zombie)
					end
				end)

				for i,zombie in pairs(zombies:GetChildren()) do
					if zombie and zombie:FindFirstChild("Zombie") and zombie:FindFirstChild("Head") and zombie.Zombie.Health > 0 then
						setupTracer(zombie)
					end
				end
			end
		else
			destroyEsps("zombies")
			Library.flags['tracer_']['zombies'] = nil
			Library.flags['connections']['esptarget_zombie']:Disconnect()
		end
		return v
	end
})

esp:AddToggle({
	text = "box",
	flag = "esptype_box",
	state = false,
	callback = function(v)
		return v
	end
})
esp:AddToggle({
	text = "nametag",
	flag = "esptype_nametag",
	state = false,
	callback = function(v)
		return v
	end
})

esp:AddToggle({
	text = "tracer",
	flag = "esptype_tracer",
	state = false,
	callback = function(v)
		return v
	end
})

local mouse = plr:GetMouse()
function tpTo(pos)
	pcall(function()
		if plr.Character:FindFirstChildOfClass('Humanoid') and plr.Character:FindFirstChildOfClass('Humanoid').SeatPart then
			plr.Character:FindFirstChildOfClass('Humanoid').Sit = false
			wait(.1)
		end
		plr.Character.HumanoidRootPart.CFrame = pos
	end)
end

local d = false
local Boomer = Library:CreateWindow("Boomer")
Boomer:AddLabel({text = " Plz use dis when spectate."})
Boomer:AddLabel({text = " Notice: u need armour"})
Boomer:AddLabel({text = "  with 'last stand' upgrd"})
Boomer:AddLabel({text = "  inother to works."})
Boomer:AddLabel({text = " *Good ping is required."})

Boomer:AddToggle({
	text = "Free cam",
	flag = "freecam",
	state = false,
	callback = function(v)
		if Library.flags['freecam'] then
			CamUtil.A()
		else
			CamUtil.UA()
		end
	end
})
Boomer:AddToggle({
	text = "Click explode",
	flag = "click_explode",
	state = false,
	callback = function(v)
		if Library.flags['click_explode'] then
			Library.flags["connections"]['click_explode'] = Mouse.Button1Down:connect(function()
				if not d then
					d = true
					tpTo(mouse.Hit + Vector3.new(0,7,0))
					wait(.5)
					d = false
				end
			end)
		else
			Library.flags["connections"]['click_explode']:Disconnect()
		end
	end
})

local function getPlayerByFewChars(word)
	if #word <= 0 then
		return nil
	end
	for _,v in pairs(game.Players:GetPlayers()) do 
		if v and v.Name:lower():sub(1,#word) == word:lower() then
			return v
		end
	end
	return nil
end

Boomer:AddToggle({
	text = "Loop explosion",
	flag = "loop_explode",
	state = false,
	callback = function(v)
		if Library.flags['loop_explode'] then
			Library.flags["connections"]['loop_explode'] = RunService.RenderStepped:connect(function()
				if #Library.flags["target_explosion_player"] > 0 then
					local results = getPlayerByFewChars(Library.flags["target_explosion_player"])
					if results and game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid") then
						local distance = 0
						if results.Character ~= nil then
							tpTo(results.CFrame + Vector3.new(distance,1,0))
						end
					end
				end
			end)
		else
			Library.flags["connections"]['loop_explode']:Disconnect()
		end
	end
})
local targetlabel = Boomer:AddLabel({text = " Player:"})
Boomer:AddBox({
	text = "Plr name here", 
	flag = "target_explosion_player", 
	value = "dont need to be full chars", 
	callback = function(a) 
		local results = getPlayerByFewChars(a)
		if results then
			print(results.Name)
			targetlabel.Text = " Player: ".. results.Name
		else
			targetlabel.Text = " Player: "
		end
	end
})



Window:AddBind({
	text = "Toggle UI",
	flag = "bind",
	key = "RightShift",
	callback = function()
		Library:Close()
	end
})

local active = true

Window:AddButton({
	text = "Exit",
	flag = "button",
	callback = function()
		if Library.flags['freecam'] then
			CamUtil.UA()
		end
		destroyEsps()
		for i,v in pairs(Library.flags['connections']) do
			v:Disconnect()
		end
		active = false
		Library:Exit()
	end
})

Library:Init()

local function updateESP()
	for _, parent in pairs(Library.flags['tracer_']) do
		for i, target in pairs(parent) do
			local t = target.tracer()
			local t2 = target.box()
			local t3 = target.nametag()
			if not t or not t2 or not t3 then
				Library.flags['tracer_'][i] = nil
			end
		end
	end
end

while active and RunService.RenderStepped:Wait() do
	UpdateLighting()
	updateESP()
end
