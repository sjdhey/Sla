-- AdminFly_Client.lua
-- Coloque este script em StarterPlayerScripts

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local remote = ReplicatedStorage:WaitForChild("AdminRemote")

local flying = false
local flySpeed = 60
local bv, bg
local keys = {W=false, A=false, S=false, D=false, Space=false, LShift=false}

-- Atualiza referência ao respawn
player.CharacterAdded:Connect(function(c)
    char = c
    flying = false
    if bv then bv:Destroy() bv=nil end
    if bg then bg:Destroy() bg=nil end
end)

-- Detecta teclas
UserInputService.InputBegan:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W then keys.W=true end
    if input.KeyCode == Enum.KeyCode.S then keys.S=true end
    if input.KeyCode == Enum.KeyCode.A then keys.A=true end
    if input.KeyCode == Enum.KeyCode.D then keys.D=true end
    if input.KeyCode == Enum.KeyCode.Space then keys.Space=true end
    if input.KeyCode == Enum.KeyCode.LeftShift then keys.LShift=true end
end)
UserInputService.InputEnded:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.W then keys.W=false end
    if input.KeyCode == Enum.KeyCode.S then keys.S=false end
    if input.KeyCode == Enum.KeyCode.A then keys.A=false end
    if input.KeyCode == Enum.KeyCode.D then keys.D=false end
    if input.KeyCode == Enum.KeyCode.Space then keys.Space=false end
    if input.KeyCode == Enum.KeyCode.LeftShift then keys.LShift=false end
end)

local function enableFly()
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    local humanoid = char:FindFirstChildWhichIsA("Humanoid")
    if humanoid then humanoid.PlatformStand = true end
    if not bv then
        bv = Instance.new("BodyVelocity")
        bv.MaxForce = Vector3.new(1e5,1e5,1e5)
        bv.Velocity = Vector3.new(0,0,0)
        bv.Parent = hrp
    end
    if not bg then
        bg = Instance.new("BodyGyro")
        bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
        bg.CFrame = hrp.CFrame
        bg.Parent = hrp
    end
    flying = true
end

local function disableFly()
    if char and char:FindFirstChildWhichIsA("Humanoid") then
        local humanoid = char:FindFirstChildWhichIsA("Humanoid")
        if humanoid then humanoid.PlatformStand=false end
    end
    if bv then bv:Destroy() bv=nil end
    if bg then bg:Destroy() bg=nil end
    flying = false
end

-- Movimento por frame
RunService.Heartbeat:Connect(function(dt)
    if not flying or not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    local cam = workspace.CurrentCamera
    local forward = cam.CFrame.LookVector
    local right = cam.CFrame.RightVector
    local y = 0
    local dir = Vector3.new(0,0,0)
    if keys.W then dir=dir+forward end
    if keys.S then dir=dir-forward end
    if keys.A then dir=dir-right end
    if keys.D then dir=dir+right end
    if keys.Space then y=y+1 end
    if keys.LShift then y=y-1 end
    local move = Vector3.new(dir.X,0,dir.Z)
    local finalVel = (move.Magnitude>0 and move.Unit*flySpeed or Vector3.new(0,0,0)) + Vector3.new(0,y*flySpeed*0.6,0)
    if bv then bv.Velocity = finalVel end
    if bg then bg.CFrame = CFrame.new(hrp.Position, hrp.Position+cam.CFrame.LookVector) end
end)

-- Eventos do servidor
remote.OnClientEvent:Connect(function(action)
    if action == "ToggleFly" then enableFly()
    elseif action == "ToggleFlyOff" then disableFly()
    elseif action == "ToggleFlyToggle" then
        if flying then disableFly() else enableFly() end
    elseif action == "CommandFeedback" then
        print("[AdminFeedback]", action)
    end
end)
