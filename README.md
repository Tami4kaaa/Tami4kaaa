--- Drawing Player Radar 
--- Made by topit 
 
_G.RadarSettings = { 
    --- Radar settings 
    RADAR_LINES = true; -- Displays distance rings + cardinal lines  
    RADAR_LINE_DISTANCE = 75; -- The distance between each distance ring 
    RADAR_SCALE = 1; -- Controls how "zoomed in" the radar display is  
    RADAR_RADIUS = 60; -- The size of the radar itself 
    RADAR_ROTATION = false; -- Toggles radar rotation. Looks kinda trippy when disabled 
    SMOOTH_ROT = true; -- Toggles smooth radar rotation 
    SMOOTH_ROT_AMNT = 20; -- Lower number is smoother, higher number is snappier  
    CARDINAL_DISPLAY = true; -- Displays the four cardinal directions (north east south west) around the radar 
     
    --- Marker settings 
    DISPLAY_OFFSCREEN = true; -- Displays offscreen / off-radar markers 
    DISPLAY_TEAMMATES = true; -- Displays markers that belong to your teammates 
    DISPLAY_TEAM_COLORS = true; -- Displays your teammates markers with either a custom color (change Team_Marker) or with that teams TeamColor (enable USE_TEAM_COLORS)  
    DISPLAY_FRIEND_COLORS = true; -- Displays your friends markers with a custom color (Friend_Marker). This takes priority over DISPLAY_TEAM_COLORS and DISPLAY_RGB 
    DISPLAY_RGB_COLORS = false; -- Displays each marker with an RGB cycle. Takes priority over DISPLAY_TEAM_COLORS, but not DISPLAY_FRIEND_COLORS 
    MARKER_SCALE_BASE = 1.40; -- Base scale that gets applied to markers 
    MARKER_SCALE_MAX = 1.25; -- The largest scale that a marker can be 
    MARKER_SCALE_MIN = 0.15; -- The smallest scale that a marker can be 
    MARKER_FALLOFF = true; -- Affects the markers' scale depending on how far away the player is - bypasses SCALE_MIN and SCALE_MAX 
    MARKER_FALLOFF_AMNT = 50; -- How close someone has to be for falloff to start affecting them  
    OFFSCREEN_TRANSPARENCY = 0.35; -- Transparency of offscreen markers 
    USE_FALLBACK = false; -- Enables an emergency "fallback mode" for StreamingEnabled games 
    USE_QUADS = true; -- Displays radar markers as arrows instead of dots  
    USE_TEAM_COLORS = false; -- Uses a team's TeamColor for marker colors 
    VISIBLITY_CHECK = false; -- Makes markers that are not visible slightly transparent  
     
    --- Theme 
    RADAR_THEME = { 
        Outline = Color3.fromRGB(25, 25, 35); -- Radar outline 
        Background = Color3.fromRGB(100, 100, 200); -- Radar background 
        DragHandle = Color3.fromRGB(0, 0, 35); -- Drag handle  
         
        Cardinal_Lines = Color3.fromRGB(10, 10, 12); -- Color of the horizontal and vertical lines 
        Distance_Lines = Color3.fromRGB(65, 65, 75); -- Color of the distance rings 
         
        Generic_Marker = Color3.fromRGB(255, 25, 115); -- Color of a player marker without a team 
        Local_Marker = Color3.fromRGB(10, 70, 150); -- Color of your marker, regardless of team 
        Team_Marker = Color3.fromRGB(255,90, 50); -- Color of your teammates markers. Used when DISPLAY_TEAM_COLORS is disabled 
        Friend_Marker = Color3.fromRGB(0, 255, 255); -- Color of your friends markers. Used when DISPLAY_FRIEND_COLORS is enabled  
    }; 
} 
 
--- Drawing Player Radar
--- Made by topit
--- Nov 28 2022
local scriptver = 'v1.3.2'

-- v1.3.2 changelog
--* Dragging the radar now moves the hovertext properly
--* Fixed USE_FALLBACK erroring if a player has no character
--+ Added Trident Survival to the unsupported games list
--+ Added a few "support warnings" for specific games that mess with the radar
--+ Added more error handling in case a game tries some fucky shit
--+ Auto-exec might possibly could be a slight bit more stable
--+ Changed a tiny bit of player manager stuff, it might be rewritten for better plugin interfacing
--+ Radar now has a loading message incase you aren't spawned in on execution

-- v1.3.1 changelog
--* May have fixed some friend stuff
--* Rewrote a bunch of setting descriptions
--* Rewrote a bunch of team color logic, now the Team_Marker entry works
--+ Added "USE_TEAM_COLORS" setting, if this is enabled then players' marker colors are set to their team

-- v1.3.0 changelog
--! Changed how scaling works a bit, you may need to re-config your old values!
--* Fixed how players that are dead when the radar is executed show up as alive
--* Fixed issue where players without teams could break the team color setting
--* Fixed issues with players not being loaded properly 
--* May have fixed some possible issues with the dot display
--* Renamed "MARKER_SCALEMIN" and "MARKER_SCALEMAX" to "MARKER_SCALE_MIN" and "MARKER_SCALE_MAX"
--+ Added the "Friend_Marker" theme entry 
--+ Added the "Team_Marker" theme entry 
--+ Added the "DISPLAY_FRIEND_COLORS" setting - self explanatory
--+ Added the "DISPLAY_RGB_COLORS" setting - self explanatory
--+ Added the "USE_FALLBACK" setting, which lets the radar work on streamingenabled games
--+ Added the "VISIBLITY_CHECK" setting, which makes non-visible (i.e. players behind walls) markers become dimmed
--+ Adding / removing a friend now changes their marker color in real time  
--+ Markers now have antialiasing (originally was gonna add black outlines / shadows, but they looked awful)
--+ The hover display is now way more smooth 

-- v1.2.1 changelog 
--+ Markers now get properly updated when someone switches teams 

-- v1.2 changelog 
--+ Cleaned up a bunch of stuff
--+ Added option to disable rotation 
--+ Added marker falloff setting, kinda sucks but it could be useful to someone
--+ Remade hover display 
--+ Re-executing the script will now exit the active radar
--+ Removed a bunch of extra manager tables, now only one is used (improves mem usage?)
--+ Added a setting that lets you change how smooth / snappy smooth rotation is
--+ Radar lines now have a cool pattern that makes them not suck
--* Fixed several errors having to do with player respawns
--* Hopefully fixed several "render object destroyed" errors

-- v1.1 changelog
--+ Added username display when you hover over a player marker 
--+ Players that leave now have their markers fade out instead of just disappearing
--* Fixed a few possible memleaks
--* Fixed markers not unfilling when someone dies in specific situations

if ( _G.RadarKill ) then
    _G.RadarKill()
end

if ( not game:IsLoaded() ) then
    game.Loaded:Wait()
end

--- Settings ---
local existingSettings = _G.RadarSettings or {}
local settings = {
    --- Radar settings
    -- lines
    RADAR_LINES = true; -- Displays distance rings + cardinal lines 
    RADAR_LINE_DISTANCE = 50; -- The distance between each distance ring
    -- scale
    RADAR_SCALE = 1; -- Controls how "zoomed in" the radar display is 
    RADAR_RADIUS = 125; -- The size of the radar itself
    -- rotation
    RADAR_ROTATION = true; -- Toggles radar rotation. Looks kinda trippy when disabled
    SMOOTH_ROT = true; -- Toggles smooth radar rotation
    SMOOTH_ROT_AMNT = 30; -- Lower number is smoother, higher number is snappier 
    -- misc
    CARDINAL_DISPLAY = true; -- Displays the four cardinal directions (north east south west) around the radar
    
    --- Marker settings
    -- display 
    DISPLAY_OFFSCREEN = true; -- Displays offscreen / off-radar markers
    DISPLAY_TEAMMATES = true; -- Displays markers that belong to your teammates
    DISPLAY_TEAM_COLORS = true; -- Displays your teammates markers with either a custom color (change Team_Marker) or with that teams TeamColor (enable USE_TEAM_COLORS) 
    DISPLAY_FRIEND_COLORS = true; -- Displays your friends markers with a custom color (Friend_Marker). This takes priority over DISPLAY_TEAM_COLORS and DISPLAY_RGB
    DISPLAY_RGB_COLORS = false; -- Displays each marker with an RGB cycle. Takes priority over DISPLAY_TEAM_COLORS, but not DISPLAY_FRIEND_COLORS
    -- scale 
    MARKER_SCALE_BASE = 1.25; -- Base scale that gets applied to markers
    MARKER_SCALE_MAX = 1.25; -- The biggest size that a marker can be
    MARKER_SCALE_MIN = 0.75; -- The smallest size that a marker can be
    -- falloff 
    MARKER_FALLOFF = true; -- Affects the markers' scale depending on how far away the player is - bypasses SCALE_MIN and SCALE_MAX
    MARKER_FALLOFF_AMNT = 125; -- How close someone has to be for falloff to start affecting them 
    -- misc 
    OFFSCREEN_TRANSPARENCY = 0.3; -- Transparency of offscreen markers
    USE_FALLBACK = false; -- Enables an emergency "fallback mode" for StreamingEnabled games
    USE_QUADS = true; -- Displays radar markers as arrows instead of dots 
    USE_TEAM_COLORS = false; -- Uses a team's TeamColor for marker colors
    VISIBLITY_CHECK = false; -- Makes markers that are not visible slightly transparent 
    
    --- Theme
    RADAR_THEME = {
        -- radar
        Outline = Color3.fromRGB(35, 35, 45); -- Radar outline
        Background = Color3.fromRGB(25, 25, 35); -- Radar background
        DragHandle = Color3.fromRGB(50, 50, 255); -- Drag handle 
        
        -- lines
        Cardinal_Lines = Color3.fromRGB(110, 110, 120); -- Color of the horizontal and vertical lines
        Distance_Lines = Color3.fromRGB(65, 65, 75); -- Color of the distance rings
        
        -- markers
        Generic_Marker = Color3.fromRGB(255, 25, 115); -- Color of a player marker without a team
        Local_Marker = Color3.fromRGB(115, 25, 255); -- Color of your marker, regardless of team
        Team_Marker = Color3.fromRGB(25, 115, 255); -- Color of your teammates markers. Used when USE_TEAM_COLORS is disabled
        Friend_Marker = Color3.fromRGB(25, 255, 115); -- Color of your friends markers. Used when DISPLAY_FRIEND_COLORS is enabled 
    };
}

-- fill in missing settings 
for k, v in pairs(existingSettings) do 
    if ( v ~= nil ) then
        settings[k] = v 
    end
end

--- Radar settings 
-- lines 
local RADAR_LINES = settings.RADAR_LINES
local RADAR_LINE_DISTANCE = settings.RADAR_LINE_DISTANCE
-- scale
local RADAR_SCALE = settings.RADAR_SCALE
local RADAR_RADIUS = settings.RADAR_RADIUS
-- rotation
local RADAR_ROTATION = settings.RADAR_ROTATION
local SMOOTH_ROT = settings.SMOOTH_ROT
local SMOOTH_ROT_AMNT = settings.SMOOTH_ROT_AMNT
-- misc
local CARDINAL_DISPLAY = settings.CARDINAL_DISPLAY

--- Marker settings
-- display
local DISPLAY_OFFSCREEN = settings.DISPLAY_OFFSCREEN
local DISPLAY_TEAMMATES = settings.DISPLAY_TEAMMATES
local DISPLAY_TEAM_COLORS = settings.DISPLAY_TEAM_COLORS
local DISPLAY_FRIEND_COLORS = settings.DISPLAY_FRIEND_COLORS
local DISPLAY_RGB_COLORS = settings.DISPLAY_RGB_COLORS
-- scale 
local MARKER_SCALE_BASE = settings.MARKER_SCALE_BASE
local MARKER_SCALE_MAX = settings.MARKER_SCALE_MAX
local MARKER_SCALE_MIN = settings.MARKER_SCALE_MIN
-- falloff 
local MARKER_FALLOFF = settings.MARKER_FALLOFF
local MARKER_FALLOFF_AMNT = settings.MARKER_FALLOFF_AMNT
-- misc 
local OFFSCREEN_TRANSPARENCY = settings.OFFSCREEN_TRANSPARENCY
local USE_FALLBACK = settings.USE_FALLBACK
local USE_QUADS = settings.USE_QUADS
local USE_TEAM_COLORS = settings.USE_TEAM_COLORS
local VISIBLITY_CHECK = settings.VISIBLITY_CHECK

if ( DISPLAY_RGB_COLORS and DISPLAY_TEAM_COLORS ) then
    DISPLAY_TEAM_COLORS = false 
end

--- Theme 
local RADAR_THEME = settings.RADAR_THEME 

--- Services ---
local inputService = game:GetService('UserInputService')
local playerService = game:GetService('Players')
local runService = game:GetService('RunService')
local starterGui = game:GetService('StarterGui')

--- Localization ---
local newV2 = Vector2.new
local newV3 = Vector3.new

local mathSin = math.sin
local mathCos = math.cos
local mathExp = math.exp

--- Important tables ---
local scriptCns = {}
local radarObjects = {}

--- Other variables
local markerScale = math.clamp(RADAR_SCALE, MARKER_SCALE_MIN, MARKER_SCALE_MAX) * MARKER_SCALE_BASE
local scaleVec = newV2(markerScale, markerScale)

local quadPointA = newV2(0, 5)   * scaleVec
local quadPointB = newV2(4, -5)  * scaleVec
local quadPointC = newV2(0, -3)  * scaleVec
local quadPointD = newV2(-4, -5) * scaleVec

--- Drawing setup ---
local drawObjects = {}
local function newDrawObj(objectClass, objectProperties) -- this method is cringe but it's easy to work with 
    local obj = Drawing.new(objectClass)
    table.insert(drawObjects, obj)
    
    for i, v in pairs(objectProperties) do
        obj[i] = v
    end

    objectProperties = nil
    return obj
end

-- Drawing tween function 
local tweenExp, tweenQuad do -- obj property dest time 
    local function numLerp(a, b, c) -- skidded from wikipedia (😱😱)
        return (1 - c) * a + c * b
    end
    
    local tweenTypes = {}
    tweenTypes.Vector2 = Vector2.zero.Lerp
    tweenTypes.number = numLerp
    tweenTypes.Color3 = Color3.new().Lerp
    
    -- https://easings.net is useful for easing funcs
    
    function tweenExp(obj, property, dest, duration) 
        task.spawn(function()
            local initialVal = obj[property]
            local tweenTime = 0
            local lerpFunc = tweenTypes[typeof(dest)]
            
            while ( tweenTime < duration ) do 
                
                obj[property] = lerpFunc(initialVal, dest, 1 - math.pow(2, -10 * tweenTime / duration))
                
                local deltaTime = task.wait()
                tweenTime += deltaTime
            end

            obj[property] = dest
        end)
    end
    
    function tweenQuad(obj, property, dest, duration, func) 
        task.spawn(function()
            local initialVal = obj[property]
            local tweenTime = 0
            local lerpFunc = tweenTypes[typeof(dest)]
            
            while ( tweenTime < duration ) do 
                obj[property] = lerpFunc(initialVal, dest, 1 - (1 - tweenTime / duration) * (1 - tweenTime / duration))
                if ( func ) then
                    func(obj[property]) 
                end
                
                local deltaTime = task.wait()
                tweenTime += deltaTime
            end

            obj[property] = dest
        end)
    end
end

--- Local object manager --- 
local errMessage = 'Failed to get the %s instance. Your game may be unsupported, or simply has not finished loading.'

local clientPlayer = playerService.LocalPlayer

if ( not clientPlayer ) then
    for _, con in pairs(scriptCns) do 
        con:Disconnect() 
    end
    
    return messagebox(string.format(errMessage, 'LocalPlayer'), 'Player Radar', 0)
end

local clientRoot do 
    scriptCns.charRespawn = clientPlayer.CharacterAdded:Connect(function(newChar) 
        clientRoot = newChar:WaitForChild('HumanoidRootPart')
        
        if ( clientRoot ) then
            radarObjects.loadText.Visible = false 
            radarObjects.loadOverlay.Visible = false  
        else
            radarObjects.loadText.Visible = true 
            radarObjects.loadOverlay.Visible = true  
        end
    end)
    
    if ( clientPlayer.Character ) then 
        clientRoot = clientPlayer.Character:FindFirstChild('HumanoidRootPart')
    end
end

local clientCamera do 
    scriptCns.cameraUpdate = workspace:GetPropertyChangedSignal('CurrentCamera'):Connect(function() 
        clientCamera = workspace.CurrentCamera or workspace:FindFirstChildOfClass('Camera')
    end)

    clientCamera = workspace.CurrentCamera or workspace:FindFirstChildOfClass('Camera')
end

if ( not clientCamera ) then -- tested this out, deleting the camera instantly makes a new one but who cares
    for _, con in pairs(scriptCns) do 
        con:Disconnect() 
    end
    
    return messagebox(string.format(errMessage, 'Camera'), 'Player Radar', 0)
end

local clientTeam do 
    scriptCns.teamUpdate = clientPlayer:GetPropertyChangedSignal('Team'):Connect(function() 
        clientTeam = clientPlayer.Team
    end)

    clientTeam = clientPlayer.Team
end

--- PlaceID Check --- 
do
    local thisId = game.PlaceId
    local retardedGames = {
        292439477;   -- Phantom forces - support might be added
        3233893879;  -- Bad business
        8130299583;  -- Trident survival (server browser?) - support might be added
        9570110925;  -- Trident survival (server) - support might be added
    }
    local gameNotes = {
        [379614936] = 'This game is known to fuck up the radar - waiting a round should fix'; -- Assassins
        [2474168535] = 'Players that are lassoed don\'t appear on the radar properly'; -- Westbound 
    }
    
    local halfWidth = clientCamera.ViewportSize.X / 2
    
    local notif = Drawing.new('Text')
    notif.Center = true
    notif.Color = Color3.fromRGB(255, 255, 255)
    notif.Font = Drawing.Fonts.UI
    notif.Outline = true
    notif.Position = newV2(halfWidth, 200)
    notif.Size = 22
    notif.Transparency = 0 
    notif.Visible = true 
    notif.ZIndex = 500 
    
    if ( table.find(retardedGames, thisId) ) then
        notif.Text = 'Games with custom character systems\naren\'t supported. Sorry!'
        
        tweenExp(notif, 'Transparency', 1, 0.25)
        tweenExp(notif, 'Position', newV2(halfWidth, 150), 0.25)
        task.wait(5)
        
        tweenExp(notif, 'Position', newV2(halfWidth, 200), 0.25)
        tweenExp(notif, 'Transparency', 0, 0.25)
        task.wait(0.5)
        
        for _, con in pairs(scriptCns) do 
            con:Disconnect()
        end
        
        notif:Remove()
        return
    else
        -- might as well place loaded notification here 
        notif.Text = ('Loaded Drawing Radar %s\n\nControls:\n[-]: zoom out     [+]: zoom in     [End]: exit script'):format(scriptver) -- [Home]: toggle radar 
        
        local gameWarning = gameNotes[thisId]
        
        if ( gameWarning ) then 
            notif.Text = notif.Text .. string.format('\n\nGame warning: %s', gameWarning) -- fuck fluxus for not having ..= support 
        end
        
        task.spawn(function()
            tweenExp(notif, 'Transparency', 1, 0.25)
            tweenExp(notif, 'Position', newV2(halfWidth, 150), 0.25)
            task.wait(gameWarning and 10 or 5)
            
            tweenExp(notif, 'Position', newV2(halfWidth, 200), 0.25)
            tweenExp(notif, 'Transparency', 0, 0.25)
            task.wait(0.5)
            
            if ( workspace.StreamingEnabled ) then
                notif.Text = 'It looks like this game uses StreamingEnabled - Fallback mode is now enabled.'
                tweenExp(notif, 'Transparency', 1, 0.25)
                tweenExp(notif, 'Position', newV2(halfWidth, 150), 0.25)
                task.wait(5)
                
                tweenExp(notif, 'Position', newV2(halfWidth, 200), 0.25)
                tweenExp(notif, 'Transparency', 0, 0.25)
                task.wait(1)
            end
            
            notif:Remove()
        end)
    end
end

--- Player managers --- 
if ( workspace.StreamingEnabled ) then
    USE_FALLBACK = true
end

local playerManagers = {}
if ( game.PlaceId == 292439477 ) then -- Phantom forces support (being developed soon 🤑)
    local function removePlayer(player) 
    end
    
    local function readyPlayer(thisPlayer) 
    
    end
    
    -- Setup managers for every existing player 
    for _, player in ipairs(playerService:GetPlayers()) do
        if ( player ~= clientPlayer ) then
            readyPlayer(player)
        end
    end

    -- Setup managers for joining players, and clean managers for leaving players
    scriptCns.pm_playerAdd = playerService.PlayerAdded:Connect(readyPlayer)
    scriptCns.pm_playerRemove = playerService.PlayerRemoving:Connect(removePlayer)
else
    
    local function removePlayer(player) 
        local thisName = player.Name
        local thisManager = playerManagers[thisName]

        -- had an error randomly happen where there was no manager made for someone before they were removed
        -- definitely not getting a 0.000001% chance of some retard joining then leaving a microsecond later
        -- so now there's this check for some random race condition that happens every 8 billion years 
        if ( not thisManager ) then
            return
        end
        local thisPlayerCns = thisManager.Cns
                
        if ( thisManager.onLeave ) then 
            thisManager.onLeave()
        end
        
        for _, con in pairs(thisPlayerCns) do
            con:Disconnect()
        end
        
        thisManager.onDeath = nil
        thisManager.onLeave = nil
        thisManager.onRemoval = nil
        thisManager.onRespawn = nil
        thisManager.onTeamChange = nil 
        
        thisManager.Player = nil
        
        playerManagers[thisName] = nil 
    end
    
    local function readyPlayer(thisPlayer) 
        local thisName = thisPlayer.Name

        local thisManager = {}
        local thisPlayerCns = {}
        
        local function deathFunc() -- Reusable on-death function - done so the same function doesnt get made 9138589135 times 
            if ( thisManager.onDeath ) then
                thisManager.onDeath()
            end
        end

        -- Setup connections
        thisPlayerCns['chr-add'] = thisPlayer.CharacterAdded:Connect(function(newChar) -- This handles when a player respawns
            if ( USE_FALLBACK ) then
                thisManager.Character = newChar 
                return  
            end
            
            -- Get the new instances 
            local RootPart = newChar:WaitForChild('HumanoidRootPart')
            local Humanoid = newChar:WaitForChild('Humanoid')
            
            -- Call onRespawn
            if ( thisManager.onRespawn ) then
                thisManager.onRespawn()
            end
            -- Update manager values 
            thisManager.Character = newChar
            thisManager.RootPart = RootPart
            thisManager.Humanoid = Humanoid
            
            -- Re-connect the death connection 
            if ( thisPlayerCns['chr-die'] ) then
                thisPlayerCns['chr-die']:Disconnect()
            end
            thisPlayerCns['chr-die'] = Humanoid.Died:Connect(deathFunc)
        end)

        thisPlayerCns['chr-remove'] = thisPlayer.CharacterRemoving:Connect(function() -- This handles when a player's character gets removed 
            if ( USE_FALLBACK ) then
                thisManager.Character = nil 
                return  
            end
            
            -- Call onRemoval
            if ( thisManager.onRemoval ) then
                thisManager.onRemoval()
            end
            
            -- Update manager values 
            thisManager.Character = nil
            thisManager.RootPart = nil
            thisManager.Humanoid = nil 
        end)
        
        thisPlayerCns['team'] = thisPlayer:GetPropertyChangedSignal('Team'):Connect(function()  -- This handles team changing, self explanatory
            thisManager.Team = thisPlayer.Team
            
            if ( thisManager.onTeamChange ) then
                thisManager.onTeamChange(thisManager.Team)
            end
        end)
        
        -- Check for an existing character
        if ( thisPlayer.Character ) then
            -- Fetch some stuff
            local Character = thisPlayer.Character
            local Humanoid = Character:FindFirstChild('Humanoid')
            local RootPart = Character:FindFirstChild('HumanoidRootPart')

            -- Set manager values 
            thisManager.Character = Character
            thisManager.RootPart = RootPart
            thisManager.Humanoid = Humanoid 

            if ( Humanoid ) then
                -- Setup death connection *only if the humanoid exists*
                -- This previously wasn't checked for which probably constantly errored, oops 🗿
                thisPlayerCns['chr-die'] = Humanoid.Died:Connect(deathFunc)
            end
        end
        
        -- Set existing values 
        thisManager.Team = thisPlayer.Team
        thisManager.Player = thisPlayer
        thisManager.Name = thisName 
        thisManager.DisplayName = thisPlayer.DisplayName  
        thisManager.Friended = clientPlayer:IsFriendsWith(thisPlayer.UserId)
        thisManager.GetCFrame = function() 
            local thisRoot = thisManager.RootPart
            local cframe  
            
            if ( thisRoot ) then
                cframe = thisRoot.CFrame
            elseif ( USE_FALLBACK and thisManager.Character ) then 
                cframe = thisManager.Character:GetPivot()
            end
            
            return cframe 
        end
        
        -- Finalize
        thisManager.Cns = thisPlayerCns 
        playerManagers[thisName] = thisManager
    end
    
    -- Setup managers for every existing player 
    for _, player in ipairs(playerService:GetPlayers()) do
        if ( player ~= clientPlayer ) then
            readyPlayer(player)
        end
    end

    -- Setup managers for joining players, and clean managers for leaving players
    scriptCns.pm_playerAdd = playerService.PlayerAdded:Connect(readyPlayer)
    scriptCns.pm_playerRemove = playerService.PlayerRemoving:Connect(removePlayer)
end

--- Radar UI --- 
local radarLines = {}
local radarPosition = newV2(100, 130)

-- main radar
radarObjects.main = newDrawObj('Circle', {
    Color = RADAR_THEME.Background;
    Position = radarPosition; 
    
    Filled = true;
    Visible = true;
    
    NumSides = 40;
    Radius = RADAR_RADIUS;
    ZIndex = 300;
})

radarObjects.outline = newDrawObj('Circle', {
    Color = RADAR_THEME.Outline;
    Position = radarPosition; 
    
    Filled = false;
    Visible = true;
    
    NumSides = 40;
    Thickness = 10;
    Radius = RADAR_RADIUS;
    ZIndex = 299;
})

radarObjects.dragHandle = newDrawObj('Circle', {
    Color = RADAR_THEME.DragHandle;
    Position = radarPosition; 
    
    Filled = false;
    Visible = false;
    
    NumSides = 40;
    Radius = RADAR_RADIUS;
    Thickness = 3;
    ZIndex = 325;
})

-- spawn overlay
radarObjects.loadOverlay = newDrawObj('Circle', {
    Color = Color3.new(0, 0, 0);
    Filled = true;
    NumSides = 40;
    Position = radarPosition; 
    Radius = RADAR_RADIUS;
    Transparency = 0.5;
    Visible = clientRoot == nil;
    ZIndex = 319;
})

radarObjects.loadText = newDrawObj('Text', {
    Center = true;
    Color = Color3.fromRGB(255, 255, 255);
    Font = Drawing.Fonts.UI;
    Outline = true;
    Position = radarPosition - newV2(0, 15);
    Size = 20;
    Text = 'Waiting for you to spawn in...';
    Transparency = 1;
    Visible = clientRoot == nil;
    ZIndex = 320;
})

-- text
radarObjects.zoomText = newDrawObj('Text', {
    Center = true;
    Color = Color3.fromRGB(255, 255, 255);
    Font = Drawing.Fonts.UI;
    Outline = true;
    Size = 20;
    Transparency = 0;
    Visible = true;
    ZIndex = 306;
})

radarObjects.hoverText = newDrawObj('Text', {
    Center = true;
    Color = Color3.fromRGB(255, 255, 255);
    Font = Drawing.Fonts.UI;
    Outline = true;
    Position = radarPosition;
    Size = 16;
    Transparency = 1;
    Visible = false;
    ZIndex = 306;
})

-- center marker
if ( USE_QUADS ) then 
    radarObjects.localMark = newDrawObj('Quad', {
        Color = RADAR_THEME.Local_Marker;
        Filled = true;
        Thickness = 2;
        Visible = true;
        ZIndex = 305;
    })
    
    radarObjects.localMarkStroke = newDrawObj('Quad', {
        Color = RADAR_THEME.Local_Marker;
        Filled = false;
        Thickness = 2;
        Visible = true;
        ZIndex = 304;
    })
else
    radarObjects.localMark = newDrawObj('Circle', {
        Color = RADAR_THEME.Local_Marker;
        Filled = true;
        NumSides = 20;
        Thickness = 2;
        Visible = true;
        ZIndex = 305;
    })
    
    radarObjects.localMarkStroke = newDrawObj('Circle', {
        Color = RADAR_THEME.Local_Marker;
        Filled = false;
        NumSides = 20;
        Thickness = 1;
        Visible = true;
        ZIndex = 304;
    })
end

-- lines
if ( RADAR_LINES ) then 
    for i = 0, RADAR_RADIUS, RADAR_SCALE * RADAR_LINE_DISTANCE do 
        local thisLine = newDrawObj('Circle', {
            Color = RADAR_THEME.Distance_Lines;
            Position = radarPosition; 
            Radius = i;
            
            Filled = false;
            Visible = true;
            
            Transparency = ((i / RADAR_LINE_DISTANCE) % 4 == 0) and 0.8 or 0.2;
            NumSides = 40;
            Thickness = 1;
            ZIndex = 300;
        })
        
        table.insert(radarLines, thisLine)
    end
    
    radarObjects.horizontalLine = newDrawObj('Line', {
        Color = RADAR_THEME.Cardinal_Lines;
        From = radarPosition - newV2(RADAR_RADIUS, 0);
        To = radarPosition + newV2(RADAR_RADIUS, 0);
                
        Visible = true; 
        
        Thickness = 1;
        Transparency = 0.2;
        ZIndex = 300;
    })
    
    radarObjects.verticalLine = newDrawObj('Line', {
        Color = RADAR_THEME.Cardinal_Lines;
        From = radarPosition - newV2(0, RADAR_RADIUS);
        To = radarPosition + newV2(0, RADAR_RADIUS);
        
        Visible = true; 
        
        Thickness = 1;
        Transparency = 0.2;
        ZIndex = 300;
    })
else
    radarLines = nil 
end

-- NSWE display 
if ( CARDINAL_DISPLAY ) then
    radarObjects.directionN = newDrawObj('Text', {
        Center = true;
        Color = Color3.fromRGB(255, 255, 255);
        Font = Drawing.Fonts.UI;
        Outline = true;
        Size = 16;
        Text = 'N';
        Transparency = 1;
        Visible = true;
        ZIndex = 303;
    })

    radarObjects.directionS = newDrawObj('Text', {
        Center = true;
        Color = Color3.fromRGB(255, 255, 255);
        Font = Drawing.Fonts.UI;
        Outline = true;
        Size = 16;
        Text = 'S';
        Transparency = 1;
        Visible = true;
        ZIndex = 303;
    })

    radarObjects.directionW = newDrawObj('Text', {
        Center = true;
        Color = Color3.fromRGB(255, 255, 255);
        Font = Drawing.Fonts.UI;
        Outline = true;
        Size = 16;
        Text = 'W';
        Transparency = 1;
        Visible = true;
        ZIndex = 303;
    })

    radarObjects.directionE = newDrawObj('Text', {
        Center = true;
        Color = Color3.fromRGB(255, 255, 255);
        Font = Drawing.Fonts.UI;
        Outline = true;
        Size = 16;
        Text = 'E';
        Transparency = 1;
        Visible = true;
        ZIndex = 303;
    })
end

--- Other functions ---
local destroying = false 
local function killScript() -- destroys the script; self explanatory
    if ( destroying ) then
        return
    end 
    
    destroying = true
    
    for _, con in pairs(scriptCns) do 
        con:Disconnect()
    end
    
    task.wait()
    for name, manager in pairs(playerManagers) do 
        for _, con in pairs(manager.Cns) do
            con:Disconnect()
        end
        
        -- just in case 
        manager.onDeath = nil 
        manager.onLeave = nil 
        manager.onRespawn = nil 
        manager.onRemoval = nil 
        manager.onTeamChange = nil 

        playerManagers[name] = nil 
    end
    
    for _, obj in ipairs(drawObjects) do 
        tweenExp(obj, 'Transparency', 0, 0.5)
    end
    
    task.wait(1)
    
    if ( not drawObjects ) then
        return
    end
    
    for _, obj in ipairs(drawObjects) do 
        obj:Remove()
    end
    _G.RadarKill = nil 
    drawObjects = nil
end

local function setRadarScale() -- updates the radar's scale using RADAR_SCALE 
    markerScale = math.clamp(RADAR_SCALE, MARKER_SCALE_MIN, MARKER_SCALE_MAX) * MARKER_SCALE_BASE
    
    if ( RADAR_LINES ) then
        -- Calculate how many radar lines can fit at this scale 
        local lineCount = math.floor(RADAR_RADIUS / (RADAR_SCALE * RADAR_LINE_DISTANCE))
        
        -- If more lines can fit than there are made, make more 
        if ( lineCount > #radarLines ) then
            for i = 1, lineCount - #radarLines do 
                
                local thisLine = newDrawObj('Circle', {
                    Color = RADAR_THEME.Distance_Lines;
                    
                    Position = radarPosition;
                    
                    Filled = false;
                    Visible = true;
                    
                    NumSides = 40;
                    Thickness = 1;
                    ZIndex = 300;
                })
                
                table.insert(radarLines, thisLine)
            end
        end
        
        -- Position every single line 
        for idx, line in ipairs(radarLines) do 
            if ( idx > lineCount ) then
                -- This line wont fit, hide it 
                line.Visible = false 
            else
                -- This line fits, set its radius and display it 
                line.Radius = idx * (RADAR_SCALE * RADAR_LINE_DISTANCE)
                line.Transparency = (idx % 4 == 0) and 0.8 or 0.2
                line.Visible = true
            end
        end
    end
    
    
    if ( USE_QUADS ) then 
        scaleVec = newV2(markerScale, markerScale)
        
        quadPointA = newV2(0, 5)   * scaleVec
        quadPointB = newV2(4, -5)  * scaleVec
        quadPointC = newV2(0, -3)  * scaleVec
        quadPointD = newV2(-4, -5) * scaleVec
    else
        radarObjects.localMark.Radius = markerScale * 3
        radarObjects.localMarkStroke.Radius = markerScale * 3 
    end
end

local function setRadarPosition(newPosition) -- sets the radar's position to newPosition
    radarPosition = newPosition
        
    radarObjects.main.Position = newPosition
    radarObjects.outline.Position = newPosition
    
    
    radarObjects.loadOverlay.Position = newPosition
    radarObjects.loadText.Position = newPosition - newV2(0, 15)
    
    if ( RADAR_LINES ) then
        for _, line in ipairs(radarLines) do 
            line.Position = newPosition
        end 
        
        radarObjects.horizontalLine.From = newPosition - newV2(RADAR_RADIUS, 0);
        radarObjects.horizontalLine.To = newPosition + newV2(RADAR_RADIUS, 0);
        
        radarObjects.verticalLine.From = newPosition - newV2(0, RADAR_RADIUS);
        radarObjects.verticalLine.To = newPosition + newV2(0, RADAR_RADIUS);
    end
    
    radarObjects.hoverText.Position = newPosition
end

--- Input and drag handling ---
do
    local radarDragging = false
    local radarHovering = false
    
    local zoomingIn = false
    local zoomingOut = false
        
    -- The keycode is only checked if its found in this dictionary,
    -- just so a giant elif chain isnt done on every keypress
    local keysToCheck = {
        End = true;
        Equals = true;
        Minus = true;
    }
    
    scriptCns.inputBegan = inputService.InputBegan:Connect(function(io) 
        local inputType = io.UserInputType.Name

        if ( inputType == 'Keyboard' ) then
            local keyCode = io.KeyCode.Name
            
            if ( not keysToCheck[keyCode] ) then
                return
            end
            
            if ( keyCode == 'End' ) then
                killScript() 
            elseif ( keyCode == 'Equals' ) then
                zoomingIn = true 
                
                local zoomText = radarObjects.zoomText
                zoomText.Position = radarPosition + newV2(0, RADAR_RADIUS + 25)
                tweenExp(zoomText, 'Transparency', 1, 0.3)
                
                local accel = 0.75
                
                scriptCns.zoomInCn = runService.Heartbeat:Connect(function(deltaTime) 
                    RADAR_SCALE = math.clamp(RADAR_SCALE + (deltaTime * accel), 0.02, 3)
                  
