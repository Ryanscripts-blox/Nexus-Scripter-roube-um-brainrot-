-- 🌌 Noxe Scripts - GUI Bonita + Save/Go + God Mode + Noclip + Speed + Infinity Jump

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Variáveis
local savedPosition = nil
local isMoving = false
local godMode = false
local healConn = nil
local noclip = false
local noclipConn = nil
local speedOn = false
local speedValue = 1 -- velocidade ajustável
local infJump = false
local infJumpConn = nil

-- Criar GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NoxeScriptsGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Fundo principal
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 300, 0, 320) -- altura aumentada para caber tudo
main.Position = UDim2.new(0.35, 0, 0.3, 0)
main.BackgroundColor3 = Color3.fromRGB(30, 28, 44)
main.Parent = screenGui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 16)

local uiStroke = Instance.new("UIStroke", main)
uiStroke.Color = Color3.fromRGB(95, 80, 200)
uiStroke.Thickness = 2

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "🌌 Noxe Scripts"
title.TextColor3 = Color3.fromRGB(230, 230, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.Parent = main

-- Função para criar botões
local function makeButton(text, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.8, 0, 0, 32)
    btn.Position = UDim2.new(0.1, 0, posY, 0)
    btn.BackgroundColor3 = Color3.fromRGB(70, 60, 130)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 15
    btn.TextColor3 = Color3.fromRGB(245, 245, 255)
    btn.Parent = main
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    return btn
end

-- Botões
local saveBtn   = makeButton("💾 Save Position", 0.15)
local goBtn     = makeButton("⚡ Go To Position", 0.28)
local godBtn    = makeButton("🛡️ God Mode: OFF", 0.41)
local noclipBtn = makeButton("🚪 Noclip: OFF", 0.54)
local speedBtn  = makeButton("🏃 Speed: OFF", 0.67)
local infJumpBtn= makeButton("🦘 Infinity Jump: OFF", 0.80)

-- Dragging do menu
local dragging, dragInput, dragStart, startPos
main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
main.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Salvar posição
saveBtn.MouseButton1Click:Connect(function()
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    savedPosition = hrp.Position
    saveBtn.Text = "✅ Saved!"
    task.delay(1.2, function() saveBtn.Text = "💾 Save Position" end)
end)

-- Ir até posição
goBtn.MouseButton1Click:Connect(function()
    if isMoving or not savedPosition then return end
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    isMoving = true
    goBtn.Text = "⚡ Moving..."
    local target = savedPosition + Vector3.new(0, 4, 0)
    local dist = (hrp.Position - target).Magnitude
    local time = math.clamp(dist / 160, 0.12, 4)
    local tween = TweenService:Create(hrp, TweenInfo.new(time, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = CFrame.new(target)})
    tween:Play()
    tween.Completed:Wait()
    isMoving = false
    goBtn.Text = "⚡ Go To Position"
end)

-- God Mode
local function setGod(on)
    if healConn then healConn:Disconnect(); healConn = nil end
    if on then
        healConn = RunService.Heartbeat:Connect(function()
            local char = player.Character
            if char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health < hum.MaxHealth then
                    hum.Health = hum.MaxHealth
                end
            end
        end)
    end
end

godBtn.MouseButton1Click:Connect(function()
    godMode = not godMode
    godBtn.Text = godMode and "🛡️ God Mode: ON" or "🛡️ God Mode: OFF"
    setGod(godMode)
end)

-- Noclip
local function setNoclip(on)
    if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
    if on then
        noclipConn = RunService.Stepped:Connect(function()
            local char = player.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end
end

noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipBtn.Text = noclip and "🚪 Noclip: ON" or "🚪 Noclip: OFF"
    setNoclip(noclip)
end)

-- Speed (CFrame mover)
local speedConn
local function setSpeed(on)
    if speedConn then speedConn:Disconnect(); speedConn = nil end
    if on then
        speedConn = RunService.Heartbeat:Connect(function()
            local char = player.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.MoveDirection.Magnitude > 0 then
                hrp.CFrame = hrp.CFrame + hum.MoveDirection * speedValue
            end
        end)
    end
end

speedBtn.MouseButton1Click:Connect(function()
    speedOn = not speedOn
    speedBtn.Text = speedOn and "🏃 Speed: ON" or "🏃 Speed: OFF"
    setSpeed(speedOn)
end)

-- Infinity Jump
local function setInfJump(on)
    if infJumpConn then infJumpConn:Disconnect(); infJumpConn = nil end
    if on then
        infJumpConn = UserInputService.JumpRequest:Connect(function()
            local char = player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then
                hum:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    end
end

infJumpBtn.MouseButton1Click:Connect(function()
    infJump = not infJump
    infJumpBtn.Text = infJump and "🦘 Infinity Jump: ON" or "🦘 Infinity Jump: OFF"
    setInfJump(infJump)
end)
