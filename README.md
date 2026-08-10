-- ==========================================
-- SCRIPT ESP - THEBA HUB
-- ==========================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESPEnabled = false

-- 1. TẠO GUI THEBA HUB (Viền Cầu Vồng)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ThebaHub_ESP"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 200, 0, 110)
MainFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Viền cầu vồng
local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 3
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Parent = MainFrame

-- Tiêu đề Theba Hub
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "THEBA HUB"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Nút Bật/Tắt ESP
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0.85, 0, 0, 40)
ToggleBtn.Position = UDim2.new(0.075, 0, 0.5, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ToggleBtn.Text = "ESP: TẮT (Chỉ Hiện Tên)"
ToggleBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
ToggleBtn.TextSize = 13
ToggleBtn.Font = Enum.Font.GothamSemibold
ToggleBtn.Parent = MainFrame

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 6)
BtnCorner.Parent = ToggleBtn

-- Hiệu ứng viền Cầu Vồng (Rainbow Loop)
RunService.RenderStepped:Connect(function()
    local hue = (tick() % 5) / 5
    UIStroke.Color = Color3.fromHSV(hue, 1, 1)
end)

-- 2. HỆ THỐNG HIỂN THỊ ESP
local ESPObjects = {}

local function CreateESP(player)
    if player == LocalPlayer then return end

    -- Bảng Tên
    local Billboard = Instance.new("BillboardGui")
    Billboard.Name = "ESP_Name"
    Billboard.AlwaysOnTop = true
    Billboard.Size = UDim2.new(0, 100, 0, 30)
    Billboard.StudsOffset = Vector3.new(0, 3, 0)

    local NameLabel = Instance.new("TextLabel")
    NameLabel.Size = UDim2.new(1, 0, 1, 0)
    NameLabel.BackgroundTransparency = 1
    NameLabel.Text = player.DisplayName or player.Name
    NameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    NameLabel.TextStrokeTransparency = 0
    NameLabel.Font = Enum.Font.SourceSansBold
    NameLabel.TextSize = 14
    NameLabel.Parent = Billboard

    -- Khung Box
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(255, 0, 0)
    Box.Thickness = 1.5
    Box.Filled = false

    -- Dây Tracer
    local Line = Drawing.new("Line")
    Line.Visible = false
    Line.Color = Color3.fromRGB(255, 255, 0)
    Line.Thickness = 1.5

    ESPObjects[player] = {
        Billboard = Billboard,
        Box = Box,
        Line = Line
    }
end

local function RemoveESP(player)
    if ESPObjects[player] then
        if ESPObjects[player].Billboard then ESPObjects[player].Billboard:Destroy() end
        if ESPObjects[player].Box then ESPObjects[player].Box:Remove() end
        if ESPObjects[player].Line then ESPObjects[player].Line:Remove() end
        ESPObjects[player] = nil
    end
end

-- Tải người chơi
for _, p in pairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(RemoveESP)

-- 3. CẬP NHẬT VỊ TRÍ ESP THEO RENDER
RunService.RenderStepped:Connect(function()
    for player, esp in pairs(ESPObjects) do
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")

        if char and hrp and head and char:FindFirstChildOfClass("Humanoid") and char.Humanoid.Health > 0 then
            -- Luôn gắn bảng tên lên đầu người chơi
            if esp.Billboard.Parent ~= head then
                esp.Billboard.Parent = head
            end
            esp.Billboard.Enabled = true

            -- Tính toán vị trí khung & dây khi bật ESP
            if ESPEnabled then
                local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                    local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
                    
                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 1.5

                    -- Cập nhật Box
                    esp.Box.Size = Vector2.new(width, height)
                    esp.Box.Position = Vector2.new(hrpPos.X - width / 2, hrpPos.Y - height / 2)
                    esp.Box.Visible = true

                    -- Cập nhật Line (nối từ đáy màn hình tới người chơi)
                    esp.Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    esp.Line.To = Vector2.new(hrpPos.X, hrpPos.Y)
                    esp.Line.Visible = true
                else
                    esp.Box.Visible = false
                    esp.Line.Visible = false
                end
            else
                esp.Box.Visible = false
                esp.Line.Visible = false
            end
        else
            esp.Billboard.Enabled = false
            esp.Box.Visible = false
            esp.Line.Visible = false
        end
    end
end)

-- 4. SỰ KIỆN NÚT BẤM
ToggleBtn.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    if ESPEnabled then
        ToggleBtn.Text = "ESP: BẬT (Hiện Khung + Dây)"
        ToggleBtn.TextColor3 = Color3.fromRGB(0, 255, 127)
    else
        ToggleBtn.Text = "ESP: TẮT (Chỉ Hiện Tên)"
        ToggleBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    end
end)
