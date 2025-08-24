-- Hix Hub - Script completo (sem WalkSpeed slider)
-- Evitar múltiplas execuções
if _G.HixHubLoaded then return end
_G.HixHubLoaded = true

-- Proteção para exploit (se houver)
if syn and syn.protect_gui then
    syn.protect_gui(gethui())
end

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ContextActionService = game:GetService("ContextActionService")
local UserInputService = game:GetService("UserInputService")

-- Criar ScreenGui e parent robusto
local gui = Instance.new("ScreenGui")
gui.Name = "HixHubGui"
gui.ResetOnSpawn = false
local function setGuiParent(g)
    local ok, env = pcall(function() return gethui and gethui() end)
    if ok and env then
        g.Parent = env
        return
    end
    if LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui") then
        g.Parent = LocalPlayer:FindFirstChild("PlayerGui")
        return
    end
    pcall(function() g.Parent = CoreGui end)
end
setGuiParent(gui)

-- Cores
local lightBlue = Color3.fromRGB(0, 170, 255)   -- texto / borda ESP
local darkBlue = Color3.fromRGB(0, 0, 139)      -- fundo do botão (uso geral)

local function createBlueGlow(instance)
    local stroke = Instance.new("UIStroke")
    stroke.Color = darkBlue
    stroke.Thickness = 3
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Transparency = 0
    stroke.Parent = instance
end

-- BOTÃO H
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0, 100, 0, 100)
toggleButton.BackgroundColor3 = lightBlue
toggleButton.BorderSizePixel = 0
toggleButton.Text = "H"
toggleButton.TextColor3 = darkBlue
toggleButton.TextSize = 20
toggleButton.Font = Enum.Font.GothamBold
toggleButton.ClipsDescendants = true
toggleButton.AutoButtonColor = true
toggleButton.ZIndex = 10
toggleButton.AnchorPoint = Vector2.new(0.5, 0.5)
toggleButton.BackgroundTransparency = 0
local roundifyButton = Instance.new("UICorner", toggleButton)
roundifyButton.CornerRadius = UDim.new(1, 0)
createBlueGlow(toggleButton)
toggleButton.Parent = gui

-- MAIN GUI
local mainGui = Instance.new("Frame")
mainGui.Name = "MainGui"
mainGui.Size = UDim2.new(0, 300, 0, 520)
mainGui.Position = UDim2.new(0.5, -150, 0.5, -260)
mainGui.BackgroundColor3 = lightBlue
mainGui.BorderSizePixel = 0
mainGui.Visible = false
mainGui.Active = true
mainGui.ZIndex = 1
local roundifyMain = Instance.new("UICorner", mainGui)
roundifyMain.CornerRadius = UDim.new(0, 10)
createBlueGlow(mainGui)
mainGui.Parent = gui

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Parent = mainGui
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.Position = UDim2.new(0,0,0,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Hix hub"
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 24
titleLabel.TextColor3 = darkBlue
titleLabel.TextStrokeTransparency = 0.8

-- Scroll
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Name = "Content"
scrollFrame.Parent = mainGui
scrollFrame.Size = UDim2.new(1, -10, 1, -50)
scrollFrame.Position = UDim2.new(0, 5, 0, 45)
scrollFrame.CanvasSize = UDim2.new(0, 0, 3, 0)
scrollFrame.ScrollBarThickness = 8
scrollFrame.BackgroundTransparency = 1
scrollFrame.BorderSizePixel = 0
local layout = Instance.new("UIListLayout", scrollFrame)
layout.Padding = UDim.new(0, 5)

-- pasta para objetos ESP (highlights, billboards etc)
local espFolder = Instance.new("Folder")
espFolder.Name = "ESP_Objects"
espFolder.Parent = CoreGui

-- === BOTÃO ESP PLAYERS (colocado primeiro) ===
local espEnabled = false
local espConnections = {
    playerAdded = nil,
    playerRemoving = nil,
    charAdded = {}  -- map player -> conn
}

-- Usa Highlight para criar apenas a borda (outline) ao redor do Model inteiro
-- e cria um BillboardGui sempre-on-top ancorado ao HumanoidRootPart (ou fallback)
local function createESP(char)
    if not char then return end
    local espName = (char.Name or tostring(char)) .. "_ESP"
    if espFolder:FindFirstChild(espName) then return end

    -- 1) Highlight (borda)
    pcall(function()
        local h = Instance.new("Highlight")
        h.Name = espName
        h.Parent = espFolder
        h.Adornee = char
        h.FillTransparency = 1
        h.OutlineTransparency = 0
        h.OutlineColor = lightBlue
        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.Enabled = true
    end)

    -- 2) BillboardGui (fallback visível mesmo com partes transparentes)
    local ok, anchorPart = pcall(function()
        return char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart or char:FindFirstChildWhichIsA("BasePart")
    end)
    anchorPart = (ok and anchorPart) and anchorPart or nil

    if anchorPart then
        pcall(function()
            local bbName = espName .. "_BILLBOARD"
            local bb = Instance.new("BillboardGui")
            bb.Name = bbName
            bb.Parent = espFolder
            bb.Adornee = anchorPart
            bb.AlwaysOnTop = true
            bb.Size = UDim2.new(0, 160, 0, 30)
            bb.StudsOffset = Vector3.new(0, 2.5, 0)
            bb.MaxDistance = 1000

            local label = Instance.new("TextLabel", bb)
            label.Name = "ESPNameLabel"
            label.Size = UDim2.new(1, 0, 1, 0)
            label.BackgroundTransparency = 0.6
            label.BackgroundColor3 = Color3.fromRGB(0,0,0)
            label.TextColor3 = Color3.new(1,1,1)
            label.Font = Enum.Font.GothamBold
            label.TextSize = 18
            label.Text = (char.Name or "Player")
            label.TextStrokeTransparency = 0.6
            label.TextScaled = true
            label.TextWrapped = false
        end)
    else
        -- Se realmente não houver parte para ancorar (casos extremos), não criamos billboard.
    end
end

local function removeESPByChar(char)
    if not char then return end
    local baseName = (char.Name or tostring(char)) .. "_ESP"
    local h = espFolder:FindFirstChild(baseName)
    if h then pcall(function() h:Destroy() end) end
    local bb = espFolder:FindFirstChild(baseName .. "_BILLBOARD")
    if bb then pcall(function() bb:Destroy() end) end
end

local function removeESPByPlayerName(name)
    local espName = name .. "_ESP"
    local obj = espFolder:FindFirstChild(espName)
    if obj then pcall(function() obj:Destroy() end) end
    local bb = espFolder:FindFirstChild(espName .. "_BILLBOARD")
    if bb then pcall(function() bb:Destroy() end) end
end

local function enableESP()
    -- criar para jogadores já existentes (exceto localplayer)
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character then
            createESP(pl.Character)
        end
        -- conectar CharacterAdded para cada jogador atual
        if pl ~= LocalPlayer then
            local conn = pl.CharacterAdded:Connect(function(char)
                task.wait(0.8)
                if espEnabled then createESP(char) end
            end)
            espConnections.charAdded[pl] = conn
        end
    end

    -- PlayerAdded -> conectar CharacterAdded do novo jogador
    espConnections.playerAdded = Players.PlayerAdded:Connect(function(pl)
        if pl == LocalPlayer then return end
        espConnections.charAdded[pl] = pl.CharacterAdded:Connect(function(char)
            task.wait(0.8)
            if espEnabled then createESP(char) end
        end)
        -- se o jogador já tiver character
        if pl.Character and espEnabled then
            task.wait(0.8)
            createESP(pl.Character)
        end
    end)

    -- PlayerRemoving -> remover ESP e desconectar
    espConnections.playerRemoving = Players.PlayerRemoving:Connect(function(pl)
        removeESPByPlayerName(pl.Name)
        if espConnections.charAdded[pl] then
            espConnections.charAdded[pl]:Disconnect()
            espConnections.charAdded[pl] = nil
        end
    end)
end

local function disableESP()
    -- desconectar tudo
    if espConnections.playerAdded then
        espConnections.playerAdded:Disconnect()
        espConnections.playerAdded = nil
    end
    if espConnections.playerRemoving then
        espConnections.playerRemoving:Disconnect()
        espConnections.playerRemoving = nil
    end
    for pl, conn in pairs(espConnections.charAdded) do
        if conn then conn:Disconnect() end
    end
    espConnections.charAdded = {}
    -- remover apenas os objetos criados para players (mantendo outros itens na pasta)
    for _, child in ipairs(espFolder:GetChildren()) do
        if child:IsA("Highlight") and tostring(child.Name):find("_ESP$") then
            pcall(function() child:Destroy() end)
        elseif child:IsA("BillboardGui") and tostring(child.Name):find("_ESP_BILLBOARD$") then
            pcall(function() child:Destroy() end)
        end
    end
end

local espButton = Instance.new("TextButton")
espButton.Name = "ESPButton"
espButton.Parent = scrollFrame
espButton.Size = UDim2.new(1, -10, 0, 40)
espButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- vermelho (OFF)
espButton.Text = "ESP players/OFF"
espButton.TextColor3 = Color3.new(1, 1, 1) -- texto branco
espButton.Font = Enum.Font.GothamBold
espButton.TextSize = 18
espButton.AutoButtonColor = false
espButton.LayoutOrder = 0
Instance.new("UICorner", espButton).CornerRadius = UDim.new(0,8)

espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        espButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- verde (ON)
        espButton.Text = "ESP players/ON"
        espButton.TextColor3 = Color3.new(1,1,1) -- branco
        -- ativar
        enableESP()
        pcall(function() game:GetService("StarterGui"):SetCore("SendNotification", {Title = "⚠ Hix Hub", Text = "ESP ativado!", Duration = 3}) end)
    else
        espButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- vermelho (OFF)
        espButton.Text = "ESP players/OFF"
        espButton.TextColor3 = Color3.new(1,1,1) -- branco
        -- desativar
        disableESP()
        pcall(function() game:GetService("StarterGui"):SetCore("SendNotification", {Title = "⚠ Hix Hub", Text = "ESP desativado!", Duration = 3}) end)
    end
end)

-- === BOTÕES BÁSICOS ===
-- Follow
local rapeEnabled = false
local rapeButton = Instance.new("TextButton")
rapeButton.Name = "RapeButton"
rapeButton.Parent = scrollFrame
rapeButton.Size = UDim2.new(1, -10, 0, 40)
rapeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
rapeButton.Text = "Follow player/OFF"
rapeButton.TextColor3 = Color3.new(1, 1, 1)
rapeButton.Font = Enum.Font.GothamBold
rapeButton.TextSize = 18
rapeButton.AutoButtonColor = false
Instance.new("UICorner", rapeButton).CornerRadius = UDim.new(0,8)

-- Target textbox
local targetBox = Instance.new("TextBox")
targetBox.Name = "TargetBox"
targetBox.Parent = scrollFrame
targetBox.Size = UDim2.new(1, -10, 0, 40)
targetBox.PlaceholderText = "Nome do alvo"
targetBox.Text = ""
targetBox.TextColor3 = Color3.new(1, 1, 1)
targetBox.Font = Enum.Font.GothamBold
targetBox.TextSize = 18
targetBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Instance.new("UICorner", targetBox).CornerRadius = UDim.new(0,8)

-- Auto Click
local autoClickEnabled = false
local autoClickButton = Instance.new("TextButton")
autoClickButton.Name = "AutoClickButton"
autoClickButton.Parent = scrollFrame
autoClickButton.Size = UDim2.new(1, -10, 0, 40)
autoClickButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
autoClickButton.Text = "Auto Click/OFF"
autoClickButton.TextColor3 = Color3.new(1, 1, 1)
autoClickButton.Font = Enum.Font.GothamBold
autoClickButton.TextSize = 18
autoClickButton.AutoButtonColor = false
Instance.new("UICorner", autoClickButton).CornerRadius = UDim.new(0,8)

-- Anti Ragdoll
local antiRagdollEnabled = false
local antiRagdollButton = Instance.new("TextButton")
antiRagdollButton.Name = "AntiRagdollButton"
antiRagdollButton.Parent = scrollFrame
antiRagdollButton.Size = UDim2.new(1, -10, 0, 40)
antiRagdollButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
antiRagdollButton.Text = "Anti Ragdoll/OFF"
antiRagdollButton.TextColor3 = Color3.new(1, 1, 1)
antiRagdollButton.Font = Enum.Font.GothamBold
antiRagdollButton.TextSize = 18
antiRagdollButton.AutoButtonColor = false
Instance.new("UICorner", antiRagdollButton).CornerRadius = UDim.new(0,8)

-- Funções utilitárias
local function notify(msg)
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "⚠ Hix Hub",
            Text = msg,
            Duration = 3
        })
    end)
end

-- draggable genérico (toggleButton e mainGui)
local function makeDraggable(element)
    local dragging = false
    local dragConn
    element.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            local startPos = element.Position
            local startMouse = input.Position
            dragConn = UserInputService.InputChanged:Connect(function(i)
                if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                    local delta = i.Position - startMouse
                    element.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                end
            end)
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    if dragConn then dragConn:Disconnect() dragConn = nil end
                end
            end)
        end
    end)
end

makeDraggable(toggleButton)
makeDraggable(mainGui)

-- === ESP BEST BRAINROT (novo botão) ===
local espBrainrotEnabled = false
local espBrainrotHighlight = nil
local espBrainrotBillboard = nil

local espBrainrotButton = Instance.new("TextButton")
espBrainrotButton.Name = "ESPBrainrotButton"
espBrainrotButton.Parent = scrollFrame
espBrainrotButton.Size = UDim2.new(1, -10, 0, 40)
espBrainrotButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- vermelho desligado
espBrainrotButton.Text = "ESP best brainrot/OFF"
espBrainrotButton.TextColor3 = Color3.new(1, 1, 1) -- branco
espBrainrotButton.Font = Enum.Font.GothamBold
espBrainrotButton.TextSize = 18
espBrainrotButton.AutoButtonColor = false
Instance.new("UICorner", espBrainrotButton).CornerRadius = UDim.new(0,8)

-- Tabela de raridade (ordem)
local rarityOrder = {
    ["Common"] = 1,
    ["Rare"] = 2,
    ["Epic"] = 3,
    ["Legendary"] = 4,
    ["Mythic"] = 5,
    ["Brainrot God"] = 6,
    ["Secret"] = 7,
}

-- helpers para detectar raridade a partir de atributo ou do nome
local function detectRarityFromObject(obj)
    if not obj then return nil end
    local attr = obj:GetAttribute("Rarity")
    if attr and type(attr) == "string" and rarityOrder[attr] then
        return attr
    end
    -- se não tem atributo, tentar inferir do nome (case-insensitive)
    local name = tostring(obj.Name or "")
    local lower = string.lower(name)
    for rarityName, _ in pairs(rarityOrder) do
        if string.find(lower, string.lower(rarityName)) then
            return rarityName
        end
    end
    return nil
end

-- tenta extrair valor de dinheiro por segundo de várias formas
local function extractMoneyPerSecond(obj)
    if not obj then return nil end
    -- 1) atributo
    local attr = obj:GetAttribute("MoneyPerSecond") or obj:GetAttribute("DPS") or obj:GetAttribute("Money")
    if attr and type(attr) == "number" then return attr end
    if attr and type(attr) == "string" then
        local n = tonumber((attr:gsub("[^%d%.]", "")))
        if n then return n end
    end
    -- 2) Value child (NumberValue, IntValue)
    local nv = obj:FindFirstChild("MoneyPerSecond") or obj:FindFirstChild("DPS") or obj:FindFirstChild("Value")
    if nv and (nv:IsA("NumberValue") or nv:IsA("IntValue")) then
        return nv.Value
    end
    -- 3) procurar dentro de descendants por NumberValue ou IntValue com nomes comuns
    for _, v in ipairs(obj:GetDescendants()) do
        if v:IsA("NumberValue") or v:IsA("IntValue") then
            local nname = v.Name:lower()
            if nname:find("money") or nname:find("per") or nname:find("dps") or nname:find("value") then
                return v.Value
            end
        end
    end
    return nil
end

-- formata ex: 500 -> R$500s, 100000 -> R$100Ks, 1500000 -> R$1Ms
local function formatMoneyForLabel(v)
    if type(v) ~= "number" then return tostring(v) end
    if v >= 1e6 then
        local m = math.floor(v / 1e6)
        return "R$"..tostring(m).."Ms"
    elseif v >= 1e3 then
        local k = math.floor(v / 1e3)
        return "R$"..tostring(k).."Ks"
    else
        return "R$"..tostring(math.floor(v)).."s"
    end
end

-- mínimo de money/s para considerar (acima de R$1s)
local minBrainrotMoney = 1

-- Função para encontrar o brainrot mais raro (usa raridade detectada)
-- Agora filtra apenas objetos com money per second > minBrainrotMoney
local function getBestBrainrot()
    local folder = workspace:FindFirstChild("Brainrots") or workspace
    local best = nil
    local bestRank = -math.huge

    for _, obj in ipairs(folder:GetDescendants()) do
        if obj:IsA("Model") or obj:IsA("BasePart") then
            local money = extractMoneyPerSecond(obj) or 0
            if money and type(money) == "number" and money > minBrainrotMoney then
                local rarity = detectRarityFromObject(obj)
                if rarity and rarityOrder[rarity] then
                    local rank = rarityOrder[rarity]
                    if rank > bestRank then
                        best = obj
                        bestRank = rank
                    elseif rank == bestRank and best then
                        -- se houver empate, escolher pelo maior money per second
                        local curMoney = money
                        local bestMoney = extractMoneyPerSecond(best) or 0
                        if curMoney > bestMoney then
                            best = obj
                            bestRank = rank
                        end
                    end
                end
            end
        end
    end

    return best
end

-- cria highlight azul claro + billboard com nome e dinheiro/s
local function highlightBrainrot(obj)
    if not obj then return end
    -- limpar anteriores
    if espBrainrotHighlight then
        pcall(function() espBrainrotHighlight:Destroy() end)
        espBrainrotHighlight = nil
    end
    if espBrainrotBillboard then
        pcall(function() espBrainrotBillboard:Destroy() end)
        espBrainrotBillboard = nil
    end

    -- highlight (borda azul claro)
    local ok, h = pcall(function()
        local hi = Instance.new("Highlight")
        hi.Name = "BestBrainrot_Highlight"
        hi.Parent = espFolder
        hi.Adornee = obj
        hi.FillTransparency = 1 -- só outline
        hi.OutlineTransparency = 0
        hi.OutlineColor = lightBlue
        hi.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hi.Enabled = true
        return hi
    end)
    if ok then espBrainrotHighlight = h end

    -- buscar uma BasePart para adorar o BillboardGui
    local anchorPart = nil
    if obj:IsA("BasePart") then
        anchorPart = obj
    else
        anchorPart = obj:FindFirstChildWhichIsA("BasePart") or obj.PrimaryPart
        if not anchorPart then
            -- tentar encontrar qualquer BasePart nos descendants
            for _, d in ipairs(obj:GetDescendants()) do
                if d:IsA("BasePart") then
                    anchorPart = d
                    break
                end
            end
        end
    end

    if anchorPart then
        local bb = Instance.new("BillboardGui")
        bb.Name = "BestBrainrot_Billboard"
        bb.Parent = espFolder
        bb.AlwaysOnTop = true
        bb.Adornee = anchorPart
        bb.Size = UDim2.new(0, 220, 0, 40)
        bb.StudsOffset = Vector3.new(0, 3.5, 0)
        bb.MaxDistance = 1000

        local label = Instance.new("TextLabel")
        label.Name = "InfoLabel"
        label.Parent = bb
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.new(1,1,1)
        label.TextStrokeTransparency = 0
        label.Font = Enum.Font.GothamBold
        label.TextScaled = true
        label.TextWrapped = true

        -- pegar infos
        local bname = tostring(obj.Name or "Brainrot")
        local money = extractMoneyPerSecond(obj)
        local moneyLabel = money and formatMoneyForLabel(money) or "R$?s"

        label.Text = bname .. " (" .. moneyLabel .. ")"

        espBrainrotBillboard = bb
    else
        -- sem parte para adorar, apenas notificar (highlight já criado)
    end
end

-- limpar brainrot highlight e billboard
local function clearBrainrotHighlight()
    if espBrainrotHighlight then
        pcall(function() espBrainrotHighlight:Destroy() end)
        espBrainrotHighlight = nil
    end
    if espBrainrotBillboard then
        pcall(function() espBrainrotBillboard:Destroy() end)
        espBrainrotBillboard = nil
    end
end

-- Toggle botão behavior
espBrainrotButton.MouseButton1Click:Connect(function()
    espBrainrotEnabled = not espBrainrotEnabled
    if espBrainrotEnabled then
        espBrainrotButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        espBrainrotButton.Text = "ESP best brainrot/ON"
        espBrainrotButton.TextColor3 = Color3.new(1,1,1)
        notify("ESP best brainrot ativado!")

        local best = getBestBrainrot()
        if best then
            highlightBrainrot(best)
            local r = detectRarityFromObject(best) or "Unknown"
            local m = extractMoneyPerSecond(best)
            notify("Brainrot destacado: ".. tostring(best.Name) .. " (" .. tostring(r) .. ")")
            -- mostra dinheiro no label também (já feito pelo billboard)
        else
            notify("Nenhum brainrot encontrado acima de R$1s!")
        end
    else
        espBrainrotButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        espBrainrotButton.Text = "ESP best brainrot/OFF"
        espBrainrotButton.TextColor3 = Color3.new(1,1,1)
        notify("ESP best brainrot desativado!")
        clearBrainrotHighlight()
    end
end)

-- === Ragdoll / Freeze utilities ===
local defaultWalkSpeed = 16
local defaultJumpPower = 50
local defaultJumpHeight = 7.2
local ragdollActive = false
local freezeActionBound = false

local function blockControls()
    if freezeActionBound then return end
    local actions = {
        Enum.PlayerActions.CharacterForward,
        Enum.PlayerActions.CharacterBackward,
        Enum.PlayerActions.CharacterLeft,
        Enum.PlayerActions.CharacterRight,
        Enum.PlayerActions.CharacterJump
    }
    ContextActionService:BindAction("__HixHubBlockMove", function()
        return Enum.ContextActionResult.Sink
    end, false, unpack(actions))
    freezeActionBound = true
end

local function unblockControls()
    if not freezeActionBound then return end
    ContextActionService:UnbindAction("__HixHubBlockMove")
    freezeActionBound = false
end

local function freezeHumanoid(humanoid)
    local hrp = humanoid.Parent and humanoid.Parent:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.AssemblyLinearVelocity = Vector3.new()
        hrp.AssemblyAngularVelocity = Vector3.new()
    end
    humanoid.WalkSpeed = 0
    if humanoid.UseJumpPower then
        humanoid.JumpPower = 0
    else
        humanoid.JumpHeight = 0
    end
    blockControls()
end

local function restoreHumanoid(humanoid)
    humanoid.WalkSpeed = defaultWalkSpeed
    if humanoid.UseJumpPower then
        humanoid.JumpPower = defaultJumpPower
    else
        humanoid.JumpHeight = defaultJumpHeight
    end
    unblockControls()
end

local function captureDefaults(humanoid)
    defaultWalkSpeed = humanoid.WalkSpeed or defaultWalkSpeed
    if humanoid.UseJumpPower then
        defaultJumpPower = humanoid.JumpPower or defaultJumpPower
    else
        defaultJumpHeight = humanoid.JumpHeight or defaultJumpHeight
    end
end

-- Ragdoll hooks
local function hookHumanoid(humanoid)
    if not humanoid then return end
    captureDefaults(humanoid)

    humanoid.StateChanged:Connect(function(_, new)
        if not antiRagdollEnabled then return end
        if new == Enum.HumanoidStateType.Ragdoll
        or new == Enum.HumanoidStateType.FallingDown
        or new == Enum.HumanoidStateType.Physics then
            ragdollActive = true
            freezeHumanoid(humanoid)
        elseif new == Enum.HumanoidStateType.GettingUp
        or new == Enum.HumanoidStateType.Running
        or new == Enum.HumanoidStateType.RunningNoPhysics
        or new == Enum.HumanoidStateType.Landed then
            if ragdollActive then
                ragdollActive = false
                restoreHumanoid(humanoid)
            end
        end
    end)

    humanoid:GetPropertyChangedSignal("PlatformStand"):Connect(function()
        if not antiRagdollEnabled then return end
        if humanoid.PlatformStand then
            ragdollActive = true
            freezeHumanoid(humanoid)
        else
            if ragdollActive then
                ragdollActive = false
                restoreHumanoid(humanoid)
            end
        end
    end)
end

local function onCharacterAdded(char)
    local Hum = char:WaitForChild("Humanoid", 5)
    if Hum then
        hookHumanoid(Hum)
        RunService.Heartbeat:Connect(function()
            if antiRagdollEnabled and ragdollActive and Hum and Hum.Health > 0 then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.AssemblyLinearVelocity = Vector3.new()
                    hrp.AssemblyAngularVelocity = Vector3.new()
                end
                Hum.WalkSpeed = 0
                if Hum.UseJumpPower then
                    Hum.JumpPower = 0
                else
                    Hum.JumpHeight = 0
                end
            end
        end)
    end
end

if LocalPlayer.Character then onCharacterAdded(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

-- followStopDistance
local followStopDistance = 3

-- Loop Follow
RunService.Heartbeat:Connect(function()
    if not rapeEnabled then return end
    local char = LocalPlayer.Character
    if not char then return end
    local Hum = char:FindFirstChildOfClass("Humanoid")
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not Hum or not hrp then return end
    if ragdollActive then return end

    local targetName = targetBox.Text
    if targetName == "" then return end
    local target = nil
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Name:lower() == targetName:lower() then
            target = pl
            break
        end
    end

    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local targetHRP = target.Character.HumanoidRootPart
        local distance = (hrp.Position - targetHRP.Position).Magnitude
        if distance <= followStopDistance then
            Hum.WalkSpeed = 0
            Hum:MoveTo(hrp.Position)
        else
            Hum.WalkSpeed = defaultWalkSpeed
            Hum:MoveTo(targetHRP.Position)
        end
    else
        if not ragdollActive and Hum then
            Hum.WalkSpeed = defaultWalkSpeed
        end
    end
end)

-- Buttons behaviors
rapeButton.MouseButton1Click:Connect(function()
    rapeEnabled = not rapeEnabled
    if rapeEnabled then
        rapeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        rapeButton.Text = "Follow player/ON"
        notify("Follow ON!")
    else
        rapeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        rapeButton.Text = "Follow player/OFF"
        notify("Follow OFF!")
        local char = LocalPlayer.Character
        local h = char and char:FindFirstChildOfClass("Humanoid")
        if h and not ragdollActive then restoreHumanoid(h) end
    end
end)

-- AutoClick loop
task.spawn(function()
    while task.wait(0.1) do
        if autoClickEnabled then
            pcall(function()
                local char = LocalPlayer.Character
                if char then
                    local tool = char:FindFirstChildOfClass("Tool")
                    if tool then pcall(function() tool:Activate() end) end
                end
            end)
        end
    end
end)

autoClickButton.MouseButton1Click:Connect(function()
    autoClickEnabled = not autoClickEnabled
    if autoClickEnabled then
        autoClickButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        autoClickButton.Text = "Auto Click/ON"
        notify("Auto Click ON!")
    else
        autoClickButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        autoClickButton.Text = "Auto Click/OFF"
        notify("Auto Click OFF!")
    end
end)

-- Anti Ragdoll toggle
antiRagdollButton.MouseButton1Click:Connect(function()
    antiRagdollEnabled = not antiRagdollEnabled
    if antiRagdollEnabled then
        antiRagdollButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        antiRagdollButton.Text = "Anti Ragdoll/ON"
        notify("Anti Ragdoll Ativado!")
        local char = LocalPlayer.Character
        local h = char and char:FindFirstChildOfClass("Humanoid")
        if h and (h.PlatformStand or h:GetState() == Enum.HumanoidStateType.Ragdoll or h:GetState() == Enum.HumanoidStateType.FallingDown or h:GetState() == Enum.HumanoidStateType.Physics) then
            ragdollActive = true
            freezeHumanoid(h)
        end
    else
        antiRagdollButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        antiRagdollButton.Text = "Anti Ragdoll/OFF"
        notify("Anti Ragdoll Desativado!")
        local char = LocalPlayer.Character
        local h = char and char:FindFirstChildOfClass("Humanoid")
        if h then ragdollActive = false restoreHumanoid(h) end
    end
end)

-- toggle GUI
toggleButton.MouseButton1Click:Connect(function()
    mainGui.Visible = not mainGui.Visible
end)
