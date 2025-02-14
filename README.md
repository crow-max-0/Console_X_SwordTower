local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ChatRemote = ReplicatedStorage:WaitForChild("DefaultChatSystemChatEvents")
local SayMessageRequest = ChatRemote:WaitForChild("SayMessageRequest")

-- ScreenGui 생성
local gui = Instance.new("ScreenGui")
gui.Name = "MobileHub"
gui.ResetOnSpawn = false
gui.Parent = playerGui

-- MainFrame (허브 UI 전체)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 180)
mainFrame.Position = UDim2.new(0.5, -125, 0.4, -90)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = gui

-- TitleBar (드래그 가능 영역)
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 30)
titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

-- 허브 UI 내용 예제 (텍스트)
local textLabel = Instance.new("TextLabel")
textLabel.Size = UDim2.new(1, 0, 1, -30)
textLabel.Position = UDim2.new(0, 0, 0, 30)
textLabel.Text = "Made by crow_0_inf"
textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
textLabel.BackgroundTransparency = 1
textLabel.Parent = mainFrame

-- 버튼 생성 함수
local function createButton(text, position, action)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.4, 0, 0, 30)
    button.Position = position
    button.Text = text
    button.TextColor3 = Color3.fromRGB(0, 0, 0)
    button.TextSize = 18
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) 
    button.Parent = mainFrame

    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 10)
    buttonCorner.Parent = button

    if action then
        button.MouseButton1Click:Connect(action)
    end

    return button
end

-- "검 올킬" 버튼 (기존 기능)
local isSwordKillRunning = false

local function swordKill()
    local player = game:GetService("Players").LocalPlayer

    local function isFriendWith(player1, player2)
        return player1:IsFriendsWith(player2.UserId)
    end

    if isSwordKillRunning then
        isSwordKillRunning = false
        swordKillButton.Text = "검 올킬"
        swordKillButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- 빨간색 버튼으로 변경
        return
    else
        isSwordKillRunning = true
        swordKillButton.Text = "검 올킬 중지"
        swordKillButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- 초록색 버튼으로 변경
    end

    game:GetService("RunService").RenderStepped:Connect(function()
        local players = game.Players:GetPlayers()
        for i = 2, #players do
            local otherPlayer = players[i]
            local character = otherPlayer.Character

            if not isFriendWith(player, otherPlayer) then
                local tool = player.Character and player.Character:FindFirstChildOfClass("Tool")
                if tool and tool:FindFirstChild("Handle") then
                    tool:Activate()
                    for _, part in next, character:GetChildren() do
                        if part:IsA("BasePart") then
                            firetouchinterest(tool.Handle, part, 0)
                            firetouchinterest(tool.Handle, part, 1)
                        end
                    end
                end
            end
        end
    end)
end

-- "도배" 버튼 (홍보 스크립트)
local isRunning = false
local messages = {
    "콘솔x화이팅!",
    "콘솔x구독ㄱㄱ",
    "크로우의스크립트서버가입ㄱㄱ",
    "당장가서스크받자!",
    "ㅋㅋㅋ안가면은손해임"
}

local function togglePromotion()
    if isRunning then
        isRunning = false
        spamButton.Text = "도배 시작"
        spamButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- 다시 빨간색으로 변경
    else
        isRunning = true
        spamButton.Text = "도배 중지"
        spamButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- 초록색으로 변경
        spawn(function()
            while isRunning do
                local message = messages[math.random(#messages)] -- 랜덤 메시지 선택
                SayMessageRequest:FireServer(message, "All")
                wait(1) -- 1초 대기
            end
        end)
    end
end

-- "인피니티야드" 버튼 (InfiniteYield 실행)
local function runInfiniteYield()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source', true))()
end

-- 버튼 배치
swordKillButton = createButton("검 올킬", UDim2.new(0.05, 0, 0.2, 0), swordKill)  
spamButton = createButton("도배 시작", UDim2.new(0.05, 0, 0.75, 0), togglePromotion)  
createButton("인피니티야드", UDim2.new(0.55, 0, 0.75, 0), runInfiniteYield)

-- 톱니 모양 버튼 생성 (허브 보이기/숨기기)
local settingsButton = Instance.new("TextButton")
settingsButton.Size = UDim2.new(0, 50, 0, 50)
settingsButton.Position = UDim2.new(0, 10, 0.5, -25)
settingsButton.Text = "⚙️"
settingsButton.TextColor3 = Color3.fromRGB(255, 255, 255)
settingsButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
settingsButton.Parent = gui

-- 허브 보이기/숨기기
mainFrame.Visible = false
settingsButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

-- 드래그 기능 추가
local UserInputService = game:GetService("UserInputService")
local dragging, dragInput, dragStart, startPos

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

titleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        local newX = startPos.X.Offset + delta.X
        local newY = startPos.Y.Offset + delta.Y

        mainFrame.Position = UDim2.new(startPos.X.Scale, newX, startPos.Y.Scale, newY)
    end
end)
