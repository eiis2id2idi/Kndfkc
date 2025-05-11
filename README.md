-- Carregar Fluent UI
local success, Fluent = pcall(function()
    return loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
end)

if not success then
    warn("Erro ao carregar Fluent: " .. tostring(Fluent))
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Erro",
        Text = "Falha ao carregar Fluent. Verifique sua conexão ou executor.",
        Duration = 10
    })
    return
end

local Window = Fluent:CreateWindow({
    Title = "Frozen & Leviathan Hub",
    SubTitle = "by xAI Helper",
    TabWidth = 160,
    Size = UDim2.fromOffset(600, 460),
    Acrylic = true,
    Theme = "Light",
    MinimizeKey = Enum.KeyCode.LeftControl
})

Fluent:Notify({
    Title = "Script Iniciado",
    Content = "Pressione LeftControl para abrir!",
    Duration = 8
})

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Tabs
local Tabs = {
    Frozen = Window:AddTab({ Title = "Frozen/Leviathan", Icon = "snowflake" }),
    Miragem = Window:AddTab({ Title = "Miragem", Icon = "map" }),
    Status = Window:AddTab({ Title = "Status", Icon = "info" })
}

-- Configurações Globais
_G.FlyFrozenActive = false
_G.FlyMiragemActive = false
_G.BoatSpeed = 0.00004
_G.SelectedBoat = "BeastHunterBoat"

-- Função Genérica de Voo
local function FlyToTarget(targetPos, flagName)
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid then return end

    while _G[flagName] and (hrp.Position - targetPos).Magnitude > 10 do
        hrp.CFrame = hrp.CFrame:Lerp(CFrame.new(targetPos + Vector3.new(0, math.sin(tick()) * 2, 0)), _G.BoatSpeed)
        RunService.Heartbeat:Wait()
    end
end

-- Voo para Frozen
local function ToggleFrozen(state)
    _G.FlyFrozenActive = state
    if state then
        Fluent:Notify({ Title = "Auto Frozen", Content = "Voo iniciado", Duration = 5 })
        task.spawn(function()
            FlyToTarget(Vector3.new(-290000, 50, -1900), "FlyFrozenActive")
        end)
    else
        Fluent:Notify({ Title = "Auto Frozen", Content = "Voo interrompido", Duration = 5 })
    end
end

-- Voo para Miragem
local function ToggleMiragem(state)
    _G.FlyMiragemActive = state
    if state then
        Fluent:Notify({ Title = "Auto Miragem", Content = "Voo iniciado", Duration = 5 })
        task.spawn(function()
            FlyToTarget(Vector3.new(-290000, 60, 1900), "FlyMiragemActive") -- ajuste o destino
        end)
    else
        Fluent:Notify({ Title = "Auto Miragem", Content = "Voo interrompido", Duration = 5 })
    end
end

-- Interrupção por Detecção
Workspace.ChildAdded:Connect(function(child)
    if child:IsA("Model") then
        local name = child.Name:lower()
        if name:find("frozen") and _G.FlyFrozenActive then
            _G.FlyFrozenActive = false
            Fluent:Notify({ Title = "Frozen", Content = "Detectado! Voo parado.", Duration = 6 })
        elseif name:find("mirage") and _G.FlyMiragemActive then
            _G.FlyMiragemActive = false
            Fluent:Notify({ Title = "Miragem", Content = "Detectada! Voo parado.", Duration = 6 })
        end
    end
end)

-- Aba Frozen/Leviathan
Tabs.Frozen:AddSection("Auto Frozen Dimension")
Tabs.Frozen:AddToggle("autofrozen", {
    Title = "Ativar Auto Frozen",
    Default = false,
    Callback = ToggleFrozen
})
Tabs.Frozen:AddSlider("boatspeed", {
    Title = "Velocidade de Voo",
    Min = 0.00001, Max = 0.0001, Default = _G.BoatSpeed, Rounding = 5,
    Callback = function(val) _G.BoatSpeed = val end
})

-- Aba Miragem
Tabs.Miragem:AddSection("Auto Miragem")
Tabs.Miragem:AddToggle("automiragem", {
    Title = "Ativar Auto Miragem",
    Default = false,
    Callback = ToggleMiragem
})
Tabs.Miragem:AddSlider("miragemboatspeed", {
    Title = "Velocidade de Voo",
    Min = 0.00001, Max = 0.0001, Default = _G.BoatSpeed, Rounding = 5,
    Callback = function(val) _G.BoatSpeed = val end
})

-- Aba Status
Tabs.Status:AddSection("Status do Jogo")
Tabs.Status:AddButton({
    Title = "Atualizar",
    Callback = function()
        local islands = 0
        for _, v in pairs(Workspace:GetChildren()) do
            if v:IsA("Model") and v.Name:find("Island") then islands += 1 end
        end
        Fluent:Notify({
            Title = "Status",
            Content = "Ilhas spawnadas: " .. islands,
            Duration = 5
        })
    end
})
