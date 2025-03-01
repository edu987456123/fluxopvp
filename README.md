local DrRayLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/AZYsGithub/DrRay-UI-Library/main/DrRay.lua"))()
local window = DrRayLibrary:Load("Fluxo PVP", "Default")

-- Aba do Aimbot
local aimbotTab = DrRayLibrary.newTab("Aimbot", "ImageIdHere")

-- Aba do ESP
local espTab = DrRayLibrary.newTab("ESP", "ImageIdHere")

-- Aba de Créditos
local creditsTab = DrRayLibrary.newTab("Créditos", "ImageIdHere")

-- Aba Misc
local miscTab = DrRayLibrary.newTab("Misc", "ImageIdHere")

-- Adicionar botões de créditos
for i = 1, 5 do
    creditsTab.newButton("Feito por NINJA", "Sem funcionalidade", function() end)
end

-- Variáveis de controle
local aimbotEnabled = false
local teamCheck = false
local wallCheck = false
local espEnabled = {Red = false, Blue = false, Black = false}
local hitboxEnabled = false

-- Função para verificar times
local function isTeammate(player)
    return player.Team == game.Players.LocalPlayer.Team
end

-- Função Aimbot
local function aimbot()
    local playerService = game:GetService("Players")
    local localPlayer = playerService.LocalPlayer
    local camera = workspace.CurrentCamera

    game:GetService("RunService").RenderStepped:Connect(function()
        if not aimbotEnabled then return end

        local closestPlayer = nil
        local shortestDistance = math.huge

        for _, player in pairs(playerService:GetPlayers()) do
            if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
                local humanoid = player.Character.Humanoid
                local rootPart = player.Character.HumanoidRootPart
                local rootPosition = camera:WorldToScreenPoint(rootPart.Position)

                -- Ignorar jogadores mortos
                if humanoid.Health <= 0 then
                    continue
                end

                -- Team Check
                if teamCheck and isTeammate(player) then
                    continue
                end

                -- Wall Check
                if wallCheck then
                    local direction = (rootPart.Position - camera.CFrame.Position).Unit
                    local ray = Ray.new(camera.CFrame.Position, direction * 500)
                    local hit, _ = workspace:FindPartOnRay(ray, localPlayer.Character, false, true)
                    if hit and not hit:IsDescendantOf(player.Character) then
                        continue
                    end
                end

                -- Calcular distância da mira
                local screenDistance = (Vector2.new(rootPosition.X, rootPosition.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude

                if screenDistance < shortestDistance then
                    shortestDistance = screenDistance
                    closestPlayer = rootPart
                end
            end
        end

        -- Mirar no jogador mais próximo
        if closestPlayer then
            camera.CFrame = CFrame.new(camera.CFrame.Position, closestPlayer.Position)
        end
    end)
end

-- Função para criar ESP usando Highlight
local function createESP(color)
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local highlight = Instance.new("Highlight", player.Character)
            highlight.FillColor = color
            highlight.OutlineTransparency = 1
            highlight.Name = "ESP"
        end
    end
end

-- Função para remover ESP
local function removeESP()
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player.Character then
            for _, child in ipairs(player.Character:GetChildren()) do
                if child:IsA("Highlight") and child.Name == "ESP" then
                    child:Destroy()
                end
            end
        end
    end
end

-- Atualizar ESP de acordo com a cor ativada
local function updateESP()
    removeESP()
    if espEnabled.Red then
        createESP(Color3.fromRGB(255, 0, 0)) -- Vermelho
    end
    if espEnabled.Blue then
        createESP(Color3.fromRGB(0, 0, 255)) -- Azul
    end
    if espEnabled.Black then
        createESP(Color3.fromRGB(0, 0, 0)) -- Preto
    end
end

-- Atualizar ESP constantemente
game:GetService("RunService").RenderStepped:Connect(updateESP)

-- Função para ajustar hitboxes automaticamente
local function adjustHitboxes()
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            if not isTeammate(player) then
                if hitboxEnabled then
                    head.Size = Vector3.new(5, 5, 5) -- Aumenta o tamanho da cabeça
                    head.CanCollide = false -- Evita problemas de colisão
                else
                    head.Size = Vector3.new(2, 1, 1) -- Tamanho padrão
                    head.CanCollide = true
                end
            end
        end
    end
end

-- Monitorar jogadores que entram no jogo ou resetam
game:GetService("Players").PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(0.5) -- Aguardar personagem carregar
        adjustHitboxes()
    end)
end)

-- Atualizar hitboxes em personagens que resetam
for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
    player.CharacterAdded:Connect(function()
        wait(0.5) -- Aguardar personagem carregar
        adjustHitboxes()
    end)
end

-- Atualizar hitboxes em tempo real
game:GetService("RunService").RenderStepped:Connect(adjustHitboxes)

-- Configurações de Aimbot
aimbotTab.newToggle("Aimbot", "Aimbot para mobile e PC.", false, function(state)
    aimbotEnabled = state
end)

aimbotTab.newToggle("Team Check", "Não mira em quem ta no seu time.", false, function(state)
    teamCheck = state
end)

aimbotTab.newToggle("Wall Check", "Não mira nos players atras da parede. ", false, function(state)
    wallCheck = state
end)

-- Configurações de ESP
espTab.newToggle("ESP Vermelho", "Ativa ESP Vermelho", false, function(state)
    espEnabled.Red = state
end)

espTab.newToggle("ESP Azul", "Ativa ESP Azul", false, function(state)
    espEnabled.Blue = state
end)

espTab.newToggle("ESP Preto", "Ativa ESP Preto", false, function(state)
    espEnabled.Black = state
end)

-- Configuração de Hitbox Cabeça
miscTab.newToggle("Hitbox Cabeça", "Aumenta a cabeça dos players.", false, function(state)
    hitboxEnabled = state
    adjustHitboxes()
end)

-- Inicializar Aimbot
aimbot()
