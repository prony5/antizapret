# antizapret
Инструкция по настройки роутера с OpenWrt для обхода блокировок.</br>
Все DNS запросы идут через HTTPS (DOH).</br>
Роутер перенаправляет запросы (на адреса из списка) через VPN.
*****

> # 1. Установка <b>DNS HTTPS Proxy</b>
> Можно через ssh: https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy</br>
> Для LiCI можно через интерфейс (System->Software) https-dns-proxy

> # 2. Установка <b>OpenVPN</b>
> Можно через ssh: https://openwrt.org/docs/guide-user/services/vpn/openvpn/client</br>
> Для LiCI можно через интерфейс (System->Software) luci-app-openvpn
>> Файлы конфигурации <b>.OVPN</b> можно найти на (https://www.vpngate.net/en/)
>>> Файл конфигурации <b>.OVPN</b> должен содержать строку `route-noexec`, для того чтобы не создавать маршрутизацию по умолчанию через vpn при подключении

> # 3. Настройка <b>OpenVPN</b> через интерфейс
> ![](https://github.com/prony5/antizapret/blob/main/openvpn1.png)
> ![](https://github.com/prony5/antizapret/blob/main/openvpn2.png)
> ![](https://github.com/prony5/antizapret/blob/main/openvpn3.png)
> ![](https://github.com/prony5/antizapret/blob/main/openvpn4.png)
> ![](https://github.com/prony5/antizapret/blob/main/openvpn5.png)
>> <b>Применяем настройки фаервола:</b> 
>> ```Shell
>> uci commit firewall && /etc/init.d/firewall restart
>> ```

> # 4. Настраиваем перенаправление заблокированных адресов
>> Создаем файл <b>/etc/pro-route/hosts</b> для хранения хостов, которые нужно перенаправлять на vpn
>> ```
>> 185.167.98.127  # rutracker.org
>> 45.132.105.85   # rutracker.org
>> 
>> 104.20.42.23    # 4pda.ru
>> 104.20.41.23    # 4pda.ru
>>
>> kinobase.org
>> kinokrad.co
>> kinokrad.net
>> kinokong.ws
>> kinokong.cc
>> hdrezka.ag
>> hdrezka.cm
>> rezka.ag
>> ```

>> Создаем скрипт <b>/etc/pro-route/apply.sh</b> (добавлет статический маршрут для адресов из файла <b>hosts</b>)
>> ```Shell
>> #!/bin/bash
>> 
>> iface="$1"
>> [ -z "$iface" ] && exit
>> 
>> logger -t pro-route "Add custom routes for '$iface'"
>> 
>> input="/etc/pro-route/hosts"
>> 
>> while IFS= read -r line
>> do
>>   host=$(echo $line | awk -F# '{print $1}')
>>   [ -z "$host" ] && continue
>> 
>>   route add $host $iface
>>   logger -t pro-route "Make route for '$host'"
>> done < "$input"
>> ```

>> Даем права на запуск скрипта <b>/etc/pro-route/apply.sh</b>
>> ```Shell
>> chmod +x /etc/pro-route/apply.sh
>> ```

>> Добавляем код в <b>/etc/openvpn.user</b>. Скрипт будет выполнятся каждый раз при запуске интерфейса (т.к. при разрыве vpn соединения маршруты сбрасываются).
>> ```Shell
>> [ "$ACTION" = "up" ] && /bin/sh /etc/pro-route/apply.sh tun0
>> ```

> # 5. Настраиваем автопереподключение к vpn
>> Соединение с vpn может быть установлено, но маршруты не пробрасываться, по этому проверяем через доступ к тестовому ресурсу. Так же проверяем шлюз, через который идет соединение, т.к. провайдер может подменять заблокированный ресурс на заглушку.
>> 
>> Создаем скрипт <b>/etc/pro-route/check.sh</b>
>> ```Shell
>> #!/bin/bash
>> log(){
>>         echo $1
>>         logger -t pro-route $1
>> }
>> 
>> host="$1"
>> gate="$2"
>> if [ -z "$host" ] || [ -z "$gate" ]
>> then
>>         log "Bad params."
>>         exit
>> fi
>> 
>> success=False
>> i=1
>> while [ $i -le 3 ]
>> do
>>         log "Checking vpn connecton by host '$host' ($i)."
>>         res=$(/bin/traceroute $host -m 3 -w 3)
>> 
>>         if [[ "$res" = *"vpngate"* ]]
>>         then
>>                 success=True
>>                 log "Success"
>>                 break
>>         else
>>                 log "Fail"
>>         fi
>> 
>>         sleep 3
>>         i=$((i+1))
>> done
>> 
>> if [ $success = False ]
>> then
>>         log "Restart vpn"
>>         /etc/init.d/openvpn restart
>> fi
>> ```
>> 
>> Даем права на запуск скрипта <b>/etc/pro-route/check.sh</b>
>> ```Shell
>> chmod +x /etc/pro-route/check.sh
>> ```
>>
>> Добавляем срипт в планировщик
>> ```Shell
>> */10 * * * * /bin/sh /etc/pro-route/check.sh rutracker.org vpngate
>> ```

