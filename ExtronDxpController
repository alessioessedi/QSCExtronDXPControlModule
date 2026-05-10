-- EXTRON DXP MATRIX MODULE --
-- Variables --

local _ipAddress = Controls["IP Address"]
local _port = 23
local _connection = TcpSocket.New()
local _password = Controls["Password"].String
local _loggedIn = false
local _updating = false
local _esc = "\27"
local _cr = "\13"

-- Functions --
local function Connection(ctl)
  if ctl.Boolean then
    _connection:Connect(_ipAddress.String, _port)
  else
    _connection:Disconnect()
  end
end

local function SendCommand(cmd)
  if _connection.IsConnected then
    _connection:Write(cmd)
  end
end

local function Send(ctl)
  if ctl.Boolean then
    if _connection.IsConnected then
      _connection:Write(Controls["TX"].String)
    else
      print("Not connected")
    end
  end
end

local function SetLoggedIn(state)
  _loggedIn = state
  Controls["LoggedIn_Fb"].Boolean = state
end

local function Route(input, output, routeType)
  local inp = tostring(math.floor(input))
  local out = tostring(math.floor(output))
  local cmd = ""
  if routeType == "AV" then
    cmd = inp .. "*" .. out .. "!" .. _cr
  elseif routeType == "V" then
    cmd = inp .. "*" .. out .. "%" .. _cr
  elseif routeType == "A" then
    cmd = inp .. "*" .. out .. "$" .. _cr
  else
    print("Route type non valido: " .. tostring(routeType))
    return
  end
  SendCommand(cmd)
end

local function HandleOutputVChange(i)
  if _updating then return end
  if Controls.AV_Break.Boolean then
    Route(Controls.Output_V[i].Value, i, "V")
  else
    Route(Controls.Output_V[i].Value, i, "AV")
  end
end

local function HandleOutputAChange(i)
  if _updating then return end
  if Controls.AV_Break.Boolean then
    Route(Controls.Output_A[i].Value, i, "A")
  end
end

local function HandleRX(data)
  if Controls["RX"] then
    Controls["RX"].String = data
  end
  print("RX: " .. data)

  _updating = true
  for output, input, sigType in string.gmatch(data, "Out(%d+) In(%d+) (%a+)") do
    local outNum = tonumber(output)
    local inNum = tonumber(input)

    if outNum and inNum and outNum >= 1 and outNum <= 16 then
      if sigType == "All" then
        Controls.Output_V[outNum].Value = inNum
        Controls.Output_A[outNum].Value = inNum
      elseif sigType == "Vid" or sigType == "RGB" then
        Controls.Output_V[outNum].Value = inNum
      elseif sigType == "Aud" then
        Controls.Output_A[outNum].Value = inNum
      end
    end
  end
  _updating = false
end

-- Connection Event Handlers --
_connection.Connected = function()
  Controls["Connect_Fb"].Boolean = true
  SetLoggedIn(false)
end

_connection.Closed = function()
  Controls["Connect_Fb"].Boolean = false
  SetLoggedIn(false)
end

_connection.Data = function()
  local data = _connection:Read(1024)
  if data == nil then return end

  if string.find(data, "Password:") then
    SendCommand(_password .. _cr)
    SetLoggedIn(false)
    return
  elseif string.find(data, "Login Administrator") then
    SendCommand(_esc .. "3CV" .. _cr)
    SetLoggedIn(true)
    return
  end

  HandleRX(data)
end

-- Event Handlers --
Controls["Connect"].EventHandler = Connection
Controls["Send"].EventHandler = Send

for i = 1, 16 do
  Controls.Output_V[i].EventHandler = function(ctl)
    HandleOutputVChange(i)
  end
  Controls.Output_A[i].EventHandler = function(ctl)
    HandleOutputAChange(i)
  end
end
