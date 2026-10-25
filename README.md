-- ═══════════════════════════════════════════════════════════════
-- SERTANEJO HUB | MEC BR | v20 (Real Executor Fix)
-- by Lancenickel5 | Key: sertanejovip5
-- ═══════════════════════════════════════════════════════════════
print("[SERTANEJO v20] Boot...")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

local HUB_NAME, HUB_TAG, CREATOR = "SERTANEJO HUB", "MEC BR", "Lancenickel5"
local HUB_KEY, HUB_VER = "sertanejovip5", "v20"
local DISCORD_CONTACT = "SEU_DISCORD_AQUI"

-- Proteções
local PROT = { antikick=false, antireport=false, rejoin=true, antiafk=true, kicks=0 }
pcall(function()
    if hookfunction and LocalPlayer.Kick then
        hookfunction(LocalPlayer.Kick, function(self, msg) PROT.kicks=PROT.kicks+1; return end)
        PROT.antikick = true
    end
end)
pcall(function()
    if getrawmetatable and setreadonly and newcclosure then
        local mt = getrawmetatable(game)
        local oldNC = mt.__namecall
        setreadonly(mt, false)
        mt.__namecall = newcclosure(function(self, ...)
            local m = getnamecallmethod()
            if m == "Kick" and self == LocalPlayer then PROT.kicks=PROT.kicks+1; return end
            if type(m)=="string" and m:lower():find("report") then return end
            return oldNC(self, ...)
        end)
        setreadonly(mt, true)
        PROT.antikick = true; PROT.antireport = true
    end
end)
task.spawn(function()
    pcall(function()
        LocalPlayer.AncestryChanged:Connect(function()
            if not LocalPlayer.Parent and PROT.rejoin then
                task.wait(1.5)
                pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer) end)
            end
        end)
    end)
end)
task.spawn(function()
    pcall(function()
        local vu = Instance.new("VirtualUser")
        vu.Parent = game:GetService("VirtualUser")
        LocalPlayer.Idled:Connect(function()
            if PROT.antiafk then vu:CaptureController(); vu:ClickButton2(Vector2.new()) end
        end)
    end)
end)

-- Config
local VELOCIDADE, ALTURA_VOO, PASSOS_POR_SEG = 380, 150, 10
local CANCELAR_TECLA = Enum.KeyCode.K
local ALTURA_SEGURA_SOLO, DESCIDA_PASSOS = 5, 100
local ANTI_STUCK_TIMEOUT, TIMEOUT_VOO, COOLDOWN_TP = 0.9, 900, 1.0
local MAX_PARTS, RAIO_ATRACAO, TAM_MIN, TAM_MAX = 25, 250, 1, 40
local INTERVALO_SCAN, RAIO_GRUDE, ALTURA_MIN, ALTURA_MAX = 1.5, 5, -3, 6
local DELIVERY_THRESHOLDS = {40000, 60000, 80000, 100000, 150000}
local DELIVERY_MIN_IDX = 1
local AUTO_ACCEPT = true

local SPEED_ATIVO, SPEED_VAL = false, 50
local JUMP_ATIVO, JUMP_VAL = false, 100
local FLY_ATIVO = false
local NOCLIP_ATIVO = false
local ESP_ATIVO = false
local ESP_NAME = true
local ESP_HEALTH = true
local INFINITE_JUMP = false
local CLICK_SPAM = false
local CHAT_SPAM = false
local CHAT_SPAM_MSG = "Sertanejo Hub"
local FULLBRIGHT = false
local flyBodyVel
local currentESP = {}

local lightingOrig = {
    Brightness = Lighting.Brightness,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    ClockTime = Lighting.ClockTime,
    GlobalShadows = Lighting.GlobalShadows,
    FogEnd = Lighting.FogEnd,
}

local NOMES_CAIXA = {"box","caixa","crate","chest","barrel","bau","baú","cargo","container","package","supply","loot","treasure"}
local NOMES_ENTREGA = {"deliver","delivery","entrega","entregar","contract","contrato","frete","freight","mission","job","task","order","pedido","aceitar","accept"}
local ATTRS_VALOR = {"Value","Price","Reward","Payment","Valor","Pagamento","Dinheiro","Money","Cash","Amount","Payout","Ganho","Lucro","Bounty"}

local LOCATIONS = {
    {nome="lugar de trabalho", pos=Vector3.new(-25676.6,-28.9,-5871.8)},
    {nome="Mectropoly posto", pos=Vector3.new(-3223.1,4.2,-3711.9)},
    {nome="Valley Drag", pos=Vector3.new(-3854.4,2.9,-4898.4)},
    {nome="auto peças", pos=Vector3.new(-3327.9,65.7,-3421.9)},
    {nome="inicio", pos=Vector3.new(-3131.1,68.7,-2841.3)},
    {nome="concessionaria", pos=Vector3.new(-3079.5,65.5,-3688.1)},
    {nome="ferro velho", pos=Vector3.new(-3146.8,65.7,-4245.6)},
    {nome="costruçao", pos=Vector3.new(-3618.0,65.2,-2508.0)},
    {nome="zona de entrega", pos=Vector3.new(-3180.2,65.4,-3716.3)},
}

-- Tema
local THEME = {
    glass=Color3.fromRGB(22,24,34), glass2=Color3.fromRGB(30,33,46),
    glass3=Color3.fromRGB(40,44,60), stroke=Color3.fromRGB(56,60,80),
    accent=Color3.fromRGB(255,128,0), accent2=Color3.fromRGB(255,60,95),
    text=Color3.fromRGB(245,247,255), textDim=Color3.fromRGB(140,145,170),
    textMuted=Color3.fromRGB(88,92,112), success=Color3.fromRGB(0,220,130),
    danger=Color3.fromRGB(250,70,90), info=Color3.fromRGB(80,170,255),
    gold=Color3.fromRGB(255,205,60), warn=Color3.fromRGB(255,175,40),
    discord=Color3.fromRGB(88,101,242), purple=Color3.fromRGB(180,100,255),
}

-- UI Helpers
local function uiCorner(p,r) local c=Instance.new("UICorner");c.CornerRadius=UDim.new(0,r or 10);c.Parent=p;return c end
local function uiStroke(p,c,t,tr) local s=Instance.new("UIStroke");s.Color=c or THEME.stroke;s.Thickness=t or 1;s.Transparency=tr or 0;s.Parent=p;return s end
local function uiGrad(p,c1,c2,r) local g=Instance.new("UIGradient");g.Color=ColorSequence.new(c1,c2);g.Rotation=r or 0;g.Parent=p;return g end
local function pressAnim(btn)
    local orig = btn.Size
    btn.MouseButton1Down:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.08), {Size=UDim2.new(orig.X.Scale*0.96,orig.X.Offset*0.96,orig.Y.Scale*0.92,orig.Y.Offset*0.92)}):Play()
    end)
    btn.MouseButton1Up:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.18, Enum.EasingStyle.Back), {Size=orig}):Play()
    end)
end
local function hoverAnim(btn, base, hover, strokeRef)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3=hover}):Play()
        if strokeRef then TweenService:Create(strokeRef, TweenInfo.new(0.15), {Color=THEME.accent, Transparency=0.2}):Play() end
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3=base}):Play()
        if strokeRef then TweenService:Create(strokeRef, TweenInfo.new(0.15), {Color=THEME.stroke, Transparency=0}):Play() end
    end)
end
local function fmtVal(v)
    if not v then return "?" end
    local s = tostring(math.floor(v))
    local out = s:reverse():gsub("(%d%d%d)","%1."):reverse()
    if out:sub(1,1)=="." then out=out:sub(2) end
    return out
end

-- Estado
local viajando, cancelarViagem, ultimoTP, marcaAtual, estadoSalvo = false,false,0,nil,nil
local IMA_ATIVO = false
local grudadas, candidatos, grudadoSet = {},{},{}
local imaProgressFill, imaCount, statusDot, statusLabel, imaKnob, imaToggle, imaToggleLbl
local deliveryCountLbl, deliveryMaxLbl, thresholdBtn, autoAcceptKnob
local dbgBtn, tabIndicator
local DEBUG = false

-- Persistência
local function carregarSalvos()
    local ok,res = pcall(function()
        local s = LocalPlayer:GetAttribute("Sertanejo_Salvos") or ""
        if s=="" then return {} end
        local t={}
        for i,v in ipairs(string.split(s,"|")) do
            if v~="" then
                local x,y,z = v:match("([^,]+),([^,]+),([^,]+)")
                if x then t[i]=Vector3.new(tonumber(x),tonumber(y),tonumber(z)) end
            end
        end
        return t
    end)
    return ok and res or {}
end
local function salvarSalvos()
    local p={}
    for i=4,#LOCATIONS do
        local v=LOCATIONS[i].pos
        if v then table.insert(p,("%.1f,%.1f,%.1f"):format(v.X,v.Y,v.Z)) else table.insert(p,"") end
    end
    pcall(function() LocalPlayer:SetAttribute("Sertanejo_Salvos",table.concat(p,"|")) end)
end
do
    local s = carregarSalvos()
    for i,v in ipairs(s) do if LOCATIONS[i+3] then LOCATIONS[i+3].pos=v end end
end

-- TP
local function getRoot() local c=LocalPlayer.Character;return c and c:FindFirstChild("HumanoidRootPart") end
local function getHumanoid() local c=LocalPlayer.Character;return c and c:FindFirstChildOfClass("Humanoid") end
local function sairDoCarro()
    local h=getHumanoid()
    if h and h.SeatPart then pcall(function() h.Sit=false;h:ChangeState(Enum.HumanoidStateType.Jumping) end);task.wait(0.15) end
end
local function congelar()
    local h,r=getHumanoid(),getRoot()
    if not h or not r then return end
    estadoSalvo={walkspeed=h.WalkSpeed,jumppower=h.JumpPower,jumpheight=h.JumpHeight,platform=h.PlatformStand}
    h.WalkSpeed,h.JumpPower,h.JumpHeight=0,0,0; h.PlatformStand=true
    r.AssemblyLinearVelocity,r.AssemblyAngularVelocity=Vector3.zero,Vector3.zero
end
local function descongelar()
    local h,r=getHumanoid(),getRoot()
    if r then r.AssemblyLinearVelocity,r.AssemblyAngularVelocity=Vector3.zero,Vector3.zero end
    if not h then estadoSalvo=nil return end
    local e=estadoSalvo or {walkspeed=16,jumppower=50,platform=false}
    pcall(function()
        h.WalkSpeed=e.walkspeed; h.JumpPower=e.jumppower
        if e.jumpheight then h.JumpHeight=e.jumpheight end
        h.PlatformStand=false
        h:ChangeState(Enum.HumanoidStateType.Landing)
        h:ChangeState(Enum.HumanoidStateType.Freefall)
        task.wait(0.05)
        h:ChangeState(Enum.HumanoidStateType.Landed)
    end)
    estadoSalvo=nil
end
local function getGroundY(pos)
    local params=RaycastParams.new()
    params.FilterType,params.IgnoreWater,params.RespectCanCollide=Enum.RaycastFilterType.Exclude,true,true
    local ex={}
    local c=LocalPlayer.Character
    if c then table.insert(ex,c) end
    if marcaAtual and marcaAtual.Parent then table.insert(ex,marcaAtual) end
    params.FilterDescendantsInstances=ex
    local r=workspace:Raycast(pos+Vector3.new(0,100,0),Vector3.new(0,-800,0),params)
    if r then return r.Position.Y end
    r=workspace:Raycast(pos+Vector3.new(0,20,0),Vector3.new(0,-200,0),params)
    return r and r.Position.Y or nil
end
local function criarMarca(destino)
    if marcaAtual and marcaAtual.Parent then marcaAtual:Destroy() end
    local p=Instance.new("Part")
    p.Shape,p.Size=Enum.PartType.Ball,Vector3.new(2,2,2)
    p.Position=destino+Vector3.new(0,6,0)
    p.Anchored,p.CanCollide,p.CanQuery=true,false,false
    p.Material,p.Color,p.Transparency=Enum.Material.Neon,THEME.accent,0.3
    p.Name="Sertanejo_MarcaTP"; p.Parent=workspace
    marcaAtual=p
    task.delay(30,function() if p.Parent then p:Destroy() end;if marcaAtual==p then marcaAtual=nil end end)
end
local function andarAte(alvo,timeout)
    local ini,up,tp=os.clock(),nil,0
    local iv=1/PASSOS_POR_SEG
    local max=VELOCIDADE*iv
    timeout=timeout or TIMEOUT_VOO
    while true do
        if cancelarViagem then return false,"cancelado" end
        if os.clock()-ini>timeout then return false,"timeout" end
        local r=getRoot()
        if not r then return false,"sem_root" end
        local a=r.Position
        if up and (a-up).Magnitude<0.15 then
            tp=tp+iv
            if tp>=ANTI_STUCK_TIMEOUT then r.CFrame=r.CFrame+Vector3.new(0,6,0);tp=0 end
        else tp=0 end
        up=a
        local d=alvo-a
        if d.Magnitude<max then r.CFrame=CFrame.new(alvo)*(r.CFrame-r.CFrame.Position);return true end
        r.CFrame=r.CFrame+d.Unit*max
        r.AssemblyLinearVelocity=Vector3.zero
        task.wait(iv)
    end
end
local function tpAndando(destino)
    if viajando then return false,"viajando" end
    if os.clock()-ultimoTP<COOLDOWN_TP then return false,"aguarde" end
    if not destino then return false,"vazio" end
    viajando,cancelarViagem,ultimoTP=true,false,os.clock()
    task.spawn(function()
        local ok,mot=false,"?"
        local s,err=pcall(function()
            sairDoCarro()
            local r=getRoot()
            if not r then mot="sem_char" return end
            criarMarca(destino);congelar()
            local s1,m1=andarAte(Vector3.new(r.Position.X,destino.Y+ALTURA_VOO,r.Position.Z))
            if not s1 then mot=m1 return end
            local s2,m2=andarAte(Vector3.new(destino.X,destino.Y+ALTURA_VOO,destino.Z))
            if not s2 then mot=m2 return end
            local r2=getRoot()
            if not r2 then mot="sem_char" return end
            local gy=getGroundY(destino)
            local af=gy and Vector3.new(destino.X,gy+ALTURA_SEGURA_SOLO,destino.Z) or (destino+Vector3.new(0,ALTURA_SEGURA_SOLO,0))
            local at=r2.Position
            for i=1,DESCIDA_PASSOS do
                if cancelarViagem then mot="cancelado" return end
                local ok2,m=andarAte(at:Lerp(af,i/DESCIDA_PASSOS),5)
                if not ok2 then mot=m return end
            end
            ok,mot=true,"ok"
        end)
        if not s then mot="err:"..tostring(err);warn("[TP] "..mot) end
        descongelar();viajando=false
        _G.Sertanejo_TP_Status=ok and "Chegou!" or ("Erro: "..mot)
    end)
    return true
end

-- Ímã
local function pareceCaixa(p)
    local n=p.Name:lower()
    for _,k in ipairs(NOMES_CAIXA) do if n:find(k,1,true) then return true end end
    local pa=p.Parent
    if pa and pa~=workspace then
        local pn=pa.Name:lower()
        for _,k in ipairs(NOMES_CAIXA) do if pn:find(k,1,true) then return true end end
    end
    return false
end
local function soltarObjeto(g)
    if not g then return end
    if g.weld then pcall(function() g.weld:Destroy() end) end
    if g.part and g.part.Parent then pcall(function() g.part.CanCollide,g.part.Massless,g.part.Anchored=true,false,false end) end
    if g.part then grudadoSet[g.part]=nil end
end
local function atualizarProgresso()
    if not imaProgressFill or not imaCount then return end
    local pct=math.clamp(#grudadas/MAX_PARTS,0,1)
    TweenService:Create(imaProgressFill,TweenInfo.new(0.3),{Size=UDim2.new(pct,0,1,0)}):Play()
    imaCount.Text=("%d / %d caixas"):format(#grudadas,MAX_PARTS)
end
local function soltarTudo()
    for _,g in ipairs(grudadas) do soltarObjeto(g) end
    grudadas,grudadoSet={},{}
    atualizarProgresso()
end
local function calcularOffset(i,t)
    local phi=math.pi*(3-math.sqrt(5))
    local y=1-(i/math.max(t,1))*2
    local raio=math.sqrt(math.max(0,1-y*y))
    local th=phi*i
    local x,z=math.cos(th)*raio,math.sin(th)*raio
    local am=(ALTURA_MIN+ALTURA_MAX)/2
    local ar=(ALTURA_MAX-ALTURA_MIN)/2
    return Vector3.new(x*RAIO_GRUDE,am+y*ar,z*RAIO_GRUDE)
end
local function grudarPart(p,r)
    if not p.Parent or grudadoSet[p] then return false end
    local t=p.Size.X+p.Size.Y+p.Size.Z
    if t<TAM_MIN or t>TAM_MAX then return false end
    if (p.Position-r.Position).Magnitude>RAIO_ATRACAO then return false end
    if p:FindFirstChildOfClass("WeldConstraint") then return false end
    if p:FindFirstChildOfClass("AlignPosition") then return false end
    for _,j in ipairs(p:GetJoints()) do
        if j:IsA("Weld") or j:IsA("WeldConstraint") or j:IsA("Motor6D") then j:Destroy() end
    end
    p.CanCollide,p.Massless=false,true
    p.CFrame=r.CFrame*CFrame.new(calcularOffset(#grudadas+1,MAX_PARTS))
    local w=Instance.new("WeldConstraint")
    w.Part0,w.Part1,w.Parent=r,p,p
    grudadoSet[p]=true
    table.insert(grudadas,{part=p,weld=w})
    return true
end
local scanning=false
local function escanearMapa()
    if scanning then return end
    scanning=true
    task.spawn(function()
        local n,d={},workspace:GetDescendants()
        for i,o in ipairs(d) do
            if o:IsA("BasePart") and pareceCaixa(o) and not o.Anchored and o.Parent
                and not o:IsDescendantOf(LocalPlayer.Character or workspace) then
                local t=o.Size.X+o.Size.Y+o.Size.Z
                if t>=TAM_MIN and t<=TAM_MAX then table.insert(n,o) end
            end
            if i%500==0 then task.wait() end
        end
        candidatos=n; scanning=false
    end)
end

-- Parser
local function parseValor(str)
    if not str then return nil end
    str=tostring(str):gsub("%s+",""):gsub("[Rr]%$",""):gsub("reais",""):gsub("[Rr]eal","")
    local k=str:match("([%d%.%,]+)[kK]")
    if k then local n=tonumber(k:gsub(",",".")); if n then return math.floor(n*1000) end end
    local br=str:match("(%d+[%.%,]%d%d%d+)")
    if br then local n=tonumber(br:gsub("%.",""):gsub(",","")); if n then return n end end
    local n=str:match("^(%d+)$"); if n then return tonumber(n) end
    local b=str:match("(%d%d%d%d+)"); if b then return tonumber(b) end
    return nil
end
local function pareceEntrega(o)
    local n=(o.Name or ""):lower()
    for _,k in ipairs(NOMES_ENTREGA) do if n:find(k,1,true) then return true end end
    local p=o.Parent
    if p then
        local pn=(p.Name or ""):lower()
        for _,k in ipairs(NOMES_ENTREGA) do if pn:find(k,1,true) then return true end end
    end
    return false
end
local function extrairValorPrompt(pr)
    local t={pr.ActionText or "",pr.ObjectText or "",pr.Name or ""}
    if pr.Parent then
        table.insert(t,pr.Parent.Name or "")
        for _,d in ipairs(pr.Parent:GetDescendants()) do
            if d:IsA("TextLabel") or d:IsA("TextButton") then table.insert(t,d.Text or "") end
        end
        for _,a in ipairs(ATTRS_VALOR) do
            local v=pr.Parent:GetAttribute(a); if type(v)=="number" and v>=1000 then return v end
        end
    end
    for _,tx in ipairs(t) do local v=parseValor(tx); if v and v>=1000 then return v end end
    return nil
end
local function extrairValorObjeto(o)
    for _,a in ipairs(ATTRS_VALOR) do
        local v=o:GetAttribute(a); if type(v)=="number" and v>=1000 then return v end
    end
    local v=parseValor(o.Name); if v and v>=1000 then return v end
    return nil
end
local function varrerEntregas()
    local n={}
    local minV=DELIVERY_THRESHOLDS[DELIVERY_MIN_IDX]
    local root=getRoot()
    local orig=root and root.Position or Vector3.zero
    for _,o in ipairs(workspace:GetDescendants()) do
        if #n>=20 then break end
        if o:IsA("ProximityPrompt") and o.Enabled then
            local v=extrairValorPrompt(o)
            if v and v>=minV then
                local p=o.Parent
                local pos=(p and p:IsA("BasePart")) and p.Position or orig
                if (pos-orig).Magnitude<1500 then
                    table.insert(n,{tipo="prompt",obj=o,valor=v,pos=pos})
                end
            end
        elseif o:IsA("ClickDetector") and pareceEntrega(o) then
            local p=o.Parent
            if p and p:IsA("BasePart") then
                local v=extrairValorObjeto(p) or extrairValorObjeto(o)
                if v and v>=minV and (p.Position-orig).Magnitude<500 then
                    table.insert(n,{tipo="click",obj=o,valor=v,pos=p.Position})
                end
            end
        end
    end
    table.sort(n,function(a,b) return a.valor>b.valor end)
    return n
end
local function firePrompt(pr)
    if not pr or not pr.Parent then return false end
    local ok=pcall(function() if fireproximityprompt then fireproximityprompt(pr) end end)
    if ok then return true end
    return pcall(function()
        pr.HoldDuration=0; pr.MaxActivationDistance=9999
        if pr.InputHoldBegin then pr:InputHoldBegin() end
        task.wait(0.08)
        if pr.InputHoldEnd then pr:InputHoldEnd() end
    end)
end
local function fireClick(cd)
    if not cd or not cd.Parent then return false end
    return pcall(function() if fireclickdetector then fireclickdetector(cd) end end)
end
local function aceitarEntrega(e)
    if not e or not e.obj or not e.obj.Parent then return false end
    if e.tipo=="prompt" then return firePrompt(e.obj) end
    if e.tipo=="click" then return fireClick(e.obj) end
    return false
end

-- GUI
print("[SERTANEJO] Criando UI...")
pcall(function()
    local old = game:GetService("CoreGui"):FindFirstChild("SertanejoHub")
    if old then old:Destroy() end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "SertanejoHub"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.DisplayOrder = 999

local parentSet = false
pcall(function()
    gui.Parent = game:GetService("CoreGui")
    parentSet = gui.Parent ~= nil
end)
if not parentSet then
    local pg = LocalPlayer:WaitForChild("PlayerGui", 5)
    if pg then gui.Parent = pg; parentSet = true end
end
if not parentSet then
    warn("[SERTANEJO] Falha ao criar GUI"); return
end
print("[SERTANEJO] GUI OK")

-- Notif
local notifHolder = Instance.new("Frame")
notifHolder.Size = UDim2.new(0, 300, 1, -40)
notifHolder.Position = UDim2.new(1, -320, 0, 20)
notifHolder.BackgroundTransparency = 1
notifHolder.ZIndex = 200
notifHolder.Parent = gui
local notifLayout = Instance.new("UIListLayout", notifHolder)
notifLayout.Padding = UDim.new(0, 6)

local function notif(msg, color, dur)
    color = color or THEME.info; dur = dur or 3
    local n = Instance.new("Frame")
    n.Size = UDim2.new(1, 0, 0, 40)
    n.BackgroundColor3 = THEME.glass; n.BackgroundTransparency = 0.05
    n.ZIndex = 201; n.Parent = notifHolder
    uiCorner(n, 8); local s = uiStroke(n, color, 1); s.Transparency = 0.3
    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(0, 3, 1, -8); bar.Position = UDim2.new(0, 4, 0, 4)
    bar.BackgroundColor3 = color; bar.BorderSizePixel = 0; bar.ZIndex = 202; bar.Parent = n
    uiCorner(bar, 2)
    local txt = Instance.new("TextLabel")
    txt.Size = UDim2.new(1, -24, 1, 0); txt.Position = UDim2.new(0, 14, 0, 0)
    txt.BackgroundTransparency = 1; txt.Text = msg; txt.TextColor3 = THEME.text
    txt.Font = Enum.Font.Gotham; txt.TextSize = 11; txt.TextXAlignment = Enum.TextXAlignment.Left
    txt.TextWrapped = true; txt.ZIndex = 202; txt.Parent = n
    n.Position = UDim2.new(1, 50, 0, 0)
    TweenService:Create(n, TweenInfo.new(0.3), {Position = UDim2.new(0,0,0,0)}):Play()
    task.delay(dur, function()
        TweenService:Create(n, TweenInfo.new(0.3), {Position = UDim2.new(1,50,0,0), BackgroundTransparency = 1}):Play()
        TweenService:Create(txt, TweenInfo.new(0.3), {TextTransparency = 1}):Play()
        task.wait(0.4); n:Destroy()
    end)
end

-- Lock
local lc = Instance.new("Frame")
lc.Size = UDim2.new(0,340,0,340); lc.Position = UDim2.new(0.5,-170,0.5,-170)
lc.BackgroundColor3 = THEME.glass; lc.BackgroundTransparency = 0.1
lc.ZIndex = 51; lc.Parent = gui
uiCorner(lc,16); uiStroke(lc,THEME.stroke,1)

local li = Instance.new("TextLabel")
li.Size = UDim2.new(1,0,0,60); li.Position = UDim2.new(0,0,0,24)
li.BackgroundTransparency = 1; li.Text = "🌀"; li.TextSize = 52; li.ZIndex = 52; li.Parent = lc

local lt = Instance.new("TextLabel")
lt.Size = UDim2.new(1,0,0,32); lt.Position = UDim2.new(0,0,0,88)
lt.BackgroundTransparency = 1; lt.Text = HUB_NAME; lt.TextColor3 = THEME.text
lt.Font = Enum.Font.GothamBlack; lt.TextSize = 24; lt.ZIndex = 52; lt.Parent = lc
uiGrad(lt, THEME.accent, THEME.accent2, 0)

local lt2 = Instance.new("TextLabel")
lt2.Size = UDim2.new(1,0,0,20); lt2.Position = UDim2.new(0,0,0,124)
lt2.BackgroundTransparency = 1; lt2.Text = HUB_TAG.." • by "..CREATOR
lt2.TextColor3 = THEME.textDim; lt2.Font = Enum.Font.Gotham; lt2.TextSize = 11
lt2.ZIndex = 52; lt2.Parent = lc

local ld = Instance.new("Frame")
ld.Size = UDim2.new(1,-56,0,1); ld.Position = UDim2.new(0,28,0,158)
ld.BackgroundColor3 = THEME.stroke; ld.BorderSizePixel = 0; ld.ZIndex = 52; ld.Parent = lc

local kl = Instance.new("TextLabel")
kl.Size = UDim2.new(1,-56,0,18); kl.Position = UDim2.new(0,28,0,174)
kl.BackgroundTransparency = 1; kl.Text = "🔑 ACESSO"; kl.TextColor3 = THEME.textDim
kl.Font = Enum.Font.GothamBold; kl.TextSize = 10; kl.TextXAlignment = Enum.TextXAlignment.Left
kl.ZIndex = 52; kl.Parent = lc

local ki = Instance.new("TextBox")
ki.Size = UDim2.new(1,-56,0,44); ki.Position = UDim2.new(0,28,0,196)
ki.BackgroundColor3 = THEME.glass2; ki.PlaceholderText = "digite sua key..."
ki.PlaceholderColor3 = THEME.textMuted; ki.Text = ""; ki.TextColor3 = THEME.text
ki.Font = Enum.Font.Gotham; ki.TextSize = 14; ki.ClearTextOnFocus = false
ki.ZIndex = 52; ki.Parent = lc
uiCorner(ki,10); local kst = uiStroke(ki,THEME.stroke,1)

local kb = Instance.new("TextButton")
kb.Size = UDim2.new(1,-56,0,44); kb.Position = UDim2.new(0,28,0,252)
kb.BackgroundColor3 = THEME.accent; kb.Text = "DESBLOQUEAR"; kb.TextColor3 = Color3.new(1,1,1)
kb.Font = Enum.Font.GothamBold; kb.TextSize = 14; kb.ZIndex = 52; kb.Parent = lc
uiCorner(kb,10); uiGrad(kb,THEME.accent,THEME.accent2,0); pressAnim(kb)

local ke = Instance.new("TextLabel")
ke.Size = UDim2.new(1,-56,0,20); ke.Position = UDim2.new(0,28,0,306)
ke.BackgroundTransparency = 1; ke.Text = "💬 Compre sua key no Discord do criador"
ke.TextColor3 = THEME.textMuted; ke.Font = Enum.Font.Gotham; ke.TextSize = 10
ke.ZIndex = 52; ke.Parent = lc

-- Main
local main = Instance.new("Frame")
main.Size = UDim2.new(0,340,0,700); main.Position = UDim2.new(0,15,0.5,-350)
main.BackgroundColor3 = THEME.glass; main.BackgroundTransparency = 0.05
main.Active = true; main.Visible = false; main.ZIndex = 10; main.Parent = gui
uiCorner(main,14); uiStroke(main,THEME.stroke,1)

local drag, ds, sp2
main.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        drag = true; ds = i.Position; sp2 = main.Position
        i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then drag = false end end)
    end
end)
main.InputChanged:Connect(function(i)
    if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - ds
        main.Position = UDim2.new(sp2.X.Scale, sp2.X.Offset+d.X, sp2.Y.Scale, sp2.Y.Offset+d.Y)
    end
end)

local tb = Instance.new("Frame")
tb.Size = UDim2.new(1,0,0,52); tb.BackgroundColor3 = THEME.accent; tb.Parent = main
uiCorner(tb,14); uiGrad(tb,THEME.accent,THEME.accent2,25)

local ttl = Instance.new("TextLabel")
ttl.Size = UDim2.new(1,-130,0,20); ttl.Position = UDim2.new(0,16,0,6)
ttl.BackgroundTransparency = 1; ttl.Text = "🌀 "..HUB_NAME; ttl.TextColor3 = Color3.new(1,1,1)
ttl.Font = Enum.Font.GothamBlack; ttl.TextSize = 14; ttl.TextXAlignment = Enum.TextXAlignment.Left; ttl.Parent = tb

local ttl2 = Instance.new("TextLabel")
ttl2.Size = UDim2.new(1,-130,0,16); ttl2.Position = UDim2.new(0,16,0,28)
ttl2.BackgroundTransparency = 1; ttl2.Text = "👤 "..CREATOR.." • "..HUB_VER
ttl2.TextColor3 = Color3.new(1,1,1); ttl2.TextTransparency = 0.2
ttl2.Font = Enum.Font.Gotham; ttl2.TextSize = 10; ttl2.TextXAlignment = Enum.TextXAlignment.Left; ttl2.Parent = tb

local mb = Instance.new("TextButton")
mb.Size = UDim2.new(0,32,0,32); mb.Position = UDim2.new(1,-40,0,10)
mb.BackgroundColor3 = Color3.new(1,1,1); mb.BackgroundTransparency = 0.85
mb.Text = "−"; mb.TextColor3 = Color3.new(1,1,1); mb.Font = Enum.Font.GothamBold
mb.TextSize = 20; mb.Parent = tb; uiCorner(mb,8)

-- Stats bar
local statsBar = Instance.new("Frame")
statsBar.Size = UDim2.new(1,-20,0,22); statsBar.Position = UDim2.new(0,10,0,60)
statsBar.BackgroundColor3 = THEME.glass2; statsBar.Parent = main
uiCorner(statsBar,6); uiStroke(statsBar,THEME.stroke,1)

local fpsLbl = Instance.new("TextLabel")
fpsLbl.Size = UDim2.new(0.5,-4,1,0); fpsLbl.Position = UDim2.new(0,6,0,0)
fpsLbl.BackgroundTransparency = 1; fpsLbl.Text = "FPS: --"
fpsLbl.TextColor3 = THEME.success; fpsLbl.Font = Enum.Font.GothamBold
fpsLbl.TextSize = 10; fpsLbl.TextXAlignment = Enum.TextXAlignment.Left; fpsLbl.Parent = statsBar

local pingLbl = Instance.new("TextLabel")
pingLbl.Size = UDim2.new(0.5,-4,1,0); pingLbl.Position = UDim2.new(0.5,0,0,0)
pingLbl.BackgroundTransparency = 1; pingLbl.Text = "Ping: --"
pingLbl.TextColor3 = THEME.info; pingLbl.Font = Enum.Font.GothamBold
pingLbl.TextSize = 10; pingLbl.TextXAlignment = Enum.TextXAlignment.Left; pingLbl.Parent = statsBar

-- Prot bar
local protBar = Instance.new("Frame")
protBar.Size = UDim2.new(1,-20,0,22); protBar.Position = UDim2.new(0,10,0,86)
protBar.BackgroundColor3 = THEME.glass2; protBar.Parent = main
uiCorner(protBar,6); uiStroke(protBar,THEME.stroke,1)

local protLbl = Instance.new("TextLabel")
protLbl.Size = UDim2.new(1,-12,1,0); protLbl.Position = UDim2.new(0,8,0,0)
protLbl.BackgroundTransparency = 1
protLbl.Text = ("KICK %s | REPORT %s | REJOIN %s | AFK %s"):format(
    PROT.antikick and "ON" or "OFF",
    PROT.antireport and "ON" or "OFF",
    PROT.rejoin and "ON" or "OFF",
    PROT.antiafk and "ON" or "OFF")
protLbl.TextColor3 = THEME.warn; protLbl.Font = Enum.Font.GothamBold
protLbl.TextSize = 9; protLbl.TextXAlignment = Enum.TextXAlignment.Left; protLbl.Parent = protBar

-- Tabs
local tr = Instance.new("Frame")
tr.Size = UDim2.new(1,-20,0,34); tr.Position = UDim2.new(0,10,0,114)
tr.BackgroundColor3 = THEME.glass2; tr.Parent = main
uiCorner(tr,10); uiStroke(tr,THEME.stroke,1)

tabIndicator = Instance.new("Frame")
tabIndicator.Size = UDim2.new(0.2,-3,1,-6); tabIndicator.Position = UDim2.new(0,2,0,3)
tabIndicator.BackgroundColor3 = THEME.accent; tabIndicator.Parent = tr
uiCorner(tabIndicator,8); uiGrad(tabIndicator,THEME.accent,THEME.accent2,25)
tabIndicator.ZIndex = 1

local TABS = {"TP", "ITENS", "PLAYER", "ESP", "EXTRA"}
local tabBtns = {}
local tabContents = {}
for i, name in ipairs(TABS) do
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.2,0,1,0); b.Position = UDim2.new((i-1)*0.2,0,0,0)
    b.BackgroundTransparency = 1; b.Text = name
    b.TextColor3 = (i==1) and Color3.new(1,1,1) or THEME.textDim
    b.Font = Enum.Font.GothamBold; b.TextSize = 10; b.Parent = tr; b.ZIndex = 2
    tabBtns[i] = b
end

local contentHolder = Instance.new("Frame")
contentHolder.Size = UDim2.new(1,0,1,-148); contentHolder.Position = UDim2.new(0,0,0,148)
contentHolder.BackgroundTransparency = 1; contentHolder.Parent = main

for i=1,#TABS do
    local c = Instance.new("Frame")
    c.Size = UDim2.new(1,0,1,0); c.BackgroundTransparency = 1
    c.Visible = (i == 1); c.Parent = contentHolder
    tabContents[i] = c
end

local function setTab(idx)
    TweenService:Create(tabIndicator, TweenInfo.new(0.3), {Position = UDim2.new((idx-1)*0.2,2,0,3)}):Play()
    for i,b in ipairs(tabBtns) do
        b.TextColor3 = (i==idx) and Color3.new(1,1,1) or THEME.textDim
        tabContents[i].Visible = (i==idx)
    end
end
for i,b in ipairs(tabBtns) do
    b.MouseButton1Click:Connect(function() setTab(i) end)
end

-- TAB 1: TP
local tc = tabContents[1]
local sb = Instance.new("TextBox")
sb.Size = UDim2.new(1,-20,0,32); sb.Position = UDim2.new(0,10,0,6)
sb.BackgroundColor3 = THEME.glass2; sb.PlaceholderText = "Buscar local..."
sb.PlaceholderColor3 = THEME.textMuted; sb.Text = ""; sb.TextColor3 = THEME.text
sb.Font = Enum.Font.Gotham; sb.TextSize = 12; sb.ClearTextOnFocus = false
sb.TextXAlignment = Enum.TextXAlignment.Left; sb.Parent = tc
uiCorner(sb,8); uiStroke(sb,THEME.stroke,1)

local scr = Instance.new("ScrollingFrame")
scr.Size = UDim2.new(1,-20,0,388); scr.Position = UDim2.new(0,10,0,44)
scr.BackgroundTransparency = 1; scr.BorderSizePixel = 0
scr.ScrollBarThickness = 4; scr.ScrollBarImageColor3 = THEME.accent
scr.AutomaticCanvasSize = Enum.AutomaticSize.Y; scr.CanvasSize = UDim2.new(0,0,0,0)
scr.Parent = tc
local ll = Instance.new("UIListLayout",scr)
ll.Padding = UDim.new(0,5); ll.SortOrder = Enum.SortOrder.LayoutOrder

local svb = Instance.new("TextButton")
svb.Size = UDim2.new(0.5,-15,0,36); svb.Position = UDim2.new(0,10,1,-82)
svb.BackgroundColor3 = THEME.info; svb.TextColor3 = Color3.new(1,1,1)
svb.Font = Enum.Font.GothamBold; svb.TextSize = 11; svb.Text = "Salvar"; svb.Parent = tc
uiCorner(svb,10); uiGrad(svb,THEME.info,Color3.fromRGB(45,120,220),25); pressAnim(svb)

local cb = Instance.new("TextButton")
cb.Size = UDim2.new(0.5,-15,0,36); cb.Position = UDim2.new(0.5,5,1,-82)
cb.BackgroundColor3 = THEME.danger; cb.TextColor3 = Color3.new(1,1,1)
cb.Font = Enum.Font.GothamBold; cb.TextSize = 11; cb.Text = "Cancelar"; cb.Parent = tc
uiCorner(cb,10); uiGrad(cb,THEME.danger,Color3.fromRGB(180,40,60),25); pressAnim(cb)

local sf = Instance.new("Frame")
sf.Size = UDim2.new(1,-20,0,30); sf.Position = UDim2.new(0,10,1,-40)
sf.BackgroundColor3 = THEME.glass2; sf.Parent = tc
uiCorner(sf,10); uiStroke(sf,THEME.stroke,1)

statusDot = Instance.new("Frame")
statusDot.Size = UDim2.new(0,9,0,9); statusDot.Position = UDim2.new(0,12,0.5,-4.5)
statusDot.BackgroundColor3 = THEME.textMuted; statusDot.Parent = sf; uiCorner(statusDot,5)

statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1,-32,1,0); statusLabel.Position = UDim2.new(0,28,0,0)
statusLabel.BackgroundTransparency = 1; statusLabel.Text = "Pronto"
statusLabel.TextColor3 = THEME.text; statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 11; statusLabel.TextXAlignment = Enum.TextXAlignment.Left; statusLabel.Parent = sf

-- TAB 2: ITENS
local ic = tabContents[2]

local dh = Instance.new("TextLabel")
dh.Size = UDim2.new(1,-20,0,14); dh.Position = UDim2.new(0,10,0,6)
dh.BackgroundTransparency = 1; dh.Text = "ENTREGAS DE ALTO VALOR"
dh.TextColor3 = THEME.gold; dh.Font = Enum.Font.GothamBold
dh.TextSize = 9; dh.TextXAlignment = Enum.TextXAlignment.Left; dh.Parent = ic

thresholdBtn = Instance.new("TextButton")
thresholdBtn.Size = UDim2.new(1,-20,0,30); thresholdBtn.Position = UDim2.new(0,10,0,24)
thresholdBtn.BackgroundColor3 = THEME.glass2
thresholdBtn.Text = " Min: R$ "..fmtVal(DELIVERY_THRESHOLDS[1])
thresholdBtn.TextColor3 = THEME.gold; thresholdBtn.Font = Enum.Font.GothamBold
thresholdBtn.TextSize = 11; thresholdBtn.TextXAlignment = Enum.TextXAlignment.Left
thresholdBtn.Parent = ic
uiCorner(thresholdBtn,8); uiStroke(thresholdBtn,THEME.gold,1,0.5); pressAnim(thresholdBtn)

local aaF = Instance.new("Frame")
aaF.Size = UDim2.new(1,-20,0,32); aaF.Position = UDim2.new(0,10,0,60)
aaF.BackgroundColor3 = THEME.glass2; aaF.Parent = ic
uiCorner(aaF,8); uiStroke(aaF,THEME.stroke,1)

autoAcceptKnob = Instance.new("Frame")
autoAcceptKnob.Size = UDim2.new(0,22,0,22); autoAcceptKnob.Position = UDim2.new(1,-28,0.5,-11)
autoAcceptKnob.BackgroundColor3 = THEME.success; autoAcceptKnob.Parent = aaF
uiCorner(autoAcceptKnob,11)

local aaL = Instance.new("TextLabel")
aaL.Size = UDim2.new(1,-50,1,0); aaL.Position = UDim2.new(0,12,0,0)
aaL.BackgroundTransparency = 1; aaL.Text = "Auto-aceitar entregas"
aaL.TextColor3 = THEME.text; aaL.Font = Enum.Font.GothamBold
aaL.TextSize = 11; aaL.TextXAlignment = Enum.TextXAlignment.Left; aaL.Parent = aaF

local aaC = Instance.new("TextButton")
aaC.Size = UDim2.new(1,0,1,0); aaC.BackgroundTransparency = 1
aaC.Text = ""; aaC.Parent = aaF

deliveryCountLbl = Instance.new("TextLabel")
deliveryCountLbl.Size = UDim2.new(1,-20,0,14); deliveryCountLbl.Position = UDim2.new(0,10,0,98)
deliveryCountLbl.BackgroundTransparency = 1; deliveryCountLbl.Text = "Aguardando scan..."
deliveryCountLbl.TextColor3 = THEME.textMuted; deliveryCountLbl.Font = Enum.Font.Gotham
deliveryCountLbl.TextSize = 10; deliveryCountLbl.TextXAlignment = Enum.TextXAlignment.Left
deliveryCountLbl.Parent = ic

deliveryMaxLbl = Instance.new("TextLabel")
deliveryMaxLbl.Size = UDim2.new(1,-20,0,14); deliveryMaxLbl.Position = UDim2.new(0,10,0,114)
deliveryMaxLbl.BackgroundTransparency = 1; deliveryMaxLbl.Text = "Maior: --"
deliveryMaxLbl.TextColor3 = THEME.gold; deliveryMaxLbl.Font = Enum.Font.GothamBold
deliveryMaxLbl.TextSize = 10; deliveryMaxLbl.TextXAlignment = Enum.TextXAlignment.Left
deliveryMaxLbl.Parent = ic

local dv2 = Instance.new("Frame")
dv2.Size = UDim2.new(1,-20,0,1); dv2.Position = UDim2.new(0,10,0,138)
dv2.BackgroundColor3 = THEME.stroke; dv2.BorderSizePixel = 0; dv2.Parent = ic

local ih = Instance.new("TextLabel")
ih.Size = UDim2.new(1,-20,0,12); ih.Position = UDim2.new(0,10,0,148)
ih.BackgroundTransparency = 1; ih.Text = "IMA DE CAIXAS"
ih.TextColor3 = THEME.textMuted; ih.Font = Enum.Font.GothamBold
ih.TextSize = 9; ih.TextXAlignment = Enum.TextXAlignment.Left; ih.Parent = ic

imaToggle = Instance.new("Frame")
imaToggle.Size = UDim2.new(1,-20,0,34); imaToggle.Position = UDim2.new(0,10,0,164)
imaToggle.BackgroundColor3 = THEME.glass2; imaToggle.Parent = ic
uiCorner(imaToggle,8); uiStroke(imaToggle,THEME.stroke,1)

imaKnob = Instance.new("Frame")
imaKnob.Size = UDim2.new(0,26,0,26); imaKnob.Position = UDim2.new(0,4,0.5,-13)
imaKnob.BackgroundColor3 = Color3.fromRGB(90,90,100); imaKnob.Parent = imaToggle
uiCorner(imaKnob,13)

imaToggleLbl = Instance.new("TextLabel")
imaToggleLbl.Size = UDim2.new(1,-44,1,0); imaToggleLbl.Position = UDim2.new(0,40,0,0)
imaToggleLbl.BackgroundTransparency = 1; imaToggleLbl.Text = "Ativar ima"
imaToggleLbl.TextColor3 = THEME.text; imaToggleLbl.Font = Enum.Font.GothamBold
imaToggleLbl.TextSize = 11; imaToggleLbl.TextXAlignment = Enum.TextXAlignment.Left
imaToggleLbl.Parent = imaToggle

local imaC = Instance.new("TextButton")
imaC.Size = UDim2.new(1,0,1,0); imaC.BackgroundTransparency = 1
imaC.Text = ""; imaC.Parent = imaToggle

local ipb = Instance.new("Frame")
ipb.Size = UDim2.new(1,-20,0,8); ipb.Position = UDim2.new(0,10,0,206)
ipb.BackgroundColor3 = THEME.glass3; ipb.Parent = ic
uiCorner(ipb,4)

imaProgressFill = Instance.new("Frame")
imaProgressFill.Size = UDim2.new(0,0,1,0); imaProgressFill.BackgroundColor3 = THEME.success
imaProgressFill.Parent = ipb; uiCorner(imaProgressFill,4)
uiGrad(imaProgressFill,THEME.success,THEME.accent,0)

imaCount = Instance.new("TextLabel")
imaCount.Size = UDim2.new(1,-20,0,14); imaCount.Position = UDim2.new(0,10,0,218)
imaCount.BackgroundTransparency = 1; imaCount.Text = "0 / "..MAX_PARTS.." caixas"
imaCount.TextColor3 = THEME.textMuted; imaCount.Font = Enum.Font.Gotham
imaCount.TextSize = 10; imaCount.TextXAlignment = Enum.TextXAlignment.Left; imaCount.Parent = ic

local solB = Instance.new("TextButton")
solB.Size = UDim2.new(0.5,-15,0,30); solB.Position = UDim2.new(0,10,0,238)
solB.BackgroundColor3 = THEME.glass2; solB.TextColor3 = THEME.text
solB.Font = Enum.Font.GothamBold; solB.TextSize = 10; solB.Text = "SOLTAR"
solB.Parent = ic
uiCorner(solB,6); uiStroke(solB,THEME.stroke,1); pressAnim(solB)

local scnB = Instance.new("TextButton")
scnB.Size = UDim2.new(0.5,-15,0,30); scnB.Position = UDim2.new(0.5,5,0,238)
scnB.BackgroundColor3 = THEME.glass2; scnB.TextColor3 = THEME.text
scnB.Font = Enum.Font.GothamBold; scnB.TextSize = 10; scnB.Text = "RE-SCAN"
scnB.Parent = ic
uiCorner(scnB,6); uiStroke(scnB,THEME.stroke,1); pressAnim(scnB)

dbgBtn = Instance.new("TextButton")
dbgBtn.Size = UDim2.new(1,-20,0,26); dbgBtn.Position = UDim2.new(0,10,0,276)
dbgBtn.BackgroundColor3 = THEME.glass2; dbgBtn.Text = "DEBUG: OFF"
dbgBtn.TextColor3 = THEME.warn; dbgBtn.Font = Enum.Font.GothamBold
dbgBtn.TextSize = 10; dbgBtn.Parent = ic
uiCorner(dbgBtn,6); uiStroke(dbgBtn,THEME.stroke,1); pressAnim(dbgBtn)

-- TAB 3: PLAYER
local pc = tabContents[3]

local function makeSlider(parent, y, label, initial, minVal, maxVal, onChange, color)
    color = color or THEME.accent
    local L = Instance.new("TextLabel")
    L.Size = UDim2.new(1,-20,0,14); L.Position = UDim2.new(0,10,0,y)
    L.BackgroundTransparency = 1; L.Text = label..": "..initial
    L.TextColor3 = THEME.text; L.Font = Enum.Font.GothamBold
    L.TextSize = 10; L.TextXAlignment = Enum.TextXAlignment.Left; L.Parent = parent

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1,-20,0,8); bar.Position = UDim2.new(0,10,0,y+18)
    bar.BackgroundColor3 = THEME.glass3; bar.Parent = parent
    uiCorner(bar,4)

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((initial-minVal)/(maxVal-minVal),0,1,0)
    fill.BackgroundColor3 = color; fill.Parent = bar
    uiCorner(fill,4); uiGrad(fill,color,THEME.accent2,0)

    local dragBar = Instance.new("TextButton")
    dragBar.Size = UDim2.new(1,0,1,0); dragBar.BackgroundTransparency = 1
    dragBar.Text = ""; dragBar.Parent = bar

    local dragging = false
    dragBar.MouseButton1Down:Connect(function() dragging = true end)
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = false end
    end)
    dragBar.MouseMoved:Connect(function()
        if dragging then
            local mx = UIS:GetMouseLocation().X
            local ap = bar.AbsolutePosition.X
            local as = bar.AbsoluteSize.X
            local pct = math.clamp((mx - ap) / as, 0, 1)
            local val = math.floor(minVal + pct * (maxVal - minVal))
            fill.Size = UDim2.new(pct, 0, 1, 0)
            L.Text = label..": "..val
            onChange(val)
        end
    end)
end

local function makeToggle(parent, y, label, initial, onChange, color)
    color = color or THEME.success
    local F = Instance.new("Frame")
    F.Size = UDim2.new(1,-20,0,34); F.Position = UDim2.new(0,10,0,y)
    F.BackgroundColor3 = THEME.glass2; F.Parent = parent
    uiCorner(F,8); local st = uiStroke(F,THEME.stroke,1)

    local L = Instance.new("TextLabel")
    L.Size = UDim2.new(1,-56,1,0); L.Position = UDim2.new(0,12,0,0)
    L.BackgroundTransparency = 1; L.Text = label
    L.TextColor3 = THEME.text; L.Font = Enum.Font.GothamBold
    L.TextSize = 11; L.TextXAlignment = Enum.TextXAlignment.Left; L.Parent = F

    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0,22,0,22)
    knob.Position = initial and UDim2.new(1,-28,0.5,-11) or UDim2.new(0,6,0.5,-11)
    knob.BackgroundColor3 = initial and color or Color3.fromRGB(90,90,100)
    knob.Parent = F; uiCorner(knob,11)

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1
    btn.Text = ""; btn.Parent = F

    local state = initial
    btn.MouseButton1Click:Connect(function()
        state = not state
        local p = state and UDim2.new(1,-28,0.5,-11) or UDim2.new(0,6,0.5,-11)
        local c = state and color or Color3.fromRGB(90,90,100)
        TweenService:Create(knob,TweenInfo.new(0.25, Enum.EasingStyle.Back),{Position=p,BackgroundColor3=c}):Play()
        onChange(state)
    end)
end

makeSlider(pc, 6, "Walk Speed", SPEED_VAL, 16, 300, function(v)
    SPEED_VAL = v
    if SPEED_ATIVO then local h = getHumanoid(); if h then h.WalkSpeed = v end end
end, THEME.success)

makeToggle(pc, 40, "Walk Speed Ativo", false, function(on)
    SPEED_ATIVO = on
    local h = getHumanoid()
    if h then h.WalkSpeed = on and SPEED_VAL or 16 end
    notif(on and "Speed ON" or "Speed OFF", THEME.success)
end, THEME.success)

makeSlider(pc, 84, "Jump Power", JUMP_VAL, 50, 500, function(v)
    JUMP_VAL = v
    if JUMP_ATIVO then local h = getHumanoid(); if h then h.JumpPower = v end end
end, THEME.info)

makeToggle(pc, 118, "Jump Power Ativo", false, function(on)
    JUMP_ATIVO = on
    local h = getHumanoid()
    if h then h.JumpPower = on and JUMP_VAL or 50 end
    notif(on and "Jump ON" or "Jump OFF", THEME.info)
end, THEME.info)

makeToggle(pc, 162, "Infinite Jump", false, function(on)
    INFINITE_JUMP = on
    notif(on and "Inf Jump ON" or "Inf Jump OFF", THEME.purple)
end, THEME.purple)

makeToggle(pc, 206, "Fly", false, function(on)
    FLY_ATIVO = on
    local r = getRoot()
    if on and r then
        flyBodyVel = Instance.new("BodyVelocity")
        flyBodyVel.Velocity = Vector3.zero
        flyBodyVel.MaxForce = Vector3.new(1e5,1e5,1e5)
        flyBodyVel.P = 1250
        flyBodyVel.Parent = r
        notif("Fly ON (WASD+Space/Shift)", THEME.purple)
    elseif flyBodyVel then
        flyBodyVel:Destroy(); flyBodyVel = nil
        notif("Fly OFF", THEME.textDim)
    end
end, THEME.purple)

makeToggle(pc, 250, "Noclip", false, function(on)
    NOCLIP_ATIVO = on
    notif(on and "Noclip ON" or "Noclip OFF", THEME.warn)
end, THEME.warn)

-- TAB 4: ESP
local ec = tabContents[4]

makeToggle(ec, 6, "ESP Ativo", false, function(on)
    ESP_ATIVO = on
    if not on then
        for _, v in pairs(currentESP) do if v and v.Parent then v:Destroy() end end
        currentESP = {}
    end
    notif(on and "ESP ON" or "ESP OFF", THEME.info)
end, THEME.info)

makeToggle(ec, 50, "Nome", true, function(on) ESP_NAME = on end, THEME.success)
makeToggle(ec, 94, "Vida", true, function(on) ESP_HEALTH = on end, THEME.success)

-- TAB 5: EXTRA
local xc = tabContents[5]

makeToggle(xc, 6, "Fullbright", false, function(on)
    FULLBRIGHT = on
    if on then
        Lighting.Brightness = 3
        Lighting.OutdoorAmbient = Color3.fromRGB(200,200,200)
        Lighting.ClockTime = 12
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e10
    else
        Lighting.Brightness = lightingOrig.Brightness
        Lighting.OutdoorAmbient = lightingOrig.OutdoorAmbient
        Lighting.ClockTime = lightingOrig.ClockTime
        Lighting.GlobalShadows = lightingOrig.GlobalShadows
        Lighting.FogEnd = lightingOrig.FogEnd
    end
    notif(on and "Fullbright ON" or "Fullbright OFF", THEME.gold)
end, THEME.gold)

makeToggle(xc, 50, "Click Spam", false, function(on)
    CLICK_SPAM = on
    notif(on and "Click Spam ON" or "Click Spam OFF", THEME.success)
end, THEME.success)

makeToggle(xc, 94, "Chat Spam", false, function(on)
    CHAT_SPAM = on
    notif(on and "Chat Spam ON" or "Chat Spam OFF", THEME.warn)
end, THEME.warn)

local rjBtn = Instance.new("TextButton")
rjBtn.Size = UDim2.new(1,-20,0,36); rjBtn.Position = UDim2.new(0,10,0,138)
rjBtn.BackgroundColor3 = THEME.info; rjBtn.Text = "REJOIN SERVER"
rjBtn.TextColor3 = Color3.new(1,1,1); rjBtn.Font = Enum.Font.GothamBold
rjBtn.TextSize = 11; rjBtn.Parent = xc
uiCorner(rjBtn,8); uiGrad(rjBtn,THEME.info,Color3.fromRGB(45,120,220),25); pressAnim(rjBtn)
rjBtn.MouseButton1Click:Connect(function()
    notif("Reconectando...", THEME.info)
    task.wait(0.5)
    pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer) end)
end)

local rsBtn = Instance.new("TextButton")
rsBtn.Size = UDim2.new(1,-20,0,32); rsBtn.Position = UDim2.new(0,10,0,182)
rsBtn.BackgroundColor3 = THEME.glass2; rsBtn.Text = "Reset Personagem"
rsBtn.TextColor3 = THEME.danger; rsBtn.Font = Enum.Font.GothamBold
rsBtn.TextSize = 10; rsBtn.Parent = xc
uiCorner(rsBtn,8); uiStroke(rsBtn,THEME.danger,1,0.5); pressAnim(rsBtn)
rsBtn.MouseButton1Click:Connect(function()
    local h = getHumanoid()
    if h then h.Health = 0; notif("Reset", THEME.danger) end
end)

-- Minimize
local min = false
mb.MouseButton1Click:Connect(function()
    min = not min
    local targetSize = UDim2.new(0,340,0,min and 52 or 700)
    TweenService:Create(main, TweenInfo.new(0.3), {Size=targetSize}):Play()
    for _,v in ipairs({statsBar,protBar,tr,contentHolder}) do v.Visible = not min end
end)

-- Handlers
local function setStatus(txt, c)
    if not statusLabel then return end
    statusLabel.Text = txt
    if c and statusDot then TweenService:Create(statusDot,TweenInfo.new(0.3),{BackgroundColor3=c}):Play() end
end

local function setImaState(on)
    if IMA_ATIVO == on then return end
    IMA_ATIVO = on
    local kp = on and UDim2.new(1,-30,0.5,-13) or UDim2.new(0,4,0.5,-13)
    local kc = on and THEME.success or Color3.fromRGB(90,90,100)
    TweenService:Create(imaKnob,TweenInfo.new(0.25, Enum.EasingStyle.Back),{Position=kp,BackgroundColor3=kc}):Play()
    imaToggleLbl.Text = on and "Ima ATIVO" or "Ativar ima"
    if on then escanearMapa() else soltarTudo() end
    notif(on and "Ima ON" or "Ima OFF", THEME.success)
end

local function shake(f)
    local o = f.Position
    local s = {
        {0.04, UDim2.new(o.X.Scale, o.X.Offset-8, o.Y.Scale, o.Y.Offset)},
        {0.08, UDim2.new(o.X.Scale, o.X.Offset+8, o.Y.Scale, o.Y.Offset)},
        {0.12, UDim2.new(o.X.Scale, o.X.Offset-5, o.Y.Scale, o.Y.Offset)},
        {0.16, UDim2.new(o.X.Scale, o.X.Offset+5, o.Y.Scale, o.Y.Offset)},
        {0.20, o},
    }
    for _,t in ipairs(s) do
        task.delay(t[1], function() TweenService:Create(f,TweenInfo.new(0.04),{Position=t[2]}):Play() end)
    end
end

local function unlock()
    for _,d in ipairs(lc:GetDescendants()) do
        if d:IsA("GuiObject") then pcall(function() TweenService:Create(d,TweenInfo.new(0.25),{BackgroundTransparency=1}):Play() end) end
        if d:IsA("TextLabel") or d:IsA("TextButton") or d:IsA("TextBox") then
            pcall(function() TweenService:Create(d,TweenInfo.new(0.25),{TextTransparency=1}):Play() end)
        end
        if d:IsA("UIStroke") then pcall(function() TweenService:Create(d,TweenInfo.new(0.25),{Transparency=1}):Play() end) end
    end
    task.wait(0.3)
    lc.Visible = false
    main.Visible = true
    main.Size = UDim2.new(0,340,0,640)
    main.Position = UDim2.new(0,15,0.5,-320)
    main.BackgroundTransparency = 1
    TweenService:Create(main,TweenInfo.new(0.4, Enum.EasingStyle.Quart),{
        BackgroundTransparency=0.05,
        Size=UDim2.new(0,340,0,700),
        Position=UDim2.new(0,15,0.5,-350),
    }):Play()
    notif("Bem-vindo ao "..HUB_NAME, THEME.success, 4)
end

local function tentarDesb()
    if ki.Text == HUB_KEY then
        ke.Text = "Acesso liberado!"; ke.TextColor3 = THEME.success
        task.wait(0.3)
        unlock()
        setStatus("Pronto", THEME.success)
    else
        ke.Text = "Key incorreta"; ke.TextColor3 = THEME.danger
        shake(lc)
        TweenService:Create(kst,TweenInfo.new(0.15),{Color=THEME.danger}):Play()
        task.delay(0.6,function() TweenService:Create(kst,TweenInfo.new(0.15),{Color=THEME.stroke}):Play() end)
    end
end
kb.MouseButton1Click:Connect(tentarDesb)
ki.FocusLost:Connect(function(e) if e then tentarDesb() end end)

-- Locations
local function addLoc(nome,pos,slot)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1,-6,0,32); b.BackgroundColor3 = THEME.glass2
    b.Text = " > "..nome; b.TextColor3 = pos and THEME.text or THEME.textMuted
    b.Font = Enum.Font.Gotham; b.TextSize = 11; b.TextXAlignment = Enum.TextXAlignment.Left
    b.LayoutOrder = slot or #scr:GetChildren(); b.Parent = scr
    uiCorner(b,8); local st = uiStroke(b,THEME.stroke,1)
    b:SetAttribute("Busca", nome:lower())
    b:SetAttribute("Slot", slot)
    hoverAnim(b, THEME.glass2, THEME.glass3, st)
    pressAnim(b)
    b.MouseButton1Click:Connect(function()
        if viajando then setStatus("Viajando...", THEME.info); return end
        if not pos then setStatus("Vazio. Use Salvar", THEME.danger); return end
        local ok,m = tpAndando(pos)
        if not ok then setStatus(m or "erro", THEME.danger); return end
        TweenService:Create(b,TweenInfo.new(0.2),{BackgroundColor3=THEME.accent}):Play()
        TweenService:Create(st,TweenInfo.new(0.2),{Color=THEME.accent}):Play()
        setStatus("Indo p/ "..nome, THEME.accent)
        task.spawn(function()
            while viajando do task.wait(0.1) end
            TweenService:Create(b,TweenInfo.new(0.2),{BackgroundColor3=THEME.glass2}):Play()
            TweenService:Create(st,TweenInfo.new(0.2),{Color=THEME.stroke}):Play()
            local s = _G.Sertanejo_TP_Status or ("Chegou em "..nome)
            setStatus(s, s:find("Chegou") and THEME.success or THEME.danger)
        end)
    end)
end
for i,l in ipairs(LOCATIONS) do addLoc(l.nome, l.pos, i) end

svb.MouseButton1Click:Connect(function()
    local r = getRoot()
    if not r then setStatus("Sem char", THEME.danger); return end
    for _,b in ipairs(scr:GetChildren()) do
        if b:IsA("TextButton") and b:GetAttribute("Slot") then
            local i = b:GetAttribute("Slot")
            if not LOCATIONS[i].pos then
                LOCATIONS[i].pos = r.Position
                b.TextColor3 = THEME.text
                b.Text = " > "..LOCATIONS[i].nome
                setStatus("Salvo em "..LOCATIONS[i].nome, THEME.success)
                salvarSalvos(); return
            end
        end
    end
    setStatus("Slots cheios", THEME.danger)
end)
cb.MouseButton1Click:Connect(function()
    if viajando then cancelarViagem = true; setStatus("Cancelando...", THEME.danger) end
end)
UIS.InputBegan:Connect(function(i,_)
    if i.KeyCode == CANCELAR_TECLA and viajando then
        cancelarViagem = true
        setStatus("Cancelando TP", THEME.danger)
    end
end)

sb:GetPropertyChangedSignal("Text"):Connect(function()
    local t = sb.Text:lower()
    for _,b in ipairs(scr:GetChildren()) do
        if b:IsA("TextButton") then
            local a = b:GetAttribute("Busca") or ""
            b.Visible = t=="" or string.find(a,t,1,true) ~= nil
        end
    end
end)

thresholdBtn.MouseButton1Click:Connect(function()
    DELIVERY_MIN_IDX = DELIVERY_MIN_IDX % #DELIVERY_THRESHOLDS + 1
    local v = DELIVERY_THRESHOLDS[DELIVERY_MIN_IDX]
    thresholdBtn.Text = " Min: R$ "..fmtVal(v)
    setStatus("Filtro: R$ "..fmtVal(v), THEME.gold)
end)
aaC.MouseButton1Click:Connect(function()
    AUTO_ACCEPT = not AUTO_ACCEPT
    local p = AUTO_ACCEPT and UDim2.new(1,-28,0.5,-11) or UDim2.new(0,6,0.5,-11)
    local c = AUTO_ACCEPT and THEME.success or Color3.fromRGB(90,90,100)
    TweenService:Create(autoAcceptKnob,TweenInfo.new(0.25, Enum.EasingStyle.Back),{Position=p,BackgroundColor3=c}):Play()
    notif(AUTO_ACCEPT and "Auto-accept ON" or "Auto-accept OFF", AUTO_ACCEPT and THEME.success or THEME.textDim)
end)
dbgBtn.MouseButton1Click:Connect(function()
    DEBUG = not DEBUG
    dbgBtn.Text = "DEBUG: "..(DEBUG and "ON" or "OFF")
    TweenService:Create(dbgBtn,TweenInfo.new(0.15),{TextColor3=DEBUG and THEME.success or THEME.warn}):Play()
end)
imaC.MouseButton1Click:Connect(function() setImaState(not IMA_ATIVO) end)
solB.MouseButton1Click:Connect(function() soltarTudo(); notif("Soltou tudo", THEME.textDim) end)
scnB.MouseButton1Click:Connect(function() escanearMapa(); notif("Re-escaneando...", THEME.info) end)

-- Loops
task.spawn(function()
    local frames = 0
    RunService.RenderStepped:Connect(function() frames = frames + 1 end)
    while true do
        task.wait(1)
        if fpsLbl then fpsLbl.Text = ("FPS: %d"):format(frames) end
        frames = 0
        local okP, p = pcall(function()
            return math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
        end)
        if okP and pingLbl then pingLbl.Text = ("Ping: %dms"):format(p) end
    end
end)

UIS.JumpRequest:Connect(function()
    if INFINITE_JUMP then
        local h = getHumanoid()
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

RunService.Heartbeat:Connect(function()
    if not FLY_ATIVO then return end
    local r = getRoot()
    if not r or not flyBodyVel then return end
    local cam = workspace.CurrentCamera
    local dir = Vector3.zero
    if UIS:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0,1,0) end
    if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then dir = dir - Vector3.new(0,1,0) end
    flyBodyVel.Velocity = dir.Magnitude > 0 and dir.Unit * 80 or Vector3.zero
end)

RunService.Stepped:Connect(function()
    if not NOCLIP_ATIVO then return end
    local c = LocalPlayer.Character
    if c then
        for _, p in ipairs(c:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(0.05)
        if CLICK_SPAM then
            pcall(function()
                local vu = game:GetService("VirtualUser")
                vu:CaptureController()
                vu:ClickButton1(Vector2.new(0,0))
            end)
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(2)
        if CHAT_SPAM then
            pcall(function()
                game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(CHAT_SPAM_MSG, "All")
            end)
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(0.3)
        if ESP_ATIVO then
            local players = Players:GetPlayers()
            local seen = {}
            for _, pl in ipairs(players) do
                if pl ~= LocalPlayer and pl.Character then
                    local hrp = pl.Character:FindFirstChild("HumanoidRootPart")
                    local hum = pl.Character:FindFirstChildOfClass("Humanoid")
                    if hrp and hum then
                        seen[pl] = true
                        if not currentESP[pl] or not currentESP[pl].Parent then
                            local bbg = Instance.new("BillboardGui")
                            bbg.Size = UDim2.new(0,200,0,60)
                            bbg.StudsOffset = Vector3.new(0,3,0)
                            bbg.AlwaysOnTop = true
                            bbg.Adornee = hrp
                            bbg.Parent = gui
                            local frame = Instance.new("Frame")
                            frame.Size = UDim2.new(1,0,1,0); frame.BackgroundTransparency = 1
                            frame.Parent = bbg
                            local nameL = Instance.new("TextLabel")
                            nameL.Name = "NameL"
                            nameL.Size = UDim2.new(1,0,0,20); nameL.BackgroundTransparency = 1
                            nameL.Text = pl.Name; nameL.TextColor3 = THEME.accent
                            nameL.TextStrokeTransparency = 0; nameL.TextStrokeColor3 = Color3.new(0,0,0)
                            nameL.Font = Enum.Font.GothamBold; nameL.TextSize = 14
                            nameL.Parent = frame
                            local hpBg = Instance.new("Frame")
                            hpBg.Name = "HpBg"
                            hpBg.Size = UDim2.new(0.8,0,0,6); hpBg.Position = UDim2.new(0.1,0,0,22)
                            hpBg.BackgroundColor3 = Color3.new(0,0,0); hpBg.BorderSizePixel = 0
                            hpBg.Parent = frame; uiCorner(hpBg,3)
                            local hpFill = Instance.new("Frame")
                            hpFill.Name = "HpFill"
                            hpFill.Size = UDim2.new(1,0,1,0); hpFill.BackgroundColor3 = THEME.success
                            hpFill.BorderSizePixel = 0; hpFill.Parent = hpBg; uiCorner(hpFill,3)
                            currentESP[pl] = bbg
                        else
                            local bbg = currentESP[pl]
                            local frame = bbg:FindFirstChildOfClass("Frame")
                            if frame then
                                local nameL = frame:FindFirstChild("NameL")
                                local hpBg = frame:FindFirstChild("HpBg")
                                local hpFill = hpBg and hpBg:FindFirstChild("HpFill")
                                if nameL then nameL.Text = pl.Name; nameL.Visible = ESP_NAME end
                                if hpFill and hpBg then
                                    hpFill.Size = UDim2.new(math.clamp(hum.Health/hum.MaxHealth,0,1),0,1,0)
                                    hpFill.BackgroundColor3 = hum.Health > 50 and THEME.success or (hum.Health > 25 and THEME.warn or THEME.danger)
                                    hpBg.Visible = ESP_HEALTH
                                end
                            end
                            bbg.Adornee = hrp
                        end
                    end
                end
            end
            for pl, bbg in pairs(currentESP) do
                if not seen[pl] then
                    if bbg and bbg.Parent then bbg:Destroy() end
                    currentESP[pl] = nil
                end
            end
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(3)
        local ok, l = pcall(varrerEntregas)
        if ok and l and deliveryCountLbl and deliveryMaxLbl then
            if #l > 0 then
                deliveryCountLbl.Text = ("%d >= R$ %s"):format(#l, fmtVal(DELIVERY_THRESHOLDS[DELIVERY_MIN_IDX]))
                deliveryCountLbl.TextColor3 = THEME.gold
                deliveryMaxLbl.Text = "Maior: R$ "..fmtVal(l[1].valor)
            else
                deliveryCountLbl.Text = "Nenhuma entrega"
                deliveryCountLbl.TextColor3 = THEME.textMuted
                deliveryMaxLbl.Text = "Maior: --"
            end
        end
    end
end)

local scanAcc, tpAcc = 0, 0
RunService.Heartbeat:Connect(function(dt)
    if #grudadas > 0 then
        local v = {}
        local precisaReorg = false
        for _,g in ipairs(grudadas) do
            if g.part and g.part.Parent and g.weld and g.weld.Parent then
                table.insert(v, g)
            else
                soltarObjeto(g)
                precisaReorg = true
            end
        end
        if #v ~= #grudadas then
            grudadas = v
            if precisaReorg then
                local r = getRoot()
                if r then
                    for i,g in ipairs(grudadas) do
                        if g.part and g.part.Parent and g.weld and g.weld.Parent then
                            g.part.CFrame = r.CFrame * CFrame.new(calcularOffset(i,#grudadas))
                        end
                    end
                end
            end
            atualizarProgresso()
        end
    end
    if not IMA_ATIVO then return end
    scanAcc = scanAcc + dt
    if scanAcc >= INTERVALO_SCAN then scanAcc = 0; escanearMapa() end
    tpAcc = tpAcc + dt
    if tpAcc < 0.1 then return end
    tpAcc = 0
    local r = getRoot()
    if not r then return end
    if #grudadas >= MAX_PARTS then return end
    local ant = #grudadas
    for _,p2 in ipairs(candidatos) do
        if #grudadas >= MAX_PARTS then break end
        if p2.Parent and not grudadoSet[p2] then grudarPart(p2, r) end
    end
    if #grudadas ~= ant then atualizarProgresso() end
end)

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    soltarTudo()
    if SPEED_ATIVO then local h = getHumanoid(); if h then h.WalkSpeed = SPEED_VAL end end
    if JUMP_ATIVO then local h = getHumanoid(); if h then h.JumpPower = JUMP_VAL end end
end)

-- Watermark
local bd = Instance.new("Frame")
bd.Size = UDim2.new(0,240,0,70); bd.Position = UDim2.new(1,-252,1,-82)
bd.BackgroundColor3 = THEME.glass; bd.BackgroundTransparency = 0.15
bd.ZIndex = 100; bd.Parent = gui
uiCorner(bd,12); uiStroke(bd,THEME.accent,1,0.4)

local bT = Instance.new("TextLabel")
bT.Size = UDim2.new(1,-20,0,18); bT.Position = UDim2.new(0,10,0,6)
bT.BackgroundTransparency = 1; bT.Text = HUB_NAME
bT.TextColor3 = THEME.text; bT.Font = Enum.Font.GothamBold; bT.TextSize = 12
bT.TextXAlignment = Enum.TextXAlignment.Left; bT.ZIndex = 101; bT.Parent = bd

local bK = Instance.new("TextButton")
bK.Size = UDim2.new(1,-20,0,22); bK.Position = UDim2.new(0,10,0,26)
bK.BackgroundColor3 = THEME.discord; bK.BackgroundTransparency = 0.7
bK.Text = " Comprar key no Discord"
bK.TextColor3 = Color3.fromRGB(200,210,255); bK.Font = Enum.Font.GothamBold
bK.TextSize = 10; bK.TextXAlignment = Enum.TextXAlignment.Left
bK.ZIndex = 101; bK.Parent = bd
uiCorner(bK,6); uiStroke(bK,THEME.discord,1,0.4); pressAnim(bK)

bK.MouseButton1Click:Connect(function()
    local ok = pcall(function() if setclipboard then setclipboard(DISCORD_CONTACT) end end)
    if ok then
        bK.Text = " Copiado!"
        task.wait(2)
        bK.Text = " Comprar key no Discord"
    end
end)

local bB = Instance.new("TextLabel")
bB.Size = UDim2.new(1,-20,0,14); bB.Position = UDim2.new(0,10,0,52)
bB.BackgroundTransparency = 1; bB.Text = CREATOR.." | "..HUB_VER.." | "..HUB_TAG
bB.TextColor3 = THEME.textDim; bB.Font = Enum.Font.Gotham; bB.TextSize = 9
bB.TextXAlignment = Enum.TextXAlignment.Left; bB.ZIndex = 101; bB.Parent = bd

print(("[SERTANEJO] %s %s OK"):format(HUB_NAME, HUB_VER))
notif(HUB_NAME.." "..HUB_VER.." carregado!", THEME.accent, 4)
