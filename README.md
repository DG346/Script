-- Futuristic Premium GUI Panel in Lua
-- Requires LOVE2D framework for rendering and animation

local lg = love.graphics
local timer = love.timer

-- Configuration
local config = {
    window = {
        width = 900,
        height = 600,
        bgColor = {10/255, 10/255, 10/255, 1}, -- #0A0A0A black matte
        borderRadius = 12,
        borderGlowRadius = 6,
        borderThickness = 3,
        sidebarWidth = 180,
        topbarHeight = 48,
        bottombarHeight = 28,
        animSpeed = 0.6,
        fontName = "Roboto-Regular.ttf",
        fontBoldName = "Roboto-Bold.ttf",
        fontSize = 14,
        fontSizeTitle = 20,
        fontSizeSmall = 12,
        iconSize = 20,
        glowAlpha = 0.3,
        glowSpread = 12,
        glowBlurSteps = 6,
        glowColorAlpha = 0.15,
        glowPulseSpeed = 2,
        hoverAnimSpeed = 0.25,
        fadeInDuration = 0.5,
        zoomDuration = 0.5,
        minimizeBarHeight = 40,
        minimizeBarWidth = 220,
        minimizeAnimDuration = 0.4,
        cornerRadius = 10,
    },
    colors = {
        grayDark = {30/255, 30/255, 30/255, 1},
        grayMedium = {50/255, 50/255, 50/255, 1},
        grayLight = {70/255, 70/255, 70/255, 1},
        white = {1, 1, 1, 1},
        transparent = {0, 0, 0, 0},
    },
    categories = {
        {name = "Home", icon = "home"},
        {name = "Player", icon = "user"},
        {name = "Visual", icon = "eye"},
        {name = "Combat", icon = "sword"},
        {name = "World", icon = "globe"},
        {name = "Teleport", icon = "location"},
        {name = "Inventory", icon = "box"},
        {name = "Settings", icon = "gear"},
        {name = "Misc", icon = "dots"},
    },
    version = "v1.0.0",
    menuName = "FUTURE PANEL",
    userCount = 128,
}

-- Load fonts
local fonts = {}

-- State
local state = {
    dragging = false,
    dragOffsetX = 0,
    dragOffsetY = 0,
    windowX = 0,
    windowY = 0,
    windowWidth = config.window.width,
    windowHeight = config.window.height,
    minimized = false,
    minimizingAnim = 0,
    fadeIn = 0,
    zoom = 0,
    selectedCategory = 1,
    hoverCategory = nil,
    hoverButton = nil,
    time = 0,
    fps = 0,
    ping = 42,
    status = "Online",
    clock = "",
}

-- Helpers

local function lerp(a, b, t)
    return a + (b - a) * t
end

local function clamp(x, minv, maxv)
    return math.max(minv, math.min(maxv, x))
end

local function round(num, idp)
    local mult = 10^(idp or 0)
    return math.floor(num * mult + 0.5) / mult
end

-- HSV to RGB conversion for rainbow effect
local function hsvToRgb(h, s, v)
    local c = v * s
    local x = c * (1 - math.abs((h * 6) % 2 - 1))
    local m = v - c
    local r, g, b = 0, 0, 0
    local hi = math.floor(h * 6)
    if hi == 0 then r, g, b = c, x, 0
    elseif hi == 1 then r, g, b = x, c, 0
    elseif hi == 2 then r, g, b = 0, c, x
    elseif hi == 3 then r, g, b = 0, x, c
    elseif hi == 4 then r, g, b = x, 0, c
    elseif hi == 5 then r, g, b = c, 0, x
    end
    return r + m, g + m, b + m
end

-- Rounded rectangle with glow border
local function drawRoundedRectGlow(x, y, w, h, radius, borderThickness, time, glowAlpha)
    -- Draw glow layers
    local glowColorAlpha = glowAlpha or config.window.glowColorAlpha
    local glowSpread = config.window.glowSpread
    local glowBlurSteps = config.window.glowBlurSteps
    local baseHue = (time * 0.1) % 1
    for i = glowBlurSteps, 1, -1 do
        local alpha = glowColorAlpha * (i / glowBlurSteps)
        local spread = glowSpread * (i / glowBlurSteps)
        local r, g, b = hsvToRgb((baseHue + i * 0.05) % 1, 1, 1)
        lg.setColor(r, g, b, alpha)
        lg.setLineWidth(borderThickness + spread)
        lg.setLineJoin("round")
        lg.rectangle("line", x, y, w, h, radius, radius)
    end
    -- Draw main border with rainbow
    local r, g, b = hsvToRgb(baseHue, 1, 1)
    lg.setColor(r, g, b, 1)
    lg.setLineWidth(borderThickness)
    lg.setLineJoin("round")
    lg.rectangle("line", x, y, w, h, radius, radius)
end

-- Draw rounded rectangle filled with color and optional border
local function drawRoundedRect(x, y, w, h, radius, fillColor, borderColor, borderWidth)
    if fillColor then
        lg.setColor(fillColor)
        lg.rectangle("fill", x, y, w, h, radius, radius)
    end
    if borderColor and borderWidth and borderWidth > 0 then
        lg.setColor(borderColor)
        lg.setLineWidth(borderWidth)
        lg.setLineJoin("round")
        lg.rectangle("line", x, y, w, h, radius, radius)
    end
end

-- Draw text with shadow for better visibility
local function drawTextShadow(text, font, x, y, color, shadowColor, ox, oy, align)
    lg.setFont(font)
    lg.setColor(shadowColor)
    lg.print(text, x + (ox or 1), y + (oy or 1), 0, 1, 1, 0, 0, align)
    lg.setColor(color)
    lg.print(text, x, y, 0, 1, 1, 0, 0, align)
end

-- Draw icon placeholder (simple vector shapes)
local function drawIcon(name, x, y, size, color)
    lg.setColor(color)
    local s = size
    local cx, cy = x + s/2, y + s/2
    if name == "home" then
        -- House shape
        lg.setLineWidth(2)
        lg.line(cx - s*0.4, cy + s*0.2, cx, cy - s*0.3, cx + s*0.4, cy + s*0.2)
        lg.rectangle("line", cx - s*0.3, cy + s*0.2, s*0.6, s*0.4, 3, 3)
    elseif name == "user" then
        -- Circle head + body
        lg.circle("line", cx, cy - s*0.1, s*0.2, 16)
        lg.line(cx, cy + s*0.1, cx, cy + s*0.4)
        lg.line(cx - s*0.25, cy + s*0.4, cx + s*0.25, cy + s*0.4)
    elseif name == "eye" then
        -- Eye shape
        lg.setLineWidth(2)
        lg.ellipse("line", cx, cy, s*0.4, s*0.2)
        lg.circle("line", cx, cy, s*0.1)
    elseif name == "sword" then
        -- Simple sword shape
        lg.setLineWidth(2)
        lg.line(cx, cy - s*0.4, cx, cy + s*0.3)
        lg.line(cx - s*0.15, cy + s*0.1, cx + s*0.15, cy + s*0.1)
        lg.polygon("line", cx - s*0.1, cy - s*0.4, cx + s*0.1, cy - s*0.4, cx, cy - s*0.6)
    elseif name == "globe" then
        -- Circle with lines
        lg.setLineWidth(2)
        lg.circle("line", cx, cy, s*0.35)
        lg.line(cx - s*0.35, cy, cx + s*0.35, cy)
        lg.line(cx, cy - s*0.35, cx, cy + s*0.35)
        lg.arc("line", "open", cx, cy, s*0.35, math.rad(45), math.rad(135))
        lg.arc("line", "open", cx, cy, s*0.35, math.rad(225), math.rad(315))
    elseif name == "location" then
        -- Pin shape
        lg.setLineWidth(2)
        lg.circle("line", cx, cy - s*0.1, s*0.15)
        lg.line(cx, cy + s*0.3, cx, cy - s*0.1)
        lg.polygon("line", cx - s*0.1, cy + s*0.3, cx + s*0.1, cy + s*0.3, cx, cy + s*0.5)
    elseif name == "box" then
        -- Cube shape
        lg.setLineWidth(2)
        lg.rectangle("line", cx - s*0.3, cy - s*0.2, s*0.6, s*0.4)
        lg.line(cx - s*0.3, cy - s*0.2, cx - s*0.1, cy - s*0.4)
        lg.line(cx + s*0.3, cy - s*0.2, cx + s*0.1, cy - s*0.4)
        lg.line(cx - s*0.1, cy - s*0.4, cx + s*0.1, cy - s*0.4)
    elseif name == "gear" then
        -- Gear shape simplified
        lg.setLineWidth(2)
        lg.circle("line", cx, cy, s*0.25)
        for i = 0, 7 do
            local angle = i * math.pi/4
            local x1 = cx + math.cos(angle) * s*0.35
            local y1 = cy + math.sin(angle) * s*0.35
            local x2 = cx + math.cos(angle) * s*0.45
            local y2 = cy + math.sin(angle) * s*0.45
            lg.line(x1, y1, x2, y2)
        end
        lg.circle("line", cx, cy, s*0.1)
    elseif name == "dots" then
        -- Three vertical dots
        lg.circle("fill", cx, cy - s*0.25, s*0.07)
        lg.circle("fill", cx, cy, s*0.07)
        lg.circle("fill", cx, cy + s*0.25, s*0.07)
    else
        -- Default: small square
        lg.rectangle("fill", cx - s*0.15, cy - s*0.15, s*0.3, s*0.3)
    end
end

-- Draw button with hover and glow effect
local function drawButton(x, y, w, h, radius, text, font, isHover, isActive, time, iconName)
    local baseColor = config.colors.grayDark
    local textColor = config.colors.white
    local borderColor = {0, 0, 0, 0}
    local glowAlpha = 0
    if isActive then
        borderColor = {1, 1, 1, 0.8}
        glowAlpha = 0.4
    elseif isHover then
        borderColor = {1, 1, 1, 0.5}
        glowAlpha = 0.25
    else
        borderColor = {0, 0, 0, 0}
        glowAlpha = 0
    end

    -- Background fill
    drawRoundedRect(x, y, w, h, radius, baseColor)

    -- Animated RGB border glow
    if glowAlpha > 0 then
        drawRoundedRectGlow(x, y, w, h, radius, 2, time, glowAlpha)
    end

    -- Border line
    if borderColor[4] > 0 then
        lg.setColor(borderColor)
        lg.setLineWidth(1.5)
        lg.rectangle("line", x, y, w, h, radius, radius)
    end

    -- Icon + Text
    local iconSize = config.window.iconSize
    local padding = 12
    if iconName then
        drawIcon(iconName, x + padding, y + (h - iconSize)/2, iconSize, textColor)
        lg.setColor(textColor)
        lg.setFont(font)
        lg.print(text, x + padding + iconSize + 10, y + (h - font:getHeight())/2)
    else
        lg.setColor(textColor)
        lg.setFont(font)
        lg.printf(text, x, y + (h - font:getHeight())/2, w, "center")
    end
end

-- Draw top bar with menu name, version, profile icon, clock, and window buttons
local function drawTopBar(x, y, w, h, time, state)
    local radius = config.window.cornerRadius
    local bgColor = config.colors.grayDark
    local borderColor = {0, 0, 0, 0}
    drawRoundedRect(x, y, w, h, radius, bgColor, borderColor, 0)

    -- Left: Menu name and version
    local padding = 16
    lg.setFont(fonts.title)
    drawTextShadow(config.menuName, fonts.title, x + padding, y + 6, config.colors.white, {0,0,0,0.6})
    lg.setFont(fonts.small)
    drawTextShadow(config.version, fonts.small, x + padding, y + 28, {0.7,0.7,0.7,1}, {0,0,0,0.5})

    -- Right: Profile icon, clock, buttons
    local btnSize = 28
    local spacing = 12
    local rightX = x + w - padding

    -- Clock
    local clockText = state.clock or "--:--:--"
    lg.setFont(fonts.small)
    local clockW = fonts.small:getWidth(clockText)
    drawTextShadow(clockText, fonts.small, rightX - btnSize*3 - spacing*3 - clockW, y + (h - fonts.small:getHeight())/2, config.colors.white, {0,0,0,0.5})

    -- Profile icon (circle with user icon)
    local profileX = rightX - btnSize*3 - spacing*2
    local profileY = y + h/2
    lg.setColor(0.15, 0.15, 0.15, 1)
    lg.circle("fill", profileX + btnSize/2, profileY, btnSize/2 - 4)
    drawIcon("user", profileX + 6, y + 6, btnSize - 12, config.colors.white)

    -- Buttons: minimize, maximize, close
    local btnY = y + (h - btnSize)/2
    local btnX = rightX - btnSize*2 - spacing
    local btns = {"minimize", "maximize", "close"}
    local btnColors = {
        minimize = {0.6, 0.6, 0.6, 1},
        maximize = {0.6, 0.6, 0.6, 1},
        close = {1, 0.3, 0.3, 1},
    }
    local hover = state.hoverButton
    for i, btn in ipairs(btns) do
        local bx = rightX - btnSize * (3 - i + 1) - spacing * (3 - i + 1)
        local isHover = (hover == btn)
        local baseCol = btnColors[btn]
        local col = {baseCol[1], baseCol[2], baseCol[3], baseCol[4]}
        if isHover then
            col = {1, 1, 1, 1}
        end
        lg.setColor(col)
        lg.rectangle("fill", bx, btnY, btnSize, btnSize, 6, 6)
        lg.setColor(0, 0, 0, 0.7)
        lg.setLineWidth(2)
        if btn == "minimize" then
            lg.line(bx + 6, btnY + btnSize/2, bx + btnSize - 6, btnY + btnSize/2)
        elseif btn == "maximize" then
            lg.rectangle("line", bx + 6, btnY + 6, btnSize - 12, btnSize - 12, 3, 3)
        elseif btn == "close" then
            lg.line(bx + 6, btnY + 6, bx + btnSize - 6, btnY + btnSize - 6)
            lg.line(bx + btnSize - 6, btnY + 6, bx + 6, btnY + btnSize - 6)
        end
    end
end

-- Draw bottom bar with FPS, Ping, Time, Status, User count (visual only)
local function drawBottomBar(x, y, w, h, state)
    local radius = config.window.cornerRadius
    local bgColor = config.colors.grayDark
    drawRoundedRect(x, y, w, h, radius, bgColor)

    local padding = 16
    local spacing = 24
    local textColor = config.colors.white
    lg.setFont(fonts.small)

    local items = {
        ("FPS: %d"):format(state.fps),
        ("Ping: %d ms"):format(state.ping),
        ("Time: %s"):format(state.clock or "--:--:--"),
        ("Status: %s"):format(state.status),
        ("Users: %d"):format(config.userCount),
    }

    local xPos = x + padding
    for i, item in ipairs(items) do
        drawTextShadow(item, fonts.small, xPos, y + (h - fonts.small:getHeight())/2, textColor, {0,0,0,0.5})
        xPos = xPos + fonts.small:getWidth(item) + spacing
    end
end

-- Draw sidebar with categories
local function drawSidebar(x, y, w, h, time, state)
    local radius = config.window.cornerRadius
    local bgColor = config.colors.grayDark
    drawRoundedRect(x, y, w, h, radius, bgColor)

    local catHeight = 44
    local padding = 12
    local baseColor = config.colors.grayDark
    local textColor = config.colors.white
    local borderThickness = 2
    local borderRadius = 8

    for i, cat in ipairs(config.categories) do
        local cy = y + padding + (i - 1) * (catHeight + 6)
        local isSelected = (state.selectedCategory == i)
        local isHover = (state.hoverCategory == i)

        -- Background fill
        local fillColor = baseColor
        if isSelected then
            fillColor = {0.12, 0.12, 0.12, 1}
        elseif isHover then
            local alpha = lerp(0, 0.3, math.sin(time * 8) * 0.5 + 0.5)
            fillColor = {0.15, 0.15, 0.15, alpha}
        end
        drawRoundedRect(x + 6, cy, w - 12, catHeight, borderRadius, fillColor)

        -- Animated RGB border
        if isSelected or isHover then
            drawRoundedRectGlow(x + 6, cy, w - 12, catHeight, borderRadius, borderThickness, time, 0.3)
        else
            -- subtle border
            lg.setColor(0.1, 0.1, 0.1, 0.6)
            lg.setLineWidth(1)
            lg.rectangle("line", x + 6, cy, w - 12, catHeight, borderRadius, borderRadius)
        end

        -- Icon and text
        local iconX = x + 6 + 12
        local iconY = cy + (catHeight - config.window.iconSize)/2
        drawIcon(cat.icon, iconX, iconY, config.window.iconSize, textColor)

        lg.setFont(fonts.regular)
        lg.setColor(textColor)
        lg.print(cat.name, iconX + config.window.iconSize + 12, cy + (catHeight - fonts.regular:getHeight())/2)
    end
end

-- Draw main content area (empty panel)
local function drawMainArea(x, y, w, h, time)
    local radius = config.window.cornerRadius
    local bgColor = config.colors.grayMedium
    drawRoundedRect(x, y, w, h, radius, bgColor)

    -- Animated RGB border glow
    drawRoundedRectGlow(x, y, w, h, radius, 3, time, 0.25)
end

-- Draw minimized bar with smooth animation
local function drawMinimizedBar(x, y, w, h, time, state)
    local radius = config.window.cornerRadius
    local bgColor = config.colors.grayDark
    drawRoundedRect(x, y, w, h, radius, bgColor)

    -- Animated RGB border glow
    drawRoundedRectGlow(x, y, w, h, radius, 3, time, 0.3)

    -- Text centered
    lg.setFont(fonts.title)
    lg.setColor(config.colors.white)
    local text = config.menuName .. " - " .. config.version
    lg.printf(text, x, y + (h - fonts.title:getHeight())/2, w, "center")
end

-- Update clock string
local function updateClock(state)
    local timeTable = os.date("*t")
    state.clock = string.format("%02d:%02d:%02d", timeTable.hour, timeTable.min, timeTable.sec)
end

-- Initialize fonts
local function loadFonts()
    fonts.regular = lg.newFont(config.window.fontName, config.window.fontSize)
    fonts.bold = lg.newFont(config.window.fontBoldName, config.window.fontSize)
    fonts.title = lg.newFont(config.window.fontBoldName, config.window.fontSizeTitle)
    fonts.small = lg.newFont(config.window.fontName, config.window.fontSizeSmall)
end

-- Check if point inside rectangle
local function pointInRect(px, py, rx, ry, rw, rh)
    return px >= rx and px <= rx + rw and py >= ry and py <= ry + rh
end

-- Window buttons hit test
local function hitTestWindowButtons(mx, my, winX, winY, winW, topH)
    local btnSize = 28
    local spacing = 12
    local rightX = winX + winW - 16
    local btnY = winY + (topH - btnSize)/2
    local buttons = {
        minimize = {x = rightX - btnSize*3 - spacing*3, y = btnY, w = btnSize, h = btnSize},
        maximize = {x = rightX - btnSize*2 - spacing*2, y = btnY, w = btnSize, h = btnSize},
        close = {x = rightX - btnSize - spacing, y = btnY, w = btnSize, h = btnSize},
    }
    for name, rect in pairs(buttons) do
        if pointInRect(mx, my, rect.x, rect.y, rect.w, rect.h) then
            return name
        end
    end
    return nil
end

-- Check if point inside sidebar category
local function hitTestSidebar(mx, my, sx, sy, sw)
    local catHeight = 44
    local padding = 12
    for i, cat in ipairs(config.categories) do
        local cy = sy + padding + (i - 1) * (catHeight + 6)
        if pointInRect(mx, my, sx + 6, cy, sw - 12, catHeight) then
            return i
        end
    end
    return nil
end

-- Main LOVE2D callbacks

function love.load()
    love.window.setMode(1280, 720, {resizable=true, minwidth=800, minheight=600})
    loadFonts()
    local w, h = love.graphics.getDimensions()
    state.windowX = (w - config.window.width) / 2
    state.windowY = (h - config.window.height) / 2
    state.windowWidth = config.window.width
    state.windowHeight = config.window.height
    state.fadeIn = 0
    state.zoom = 0
    state.minimized = false
    state.minimizingAnim = 0
    updateClock(state)
    state.time = 0
end

function love.update(dt)
    state.time = state.time + dt
    state.fps = love.timer.getFPS()
    updateClock(state)

    -- Fade in and zoom animation on open
    if state.fadeIn < 1 then
        state.fadeIn = math.min(1, state.fadeIn + dt / config.window.fadeInDuration)
    end
    if state.zoom < 1 then
        state.zoom = math.min(1, state.zoom + dt / config.window.zoomDuration)
    end

    -- Minimize animation
    if state.minimized then
        state.minimizingAnim = math.min(1, state.minimizingAnim + dt / config.window.minimizeAnimDuration)
    else
        state.minimizingAnim = math.max(0, state.minimizingAnim - dt / config.window.minimizeAnimDuration)
    end

    -- Dragging window
    if state.dragging then
        local mx, my = love.mouse.getPosition()
        state.windowX = mx - state.dragOffsetX
        state.windowY = my - state.dragOffsetY
        -- Clamp inside screen
        local sw, sh = love.graphics.getDimensions()
        state.windowX = clamp(state.windowX, 0, sw - state.windowWidth)
        state.windowY = clamp(state.windowY, 0, sh - state.windowHeight)
    end
end

function love.draw()
    local alpha = state.fadeIn
    local zoom = state.zoom
    local anim = state.minimizingAnim

    -- Background dim
    lg.clear(config.colors.grayDark)

    -- Calculate window position and size with minimize animation
    local wx, wy, ww, wh
    if anim > 0 then
        -- Interpolate to minimized bar
        local barW = config.window.minimizeBarWidth
        local barH = config.window.minimizeBarHeight
        local centerX = love.graphics.getWidth() / 2
        local centerY = love.graphics.getHeight() / 2
        local targetX = centerX - barW / 2
        local targetY = love.graphics.getHeight() - barH - 20

        wx = lerp(state.windowX, targetX, anim)
        wy = lerp(state.windowY, targetY, anim)
        ww = lerp(state.windowWidth, barW, anim)
        wh = lerp(state.windowHeight, barH, anim)
    else
        wx = state.windowX
        wy = state.windowY
        ww = state.windowWidth
        wh = state.windowHeight
    end

    -- Apply zoom and fade
    lg.push()
    lg.translate(wx + ww/2, wy + wh/2)
    lg.scale(zoom, zoom)
    lg.translate(-(wx + ww/2), -(wy + wh/2))
    lg.setColor(1, 1, 1, alpha)

    if anim < 1 then
        -- Draw full window

        -- Main window background with glow border
        drawRoundedRectGlow(wx, wy, ww, wh, config.window.borderRadius, config.window.borderThickness, state.time, config.window.glowAlpha)
        drawRoundedRect(wx, wy, ww, wh, config.window.borderRadius, config.window.bgColor)

        -- Top bar
        drawTopBar(wx, wy, ww, config.window.topbarHeight, state.time, state)

        -- Bottom bar
        drawBottomBar(wx, wy + wh - config.window.bottombarHeight, ww, config.window.bottombarHeight, state)

        -- Sidebar
        drawSidebar(wx, wy + config.window.topbarHeight, config.window.sidebarWidth, wh - config.window.topbarHeight - config.window.bottombarHeight, state.time, state)

        -- Main content area
        local mainX = wx + config.window.sidebarWidth + 12
        local mainY = wy + config.window.topbarHeight + 12
        local mainW = ww - config.window.sidebarWidth - 24
        local mainH = wh - config.window.topbarHeight - config.window.bottombarHeight - 24
        drawMainArea(mainX, mainY, mainW, mainH, state.time)

    else
        -- Draw minimized bar
        drawMinimizedBar(wx, wy, ww, wh, state.time, state)
    end

    lg.pop()
end

function love.mousepressed(mx, my, button)
    if button ~= 1 then return end
    local wx, wy, ww, wh = state.windowX, state.windowY, state.windowWidth, state.windowHeight
    local topH = config.window.topbarHeight

    -- If minimized bar active, check click to restore
    if state.minimized then
        local barW = config.window.minimizeBarWidth
        local barH = config.window.minimizeBarHeight
        local centerX = love.graphics.getWidth() / 2
        local targetX = centerX - barW / 2
        local targetY = love.graphics.getHeight() - barH - 20
        if pointInRect(mx, my, targetX, targetY, barW, barH) then
            state.minimized = false
            return
        end
        return
    end

    -- Check window buttons
    local btn = hitTestWindowButtons(mx, my, wx, wy, ww, topH)
    if btn then
        if btn == "close" then
            love.event.quit()
        elseif btn == "minimize" then
            state.minimized = true
        elseif btn == "maximize" then
            local sw, sh = love.graphics.getDimensions()
            if state.windowWidth == sw and state.windowHeight == sh then
                -- Restore to default size
                state.windowWidth = config.window.width
                state.windowHeight = config.window.height
                state.windowX = (sw - state.windowWidth) / 2
                state.windowY = (sh - state.windowHeight) / 2
            else
                -- Maximize
                state.windowX = 0
                state.windowY = 0
                state.windowWidth = sw
                state.windowHeight = sh
            end
        end
        return
    end

    -- Check top bar drag area (exclude buttons area)
    local btnAreaX = wx + ww - 28*3 - 12*4
    if pointInRect(mx, my, wx, wy, ww, topH) and mx < btnAreaX then
        state.dragging = true
        state.dragOffsetX = mx - wx
        state.dragOffsetY = my - wy
        return
    end

    -- Check sidebar categories
    local sx, sy, sw = wx, wy + topH, config.window.sidebarWidth
    local catIndex = hitTestSidebar(mx, my, sx, sy, sw)
    if catIndex then
        state.selectedCategory = catIndex
        return
    end
end

function love.mousereleased(mx, my, button)
    if button ~= 1 then return end
    state.dragging = false
end

function love.mousemoved(mx, my, dx, dy)
    -- Hover window buttons
    local wx, wy, ww, wh = state.windowX, state.windowY, state.windowWidth, state.windowHeight
    local topH = config.window.topbarHeight
    local hoverBtn = hitTestWindowButtons(mx, my, wx, wy, ww, topH)
    state.hoverButton = hoverBtn

    -- Hover sidebar categories
    local sx, sy, sw = wx, wy + topH, config.window.sidebarWidth
    local hoverCat = hitTestSidebar(mx, my, sx, sy, sw)
    state.hoverCategory = hoverCat
end

function love.resize(w, h)
    -- Keep window centered on resize if not maximized
    if state.windowWidth < w and state.windowHeight < h then
        state.windowX = (w - state.windowWidth) / 2
        state.windowY = (h - state.windowHeight) / 2
    end
end
