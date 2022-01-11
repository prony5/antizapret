# antizapret
Инструкция по настройки роутера с OpenWrt для обхода блокировок.
Все DNS запросы идут через HTTPS (DOH).
Роутер перенаправляет запросы на ip из списка через VPN.
*****

> # 1. Установка <b>DNS HTTPS Proxy</b>
> Можно через ssh: https://openwrt.org/docs/guide-user/services/dns/doh_dnsmasq_https-dns-proxy</br>
> Для LiCI можно через интерфейс (System->Software) https-dns-proxy

> # 2. Установка <b>OpenVPN</b>
> Можно через ssh: https://openwrt.org/docs/guide-user/services/vpn/openvpn/client</br>
> Для LiCI можно через интерфейс (System->Software) luci-app-openvpn

> # 3. Настройка <b>OpenVPN</b> через интерфейс
> 

