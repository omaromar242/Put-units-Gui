local player = game.Players.LocalPlayer

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TowerSpawnTool"
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Create Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 400, 0, 300)
frame.Position = UDim2.new(0.5, -200, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.BorderSizePixel = 2
frame.Parent = screenGui

-- Make the frame draggable
local dragging, dragStart, startPos
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

frame.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Create Tower ID TextBox
local idLabel = Instance.new("TextLabel")
idLabel.Size = UDim2.new(0.8, 0, 0.15, 0)
idLabel.Position = UDim2.new(0.1, 0, 0.1, 0)
idLabel.Text = "Tower ID:"
idLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
idLabel.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
idLabel.Font = Enum.Font.SourceSans
idLabel.TextSize = 16
idLabel.Parent = frame

local idTextBox = Instance.new("TextBox")
idTextBox.Size = UDim2.new(0.8, 0, 0.15, 0)
idTextBox.Position = UDim2.new(0.1, 0, 0.25, 0)
idTextBox.PlaceholderText = "Enter Tower ID"
idTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
idTextBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
idTextBox.Font = Enum.Font.SourceSans
idTextBox.TextSize = 16
idTextBox.ClearTextOnFocus = false
idTextBox.Parent = frame

-- Create CFrame TextBox
local cframeLabel = Instance.new("TextLabel")
cframeLabel.Size = UDim2.new(0.8, 0, 0.15, 0)
cframeLabel.Position = UDim2.new(0.1, 0, 0.4, 0)
cframeLabel.Text = "CFrame (Position & Rotation):"
cframeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
cframeLabel.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
cframeLabel.Font = Enum.Font.SourceSans
cframeLabel.TextSize = 16
cframeLabel.Parent = frame

local cframeTextBox = Instance.new("TextBox")
cframeTextBox.Size = UDim2.new(0.8, 0, 0.15, 0)
cframeTextBox.Position = UDim2.new(0.1, 0, 0.55, 0)
cframeTextBox.PlaceholderText = "Enter CFrame (Position & Rotation)"
cframeTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
cframeTextBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
cframeTextBox.Font = Enum.Font.SourceSans
cframeTextBox.TextSize = 16
cframeTextBox.ClearTextOnFocus = false
cframeTextBox.Parent = frame

-- Create Put Button
local putButton = Instance.new("TextButton")
putButton.Size = UDim2.new(0.8, 0, 0.15, 0)
putButton.Position = UDim2.new(0.1, 0, 0.8, 0)
putButton.Text = "Put"
putButton.TextColor3 = Color3.fromRGB(255, 255, 255)
putButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
putButton.Font = Enum.Font.SourceSans
putButton.TextSize = 16
putButton.Parent = frame

-- Save previous values
local savedTowerID = ""
local savedCFrame = ""

-- Load saved values when the GUI is opened
idTextBox.Text = savedTowerID
cframeTextBox.Text = savedCFrame

-- Function to save entered values
local function saveValues()
    savedTowerID = idTextBox.Text
    savedCFrame = cframeTextBox.Text
end

-- Function to spawn the tower with provided ID and CFrame position + rotation
putButton.MouseButton1Click:Connect(function()
    local towerID = idTextBox.Text
    local cframeInput = cframeTextBox.Text

    -- Validate CFrame input (12 numbers)
    local cframeValues = {}
    for value in string.gmatch(cframeInput, "(-?%d+%.?%d*)") do
        table.insert(cframeValues, tonumber(value))
    end

    if #cframeValues == 12 then
        -- Create CFrame with 3 for position and 9 for rotation
        local position = Vector3.new(cframeValues[1], cframeValues[2], cframeValues[3])
        local rotation = CFrame.new(
            cframeValues[1], cframeValues[2], cframeValues[3], 
            cframeValues[4], cframeValues[5], cframeValues[6], 
            cframeValues[7], cframeValues[8], cframeValues[9], 
            cframeValues[10], cframeValues[11], cframeValues[12]
        )

        -- Validate tower ID and call the server function
        if towerID ~= "" then
            local args = {
                [1] = towerID,
                [2] = rotation
            }

            local success, errorMsg = pcall(function()
                game:GetService("ReplicatedStorage").Functions.SpawnNewTower:InvokeServer(unpack(args))
            end)

            if success then
                print("Tower Spawned Successfully!")
                saveValues()  -- Save the entered values after the button is clicked
            else
                warn("Error spawning tower:", errorMsg)
            end
        else
            warn("Please enter a valid Tower ID.")
        end
    else
        warn("Invalid CFrame input. Please enter 12 numerical values (Position and Rotation).")
    end
end)

-- Add a Hide/Show button in the top-left corner
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Hide"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 200)
toggleButton.Font = Enum.Font.SourceSans
toggleButton.TextSize = 16
toggleButton.Parent = screenGui

-- Toggle GUI visibility
local isVisible = true
toggleButton.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    frame.Visible = isVisible
    toggleButton.Text = isVisible and "Hide" or "Show"
end)
