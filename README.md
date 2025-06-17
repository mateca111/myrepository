-- ========================================
-- SCRIPTS PARA BLOX FRUITS
-- ========================================

-- SCRIPT 1: AUTO FARM BÁSICO
-- ========================================
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Configurações do Auto Farm
local autoFarmEnabled = false
local targetDistance = 50

-- Função para encontrar inimigos próximos
local function findNearestEnemy()
    local nearestEnemy = nil
    local shortestDistance = math.huge
    
    for _, obj in pairs(workspace.Enemies:GetChildren()) do
        if obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            if obj.Humanoid.Health > 0 then
                local distance = (rootPart.Position - obj.HumanoidRootPart.Position).Magnitude
                if distance < shortestDistance and distance <= targetDistance then
                    shortestDistance = distance
                    nearestEnemy = obj
                end
            end
        end
    end
    
    return nearestEnemy
end

-- Função de ataque
local function attackEnemy(enemy)
    if enemy and enemy:FindFirstChild("HumanoidRootPart") then
        -- Teleportar para o inimigo
        rootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5)
        
        -- Atacar
        local args = {
            [1] = "Sword",
            [2] = enemy.HumanoidRootPart.Position
        }
        ReplicatedStorage.Remotes.Combat:FireServer(unpack(args))
    end
end

-- Loop principal do auto farm
local function autoFarmLoop()
    while autoFarmEnabled do
        local enemy = findNearestEnemy()
        if enemy then
            attackEnemy(enemy)
        end
        wait(0.1)
    end
end

-- Toggle do auto farm
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F1 then
        autoFarmEnabled = not autoFarmEnabled
        print("Auto Farm:", autoFarmEnabled and "LIGADO" or "DESLIGADO")
        
        if autoFarmEnabled then
            spawn(autoFarmLoop)
        end
    end
end)

-- ========================================
-- SCRIPT 2: AUTO COLLECT FRUITS
-- ========================================

local autoCollectFruits = false

local function collectNearbyFruits()
    for _, fruit in pairs(workspace:GetChildren()) do
        if fruit.Name:find("Fruit") and fruit:FindFirstChild("Handle") then
            local distance = (rootPart.Position - fruit.Handle.Position).Magnitude
            if distance <= 50 then
                -- Teleportar para a fruta
                rootPart.CFrame = fruit.Handle.CFrame
                wait(0.5)
                
                -- Tentar coletar
                for i = 1, 10 do
                    rootPart.CFrame = fruit.Handle.CFrame
                    wait(0.1)
                end
            end
        end
    end
end

-- Loop de coleta de frutas
local function fruitCollectionLoop()
    while autoCollectFruits do
        collectNearbyFruits()
        wait(1)
    end
end

-- Toggle da coleta de frutas
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F2 then
        autoCollectFruits = not autoCollectFruits
        print("Auto Collect Fruits:", autoCollectFruits and "LIGADO" or "DESLIGADO")
        
        if autoCollectFruits then
            spawn(fruitCollectionLoop)
        end
    end
end)

-- ========================================
-- SCRIPT 3: TELEPORT PARA LOCAIS
-- ========================================

local teleportLocations = {
    ["Spawn"] = CFrame.new(-6, 16, 1),
    ["Jungle"] = CFrame.new(-1612, 37, 149),
    ["Desert"] = CFrame.new(944, 21, 4488),
    ["Frozen Village"] = CFrame.new(1347, 104, -1319),
    ["Marine Base"] = CFrame.new(-2573, 7, -3088),
    ["Sky Island"] = CFrame.new(-4607, 872, -1667),
    ["Prison"] = CFrame.new(4875, 20, 734),
    ["Colosseum"] = CFrame.new(-1427, 7, 1600)
}

local function teleportTo(locationName)
    if teleportLocations[locationName] then
        rootPart.CFrame = teleportLocations[locationName]
        print("Teleportado para:", locationName)
    else
        print("Local não encontrado:", locationName)
    end
end

-- Comandos de teleporte via chat
player.Chatted:Connect(function(message)
    local command = message:lower()
    
    if command:sub(1, 3) == "/tp" then
        local location = command:sub(5)
        
        for name, _ in pairs(teleportLocations) do
            if name:lower():find(location) then
                teleportTo(name)
                break
            end
        end
    end
end)

-- ========================================
-- SCRIPT 4: ESP (VISUALIZAÇÃO DE INIMIGOS)
-- ========================================

local espEnabled = false
local espBoxes = {}

local function createESP(obj)
    if obj:FindFirstChild("HumanoidRootPart") and not espBoxes[obj] then
        local box = Instance.new("BillboardGui")
        box.Name = "ESP"
        box.Adornee = obj.HumanoidRootPart
        box.Size = UDim2.new(4, 0, 6, 0)
        box.StudsOffset = Vector3.new(0, 0, 0)
        box.AlwaysOnTop = true
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundTransparency = 0.8
        frame.BackgroundColor3 = Color3.new(1, 0, 0)
        frame.BorderSizePixel = 2
        frame.BorderColor3 = Color3.new(1, 1, 1)
        frame.Parent = box
        
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 0.2, 0)
        nameLabel.Position = UDim2.new(0, 0, -0.3, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = obj.Name
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextScaled = true
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.Parent = box
        
        box.Parent = obj
        espBoxes[obj] = box
    end
end

local function removeESP(obj)
    if espBoxes[obj] then
        espBoxes[obj]:Destroy()
        espBoxes[obj] = nil
    end
end

local function updateESP()
    if espEnabled then
        for _, enemy in pairs(workspace.Enemies:GetChildren()) do
            if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                createESP(enemy)
            end
        end
    else
        for obj, box in pairs(espBoxes) do
            box:Destroy()
        end
        espBoxes = {}
    end
end

-- Toggle do ESP
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F3 then
        espEnabled = not espEnabled
        print("ESP:", espEnabled and "LIGADO" or "DESLIGADO")
        updateESP()
    end
end)

-- Atualizar ESP continuamente
spawn(function()
    while true do
        if espEnabled then
            updateESP()
        end
        wait(1)
    end
end)

-- ========================================
-- SCRIPT 5: AUTO STATS
-- ========================================

local autoStats = false
local statPriority = {"Melee", "Defense", "Sword", "Gun", "Fruit"}
local currentStatIndex = 1

local function upgradeStats()
    local args = {
        [1] = statPriority[currentStatIndex]
    }
    
    ReplicatedStorage.Remotes.StatAdd:FireServer(unpack(args))
    
    -- Rotacionar para a próxima stat
    currentStatIndex = currentStatIndex + 1
    if currentStatIndex > #statPriority then
        currentStatIndex = 1
    end
end

-- Loop de upgrade de stats
local function autoStatsLoop()
    while autoStats do
        upgradeStats()
        wait(0.1)
    end
end

-- Toggle do auto stats
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F4 then
        autoStats = not autoStats
        print("Auto Stats:", autoStats and "LIGADO" or "DESLIGADO")
        
        if autoStats then
            spawn(autoStatsLoop)
        end
    end
end)

-- ========================================
-- INSTRUÇÕES DE USO
-- ========================================

print("========================================")
print("SCRIPTS BLOX FRUITS CARREGADOS!")
print("========================================")
print("F1 - Toggle Auto Farm")
print("F2 - Toggle Auto Collect Fruits")
print("F3 - Toggle ESP (Visualização de Inimigos)")
print("F4 - Toggle Auto Stats")
print("Chat: /tp [local] - Teleportar")
print("Locais disponíveis: spawn, jungle, desert, frozen, marine, sky, prison, colosseum")
print("========================================")

-- ========================================
-- SCRIPT ADICIONAL: ANTI-AFK
-- ========================================

-- Previne o AFK movendo o personagem levemente
spawn(function()
    while true do
        wait(300) -- 5 minutos
        if rootPart then
            rootPart.CFrame = rootPart.CFrame * CFrame.new(0, 0, 0.1)
            wait(0.1)
            rootPart.CFrame = rootPart.CFrame * CFrame.new(0, 0, -0.1)
        end
    end
end)
