--[[
Requer uma integração com ImGui! (exemplo: para FiveM, CitizenFX, ou outros wrappers que suportam ImGui em Lua)
]]

local ImGui = require("ImGui") -- Adapte para seu ambiente
local showMenu = true

local FOV = 30
local AIM_SMOOTH = 0.1
local aimbotEnabled = true

function GetAngleToTarget(playerPos, targetPos)
    local dx = targetPos.x - playerPos.x
    local dy = targetPos.y - playerPos.y
    local dz = targetPos.z - playerPos.z
    return math.atan2(dy, dx) * (180 / math.pi)
end

function IsInFOV(playerAngle, targetAngle, fov)
    local diff = math.abs(targetAngle - playerAngle)
    return diff <= (fov / 2)
end

function Aimbot(player, targets)
    if not aimbotEnabled then return end
    local bestTarget = nil
    local bestAngleDiff = FOV
    for _, target in pairs(targets) do
        if target.isAlive and target.team ~= player.team then
            local angleToTarget = GetAngleToTarget(player.position, target.position)
            if IsInFOV(player.angle, angleToTarget, FOV) then
                local angleDiff = math.abs(player.angle - angleToTarget)
                if angleDiff < bestAngleDiff then
                    bestAngleDiff = angleDiff
                    bestTarget = target
                end
            end
        end
    end
    if bestTarget then
        local angleToAim = GetAngleToTarget(player.position, bestTarget.position)
        player.angle = player.angle + (angleToAim - player.angle) * AIM_SMOOTH
    end
end

-- Função para desenhar a interface ImGui
function DrawAimbotMenu()
    ImGui.Begin("Aimbot Settings", showMenu)
    _, aimbotEnabled = ImGui.Checkbox("Aimbot On/Off", aimbotEnabled)
    _, FOV = ImGui.SliderInt("FOV", FOV, 1, 180)
    _, AIM_SMOOTH = ImGui.SliderFloat("Aimbot Smooth", AIM_SMOOTH, 0.01, 1.0)
    ImGui.Text("Pressione INSERT para mostrar/ocultar o menu")
    ImGui.End()
end

-- Exemplo de integração no loop do jogo (adapte para seu ambiente)
Citizen.CreateThread(function()
    while true do
        if showMenu then DrawAimbotMenu() end
        -- Aimbot logic
        local player = GetLocalPlayer()
        local targets = GetAllEnemies()
        Aimbot(player, targets)
        Citizen.Wait(10)
    end
end)

-- Atalho para mostrar/ocultar o menu (INSERT)
RegisterCommand("toggle_aimbot_menu", function()
    showMenu = not showMenu
end, false)

-- Mapear tecla INSERT para abrir/fechar menu (adapte para seu ambiente)
Citizen.CreateThread(function()
    while true do
        if IsKeyJustPressed(0x2D) then -- 0x2D = VK_INSERT
            showMenu = not showMenu
        end
        Citizen.Wait(10)
    end
end)
