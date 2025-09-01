-- carregar biblioteca 
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- aviso ao executar
Fluent:Notify({ Title = "executado!", Content = "executado com sucesso" })

-- janela
local Window = Fluent:CreateWindow({
    Title = "MK HUB" .. Fluent.Version,
    SubTitle = "by NOVAX",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Theme = "Dark"
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "home" }),
    Farm = Window:AddTab({ Title = "Farm", Icon = "sword" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- parágrafos 
Tabs.Main:AddParagraph({ Title = "SCRIPT NEW", Content = "SCRIPT NEW" })

-- botão infinite jump
local InfiniteJumpEnabled = false
Tabs.Main:AddButton({ Title = "Infinite Jump", Callback = function()
    InfiniteJumpEnabled = true
    Fluent:Notify({ Title = "Infinite Jump", Content = "Ativado" })
    game:GetService("UserInputService").JumpRequest:Connect(function()
        if InfiniteJumpEnabled then
            local humanoid = game:GetService("Players").LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:ChangeState("Jumping")
            end
        end
    end)
end })

-- slider de pulo
local Slider = Tabs.Main:AddSlider("pulo", {
    Title = "Ajustar Pulo",
    Description = "Muda o pulo do jogador",
    Default = 50,
    Min = 10,
    Max = 500,
    Rounding = 0,
    Callback = function(Value)
        local humanoid = game:GetService("Players").LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.JumpPower = Value
        end
    end
})

-- slider de WalkSpeed
local WalkSlider = Tabs.Main:AddSlider("velocidade", {
    Title = "Ajustar Velocidade",
    Description = "Muda a velocidade do jogador",
    Default = 16,
    Min = 10,
    Max = 500,
    Rounding = 0,
    Callback = function(Value)
        local humanoid = game:GetService("Players").LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = Value
        end
    end
})

-- Toggle Auto Shoot
local AutoShoot = false
Tabs.Main:AddToggle("autoShoot", {
    Title = "Auto Shoot",
    Description = "Atira automaticamente com qualquer arma equipada",
    Default = false,
    Callback = function(state)
        AutoShoot = state
        Fluent:Notify({ Title = "Auto Shoot", Content = state and "Ligado" or "Desligado" })
        task.spawn(function()
            local plr = game.Players.LocalPlayer
            while AutoShoot do
                task.wait(0.1)
                local char = plr.Character
                if char then
                    for _, tool in pairs(char:GetChildren()) do
                        if tool:IsA("Tool") then
                            tool:Activate()
                        end
                    end
                end
            end
        end)
    end
})

-- Toggle NoClip
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local NoClip = false

Tabs.Main:AddToggle("noclip", {
    Title = "NoClip",
    Description = "Atravessa todas as paredes/objetos",
    Default = false,
    Callback = function(state)
        NoClip = state
        Fluent:Notify({ Title = "NoClip", Content = state and "Ativado" or "Desativado" })
    end
})

RunService.Stepped:Connect(function()
    if NoClip and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

-- Super Fast Attack
local FastAttackEnabled = false
local mouse = LocalPlayer:GetMouse()

local function SuperFastAttack()
    if FastAttackEnabled and mouse.Target then
        pcall(function()
            game:GetService("ReplicatedStorage").Remotes.Combat:FireServer(mouse.Target)
        end)
    end
end

RunService.Heartbeat:Connect(function()
    if FastAttackEnabled then
        SuperFastAttack()
    end
end)

Tabs.Main:AddToggle("superFastAttack", {
    Title = "Super Fast Attack",
    Description = "Ataca super rápido qualquer alvo",
    Default = false,
    Callback = function(state)
        FastAttackEnabled = state
        Fluent:Notify({ Title = "Super Fast Attack", Content = state and "Ativado" or "Desativado" })
    end
})

-- ===== AUTO FARM LEVEL AVANÇADO =====
local AutoFarmEnabled = false
local CurrentQuest = nil
local TargetNPCs = {}

local function GetQuest()
    local MyLevel = LocalPlayer.Data.Level.Value
    for _, quest in pairs(workspace.Quests:GetChildren()) do
        if MyLevel >= quest.MinLevel.Value and MyLevel <= quest.MaxLevel.Value then
            CurrentQuest = quest
            pcall(function()
                game:GetService("ReplicatedStorage").Remotes.StartQuest:FireServer(quest.Name)
            end)
            return
        end
    end
end

local function FindQuestNPCs()
    if not CurrentQuest then return end
    local npcs = {}
    for _, npc in pairs(workspace.NPCs:GetChildren()) do
        if npc.Name == CurrentQuest.NPCName.Value and npc:FindFirstChild("Humanoid") then
            table.insert(npcs, npc)
        end
    end
    TargetNPCs = npcs
end

local function TeleportToNPC(npc)
    if npc and npc:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame + Vector3.new(0,3,0)
    end
end

RunService.Heartbeat:Connect(function()
    if AutoFarmEnabled then
        -- Pega missão se não houver
        if not CurrentQuest then
            GetQuest()
            task.wait(1)
        end
        -- Atualiza NPCs da missão
        FindQuestNPCs()
        -- Ataca NPCs
        for _, npc in pairs(TargetNPCs) do
            if npc and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                TeleportToNPC(npc)
                pcall(function()
                    game:GetService("ReplicatedStorage").Remotes.Combat:FireServer(npc)
                end)
            end
        end
        -- Verifica se missão concluída
        if CurrentQuest and LocalPlayer.Data[CurrentQuest.Name .. "Progress"].Value >= CurrentQuest.RequiredAmount.Value then
            pcall(function()
                game:GetService("ReplicatedStorage").Remotes.FinishQuest:FireServer(CurrentQuest.Name)
            end)
            CurrentQuest = nil
            TargetNPCs = {}
            task.wait(2)
        end
    end
end)

Tabs.Farm:AddToggle("autoFarmAdvanced", {
    Title = "AutoFarm LEVEL",
    Description = "Faz farm completo de level automaticamente",
    Default = false,
    Callback = function(state)
        AutoFarmEnabled = state
        Fluent:Notify({ Title = "AutoFarm", Content = state and "Ativado" or "Desativado" })
    end
})

-- aba settings
Tabs.Settings:AddButton({ Title = "Resetar Posição", Callback = function()
    LocalPlayer.Character:BreakJoints()
    Fluent:Notify({ Title = "Resetado", Content = "Seu personagem foi resetado" })
end })

Tabs.Settings:AddToggle("fpsboost", {
    Title = "FPS Boost",
    Description = "Melhora desempenho desativando efeitos",
    Default = false,
    Callback = function(state)
        if state then
            setfpscap(60)
            game.Lighting.GlobalShadows = false
            game.Lighting.FogEnd = 9e9
            Fluent:Notify({ Title = "FPS Boost", Content = "Ativado" })
        else
            setfpscap(999)
            game.Lighting.GlobalShadows = true
            Fluent:Notify({ Title = "FPS Boost", Content = "Desativado" })
        end
    end
})

Tabs.Settings:AddButton({ Title = "Salvar Config", Callback = function()
    Fluent:SaveConfig("MinhaConfig")
    Fluent:Notify({ Title = "Configuração", Content = "Salva com sucesso!" })
end })

Tabs.Settings:AddButton({ Title = "Carregar Config", Callback = function()
    Fluent:LoadConfig("MinhaConfig")
    Fluent:Notify({ Title = "Configuração", Content = "Carregada com sucesso!" })
end })
