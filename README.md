-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LP = Players.LocalPlayer
local Cam = workspace.CurrentCamera

-- ESP settings (modifiable by GUI)
local ESP_RADIUS = 250
local ESP_COLOR = Color3.fromRGB(255, 0, 0)
local ESP_ENABLED = true

local AIMBOT_ENABLED = false           -- Overall aimbot feature toggle (GUI switch)
local AIMBOT_TOGGLE_ACTIVE = false    -- Actual aiming active state toggled by J key
local AIMBOT_TARGET = nil
local AIMBOT_SMOOTHNESS = 0.15 -- smaller = faster snap, bigger = slower smooth

local esp = {}
local UPDATE_RATE = 0.1
local timer = 0

-- ===== ESP LOGIC =====

local function newESP(char)
    local h = Instance.new("Highlight")
    h.FillColor = ESP_COLOR
    h.OutlineColor = Color3.new(1,1,1)
    h.FillTransparency = 0.5
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Adornee = char
    h.Parent = workspace
    return h
end

RunService.RenderStepped:Connect(function(dt)
    timer += dt
    if timer < UPDATE_RATE then return end
    timer = 0

    if ESP_ENABLED then
        for _, p in Players:GetPlayers() do
            if p == LP then continue end
            local c = p.Character
            local hum = c and c:FindFirstChildOfClass("Humanoid")
            local root = c and c:FindFirstChild("HumanoidRootPart")

            if not (c and hum and root and hum.Health > 0) then
                if esp[c] then esp[c]:Destroy(); esp[c] = nil end
                continue
            end

            local d = (Cam.CFrame.Position - root.Position).Magnitude
            if d <= ESP_RADIUS then
                if not esp[c] then
                    esp[c] = newESP(c)
                end
                esp[c].FillColor = ESP_COLOR
            elseif esp[c] then
                esp[c]:Destroy()
                esp[c] = nil
            end
        end
    else
        -- ESP disabled, remove all highlights
        for c, h in pairs(esp) do
            h:Destroy()
            esp[c] = nil
        end
    end
end)

-- ===== AIMBOT LOGIC =====

local function getNearestHead()
    local nearestDist = math.huge
    local nearestHead = nil
    local camPos = Cam.CFrame.Position

    for _, p in Players:GetPlayers() do
        if p == LP then continue end
        local c = p.Character
        if c then
            local hum = c:FindFirstChildOfClass("Humanoid")
            local head = c:FindFirstChild("Head")
            if hum and hum.Health > 0 and head then
                local dist = (head.Position - camPos).Magnitude
                if dist < nearestDist and dist <= ESP_RADIUS then
                    nearestDist = dist
                    nearestHead = head
                end
            end
        end
    end

    return nearestHead
end

local function lerp(a, b, t)
    return a + (b - a) * t
end

RunService.RenderStepped:Connect(function()
    if AIMBOT_ENABLED and AIMBOT_TOGGLE_ACTIVE then
        -- Find closest target every frame (so it switches automatically)
        AIMBOT_TARGET = getNearestHead()

        if AIMBOT_TARGET then
            local camPos = Cam.CFrame.Position
            local targetPos = AIMBOT_TARGET.Position

            local direction = (targetPos - camPos).Unit
            local currentLook = Cam.CFrame.LookVector

            -- Smoothly lerp from current look vector toward target direction
            local newLook = currentLook:Lerp(direction, AIMBOT_SMOOTHNESS)

            -- Rebuild the camera CFrame with same position and new look vector
            Cam.CFrame = CFrame.new(camPos, camPos + newLook)
        end
        -- If no target, do nothing and camera stays free
    else
        -- Not aiming, clear target ref
        AIMBOT_TARGET = nil
    end
end)

UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if input.KeyCode == Enum.KeyCode.J then
            if AIMBOT_ENABLED then
                AIMBOT_TOGGLE_ACTIVE = not AIMBOT_TOGGLE_ACTIVE
            end
        end
    end
end)

-- ===== GUI =====

local playerGui = LP:WaitForChild("PlayerGui")
local gui = playerGui:FindFirstChild("ESP_GUI")

if not gui then
    gui = Instance.new("ScreenGui")
    gui.Name = "ESP_GUI"
    gui.ResetOnSpawn = false
    gui.Parent = playerGui
end

-- Clear old UI if it exists to prevent duplicates on script re-run
for _, child in ipairs(gui:GetChildren()) do
    child:Destroy()
end

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.fromScale(0.25, 0.4)
frame.Position = UDim2.fromScale(0.05, 0.3)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true
frame.ClipsDescendants = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 16)
Instance.new("UIStroke", frame).Thickness = 2

-- Title
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.fromScale(1, 0.15)
title.BackgroundTransparency = 1
title.Text = "ToTAElly safe GUI"  -- <-- updated title here
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextScaled = true-- Title

-- ScrollingFrame to hold controls
local scrollFrame = Instance.new("ScrollingFrame", frame)
scrollFrame.Size = UDim2.fromScale(1, 0.7)
scrollFrame.Position = UDim2.fromScale(0, 0.18)
scrollFrame.CanvasSize = UDim2.new(0, 0, 1, 0)
scrollFrame.ScrollBarThickness = 6
scrollFrame.BackgroundTransparency = 1
scrollFrame.VerticalScrollBarInset = Enum.ScrollBarInset.Always

local grid = Instance.new("UIGridLayout", scrollFrame)
grid.CellSize = UDim2.new(1, -20, 0, 50)
grid.CellPadding = UDim2.new(0, 0, 0, 10)

-- Switch creation function
local function createSwitch(name, initialState, parent)
    local container = Instance.new("Frame", parent)
    container.Size = UDim2.new(1, 0, 0, 50)
    container.BackgroundTransparency = 1

    local label = Instance.new("TextLabel", container)
    label.Text = name
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.GothamBold
    label.TextScaled = true
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Position = UDim2.new(0.03, 0, 0, 0)

    local button = Instance.new("TextButton", container)
    button.Size = UDim2.new(0.25, 0, 0.6, 0)
    button.Position = UDim2.new(0.72, 0, 0.2, 0)
    button.BackgroundColor3 = initialState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
    button.Text = initialState and "ON" or "OFF"
    button.Font = Enum.Font.GothamBold
    button.TextScaled = true
    button.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", button)

    button.MouseButton1Click:Connect(function()
        local newState = not (button.Text == "ON")
        button.Text = newState and "ON" or "OFF"
        button.BackgroundColor3 = newState and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(170, 0, 0)
        if name == "ESP" then
            ESP_ENABLED = newState
        elseif name == "Aimbot" then
            AIMBOT_ENABLED = newState
            if not AIMBOT_ENABLED then
                AIMBOT_TOGGLE_ACTIVE = false
                AIMBOT_TARGET = nil
            end
        end
    end)

    return container
end

local espSwitch = createSwitch("ESP", ESP_ENABLED, scrollFrame)
local aimbotSwitch = createSwitch("Aimbot", AIMBOT_ENABLED, scrollFrame)

-- Radius box
local radiusBox = Instance.new("TextBox", scrollFrame)
radiusBox.Size = UDim2.new(1, 0, 0, 50)
radiusBox.Text = tostring(ESP_RADIUS)
radiusBox.PlaceholderText = "ESP Radius"
radiusBox.Font = Enum.Font.Gotham
radiusBox.TextScaled = true
radiusBox.BackgroundColor3 = Color3.fromRGB(45,45,45)
radiusBox.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", radiusBox)

radiusBox.FocusLost:Connect(function()
    local n = tonumber(radiusBox.Text)
    if n and n > 0 then
        ESP_RADIUS = math.clamp(n, 10, 2000)
        radiusBox.Text = ESP_RADIUS
    else
        radiusBox.Text = ESP_RADIUS
    end
end)

-- Color buttons container inside scroll frame
local colorContainer = Instance.new("Frame", scrollFrame)
colorContainer.Size = UDim2.new(1, 0, 0, 50)
colorContainer.BackgroundTransparency = 1

local colorGrid = Instance.new("UIGridLayout", colorContainer)
colorGrid.CellSize = UDim2.new(0.2, -8, 1, 0)  -- 5 columns
colorGrid.CellPadding = UDim2.new(0.02, 0, 0, 0)

local colors = {
    Red = Color3.fromRGB(255,0,0),
    Blue = Color3.fromRGB(0,170,255),
    Green = Color3.fromRGB(0,255,120),
    Black = Color3.fromRGB(0,0,0),
    Yellow = Color3.fromRGB(255, 255, 0)
}

for name, col in pairs(colors) do
    local btn = Instance.new("TextButton", colorContainer)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.Text = name
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.BackgroundColor3 = col
    btn.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        ESP_COLOR = col
        for _, h in pairs(esp) do
            h.FillColor = ESP_COLOR
        end
    end)
end

-- Adjust CanvasSize when content changes
local function updateCanvasSize()
    local layout = scrollFrame:FindFirstChildOfClass("UIGridLayout")
    if layout then
        local contentHeight = layout.AbsoluteContentSize.Y
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, contentHeight)
    end
end

scrollFrame.ChildAdded:Connect(updateCanvasSize)
scrollFrame.ChildRemoved:Connect(updateCanvasSize)
updateCanvasSize()
