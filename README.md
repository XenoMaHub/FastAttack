    local env = (getgenv or getrenv or getfenv)()
    local rs = game:GetService("ReplicatedStorage")
    local players = game:GetService("Players")
    local client = players.LocalPlayer
    local modules = rs:WaitForChild("Modules")
    local net = modules:WaitForChild("Net")
    local charFolder = workspace:WaitForChild("Characters")
    local enemyFolder = workspace:WaitForChild("Enemies")

    local Settings = {
        ClickDelay = 0.000000000000001,
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
                    RegisterAttack:FireServer(Settings.ClickDelay or 0.125)
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
            while task.wait(Settings.ClickDelay or 0.125) do
                if (tick() - Module.AttackCooldown) >= 0.483 
                    and Settings.AutoClick 
                    and Module.IsAlive(client.Character) 
                    and client.Character:FindFirstChildOfClass("Tool") then

                    AttackModule:BladeHits()
                end
            end
        end)

        env._trash_attack = AttackModule
        return AttackModule
    end)()
