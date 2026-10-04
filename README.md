loadstring([[
-- ⚒️ Ban Hammer Completo — Efeitos Visuais + Banimento
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- === CONFIGURAÇÕES ===
local CONFIG = {
    MensagemBan = "⚒️ Você foi banido pelo Ban Hammer!",
    CorMartelo = Color3.fromRGB(255, 0, 0),
    TamanhoMartelo = 3,
    DuracaoAnimacao = 0.8,
    BrilhoIntensidade = 10
}

-- === SISTEMA DE PERMISSÕES — COLOQUE SEU ID AQUI ===
local ADMINS = {
    [12345678] = true -- ← Troque pelo seu ID de usuário do Roblox
}

-- Cria RemoteEvent automaticamente
if not ReplicatedStorage:FindFirstChild("BanHammerEvent") then
    local Evento = Instance.new("RemoteEvent")
    Evento.Name = "BanHammerEvent"
    Evento.Parent = ReplicatedStorage
end
local BanHammerEvent = ReplicatedStorage:WaitForChild("BanHammerEvent")

-- === LADO DO SERVIDOR — EFEITOS E BANIMENTO ===
BanHammerEvent.OnServerEvent:Connect(function(QuemUsou, Alvo)
    -- Verifica permissão
    if not ADMINS[QuemUsou.UserId] then return end
    if not Alvo or not Alvo.Character or not Alvo.Character.Head then return end
    
    -- Cria o Martelo
    local Martelo = Instance.new("Part")
    Martelo.Name = "BanHammer"
    Martelo.Shape = Enum.PartType.Brick
    Martelo.Size = Vector3.new(CONFIG.TamanhoMartelo, CONFIG.TamanhoMartelo * 0.6, CONFIG.TamanhoMartelo * 0.8)
    Martelo.Position = Alvo.Character.Head.Position + Vector3.new(0, 4, 0)
    Martelo.Anchored = true
    Martelo.CanCollide = false
    Martelo.Material = Enum.Material.Neon
    Martelo.BrickColor = BrickColor.new("Bright red")
    Martelo.Parent = workspace

    -- Brilho/Luz
    local Luz = Instance.new("PointLight")
    Luz.Color = CONFIG.CorMartelo
    Luz.Brightness = CONFIG.BrilhoIntensidade
    Luz.Range = 20
    Luz.Parent = Martelo

    -- Animação de queda
    local Animacao = TweenService:Create(
        Martelo,
        TweenInfo.new(CONFIG.DuracaoAnimacao, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {Position = Alvo.Character.Head.Position + Vector3.new(0, 0.5, 0)}
    )
    Animacao:Play()

    -- Quando acertar
    Animacao.Completed:Connect(function()
        -- Explosão
        local Explosao = Instance.new("Explosion")
        Explosao.Position = Alvo.Character.Head.Position
        Explosao.BlastRadius = 6
        Explosao.Parent = Martelo

        -- Banimento
        Alvo:Kick(CONFIG.MensagemBan)

        -- Limpa o martelo
        task.wait(0.5)
        Martelo:Destroy()
    end)
end)

-- === LADO DO CLIENTE — BOTÃO DE ACIONAMENTO ===
local Player = Players.LocalPlayer
local Mouse = Player:GetMouse()
local Evento = ReplicatedStorage:WaitForChild("BanHammerEvent")

-- Cria a interface do botão automaticamente
Player.CharacterAdded:Connect(function() end)
local Gui = Instance.new("ScreenGui")
Gui.Name = "BanHammerUI"
Gui.Parent = Player:WaitForChild("PlayerGui")

local Botao = Instance.new("TextButton")
Botao.Name = "BotaoBanHammer"
Botao.Size = UDim2.new(0, 180, 0, 60)
Botao.Position = UDim2.new(0.02, 0, 0.5, 0)
Botao.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
Botao.Text = "⚒️ BAN HAMMER"
Botao.TextColor3 = Color3.fromRGB(255, 255, 255)
Botao.Font = Enum.Font.GothamBold
Botao.TextSize = 18
Botao.Parent = Gui

-- Ação ao clicar
Botao.MouseButton1Click:Connect(function()
    if Mouse.Target then
        local Modelo = Mouse.Target:FindFirstAncestorWhichIsA("Model")
        if Modelo then
            local Alvo = Players:GetPlayerFromCharacter(Modelo)
            if Alvo and Alvo ~= Player then
                Evento:FireServer(Alvo)
            end
        end
    end
end)
]])()
