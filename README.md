-- Painel estilo REDZ com ESP, Teleport e Modo Leve
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "PainelREDZ"

local MainButton = Instance.new("TextButton", ScreenGui)
MainButton.Size = UDim2.new(0, 100, 0, 30)
MainButton.Position = UDim2.new(0, 10, 0, 100)
MainButton.Text = "Abrir Painel"
MainButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainButton.TextColor3 = Color3.new(1, 1, 1)
MainButton.BorderSizePixel = 0

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 250)
Frame.Position = UDim2.new(0, 10, 0, 140)
Frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Frame.Visible = false
Frame.Active = true
Frame.Draggable = true
Frame.Selectable = true
Frame.BorderSizePixel = 0
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 6)

MainButton.MouseButton1Click:Connect(function()
    Frame.Visible = not Frame.Visible
end)

local abas = { "ESP", "Teleport", "Modo Leve" }
local teleportPopup = nil

-- ESP
local espEnabled = false
local function criarESP(player)
    local torso = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not torso then return end

    if torso:FindFirstChild("ESPHighlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ESPHighlight"
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.new(1, 0, 0)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = torso
    highlight.Adornee = player.Character

    local espGui = Instance.new("BillboardGui", torso)
    espGui.Name = "ESPInfo"
    espGui.Size = UDim2.new(0, 150, 0, 30)
    espGui.AlwaysOnTop = true
    espGui.StudsOffset = Vector3.new(0, 3, 0)

    local label = Instance.new("TextLabel", espGui)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 0, 0)
    label.TextStrokeTransparency = 0
    label.TextScaled = true
    label.Font = Enum.Font.SourceSansBold
    label.Text = player.Name .. " - ...m"

    coroutine.wrap(function()
        while espEnabled and player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
            local dist = (LocalPlayer.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
            label.Text = player.Name .. string.format(" - %.1f metros", dist)
            wait(0.3)
        end
        if torso:FindFirstChild("ESPHighlight") then
            torso:FindFirstChild("ESPHighlight"):Destroy()
        end
        if torso:FindFirstChild("ESPInfo") then
            torso:FindFirstChild("ESPInfo"):Destroy()
        end
    end)()
end

local function ativarESP()
    espEnabled = not espEnabled
    if espEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                criarESP(p)
            end
        end
        Players.PlayerAdded:Connect(function(p)
            if espEnabled then
                p.CharacterAdded:Connect(function()
                    wait(1)
                    criarESP(p)
                end)
            end
        end)
    end
end

-- TELEPORT
local function abrirTeleport()
    if teleportPopup and teleportPopup.Parent then
        teleportPopup:Destroy()
    end

    teleportPopup = Instance.new("Frame", ScreenGui)
    teleportPopup.Size = UDim2.new(0, 200, 0, 300)
    teleportPopup.Position = UDim2.new(0, 220, 0, 140)
    teleportPopup.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    teleportPopup.BorderSizePixel = 0
    teleportPopup.Active = true
    teleportPopup.Draggable = true
    Instance.new("UICorner", teleportPopup).CornerRadius = UDim.new(0, 6)

    local titulo = Instance.new("TextLabel", teleportPopup)
    titulo.Size = UDim2.new(1, 0, 0, 30)
    titulo.Text = "Teleport Jogadores"
    titulo.TextColor3 = Color3.new(1, 1, 1)
    titulo.BackgroundTransparency = 1
    titulo.Font = Enum.Font.SourceSansBold
    titulo.TextSize = 16

    local fecharBtn = Instance.new("TextButton", teleportPopup)
    fecharBtn.Size = UDim2.new(0, 25, 0, 25)
    fecharBtn.Position = UDim2.new(1, -30, 0, 2)
    fecharBtn.Text = "X"
    fecharBtn.TextColor3 = Color3.new(1, 0.3, 0.3)
    fecharBtn.BackgroundColor3 = Color3.fromRGB(40, 0, 0)
    fecharBtn.Font = Enum.Font.SourceSansBold
    fecharBtn.TextScaled = true
    fecharBtn.BorderSizePixel = 0
    fecharBtn.MouseButton1Click:Connect(function()
        teleportPopup:Destroy()
        teleportPopup = nil
    end)

    local lista = Instance.new("ScrollingFrame", teleportPopup)
    lista.Size = UDim2.new(1, 0, 1, -40)
    lista.Position = UDim2.new(0, 0, 0, 35)
    lista.CanvasSize = UDim2.new(0, 0, 0, 0)
    lista.BackgroundTransparency = 1
    lista.ScrollBarThickness = 4
    lista.Name = "ListaScroll"

    local layout = Instance.new("UIListLayout", lista)
    layout.Padding = UDim.new(0, 5)
    layout.SortOrder = Enum.SortOrder.LayoutOrder

    local function atualizarLista()
        for _, child in ipairs(lista:GetChildren()) do
            if child:IsA("TextButton") then
                child:Destroy()
            end
        end

        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                local btn = Instance.new("TextButton", lista)
                btn.Size = UDim2.new(1, -10, 0, 30)
                btn.Text = p.Name
                btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
                btn.TextColor3 = Color3.new(1, 1, 1)
                btn.BorderSizePixel = 0
                Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
                btn.MouseButton1Click:Connect(function()
                    local char = p.Character
                    if char and char:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                        LocalPlayer.Character.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame + Vector3.new(2, 0, 0)
                    end
                end)
            end
        end

        lista.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
    end

    atualizarLista()
    Players.PlayerAdded:Connect(atualizarLista)
    Players.PlayerRemoving:Connect(atualizarLista)
end

-- BOTÕES DO MENU
for i, nome in ipairs(abas) do
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Position = UDim2.new(0, 5, 0, (i - 1) * 35 + 5)
    btn.Text = nome
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)

    if nome == "ESP" then
        btn.MouseButton1Click:Connect(ativarESP)
    elseif nome == "Teleport" then
        btn.MouseButton1Click:Connect(abrirTeleport)
    elseif nome == "Modo Leve" then
        btn.MouseButton1Click:Connect(function()
            print("Modo leve ativado (sem função)")
        end)
    end
end
