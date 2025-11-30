-- Script (no ServerScriptService)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Certifique-se de que este RemoteEvent exista no ReplicatedStorage
local FishingEvent = ReplicatedStorage:WaitForChild("InteractionEvent")

-- Configurações de validação
local COOLDOWN_TIME = 5 -- Tempo em segundos antes de roubar novamente
local MAX_DISTANCE = 10 -- Distância máxima permitida para roubar

-- Lógica para rastrear o cooldown por jogador
local lastRobTime = {}

-- Função de validação e execução do roubo
local function ProcessRobbery(player, targetPart)
    local character = player.Character
    if not character then return end

    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    -- 1. Validação de Cooldown
    local currentTime = tick()
    if lastRobTime[player.UserId] and (currentTime - lastRobTime[player.UserId] < COOLDOWN_TIME) then
        -- O jogador está em cooldown - não faça nada
        return
    end

    -- 2. Validação de Distância (Crítico para prevenir "Auto Roubar")
    local distance = (humanoidRootPart.Position - targetPart.Position).Magnitude
    if distance > MAX_DISTANCE then
        -- O jogador está muito longe - não faça nada
        warn(player.Name .. " tentou roubar de muito longe! Distância: " .. distance)
        return
    end

    -- 3. Execução Legitima
    lastRobTime[player.UserId] = currentTime
    
    -- Lógica de roubo/coleta:
    print(player.Name .. " roubou/coletou um item com sucesso!")
    -- Exemplo: Dá dinheiro ao jogador
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats and leaderstats:FindFirstChild("Cash") then
        leaderstats.Cash.Value = leaderstats.Cash.Value + 50
    end
    
    -- Lógica para o objeto alvo (ex: Desaparecer, Cooldown visual, etc.)
    targetPart.Transparency = 1
    targetPart.CanCollide = false
    
    -- Espera um pouco e faz o objeto reaparecer
    wait(COOLDOWN_TIME)
    targetPart.Transparency = 0
    targetPart.CanCollide = true
end

-- Conexão com o RemoteEvent
FishingEvent.OnServerEvent:Connect(function(player, action, partToRob)
    if action == "RequestRob" then
        if typeof(partToRob) == "Instance" and partToRob:IsA("BasePart") then
            ProcessRobbery(player, partToRob)
        end
    end
end)

print("Sistema de Interação Segura iniciado.")
