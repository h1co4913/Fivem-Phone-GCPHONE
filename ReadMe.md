# Crew Phone

# After installing all the phone files into your server do this BEFORE starting the server:

In your es_extended or extendedmode client folder, open the main.lua

Find your spawn thread and insert this (usually right below 'esx:restoreLoadout'):

TriggerServerEvent('crew:onPlayerLoaded', GetPlayerServerId(PlayerId()))

# You can now start your server and the phone will register your number. Enjoy!

# To get your screenshots working:

Open your gcphone client/client.lua and find line 713-718, you should see the following code:

exports['screenshot-basic']:requestScreenshotUpload(data.url, data.field, function(data)
  local resp = json.decode(data)
  DestroyMobilePhone()
  CellCamActivate(false, false)
  cb(json.encode({ url = resp.files[1].url }))   
end)

# You need to change that to this below. Don't forget to insert your discord webhook!

exports['screenshot-basic']:requestScreenshotUpload("https://discordapp.com/api/webhooks/yourdiscordchannelwebhookurl", data.field, function(data)
  local image = json.decode(data)
  DestroyMobilePhone()
  CellCamActivate(false, false)
  cb(json.encode({ url = image.attachments[1].proxy_url }))
end)

# To get the phone to work with mumble voice (default is Toko-Voip):

Open your gcphone client/client.lua and find this code:

RegisterNetEvent("gcPhone:acceptCall")
AddEventHandler("gcPhone:acceptCall", function(infoCall, initiator)
  if inCall == false and USE_RTC == false then
    inCall = true
    exports['tokovoip_script']:setPlayerData(GetPlayerName(PlayerId()), "call:channel", infoCall.id + 1120.00, true)
	  exports.tokovoip_script:addPlayerToRadio(infoCall.id + 1120.00)
	  TokoVoipID = infoCall.id + 1120.00
  end
  if menuIsOpen == false then
    TooglePhone()
  end
  PhonePlayCall()
  SendNUIMessage({event = 'acceptCall', infoCall = infoCall, initiator = initiator})
end)

RegisterNetEvent("gcPhone:rejectCall")
AddEventHandler("gcPhone:rejectCall", function(infoCall)
  if inCall == true then
    inCall = false
    Citizen.InvokeNative(0xE036A705F989E049)
    exports['tokovoip_script']:setPlayerData(GetPlayerName(PlayerId()), "call:channel", 'nil', true)
  	exports.tokovoip_script:removePlayerFromRadio(TokoVoipID)
  	TokoVoipID = nil
  end
  PhonePlayText()
  SendNUIMessage({event = 'rejectCall', infoCall = infoCall})
end)

# Replace that code with this below

RegisterNetEvent("gcPhone:acceptCall")
AddEventHandler("gcPhone:acceptCall", function(infoCall, initiator)
  if inCall == false and USE_RTC == false then
    inCall = true
    exports["mumble-voip"]:SetCallChannel(infoCall.id + 1)
  end
  if menuIsOpen == false then
    TooglePhone()
  end
  PhonePlayCall()
  SendNUIMessage({event = 'acceptCall', infoCall = infoCall, initiator = initiator})
end)

RegisterNetEvent("gcPhone:rejectCall")
AddEventHandler("gcPhone:rejectCall", function(infoCall)
  if inCall == true then
    inCall = false
    exports["mumble-voip"]:SetCallChannel(0)
  end
  PhonePlayText()
  SendNUIMessage({event = 'rejectCall', infoCall = infoCall})
end)

# Also find this code in the same client.lua

RegisterNUICallback('notififyUseRTC', function (use, cb)
  USE_RTC = use
  if USE_RTC == true and inCall == true then
    inCall = false
    Citizen.InvokeNative(0xE036A705F989E049)
    exports['tokovoip_script']:setPlayerData(GetPlayerName(PlayerId()), "call:channel", 'nil', true)
  	exports.tokovoip_script:removePlayerFromRadio(TokoVoipID)
	  TokoVoipID = nil
  end
  cb()
end)

# Replace that code with this below 

RegisterNUICallback('notififyUseRTC', function (use, cb)
  USE_RTC = use
  if USE_RTC == true and inCall == true then
    inCall = false
    exports["mumble-voip"]:SetCallChannel(infoCall.id+1)
  end
  cb()
end)

# If you use esx_voice then just change your fxmanifest.lua to use the client_esx_voice.lua instead of the regular client.lua