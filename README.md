--////////////////////////////////////////////////////////////--
--  LOADSCRIPT UNIFICADO V25 (ORIGINAL + BAGUAS INTEGRADO)    --
--  ESTRUTURA ORIGINAL V21 - TUDO RESTAURADO                  --
--////////////////////////////////////////////////////////////--

if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

task.spawn(function()
    local character, humanoid, animator, hrp, hitboxPart
    local R = ReplicatedStorage:WaitForChild("LightsaberRemotes")
    local MouseDown, Attack, Swing, MouseUp = R:WaitForChild("MouseDown"), R:WaitForChild("Attack"), R:WaitForChild("Swing"), R:WaitForChild("MouseUp")
    local Block, Unblock, ResetDir, FinishSwing, OnHit = R:WaitForChild("Block"), R:WaitForChild("Unblock"), R:WaitForChild("ResetSwingDirection"), R:WaitForChild("FinishSwing"), R:WaitForChild("OnHit")

    local enabled = false
    local flickEnabled = true 
    local bodyFlickEnabled = true 
    local guarding = false
    local currentMode = "Saber" 
    local flickDuration, flickSmooth = 0.06, 8
    local isResetting = false -- Trava para evitar hit sem animação no reset

    -- [SISTEMA BODY FLICK ORIGINAL]
    local b_yaw, b_flickActive = 0, false
    local function executeBodyFlick()
        if not bodyFlickEnabled or b_flickActive then return end
        b_flickActive = true
        if humanoid then for _, t in ipairs(humanoid:GetPlayingAnimationTracks()) do t:AdjustSpeed(3) end end
        task.delay(0.12, function() b_flickActive = false end)
    end

    local oldNameCall; oldNameCall = hookmetamethod(game, "__namecall", function(self, ...)
        if self == Attack and getnamecallmethod() == "FireServer" then task.spawn(executeBodyFlick) end
        return oldNameCall(self, ...)
    end)

    RunService.RenderStepped:Connect(function()
        if not hrp or not bodyFlickEnabled then if humanoid then humanoid.AutoRotate = true end return end
        if b_flickActive then
            humanoid.AutoRotate = false
            b_yaw = (b_yaw + 180) % 360 
            hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(b_yaw), 0)
        else
            humanoid.AutoRotate = true
            local _, y, _ = hrp.CFrame:ToEulerAnglesYXZ()
            b_yaw = math.degrees(y)
        end
    end)

    -- [PROTEÇÃO ANTI-BUG]
    local BAD = {
        ["rbxassetid://12625862519"]=true, ["rbxassetid://12625868684"]=true,
        ["rbxassetid://12625858434"]=true, ["rbxassetid://12625860439"]=true,
        ["rbxassetid://12625864413"]=true, ["rbxassetid://12625856098"]=true,
        ["rbxassetid://12625866538"]=true,
    }

    local function watchAnimator(anim)
        if not anim or anim:FindFirstChild("__HitBlocker") then return end
        Instance.new("BoolValue", anim).Name = "__HitBlocker"
        anim.AnimationPlayed:Connect(function(track)
            local id = track.Animation and track.Animation.AnimationId
            if id and BAD[id] then track:Stop() end
        end)
    end

    local function refreshCharacter()
        character = LP.Character or LP.CharacterAdded:Wait()
        humanoid  = character:WaitForChild("Humanoid")
        animator  = humanoid:WaitForChild("Animator")
        hrp       = character:WaitForChild("HumanoidRootPart")
        watchAnimator(animator)
    end
    LP.CharacterAdded:Connect(refreshCharacter); refreshCharacter()

    -- [IDs ORIGINAIS + BAGUAS]
    local saberIds = {[1]="rbxassetid://12625839385",[2]="rbxassetid://12625848489",[4]="rbxassetid://12625853257",[5]="rbxassetid://12625843823",[6]="rbxassetid://12625846167",[8]="rbxassetid://12625851115"}
    local staffIds = {[1]="rbxassetid://12718503706",[2]="rbxassetid://12718486016",[3]="rbxassetid://12718504431",[4]="rbxassetid://12718500875"}
    local formaIIds = {[1]="rbxassetid://12625848489",[2]="rbxassetid://12625853257",[3]="rbxassetid://12625851115",[4]="rbxassetid://12625846167"}
    local wizardIds = {[1]="rbxassetid://12625841878",[2]="rbxassetid://12625841878",[3]="rbxassetid://12625853257",[4]="rbxassetid://12625841878",[5]="rbxassetid://12625848489",[6]="rbxassetid://12625846167",[7]="rbxassetid://12625848489",[8]="rbxassetid://12625853257",[9]="rbxassetid://12625841878",[10]="rbxassetid://12625848489",[11]="rbxassetid://12625846167",[12]="rbxassetid://12625843823",[13]="rbxassetid://12625846167",[14]="rbxassetid://12625841878"}
    local comboZuIds = {"rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718504431","rbxassetid://12718504431","rbxassetid://12718504431","rbxassetid://12718504431"}
    local godComboIds = {"rbxassetid://12625853257", "rbxassetid://12625843823", "rbxassetid://12625846167", "rbxassetid://12625846167", "rbxassetid://12625846167", "rbxassetid://12625846167"}
    local baguasIds = {"rbxassetid://12625846167", "rbxassetid://12625848489", "rbxassetid://12625853257"}

    local function doCamera360Flick()
        if not flickEnabled then return end
        local cam = workspace.CurrentCamera
        local startCF = cam.CFrame
        local startPos, pitch, yaw = startCF.Position, startCF:ToEulerAnglesYXZ()
        coroutine.wrap(function()
            for i=1, flickSmooth do
                cam.CFrame = CFrame.new(startPos) * CFrame.fromEulerAnglesYXZ(pitch, yaw + math.rad(360)*(i/flickSmooth), 0)
                task.wait(flickDuration / flickSmooth)
            end
        end)()
    end

    local function findHitbox()
        hitboxPart = nil
        local w = character:FindFirstChild(LP.Name.."Saber1", true) or character:FindFirstChild("Staff", true) or character:FindFirstChild("Double", true)
        if w then hitboxPart = w:FindFirstChild("Hitbox", true) end
    end

    local seq, ptr, touchConn = {}, 1, nil
    local function newSeq()
        seq, ptr = {}, 1
        if currentMode == "Wizard" then for i=1,#wizardIds do table.insert(seq,i) end
        elseif currentMode == "Combo Zu" then for i=1,#comboZuIds do table.insert(seq,i) end
        elseif currentMode == "GOD Combo" then for i=1,#godComboIds do table.insert(seq,i) end
        elseif currentMode == "BAGUAS AURA" then for i=1,#baguasIds do table.insert(seq,i) end
        else
            local pool = (currentMode=="Saber" and {1,2,4,5,6,8}) or (currentMode=="Staff" and {1,2,3,4}) or (currentMode=="Forma I" and {1,2,3,4}) or {1,2,3,4}
            for i=1,#pool do table.insert(seq, table.remove(pool, math.random(#pool))) end
        end
    end

    local function guardOn() if not guarding then Block:FireServer() guarding=true end end
    local function guardOff() if guarding then Unblock:FireServer() guarding=false end end

    local function swingNext()
        if not enabled or isResetting then return end
        
        if currentMode == "GOD Combo" and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
            ptr = 1; task.wait(0.1); if enabled then swingNext() end return
        end

        if ptr > #seq then 
            isResetting = true
            task.wait(0.2) -- TEMPO DE RESET DO COMBO PARA EVITAR HIT FANTASMA
            isResetting = false
            newSeq() 
        end
        local dir = seq[ptr]
        
        local speed = (currentMode=="BAGUAS AURA" and 1.2) or (currentMode=="GOD Combo" and 1.2) or (currentMode=="Combo Zu" and 2.8) or (currentMode=="Wizard" and 1.95) or (currentMode=="Staff" and 1.8) or (currentMode=="Forma I" and 1.25) or 1.25
        
        findHitbox()
        local function getAnim(id) local a=Instance.new("Animation") a.AnimationId=id return a end
        local currentIds = (currentMode=="Saber" and saberIds) or (currentMode=="Staff" and staffIds) or (currentMode=="Forma I" and formaIIds) or (currentMode=="Wizard" and wizardIds) or (currentMode=="GOD Combo" and godComboIds) or (currentMode=="BAGUAS AURA" and baguasIds) or comboZuIds
        
        local track = animator:LoadAnimation(getAnim(currentIds[dir] or currentIds[1]))
        track:Play(0.01, 1, speed)
        
        local gotHit = false
        task.delay(0.06, doCamera360Flick)
        
        track.KeyframeReached:Connect(function(f)
            if f ~= "Start" or touchConn or not hitboxPart then return end
            touchConn = hitboxPart.Touched:Connect(function(p)
                if gotHit then return end
                local tgt = p:FindFirstAncestorWhichIsA("Model")
                if tgt and tgt ~= character and tgt:FindFirstChild("Humanoid") then
                    gotHit=true; guardOff(); MouseDown:FireServer(); Attack:FireServer(dir,1,false,false)
                    Swing:FireServer(); OnHit:FireServer(tgt); track:AdjustSpeed(-speed*1.3); guardOn()
                end
            end)
        end)
        
        track.Stopped:Connect(function()
            if touchConn then touchConn:Disconnect() touchConn=nil end
            ResetDir:FireServer(); FinishSwing:FireServer(); MouseUp:FireServer()
            ptr = ptr + 1; guardOn()
            local rTime = (currentMode=="BAGUAS AURA" and 0.05) or (currentMode=="GOD Combo" and 0.05) or (currentMode=="Combo Zu" and 0.005) or (currentMode=="Wizard" and 0.05) or 0.06
            if enabled then task.delay(rTime, swingNext) end
        end)
    end

    -- [GUI SETUP ORIGINAL]
    local gui = Instance.new("ScreenGui", LP.PlayerGui); gui.Name = "SaberUniversal"; gui.ResetOnSpawn = false
    local toggleBtn = Instance.new("TextButton", gui)
    toggleBtn.Size, toggleBtn.Position = UDim2.new(0, 70, 0, 30), UDim2.fromOffset(50, 120)
    toggleBtn.Text = "MENU"; toggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80); toggleBtn.TextColor3 = Color3.new(1,1,1); toggleBtn.Draggable, toggleBtn.Active = true, true; Instance.new("UICorner", toggleBtn)

    local autoBtn = Instance.new("TextButton", gui)
    autoBtn.Size, autoBtn.Position = UDim2.new(0, 150, 0, 40), UDim2.fromOffset(130, 115)
    autoBtn.Text = "AUTO: OFF (Y)"; autoBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 65); autoBtn.TextColor3 = Color3.new(1,1,1); autoBtn.Draggable, autoBtn.Active = true, true; Instance.new("UICorner", autoBtn)

    local win = Instance.new("Frame", gui)
    win.Size, win.Position = UDim2.new(0, 200, 0, 155), UDim2.fromOffset(50, 160)
    win.BackgroundColor3 = Color3.fromRGB(35, 35, 40); win.Active, win.Draggable = true, true; win.Visible = true; Instance.new("UICorner", win)

    local modeBtn = Instance.new("TextButton", win)
    modeBtn.Size, modeBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 10)
    modeBtn.Text = "MODE: SABER (M)"; modeBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 200); modeBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", modeBtn)

    local flickBtn = Instance.new("TextButton", win) 
    flickBtn.Size, flickBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 55)
    flickBtn.Text = "360 CAM: ON (F8)"; flickBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0); flickBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", flickBtn)

    local bodyBtn = Instance.new("TextButton", win)
    bodyBtn.Size, bodyBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 100)
    bodyBtn.Text = "BODY GIRO: ON (J)"; bodyBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0); bodyBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", bodyBtn)

    local function doToggleAuto()
        enabled = not enabled
        autoBtn.Text = "AUTO: " .. (enabled and "ON (Y)" or "OFF (Y)")
        autoBtn.BackgroundColor3 = enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 65)
        if enabled then isResetting = false; newSeq(); swingNext() end
    end

    local function doToggleMode()
        if currentMode == "Saber" then currentMode = "Staff"
        elseif currentMode == "Staff" then currentMode = "Forma I"
        elseif currentMode == "Forma I" then currentMode = "Wizard"
        elseif currentMode == "Wizard" then currentMode = "Combo Zu"
        elseif currentMode == "Combo Zu" then currentMode = "GOD Combo"
        elseif currentMode == "GOD Combo" then currentMode = "BAGUAS AURA"
        else currentMode = "Saber" end
        modeBtn.Text = "MODE: " .. currentMode:upper() .. " (M)"
        modeBtn.BackgroundColor3 = (currentMode=="BAGUAS AURA" and Color3.fromRGB(255, 120, 0)) or (currentMode=="GOD Combo" and Color3.fromRGB(255, 215, 0)) or (currentMode=="Combo Zu" and Color3.fromRGB(255, 0, 50)) or (currentMode=="Forma I" and Color3.fromRGB(150, 0, 150)) or Color3.fromRGB(0, 100, 200)
        newSeq()
    end

    toggleBtn.MouseButton1Click:Connect(function() win.Visible = not win.Visible end)
    autoBtn.MouseButton1Click:Connect(doToggleAuto)
    modeBtn.MouseButton1Click:Connect(doToggleMode)
    flickBtn.MouseButton1Click:Connect(function()
        flickEnabled = not flickEnabled
        flickBtn.Text = "360 CAM: " .. (flickEnabled and "ON (F8)" or "OFF (F8)")
        flickBtn.BackgroundColor3 = flickEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    end)
    bodyBtn.MouseButton1Click:Connect(function()
        bodyFlickEnabled = not bodyFlickEnabled
        bodyBtn.Text = "BODY GIRO: " .. (bodyFlickEnabled and "ON (J)" or "OFF (J)")
        bodyBtn.BackgroundColor3 = bodyFlickEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
    end)

    UserInputService.InputBegan:Connect(function(input, p)
        if p then return end
        if input.KeyCode == Enum.KeyCode.Y then doToggleAuto()
        elseif input.KeyCode == Enum.KeyCode.M then doToggleMode()
        elseif input.KeyCode == Enum.KeyCode.F8 then flickBtn:Click()
        elseif input.KeyCode == Enum.KeyCode.J then bodyBtn:Click() end
    end)
end)
