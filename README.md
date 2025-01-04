local expirationDate = os.time({year=2025, month=12, day=31}) -- Data de expiração: 31 de dezembro de 2024
local hasSpoken = false -- Variável para garantir que fale apenas uma vez

-- Função para verificar a validade do script
local function checkValidity()
    if hasSpoken then
        return
    end

    local currentDate = os.time()
    if currentDate <= expirationDate then
        g_game.talk("script liberado")
    else
        g_game.talk("script vencido, vacilão")
    end
    hasSpoken = true
end

-- Verifica a validade do script
checkValidity()

local function calculateDistance(pos1, pos2)
    return math.abs(pos1.x - pos2.x) + math.abs(pos1.y - pos2.y)
end

local function isValidTile(tile)
    if not tile then
        return false
    end
    local topThing = tile:getTopUseThing()
    return topThing and not tile:hasCreature()
end

local function getNeighborTiles(pos)
    local directions = {
        {x = 0, y = -1},   -- Norte
        {x = 0, y = 1},    -- Sul
        {x = -1, y = 0},   -- Oeste
        {x = 1, y = 0},    -- Leste
        {x = -1, y = -1},  -- Noroeste
        {x = 1, y = -1},   -- Nordeste
        {x = 1, y = 1},    -- Sudeste
        {x = -1, y = 1}    -- Sudoeste
    }
    local neighbors = {}
    for _, dir in ipairs(directions) do
        local neighborPos = {x = pos.x + dir.x, y = pos.y + dir.y, z = pos.z}
        local neighborTile = g_map.getTile(neighborPos)
        if neighborTile and isValidTile(neighborTile) then
            table.insert(neighbors, neighborTile)
        end
    end
    return neighbors
end

local function getPassageTiles(target)
    local targetPos = target:getPosition()
    local validTiles = getNeighborTiles(targetPos)

    if #validTiles == 2 then
        return validTiles
    end

    table.sort(validTiles, function(a, b)
        local distA = calculateDistance(a:getPosition(), targetPos)
        local distB = calculateDistance(b:getPosition(), targetPos)
        return distA < distB
    end)

    return validTiles
end

local lastPosition = nil

macro(1, function()
    -- Verifica a validade do script
    checkValidity()

    local target = g_game.getAttackingCreature()
    if not target then return end

    local currentPosition = target:getPosition()

    if lastPosition and calculateDistance(lastPosition, currentPosition) > 0 then
        local direction = target:getDirection()
        local mwPos = {x = currentPosition.x, y = currentPosition.y, z = currentPosition.z}

        if direction == 0 then
            mwPos.y = mwPos.y - 3 -- Norte
        elseif direction == 1 then
            mwPos.x = mwPos.x + 3 -- Leste
        elseif direction == 2 then
            mwPos.y = mwPos.y + 3 -- Sul
        elseif direction == 3 then
            mwPos.x = mwPos.x - 3 -- Oeste
        end

        local mwTile = g_map.getTile(mwPos)
        if mwTile then
            g_game.useInventoryItemWith(3180, mwTile:getTopUseThing())
        end
    else
        local passageTiles = getPassageTiles(target)
        for _, tile in ipairs(passageTiles) do
            local topThing = tile:getTopUseThing()
            if topThing then
                g_game.useInventoryItemWith(3180, topThing)
            end
        end
    end

    lastPosition = currentPosition
end)
