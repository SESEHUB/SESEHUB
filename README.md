local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 300, 0, 400)
Frame.Position = UDim2.new(0.5, -150, 0.5, -200)
Frame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Text = "Player Teleport GUI"
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.SourceSans
Title.TextSize = 24
Title.Parent = Frame

local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Size = UDim2.new(1, 0, 1, -50)
ScrollingFrame.Position = UDim2.new(0, 0, 0, 50)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0) 
ScrollingFrame.ScrollBarThickness = 10
ScrollingFrame.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
ScrollingFrame.Parent = Frame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.FillDirection = Enum.FillDirection.Vertical
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Top
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = ScrollingFrame

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Text = "-"
MinimizeButton.Size = UDim2.new(0, 30, 0, 30)
MinimizeButton.Position = UDim2.new(1, -30, 0, 0)
MinimizeButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
MinimizeButton.TextColor3 = Color3.new(1, 1, 1)
MinimizeButton.Font = Enum.Font.SourceSans
MinimizeButton.TextSize = 24
MinimizeButton.Parent = Frame

local dragging = false
local dragStartPos = nil
local offset = nil

Title.InputBegan:Connect(function(input, gameProcessedEvent)
    if gameProcessedEvent then return end
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartPos = input.Position
        offset = Frame.Position - UDim2.new(0, dragStartPos.X, 0, dragStartPos.Y)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging then
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStartPos
            Frame.Position = UDim2.new(0, offset.X + delta.X, 0, offset.Y + delta.Y)
        end
    end
end)

Title.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

local function UpdatePlayerList()
   
    for _, button in ipairs(ScrollingFrame:GetChildren()) do
        if button:IsA("TextButton") then
            button:Destroy()
        end
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local Button = Instance.new("TextButton")
            Button.Text = player.Name
            Button.Size = UDim2.new(1, 0, 0, 30)
            Button.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
            Button.TextColor3 = Color3.new(1, 1, 1)
            Button.Font = Enum.Font.SourceSans
            Button.TextSize = 20
            Button.Parent = ScrollingFrame

            Button.MouseButton1Click:Connect(function()
                local character = player.Character
                if character then
                    local rootPart = character:FindFirstChild("HumanoidRootPart")
                    if rootPart and LocalPlayer.Character then
                        LocalPlayer.Character.HumanoidRootPart.CFrame = rootPart.CFrame
                    end
                end
            end)
        end
    end

    local totalHeight = #Players:GetPlayers() * 30
    ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, totalHeight)
end

UpdatePlayerList()

Players.PlayerAdded:Connect(UpdatePlayerList)
Players.PlayerRemoving:Connect(UpdatePlayerList)

local isMinimized = false
MinimizeButton.MouseButton1Click:Connect(function()
    if isMinimized then
        Frame.Size = UDim2.new(0, 300, 0, 400)
        ScrollingFrame.Visible = true
        MinimizeButton.Text = "-"
    else
        Frame.Size = UDim2.new(0, 300, 0, 50)
        ScrollingFrame.Visible = false
        MinimizeButton.Text = "+"
    end
    isMinimized = not isMinimized
end)
