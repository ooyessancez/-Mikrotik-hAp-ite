# -Mikrotik-hAp-ite
## Полная инструкция: Сброс и настройка MikroTik hAP lite

### Часть 1: Сброс роутера к заводским настройкам

#### Вариант A: Через WinBox/WebFig (если есть доступ)
1. **Подключитесь** к роутеру через WinBox или зайдите в WebFig (192.168.88.1)
2. **Системный сброс:**
   ```
   /system reset-configuration
   ```
   В появившемся диалоге:
   - ✅ `No Default Configuration` - **НЕ ставить галочку**
   - ✅ `Keep Users` - снять галочку (если нужно удалить всех пользователей)
   - ✅ `Skip Backup` - поставить галочку
   - Нажать `Reset Configuration`

#### Вариант B: Через кнопку Reset (физический сброс)
1. **Отключите** питание роутера
2. **Зажмите** кнопку Reset на задней панели
3. **Подключите** питание, не отпуская кнопку
4. **Держите** кнопку 10-15 секунд до мигания всех светодиодов
5. **Отпустите** кнопку - роутер перезагрузится с заводскими настройками

#### Вариант C: Через терминал (SSH/Telnet/Console)
```bash
# Подключитесь к роутеру
ssh admin@192.168.88.1

# Выполните команду сброса
/system reset-configuration no-defaults=no skip-backup=yes
```

После сброса:
- IP адрес: **192.168.88.1**
- Логин: **admin**
- Пароль: **пустой** (оставить поле пустым)

---

### Часть 2: Базовая настройка роутера

#### Шаг 1: Подключение и вход
1. Подключите компьютер к порту **ether2** на hAP lite
2. Настройте на компьютере IP автоматически (DHCP)
3. Откройте браузер: `http://192.168.88.1`
4. Войдите с логином `admin`, пароль оставьте пустым

#### Шаг 2: Основные команды настройки (через терминал)

```bash
# --- 1. Смена пароля администратора ---
/user set admin password="ваш_надежный_пароль"

# --- 2. Настройка интернета (WAN) ---
# Предполагаем, что интернет подключен в port1 (ether1)
/ip address add address=192.168.88.1/24 interface=bridge
/ip dhcp-client add interface=ether1 disabled=no

# ИЛИ если нужен статический IP:
/ip address add address=ваш_wan_IP/маска interface=ether1
/ip route add gateway=шлюз_провайдера

# --- 3. Настройка локальной сети (LAN) ---
# Добавляем порты в бридж (по умолчанию уже есть bridge1)
/interface bridge port add bridge=bridge1 interface=ether2
/interface bridge port add bridge=bridge1 interface=ether3
/interface bridge port add bridge=bridge1 interface=ether4
/interface bridge port add bridge=bridge1 interface=ether5

# --- 4. Настройка DHCP сервера ---
/ip pool add name=dhcp_pool ranges=192.168.88.10-192.168.88.254
/ip dhcp-server add address-pool=dhcp_pool disabled=no interface=bridge1 name=dhcp1
/ip dhcp-server network add address=192.168.88.0/24 dns-server=8.8.8.8,1.1.1.1 gateway=192.168.88.1

# --- 5. Настройка NAT (маскарадинг) ---
/ip firewall nat add action=masquerade chain=srcnat out-interface=ether1

# --- 6. Настройка DNS ---
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes

# --- 7. Настройка Wi-Fi (для hAP lite) ---
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid="MikroTik-XXXX" band=2ghz-b/g/n channel-width=20mhz frequency=auto wireless-protocol=802.11
/interface wireless security-profiles set [ find default=yes ] authentication-types=wpa2-psk mode=dynamic-keys wpa2-pre-shared-key="ваш_пароль_wi-fi"

# Добавляем Wi-Fi в бридж
/interface bridge port add bridge=bridge1 interface=wlan1

# --- 8. Включение защиты (опционально) ---
# Блокировка внешних запросов
/ip firewall filter add chain=input action=drop in-interface=ether1
/ip firewall filter add chain=input action=accept connection-state=established,related
/ip firewall filter add chain=input action=accept in-interface=bridge1

# --- 9. Сохранение конфигурации ---
/system backup save name=initial-configuration
```

#### Шаг 3: Проверка работы
```bash
# Проверка интерфейсов
/interface print

# Проверка IP адресов
/ip address print

# Проверка соединения
/ping 8.8.8.8
```

---

### Часть 3: Альтернативный способ - QuickSet

Для быстрой настройки через веб-интерфейс:

1. Откройте **WebFig** → **QuickSet**
2. Выберите режим:
   - **Home AP Dual** - если роутер подключен к интернету
   - **Home AP** - если только точка доступа
3. Заполните:
   - `Internet Port` - выберите ether1
   - `Address Acquisition` - DHCP или Static
   - `WiFi Password` - пароль Wi-Fi
   - `Router Password` - пароль администратора
4. Нажмите **Apply**

---

### Важные примечания:

1. **Порты hAP lite:**
   - **ether1** - обычно WAN (интернет)
   - **ether2-ether5** - LAN порты
   - **wlan1** - беспроводной интерфейс

2. **После сброса:**
   - Все предыдущие настройки удаляются
   - Роутер получает адрес 192.168.88.1
   - Включается DHCP сервер для клиентов
   - Wi-Fi с названием "MikroTik" без пароля

3. **Безопасность:**
   - Обязательно смените пароль администратора
   - Установите пароль на Wi-Fi
   - Рассмотрите настройку firewall

4. **Сохраните конфигурацию:**
   ```
   /system backup save name=my-config
   ```

При возникновении проблем можно снова выполнить сброс и начать заново.
