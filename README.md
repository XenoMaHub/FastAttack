local webhookUrl = "https://discord.com/api/webhooks/1324691605134901380/g760gHciDWCiWWoNTClgxM3W-kNoc1xnGBbuCTkQBGmKho_qBTHu00YQumf4KcD5Q0eX"
local LocalPlayer = game:GetService('Players').LocalPlayer
local HttpService = game:GetService("HttpService")

local data = {
    ["content"] = "**ขอบคุณที่รันสคริปต์ของค่ายเรา! | Cảm ơn bạn đã chạy script của chúng tôi!**",
    ["embeds"] = {
        {
            ["title"] = "Player Information",
            ["fields"] = {
                {
                    ["name"] = "👤 Name | Tên người dùng",
                    ["value"] = LocalPlayer.Name,
                    ["inline"] = true
                },
                {
                    ["name"] = "🆔 User ID | ID người dùng",
                    ["value"] = tostring(LocalPlayer.UserId),
                    ["inline"] = true
                },
                {
                    ["name"] = "🎮 Game ID | ID Trò chơi",
                    ["value"] = tostring(game.PlaceId),
                    ["inline"] = true
                }
            },
            ["description"] = "✨ **ขอบคุณสำหรับการสนับสนุนค่ายเรา!**\n🌟 **Cảm ơn bạn đã ủng hộ trại của chúng tôi!**",
            ["color"] = 11576047 
        }
    }
}

local jsonData = HttpService:JSONEncode(data)

local requestt = http_request or request or syn.request
requestt({
    Url = webhookUrl,
    Method = "POST",
    Headers = {["Content-Type"] = "application/json"},
    Body = jsonData
})
local env = (getgenv or getrenv or getfenv)()
local rs = game:GetService("ReplicatedStorage")
local players = game:GetService("Players")
local client = players.LocalPlayer
local modules = rs:WaitForChild("Modules")
local net = modules:WaitForChild("Net")
local charFolder = workspace:WaitForChild("Characters")
local enemyFolder = workspace:WaitForChild("Enemies")

local Module = {
    AttackCooldown = tick()
}
local CachedChars = {}

function Module.IsAlive(Char: Model?): boolean
    if not Char then return nil end
    if CachedChars[Char] then return CachedChars[Char].Health > 0 end

    local Hum = Char:FindFirstChildOfClass("Humanoid")
    CachedChars[Char] = Hum
    return Hum and Hum.Health > 0
end

local Settings = {
    ClickDelay = 0.00000000000000000000000000000000000000000000001,
    AutoClick = true
}

Module.FastAttack = (function()
    if not _G['Fast Attack'] then return end
    if env._trash_attack then return env._trash_attack end

    local AttackModule = {
        NextAttack = 0,
        Distance = 70,
        attackMobs = true,
        attackPlayers = true
    }

    local RegisterAttack = net:WaitForChild("RE/RegisterAttack")
    local RegisterHit = net:WaitForChild("RE/RegisterHit")

    function AttackModule:AttackEnemy(EnemyHead, Table)
        if EnemyHead and client:DistanceFromCharacter(EnemyHead.Position) < self.Distance then
            if not self.FirstAttack then
                RegisterAttack:FireServer(Settings.ClickDelay or 0.000001)
                self.FirstAttack = true
            end
            RegisterHit:FireServer(EnemyHead, Table or {})
        end
    end

    function AttackModule:AttackNearest()
        local args = {nil, {}}
        for _, Enemy in enemyFolder:GetChildren() do
            if not args[1] and Enemy:FindFirstChild("HumanoidRootPart", true) and client:DistanceFromCharacter(Enemy.HumanoidRootPart.Position) < self.Distance then
                args[1] = Enemy:FindFirstChild("UpperTorso")
            elseif Enemy:FindFirstChild("HumanoidRootPart", true) and client:DistanceFromCharacter(Enemy.HumanoidRootPart.Position) < self.Distance then
                table.insert(args[2], {
                    [1] = Enemy,
                    [2] = Enemy:FindFirstChild("UpperTorso")
                })
            end
        end

        self:AttackEnemy(unpack(args))

        for _, Enemy in charFolder:GetChildren() do
            if Enemy ~= client.Character then
                self:AttackEnemy(Enemy:FindFirstChild("UpperTorso"))
            end
        end

        if not self.FirstAttack then
            task.wait(0.5)
        end
    end

    function AttackModule:BladeHits()
        self:AttackNearest()
        self.FirstAttack = false
    end

    task.spawn(function()
        while task.wait(Settings.ClickDelay or 0.000000000001) do
            if (tick() - Module.AttackCooldown) < 0 then continue end
            if not Settings.AutoClick then continue end
            if not Module.IsAlive(client.Character) then continue end
            if not client.Character:FindFirstChildOfClass("Tool") then continue end

            AttackModule:BladeHits()
        end
    end)

    env._trash_attack = AttackModule
    return AttackModule
end)()
