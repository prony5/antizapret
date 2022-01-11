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
>> Создаем файл <b>/var/pro-route/hosts</b> для хранения хостов, которые нужно перенаправлять на vpn
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

>> Создаем скрипт <b>/var/pro-route/apply.sh</b> (добавлет статический маршрут для адресов из файла <b>/var/pro-route/hosts</b>)
>> ```Shell
>> #!/bin/bash
>> 
>> iface="$1"
>> [ -z "$iface" ] && exit
>> 
>> logger -t pro-route "Add custom routes for '$iface'"
>> 
>> input="/var/pro-route/hosts"
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

>> Даем права на запуск скрипта <b>/var/pro-route/apply.sh</b>
>> ```Shell
>> chmod +x /var/pro-route/apply.sh
>> ```

>> Добавляем код в <b>/etc/openvpn.user</b>. Скрипт будет выполнятся каждый раз при запуске интерфейса (т.к. при разрыве vpn соединения маршруты сбрасываются).
>> ```Shell
>> [ "$ACTION" = "up" ] && /bin/sh /var/pro-route/apply.sh tun0
>> ```
