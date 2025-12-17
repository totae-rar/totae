local Players=game:GetService("Players")
local RunService=game:GetService("RunService")
local UIS=game:GetService("UserInputService")
local LP=Players.LocalPlayer
local Cam=workspace.CurrentCamera
local ESP_ENABLED=true
local ESP_RADIUS=250
local ESP_COLOR=Color3.fromRGB(255,80,80)
local AIMBOT_ENABLED=false
local AIMBOT_ACTIVE=false
local AIMBOT_SMOOTHNESS=0.15
local FOV_VISIBLE=false
local FOV_RADIUS=120
local esp={}
local function newESP(c)
local h=Instance.new("Highlight")
h.FillColor=ESP_COLOR
h.FillTransparency=0.5
h.OutlineTransparency=0
h.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
h.Adornee=c
h.Parent=workspace
return h
end
RunService.RenderStepped:Connect(function()
if not ESP_ENABLED then
for _,h in pairs(esp)do h:Destroy()end
table.clear(esp)
return
end
for _,p in ipairs(Players:GetPlayers())do
if p==LP then continue end
local c=p.Character
local hum=c and c:FindFirstChildOfClass("Humanoid")
local root=c and c:FindFirstChild("HumanoidRootPart")
if not(c and hum and root and hum.Health>0)then
if esp[c]then esp[c]:Destroy()esp[c]=nil end
continue
end
local d=(Cam.CFrame.Position-root.Position).Magnitude
if d<=ESP_RADIUS then
if not esp[c]then esp[c]=newESP(c)end
esp[c].FillColor=ESP_COLOR
elseif esp[c]then
esp[c]:Destroy()
esp[c]=nil
end
end
end)
local function nearestHead()
local best=nil
local bd=nil
for _,p in ipairs(Players:GetPlayers())do
if p==LP then continue end
local c=p.Character
local hum=c and c:FindFirstChildOfClass("Humanoid")
local head=c and c:FindFirstChild("Head")
if hum and head and hum.Health>0 then
local d=(Cam.CFrame.Position-head.Position).Magnitude
if not bd or d<bd then
bd=d
best=head
end
end
end
return best
end
RunService.RenderStepped:Connect(function()
if AIMBOT_ENABLED and AIMBOT_ACTIVE then
local h=nearestHead()
if h then
local cp=Cam.CFrame.Position
local dir=(h.Position-cp).Unit
local look=Cam.CFrame.LookVector:Lerp(dir,AIMBOT_SMOOTHNESS)
Cam.CFrame=CFrame.new(cp,cp+look)
end
end
end)
UIS.InputBegan:Connect(function(i,gp)
if gp then return end
if i.KeyCode==Enum.KeyCode.J and AIMBOT_ENABLED then
AIMBOT_ACTIVE=not AIMBOT_ACTIVE
end
end)
local gui=LP.PlayerGui:FindFirstChild("CleanGUI")or Instance.new("ScreenGui",LP.PlayerGui)
gui.Name="CleanGUI"
gui.ResetOnSpawn=false
gui.DisplayOrder=1000000
gui.IgnoreGuiInset=true
gui.ZIndexBehavior=Enum.ZIndexBehavior.Global
gui:ClearAllChildren()
local main=Instance.new("Frame",gui)
main.Size=UDim2.fromScale(0.28,0.7)
main.Position=UDim2.fromScale(0.05,0.15)
main.BackgroundColor3=Color3.fromRGB(20,20,25)
main.Active=true
main.Draggable=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,18)
Instance.new("UIStroke",main).Color=Color3.fromRGB(60,60,70)
local title=Instance.new("TextLabel",main)
title.Size=UDim2.fromScale(1,0.1)
title.BackgroundTransparency=1
title.Text="ToTAElly safe GUI"
title.Font=Enum.Font.GothamBold
title.TextScaled=true
title.TextColor3=Color3.fromRGB(235,235,235)
local content=Instance.new("Frame",main)
content.Size=UDim2.fromScale(1,0.8)
content.Position=UDim2.fromScale(0,0.1)
content.BackgroundTransparency=1
local layout=Instance.new("UIListLayout",content)
layout.Padding=UDim.new(0,10)
layout.HorizontalAlignment=Enum.HorizontalAlignment.Center
local function toggle(text,state,cb)
local row=Instance.new("Frame",content)
row.Size=UDim2.new(0.92,0,0,40)
row.BackgroundColor3=Color3.fromRGB(30,30,36)
Instance.new("UICorner",row).CornerRadius=UDim.new(0,12)
local label=Instance.new("TextLabel",row)
label.Size=UDim2.fromScale(0.7,1)
label.Position=UDim2.fromScale(0.05,0)
label.BackgroundTransparency=1
label.Text=text
label.Font=Enum.Font.GothamSemibold
label.TextScaled=true
label.TextXAlignment=Enum.TextXAlignment.Left
label.TextColor3=Color3.fromRGB(230,230,230)
local btn=Instance.new("TextButton",row)
btn.Size=UDim2.fromScale(0.22,0.6)
btn.Position=UDim2.fromScale(0.73,0.2)
btn.Text=state and "ON" or "OFF"
btn.BackgroundColor3=state and Color3.fromRGB(0,170,120)or Color3.fromRGB(170,60,60)
btn.TextScaled=true
btn.Font=Enum.Font.GothamBold
btn.TextColor3=Color3.new(1,1,1)
Instance.new("UICorner",btn).CornerRadius=UDim.new(1,0)
btn.MouseButton1Click:Connect(function()
state=not state
btn.Text=state and "ON" or "OFF"
btn.BackgroundColor3=state and Color3.fromRGB(0,170,120)or Color3.fromRGB(170,60,60)
cb(state)
end)
end
local function radiusBox()
local row=Instance.new("Frame",content)
row.Size=UDim2.new(0.92,0,0,40)
row.BackgroundColor3=Color3.fromRGB(30,30,36)
Instance.new("UICorner",row).CornerRadius=UDim.new(0,12)
local box=Instance.new("TextBox",row)
box.Size=UDim2.fromScale(0.9,0.7)
box.Position=UDim2.fromScale(0.05,0.15)
box.Text=tostring(ESP_RADIUS)
box.PlaceholderText="ESP Radius"
box.Font=Enum.Font.GothamSemibold
box.TextScaled=true
box.BackgroundColor3=Color3.fromRGB(40,40,46)
box.TextColor3=Color3.new(1,1,1)
Instance.new("UICorner",box)
box.FocusLost:Connect(function()
local n=tonumber(box.Text)
if n then ESP_RADIUS=math.clamp(n,10,2000)end
box.Text=tostring(ESP_RADIUS)
end)
end
local function colorButtons()
local row=Instance.new("Frame",content)
row.Size=UDim2.new(0.92,0,0,40)
row.BackgroundTransparency=1
local grid=Instance.new("UIGridLayout",row)
grid.CellSize=UDim2.new(0.23,0,1,0)
grid.CellPadding=UDim2.new(0.02,0,0,0)
local colors={{"Red",Color3.fromRGB(255,80,80)},{"Blue",Color3.fromRGB(80,170,255)},{"Green",Color3.fromRGB(80,255,160)},{"Yellow",Color3.fromRGB(255,220,80)}}
for _,v in ipairs(colors)do
local b=Instance.new("TextButton",row)
b.Text=v[1]
b.Font=Enum.Font.GothamBold
b.TextScaled=true
b.TextColor3=Color3.new(1,1,1)
b.BackgroundColor3=Color3.fromRGB(45,45,52)
Instance.new("UICorner",b).CornerRadius=UDim.new(1,0)
b.MouseButton1Click:Connect(function()
ESP_COLOR=v[2]
for _,h in pairs(esp)do h.FillColor=ESP_COLOR end
end)
end
end
local function fovSlider()
local row=Instance.new("Frame",content)
row.Size=UDim2.new(0.92,0,0,46)
row.BackgroundColor3=Color3.fromRGB(30,30,36)
Instance.new("UICorner",row).CornerRadius=UDim.new(0,12)
local bar=Instance.new("Frame",row)
bar.Size=UDim2.fromScale(0.9,0.25)
bar.Position=UDim2.fromScale(0.05,0.6)
bar.BackgroundColor3=Color3.fromRGB(45,45,52)
Instance.new("UICorner",bar).CornerRadius=UDim.new(1,0)
local fill=Instance.new("Frame",bar)
fill.BackgroundColor3=Color3.fromRGB(0,170,120)
fill.Size=UDim2.fromScale(FOV_RADIUS/300,1)
Instance.new("UICorner",fill).CornerRadius=UDim.new(1,0)
local dot=Instance.new("Frame",bar)
dot.Size=UDim2.fromOffset(14,14)
dot.AnchorPoint=Vector2.new(0.5,0.5)
dot.Position=UDim2.fromScale(fill.Size.X.Scale,0.5)
dot.BackgroundColor3=Color3.fromRGB(235,235,235)
Instance.new("UICorner",dot).CornerRadius=UDim.new(1,0)
local dragging=false
local function update()
local x=(UIS:GetMouseLocation().X-bar.AbsolutePosition.X)/bar.AbsoluteSize.X
x=math.clamp(x,0,1)
FOV_RADIUS=math.floor(50+250*x)
fill.Size=UDim2.fromScale(x,1)
dot.Position=UDim2.fromScale(x,0.5)
end
bar.InputBegan:Connect(function(i)
if i.UserInputType==Enum.UserInputType.MouseButton1 then
dragging=true
main.Draggable=false
update()
end
end)
UIS.InputEnded:Connect(function(i)
if i.UserInputType==Enum.UserInputType.MouseButton1 then
dragging=false
main.Draggable=true
end
end)
RunService.RenderStepped:Connect(function()
if dragging then update()end
end)
end
local fov=Instance.new("Frame",gui)
fov.BackgroundTransparency=1
fov.BorderSizePixel=0
local stroke=Instance.new("UIStroke",fov)
stroke.Color=Color3.fromRGB(255,255,255)
stroke.Thickness=1.5
Instance.new("UICorner",fov).CornerRadius=UDim.new(1,0)
RunService.RenderStepped:Connect(function()
fov.Visible=FOV_VISIBLE
fov.Size=UDim2.fromOffset(FOV_RADIUS*2,FOV_RADIUS*2)
local vp=Cam.ViewportSize
fov.Position=UDim2.fromOffset(vp.X/2-FOV_RADIUS,vp.Y/2-FOV_RADIUS)
end)
toggle("ESP",ESP_ENABLED,function(v)ESP_ENABLED=v end)
radiusBox()
colorButtons()
toggle("Aimbot (J)",AIMBOT_ENABLED,function(v)AIMBOT_ENABLED=v if not v then AIMBOT_ACTIVE=false end end)
toggle("FOV Circle",FOV_VISIBLE,function(v)FOV_VISIBLE=v end)
fovSlider()
local thanks=Instance.new("TextLabel",main)
thanks.Size=UDim2.fromScale(1,0.05)
thanks.Position=UDim2.fromScale(0,0.95)
thanks.BackgroundTransparency=1
thanks.Text="Thanks for using <3"
thanks.Font=Enum.Font.Gotham
thanks.TextScaled=true
thanks.TextColor3=Color3.fromRGB(180,180,180)
