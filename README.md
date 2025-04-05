local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local ContextActionService = game:GetService("ContextActionService")

local localPlayer = Players.LocalPlayer
local carrying = false
local target = nil

-- Мобильный интерфейс
local touchButton = Instance.new("TextButton")
touchButton.Size = UDim2.new(0, 100, 0, 100)
touchButton.Position = UDim2.new(0.8, 0, 0.7, 0)
touchButton.Text = "ПОДНЯТЬ"
touchButton.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("TouchGui")

-- Адаптация под сенсорный ввод
local function getPlayerUnderTouch()
    local touchPos = UIS:GetTouchLocation()
    local camera = workspace.CurrentCamera
    local ray = camera:ViewportPointToRay(touchPos.X, touchPos.Y)
    local part = workspace:FindPartOnRayWithIgnoreList(ray, {localPlayer.Character})
    
    if part and part.Parent:FindFirstChild("Humanoid") then
        return part.Parent
    end
    return nil
end

-- Основная логика
local function handleAction(_, inputState)
    if inputState == Enum.UserInputState.Begin then
        if carrying then return end
        
        target = getPlayerUnderTouch()
        if target then
            carrying = true
            local root = target:FindFirstChild("HumanoidRootPart")
            local lroot = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if root and lroot then
                local weld = Instance.new("Weld")
                weld.Part0 = lroot
                weld.Part1 = root
                weld.C0 = CFrame.new(0, 2, 0)
                weld.Parent = lroot
                
                RunService.Heartbeat:Connect(function()
                    if not carrying or not root:IsDescendantOf(workspace) then
                        weld:Destroy()
                        carrying = false
                    end
                end)
            end
        end
    elseif inputState == Enum.UserInputState.End then
        carrying = false
        if target then
            local root = target:FindFirstChild("HumanoidRootPart")
            if root then
                for _, weld in pairs(root:GetChildren()) do
                    if weld:IsA("Weld") then
                        weld:Destroy()
                    end
                end
            end
            target = nil
        end
    end
end

-- Настройка управления
ContextActionService:BindActionAtPriority(
    "LiftAction",
    handleAction,
    false,
    1000,
    Enum.KeyCode.ButtonR2,  -- Для контроллеров
    Enum.UserInputType.Touch -- Для сенсорных устройств
)

-- Создаем виртуальную кнопку для мобилок
if UIS.TouchEnabled then
    touchButton.Visible = true
    touchButton.Activated:Connect(function()
        handleAction(nil, Enum.UserInputState.Begin)
    end)
    touchButton.Deactivated:Connect(function()
        handleAction(nil, Enum.UserInputState.End)
    end)
else
    touchButton.Visible = false
end
