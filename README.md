```lua
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local function create(className, props, parent)
	local obj = Instance.new(className)
	if props then
		for k,v in pairs(props) do
			if k == "Children" then
				for _, child in pairs(v) do
					child.Parent = obj
				end
			elseif k == "Parent" then
				-- ignore, set later
			else
				obj[k] = v
			end
		end
	end
	if parent then obj.Parent = parent end
	return obj
end

local function rgbRainbow(t, speed)
	speed = speed or 1
	local hue = (tick()*speed + t) % 1
	local r, g, b = Color3.fromHSV(hue,1,1):ToRGB()
	return Color3.new(r/255,g/255,b/255)
end

local function createRoundedFrame(props, parent)
	local frame = create("Frame", {
		BackgroundColor3 = props.BackgroundColor3 or Color3.new(0.1,0.1,0.1),
		BorderSizePixel = 0,
		Size = props.Size or UDim2.new(0,200,0,200),
		Position = props.Position or UDim2.new(0,0,0,0),
		Parent = parent
	})
	local corner = create("UICorner", {CornerRadius = props.CornerRadius or UDim.new(0,12), Parent = frame})
	return frame
end

local function createGlow(frame, thickness, color)
	thickness = thickness or 4
	color = color or Color3.new(1,1,1)
	local glow = create("ImageLabel", {
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Size = UDim2.new(1,thickness*2,1,thickness*2),
		Position = UDim2.new(0,-thickness,0,-thickness),
		Image = "rbxassetid://3526306059",
		ImageColor3 = color,
		Parent = frame,
		ZIndex = frame.ZIndex - 1,
	})
	glow.ImageTransparency = 0.7
	glow.ScaleType = Enum.ScaleType.Slice
	glow.SliceCenter = Rect.new(10,10,118,118)
	return glow
end

local function createRainbowBorder(frame, thickness)
	thickness = thickness or 3
	local border = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,1,0),
		Position = UDim2.new(0,0,0,0),
		Parent = frame,
		ZIndex = frame.ZIndex + 1,
	})
	local corners = create("UICorner", {CornerRadius = UDim.new(0,12), Parent = border})
	local lines = {}
	local function line(pos,size)
		local l = create("Frame", {
			BackgroundColor3 = Color3.new(1,0,0),
			BorderSizePixel = 0,
			Position = pos,
			Size = size,
			Parent = border,
			ZIndex = border.ZIndex + 1,
		})
		return l
	end
	lines.top = line(UDim2.new(0,0,0,0), UDim2.new(1,0,0,thickness))
	lines.bottom = line(UDim2.new(0,0,1,-thickness), UDim2.new(1,0,0,thickness))
	lines.left = line(UDim2.new(0,0,0,0), UDim2.new(0,thickness,1,0))
	lines.right = line(UDim2.new(1,-thickness,0,0), UDim2.new(0,thickness,1,0))
	RunService.Heartbeat:Connect(function()
		local t = tick()
		for _, l in pairs(lines) do
			local hue = (t + l.Position.X.Offset*0.01) % 1
			l.BackgroundColor3 = Color3.fromHSV(hue,1,1)
		end
	end)
	return border
end

local function createButton(text, size, parent)
	local btn = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = size,
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.GothamSemibold,
		TextSize = 16,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,1,0),
		Parent = btn,
	})
	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
	end)
	btn.MouseLeave:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(30,30,30)}):Play()
	end)
	return btn
end

local function createIconLabel(iconText, size, parent)
	local frame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = size,
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = iconText,
		Font = Enum.Font.GothamBold,
		TextSize = 20,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,1,0),
		Parent = frame,
		TextYAlignment = Enum.TextYAlignment.Center,
		TextXAlignment = Enum.TextXAlignment.Center,
	})
	return frame
end

local function createToggle(text, parent)
	local frame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(0,180,0,30),
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0,10,0,0),
		Size = UDim2.new(1,-40,1,0),
		Parent = frame,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	local toggle = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(50,50,50),
		Size = UDim2.new(0,30,0,18),
		Position = UDim2.new(1,-40,0,6),
		Parent = frame,
		CornerRadius = UDim.new(0,9),
	})
	local circle = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(200,200,200),
		Size = UDim2.new(0,14,0,14),
		Position = UDim2.new(0,2,0,2),
		Parent = toggle,
		CornerRadius = UDim.new(0,7),
	})
	local toggled = false
	local function setToggle(state)
		toggled = state
		if toggled then
			TweenService:Create(circle, TweenInfo.new(0.2), {Position = UDim2.new(1,-16,0,2)}):Play()
			TweenService:Create(toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0,170,255)}):Play()
		else
			TweenService:Create(circle, TweenInfo.new(0.2), {Position = UDim2.new(0,2,0,2)}):Play()
			TweenService:Create(toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
		end
	end
	setToggle(false)
	toggle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			setToggle(not toggled)
		end
	end)
	return frame, function() return toggled end, setToggle
end

local function createSlider(text, min, max, default, parent)
	local frame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(0,180,0,40),
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0,10,0,0),
		Size = UDim2.new(1,-20,0,18),
		Parent = frame,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	local sliderBar = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(50,50,50),
		Size = UDim2.new(1,-20,0,10),
		Position = UDim2.new(0,10,0,25),
		Parent = frame,
		CornerRadius = UDim.new(0,5),
	})
	local fill = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(0,170,255),
		Size = UDim2.new((default-min)/(max-min),0,1,0),
		Parent = sliderBar,
		CornerRadius = UDim.new(0,5),
	})
	local dragging = false
	local value = default
	local function updateValue(x)
		local relative = math.clamp((x - sliderBar.AbsolutePosition.X) / sliderBar.AbsoluteSize.X, 0, 1)
		value = min + (max - min) * relative
		fill.Size = UDim2.new(relative,0,1,0)
	end
	sliderBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			updateValue(input.Position.X)
		end
	end)
	sliderBar.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			updateValue(input.Position.X)
		end
	end)
	return frame, function() return value end
end

local function createDropdown(text, options, parent)
	local frame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(0,180,0,30),
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0,10,0,0),
		Size = UDim2.new(1,-40,1,0),
		Parent = frame,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	local arrow = create("TextLabel", {
		Text = "▼",
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(1,-30,0,0),
		Size = UDim2.new(0,20,1,0),
		Parent = frame,
		TextXAlignment = Enum.TextXAlignment.Center,
		TextYAlignment = Enum.TextYAlignment.Center,
	})
	local dropdownOpen = false
	local selected = options[1]
	local dropdownFrame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(20,20,20),
		Size = UDim2.new(0,180,0,#options*25),
		Position = UDim2.new(0,0,1,2),
		Parent = frame,
		CornerRadius = UDim.new(0,8),
	})
	dropdownFrame.Visible = false
	local buttons = {}
	for i, option in ipairs(options) do
		local btn = createRoundedFrame({
			BackgroundColor3 = Color3.fromRGB(30,30,30),
			Size = UDim2.new(1,0,0,25),
			Position = UDim2.new(0,0,0,(i-1)*25),
			Parent = dropdownFrame,
			CornerRadius = UDim.new(0,6),
		})
		local lbl = create("TextLabel", {
			Text = option,
			Font = Enum.Font.Gotham,
			TextSize = 14,
			TextColor3 = Color3.fromRGB(220,220,220),
			BackgroundTransparency = 1,
			Size = UDim2.new(1,0,1,0),
			Parent = btn,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextYAlignment = Enum.TextYAlignment.Center,
			Position = UDim2.new(0,10,0,0),
		})
		btn.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 then
				selected = option
				label.Text = text .. ": " .. selected
				dropdownFrame.Visible = false
				dropdownOpen = false
			end
		end)
		btn.MouseEnter:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
		end)
		btn.MouseLeave:Connect(function()
			TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(30,30,30)}):Play()
		end)
		buttons[#buttons+1] = btn
	end
	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dropdownOpen = not dropdownOpen
			dropdownFrame.Visible = dropdownOpen
		end
	end)
	label.Text = text .. ": " .. selected
	return frame, function() return selected end
end

local function createTextBox(text, placeholder, parent)
	local frame = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(0,180,0,30),
		Parent = parent,
		CornerRadius = UDim.new(0,8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0,10,0,0),
		Size = UDim2.new(1,-20,0,14),
		Parent = frame,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	local textbox = create("TextBox", {
		BackgroundTransparency = 1,
		Text = "",
		PlaceholderText = placeholder or "",
		Font = Enum.Font.Gotham,
		TextSize = 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		Position = UDim2.new(0,10,0,14),
		Size = UDim2.new(1,-20,0,16),
		Parent = frame,
		ClearTextOnFocus = false,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	return frame, textbox
end

local function createLabel(text, size, parent)
	return create("TextLabel", {
		Text = text,
		Font = Enum.Font.Gotham,
		TextSize = size or 14,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,0,20),
		Parent = parent,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
end

-- Main GUI container
local screenGui = create("ScreenGui", {Name = "PremiumFuturisticGUI", ResetOnSpawn = false, Parent = game:GetService("CoreGui")})

local mainWindow = createRoundedFrame({
	BackgroundColor3 = Color3.fromHex("#0A0A0A"),
	Size = UDim2.new(0, 900, 0, 600),
	Position = UDim2.new(0.5, -450, 0.5, -300),
	CornerRadius = UDim.new(0, 20),
	Parent = screenGui,
})

local rainbowBorder = createRainbowBorder(mainWindow, 4)
local glow = createGlow(mainWindow, 8, Color3.fromRGB(0,170,255))

-- Dragging support
do
	local dragging, dragInput, dragStart, startPos
	local function update(input)
		local delta = input.Position - dragStart
		mainWindow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
	mainWindow.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = mainWindow.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	mainWindow.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			update(input)
		end
	end)
end

-- Animations open/close fade zoom
local function openAnimation()
	mainWindow.AnchorPoint = Vector2.new(0.5,0.5)
	mainWindow.Position = UDim2.new(0.5,0,0.5,0)
	mainWindow.Size = UDim2.new(0, 0, 0, 0)
	mainWindow.BackgroundTransparency = 1
	TweenService:Create(mainWindow, TweenInfo.new(0.4, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
		Size = UDim2.new(0, 900, 0, 600),
		BackgroundTransparency = 0,
		Position = UDim2.new(0.5, -450, 0.5, -300),
	}):Play()
end
local function closeAnimation()
	local tween = TweenService:Create(mainWindow, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {
		Size = UDim2.new(0, 0, 0, 0),
		BackgroundTransparency = 1,
		Position = UDim2.new(0.5, 0, 0.5, 0),
	})
	tween:Play()
	tween.Completed:Wait()
	screenGui.Enabled = false
end

openAnimation()

-- Top bar
local topBar = createRoundedFrame({
	BackgroundColor3 = Color3.fromRGB(20,20,20),
	Size = UDim2.new(1,0,0,40),
	Position = UDim2.new(0,0,0,0),
	CornerRadius = UDim.new(0, 20),
	Parent = mainWindow,
})
local topGlow = createGlow(topBar, 6, Color3.fromRGB(0,170,255))

local titleLabel = create("TextLabel", {
	Text = "Premium Futuristic GUI",
	Font = Enum.Font.GothamBold,
	TextSize = 18,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 12, 0, 6),
	Size = UDim2.new(0.3, 0, 1, 0),
	Parent = topBar,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local versionLabel = create("TextLabel", {
	Text = "v1.0",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(150,150,150),
	BackgroundTransparency = 1,
	Position = UDim2.new(0.3, 0, 0, 6),
	Size = UDim2.new(0.1, 0, 1, 0),
	Parent = topBar,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local clockLabel = create("TextLabel", {
	Text = os.date("%H:%M:%S"),
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(150,150,150),
	BackgroundTransparency = 1,
	Position = UDim2.new(0.7, 0, 0, 6),
	Size = UDim2.new(0.15, 0, 1, 0),
	Parent = topBar,
	TextXAlignment = Enum.TextXAlignment.Right,
	TextYAlignment = Enum.TextYAlignment.Center,
})

RunService.Heartbeat:Connect(function()
	clockLabel.Text = os.date("%H:%M:%S")
end)

local function createWindowButton(text, pos)
	local btn = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(0, 30, 0, 30),
		Position = pos,
		Parent = topBar,
		CornerRadius = UDim.new(0, 8),
	})
	local label = create("TextLabel", {
		Text = text,
		Font = Enum.Font.GothamBold,
		TextSize = 18,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,1,0),
		Parent = btn,
		TextXAlignment = Enum.TextXAlignment.Center,
		TextYAlignment = Enum.TextYAlignment.Center,
	})
	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
	end)
	btn.MouseLeave:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(30,30,30)}):Play()
	end)
	return btn
end

local minimizeBtn = createWindowButton("—", UDim2.new(1, -100, 0, 5))
local maximizeBtn = createWindowButton("⬜", UDim2.new(1, -60, 0, 5))
local closeBtn = createWindowButton("✕", UDim2.new(1, -20, 0, 5))

minimizeBtn.MouseButton1Click:Connect(function()
	mainWindow.Visible = false
end)
maximizeBtn.MouseButton1Click:Connect(function()
	if mainWindow.Size == UDim2.new(1,0,1,0) then
		mainWindow:TweenSize(UDim2.new(0,900,0,600), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.3, true)
		mainWindow:TweenPosition(UDim2.new(0.5, -450, 0.5, -300), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.3, true)
	else
		mainWindow:TweenSize(UDim2.new(1,0,1,0), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.3, true)
		mainWindow:TweenPosition(UDim2.new(0,0,0,0), Enum.EasingDirection.Out, Enum.EasingStyle.Quart, 0.3, true)
	end
end)
closeBtn.MouseButton1Click:Connect(function()
	closeAnimation()
end)

-- Sidebar
local sidebar = createRoundedFrame({
	BackgroundColor3 = Color3.fromRGB(30,30,30),
	Size = UDim2.new(0, 140, 1, -40),
	Position = UDim2.new(0, 0, 0, 40),
	CornerRadius = UDim.new(0, 20),
	Parent = mainWindow,
})

local sidebarGlow = createGlow(sidebar, 6, Color3.fromRGB(0,170,255))

local categories = {
	{icon = "🏠", name = "Home"},
	{icon = "👤", name = "Player"},
	{icon = "👁️", name = "Visual"},
	{icon = "⚔️", name = "Combat"},
	{icon = "🌍", name = "World"},
	{icon = "📍", name = "Teleport"},
	{icon = "🎒", name = "Inventory"},
	{icon = "⚙️", name = "Settings"},
	{icon = "❓", name = "Misc"},
}

local selectedCategory = nil
local categoryButtons = {}

local categoryContainer = create("ScrollingFrame", {
	BackgroundTransparency = 1,
	Size = UDim2.new(1,0,1,0),
	CanvasSize = UDim2.new(0,0,0,#categories*50),
	ScrollBarThickness = 0,
	Parent = sidebar,
})

local uiListLayout = create("UIListLayout", {
	Padding = UDim.new(0, 8),
	Parent = categoryContainer,
})

local function deselectAll()
	for _, btn in pairs(categoryButtons) do
		btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
		btn.RainbowGlow.Visible = false
		btn.TextLabel.TextColor3 = Color3.fromRGB(220,220,220)
	end
end

local function createCategoryButton(icon, name)
	local btn = createRoundedFrame({
		BackgroundColor3 = Color3.fromRGB(30,30,30),
		Size = UDim2.new(1, -20, 0, 40),
		Parent = categoryContainer,
		CornerRadius = UDim.new(0, 12),
	})
	btn.Position = UDim2.new(0, 10, 0, 0)
	local iconLabel = create("TextLabel", {
		Text = icon,
		Font = Enum.Font.GothamBold,
		TextSize = 20,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 10, 0, 0),
		Size = UDim2.new(0, 30, 1, 0),
		Parent = btn,
		TextXAlignment = Enum.TextXAlignment.Center,
		TextYAlignment = Enum.TextYAlignment.Center,
	})
	local textLabel = create("TextLabel", {
		Text = name,
		Font = Enum.Font.Gotham,
		TextSize = 16,
		TextColor3 = Color3.fromRGB(220,220,220),
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 50, 0, 0),
		Size = UDim2.new(1, -50, 1, 0),
		Parent = btn,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextYAlignment = Enum.TextYAlignment.Center,
	})
	btn.TextLabel = textLabel
	local rainbowGlow = create("Frame", {
		Size = UDim2.new(1,0,1,0),
		Position = UDim2.new(0,0,0,0),
		BackgroundTransparency = 1,
		Parent = btn,
		ZIndex = btn.ZIndex - 1,
	})
	local corners = create("UICorner", {CornerRadius = UDim.new(0,12), Parent = rainbowGlow})
	btn.RainbowGlow = rainbowGlow

	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
	end)
	btn.MouseLeave:Connect(function()
		if selectedCategory ~= btn then
			TweenService:Create(btn, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(30,30,30)}):Play()
		end
	end)
	btn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			deselectAll()
			selectedCategory = btn
			btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
			btn.RainbowGlow.BackgroundTransparency = 0
			btn.TextLabel.TextColor3 = Color3.fromRGB(0, 170, 255)
			updateCategoryPanel(name)
		end
	end)
	return btn
end

for _, cat in ipairs(categories) do
	local btn = createCategoryButton(cat.icon, cat.name)
	categoryButtons[#categoryButtons+1] = btn
	btn.Parent = categoryContainer
end

-- Animate rainbow glow on selected category
RunService.Heartbeat:Connect(function()
	if selectedCategory then
		local t = tick()
		local hue = (t*2) % 1
		selectedCategory.RainbowGlow.BackgroundColor3 = Color3.fromHSV(hue,1,1)
		selectedCategory.RainbowGlow.BackgroundTransparency = 0.7
	else
		for _, btn in pairs(categoryButtons) do
			btn.RainbowGlow.BackgroundTransparency = 1
		end
	end
end)

-- Main panel
local mainPanel = createRoundedFrame({
	BackgroundColor3 = Color3.fromRGB(20,20,20),
	Size = UDim2.new(1, -160, 1, -80),
	Position = UDim2.new(0, 150, 0, 40),
	CornerRadius = UDim.new(0, 20),
	Parent = mainWindow,
})
local mainPanelBorder = createRainbowBorder(mainPanel, 3)

-- Bottom bar
local bottomBar = createRoundedFrame({
	BackgroundColor3 = Color3.fromRGB(20,20,20),
	Size = UDim2.new(1, 0, 0, 40),
	Position = UDim2.new(0, 0, 1, -40),
	CornerRadius = UDim.new(0, 20),
	Parent = mainWindow,
})
local bottomGlow = createGlow(bottomBar, 6, Color3.fromRGB(0,170,255))

local fpsLabel = create("TextLabel", {
	Text = "FPS: 0",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 10, 0, 10),
	Size = UDim2.new(0, 80, 0, 20),
	Parent = bottomBar,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local pingLabel = create("TextLabel", {
	Text = "Ping: 0ms",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 100, 0, 10),
	Size = UDim2.new(0, 80, 0, 20),
	Parent = bottomBar,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local timeLabel = create("TextLabel", {
	Text = os.date("%H:%M"),
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(0.5, -40, 0, 10),
	Size = UDim2.new(0, 80, 0, 20),
	Parent = bottomBar,
	TextXAlignment = Enum.TextXAlignment.Center,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local statusLabel = create("TextLabel", {
	Text = "Status: Online",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(1, -180, 0, 10),
	Size = UDim2.new(0, 100, 0, 20),
	Parent = bottomBar,
	TextXAlignment = Enum.TextXAlignment.Right,
	TextYAlignment = Enum.TextYAlignment.Center,
})

local playersLabel = create("TextLabel", {
	Text = "Players: 0",
	Font = Enum.Font.Gotham,
	TextSize = 14,
	TextColor3 = Color3.fromRGB(220,220,220),
	BackgroundTransparency = 1,
	Position = UDim2.new(1, -80, 0, 10),
	Size = UDim2.new(0, 70, 0, 20),
	Parent = bottomBar,
	TextXAlignment = Enum.TextXAlignment.Right,
	TextYAlignment = Enum.TextYAlignment.Center,
})

-- Update FPS, Ping, Players
local lastTick = tick()
local frameCount = 0
RunService.Heartbeat:Connect(function()
	frameCount = frameCount + 1
	local now = tick()
	if now - lastTick >= 1 then
		fpsLabel.Text = "FPS: "..frameCount
		frameCount = 0
		lastTick = now
		pingLabel.Text = "Ping: "..math.floor(LocalPlayer:GetNetworkPing()*1000).."ms"
		playersLabel.Text = "Players: "..#Players:GetPlayers()
		timeLabel.Text = os.date("%H:%M")
	end
end)

-- Category Panels and content

local function clearMainPanel()
	for _, child in pairs(mainPanel:GetChildren()) do
		if not child:IsA("UIListLayout") and not child:IsA("UIPadding") then
			child:Destroy()
		end
	end
end

local uiListLayoutMain = create("UIListLayout", {
	Padding = UDim.new(0, 10),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = mainPanel,
})
local uiPaddingMain = create("UIPadding", {
	PaddingTop = UDim.new(0, 15),
	PaddingLeft = UDim.new(0, 15),
	PaddingRight = UDim.new(0, 15),
	PaddingBottom = UDim.new(0, 15),
	Parent = mainPanel,
})

-- Player Category Content
local function createPlayerCategory()
	clearMainPanel()
	local scroll = create("ScrollingFrame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1,0,1,0),
		CanvasSize = UDim2.new(0,0,0,1000),
		ScrollBarThickness = 6,
		Parent = mainPanel,
	})
	local layout = create("UIListLayout", {Padding = UDim.new(0,10), Parent = scroll})
	local padding = create("UIPadding", {PaddingTop = UDim.new(0,10), PaddingLeft = UDim.new(0,10), PaddingRight = UDim.new(0,10),
