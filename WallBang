local lastKeys = {
    add = false,
    remove = false
}

local function loadWaypoints()
    local waypoints = {}

    local fileHandle = file.Open("waypoints.txt", "r")
    if not fileHandle then
        print("Failed to open waypoints.txt")
        return waypoints
    end

    local content = fileHandle:Read()
    fileHandle:Close()

    if not content then
        return waypoints
    end

    for line in content:gmatch("[^\r\n]+") do
        if line:sub(1,1) ~= "#" then
            local map, label, x, y, z = line:match("([^|]+)|([^|]+)|([^|]+)|([^|]+)|([^|]+)")

            if map and label and x and y and z then
                waypoints[#waypoints + 1] = {
                    map = map,
                    label = label,
                    pos = {
                        x = tonumber(x),
                        y = tonumber(y),
                        z = tonumber(z)
                    }
                }
            end
        end
    end

    return waypoints
end

local waypoints = loadWaypoints()

local function normalizeMapName(map)
    map = map:lower()
    map = map:gsub("maps/", "")
    map = map:gsub("%.vpk", "")
    map = map:gsub("%.bsp", "")
    return map
end

local function getDistanceToTarget(x1, y1, z1, x2, y2, z2)
    local dx = x2 - x1
    local dy = y2 - y1
    local dz = z2 - z1
    return math.sqrt(dx * dx + dy * dy + dz * dz)
end

local function drawWorldCircle(x, y, z, radius, points)
    local step = (math.pi * 2) / points
    local lastX, lastY = nil, nil

    for i = 0, points do
        local angle = i * step

        local worldPoint = Vector3(
            x + math.cos(angle) * radius,
            y + math.sin(angle) * radius,
            z
        )

        local sx, sy = client.WorldToScreen(worldPoint)

        if sx and sy then
            if lastX and lastY then
                draw.Line(lastX, lastY, sx, sy)
            end

            lastX, lastY = sx, sy
        end
    end
end

local function getClosestWaypoint(myPos, map)
    local closest = nil
    local bestDist = math.huge

    for i, wp in ipairs(waypoints) do
        if wp.map == map then
            local dx = wp.pos.x - myPos.x
            local dy = wp.pos.y - myPos.y
            local dz = wp.pos.z - myPos.z

            local dist = dx*dx + dy*dy + dz*dz

            if dist < bestDist then
                bestDist = dist
                closest = i
            end
        end
    end

    return closest
end

local function saveWaypoints()
    local fileHandle = file.Open("waypoints.txt", "w")
    if not fileHandle then return end

    for i, wp in ipairs(waypoints) do
        local line = string.format(
            "%s|%s|%f|%f|%f\n",
            wp.map,
            wp.label,
            wp.pos.x,
            wp.pos.y,
            wp.pos.z
        )

        fileHandle:Write(line)
    end

    fileHandle:Close()
end

local function handleWaypointKeys()

    local me = entities.GetLocalPlayer()
    if not me or not me:IsAlive() then return end

    local map = normalizeMapName(engine.GetMapName())
    local pos = me:GetAbsOrigin()

    local addKey = input.IsButtonDown(0x21)   -- PAGE UP
    local remKey = input.IsButtonDown(0x22)   -- PAGE DOWN

    -- PAGE UP (only on press, not hold)
    if addKey and not lastKeys.add then

        table.insert(waypoints, {
            map = map,
            label = "Waypoint " .. tostring(#waypoints + 1),
            pos = { x = pos.x, y = pos.y, z = pos.z }
        })

        saveWaypoints()
        print("Waypoint added")
    end

    -- PAGE DOWN (only on press, not hold)
    if remKey and not lastKeys.remove then

        local idx = getClosestWaypoint(pos, map)

        if idx then
            table.remove(waypoints, idx)
            saveWaypoints()
            print("Waypoint removed")
        end
    end

    -- update state
    lastKeys.add = addKey
    lastKeys.remove = remKey
end

function drawEventHandler()
    local me = entities.GetLocalPlayer()

    if me == nil or not me:IsAlive() then
        return
    end

    local current_map = normalizeMapName(engine.GetMapName())
    local myPos = me:GetAbsOrigin()

    for i, spot in ipairs(waypoints) do

        if spot.map == current_map then

            local distance = getDistanceToTarget(
                myPos.x,
                myPos.y,
                myPos.z,
                spot.pos.x,
                spot.pos.y,
                spot.pos.z
            )

            if distance <= 200 then

                local sx, sy = client.WorldToScreen(
                    Vector3(
                        spot.pos.x,
                        spot.pos.y,
                        spot.pos.z + 10
                    )
                )

                if sx ~= nil and sy ~= nil then
                    draw.Color(0, 255, 0, 255)
                    drawWorldCircle(spot.pos.x, spot.pos.y, spot.pos.z, 10, 10)
                    draw.Text(sx, sy, spot.label)
                end
            end
        end
    end
end

callbacks.Register("Draw", drawEventHandler)
callbacks.Register("Draw", handleWaypointKeys)
