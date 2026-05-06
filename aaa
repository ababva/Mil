# 7. Создание bridge-интерфейсов
interface bridge add name=LVS
interface bridge add name=BSHPD
interface bridge print

# 8. Настройка DHCP-сервера для LVS (пул и сеть изменены на 123)
ip pool add name=dhcp_pool_LVS ranges=10.245.123.2-10.245.123.191
ip dhcp-server add name=dhcp_LVS interface=LVS address-pool=dhcp_pool_LVS lease-time=10m
ip dhcp-server network add address=10.245.123.0/25 gateway=10.245.123.1 dns-server=8.8.8.8
ip dhcp-server print

# 9. Добавление портов в bridge
interface bridge port remove [find interface=ether8]
interface bridge port add interface=ether8 bridge=BSHPD
interface bridge port remove [find interface=ether1]
interface bridge port add interface=ether1 bridge=LVS
interface bridge port remove [find interface=ether3]
interface bridge port add interface=ether3 bridge=LVS

# Дополнительные порты (ether4,5,6,9,10) в LVS
interface bridge port remove [find interface=ether4]
interface bridge port add interface=ether4 bridge=LVS
interface bridge port remove [find interface=ether5]
interface bridge port add interface=ether5 bridge=LVS
interface bridge port remove [find interface=ether6]
interface bridge port add interface=ether6 bridge=LVS
interface bridge port remove [find interface=ether9]
interface bridge port add interface=ether9 bridge=LVS
interface bridge port remove [find interface=ether10]
interface bridge port add interface=ether10 bridge=LVS
interface bridge port print

# Назначение IP-адресов (подсеть 122 заменена на 123)
ip address add address=172.16.20.3/24 interface=BSHPD
ip address add address=10.245.123.1/25 interface=LVS
ip address add address=10.245.123.129/26 interface=LVS
ip address add address=10.245.123.193/30 interface=ether7
ip address add address=10.245.123.197/30 interface=ether7
ip address print

# Отключение дефолтного запрещающего правила
ip firewall filter set [find comment="defconf: drop all not coming from LAN"] disabled=yes

# Список bgp_export с сетями 123 (132 оставлена)
ip firewall address-list add address=10.245.123.0/25 list=bgp_export
ip firewall address-list add address=10.245.123.128/26 list=bgp_export
ip firewall address-list add address=10.245.123.192/30 list=bgp_export
ip firewall address-list add address=10.245.123.196/30 list=bgp_export
ip firewall address-list add address=10.245.132.0/30 list=bgp_export
ip firewall address-list print

# 12. Правила фильтрации BGP (122.0/24 -> 123.0/24)
routing filter rule add chain=bgp_in disabled=no rule="if(dst in 192.168.0.0/16){reject;}"
routing filter rule add chain=export_local disable=no rule="if(dst in 10.245.123.0/24){accept;}"
routing filter rule add chain=export_local disable=no rule="if(dst = 10.245.132.0/24){accept}"

# 13. Правило local-pref
routing filter rule add chain=import_local_pref300 disable=no rule="set bgp-local-pref 300; accept"

# 14. Статический маршрут (шлюз изменён на 123.194)
ip route add dst-address=10.245.132.0/24 gateway=10.245.123.194
ip route print

# 15. BGP-шаблон (router-id изменён на 123.1)
routing bgp template add as=64522 disable=no input.filter=bgp_in multihop=yes output.filter-chain=export_local network=bgp_export router-id=10.245.123.1 name=GONETS routing-table=main
routing bgp template print

# 16. BGP-соединение (router-id изменён)
routing bgp connection add disable=no connect=yes listen=yes input.filter=import_local_pref300 output.filter-chain=export_local local.role=ebgp local.address=172.16.20.3 name=GONETS_BSHPD remote.address=172.16.20.2 remote.as=64521 router-id=10.245.123.1 routing-table=main templates=GONETS
routing bgp session print
ip route print
