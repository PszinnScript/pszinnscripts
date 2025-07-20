-- INÍCIO PSZINN SCRIPT ROLEPLAY

-- Proteção Anti-Detector Simples
if not game:IsLoaded() then game.Loaded:Wait() end
pcall(function()
    for _,gc in pairs(getgc(true)) do
        if type(gc)=="function" then
            hookfunction(gc, function() end)
        end
    end
end)

-- Serviços
local P = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local LP = P.LocalPlayer
local M = LP:GetMouse()

-- GUI Base
local sg = Instance.new("ScreenGui", game:GetService("CoreGui"))
sg.Name = "PszinnScript"
sg.ResetOnSpawn = false

-- Painel Inicial
local main = Instance.new("Frame", sg)
main.Size = UDim2.new(0,500,0,300)
main.Position = UDim2.new(0.5,-250,0.5,-150)
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.Active = true
main.Draggable = true

local title = Instance.new("TextLabel", main)
title.Text = "Pszinn Script Roleplay"
title.Size = UDim2.new(1,0,0,40)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundColor3 = Color3.fromRGB(50,50,50)

local btn = Instance.new("TextButton", main)
btn.Text = "Continuar"
btn.Size = UDim2.new(0,150,0,40)
btn.Position = UDim2.new(0,10,0,250)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 18
btn.TextColor3 = Color3.new(1,1,1)

spawn(function()
    while btn.Parent do
        for h=0,1,0.01 do
            btn.BackgroundColor3 = Color3.fromHSV(h,1,1)
            wait(0.03)
        end
    end
end)

-- Painel de Funções
local fpanel = Instance.new("Frame", sg)
fpanel.Size = main.Size
fpanel.Position = main.Position
fpanel.BackgroundColor3 = Color3.fromRGB(35,35,35)
fpanel.Visible = false

local back = Instance.new("TextButton", fpanel)
back.Text = "Voltar"
back.Size = UDim2.new(0,150,0,40)
back.Position = UDim2.new(0,10,1,-50)
back.Font = Enum.Font.GothamBold
back.TextSize = 18

-- Troca de painéis
btn.MouseButton1Click:Connect(function()
    main.Visible = false
    fpanel.Visible = true
end)
back.MouseButton1Click:Connect(function()
    fpanel.Visible = false
    main.Visible = true
end)

-- Funções da aba
local flyEnabled, aimbotEnabled, espEnabled = false, false, false
local flySpeed, aimPrec, walkSpeed = 50, 75, 16
local bodyVel

-- Criar slider genérico
local function addSlider(txt, y, min, max, initial, onChange)
    local lbl = Instance.new("TextLabel", fpanel)
    lbl.Text = txt..": "..initial
    lbl.Size = UDim2.new(1,-20,0,20)
    lbl.Position = UDim2.new(0,10,0,y)
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 16

    local bar = Instance.new("Frame", fpanel)
    bar.Size = UDim2.new(0.9,0,0,20)
    bar.Position = UDim2.new(0.05,0,0,y+25)
    bar.BackgroundColor3 = Color3.fromRGB(70,70,70)

    local fill = Instance.new("Frame", bar)
    fill.Size = UDim2.new((initial-min)/(max-min),0,1,0)
    fill.BackgroundColor3 = Color3.fromRGB(0,170,255)

    local dragging = false
    bar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
    end)
    bar.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    bar.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local x = i.Position.X - bar.AbsolutePosition.X
            fill.Size = UDim2.new(x/bar.AbsoluteSize.X,0,1,0)
            local v = min + (max-min)*(x/bar.AbsoluteSize.X)
            lbl.Text = txt..": "..math.floor(v)
            onChange(math.floor(v))
        end
    end)
    onChange(initial)
end

-- Checkboxes
local function addToggle(text, y, default, onToggle)
    local t = Instance.new("TextButton", fpanel)
    t.Size = UDim2.new(0,200,0,30)
    t.Position = UDim2.new(0,10,0,y)
    t.Text = (default and "[x] " or "[ ] ")..text
    t.Font = Enum.Font.SourceSansBold
    t.TextSize = 16
    t.TextColor3 = Color3.new(1,1,1)
    t.BackgroundColor3 = Color3.fromRGB(70,70,70)

    t.MouseButton1Click:Connect(function()
        default = not default
        t.Text = (default and "[x] " or "[ ] ")..text
        onToggle(default)
    end)

    onToggle(default)
end

-- Adiciona controles
local y=10
addSlider("Fly Speed", y, 10, 150, flySpeed, function(v) flySpeed=v end); y=y+60
addSlider("Aimbot %", y, 0, 100, aimPrec, function(v) aimPrec=v end); y=y+60
addSlider("Speed", y, 16, 200, walkSpeed, function(v) walkSpeed=v end); y=y+60
addToggle("Fly (F)", y, flyEnabled, function(v) flyEnabled=v end); y=y+40
addToggle("Aimbot (K)", y, aimbotEnabled, function(v) aimbotEnabled=v end); y=y+40
addToggle("ESP (J)", y, espEnabled, function(v) espEnabled=v end)

-- Teclas
UIS.InputBegan:Connect(function(inp)
    if inp.KeyCode == Enum.KeyCode.F then flyEnabled = not flyEnabled end
    if inp.KeyCode == Enum.KeyCode.K then aimbotEnabled = not aimbotEnabled end
    if inp.KeyCode == Enum.KeyCode.J then espEnabled = not espEnabled end
end)

-- Lógica principal
RS.RenderStepped:Connect(function()
    -- Fly
    if flyEnabled then
        if not bodyVel then
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                bodyVel = Instance.new("BodyVelocity", hrp)
                bodyVel.MaxForce = Vector3.new(1e5,1e5,1e5)
            end
        end
        local dir = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then dir += workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= workspace.CurrentCamera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then dir += workspace.CurrentCamera.CFrame.RightVector end
        bodyVel.Velocity = dir.Unit * flySpeed
    elseif bodyVel then bodyVel:Destroy(); bodyVel=nil end

    -- Speed
    if LP.Character and LP.Character:FindFirstChild("Humanoid") then
        LP.Character.Humanoid.WalkSpeed = walkSpeed
    end

    -- Aimbot
    if aimbotEnabled then
        local tx, d = nil, math.huge
        for _,pl in ipairs(P:GetPlayers()) do
            if pl~=LP and pl.Character and pl.Character:FindFirstChild("Head") then
                local sp, onS = workspace.CurrentCamera:WorldToViewportPoint(pl.Character.Head.Position)
                local dist = (Vector2.new(M.X,M.Y) - Vector2.new(sp.X,sp.Y)).Magnitude
                if onS and dist<d then d, tx = dist, pl end
            end
        end
        if tx and d<300 and math.random(0,100)<=aimPrec then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, tx.Character.Head.Position)
        end
    end

    -- ESP
    for _,pl in pairs(P:GetPlayers()) do
        if pl~=LP and pl.Character and espEnabled then
            if not pl.Character:FindFirstChild("ESPBox") then
                local box = Instance.new("BoxHandleAdornment", pl.Character)
                box.Name="ESPBox"
                box.Adornee = pl.Character:FindFirstChild("HumanoidRootPart")
                box.AlwaysOnTop = true
                box.Size = Vector3.new(3,6,1)
                box.Color3 = Color3.fromRGB(255,0,0)
                box.Transparency = 0.5
            end
        elseif pl.Character and pl.Character:FindFirstChild("ESPBox") then
            pl.Character.ESPBox:Destroy()
        end
    end
end)
-- Fim PSZINN SCRIPT ROLEPLAY
