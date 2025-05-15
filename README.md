--[[ AUTO COLETAR OTIMIZADO - ANTI LAG MELHORADO ]]

local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Enabled = false
local RANGE = 15

-- Proteção Anti-Kick
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local kick = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(method) == "Kick" then
            return warn("Tentativa de Kick Bloqueada.")
        end
        return kick(self, ...)
    end)
end)

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoColetarGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = (CoreGui or player:WaitForChild("PlayerGui"))

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0, 100, 0, 40)
Button.Position = UDim2.new(0, 20, 0, 100)
Button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.Text = "Auto Coletar [OFF]"
Button.Font = Enum.Font.SourceSansBold
Button.TextSize = 16
Button.Parent = ScreenGui
Button.Active = true
Button.Draggable = true

-- Anti Lag
local function AntiLag()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Explosion") then
            obj:Destroy()
        elseif obj:IsA("Decal") then
            obj.Transparency = 1
        end
    end
end
AntiLag()

-- Armazenar prompts e clicks para evitar varredura constante
local prompts = {}
local clicks = {}

-- Atualiza a lista apenas a cada 5 segundos (muito mais leve)
task.spawn(function()
    while true do
        prompts = {}
        clicks = {}
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("ProximityPrompt") then
                table.insert(prompts, v)
            elseif v:IsA("ClickDetector") then
                table.insert(clicks, v)
            end
        end
        task.wait(5)
    end
end)

-- Loop leve de execução
task.spawn(function()
    while true do
        task.wait(0.5) -- mais leve ainda
        if Enabled then
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local root = char.HumanoidRootPart
                for _, v in ipairs(prompts) do
                    if v:IsDescendantOf(workspace) and v.Enabled then
                        local ok, dist = pcall(function()
                            return (v.Parent.Position - root.Position).Magnitude
                        end)
                        if ok and dist <= v.MaxActivationDistance then
                            pcall(function()
                                fireproximityprompt(v)
                            end)
                        end
                    end
                end
                for _, c in ipairs(clicks) do
                    if c:IsDescendantOf(workspace) then
                        local ok, dist = pcall(function()
                            return (c.Parent.Position - root.Position).Magnitude
                        end)
                        if ok and dist <= RANGE then
                            pcall(function()
                                fireclickdetector(c)
                            end)
                        end
                    end
                end
            end
        end
    end
end)

-- Botão ativar/desativar
Button.MouseButton1Click:Connect(function()
    Enabled = not Enabled
    Button.Text = Enabled and "Auto Coletar [ON]" or "Auto Coletar [OFF]"
    Button.BackgroundColor3 = Enabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
end)
