-- Carrega a biblioteca Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Cria a janela
local Window = Fluent:CreateWindow({
    Title = "🔥 Arthur-Menu City(AZARALHO) 🔥",
    SubTitle = "by Bernardo e Arthur",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Amethyst",
    MinimizeKey = Enum.KeyCode.K
})

-- Função para notificação
local function notificar(titulo, mensagem, duracao)
    Fluent:Notify({
        Title = titulo,
        Content = mensagem,
        Duration = duracao or 3
    })
end





-- Aba Infos
local InfosTab = Window:AddTab({ Title = "Infos", Icon = "info" })
notificar("Arthur-Menu Injected 💸", "Obrigado por Adquirir nosso menu!", 36)
InfosTab:AddParagraph({
    Title = "Arthur-Menu ℹ️",
    Content = "+Menu Reconstruído "
})
InfosTab:AddButton({
    Title = "Ver Atualizações 📜",
    Callback = function()
        print("Abrindo atualizações...")
    end
})

-- Botão para copiar invite do Discord
InfosTab:AddButton({
    Title = "Copiar Invite do Discord",
    Callback = function()
        setclipboard("https://discord.gg/jKYYKd6ncE") -- Substitua por um invite válido
        notificar("Discord", "Invite copiado para a área de transferência!")
    end
})

-- Aba Combate
local CombatTab = Window:AddTab({ Title = "Combate 🗡️", Icon = "sword" })

-- Noclip
local noclipEnabled = false
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()

local function toggleNoclip(state)
    noclipEnabled = state
    notificar(noclipEnabled and "Noclip Ativado" or "Noclip Desativado", noclipEnabled and "Agora você pode atravessar objetos!" or "Agora você colide normalmente.", 2)
end

RunService.Stepped:Connect(function()
    if noclipEnabled then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

CombatTab:AddToggle("NoclipToggle", {
    Title = "NoClip 💥",
    Default = false,
    Callback = function(state)
        toggleNoclip(state)
    end
})

-- Speed
local speedValue = 16
local speedEnabled = false
local function getHumanoid()
    if localPlayer.Character then
        return localPlayer.Character:FindFirstChildOfClass("Humanoid")
    end
    return nil
end

local function setSpeed(value)
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.WalkSpeed = value
    end
end

task.spawn(function()
    while task.wait(0.05) do
        if speedEnabled then
            local humanoid = getHumanoid()
            if humanoid and humanoid.WalkSpeed ~= speedValue then
                humanoid.WalkSpeed = speedValue
            end
        end
    end
end)

localPlayer.CharacterAdded:Connect(function(character)
    repeat wait() until character:FindFirstChildOfClass("Humanoid")
    if speedEnabled then
        setSpeed(speedValue)
    end
end)

CombatTab:AddToggle("SpeedToggle", {
    Title = "Ativar ajuste Velocidade⚡",
    Default = false,
    Callback = function(value)
        speedEnabled = value
        if speedEnabled then
            setSpeed(speedValue)
        else
            setSpeed(16)
        end
    end
})

CombatTab:AddSlider("SpeedSlider", {
    Title = "Ajustar Velocidade🌪️",
    Min = 16,
    Max = 100,
    Default = 16,
    Rounding = 1,
    Suffix = "Studs/s",
    Callback = function(value)
        speedValue = value
    end
})

-- Anti-Fall Damage
local antiFallEnabled = false

local function antiFallDamage()
    while antiFallEnabled do
        task.wait(0.1)
        if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart then
                if rootPart.Velocity.Y < -50 then
                    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z)
                end
            end
        end
    end
end

CombatTab:AddToggle("AntiFallToggle", {
    Title = "Anti-Queda🪂",
    Default = false,
    Callback = function(value)
        antiFallEnabled = value
        if antiFallEnabled then
            task.spawn(antiFallDamage)
        end
    end
})

-- Hitbox
local hitboxEnabled = false
local defaultHitboxSize = Vector3.new(5, 5, 5)
local hitboxSize = defaultHitboxSize
local originalSizes = {}
local minSize = 1
local maxSize = 20
local sizeStep = 1

local function expandHitbox()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            if humanoidRootPart and head then
                if not originalSizes[player] then
                    originalSizes[player] = {
                        HumanoidRootPart = humanoidRootPart.Size,
                        Head = head.Size
                    }
                end
                if hitboxEnabled then
                    humanoidRootPart.Size = hitboxSize
                    head.Size = hitboxSize
                    humanoidRootPart.Transparency = 0.7
                    head.Transparency = 0.7
                    humanoidRootPart.CanCollide = false
                    head.CanCollide = false
                else
                    humanoidRootPart.Size = originalSizes[player].HumanoidRootPart
                    head.Size = originalSizes[player].Head
                    humanoidRootPart.Transparency = 0
                    head.Transparency = 0
                    humanoidRootPart.CanCollide = true
                    head.CanCollide = true
                end
            end
        end
    end
end

game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if hitboxEnabled then
            task.wait(0.1)
            expandHitbox()
        end
    end)
end)

game.Players.PlayerRemoving:Connect(function(player)
    originalSizes[player] = nil
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if hitboxEnabled then
        expandHitbox()
    end
end)

CombatTab:AddToggle("HitboxToggle", {
    Title = "Ativar Hitbox 📦",
    Default = false,
    Callback = function(value)
        hitboxEnabled = value
        if not hitboxEnabled then
            for player, sizes in pairs(originalSizes) do
                if player.Character then
                    local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
                    local head = player.Character:FindFirstChild("Head")
                    if humanoidRootPart and head then
                        humanoidRootPart.Size = sizes.HumanoidRootPart
                        head.Size = sizes.Head
                        humanoidRootPart.Transparency = 0
                        head.Transparency = 0
                        humanoidRootPart.CanCollide = true
                        head.CanCollide = true
                    end
                end
            end
        end
    end
})

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HitboxGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.Enabled = false

local frame = Instance.new("Frame")
frame.Name = "HitboxFrame"
frame.Size = UDim2.new(0, 220, 0, 50)
frame.Position = UDim2.new(0.5, -110, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

local sizeLabel = Instance.new("TextLabel")
sizeLabel.Name = "SizeLabel"
sizeLabel.Size = UDim2.new(0.9, 0, 0.8, 0)
sizeLabel.Position = UDim2.new(0.05, 0, 0.1, 0)
sizeLabel.BackgroundTransparency = 1
sizeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
sizeLabel.Font = Enum.Font.Gotham
sizeLabel.TextSize = 16
sizeLabel.Text = "Tamanho: " .. hitboxSize.X
sizeLabel.Parent = frame

CombatTab:AddSlider("HitboxSizeSlider", {
    Title = "Tamanho da Hitbox 🎯",
    Min = minSize,
    Max = maxSize,
    Default = hitboxSize.X,
    Rounding = sizeStep,
    Suffix = " studs",
    Callback = function(value)
        hitboxSize = Vector3.new(value, value, value)
        sizeLabel.Text = "Tamanho: " .. hitboxSize.X
        if hitboxEnabled then
            expandHitbox()
        end
    end
})








-- Criar nova aba para Aimbot
local AimbotTab = Window:AddTab({ Title = "Aimbot 🎯", Icon = "target" })

-- 🔥 Variáveis principais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

-- 🧠 Configuração
local Settings = {
    Aimbot = true,
    FOV = 100
}

-- 🟢 Toggle no Fluent UI
local AimbotToggle = AimbotTab:AddToggle("AimbotToggle", {
    Title = "Ativar Aimbot mobile",
    Description = "Ativa ou desativa o Aimbot com FOV",
    Default = true
})

-- 🔵 Desenho do FOV Circle
local FOV = Drawing.new("Circle")
FOV.Color = Color3.fromRGB(0, 255, 0)
FOV.Thickness = 1
FOV.Transparency = 0.5
FOV.Filled = false
FOV.NumSides = 100
FOV.Radius = Settings.FOV
FOV.Visible = true

-- 🎯 Função para pegar o player mais próximo do centro da tela (dentro do FOV)
function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = Settings.FOV

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onScreen then
                local distance = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end

    return closestPlayer
end

-- 🔗 Conexão do botão com o Aimbot
AimbotToggle:OnChanged(function(state)
    Settings.Aimbot = state
    FOV.Visible = state
end)

-- 🔁 Loop do Aimbot
RunService.RenderStepped:Connect(function()
    -- Atualiza a posição do círculo FOV no centro da tela
    FOV.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    FOV.Radius = Settings.FOV

    -- Executa o aimbot se estiver ativado
    if Settings.Aimbot then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.HumanoidRootPart.Position)
        end
    end
end)

-- Variáveis de controle
local aimbotEnabled = false
local isMouseButton2Down = false
local Typing = false
local isDragging = false
local dragStart, startPos

-- Serviços
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Ambiente isolado
local Environment = {
    Settings = {
        Enabled = true,
        TeamCheck = false,
        AliveCheck = true,
        WallCheck = false,
        Sensitivity = 0.07, -- Suavização mais fluida, mas ainda forte
        LockPart = "Head" -- Mira na cabeça
    },
    FOVSettings = {
        Enabled = true,
        Visible = true,
        Amount = 90,
        Color = Color3.fromRGB(0, 255, 255), -- Ciano claro para suavidade
        LockedColor = Color3.fromRGB(255, 0, 0), -- Vermelho ao travar
        Transparency = 0.8, -- Levemente translúcido
        Sides = 60,
        Thickness = 1, -- Borda mais fina
        Filled = false
    },
    FOVCircle = nil, -- Inicializado em InitializeFOVCircle
    Locked = nil,
    ServiceConnections = {},
    Animation = nil
}

-- Inicializar o círculo de FOV
local function InitializeFOVCircle()
    if Environment.FOVCircle and Environment.FOVCircle.Remove then
        Environment.FOVCircle:Remove()
    end
    if Drawing then
        Environment.FOVCircle = Drawing.new("Circle")
        Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
        Environment.FOVCircle.Color = Environment.FOVSettings.Color
        Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
        Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
        Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
        Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
        print("Aimbot: Círculo de FOV inicializado")
    else
        print("Aimbot: ERRO: Biblioteca 'Drawing' não suportada pelo executor")
    end
end

-- Funções do aimbot
local function CancelLock()
    Environment.Locked = nil
    if Environment.Animation then Environment.Animation:Cancel() end
    if Environment.FOVCircle then
        Environment.FOVCircle.Color = Environment.FOVSettings.Color
    end
end

local function GetClosestPlayer()
    if not Environment.Locked then
        local RequiredDistance = Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000
        local closestPlayer = nil
        for _, v in ipairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                local humanoid = v.Character:FindFirstChildOfClass("Humanoid")
                local rootPart = v.Character:FindFirstChild("HumanoidRootPart")
                if not (humanoid and rootPart) then
                elseif Environment.Settings.AliveCheck and humanoid.Health <= 0 then
                elseif Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then
                elseif Environment.Settings.WallCheck then
                    local parts = Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, {LocalPlayer.Character, v.Character})
                    if #parts > 0 then
                    else
                        local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                        if Distance < RequiredDistance and OnScreen then
                            RequiredDistance = Distance
                            closestPlayer = v
                        end
                    end
                else
                    local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        closestPlayer = v
                    end
                end
            end
        end
        Environment.Locked = closestPlayer
        if closestPlayer then
            print("Aimbot: Jogador travado: " .. closestPlayer.Name)
        end
    elseif Environment.Locked and Environment.Locked.Character and Environment.Locked.Character:FindFirstChild(Environment.Settings.LockPart) then
        local Vector, OnScreen = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude
        if Distance > (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000) or not OnScreen then
            CancelLock()
        end
    else
        CancelLock()
    end
end

-- Conexões de eventos
local function SetupConnections()
    Environment.ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
        Typing = true
        print("Aimbot: Digitando: Aimbot pausado")
    end)

    Environment.ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
        Typing = false
        print("Aimbot: Digitando finalizado: Aimbot retomado")
    end)

    Environment.ServiceConnections.MouseButton2Down = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            isMouseButton2Down = true
        end
    end)

    Environment.ServiceConnections.MouseButton2Up = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            isMouseButton2Down = false
            CancelLock()
        end
    end)
end

-- Loop principal
local function LoadAimbot()
    InitializeFOVCircle() -- Reinicializa o círculo ao ativar
    Environment.ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled and Environment.Settings.Enabled and not Typing then
            if Environment.FOVCircle and Environment.FOVSettings.Enabled then
                Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
                Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
                Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
                Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
                Environment.FOVCircle.Color = Environment.FOVSettings.Color
                Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
                Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
                Environment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
            elseif Environment.FOVCircle then
                Environment.FOVCircle.Visible = false
            end

            if isMouseButton2Down then
                GetClosestPlayer()
                if Environment.Locked and Environment.Locked.Character and Environment.Locked.Character:FindFirstChild(Environment.Settings.LockPart) then
                    local targetPos = Environment.Locked.Character[Environment.Settings.LockPart].Position
                    if Environment.Settings.Sensitivity > 0 then
                        Environment.Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, targetPos)})
                        Environment.Animation:Play()
                    else
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
                    end
                    if Environment.FOVCircle then
                        Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor
                    end
                end
            end
        else
            if Environment.FOVCircle then
                Environment.FOVCircle.Visible = false
            end
            CancelLock()
        end
    end)
end

-- Função para limpar conexões
local function CleanupAimbot()
    for _, connection in pairs(Environment.ServiceConnections) do
        connection:Disconnect()
    end
    Environment.ServiceConnections = {}
    if Environment.FOVCircle and Environment.FOVCircle.Remove then
        Environment.FOVCircle:Remove()
        Environment.FOVCircle = nil
    end
    CancelLock()
    print("Aimbot: Aimbot encerrado")
end

-- Criar ScreenGui (inicialmente invisível)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotGui"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.Enabled = false -- Invisível até clicar no botão

-- Criar Frame
local frame = Instance.new("Frame")
frame.Name = "AimbotFrame"
frame.Size = UDim2.new(0, 220, 0, 150)
frame.Position = UDim2.new(0.5, -110, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Sistema de arraste personalizado
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Criar TextButton para ativar/desativar aimbot
local button = Instance.new("TextButton")
button.Name = "AimbotButton"
button.Size = UDim2.new(0.9, 0, 0.2, 0)
button.Position = UDim2.new(0.05, 0, 0.1, 0)
button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 18
button.Text = "Ativar Aimbot 🎯"
button.Parent = frame
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = button

-- Criar TextLabel para mostrar o FOV
local fovLabel = Instance.new("TextLabel")
fovLabel.Name = "FovLabel"
fovLabel.Size = UDim2.new(0.9, 0, 0.2, 0)
fovLabel.Position = UDim2.new(0.05, 0, 0.35, 0)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextSize = 16
fovLabel.Text = "FOV: 90"
fovLabel.Parent = frame

-- Criar botões + e - para ajustar FOV
local plusButton = Instance.new("TextButton")
plusButton.Name = "PlusButton"
plusButton.Size = UDim2.new(0.2, 0, 0.2, 0)
plusButton.Position = UDim2.new(0.75, 0, 0.35, 0)
plusButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
plusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
plusButton.Font = Enum.Font.GothamBold
plusButton.TextSize = 18
plusButton.Text = "+"
plusButton.Parent = frame
local plusCorner = Instance.new("UICorner")
plusCorner.CornerRadius = UDim.new(0, 8)
plusCorner.Parent = plusButton

local minusButton = Instance.new("TextButton")
minusButton.Name = "MinusButton"
minusButton.Size = UDim2.new(0.2, 0, 0.2, 0)
minusButton.Position = UDim2.new(0.05, 0, 0.35, 0)
minusButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minusButton.Font = Enum.Font.GothamBold
minusButton.TextSize = 18
minusButton.Text = "-"
minusButton.Parent = frame
local minusCorner = Instance.new("UICorner")
minusCorner.CornerRadius = UDim.new(0, 8)
minusCorner.Parent = minusButton

-- Criar toggle para TeamCheck
local teamCheckToggle = Instance.new("TextButton")
teamCheckToggle.Name = "TeamCheckToggle"
teamCheckToggle.Size = UDim2.new(0.9, 0, 0.2, 0)
teamCheckToggle.Position = UDim2.new(0.05, 0, 0.6, 0)
teamCheckToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
teamCheckToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
teamCheckToggle.Font = Enum.Font.GothamBold
teamCheckToggle.TextSize = 18
teamCheckToggle.Text = "TeamCheck: OFF"
teamCheckToggle.Parent = frame
local teamCheckCorner = Instance.new("UICorner")
teamCheckCorner.CornerRadius = UDim.new(0, 8)
teamCheckCorner.Parent = teamCheckToggle

-- Criar botão de fechar
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0.1, 0, 0.1, 0)
closeButton.Position = UDim2.new(0.85, 0, 0.05, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 14
closeButton.Text = "X"
closeButton.Parent = frame
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = closeButton

-- Criar botão para minimizar/maximizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Name = "MinimizeButton"
minimizeButton.Size = UDim2.new(0.1, 0, 0.1, 0)
minimizeButton.Position = UDim2.new(0.95, 0, 0.05, 0)
minimizeButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 14
minimizeButton.Text = "-"
minimizeButton.Parent = frame
local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 4)
minimizeCorner.Parent = minimizeButton

-- Variáveis de controle para UI
local minimized = false
local originalSize = frame.Size
local minimizedSize = UDim2.new(0, 50, 0, 50)

-- Conectar botões da UI
button.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
    if aimbotEnabled then
        SetupConnections()
        LoadAimbot()
    else
        CleanupAimbot()
    end
    print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
end)

plusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount + 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
    if Environment.FOVCircle then
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
    end
    print("Aimbot: FOV ajustado para " .. Environment.FOVSettings.Amount)
end)

minusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount - 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
    if Environment.FOVCircle then
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
    end
    print("Aimbot: FOV ajustado para " .. Environment.FOVSettings.Amount)
end)

teamCheckToggle.MouseButton1Click:Connect(function()
    Environment.Settings.TeamCheck = not Environment.Settings.TeamCheck
    teamCheckToggle.Text = "TeamCheck: " .. (Environment.Settings.TeamCheck and "ON" or "OFF")
    print("Aimbot: TeamCheck: " .. (Environment.Settings.TeamCheck and "ON" or "OFF"))
end)

closeButton.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
    print("Aimbot: UI fechada")
end)

minimizeButton.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        frame.Size = minimizedSize
        minimizeButton.Text = "+"
        button.Visible = false
        fovLabel.Visible = false
        plusButton.Visible = false
        minusButton.Visible = false
        teamCheckToggle.Visible = false
        closeButton.Visible = false
    else
        frame.Size = originalSize
        minimizeButton.Text = "-"
        button.Visible = true
        fovLabel.Visible = true
        plusButton.Visible = true
        minusButton.Visible = true
        teamCheckToggle.Visible = true
        closeButton.Visible = true
    end
    print("Aimbot: UI " .. (minimized and "minimizada" or "maximizada"))
end)

-- Adicionar controles na aba Aimbot
AimbotTab:AddToggle("AimbotToggle", {
    Title = "Ativar Aimbot 🔥",
    Default = false,
    Callback = function(value)
        aimbotEnabled = value
        button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
        if aimbotEnabled then
            SetupConnections()
            LoadAimbot()
        else
            CleanupAimbot()
        end
        print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
    end
})

AimbotTab:AddKeybind("AimbotKeybind", {
    Title = "Ativar/Desativar Aimbot 🥷",
    Default = "E",
    Callback = function()
        aimbotEnabled = not aimbotEnabled
        button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
        if aimbotEnabled then
            SetupConnections()
            LoadAimbot()
        else
            CleanupAimbot()
        end
        print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
    end
})

AimbotTab:AddButton({
    Title = "Abrir/Fechar UI Aimbot 🎯",
    Callback = function()
        screenGui.Enabled = not screenGui.Enabled
        print("Aimbot: " .. (screenGui.Enabled and "UI Aimbot aberta!" or "UI Aimbot fechada!"))
    end
})

-- Inicializar o aimbot
local success, errorMsg = pcall(function()
    InitializeFOVCircle()
    SetupConnections()
    if aimbotEnabled then LoadAimbot() end
end)
if not success then
    print("Aimbot: Erro ao inicializar aimbot: " .. tostring(errorMsg))
else
    print("Aimbot: Aimbot inicializado com sucesso!")
end

-- Criar abas
local Tabs = {
    Main = Window:AddTab({ Title = "Revistar", Icon = "search" }),
}

-- Seção NECESSÁRIO
Tabs.Main:AddParagraph({
    Title = "Arthur ,menu e polengo revist",
    Content = "Feito por Polengo e arthur 👨‍💻"
})

Tabs.Main:AddSection("NECESSÁRIO")

-- Botão Puxar Itens
Tabs.Main:AddButton({
    Title = "Puxar Itens 🎒",
    Description = "Puxa todos os itens automaticamente",
    Callback = function()
        -- Função para deletar todas as NotifyGui
        local function deletarNotifyGui()
            local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
            for _, gui in ipairs(playerGui:GetChildren()) do
                if gui.Name == "NotifyGui" and gui:IsA("ScreenGui") then
                    gui:Destroy() -- Deleta a NotifyGui
                end
            end
        end

        -- Lista de itens para pegar
        local itens = {"AK47", "Uzi", " Planta Limpa", "IA2", "Parafal", "Faca", "AR-15", "Glock 17", "IA2", "G3", "IPhone 14", "Agua", "Hamburguer", "Hi Power", "Natalina"}

        -- Argumentos para a requisição
        local args = {
            [1] = "mudaInv",
            [2] = "2",
            [4] = "1"
        }

        -- Loop principal
        while true do
            -- Deletar todas as NotifyGui antes de pegar os itens
            deletarNotifyGui()

            -- Pegar itens
            for i, item in ipairs(itens) do
                if i <= 16 then  -- Só tenta alocar até 16 slots
                    args[3] = item  -- Atualiza o item para o valor da vez
                    args[2] = tostring(i)  -- Atribui o slot dinamicamente (1, 2, 3, ..., 16)
                    task.spawn(function()
                        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("InvRemotes"):WaitForChild("InvRequest"):InvokeServer(unpack(args))
                    end)
                end
            end

            wait(0)  -- Espera um frame para evitar lag
        end
    end
})

-- Seção PC
Tabs.Main:AddSection("PC")

-- Toggle para mandar revistar (TECLA T)
local revistarToggle = Tabs.Main:AddToggle("RevistToggle", {
    Title = "mandar revistar (TECLA T)",
    Description = "Ativa/desativa o revistar com a tecla T",
    Default = false
})

revistarToggle:OnChanged(function(Value)
    getgenv().Enabled = Value
    Fluent:Notify({
        Title = "Toggle Revistar",
        Content = Value and "Revistar com tecla T ativado!" or "Revistar com tecla T desativado!",
        Duration = 3
    })
end)

-- Sistema de detecção da tecla T
local UserInputService = game:GetService("UserInputService")
local TextChatService = game:GetService("TextChatService")

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.T and getgenv().Enabled then
        -- Função para enviar /revistar morto
        local function sendRevistarMessage()
            local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
            channel:SendAsync("/revistar morto")
        end
        
        -- Executa a função
        sendRevistarMessage()
        
       
    end
end)

-- Seção MOBILE
Tabs.Main:AddSection("MOBILE")

-- Botão para mobile
Tabs.Main:AddButton({
    Title = "mandar revistar UI",
    Description = "Abre interface para revistar (Mobile)",
    Callback = function()
        -- Verifica se a UI já existe
        local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
        if playerGui:FindFirstChild("RevistarUI") then
            Fluent:Notify({
                Title = "Aviso",
                Content = "A interface Revistar já está aberta!",
                Duration = 3
            })
            return
        end

        local TextChatService = game:GetService("TextChatService")

        -- Função para enviar a mensagem /revistar morto
        local function sendRevistarMessage()
            local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
            channel:SendAsync("/revistar morto")
           
        end

        -- Cria a interface
        local ScreenGui = Instance.new("ScreenGui")
        local Frame = Instance.new("Frame")
        local UICorner = Instance.new("UICorner")
        local UIStroke = Instance.new("UIStroke")
        local CloseButton = Instance.new("TextButton")
        local CloseButtonCorner = Instance.new("UICorner")
        local RevistarButton = Instance.new("TextButton")
        local RevistarButtonCorner = Instance.new("UICorner")
        local Title = Instance.new("TextLabel")

        ScreenGui.Name = "RevistarUI"
        ScreenGui.Parent = playerGui
        ScreenGui.ResetOnSpawn = false

        -- Estilo do Frame (vermelho escuro)
        Frame.Size = UDim2.new(0, 300, 0, 150)
        Frame.Position = UDim2.new(0.5, -150, 0.5, -100) -- Start above for animation
        Frame.BackgroundColor3 = Color3.fromRGB(120, 20, 20) -- Vermelho escuro
        Frame.BorderSizePixel = 0
        Frame.BackgroundTransparency = 0.1
        Frame.Active = true
        UICorner.CornerRadius = UDim.new(0, 8)
        UICorner.Parent = Frame
        UIStroke.Thickness = 1
        UIStroke.Color = Color3.fromRGB(180, 50, 50) -- Borda vermelha clara
        UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        UIStroke.Parent = Frame

        -- Modern draggable behavior
        local dragging, dragInput, dragStart, startPos
        Frame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = input.Position
                startPos = Frame.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                    end
                end)
            end
        end)
        Frame.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if input == dragInput and dragging then
                local delta = input.Position - dragStart
                Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)

        -- Título (vermelho mais escuro)
        Title.Size = UDim2.new(1, 0, 0, 30)
        Title.Position = UDim2.new(0, 0, 0, 0)
        Title.BackgroundColor3 = Color3.fromRGB(80, 0, 0) -- Vermelho muito escuro
        Title.Text = "Revistar UI"
        Title.TextColor3 = Color3.fromRGB(255, 255, 255)
        Title.Font = Enum.Font.SourceSansBold
        Title.TextSize = 18

        -- Botão Fechar (vermelho brilhante)
        CloseButton.Size = UDim2.new(0, 30, 0, 30)
        CloseButton.Position = UDim2.new(1, -30, 0, 0)
        CloseButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0) -- Vermelho brilhante
        CloseButton.Text = "X"
        CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        CloseButton.Font = Enum.Font.SourceSansBold
        CloseButton.TextSize = 18
        CloseButtonCorner.CornerRadius = UDim.new(0, 6)
        CloseButtonCorner.Parent = CloseButton
        CloseButton.MouseButton1Click:Connect(function()
            ScreenGui:Destroy()
            game:GetService("SoundService"):PlayLocalSound(Instance.new("Sound", game.SoundService) { SoundId = "rbxassetid://4590662766", Volume = 0.5 })
        end)

        -- Botão Revistar (vermelho vibrante)
        RevistarButton.Size = UDim2.new(0.8, 0, 0.4, 0)
        RevistarButton.Position = UDim2.new(0.1, 0, 0.5, -30)
        RevistarButton.BackgroundColor3 = Color3.fromRGB(180, 30, 30) -- Vermelho vibrante
        RevistarButton.Text = "manda /revistar morto"
        RevistarButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        RevistarButton.Font = Enum.Font.SourceSansBold
        RevistarButton.TextSize = 20
        RevistarButton.AutoButtonColor = true
        RevistarButtonCorner.CornerRadius = UDim.new(0, 6)
        RevistarButtonCorner.Parent = RevistarButton
        RevistarButton.MouseButton1Click:Connect(function()
            sendRevistarMessage()
            game:GetService("SoundService"):PlayLocalSound(Instance.new("Sound", game.SoundService) { SoundId = "rbxassetid://4590662766", Volume = 0.5 })
        end)

        -- Adiciona os elementos ao frame
        Frame.Parent = ScreenGui
        Title.Parent = Frame
        CloseButton.Parent = Frame
        RevistarButton.Parent = Frame

        -- Animação de abertura
        Frame:TweenPosition(UDim2.new(0.5, -150, 0.5, -75), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)

        -- Notificação de abertura
        Fluent:Notify({
            Title = "Interface Aberta",
            Content = "Interface mobile de revistar aberta!",
            Duration = 3
        })
    end
})


-- Inicializar variáveis globais
getgenv().ESPPlayersEnabled = getgenv().ESPPlayersEnabled or false
getgenv().ESPStaffEnabled = getgenv().ESPStaffEnabled or false
getgenv().ESPTextEnabled = getgenv().ESPTextEnabled or false



-- Criar aba Visual
local Tabs = {
    Visual = Window:AddTab({ Title = "Visual 👁️‍", Icon = "eye" })
}

-- ESP de Players (Box e Skeleton)
local ESP = {
    Enabled = false,
    Settings = {
        TeamCheck = false,
        AliveCheck = true,
        BorderColor = Color3.fromRGB(255, 255, 255),
        SkeletonColor = Color3.fromRGB(255, 0, 0),
        BorderTransparency = 0.5,
        SkeletonTransparency = 0.5,
        BorderThickness = 3,
        SkeletonThickness = 1
    },
    Borders = {},
    Skeletons = {},
    Connections = {}
}

local function CreateEsp(player)
    if player == game.Players.LocalPlayer then return end
    
    local border = Drawing.new("Square")
    border.Visible = false
    border.Color = ESP.Settings.BorderColor
    border.Thickness = ESP.Settings.BorderThickness
    border.Transparency = ESP.Settings.BorderTransparency
    border.Filled = false
    ESP.Borders[player] = border

    local skeleton = {
        headToTorso = Drawing.new("Line"),
        torsoToLeftArm = Drawing.new("Line"),
        torsoToRightArm = Drawing.new("Line"),
        torsoToLeftLeg = Drawing.new("Line"),
        torsoToRightLeg = Drawing.new("Line")
    }

    for _, line in pairs(skeleton) do
        line.Visible = false
        line.Color = ESP.Settings.SkeletonColor
        line.Thickness = ESP.Settings.SkeletonThickness
        line.Transparency = ESP.Settings.SkeletonTransparency
    end

    ESP.Skeletons[player] = skeleton
end

local function UpdateEsp()
    for player, border in pairs(ESP.Borders) do
        local skeleton = ESP.Skeletons[player]
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or not player.Character:FindFirstChildOfClass("Humanoid") then
            border.Visible = false
            for _, line in pairs(skeleton) do line.Visible = false end
            continue
        end

        if ESP.Settings.AliveCheck and player.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then
            border.Visible = false
            for _, line in pairs(skeleton) do line.Visible = false end
            continue
        end

        local rootPart = player.Character.HumanoidRootPart
        local head = player.Character:FindFirstChild("Head")
        if not rootPart or not head then
            border.Visible = false
            for _, line in pairs(skeleton) do line.Visible = false end
            continue
        end

        local headPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
        local rootPos = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position - Vector3.new(0, 3, 0))

        if onScreen then
            local boxSize = Vector2.new((headPos.Y - rootPos.Y) * 1.5, (headPos.Y - rootPos.Y) * 1.5)
            border.Size = Vector2.new(boxSize.X + 4, boxSize.Y + 4)
            border.Position = Vector2.new(headPos.X - (boxSize.X + 4) / 2, headPos.Y - (boxSize.Y + 4) / 1.5)
            border.Visible = ESP.Enabled

            local torsoPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("UpperTorso") and player.Character.UpperTorso.Position or rootPart.Position)
            local leftArmPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("LeftHand") and player.Character.LeftHand.Position or (rootPart.Position - Vector3.new(1, 0, 0)))
            local rightArmPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("RightHand") and player.Character.RightHand.Position or (rootPart.Position + Vector3.new(1, 0, 0)))
            local leftLegPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("LeftFoot") and player.Character.LeftFoot.Position or (rootPart.Position - Vector3.new(0.5, 2, 0)))
            local rightLegPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("RightFoot") and player.Character.RightFoot.Position or (rootPart.Position + Vector3.new(0.5, 2, 0)))

            skeleton.headToTorso.From = Vector2.new(headPos.X, headPos.Y)
            skeleton.headToTorso.To = Vector2.new(torsoPos.X, torsoPos.Y)
            skeleton.torsoToLeftArm.From = Vector2.new(torsoPos.X, torsoPos.Y)
            skeleton.torsoToLeftArm.To = Vector2.new(leftArmPos.X, leftArmPos.Y)
            skeleton.torsoToRightArm.From = Vector2.new(torsoPos.X, torsoPos.Y)
            skeleton.torsoToRightArm.To = Vector2.new(rightArmPos.X, rightArmPos.Y)
            skeleton.torsoToLeftLeg.From = Vector2.new(torsoPos.X, torsoPos.Y)
            skeleton.torsoToLeftLeg.To = Vector2.new(leftLegPos.X, leftLegPos.Y)
            skeleton.torsoToRightLeg.From = Vector2.new(torsoPos.X, torsoPos.Y)
            skeleton.torsoToRightLeg.To = Vector2.new(rightLegPos.X, rightLegPos.Y)

            for _, line in pairs(skeleton) do line.Visible = ESP.Enabled end
        else
            border.Visible = false
            for _, line in pairs(skeleton) do line.Visible = false end
        end
    end
end

-- Inicializar ESP para jogadores existentes
for _, player in pairs(game.Players:GetPlayers()) do CreateEsp(player) end

-- Conexões do ESP
ESP.Connections.PlayerAdded = game.Players.PlayerAdded:Connect(CreateEsp)
ESP.Connections.PlayerRemoving = game.Players.PlayerRemoving:Connect(function(player)
    if ESP.Borders[player] then
        ESP.Borders[player]:Remove()
        ESP.Borders[player] = nil
    end
    if ESP.Skeletons[player] then
        for _, line in pairs(ESP.Skeletons[player]) do line:Remove() end
        ESP.Skeletons[player] = nil
    end
end)
ESP.Connections.RenderStepped = game:GetService("RunService").RenderStepped:Connect(UpdateEsp)

-- Toggle ESP Players
Tabs.Visual:AddToggle("PlayerESP", {
    Title = "ESP Players 🔍",
    Description = "Mostra caixas e esqueletos dos jogadores",
    Default = false
}):OnChanged(function(Value)
    ESP.Enabled = Value
    Fluent:Notify({
        Title = "ESP Players",
        Content = ESP.Enabled and "ESP Players ativado!" or "ESP Players desativado!",
        Duration = 3
    })
end)


-- Variáveis de controle
local espData = {}
local rgbCycleEnabled = false
local currentStaticColor = Color3.fromRGB(180, 50, 50)
local isInventoryESPActive = false
local playerAddedConnection
local playerRemovedConnection
local rgbCoroutine

-- Função para verificar se é staff
local function isStaff(player)
    if not player or not player.Parent then return false end
    local success, result = pcall(function()
        return player.Team and player.Team.Name == "STAFF"
    end)
    return success and result or false
end

-- ESP de Inventário
local function createInventoryESP(player)
    if player == game.Players.LocalPlayer then return end
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end

    -- Remove ESP existente
    local existingGui = character:FindFirstChild("InventoryESP")
    if existingGui then existingGui:Destroy() end

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "InventoryESP"
    billboardGui.Parent = character
    billboardGui.Size = UDim2.new(0, 120, 0, 100)
    billboardGui.StudsOffset = Vector3.new(0, 4.5, 0)
    billboardGui.Adornee = character.HumanoidRootPart
    billboardGui.AlwaysOnTop = true
    billboardGui.MaxDistance = 500

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.7
    frame.BorderSizePixel = 0
    frame.Parent = billboardGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = frame

    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 1
    stroke.Color = Color3.fromRGB(180, 50, 50)
    stroke.Parent = frame

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.2, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(180, 50, 50)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = 14
    nameLabel.Text = player.Name
    nameLabel.TextWrapped = true
    nameLabel.Parent = frame

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, -10, 0.75, 0)
    textLabel.Position = UDim2.new(0, 5, 0.2, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextStrokeTransparency = 0
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.Font = Enum.Font.SourceSans
    textLabel.TextSize = 12
    textLabel.TextWrapped = true
    textLabel.TextYAlignment = Enum.TextYAlignment.Top
    textLabel.Parent = frame

    local function updateInventory()
        if not player or not player.Parent or not character or not character.Parent then
            return false
        end

        local backpack = player:FindFirstChild("Backpack")
        local items = {}
        
        if backpack then
            for _, obj in ipairs(backpack:GetChildren()) do
                if obj:IsA("Tool") or obj:IsA("HopperBin") then
                    table.insert(items, obj.Name)
                end
            end
        end
        
        if character then
            for _, obj in ipairs(character:GetChildren()) do
                if obj:IsA("Tool") or obj:IsA("HopperBin") then
                    table.insert(items, obj.Name)
                end
            end
        end
        
        if #items == 0 then
            textLabel.Text = "Inventário vazio"
        else
            local itemList = "Itens:\n"
            for i, item in ipairs(items) do
                if i <= 16 then
                    itemList = itemList .. "- " .. item .. "\n"
                end
            end
            textLabel.Text = itemList
        end
        return true
    end

    -- Atualização inicial
    local success = pcall(updateInventory)
    if not success then
        billboardGui:Destroy()
        return
    end

    -- Loop de atualização
    task.spawn(function()
        while billboardGui and billboardGui.Parent and isInventoryESPActive do
            if not updateInventory() then
                break
            end
            task.wait(1)
        end
        if billboardGui then
            billboardGui:Destroy()
        end
    end)
end

-- ESP de Jogadores (Health Bar)
local function createESPPlayer(player)
    if player == game.Players.LocalPlayer or not player.Character then return nil end
    local character = player.Character
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    if not rootPart or not humanoid then return nil end

    -- Remove ESP existente
    local existingESP = character:FindFirstChild("HealthESP")
    if existingESP then existingESP:Destroy() end

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "HealthESP"
    billboardGui.Parent = character
    billboardGui.Size = UDim2.new(0, 100, 0, 20)
    billboardGui.StudsOffset = Vector3.new(0, 2.5, 0)
    billboardGui.Adornee = rootPart
    billboardGui.AlwaysOnTop = true
    billboardGui.MaxDistance = 500

    local healthFrame = Instance.new("Frame")
    healthFrame.Size = UDim2.new(1, 0, 0.3, 0)
    healthFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    healthFrame.BackgroundTransparency = 0.5
    healthFrame.BorderSizePixel = 0
    healthFrame.Parent = billboardGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = healthFrame

    local healthBar = Instance.new("Frame")
    healthBar.Size = UDim2.new(0.98, 0, 0.8, 0)
    healthBar.Position = UDim2.new(0.01, 0, 0.1, 0)
    healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    healthBar.BorderSizePixel = 0
    healthBar.Parent = healthFrame

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.7, 0)
    nameLabel.Position = UDim2.new(0, 0, 0.3, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = 12
    nameLabel.Text = player.Name
    nameLabel.TextWrapped = true
    nameLabel.Parent = billboardGui

    local function updateHealthBar()
        if not humanoid or not billboardGui.Parent or not player.Parent then
            return false
        end
        
        local healthPercent = math.max(0, math.min(1, humanoid.Health / humanoid.MaxHealth))
        healthBar.Size = UDim2.new(0.98 * healthPercent, 0, 0.8, 0)
        healthBar.BackgroundColor3 = Color3.fromHSV(healthPercent * 0.3, 1, 1)
        
        if getgenv().ESPTextEnabled then
            nameLabel.Text = string.format("%s\nHP: %.0f/%.0f", player.Name, humanoid.Health, humanoid.MaxHealth)
        else
            nameLabel.Text = player.Name
        end
        return true
    end

    -- Atualização inicial
    if not updateHealthBar() then
        billboardGui:Destroy()
        return nil
    end

    -- Conectar eventos
    local healthChangedConnection = humanoid.HealthChanged:Connect(function()
        updateHealthBar()
    end)

    -- Loop de atualização
    task.spawn(function()
        while billboardGui and billboardGui.Parent and getgenv().ESPPlayersEnabled do
            if not updateHealthBar() then
                break
            end
            task.wait(0.5)
        end
        if healthChangedConnection then
            healthChangedConnection:Disconnect()
        end
        if billboardGui then
            billboardGui:Destroy()
        end
    end)

    return { billboardGui = billboardGui, healthBar = healthBar }
end

-- ESP de Staff
local function createESPStaff(player)
    if not isStaff(player) or not player.Character then return nil end
    local character = player.Character
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    if not rootPart or not humanoid then return nil end

    -- Remove ESP existente
    local existingHighlight = character:FindFirstChild("Highlight")
    if existingHighlight then existingHighlight:Destroy() end
    local existingESP = character:FindFirstChild("StaffESP")
    if existingESP then existingESP:Destroy() end

    local highlight = Instance.new("Highlight")
    highlight.Adornee = character
    highlight.FillColor = currentStaticColor
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = character

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "StaffESP"
    billboardGui.Parent = character
    billboardGui.Size = UDim2.new(0, 100, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, 4, 0)
    billboardGui.Adornee = rootPart
    billboardGui.AlwaysOnTop = true
    billboardGui.MaxDistance = 500

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.TextStrokeTransparency = 0
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextScaled = true
    textLabel.Parent = billboardGui

    local function updateStaffESP()
        if not humanoid or not billboardGui.Parent or not player.Parent then
            return false
        end
        
        local localPlayer = game.Players.LocalPlayer
        local localRootPart = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
        local distance = localRootPart and (localRootPart.Position - rootPart.Position).Magnitude or 0
        
        textLabel.Text = string.format("%s [STAFF]\nHP: %.0f/%.0f\nDist: %.1f", 
            player.Name, humanoid.Health, humanoid.MaxHealth, distance)
        return true
    end

    -- Atualização inicial
    if not updateStaffESP() then
        highlight:Destroy()
        billboardGui:Destroy()
        return nil
    end

    -- Loop de atualização
    task.spawn(function()
        while billboardGui and billboardGui.Parent and getgenv().ESPStaffEnabled do
            if not updateStaffESP() then
                break
            end
            task.wait(0.5)
        end
        if highlight then
            highlight:Destroy()
        end
        if billboardGui then
            billboardGui:Destroy()
        end
    end)

    return { highlight = highlight, billboardGui = billboardGui, textLabel = textLabel }
end

-- Função para limpar ESPs
local function cleanupESPs()
    for player, data in pairs(espData) do
        if data then
            if data.billboardGui then data.billboardGui:Destroy() end
            if data.highlight then data.highlight:Destroy() end
        end
    end
    espData = {}
end

-- Função para atualizar ESPs
local function updateAllESPs()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            -- ESP Staff
            if isStaff(player) and getgenv().ESPStaffEnabled and not espData[player] then
                espData[player] = createESPStaff(player)
            -- ESP Jogadores
            elseif not isStaff(player) and getgenv().ESPPlayersEnabled and not espData[player] then
                espData[player] = createESPPlayer(player)
            end
            
            -- ESP Inventário
            if isInventoryESPActive then
                createInventoryESP(player)
            end
        end
    end
end

-- Função para ciclo RGB
local function cycleRGB()
    if rgbCoroutine then
        task.cancel(rgbCoroutine)
    end
    
    rgbCoroutine = task.spawn(function()
        local hue = 0
        while rgbCycleEnabled do
            hue = (hue + 0.01) % 1
            local color = Color3.fromHSV(hue, 1, 1)
            currentStaticColor = color
            
            for _, data in pairs(espData) do
                if data and data.highlight then
                    data.highlight.FillColor = color
                end
            end
            
            task.wait(0.05)
        end
    end)
end

-- Configurar eventos de jogadores
local function setupPlayerEvents()
    if playerAddedConnection then playerAddedConnection:Disconnect() end
    if playerRemovedConnection then playerRemovedConnection:Disconnect() end
    
    playerAddedConnection = game.Players.PlayerAdded:Connect(function(player)
        task.wait(1) -- Aguarda o jogador carregar
        updateAllESPs()
    end)
    
    playerRemovedConnection = game.Players.PlayerRemoving:Connect(function(player)
        if espData[player] then
            local data = espData[player]
            if data.billboardGui then data.billboardGui:Destroy() end
            if data.highlight then data.highlight:Destroy() end
            espData[player] = nil
        end
    end)
end

-- Toggle ESP Inventário
Tabs.Visual:AddToggle("InventoryESP", {
    Title = "ESP Inventário 🎒",
    Description = "Mostra o inventário dos jogadores",
    Default = false
}):OnChanged(function(value)
    isInventoryESPActive = value
    
    -- Som de feedback
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://4590662766"
        sound.Volume = 0.5
        sound.Parent = game.SoundService
        sound:Play()
        sound.Ended:Connect(function() sound:Destroy() end)
    end)
    
    if value then
        setupPlayerEvents()
        updateAllESPs()
        Fluent:Notify({
            Title = "ESP Ativado",
            Content = "ESP de inventário ativado!",
            Duration = 3
        })
    else
        -- Remove todos os ESPs de inventário
        for _, player in ipairs(game.Players:GetPlayers()) do
            if player.Character then
                local esp = player.Character:FindFirstChild("InventoryESP")
                if esp then esp:Destroy() end
            end
        end
        Fluent:Notify({
            Title = "ESP Desativado",
            Content = "ESP de inventário desativado!",
            Duration = 3
        })
    end
end)



-- Toggle ESP Jogadores
Tabs.Visual:AddToggle("ESPPlayers", {
    Title = "ESP Jogadores 👁️",
    Description = "Mostra barra de vida dos jogadores",
    Default = false
}):OnChanged(function(value)
    getgenv().ESPPlayersEnabled = value
    
    -- Som de feedback
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://4590662766"
        sound.Volume = 0.5
        sound.Parent = game.SoundService
        sound:Play()
        sound.Ended:Connect(function() sound:Destroy() end)
    end)
    
    if value then
        setupPlayerEvents()
        updateAllESPs()
        if rgbCycleEnabled then cycleRGB() end
        Fluent:Notify({
            Title = "ESP Jogadores",
            Content = "ESP com barra de vida ativado!",
            Duration = 3
        })
    else
        -- Remove ESPs de jogadores
        for player, data in pairs(espData) do
            if not isStaff(player) and data then
                if data.billboardGui then data.billboardGui:Destroy() end
                espData[player] = nil
            end
        end
        Fluent:Notify({
            Title = "ESP Jogadores",
            Content = "ESP com barra de vida desativado!",
            Duration = 3
        })
    end
end)

-- Toggle ESP Staff
Tabs.Visual:AddToggle("ESPStaff", {
    Title = "ESP Staff 👮",
    Description = "Destaca membros da staff",
    Default = false
}):OnChanged(function(value)
    getgenv().ESPStaffEnabled = value
    
    -- Som de feedback
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://4590662766"
        sound.Volume = 0.5
        sound.Parent = game.SoundService
        sound:Play()
        sound.Ended:Connect(function() sound:Destroy() end)
    end)
    
    if value then
        setupPlayerEvents()
        updateAllESPs()
        if rgbCycleEnabled then cycleRGB() end
        Fluent:Notify({
            Title = "ESP Staff",
            Content = "ESP Staff ativado!",
            Duration = 3
        })
    else
        -- Remove ESPs de staff
        for player, data in pairs(espData) do
            if isStaff(player) and data then
                if data.highlight then data.highlight:Destroy() end
                if data.billboardGui then data.billboardGui:Destroy() end
                espData[player] = nil
            end
        end
        Fluent:Notify({
            Title = "ESP Staff",
            Content = "ESP Staff desativado!",
            Duration = 3
        })
    end
end)



-- Botão para limpar todos os ESPs
Tabs.Visual:AddButton({
    Title = "Limpar ESPs",
    Description = "Remove todos os ESPs ativos",
    Callback = function()
        cleanupESPs()
        
        -- Remove ESPs de inventário
        for _, player in ipairs(game.Players:GetPlayers()) do
            if player.Character then
                local esp = player.Character:FindFirstChild("InventoryESP")
                if esp then esp:Destroy() end
            end
        end
        
        Fluent:Notify({
            Title = "ESPs Limpos",
            Content = "Todos os ESPs foram removidos!",
            Duration = 3
        })
    end
})






-- Aba de Teleportes
local TeleportTab = Window:AddTab({
    Title = "Teleports 🌀",
    Icon = "portal" -- Ícone de teleporte (ícone padrão da Fluent)
})

-- Adiciona um parágrafo inicial
TeleportTab:AddParagraph({
    Title = "GRAND Teleports",
    Content = "Sistema de teleporte rápido - Feito por Bernardo"
})

-- Função para adicionar botões de teleporte
local function addTeleportButton(name, cframe)
    TeleportTab:AddButton({
        Title = name,
        Description = "Teleportar para " .. name,
        Callback = function()
            local player = game:GetService("Players").LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                -- Teleporta com offset para evitar bugs no chão
                player.Character.HumanoidRootPart.CFrame = cframe * CFrame.new(0, 3, 0)
                Fluent:Notify({
                    Title = "Teleporte",
                    Content = "Teleportado para " .. name .. "!",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Personagem não encontrado!",
                    Duration = 2
                })
            end
        end
    })
end

-- Lista de locais com CFrames
local teleportLocations = {
    {Name = "Praça", CFrame = CFrame.new(-291.579559, 3.26299787, 342.192535)},
    {Name = "Gás", CFrame = CFrame.new(-469.959015, 3.25349784, -54.3936005)},
    {Name = "HP", CFrame = CFrame.new(-543.439941, 3.26299858, 645.16864)},
    {Name = "Tabacaria", CFrame = CFrame.new(-83.1141129, 13.1430578, 74.7073364)},
    {Name = "Garagem", CFrame = CFrame.new(-466.870148, 7.64567232, 350.242737)},
    {Name = "Concessionaria", CFrame = CFrame.new(-91.3902893, 8.07136822, 520.355347)},
    {Name = "Gari", CFrame = CFrame.new(-518.672852, 3.16749811, -1.16962147, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "Imobiliaria", CFrame = CFrame.new(-284.904785, 8.26088619, -72.2896194, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "PM", CFrame = CFrame.new(-980.181458, 2.27553082, 467.080536, 1, 0, 0, 0, 1, 0, 0, 0, 1)},
    {Name = "PRF", CFrame = CFrame.new(6662.24512, 36.6637421, 5047.83838, 0.707134247, 0, 0.707079291, 0, 1, 0, -0.707079291, 0, 0.707134247)},
    {Name = "Mineração", CFrame = CFrame.new(201.932144, 2.76136589, 145.50531, 0, 0, 1, 0, 1, -0, -1, 0, 0)},
    {Name = "Mecânica", CFrame = CFrame.new(-180.608261, 3.29813337, -532.4151, 0.422592998, -0, -0.906319618, 0, 1, -0, 0.906319618, 0, 0.422592998)},
    {Name = "Fazenda", CFrame = CFrame.new(817.243225, 3.26249814, -87.316864, 0, 0, 1, 0, 1, 0, -1, 0, 0)},
    {Name = "Prefeitura", CFrame = CFrame.new(-284.388458, 15.1148872, 88.0397873, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "Banco", CFrame = CFrame.new(-27.2709007, 11.5685892, 418.200653, 1, 0, 0, 0, 1, 0, 0, 0, 1)},
    {Name = "Ilegal", CFrame = CFrame.new(12045.0146, 37.3188896, 12826.2041, -0.257956624, 0.00115467981, -0.966156006, -0.0795417428, 0.99657917, 0.0224280898, 0.962876916, 0.0826351941, -0.256982446)},
    {Name = "Predio 1", CFrame = CFrame.new(-1595.23328, 204.074341, 555.895386, 0.939687431, -0.34203434, 1.81794167e-06, 1.81794167e-06, 1.02519989e-05, 1, -0.34203434, -0.93968749, 1.02519989e-05)},
    {Name = "Devs Mini City", CFrame = CFrame.new(2555.44263, 303.167755, -1004.13763, -0.422592998, 0, 0.906319618, 0, 1, 0, -0.906319618, 0, -0.422592998)}
}

-- Adiciona botões para cada local
for _, location in ipairs(teleportLocations) do
    addTeleportButton(location.Name, location.CFrame)
end

-- Seção para teleporte personalizado
TeleportTab:AddSection("Teleporte Personalizado")

-- Variáveis para salvar posição
local savedPosition = nil

-- Botão para salvar posição atual
TeleportTab:AddButton({
    Title = "Salvar Posição Atual 📍",
    Description = "Salva sua posição atual",
    Callback = function()
        local player = game:GetService("Players").LocalPlayer
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            savedPosition = player.Character.HumanoidRootPart.CFrame
            Fluent:Notify({
                Title = "Posição Salva",
                Content = "Posição atual salva com sucesso!",
                Duration = 2
            })
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Personagem não encontrado!",
                Duration = 2
            })
        end
    end
})

-- Botão para teleportar para posição salva
TeleportTab:AddButton({
    Title = "Teleportar para Posição Salva 🏠",
    Description = "Teleporta para a posição salva",
    Callback = function()
        if savedPosition then
            local player = game:GetService("Players").LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = savedPosition
                Fluent:Notify({
                    Title = "Teleporte",
                    Content = "Teleportado para a posição salva!",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Personagem não encontrado!",
                    Duration = 2
                })
            end
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Nenhuma posição salva!",
                Duration = 2
            })
        end
    end
})

-- Inputs para teleporte por coordenadas
local xInput = TeleportTab:AddInput("XInput", {
    Title = "Coordenada X",
    Placeholder = "Digite X",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de X
    end
})

local yInput = TeleportTab:AddInput("YInput", {
    Title = "Coordenada Y",
    Placeholder = "Digite Y",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de Y
    end
})

local zInput = TeleportTab:AddInput("ZInput", {
    Title = "Coordenada Z",
    Placeholder = "Digite Z",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de Z
    end
})

-- Botão para teleporte por coordenadas
TeleportTab:AddButton({
    Title = "Teleportar para Coordenadas 📌",
    Description = "Teleporta para as coordenadas inseridas",
    Callback = function()
        local x = tonumber(xInput.Value) or 0
        local y = tonumber(yInput.Value) or 0
        local z = tonumber(zInput.Value) or 0
        local player = game:GetService("Players").LocalPlayer
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = CFrame.new(x, y, z)
            Fluent:Notify({
                Title = "Teleporte",
                Content = "Teleportado para (" .. x .. ", " .. y .. ", " .. z .. ")!",
                Duration = 2
            })
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Personagem não encontrado!",
                Duration = 2
            })
        end
    end
})



-- Exibe a janela
Window:SelectTab(1)



-- Variáveis do sistema de voo
local flying = false
local flySpeed = 15
local minSpeed = 1
local maxSpeed = 999
local speedStep = 5
local antiFallEnabled = false
local flyConnection = nil
local uiMinimized = false

-- Criando a interface de voo personalizada
local screenGuiFly = Instance.new("ScreenGui")
screenGuiFly.Name = "FlyGui"
screenGuiFly.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGuiFly.ResetOnSpawn = false
screenGuiFly.Enabled = false

local frameFly = Instance.new("Frame")
frameFly.Name = "FlyFrame"
frameFly.Size = UDim2.new(0, 220, 0, 140)
frameFly.Position = UDim2.new(0.5, -110, 0.1, 0)
frameFly.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frameFly.BorderSizePixel = 0
frameFly.Active = true
frameFly.Draggable = true
frameFly.Parent = screenGuiFly
local cornerFly = Instance.new("UICorner")
cornerFly.CornerRadius = UDim.new(0, 12)
cornerFly.Parent = frameFly

-- Título da interface
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(0.7, 0, 0.15, 0)
titleLabel.Position = UDim2.new(0.05, 0, 0.05, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.Text = "Sistema de Voo ✈️"
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = frameFly

local buttonFly = Instance.new("TextButton")
buttonFly.Name = "FlyButton"
buttonFly.Size = UDim2.new(0.9, 0, 0.2, 0)
buttonFly.Position = UDim2.new(0.05, 0, 0.22, 0)
buttonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
buttonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
buttonFly.Font = Enum.Font.GothamBold
buttonFly.TextSize = 16
buttonFly.Text = "Ativar Voo ✈️"
buttonFly.Parent = frameFly
local buttonCornerFly = Instance.new("UICorner")
buttonCornerFly.CornerRadius = UDim.new(0, 8)
buttonCornerFly.Parent = buttonFly

local speedLabel = Instance.new("TextLabel")
speedLabel.Name = "SpeedLabel"
speedLabel.Size = UDim2.new(0.55, 0, 0.15, 0)
speedLabel.Position = UDim2.new(0.05, 0, 0.45, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 14
speedLabel.Text = "Velocidade: " .. flySpeed
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = frameFly

local minusButtonFly = Instance.new("TextButton")
minusButtonFly.Name = "MinusButton"
minusButtonFly.Size = UDim2.new(0.15, 0, 0.15, 0)
minusButtonFly.Position = UDim2.new(0.65, 0, 0.45, 0)
minusButtonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minusButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
minusButtonFly.Font = Enum.Font.GothamBold
minusButtonFly.TextSize = 16
minusButtonFly.Text = "-"
minusButtonFly.Parent = frameFly
local minusCornerFly = Instance.new("UICorner")
minusCornerFly.CornerRadius = UDim.new(0, 6)
minusCornerFly.Parent = minusButtonFly

local plusButtonFly = Instance.new("TextButton")
plusButtonFly.Name = "PlusButton"
plusButtonFly.Size = UDim2.new(0.15, 0, 0.15, 0)
plusButtonFly.Position = UDim2.new(0.82, 0, 0.45, 0)
plusButtonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
plusButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
plusButtonFly.Font = Enum.Font.GothamBold
plusButtonFly.TextSize = 16
plusButtonFly.Text = "+"
plusButtonFly.Parent = frameFly
local plusCornerFly = Instance.new("UICorner")
plusCornerFly.CornerRadius = UDim.new(0, 6)
plusCornerFly.Parent = plusButtonFly

local antiFallToggle = Instance.new("TextButton")
antiFallToggle.Name = "AntiFallToggle"
antiFallToggle.Size = UDim2.new(0.9, 0, 0.15, 0)
antiFallToggle.Position = UDim2.new(0.05, 0, 0.63, 0)
antiFallToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
antiFallToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
antiFallToggle.Font = Enum.Font.GothamBold
antiFallToggle.TextSize = 14
antiFallToggle.Text = "Anti-Dano: OFF"
antiFallToggle.Parent = frameFly
local antiFallCorner = Instance.new("UICorner")
antiFallCorner.CornerRadius = UDim.new(0, 6)
antiFallCorner.Parent = antiFallToggle

local controlsLabel = Instance.new("TextLabel")
controlsLabel.Name = "ControlsLabel"
controlsLabel.Size = UDim2.new(0.9, 0, 0.12, 0)
controlsLabel.Position = UDim2.new(0.05, 0, 0.81, 0)
controlsLabel.BackgroundTransparency = 1
controlsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
controlsLabel.Font = Enum.Font.Gotham
controlsLabel.TextSize = 11
controlsLabel.Text = "WASD + Space/Shift | G = Toggle"
controlsLabel.TextXAlignment = Enum.TextXAlignment.Center
controlsLabel.Parent = frameFly

local minimizeButtonFly = Instance.new("TextButton")
minimizeButtonFly.Name = "MinimizeButton"
minimizeButtonFly.Size = UDim2.new(0.12, 0, 0.12, 0)
minimizeButtonFly.Position = UDim2.new(0.78, 0, 0.05, 0)
minimizeButtonFly.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
minimizeButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButtonFly.Font = Enum.Font.GothamBold
minimizeButtonFly.TextSize = 12
minimizeButtonFly.Text = "−"
minimizeButtonFly.Parent = frameFly
local minimizeCornerFly = Instance.new("UICorner")
minimizeCornerFly.CornerRadius = UDim.new(0, 4)
minimizeCornerFly.Parent = minimizeButtonFly

local closeButtonFly = Instance.new("TextButton")
closeButtonFly.Name = "CloseButton"
closeButtonFly.Size = UDim2.new(0.12, 0, 0.12, 0)
closeButtonFly.Position = UDim2.new(0.91, 0, 0.05, 0)
closeButtonFly.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
closeButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButtonFly.Font = Enum.Font.GothamBold
closeButtonFly.TextSize = 12
closeButtonFly.Text = "✕"
closeButtonFly.Parent = frameFly
local closeCornerFly = Instance.new("UICorner")
closeCornerFly.CornerRadius = UDim.new(0, 4)
closeCornerFly.Parent = closeButtonFly

-- Funções auxiliares
local function disableCollisions(character)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then 
            part.CanCollide = false 
        end
    end
end

local function enableCollisions(character)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then 
            part.CanCollide = true 
        end
    end
end

local function antiFallDamage()
    while antiFallEnabled do
        task.wait(0.1)
        if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart and not flying and rootPart.Velocity.Y < -50 then
                rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z)
            end
        end
    end
end

local function startFlying()
    local character = game.Players.LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") or not character:FindFirstChildOfClass("Humanoid") then 
        return false 
    end
    
    local humanoidRootPart = character.HumanoidRootPart
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    -- Configurando o voo
    humanoid.PlatformStand = true
    humanoidRootPart.Anchored = true
    disableCollisions(character)
    flying = true
    
    -- Atualizando interface
    buttonFly.Text = "Desativar Voo ✈️"
    buttonFly.BackgroundColor3 = Color3.fromRGB(100, 50, 50)

    -- Loop de movimento
    flyConnection = game:GetService("RunService").Heartbeat:Connect(function(deltaTime)
        if not flying or not character or not humanoidRootPart or not humanoid then
            if flyConnection then 
                flyConnection:Disconnect() 
                flyConnection = nil
            end
            return
        end

        local moveDirection = Vector3.new(0, 0, 0)
        local UserInputService = game:GetService("UserInputService")
        
        -- Controles de movimento
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then 
            moveDirection = moveDirection + workspace.CurrentCamera.CFrame.LookVector 
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then 
            moveDirection = moveDirection - workspace.CurrentCamera.CFrame.LookVector 
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then 
            moveDirection = moveDirection - workspace.CurrentCamera.CFrame.RightVector 
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then 
            moveDirection = moveDirection + workspace.CurrentCamera.CFrame.RightVector 
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then 
            moveDirection = moveDirection + Vector3.new(0, 1, 0) 
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then 
            moveDirection = moveDirection - Vector3.new(0, 1, 0) 
        end

        -- Aplicando movimento
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
            local speed = flySpeed * deltaTime * 10
            humanoidRootPart.Anchored = false
            humanoidRootPart.CFrame = humanoidRootPart.CFrame + moveDirection * speed
            
            -- Rotação do personagem
            if moveDirection.X ~= 0 or moveDirection.Z ~= 0 then
                local lookVector = workspace.CurrentCamera.CFrame.LookVector
                humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position) * CFrame.Angles(0, math.atan2(lookVector.X, lookVector.Z), 0)
            end
            
            humanoidRootPart.Anchored = true
        end
    end)
    
    return true
end

local function stopFlying()
    local character = game.Players.LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
    
    -- Parando o voo
    if flyConnection then 
        flyConnection:Disconnect() 
        flyConnection = nil
    end
    
    if humanoid then 
        humanoid.PlatformStand = false 
    end
    if character then 
        enableCollisions(character) 
    end
    if humanoidRootPart then 
        humanoidRootPart.Anchored = false 
    end
    
    flying = false
    
    -- Atualizando interface
    buttonFly.Text = "Ativar Voo ✈️"
    buttonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
end

local function adjustSpeed(delta)
    flySpeed = math.clamp(flySpeed + delta, minSpeed, maxSpeed)
    speedLabel.Text = "Velocidade: " .. flySpeed
end

local function toggleUIMinimize()
    if uiMinimized then
        frameFly.Size = UDim2.new(0, 220, 0, 140)
        buttonFly.Visible = true
        speedLabel.Visible = true
        plusButtonFly.Visible = true
        minusButtonFly.Visible = true
        antiFallToggle.Visible = true
        controlsLabel.Visible = true
        minimizeButtonFly.Text = "−"
        uiMinimized = false
    else
        frameFly.Size = UDim2.new(0, 220, 0, 30)
        buttonFly.Visible = false
        speedLabel.Visible = false
        plusButtonFly.Visible = false
        minusButtonFly.Visible = false
        antiFallToggle.Visible = false
        controlsLabel.Visible = false
        minimizeButtonFly.Text = "+"
        uiMinimized = true
    end
end

-- Conectando eventos da interface
buttonFly.MouseButton1Click:Connect(function()
    if flying then 
        stopFlying() 
    else 
        startFlying() 
    end
end)

antiFallToggle.MouseButton1Click:Connect(function()
    antiFallEnabled = not antiFallEnabled
    antiFallToggle.Text = "Anti-Dano: " .. (antiFallEnabled and "ON" or "OFF")
    antiFallToggle.BackgroundColor3 = antiFallEnabled and Color3.fromRGB(50, 100, 50) or Color3.fromRGB(60, 60, 60)
    if antiFallEnabled then 
        task.spawn(antiFallDamage) 
    end
end)

plusButtonFly.MouseButton1Click:Connect(function() 
    adjustSpeed(speedStep) 
end)

minusButtonFly.MouseButton1Click:Connect(function() 
    adjustSpeed(-speedStep) 
end)

minimizeButtonFly.MouseButton1Click:Connect(toggleUIMinimize)

closeButtonFly.MouseButton1Click:Connect(function()
    screenGuiFly.Enabled = false
    if flying then 
        stopFlying() 
    end
end)



local MainTab = Window:AddTab({ Title = "Voo", Icon = "plane" })

-- Toggle para mostrar/esconder interface de voo
local ShowFlyUI = MainTab:AddToggle("ShowFlyUI", {
    Title = "Abrir Interface de Voo",
    Description = "Abre/fecha a interface de controle do voo",
    Default = false
})

ShowFlyUI:OnChanged(function(Value)
    screenGuiFly.Enabled = Value
end)

-- Keybind para voo
local FlyKeybind = MainTab:AddKeybind("FlyKeybind", {
    Title = "Atalho do Voo",
    Mode = "Toggle",
    Default = "G",
    Callback = function()
        if flying then
            stopFlying()
        else
            startFlying()
        end
    end
})

-- Limpeza ao sair
game.Players.LocalPlayer.CharacterRemoving:Connect(function()
    if flying then
        stopFlying()
    end
end)

local ShowFlyUI = MainTab:AddToggle("ShowFlyUI", {
    Title = "Abrir Interface de Voo 2",
    Description = "Abre/fecha a interface de controle do voo",
    Default = false
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local Settings = {
    Fly = false,
    Speed = 2,
    NoFall = false
}

local flySpeed = 50
local descendantConnection = nil

-- 📛 Função para deletar eventos de dano, queda e similares
local function deleteDamageEvents()
    for _, obj in ipairs(game:GetDescendants()) do
        if obj:IsA("RemoteEvent") or obj:IsA("BindableEvent") then
            local name = string.lower(obj.Name)
            if name:find("damage") or name:find("dano") or name:find("fall") or name:find("hurt") then
                obj:Destroy()
            end
        end
    end
end

-- 🛡️ Função para ativar/desativar No Fall
local function toggleNoFall()
    Settings.NoFall = not Settings.NoFall
    if Settings.NoFall then
        print("✅ No Fall Ativado!")
        deleteDamageEvents()
        descendantConnection = game.DescendantAdded:Connect(function(obj)
            if obj:IsA("RemoteEvent") or obj:IsA("BindableEvent") then
                local name = string.lower(obj.Name)
                if name:find("damage") or name:find("dano") or name:find("fall") or name:find("hurt") then
                    obj:Destroy()
                end
            end
        end)
    else
        print("❌ No Fall Desativado!")
        if descendantConnection then
            descendantConnection:Disconnect()
            descendantConnection = nil
        end
    end
end

-- 🖥️ Criação da interface Fly GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FlyGui"
ScreenGui.Parent = game.CoreGui
ScreenGui.Enabled = false -- Começa fechado

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 180, 0, 140)
MainFrame.Position = UDim2.new(0, 20, 0, 100)
MainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainFrame.BorderSizePixel = 0

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 8)
Corner.Parent = MainFrame

-- Botão Fly
local FlyButton = Instance.new("TextButton")
FlyButton.Parent = MainFrame
FlyButton.Size = UDim2.new(0.9, 0, 0, 35)
FlyButton.Position = UDim2.new(0.05, 0, 0, 10)
FlyButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
FlyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FlyButton.TextScaled = true
FlyButton.Font = Enum.Font.SourceSansBold
FlyButton.Text = "Ativar Fly ✈️"

local FlyCorner = Instance.new("UICorner", FlyButton)
FlyCorner.CornerRadius = UDim.new(0, 4)

-- Botão No Fall
local NoFallButton = Instance.new("TextButton")
NoFallButton.Parent = MainFrame
NoFallButton.Size = UDim2.new(0.9, 0, 0, 25)
NoFallButton.Position = UDim2.new(0.05, 0, 0, 50)
NoFallButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
NoFallButton.TextColor3 = Color3.fromRGB(255, 255, 255)
NoFallButton.TextScaled = true
NoFallButton.Font = Enum.Font.SourceSans
NoFallButton.Text = "Ativar No Fall 🛡️"

local NoFallCorner = Instance.new("UICorner", NoFallButton)
NoFallCorner.CornerRadius = UDim.new(0, 4)

-- Label da Velocidade
local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Parent = MainFrame
SpeedLabel.Size = UDim2.new(0.9, 0, 0, 20)
SpeedLabel.Position = UDim2.new(0.05, 0, 0, 80)
SpeedLabel.BackgroundTransparency = 1
SpeedLabel.Text = "Velocidade: " .. flySpeed
SpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
SpeedLabel.TextScaled = true
SpeedLabel.Font = Enum.Font.SourceSans

-- Botões de velocidade
local SpeedDownButton = Instance.new("TextButton")
SpeedDownButton.Parent = MainFrame
SpeedDownButton.Size = UDim2.new(0.25, 0, 0, 25)
SpeedDownButton.Position = UDim2.new(0.05, 0, 0, 105)
SpeedDownButton.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
SpeedDownButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedDownButton.TextScaled = true
SpeedDownButton.Font = Enum.Font.SourceSansBold
SpeedDownButton.Text = "- 10"

local SpeedUpButton = Instance.new("TextButton")
SpeedUpButton.Parent = MainFrame
SpeedUpButton.Size = UDim2.new(0.25, 0, 0, 25)
SpeedUpButton.Position = UDim2.new(0.7, 0, 0, 105)
SpeedUpButton.BackgroundColor3 = Color3.fromRGB(40, 80, 40)
SpeedUpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedUpButton.TextScaled = true
SpeedUpButton.Font = Enum.Font.SourceSansBold
SpeedUpButton.Text = "+ 10"

for _, button in pairs({SpeedDownButton, SpeedUpButton}) do
    local corner = Instance.new("UICorner", button)
    corner.CornerRadius = UDim.new(0, 3)
end

-- 🔥 Funções
local function SetNoClip(state)
    if LocalPlayer.Character then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = not state
            end
        end
    end
end

local function ToggleFly()
    Settings.Fly = not Settings.Fly
    if Settings.Fly then
        LocalPlayer.Character.Humanoid.PlatformStand = true
        FlyButton.Text = "Desativar Fly ✈️"
        FlyButton.BackgroundColor3 = Color3.fromRGB(100, 50, 50)
        SetNoClip(true)
    else
        LocalPlayer.Character.Humanoid.PlatformStand = false
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
        FlyButton.Text = "Ativar Fly ✈️"
        FlyButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        SetNoClip(false)
    end
end

local function ToggleNoFallButton()
    toggleNoFall()
    if Settings.NoFall then
        NoFallButton.Text = "Desativar No Fall 🛡️"
        NoFallButton.BackgroundColor3 = Color3.fromRGB(50, 100, 50)
    else
        NoFallButton.Text = "Ativar No Fall 🛡️"
        NoFallButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    end
end

local function AdjustSpeed(change)
    flySpeed = math.clamp(flySpeed + change, 10, 200)
    SpeedLabel.Text = "Velocidade: " .. flySpeed
end

-- 🖱️ Conexões dos botões
FlyButton.MouseButton1Click:Connect(ToggleFly)
NoFallButton.MouseButton1Click:Connect(ToggleNoFallButton)
SpeedUpButton.MouseButton1Click:Connect(function()
    AdjustSpeed(10)
end)
SpeedDownButton.MouseButton1Click:Connect(function()
    AdjustSpeed(-10)
end)

-- 🎛️ Teclas de atalho
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        Settings.Speed = 3
    elseif input.KeyCode == Enum.KeyCode.j then
        ToggleFly()
    elseif input.KeyCode == Enum.KeyCode.b then
        ToggleNoFallButton()
    elseif input.KeyCode == Enum.KeyCode.V then
        ToggleFly() -- Tecla V agora também ativa/desativa o fly
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftShift then
        Settings.Speed = 2
    end
end)

-- 🏃 Loop do voo
RunService.Heartbeat:Connect(function()
    if Settings.Fly then
        local direction = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then direction += Camera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then direction -= Camera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then direction -= Camera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then direction += Camera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then direction += Vector3.new(0, 1, 0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then direction -= Vector3.new(0, 1, 0) end

        LocalPlayer.Character.HumanoidRootPart.Velocity = direction * flySpeed * Settings.Speed
        SetNoClip(true)
    end
end)

-- 🔘 Toggle do Fluent UI para abrir/fechar o menu
ShowFlyUI:OnChanged(function(state)
    ScreenGui.Enabled = state
end)

toggleNoFall()



MainTab:AddParagraph({
    Title = "Nossos fly",
    Content = "Recomendações: Voo 1: bom pra azaralhar sem ban (ninguém te vê)Voo 2: ótimo pra farmar — usa a tecla V pra ativar"
})



