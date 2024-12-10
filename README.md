# Customized ESX 1.11.4 Identity to VMS resources
| Compatible Resources  |
| ------------- |
| ✅ vms_cityhall| 

#### esx_identity/server/main.lua
###### Lines 15-33 (`function SetPlayerData`)
```diff
function SetPlayerData(xPlayer, data)
    local name = ("%s %s"):format(data.firstName, data.lastName)
    xPlayer.setName(name)
    xPlayer.set("firstName", data.firstName)
    xPlayer.set("lastName", data.lastName)
    xPlayer.set("dateofbirth", data.dateOfBirth)
    xPlayer.set("sex", data.sex)
    xPlayer.set("height", data.height)
+   xPlayer.set("ssn", data.ssn)

    local state = Player(xPlayer.source).state
    state:set("name", name, true)
    state:set("firstName", data.firstName, true)
    state:set("lastName", data.lastName, true)
    state:set("dateofbirth", data.dateOfBirth, true)
    state:set("sex", data.sex, true)
    state:set("height", data.height, true)
+   state:set("ssn", data.ssn, true)
end
```

###### Lines 45-47 (`function saveIdentityToDatabase`)
```diff
local function saveIdentityToDatabase(identifier, identity)
+   MySQL.update.await("UPDATE users SET firstname = ?, lastname = ?, dateofbirth = ?, sex = ?, height = ?, ssn = ? WHERE identifier = ?", { identity.firstName, identity.lastName, identity.dateOfBirth, identity.sex, identity.height, identity.ssn, identifier })
end
```

###### Lines 146-169 (`function checkIdentity`)
```diff
local function checkIdentity(xPlayer)
+   MySQL.single("SELECT firstname, lastname, dateofbirth, sex, height, ssn FROM users WHERE identifier = ?", { xPlayer.identifier }, function(result)
        if not result then
            return TriggerClientEvent("esx_identity:showRegisterIdentity", xPlayer.source)
        end
        if not result.firstname then
            playerIdentity[xPlayer.identifier] = nil
            alreadyRegistered[xPlayer.identifier] = false
            return TriggerClientEvent("esx_identity:showRegisterIdentity", xPlayer.source)
        end
        playerIdentity[xPlayer.identifier] = {
            firstName = result.firstname,
            lastName = result.lastname,
            dateOfBirth = result.dateofbirth,
            sex = result.sex,
            height = result.height,
+           ssn = result.ssn,
        }
        alreadyRegistered[xPlayer.identifier] = true
        setIdentity(xPlayer)
    end)
end
```

###### Lines 172-205
```diff
AddEventHandler("playerConnecting", function(_, _, deferrals)
    deferrals.defer()
    local _, identifier = source, ESX.GetIdentifier(source)
    Wait(40)
    if not identifier then
        return deferrals.done(TranslateCap("no_identifier"))
    end
+   MySQL.single("SELECT firstname, lastname, dateofbirth, sex, height, ssn FROM users WHERE identifier = ?", { identifier }, function(result)
        if not result then
            playerIdentity[identifier] = nil
            alreadyRegistered[identifier] = false
            return deferrals.done()
        end
        if not result.firstname then
            playerIdentity[identifier] = nil
            alreadyRegistered[identifier] = false
            return deferrals.done()
        end
        playerIdentity[identifier] = {
            firstName = result.firstname,
            lastName = result.lastname,
            dateOfBirth = result.dateofbirth,
            sex = result.sex,
            height = result.height,
+           ssn = result.ssn,
        }
        alreadyRegistered[identifier] = true
        deferrals.done()
    end)
end)
```

###### Lines 272-289
```diff
if xPlayer then
    if alreadyRegistered[xPlayer.identifier] then
        xPlayer.showNotification(TranslateCap("already_registered"), "error")
        return cb(false)
    end
    playerIdentity[xPlayer.identifier] = {
        firstName = formatName(data.firstname),
        lastName = formatName(data.lastname),
        dateOfBirth = data.dateofbirth,
        sex = data.sex,
        height = data.height,
+       ssn = exports['vms_cityhall']:GenerateSSN(data.dateofbirth, data.sex)
    }
    local currentIdentity = playerIdentity[xPlayer.identifier]
    SetPlayerData(xPlayer, currentIdentity)
```
