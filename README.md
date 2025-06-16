-- 👁️‍🗨️ GUI EXECUTOR (DRAG + OBFS)
local CoreGui = game:GetService("CoreGui")
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "ExGui_"..tostring(math.random(1000,9999))

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 400, 0, 300)
Main.Position = UDim2.new(0.5, -200, 0.5, -150)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.Active = true
Main.Draggable = true

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "⚙️ Secure Executor GUI"
Title.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)

local ScriptBox = Instance.new("TextBox", Main)
ScriptBox.Size = UDim2.new(1, -20, 0, 160)
ScriptBox.Position = UDim2.new(0, 10, 0, 40)
ScriptBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ScriptBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ScriptBox.ClearTextOnFocus = false
ScriptBox.MultiLine = true
ScriptBox.Text = "-- Masukkan script Lua kamu"
ScriptBox.TextXAlignment = Enum.TextXAlignment.Left
ScriptBox.TextYAlignment = Enum.TextYAlignment.Top

local Inject = Instance.new("TextButton", Main)
Inject.Text = "Inject"
Inject.Size = UDim2.new(0, 80, 0, 30)
Inject.Position = UDim2.new(0, 10, 0, 210)

local Execute = Instance.new("TextButton", Main)
Execute.Text = "Execute"
Execute.Size = UDim2.new(0, 80, 0, 30)
Execute.Position = UDim2.new(0, 100, 0, 210)

local Clear = Instance.new("TextButton", Main)
Clear.Text = "Clear"
Clear.Size = UDim2.new(0, 80, 0, 30)
Clear.Position = UDim2.new(0, 190, 0, 210)

local Close = Instance.new("TextButton", Main)
Close.Text = "Close"
Close.Size = UDim2.new(0, 80, 0, 30)
Close.Position = UDim2.new(0, 280, 0, 210)

local Status = Instance.new("TextLabel", Main)
Status.Size = UDim2.new(1, -20, 0, 20)
Status.Position = UDim2.new(0, 10, 1, -30)
Status.Text = "Status: Belum Inject"
Status.TextColor3 = Color3.new(1, 1, 1)
Status.BackgroundTransparency = 1
Status.TextXAlignment = Enum.TextXAlignment.Left

-- 🛡️ ANTI KICK & SERVER DISTRACTOR
local mt = getrawmetatable(game)
setreadonly(mt, false)
local old = mt.__namecall
local remote = nil
local suspectedRemotes = {}

mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    local method = getnamecallmethod()

    if tostring(method):lower() == "kick" then
        warn("⚠️ Kick dicegah! Menyerang balik server...")

        spawn(function()
            for i = 1, 3 do
                pcall(function()
                    game:GetService("ReplicatedStorage"):FireServer("OpenMenu", "Shop")
                    wait(0.5)
                end)
            end
        end)

        pcall(function()
            game.Players.LocalPlayer.Kick = function()
                print("Server tidak bisa kick.")
            end
        end)

        return
    end

    -- Pantau RemoteEvent yang mungkin dipakai untuk kick
    if typeof(self) == "Instance" and self:IsA("RemoteEvent") then
        if typeof(args[1]) == "string" and args[1]:lower():find("kick") then
            table.insert(suspectedRemotes, self)
            warn("🚨 Terindikasi remote kick: ", self.Name)
        end
    end

    return old(self, ...)
end)

hookfunction(game.Players.LocalPlayer.Kick, function()
    warn("⚠️ Kick dicegah (HookFunction)")
end)

-- Loop distractor remote
spawn(function()
    while true do
        for _, r in pairs(suspectedRemotes) do
            pcall(function()
                r:FireServer("Ping")
                r:FireServer("RequestHelp", "PlayerLost")
            end)
        end
        wait(2)
    end
end)

-- 🧠 Remote Scanner + Inject Handler
Inject.MouseButton1Click:Connect(function()
    Status.Text = "Status: Scanning Remote..."
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("RemoteEvent") then
            pcall(function()
                v:FireServer("loadstring('print(\"scan\")')()")
            end)
            wait(0.15)
            if not remote then
                remote = v
                Status.Text = "✅ Remote ditemukan: " .. v.Name
            end
        end
    end
    if not remote then
        Status.Text = "❌ Remote tidak ditemukan"
    end
end)

-- 🚀 Execute ke server
Execute.MouseButton1Click:Connect(function()
    if not remote then
        Status.Text = "❗ Belum Inject"
        return
    end

    local payload = ScriptBox.Text
    Status.Text = "Status: Kirim dummy dulu..."
    remote:FireServer("loadstring([[-- disguise dummy]])()")
    wait(1.5)

    Status.Text = "Status: Menjalankan script..."
    pcall(function()
        remote:FireServer("loadstring([["..payload.."]])()")
    end)
    Status.Text = "✅ Script dikirim!"
end)

-- 🧽 Clear
Clear.MouseButton1Click:Connect(function()
    ScriptBox.Text = ""
end)

-- ❌ Close
Close.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)
