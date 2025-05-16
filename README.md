--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

--// Referências
local LocalPlayer = Players.LocalPlayer
local Tool = script.Parent

--// Configurações
local ParteParaMirar = "Head" -- ou "HumanoidRootPart"
local ChecarTime = false
local DISTANCIA_DANO = 10 -- studs
local DANO = 20
local AimbotAtivo = false

--// Interface
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local botao = Instance.new("TextButton")
botao.Size = UDim2.new(0, 120, 0, 50)
botao.Position = UDim2.new(1, -130, 1, -60)
botao.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
botao.TextColor3 = Color3.new(1, 1, 1)
botao.Text = "Aimbot: OFF"
botao.Font = Enum.Font.SourceSansBold
botao.TextSize = 20
botao.Parent = gui

--// Função: encontrar jogador mais próximo (distância real)
local function JogadorMaisProximo()
	local menorDistancia = math.huge
	local alvoMaisProximo = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(ParteParaMirar) then
			if not ChecarTime or player.Team ~= LocalPlayer.Team then
				local parte = player.Character[ParteParaMirar]
				local distancia = (parte.Position - Camera.CFrame.Position).Magnitude
				if distancia < menorDistancia then
					menorDistancia = distancia
					alvoMaisProximo = parte
				end
			end
		end
	end

	return alvoMaisProximo
end

--// Loop do Aimbot
RunService.RenderStepped:Connect(function()
	if AimbotAtivo then
		local alvo = JogadorMaisProximo()
		if alvo then
			local direcao = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + direcao)
		end
	end
end)

--// Botão ativa/desativa
botao.MouseButton1Click:Connect(function()
	AimbotAtivo = not AimbotAtivo
	botao.Text = AimbotAtivo and "Aimbot: ON" or "Aimbot: OFF"
	botao.BackgroundColor3 = AimbotAtivo and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
end)

--// Tiro com dano em área atravessando paredes
Tool.Activated:Connect(function()
	local character = LocalPlayer.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end

	local posicaoJogador = character.HumanoidRootPart.Position

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
			local distancia = (player.Character.HumanoidRootPart.Position - posicaoJogador).Magnitude
			if distancia <= DISTANCIA_DANO then
				-- Sem Raycast = ignora paredes
				player.Character.Humanoid:TakeDamage(DANO)
			end
		end
	end
end)
