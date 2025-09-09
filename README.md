-- Services
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local players = game:GetService("Players")
local runService = game:GetService("RunService")

-- Aimbot & ESP Variables
local aimbotEnabled = false
local espEnabled = false
local teamCheckEnabled = false
local fovRadius = 100
local guiName = "AimbotToggleGUI"
local guiVisible = true
local espObjects = {}

-- FOV Circle
local circle = Drawing.new("Circle")
circle.Color = Color3.fromRGB(255, 255, 255)
circle.Thickness = 1
circle.Filled = false
circle.Radius = fovRadius
circle.Visible = true
circle.Position = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)

-- Show Notification
local function showNotification()
    local notification = Instance.new("ScreenGui")
    notification.Name = "NotificationGUI"
    notification.ResetOnSpawn = false
    notification.Parent = player:WaitForChild("PlayerGui")

    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6073491164"
    sound.Volume = 1
    sound.Parent = notification
    sound:Play()

    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = notification
    textLabel.Size = UDim2.new(0, 250, 0, 50)
    textLabel.Position = UDim2.new(1, -260, 1, -60)
    textLabel.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    textLabel.BorderSizePixel = 0
    textLabel.Text = "✅ Script Successfully Executed!"
    textLabel.TextColor3 = Color3.new(1, 1, 1)
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.SourceSansBold

    task.delay(3, function()
        for i = 1, 10 do
            textLabel.TextTransparency = i * 0.1
            textLabel.BackgroundTransparency = i * 0.1
            task.wait(0.05)
        end
        notification:Destroy()
    end)
end

-- Create ESP for a player
local function createESPForPlayer(p)
    local nameTag = Drawing.new("Text")
    nameTag.Size = 14
    nameTag.Color = Color3.fromRGB(255, 0, 0)
    nameTag.Center = true
    nameTag.Outline = true

    local distanceTag = Drawing.new("Text")
    distanceTag.Size = 13
    distanceTag.Color = Color3.fromRGB(255, 0, 0)
    distanceTag.Center = true
    distanceTag.Outline = true

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Filled = false

    local tracer = Drawing.new("Line")
    tracer.Thickness = 1
    tracer.Color = Color3.fromRGB(255, 0, 0)

    espObjects[p] = {
        name = nameTag,
        distance = distanceTag,
        box = box,
        tracer = tracer
    }
end

-- Remove ESP
local function removeESPForPlayer(p)
    if espObjects[p] then
        for _, drawing in pairs(espObjects[p]) do
            drawing:Remove()
        end
        espObjects[p] = nil
    end
end

players.PlayerRemoving:Connect(removeESPForPlayer)

-- Visibility Check
local function isVisible(part)
    if not part then return false end
    local origin = camera.CFrame.Position
    local direction = (part.Position - origin)
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
    rayParams.FilterDescendantsInstances = { player.Character or workspace }
    local result = workspace:Raycast(origin, direction, rayParams)
    if result then
        return part:IsDescendantOf(result.Instance.Parent) or result.Instance:IsDescendantOf(part.Parent)
    else
        return true
    end
end

-- Closest Player Function (with team check)
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = fovRadius

    for _, p in pairs(players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("Head") then
            if teamCheckEnabled and p.Team == player.Team then
                continue
            end
            local head = p.Character.Head
            local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local distanceFromCenter = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)).Magnitude
                if distanceFromCenter < shortestDistance and isVisible(head) then
                    shortestDistance = distanceFromCenter
                    closestPlayer = p
                end
            end
        end
    end

    return closestPlayer
end

-- GUI Creation Function
local function createGUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = guiName
    gui.ResetOnSpawn = false
    gui.Parent = player:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame", gui)
    frame.Position = UDim2.new(0, 20, 0, 100)
    frame.Size = UDim2.new(0, 200, 0, 175)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, 0, 0, 25)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    title.Text = "ligma aimbot v1 by @Ryker_plays(on YT)"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.TextScaled = true
    title.Font = Enum.Font.SourceSansBold

    local aimbotOn = Instance.new("TextButton", frame)
    aimbotOn.Size = UDim2.new(0.5, -5, 0.4, -5)
    aimbotOn.Position = UDim2.new(0, 5, 0, 30)
    aimbotOn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    aimbotOn.Text = "Aimbot ON"
    aimbotOn.TextColor3 = Color3.new(1, 1, 1)
    aimbotOn.TextScaled = true

    local aimbotOff = Instance.new("TextButton", frame)
    aimbotOff.Size = UDim2.new(0.5, -5, 0.4, -5)
    aimbotOff.Position = UDim2.new(0.5, 5, 0, 30)
    aimbotOff.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    aimbotOff.Text = "Aimbot OFF ✅"
    aimbotOff.TextColor3 = Color3.new(1, 1, 1)
    aimbotOff.TextScaled = true

    local espOn = Instance.new("TextButton", frame)
    espOn.Size = UDim2.new(0.5, -5, 0.4, -5)
    espOn.Position = UDim2.new(0, 5, 0.5, 10)
    espOn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    espOn.Text = "ESP ON"
    espOn.TextColor3 = Color3.new(1, 1, 1)
    espOn.TextScaled = true

    local espOff = Instance.new("TextButton", frame)
    espOff.Size = UDim2.new(0.5, -5, 0.4, -5)
    espOff.Position = UDim2.new(0.5, 5, 0.5, 10)
    espOff.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    espOff.Text = "ESP OFF ✅"
    espOff.TextColor3 = Color3.new(1, 1, 1)
    espOff.TextScaled = true

    local hideButton = Instance.new("TextButton", gui)
    hideButton.Size = UDim2.new(0, 100, 0, 30)
    hideButton.Position = UDim2.new(0, 20, 0, 285)
    hideButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    hideButton.Text = "Hide GUI"
    hideButton.TextColor3 = Color3.new(1, 1, 1)
    hideButton.TextScaled = true

    local teamCheckButton = Instance.new("TextButton", gui)
    teamCheckButton.Size = UDim2.new(0, 200, 0, 30)
    teamCheckButton.Position = UDim2.new(0, 20, 0, 325)
    teamCheckButton.BackgroundColor3 = Color3.fromRGB(120, 120, 255)
    teamCheckButton.Text = "Team Check: OFF"
    teamCheckButton.TextColor3 = Color3.new(1, 1, 1)
    teamCheckButton.TextScaled = true

    teamCheckButton.MouseButton1Click:Connect(function()
        teamCheckEnabled = not teamCheckEnabled
        teamCheckButton.Text = "Team Check: " .. (teamCheckEnabled and "ON ✅" or "OFF")
        teamCheckButton.BackgroundColor3 = teamCheckEnabled and Color3.fromRGB(0, 170, 255) or Color3.fromRGB(120, 120, 255)
    end)

    hideButton.MouseButton1Click:Connect(function()
        guiVisible = not guiVisible
        frame.Visible = guiVisible
        hideButton.Text = guiVisible and "Hide GUI" or "Show GUI"
    end)

    aimbotOn.MouseButton1Click:Connect(function()
        aimbotEnabled = true
        aimbotOn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        aimbotOff.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        aimbotOn.Text = "Aimbot ON ✅"
        aimbotOff.Text = "Aimbot OFF"
    end)

    aimbotOff.MouseButton1Click:Connect(function()
        aimbotEnabled = false
        aimbotOn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        aimbotOff.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        aimbotOn.Text = "Aimbot ON"
        aimbotOff.Text = "Aimbot OFF ✅"
    end)

    espOn.MouseButton1Click:Connect(function()
        espEnabled = true
        espOn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        espOff.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        espOn.Text = "ESP ON ✅"
        espOff.Text = "ESP OFF"
    end)

    espOff.MouseButton1Click:Connect(function()
        espEnabled = false
        espOn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        espOff.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        espOn.Text = "ESP ON"
        espOff.Text = "ESP OFF ✅"
        for _, drawings in pairs(espObjects) do
            drawings.box.Visible = false
            drawings.name.Visible = false
            drawings.distance.Visible = false
            drawings.tracer.Visible = false
        end
    end)
end

createGUI()
showNotification()

player.CharacterAdded:Connect(function()
    task.wait(1)
    if not player:WaitForChild("PlayerGui"):FindFirstChild(guiName) then
        createGUI()
    end
end)

runService.RenderStepped:Connect(function()
    circle.Position = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)

    if aimbotEnabled then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            camera.CFrame = CFrame.new(camera.CFrame.Position, headPos)
        end
    end

    for _, p in pairs(players:GetPlayers()) do
        if p ~= player then
            if not espObjects[p] then
                createESPForPlayer(p)
            end

            local drawings = espObjects[p]
            local char = p.Character
            if espEnabled and char and char:FindFirstChild("Head") and char:FindFirstChild("HumanoidRootPart") then
                if teamCheckEnabled and p.Team == player.Team then
                    drawings.box.Visible = false
                    drawings.name.Visible = false
                    drawings.distance.Visible = false
                    drawings.tracer.Visible = false
                    continue
                end

                local head = char.Head
                local hrp = char.HumanoidRootPart
                local headPos2D, onScreen1 = camera:WorldToViewportPoint(head.Position)
                local rootPos2D, onScreen2 = camera:WorldToViewportPoint(hrp.Position)

                if onScreen1 and onScreen2 then
                    local height = (headPos2D - rootPos2D).Magnitude * 2
                    local width = height / 2

                    drawings.box.Size = Vector2.new(width, height)
                    drawings.box.Position = Vector2.new(rootPos2D.X - width/2, rootPos2D.Y - height/2)
                    drawings.box.Visible = true

                    drawings.name.Text = p.Name
                    drawings.name.Position = Vector2.new(headPos2D.X, headPos2D.Y - 20)
                    drawings.name.Visible = true

                    local distance = math.floor((player.Character.HumanoidRootPart.Position - hrp.Position).Magnitude)
                    drawings.distance.Text = tostring(distance) .. "m"
                    drawings.distance.Position = Vector2.new(rootPos2D.X, rootPos2D.Y + height/2 + 5)
                    drawings.distance.Visible = true

                    drawings.tracer.From = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y)
                    drawings.tracer.To = Vector2.new(rootPos2D.X, rootPos2D.Y)
                    drawings.tracer.Visible = true
                else
                    drawings.box.Visible = false
                    drawings.name.Visible = false
                    drawings.distance.Visible = false
                    drawings.tracer.Visible = false
                end
            else
                drawings.box.Visible = false
                drawings.name.Visible = false
                drawings.distance.Visible = false
                drawings.tracer.Visible = false
            end
        end
    end
end)
