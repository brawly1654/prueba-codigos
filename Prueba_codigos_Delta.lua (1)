-- PRUEBA LOCAL v1.1 | Copia TODO este archivo en Delta y ejecutalo.
-- Cierra el panel de Delta antes de pulsar PROBAR. Usa el celular horizontal.
-- No escucha anuncios reales, no accede a remotos ni al cuadro Codes del juego.
-- "Teclas" prueba eventos virtuales; "Visual" modifica SOLO nuestro TextBox.
-- "Texto API" envia un caracter por evento de texto: NO son pulsaciones.
-- No verifica identidad real de Sammy ni compatibilidad con el canje del juego.

-- CORE_BEGIN (logica independiente, comprobada fuera de Roblox)
local Core = {}
function Core.new(count, source)
    assert(count >= 1 and count <= 12 and count == math.floor(count), "Cantidad invalida")
    return {count = count, source = source, parts = {}, active = false, deadline = 0}
end
function Core.feed(s, source, text, now)
    if source ~= s.source then return nil, "IGNORADO: otro emisor simulado" end
    if s.active and now > s.deadline then
        s.active, s.parts = false, {}
    end
    text = tostring(text):match("^%s*(.-)%s*$")
    local lower = text:lower()
    local marker = lower:match("^code%s+is$") or lower:match("^code%s+is[%s%.:]")
    if marker then
        s.parts, s.active, s.deadline = {}, true, now + 12
        text = text:gsub("^[Cc][Oo][Dd][Ee]%s+[Ii][Ss][%s%.:]*", "")
        if text == "" then return nil, "INICIO reconocido" end
    elseif not s.active then
        return nil, "IGNORADO: falta anuncio de inicio o expiro"
    end
    -- En esta prueba solo se admiten fragmentos alfanumericos sin espacios.
    if not text:match("^[A-Za-z0-9]+$") then return nil, "IGNORADO: no es un fragmento valido" end
    table.insert(s.parts, text)
    local complete = #s.parts == s.count
    if complete then s.active = false end
    return {chunk = text, code = table.concat(s.parts), complete = complete,
        index = #s.parts}, complete and "CODIGO COMPLETO" or "Fragmento aceptado"
end
-- CORE_END

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
-- INPUT_HELPERS_BEGIN
local function waitFor(predicate, seconds, check)
    local deadline=os.clock()+seconds
    repeat
        check()
        if predicate() then return true end
        RS.Heartbeat:Wait()
    until os.clock()>=deadline
    check()
    return predicate()
end
local function chooseBackend(candidates, probe, record, check)
    for _,candidate in ipairs(candidates) do
        check()
        record("Comprobando "..candidate.name)
        local ok,err=pcall(probe,candidate)
        check()
        if ok then
            record("API COMPROBADA="..candidate.name)
            return candidate
        end
        record(candidate.name.." FALLO: "..tostring(err))
    end
    error("Ningun metodo del modo elegido escribio correctamente. Revisa los fallos anteriores.",0)
end
-- INPUT_HELPERS_END
local player = Players.LocalPlayer
assert(player, "Ejecuta esta prueba desde el cliente")
local parent = player:WaitForChild("PlayerGui", 10)
assert(parent, "No se encontro PlayerGui; espera a entrar al juego")
local previous = parent:FindFirstChild("PruebaLocalCodigosV1")
if previous then previous:Destroy() end

local function make(class, properties, owner)
    local object = Instance.new(class)
    for k, v in pairs(properties) do object[k] = v end
    object.Parent = owner
    return object
end
local gui = make("ScreenGui", {Name="PruebaLocalCodigosV1", ResetOnSpawn=false,
    DisplayOrder=900, ZIndexBehavior=Enum.ZIndexBehavior.Sibling}, parent)
local panel = make("Frame", {AnchorPoint=Vector2.new(0.5,0.5),
    Position=UDim2.fromScale(0.5,0.5), Size=UDim2.fromScale(0.92,0.9),
    BackgroundColor3=Color3.fromRGB(20,25,35), BorderSizePixel=0}, gui)
make("UISizeConstraint", {MaxSize=Vector2.new(660,650)}, panel)
make("UICorner", {CornerRadius=UDim.new(0,12)}, panel)
local close = make("TextButton", {Text="X", Size=UDim2.fromOffset(40,32),
    Position=UDim2.new(1,-48,0,6), BackgroundColor3=Color3.fromRGB(120,45,55),
    TextColor3=Color3.new(1,1,1), TextSize=18}, panel)
local scroll = make("ScrollingFrame", {Position=UDim2.fromOffset(12,90),
    Size=UDim2.new(1,-24,1,-152), CanvasSize=UDim2.new(0,0,0,712),
    ScrollBarThickness=6, BackgroundTransparency=1, BorderSizePixel=0}, panel)
local function item(class, text, x, y, w, h)
    local obj = make(class, {Text=text, Position=UDim2.new(x,0,0,y),
        Size=UDim2.new(w,-6,0,h), Font=Enum.Font.Gotham, TextSize=14,
        TextColor3=Color3.fromRGB(235,240,248), BorderSizePixel=0,
        BackgroundColor3=Color3.fromRGB(40,49,65)}, scroll)
    if class == "TextLabel" then
        obj.BackgroundTransparency=1
        obj.TextXAlignment=Enum.TextXAlignment.Left
        obj.TextWrapped=true
    else
        make("UICorner", {CornerRadius=UDim.new(0,6)}, obj)
    end
    if class == "TextBox" then obj.ClearTextOnFocus=false end
    return obj
end
make("TextLabel", {Text="PRUEBA LOCAL v1.1", Position=UDim2.fromOffset(12,6),
    Size=UDim2.new(1,-70,0,32), Font=Enum.Font.GothamBold, TextSize=14,
    TextXAlignment=Enum.TextXAlignment.Left, BackgroundTransparency=1,
    TextColor3=Color3.fromRGB(110,220,200)}, panel)
item("TextLabel", "Sin evento: simula mensajes y escribe solo en el campo de prueba.", 0,0,1,32)
item("TextLabel", "Fragmentos (1-12)",0,38,0.5,20)
item("TextLabel", "Pausa por tecla, ms (10-300)",0.5,38,0.5,20)
local countBox = item("TextBox","3",0,62,0.5,34)
local delayBox = item("TextBox","30",0.5,62,0.5,34)
local markerButton = item("TextButton","Inicio: code is...",0,104,0.5,36)
local formatButton = item("TextButton","Texto: Original",0.5,104,0.5,36)
local modeButton = item("TextButton","Modo: Teclas",0,148,0.5,36)
local enterButton = item("TextButton","Enter automatico: SI",0.5,148,0.5,36)
item("TextLabel","Un fragmento por linea. La cantidad debe coincidir con las lineas.",0,190,1,30)
local fragmentsBox = item("TextBox","Free\nMeowl\n12345",0,224,1,80)
fragmentsBox.MultiLine=true
fragmentsBox.TextYAlignment=Enum.TextYAlignment.Top
local runButton = item("TextButton","PROBAR",0,312,0.5,40)
runButton.BackgroundColor3=Color3.fromRGB(21,115,100)
local stopButton = item("TextButton","DETENER",0.5,312,0.5,40)
runButton.Parent=panel
runButton.Position=UDim2.new(0,12,1,-50)
runButton.Size=UDim2.new(0.5,-18,0,40)
stopButton.Parent=panel
stopButton.Position=UDim2.new(0.5,6,1,-50)
stopButton.Size=UDim2.new(0.5,-18,0,40)
item("TextLabel","Campo local de prueba (NO es el canje del juego)",0,360,1,24)
local output = item("TextBox","",0,388,1,40)
output.PlaceholderText="Aqui aparecera el codigo"
output.MultiLine=false
local status = item("TextLabel","Listo. Prueba Teclas. Si falla, copia el informe y prueba Texto API.",0,434,1,42)
status.Parent=panel
status.Position=UDim2.fromOffset(12,40)
status.Size=UDim2.new(1,-24,0,42)
local logBox = item("TextBox","",0,482,1,150)
logBox.MultiLine=true
logBox.TextEditable=false
logBox.TextXAlignment=Enum.TextXAlignment.Left
logBox.TextYAlignment=Enum.TextYAlignment.Top
logBox.TextSize=12
logBox.TextWrapped=true
local copyButton = item("TextButton","COPIAR INFORME",0,640,1,36)
item("TextLabel","Teclas = pulsaciones. Texto API = caracteres por evento de texto. Visual = animacion.",0,680,1,30)

local markers = {"code is...", "code is", "CODE IS..."}
local formats = {"Original", "MAYUSCULAS", "minusculas"}
local modes = {"Teclas", "Texto API", "Visual"}
local modeIndex=1
local markerIndex, formatIndex, mode, autoEnter = 1,1,"Teclas",true
local running, alive, generation = false,true,0
local lines, entered, started, submittedText = {},false,0,nil
local function log(text)
    if not alive then return end
    table.insert(lines, string.format("%.2fs %s", os.clock()-started, text))
    if #lines > 45 then table.remove(lines,1) end
    logBox.Text=table.concat(lines,"\n")
end
local function transform(text, index)
    if index == 2 then return text:upper() end
    if index == 3 then return text:lower() end
    return text
end
markerButton.Activated:Connect(function()
    if running then return end
    markerIndex=markerIndex % #markers+1
    markerButton.Text="Inicio: "..markers[markerIndex]
end)
formatButton.Activated:Connect(function()
    if running then return end
    formatIndex=formatIndex % #formats+1
    formatButton.Text="Texto: "..formats[formatIndex]
end)
modeButton.Activated:Connect(function()
    if running then return end
    modeIndex=modeIndex % #modes+1
    mode=modes[modeIndex]
    modeButton.Text="Modo: "..mode
end)
enterButton.Activated:Connect(function()
    if running then return end
    autoEnter=not autoEnter
    enterButton.Text="Enter automatico: "..(autoEnter and "SI" or "NO")
end)
output.FocusLost:Connect(function(isEnter)
    if running and mode~="Visual" and isEnter then
        entered, submittedText = true,output.Text
        log("Enter observado por FocusLost; texto="..output.Text)
    end
end)
local function releaseOwnFocus()
    if UIS:GetFocusedTextBox()==output then output:ReleaseFocus(false) end
end
local function cancel()
    generation=generation+1
    running=false
    releaseOwnFocus()
    if alive then status.Text="Detenido. Puedes volver a probar." end
end
stopButton.Activated:Connect(cancel)
close.Activated:Connect(function() gui:Destroy() end)
gui.Destroying:Connect(function()
    cancel()
    alive=false
end)
copyButton.Activated:Connect(function()
    if running then return end
    local report="PRUEBA LOCAL v1.1\n"..status.Text.."\n"..table.concat(lines,"\n")
    local ok = pcall(function()
        assert(type(setclipboard)=="function", "Portapapeles no disponible")
        setclipboard(report)
    end)
    copyButton.Text=ok and "INFORME COPIADO" or "Sin portapapeles: toma una captura"
end)

runButton.Activated:Connect(function()
    if running then return end
    local count, milliseconds = tonumber(countBox.Text),tonumber(delayBox.Text)
    local parts={}
    for line in (fragmentsBox.Text.."\n"):gmatch("([^\n]*)\n") do
        line=line:match("^%s*(.-)%s*$")
        if line~="" then table.insert(parts,line) end
    end
    if not count or count~=math.floor(count) or count<1 or count>12 then
        status.Text="Cantidad: escribe un entero entre 1 y 12."; return
    end
    if not milliseconds or milliseconds<10 or milliseconds>300 then
        status.Text="Pausa: escribe un numero entre 10 y 300 ms."; return
    end
    if #parts~=count then status.Text="La cantidad debe coincidir con las lineas de fragmentos."; return end
    for _,part in ipairs(parts) do
        if not part:match("^[A-Za-z0-9]+$") or #part>40 then
            status.Text="Cada fragmento: 1-40 letras A-Z o numeros, sin espacios."; return
        end
    end
    generation=generation+1
    local token=generation
    local runMode, runEnter, runFormat = mode,autoEnter,formatIndex
    local marker, pause = markers[markerIndex],milliseconds/1000
    running, entered, submittedText = true,false,nil
    started, lines = os.clock(),{}
    output.Text=""
    copyButton.Text="COPIAR INFORME"
    status.Text="Probando. Espera sin tocar otros campos; puede tardar varios segundos."
    local function check()
        assert(alive and running and generation==token,"Prueba cancelada")
    end
    task.spawn(function()
        local backend, send, sendText
        local ok, err = pcall(function()
            log("Modo="..runMode.."; fragmentos="..count.."; pausa="..milliseconds.." ms")
            local function tap(key, shifted)
                check()
                assert(UIS:GetFocusedTextBox()==output,"Foco cambiado: detenido antes de enviar otra tecla.")
                local pressed = {}
                local keyOK,keyErr=pcall(function()
                    if shifted then
                        table.insert(pressed,Enum.KeyCode.LeftShift)
                        send(true,Enum.KeyCode.LeftShift)
                    end
                    table.insert(pressed,key)
                    send(true,key)
                    RS.Heartbeat:Wait()
                end)
                -- Siempre soltar las teclas, incluso si se cancela durante la espera.
                local released=true
                for i=#pressed,1,-1 do
                    local upOK=pcall(send,false,pressed[i])
                    released=released and upOK
                end
                assert(keyOK,tostring(keyErr))
                assert(released,"La API rechazo soltar una tecla")
                check()
            end
            local digits={"Zero","One","Two","Three","Four","Five","Six","Seven","Eight","Nine"}
            local function keyFor(char)
                local digit=tonumber(char)
                return Enum.KeyCode[digit and digits[digit+1] or char:upper()]
            end
            local function focusLocal()
                check()
                output:CaptureFocus()
                task.wait(0.45)
                check()
                assert(UIS:GetFocusedTextBox()==output,"No se pudo enfocar el campo local. Cierra Delta y repite.")
                output.CursorPosition=#output.Text+1
                output.SelectionStart=-1
            end
            local function inject(char)
                check()
                assert(UIS:GetFocusedTextBox()==output,"Foco cambiado: escritura detenida.")
                if runMode=="Texto API" then
                    sendText(char)
                else
                    tap(keyFor(char),char:match("^[A-Z]$")~=nil)
                end
            end
            if runMode~="Visual" then
                local candidates={}
                local available,virtual=pcall(function() return UIS:CreateVirtualInput() end)
                if available and virtual then
                    table.insert(candidates,{
                        name=runMode=="Texto API" and "VirtualInput.SendTextInput" or "VirtualInput.SendKey",
                        send=function(down,key) virtual:SendKey(down,key,false) end,
                        text=function(char) virtual:SendTextInput(char) end,
                    })
                else
                    log("CreateVirtualInput no disponible: "..tostring(virtual))
                end
                if runMode=="Teclas" then
                    local legacyOK,manager=pcall(function() return game:GetService("VirtualInputManager") end)
                    if legacyOK and manager then
                        table.insert(candidates,{
                            name="VirtualInputManager.SendKeyEvent",
                            send=function(down,key) manager:SendKeyEvent(down,key,false,game) end,
                        })
                    else
                        log("VirtualInputManager no disponible: "..tostring(manager))
                    end
                else
                    log("TEXTO API: un caracter por evento de texto; no son pulsaciones.")
                end
                backend=chooseBackend(candidates,function(candidate)
                    send,sendText=candidate.send,candidate.text
                    releaseOwnFocus()
                    task.wait(0.2)
                    check()
                    output.Text=""
                    focusLocal()
                    local probe="aB1"
                    for i=1,#probe do
                        local expectedProbe=probe:sub(1,i)
                        inject(probe:sub(i,i))
                        local matches=waitFor(function() return output.Text==expectedProbe end,0.6,check)
                        assert(matches,"Sondeo: esperado ["..expectedProbe.."], observado ["..output.Text.."], foco local="..tostring(UIS:GetFocusedTextBox()==output))
                    end
                end,log,check)
                send,sendText=backend.send,backend.text
                releaseOwnFocus()
                task.wait(0.2)
                check()
                output.Text=""
                focusLocal()
            else
                log("VISUAL: modifica Text; no verifica pulsaciones ni Enter real.")
            end
            local expected=""
            local function write(chunk)
                chunk=transform(chunk,runFormat)
                for i=1,#chunk do
                    check()
                    local char=chunk:sub(i,i)
                    expected=expected..char
                    if runMode=="Visual" then
                        output.Text=expected
                    else
                        inject(char)
                        local matched=waitFor(function() return output.Text==expected end,0.6,check)
                        assert(matched,"Texto diferente: ["..output.Text.."] frente a ["..expected.."]")
                    end
                    task.wait(pause)
                    check()
                    assert(output.Text==expected,"Texto diferente al esperado: ["..output.Text.."] frente a ["..expected.."]. No se corrige automaticamente.")
                end
            end
            -- Objetos locales: NO son identidades de jugadores ni una validacion real de Sammy.
            local sammy, other = {},{}
            local state=Core.new(count,sammy)
            local simulatedTime=0
            local function feed(source,text)
                check()
                simulatedTime=simulatedTime+0.15
                local event,reason=Core.feed(state,source,text,simulatedTime)
                log((source==sammy and "Sammy simulado: " or "Otro simulado: ")..text.." -> "..reason)
                if event then
                    write(event.chunk)
                    if event.complete then
                        assert(expected==transform(event.code,runFormat),"Error de ensamblado")
                        if runEnter then
                            if runMode~="Visual" then
                                tap(Enum.KeyCode.Return,false)
                                local enterOK=waitFor(function() return entered and submittedText==expected end,0.8,check)
                                assert(enterOK,"El texto coincide, pero no se observo Enter con el codigo completo.")
                            else
                                log("ENVIO SIMULADO="..expected.." (sin pulsar Enter)")
                            end
                        end
                    end
                end
                return event
            end
            feed(other,"code is...")
            feed(sammy,"IGNORARANTES")
            feed(sammy,marker)
            for _,part in ipairs(parts) do
                feed(other,"NOINSERTAR")
                feed(sammy,part)
                task.wait(0.15)
            end
            assert(not state.active and #state.parts==count,"Secuencia incompleta")
            local extra=feed(sammy,"NOAGREGAR")
            assert(not extra and output.Text==expected,"Se agrego texto despues del final")
            local method=runMode=="Texto API" and "TEXTO API" or "TECLAS"
            status.Text=runMode=="Visual" and "LOGICA OK | teclas y canje real pendientes" or
                (runEnter and method.." + ENTER OK | solo campo local" or method.." OK | Enter desactivado")
            log("RESULTADO="..output.Text)
        end)
        if not alive or generation~=token then return end
        if not ok then
            status.Text="PRUEBA FALLIDA. Copia el informe. Puedes cambiar el modo y repetir."
            log("ERROR="..tostring(err))
        end
        running=false
        releaseOwnFocus()
    end)
end)
