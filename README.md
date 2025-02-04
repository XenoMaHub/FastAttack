print("รันหาพ่อง")



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
    local Hum = CachedChars[Char] or Char:FindFirstChildOfClass("Humanoid")
    if Hum then
        CachedChars[Char] = Hum
        return Hum.Health > 0
    end
    return false
end

local Settings = {
    ClickDelay = 0.01,
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
                RegisterAttack:FireServer(Settings.ClickDelay or 0)
                self.FirstAttack = true
            end
            RegisterHit:FireServer(EnemyHead, Table or {})
        end
    end

    function AttackModule:AttackNearest()
        local args = {nil, {}}
        for _, Enemy in ipairs(enemyFolder:GetChildren()) do
            local humanoidPart = Enemy:FindFirstChild("HumanoidRootPart")
            if humanoidPart and client:DistanceFromCharacter(humanoidPart.Position) < self.Distance then
                local upperTorso = Enemy:FindFirstChild("UpperTorso")
                if not args[1] then
                    args[1] = upperTorso
                else
                    table.insert(args[2], {Enemy, upperTorso})
                end
            end
        end

        self:AttackEnemy(unpack(args))

        for _, Char in ipairs(charFolder:GetChildren()) do
            if Char ~= client.Character then
                self:AttackEnemy(Char:FindFirstChild("UpperTorso"))
            end
        end

        if not self.FirstAttack then
            task.wait(0.01)
        end
    end

    function AttackModule:BladeHits()
        self:AttackNearest()
        self.FirstAttack = false
    end

    task.spawn(function()
        while task.wait(Settings.ClickDelay or 0) do
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
