# qb-vehiclekeys  
Araç anahtarları item olarak

## Destek  
Lütfen Discord sunucuma katıl:https://discord.gg/qKNCwpEu6Q

## Özellikler  
- Anahtar, araç plakası ve modeli ile item açıklamasında oluşturulur  
- Anahtarlar araç satın alındığında verilir (ilgili eventler mevcuttur)  
- Çilingir, ek anahtar satın almak ve araç kilitlerini değiştirmek için kullanılır  
- Oyuncu araca binmeye çalıştığında, aracın kilit değeri kontrol edilir  
- NPC araçları GTA usulü erişilebilir (araba kapma veya cam kırma ile)  
- Araç kilitliyse oyuncu lockpick kullanabilir, değilse doğrudan binebilir  
- Oyuncu araçta ise envanterinde anahtar itemi olup olmadığı kontrol edilir, motoru çalıştırmak için  
- Anahtarı yoksa, aracı düzkontak (hotwire) yapmayı deneyebilir  
- Admin araçları (/car komutu ile) için araç geçici olarak oyuncunun olur, yani "görünmez eski tip anahtar" verilir  
- Bir meslek ücretsiz araç spawn ettiğinde oyuncuya yine bu eski tip geçici anahtar verilir, düzkontak yapılmaz  
- Araç satıldığında, anahtar envanterden kaldırılabilir  
- Anahtarlar asla silinmez veya garaj tarafından tekrar verilmez, anahtarları envanterde veya depoda tutmanız gerekir

## Gereksinimler  
- [qb-core](https://github.com/qbcore-framework/qb-core)

## Kurulum  
- qbcore içinden eski qb-vehiclekeys kaynak dosyasını silin  
- Yeni kaynağı yükleyin 
- `vehiclekeys` görselini `img` klasöründen `qb-inventory\html\images` içine kopyalayın  
- `player_vehicles.sql` dosyasını veritabanınıza import edin  
- `qb-core/shared/items.lua` dosyasına aşağıdaki satırı ekleyin:  
```lua
    vehiclekey = { name = 'vehiclekey', label = 'Araç Anahtarı', weight = 10, type = 'item', image = 'vehiclekeys.png', unique = true, useable = true, shouldClose = true, combinable = nil, description = "Bu bir araç anahtarıdır, iyi koruyun, kaybederseniz aracınızı kullanamayabilirsiniz" },
qb-inventory\html\js\app.js dosyasında, yaklaşık 395. satırda bulunan generateDescription fonksiyonuna şu satırları ekleyin:

js
Kopyala
Düzenle
        case "labkey":
            return `<p>Lab: ${itemData.info.lab}</p>`;
        case "vehiclekey":                                                                       // Yeni ekleme
            return `<p><strong>Araç: </strong><span>${itemData.info.model}</span></p>
                    <p><strong>Plaka: </strong><span>${itemData.info.plate}</span></p>`;          // Yeni ekleme
        default:
            return itemData.description;
Araç anahtarları için kullanılan iki önemli event qb-vehicleshop/client.lua içinde:
Araç satın alındığında anahtar yaratılır:
'qb-vehiclekeys:server:BuyVehicle'
Bu eventi araç mağazası scriptine şöyle eklemeniz gerekir:

lua
Kopyala
Düzenle
RegisterNetEvent('qb-vehicleshop:client:buyShowroomVehicle', function(vehicle, plate)
    tempShop = insideShop -- Geçici, shop bilgisini ayarlamak için
    QBCore.Functions.TriggerCallback('QBCore:Server:SpawnVehicle', function(netId)
        local veh = NetToVeh(netId)
        exports['LegacyFuel']:SetFuel(veh, 100)
        SetVehicleNumberPlateText(veh, plate)
        SetEntityHeading(veh, Config.Shops[tempShop]["VehicleSpawn"].w)
        --TriggerEvent("vehiclekeys:client:SetOwner", QBCore.Functions.GetPlate(veh))   -- Bu satırı yorum satırına al (sil)
        TriggerServerEvent('qb-vehiclekeys:server:BuyVehicle', plate, GetLabelText(GetDisplayNameFromVehicleModel(GetEntityModel(veh))))  -- Bu satırı ekle
    end, vehicle, Config.Shops[tempShop]["VehicleSpawn"], true)
end)
Anahtarlar mağaza tarafından verildiği için, garajda verilen geçici anahtar eventi 'vehiclekeys:client:SetOwner' yorum satırına alınmalıdır.

Bu event bazen 'qb-vehiclekeys:server:AcquireVehicleKeys' olarak farklı adlandırılmış olabilir, işlevi aynıdır.

Garaj scriptinde, örneğin qb-garage/client/main.lua dosyasında:

lua
Kopyala
Düzenle
RegisterNetEvent('qb-garages:client:takeOutGarage', function(data)
    local type = data.type
    local vehicle = data.vehicle
    local garage = data.garage
    local index = data.index
    QBCore.Functions.TriggerCallback('qb-garage:server:IsSpawnOk', function(spawn)
        if spawn then
            local location
            if type == "house" then
                location = garage.takeVehicle
            else
                location = garage.spawnPoint
            end
            QBCore.Functions.TriggerCallback('qb-garage:server:spawnvehicle', function(netId, properties)
                local veh = NetToVeh(netId)
                QBCore.Functions.SetVehicleProperties(veh, properties)
                exports['LegacyFuel']:SetFuel(veh, vehicle.fuel)
                doCarDamage(veh, vehicle)
                TriggerServerEvent('qb-garage:server:updateVehicleState', 0, vehicle.plate, index)
                closeMenuFull()
                --TriggerEvent("vehiclekeys:client:SetOwner", QBCore.Functions.GetPlate(veh))   -- Bu satırı yorum satırına al
                SetVehicleEngineOn(veh, true, true)
                if type == "house" then
                    exports['qb-core']:DrawText(Lang:t("info.park_e"), 'left')
                    InputOut = false
                    InputIn = true
                end
            end, vehicle, location, true)
        else
            QBCore.Functions.Notify(Lang:t("error.not_impound"), "error", 5000)
        end
    end, vehicle.plate, type)
end)
Oyuncunun envanterinden anahtarı otomatik kaldırmak için şu event kullanılabilir:
lua
Kopyala
Düzenle
TriggerServerEvent('qb-vehiclekeys:server:RemoveKey', plate)
Özet:
Bu scriptte araç anahtarları fiziksel item olarak tutulur, anahtarlar aracın plakası ve modeliyle bağlanır. Araç satın alındığında anahtar verilir, garajdan çıkışta eski geçici anahtar mekanizması iptal edilir, böylece gerçek anahtar sistemi kullanılır.

