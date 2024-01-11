local scriptCode = [[
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

local sensitivity = 0.02
local targetRefreshRate = 0.1
local maxAimDistance = 300
local smoothnessFactor = 0.15
local fovAdjustment = 5
local highlightColor = Color3.new(1, 0, 0)

local automaticAim = true
local highlightPlayers = true

local function findNearestPlayerHead()
    local closestPlayer = nil
    local minDistance = maxAimDistance

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local distance = (player.Character.Head.Position - Camera.CFrame.Position).Magnitude
            if distance < minDistance then
                minDistance = distance
                closestPlayer = player
            end
        end
    end

    return closestPlayer
end

local function highlightPlayer(player)
    if player and player.Character and player.Character:FindFirstChild("Head") then
        local head = player.Character.Head
        local highlight = Instance.new("SelectionBox")
        highlight.Name = "PlayerHighlight"
        highlight.Color3 = highlightColor
        highlight.LineThickness = 0.01
        highlight.Adornee = head
        highlight.Parent = head
    end
end

local function removePlayerHighlight(player)
    if player and player.Character then
        local head = player.Character:FindFirstChild("Head")
        if head then
            local highlight = head:FindFirstChild("PlayerHighlight")
            if highlight then
                highlight:Destroy()
            end
        end
    end
end

local function aimAtPlayerHead()
    if not automaticAim then
        return
    end

    local nearestPlayer = findNearestPlayerHead()

    if nearestPlayer then
        local headPosition = nearestPlayer.Character.Head.Position
        local lookVector = (headPosition - Camera.CFrame.Position).Unit
        local newRotation = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + lookVector)

        Camera.CFrame = Camera.CFrame:Lerp(newRotation, smoothnessFactor * sensitivity)
        Camera.FieldOfView = math.rad(Camera.FieldOfView + fovAdjustment)
    end
end

local function onButtonActivated(button)
    if button == "AutomaticAim" then
        automaticAim = not automaticAim
    elseif button == "HighlightPlayers" then
        highlightPlayers = not highlightPlayers
        StarterGui:SetCore("GuiService", {
            SetWaypoint = {
                WaypointType = Enum.WaypointType.Prominent,
                Position = Players.LocalPlayer.Character:WaitForChild("HumanoidRootPart").Position,
                Text = "Highlighted Player",
                ShowArrow = true,
                AlwaysOnTop = true,
            },
        })
    end
end

RunService.RenderStepped:Connect(aimAtPlayerHead)

while wait(targetRefreshRate) do
    aimAtPlayerHead()
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch and input.UserInputState == Enum.UserInputState.Begin then
        onButtonActivated("HighlightPlayers")
    end
end)
]]

local success, errorMessage = loadstring(scriptCode)
if success then
    success()
else
    warn("Error in script: " .. errorMessage)
end
