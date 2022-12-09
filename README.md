
local Client = game.Players.LocalPlayer
local ReplicatedStorage = game.ReplicatedStorage
local Mobs = workspace["__GAME"]["__Mobs"]

local Bridge = require(ReplicatedStorage.BridgeNet)
local ToolNet = Bridge.CreateBridge("TOOL_EVENT")
local AttackNet = Bridge.CreateBridge("ATTACK_EVENT")
local RemoteNet = Bridge.CreateBridge("REMOTE_EVENT")

local Settings = {
	AutoPunch = false,
	AutoAttack = false,
	AutoDefense = false,
	AutoChest = false,
	AutoQuest = false,
	SelectedEnemy = nil,
	SelectedMode = nil,
}

local EnemyTable = {} --Create a table called EnemyTable
for _, v in pairs(Mobs:GetChildren()) do --For every child in Mobs
	for _, enemy in pairs(v:GetChildren()) do --For every child in the child
		if enemy:FindFirstChild("NpcConfiguration") then --If the child has a child called NpcConfiguration
			if not table.find(EnemyTable, enemy.NpcConfiguration:GetAttribute("Name")) then --If the table does not already have an entry for the enemy
				table.insert(EnemyTable, enemy.NpcConfiguration:GetAttribute("Name")) --Add the enemy to the table
			end
		end
	end
end

local function RemoveTableDupes(tab)
	local hash = {}
	local res = {}
	for _, v in ipairs(tab) do
		if not hash[v] then
			res[#res + 1] = v
			hash[v] = true
		end
	end
	return res
end

local outputtedEnemies = RemoveTableDupes(EnemyTable)

--ToolNet:Fire("Combat", 1, false, Client.Character.Combat, "Melee")
--ToolNet:Fire("Defence", Client.Character.Defence, "Defence")
--AttackNet:Fire(getClosestMob(), Client.Character.Combat)

local function getClosestMob(name)
	local closest, maxDist = nil, 9e9
	for _, v in pairs(Mobs:GetChildren()) do
		for _, mob in pairs(v:GetChildren()) do
			if mob:FindFirstChild("NpcConfiguration") and mob.NpcConfiguration:GetAttribute("Health") > 0 then
				if mob.NpcConfiguration:GetAttribute("Name") == name then
					local dist = (mob.PrimaryPart.Position - Client.Character.PrimaryPart.Position).magnitude
					if dist < maxDist then
						maxDist = dist
						closest = mob
					end
				end
			end
		end
	end
	return closest
end

--[[
1. The function takes in a string parameter which is the name of the mob you want to get the closest one of
2. It then creates two variables, closest and maxDist. The closest variable is the closest mob to the player and maxDist is the maximum distance between the player and the closest mob
3. It then loops through all of the mobs in the game and checks if they are valid enemies (have a NpcConfiguration and their health is greater than 0)
4. If the mob is a valid enemy, it checks if the mob's name matches the name parameter
5. If the mob's name matches the name parameter, it checks if the mob is closer to the player than the previous closest mob. If it is, it sets the closest variable to the mob and the maxDist variable to the distance between the mob and the player
6. Once the loop is done, it returns the closest mob 
]]

local function getWeapon()
	local weapon = Client.Character:FindFirstChildOfClass("Tool")
	if weapon and weapon:GetAttribute("Type") ~= "Defence" then
		return weapon
	end
end

local function getFruit()
	local fruit = Client.Character:FindFirstChildOfClass("Tool")
	if fruit and fruit:GetAttribute("Type") == "Fruit" then
		return fruit
	end
end

-- check if new mobs are spawned and if they arent in the table, add them
Mobs.ChildAdded:Connect(function(child)
	for _, v in pairs(child:GetChildren()) do
		if v:FindFirstChild("NpcConfiguration") then
			if not table.find(EnemyTable, v.NpcConfiguration:GetAttribute("Name")) then
				table.insert(EnemyTable, v.NpcConfiguration:GetAttribute("Name"))
			end
		end
	end
end)

local Functions = {}
do
	function Functions.AutoPunch()
		while true do
			task.wait()
			if Settings.AutoPunch then
				pcall(function()
					local weapon = getWeapon()
					if weapon then
						ToolNet:Fire("Combat", 1, false, weapon, weapon:GetAttribute("Type"))
					end
				end)
			end
		end
	end

	function Functions.AutoFarm()
		while true do
			task.wait()
			if Settings.AutoFarm then
				pcall(function()
					local weapon = getWeapon()
					local enemy = getClosestMob(Settings.SelectedEnemy)
					if weapon and enemy then
						Client.Character:PivotTo(enemy.PrimaryPart.CFrame * CFrame.new(0, -10, 20))
						AttackNet:Fire(enemy, weapon)
					end
				end)
			end
		end
	end

    function Functions.AutoAttack()
		while true do
			task.wait()
			if Settings.AutoAttack then
				pcall(function()
					local weapon = getWeapon()
					local enemy = getClosestMob(Settings.SelectedEnemy)
					if weapon and enemy then
						AttackNet:Fire(enemy, weapon)
					end
				end)
			end
		end
	end

	function Functions.AutoDefense()
		while true do
			task.wait()
			if Settings.AutoDefense then
				local weapon = Client.Backpack:FindFirstChild("Defence") or Client.Character:FindFirstChild("Defence")
				if weapon then
					ToolNet:Fire("Defence", weapon, "Defence")
				end
			end
		end
	end

	function Functions.AutoChest()
		while true do
			task.wait()
			if Settings.AutoChest then
				pcall(function()
					for _, v in pairs(workspace:GetChildren()) do
						if v:FindFirstChild("ChestInteract") then
							Client.Character:PivotTo(v.PrimaryPart.CFrame * CFrame.new(0, -10, 0))
							fireproximityprompt(v.ChestInteract)
						end
					end
				end)
			end
		end
	end

	function Functions.AutoQuest()
		while true do
			task.wait()
			if Settings.AutoQuest then
				pcall(function()
					local container = Client.PlayerGui.Quests.CurrentQuestContainer
					container:GetPropertyChangedSignal("Position"):Wait()
					if container.Position ~= UDim2.new({ { 0.823, 0 }, { 0.362, 0 } }) then
						RemoteNet:Fire("GetQuest", Client.PlayerGui.Quests:GetAttribute("CurrentQuest"))
					end
				end)
			end
		end
	end

function Alltools()
    while true do
        task.wait()
        for i,v in pairs(speaker:FindFirstChildOfClass("Backpack"):GetChildren()) do
            if v:IsA("Tool") or v:IsA("HopperBin") then
                v.Parent = speaker.Character
            end
        end
    end
end
if not game:IsLoaded() then
	game.Loaded:Wait()
end

local Rayfield = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Rayfield/main/source"))()


local Window = Rayfield:CreateWindow({
	Name = "One Fruit Simulator",
	LoadingTitle = "One Fruit Simulator",
	LoadingSubtitle = "",
	ConfigurationSaving = {
		Enabled = true,
		FolderName = nil, -- Create a custom folder for your hub/game
		FileName = "script"

}	})
local Tab = Window:CreateTab("Main", 4483362458) -- Title, Image
Tab:CreateToggle({
	Name = "Auto Punch",
	CurrentValue = false,
	Callback = function(Value)
		Settings.AutoPunch = Value
	end,
})

Tab:CreateToggle({
	Name = "Auto Farm",
	CurrentValue = false,
	Callback = function(Value)
		Settings.AutoFarm = Value
	end,
})

Tab:CreateToggle({
	Name = "Auto Attack",
	CurrentValue = false,
	Callback = function(Value)
		Settings.AutoAttack = Value
	end,
})

Tab:CreateDropdown({
	Name = "Select Enemy",
	Options = outputtedEnemies,
	CurrentOption = "CLICK ME",
	Callback = function(Option)
		Settings.SelectedEnemy = Option
	end,
})

Tab:CreateToggle({
	Name = "Auto Defense",
	CurrentValue = false,
	Callback = function(Value)
		Settings.AutoDefense = Value
	end,
})

Tab:CreateToggle({
	Name = "Auto Haki",
	CurrentValue = false,
	Callback = function(t)
		Settings.AutoChest = t
	end,
})
local QuestSelected;

Tab:CreateDropdown({
	Name = "Select Quest",
	Options = {"1","2","3","4","5","6","7","8","9","10","11","12","13","14","15"},
	CurrentOption = "None",
	Callback = function(Option)
		QuestSelected = Option
	end,
})

Tab:CreateButton({
	Name = "Get Quest",
	Callback = function()
		local args = {[1] = {[1] = {[1] = "\7",[2] = "GetQuest",[3] = QuestSelected}}}game:GetService("ReplicatedStorage").RemoteEvent:FireServer(unpack(args))
	end,
})

Tab:CreateLabel("Get the Quest before enabling Auto Quest")

Tab:CreateToggle({
	Name = "Auto Quest",
	CurrentValue = false,
	Callback = function(Value)
		Settings.AutoQuest = Value
	end,
})
local Tab = Window:CreateTab("Teleports", 4483362458) -- Title, Image

Tab:CreateButton({
	Name = "Starter Island",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(3388.55713, 136.208038, 1728.40283, -1, 0, 0, 0, 1, 0, 0, 0, -1)
	end,
})
Tab:CreateButton({
	Name = "Jungle",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1986.62134, 131.401001, 598.679565, 0.578843892, 0, 0.81543839, 0, 1, 0, -0.81543839, 0, 0.578843892)
	end,
})
Tab:CreateButton({
	Name = "Buggy",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(3007.479, 141.433548, -587.198181, 1, 0, 0, 0, 1, 0, 0, 0, 1)
	end,
})
Tab:CreateButton({
	Name = "Marine",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(4941.4248, 137.19136, 56.7856445, 0, 0, -1, 0, 1, 0, 1, 0, 0)
	end,
})
Tab:CreateButton({
	Name = "GrandCity",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(1038.11609, 124.763535, -1008.62189, 1, 0, 0, 0, 1, 0, 0, 0, 1)
	end,
})
Tab:CreateButton({
	Name = "Gacha",
	Callback = function()
        Game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(3432.12598, 140.795761, 1771.28772, 0.374604106, 0, 0.92718488, 0, 1, 0, -0.92718488, 0, 0.374604106)
	end,
})
local Tab = Window:CreateTab("Misc", 4483362458) -- Title, Image

Tab:CreateButton({
	Name = "Equip All Tools",
	Callback = function()
		for _, tool in ipairs(game:GetService("Players").LocalPlayer.Backpack:GetChildren()) do
            if tool:IsA("Tool") then
                 tool.Parent = game:GetService("Players").LocalPlayer.Character
            end
        end
	end,
})

Tab:CreateInput({
	Name = "FPS CAP",
	PlaceholderText = "Input Placeholder",
	RemoveTextAfterFocusLost = false,
	Callback = function(Text)
	    setfpscap(Text)
		-- The function that takes place when the input is changed
    		-- The variable (Text) is a string for the value in the text box
	end,
})

Tab:CreateButton({
	Name = "Remove Effects for suna",
	Callback = function()
        game:GetService("ReplicatedStorage").Game["__Extra"].Vfx.Suna:Destroy()
	end,
})

Tab:CreateButton({
	Name = "Remove Effects for Mera",
	Callback = function()
        game:GetService("ReplicatedStorage").Game["__Assets"].SkillAssets["Mera Mera no Mi"]:Destroy()
	end,
})

Tab:CreateButton({
	Name = "Remove Effects for Pika",
	Callback = function()
        game:GetService("ReplicatedStorage").Game["__Extra"].Vfx.Light:Destroy()
	end,
})

for _, v in pairs(Functions) do
	task.spawn(v)
end
end
