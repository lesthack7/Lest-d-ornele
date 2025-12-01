local TextChatService = game:GetService("TextChatService")
local UserInputService = game:GetService("UserInputService")
local player = game.Players.LocalPlayer

local comandos = { "/mat", "/render", "/fura pneu" }
local iniciais = { ["/mat"] = "M", ["/render"] = "R", ["/fura pneu"] = "F" }
local delays = { ["/mat"] = 0.2, ["/render"] = 0.2, ["/fura pneu"] = 0.2 }
local reps = { ["/mat"] = 1, ["/render"] = 1, ["/fura pneu"] = 1 }
local spamThreads, spamON = {}, {}
local mainGui = nil
local actionBtnsGui = nil

-- Função universal de arrasto (mobile e PC!)
function dragify(Frame)
    Frame.Active = true
    Frame.Selectable = true -- ESSENCIAL para mobile!
    local UIS = game:GetService("UserInputService")
    local dragging, dragInput, dragStart, startPos

    Frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = Frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)

    Frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            Frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

local function spammarComando(comando)
    spamON[comando] = true
    local delay = tonumber(delays[comando]) or 0.2
    if delay < 0.1 then delay = 0.1 end
    local repeticoes = tonumber(reps[comando]) or 1

    local channel = (TextChatService.TextChannels and TextChatService.TextChannels:FindFirstChild("RBXGeneral"))
        or (TextChatService:FindFirstChild("TextChannels") and TextChatService:FindFirstChild("TextChannels"):FindFirstChild("RBXGeneral"))
        or TextChatService:FindFirstChild("RBXGeneral")
        or TextChatService:FindFirstChild("General")
        or TextChatService:FindFirstChild("All")

    spamThreads[comando] = task.spawn(function()
        for i = 1, repeticoes do
            if channel and spamON[comando] then
                channel:SendAsync(comando)
            end
            task.wait(delay)
        end
        spamON[comando] = false
    end)
end

local function pararSpam(comando)
    spamON[comando] = false
end

-- Botões M/R/F arrastáveis, só letras, circulares com borda preta
local function showActionButtons()
    if actionBtnsGui then actionBtnsGui:Destroy() end
    actionBtnsGui = Instance.new("ScreenGui")
    actionBtnsGui.Parent = player:WaitForChild("PlayerGui")
    actionBtnsGui.Name = "ActionBtns"

    local cores = {
        [1] = Color3.fromRGB(220,30,30),   -- M
        [2] = Color3.fromRGB(255,210,0),   -- R
        [3] = Color3.fromRGB(130,0,255),   -- F
    }

    for i,cmd in ipairs(comandos) do
        local letra = iniciais[cmd]
        local btn = Instance.new("TextButton")
        btn.Parent = actionBtnsGui
        btn.Size = UDim2.new(0,54,0,54)
        btn.Position = UDim2.new(0, 16 + (i-1)*70, 1, -120)
        btn.BackgroundColor3 = cores[i]
        btn.BackgroundTransparency = 0
        btn.Text = letra
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 36
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.Name = "Btn_"..cmd
        btn.Active = true
        btn.Selectable = true

        local corner = Instance.new("UICorner", btn)
        corner.CornerRadius = UDim.new(0.5,0)
        local stroke = Instance.new("UIStroke", btn)
        stroke.Color = Color3.fromRGB(0,0,0)
        stroke.Thickness = 3

        dragify(btn)
        btn.MouseButton1Click:Connect(function()
            btn.Text = "▶️"
            spammarComando(cmd)
            coroutine.wrap(function()
                while spamON[cmd] do
                    wait(0.1)
                end
                btn.Text = letra
            end)()
        end)
    end
end

local function hideActionButtons()
    if actionBtnsGui then actionBtnsGui:Destroy() actionBtnsGui = nil end
end

-- Menu Principal arrastável
local function createMainGui()
    local gui = Instance.new("ScreenGui")
    gui.Name = "Spammar"
    gui.Parent = player:WaitForChild("PlayerGui")
    mainGui = gui

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 225, 0, 155 + #comandos*58)
    frame.Position = UDim2.new(0.5, -112, 0.5, -((155 + #comandos*58)//2))
    frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
    frame.Active = true
    frame.Selectable = true
    dragify(frame)
    local uicorner = Instance.new("UICorner", frame) uicorner.CornerRadius = UDim.new(0.15,0)
    local uistroke = Instance.new("UIStroke", frame)
    uistroke.Color = Color3.fromRGB(255,255,255)
    uistroke.Thickness = 4

    -- Título
    local titlebox = Instance.new("Frame", frame)
    titlebox.Size = UDim2.new(0.7,0,0,32)
    titlebox.Position = UDim2.new(0.15,0,0,10)
    titlebox.BackgroundColor3 = Color3.fromRGB(0,0,0)
    titlebox.BackgroundTransparency = 0
    local tcorner = Instance.new("UICorner", titlebox) tcorner.CornerRadius = UDim.new(0.7,0)
    local tstroke = Instance.new("UIStroke", titlebox)
    tstroke.Color = Color3.fromRGB(200,200,200)
    tstroke.Thickness = 2
    local titulo = Instance.new("TextLabel", titlebox)
    titulo.Text = "🗼 LEST"
    titulo.Size = UDim2.new(1,0,1,0)
    titulo.Font = Enum.Font.GothamBold
    titulo.TextSize = 28
    titulo.TextColor3 = Color3.fromRGB(255,255,255)
    titulo.BackgroundTransparency = 1

    for i,cmd in ipairs(comandos) do
        local yBase = 52 + (i-1)*58

        local lbl = Instance.new("TextLabel", frame)
        lbl.Text = cmd
        lbl.Size = UDim2.new(0.7,0,0,22)
        lbl.Position = UDim2.new(0.05,0,0,yBase+6)
        lbl.BackgroundTransparency = 1
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 16
        lbl.TextColor3 = Color3.fromRGB(255,255,255)
        lbl.TextXAlignment = Enum.TextXAlignment.Left

        local delayBox = Instance.new("TextBox", frame)
        delayBox.Size = UDim2.new(0.4,0,0,19)
        delayBox.Position = UDim2.new(0.05,0,0,yBase+28)
        delayBox.Text = tostring(delays[cmd])
        delayBox.PlaceholderText = "⏱️ Delay"
        delayBox.Font = Enum.Font.GothamBold
        delayBox.TextSize = 13
        delayBox.BackgroundColor3 = Color3.fromRGB(24,0,66)
        delayBox.TextColor3 = Color3.fromRGB(255,255,255)
        local inputCorner1 = Instance.new("UICorner", delayBox)
        inputCorner1.CornerRadius = UDim.new(0.32,0)
        delayBox.FocusLost:Connect(function()
            delays[cmd] = tonumber(delayBox.Text) or 0.2
            if delays[cmd] < 0.1 then delays[cmd] = 0.1 end
            delayBox.Text = tostring(delays[cmd])
        end)

        local repsBox = Instance.new("TextBox", frame)
        repsBox.Size = UDim2.new(0.4,0,0,19)
        repsBox.Position = UDim2.new(0.55,0,0,yBase+28)
        repsBox.Text = tostring(reps[cmd])
        repsBox.PlaceholderText = "🔁 Repetições"
        repsBox.Font = Enum.Font.GothamBold
        repsBox.TextSize = 13
        repsBox.BackgroundColor3 = Color3.fromRGB(24,0,66)
        repsBox.TextColor3 = Color3.fromRGB(255,255,255)
        local inputCorner2 = Instance.new("UICorner", repsBox)
        inputCorner2.CornerRadius = UDim.new(0.32,0)
        repsBox.FocusLost:Connect(function()
            reps[cmd] = math.max(1, math.floor(tonumber(repsBox.Text) or 1))
            repsBox.Text = tostring(reps[cmd])
        end)

        local startBtn = Instance.new("TextButton", frame)
        startBtn.Text = "🚀 INICIAR"
        startBtn.Size = UDim2.new(0.4,0,0,18)
        startBtn.Position = UDim2.new(0.05,0,0,yBase+48)
        startBtn.BackgroundColor3 = Color3.fromRGB(40,130,40)
        startBtn.TextColor3 = Color3.fromRGB(255,255,255)
        startBtn.Font = Enum.Font.GothamBold
        startBtn.TextSize = 13
        local btnStartCorner = Instance.new("UICorner", startBtn)
        btnStartCorner.CornerRadius = UDim.new(0.4,0)
        startBtn.MouseButton1Click:Connect(function()
            pararSpam(cmd)
            spammarComando(cmd)
        end)

        local stopBtn = Instance.new("TextButton", frame)
        stopBtn.Text = "🛑 PARAR"
        stopBtn.Size = UDim2.new(0.4,0,0,18)
        stopBtn.Position = UDim2.new(0.55,0,0,yBase+48)
        stopBtn.BackgroundColor3 = Color3.fromRGB(150,40,140)
        stopBtn.TextColor3 = Color3.fromRGB(255,255,255)
        stopBtn.Font = Enum.Font.GothamBold
        stopBtn.TextSize = 13
        local btnStopCorner = Instance.new("UICorner", stopBtn)
        btnStopCorner.CornerRadius = UDim.new(0.4,0)
        stopBtn.MouseButton1Click:Connect(function() pararSpam(cmd) end)
    end

    -- Botões ativar/desativar
    local botaoAtivar = Instance.new("TextButton", frame)
    botaoAtivar.Text = "Ativar Botões"
    botaoAtivar.Size = UDim2.new(0.36,0,0,22)
    botaoAtivar.Position = UDim2.new(0.1,0,1,-32)
    botaoAtivar.BackgroundColor3 = Color3.fromRGB(80,80,180)
    botaoAtivar.TextColor3 = Color3.fromRGB(255,255,255)
    botaoAtivar.Font = Enum.Font.GothamBold
    botaoAtivar.TextSize = 12
    local bcorner = Instance.new("UICorner", botaoAtivar)
    bcorner.CornerRadius = UDim.new(0.5,0)
    botaoAtivar.MouseButton1Click:Connect(showActionButtons)

    local botaoDesativar = Instance.new("TextButton", frame)
    botaoDesativar.Text = "Desativar Botões"
    botaoDesativar.Size = UDim2.new(0.36,0,0,22)
    botaoDesativar.Position = UDim2.new(0.54,0,1,-32)
    botaoDesativar.BackgroundColor3 = Color3.fromRGB(150,20,60)
    botaoDesativar.TextColor3 = Color3.fromRGB(255,255,255)
    botaoDesativar.Font = Enum.Font.GothamBold
    botaoDesativar.TextSize = 12
    local dcorner = Instance.new("UICorner", botaoDesativar)
    dcorner.CornerRadius = UDim.new(0.5,0)
    botaoDesativar.MouseButton1Click:Connect(hideActionButtons)
end

-- Botão abrir/fechar menu (círculo preto, arrastável)
local function createMenuToggleBtn()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MenuToggleBtn"
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local btn = Instance.new("ImageButton", screenGui)
    btn.Size = UDim2.new(0, 44, 0, 44)
    btn.Position = UDim2.new(0, 16, 1, -56)
    btn.BackgroundColor3 = Color3.fromRGB(0,0,0)
    btn.BackgroundTransparency = 0
    btn.Name = "ToggleMenuBtn"
    btn.Image = "rbxassetid://11648237415"
    local uicorner = Instance.new("UICorner", btn)
    uicorner.CornerRadius = UDim.new(0.5,0)

    dragify(btn)

    btn.MouseButton1Click:Connect(function()
        if mainGui then
            mainGui.Enabled = not mainGui.Enabled
        else
            createMainGui()
            mainGui.Enabled = true
        end
    end)
end

createMainGui()
createMenuToggleBtn()
mainGui.Enabled = false
