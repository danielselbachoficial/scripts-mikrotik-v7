# Script firewall IPv4 - MikroTik RouterOS v7

```bash
# 2025-08-04 22:47:57 by RouterOS 7.19.3

# Lista de interfaces
/interface list add name=WAN
/interface list add name=REDE-LOCAL
/interface list add name=PERMITIR-MNDP

#Tabela de rota - FIB
/routing table add disabled=no fib name=rota-operadora1
/routing table add disabled=no fib name=rota-operadora2

# Boas praticas
/ip smb set enabled=no
/ip firewall connection tracking set enabled=yes tcp-established-timeout=30m
/ip neighbor discovery-settings set discover-interface-list=PERMITIR-MNDP
/ip settings set rp-filter=loose tcp-syncookies=yes
/ip dns set allow-remote-requests=yes cache-size=4096KiB servers=1.1.1.1,1.0.0.1
/ip hotspot service-port set ftp disabled=yes
/ip service set ftp disabled=yes
/ip service set telnet disabled=yes
/ip service set api disabled=yes
/ip service set api-ssl disabled=yes
/ip service set ssh address=<IPs-PUBLICOS-FIXO>,<IPs-PRIVADOS-PERMITIDOS> port=<PORTA>
/ip service set www disabled=yes
/ip service set winbox address=<IPs-PUBLICOS-FIXO>,<IPs-PRIVADOS-PERMITIDOS> port=<PORTA>
/snmp community set [ find default=yes ] disabled=yes
/system clock set time-zone-autodetect=no time-zone-name=America/Sao_Paulo
/system identity set name=<NOME-DO-MK>
/system note set show-at-login=no
/system ntp client set enabled=yes
/system ntp client servers add address=a.ntp.br
/system ntp client servers add address=b.ntp.br
/system ntp client servers add address=c.ntp.br
/tool bandwidth-server set enabled=no
/tool mac-server set allowed-interface-list=none
/tool mac-server mac-winbox set allowed-interface-list=none
/tool mac-server ping set enabled=no
/tool netwatch add comment="=== MONITORAMENTO DO LINK PRINCIPAL ===" disabled=no down-script="ip route/disable [ find comment=\"=== MONITORAMENTO DO LINK PRINCIPAL ===\"]  \r\
    \nlog/error \"LINK PRINCIPAL DO <NOME-DO-MK> CAIU\"" host=199.7.83.42 http-codes="" interval=10s name="MONITORAMENTO DO LINK PRINCIPAL" packet-count=10 packet-size=54 startup-delay=0s test-script="" thr-loss-percent=100% timeout=1s type=icmp up-script="ip route/enable [ find comment=\"=== MONITORAMENTO DO LINK PRINCIPAL ===\"]  \r\
    \nlog/error message=\"LINK PRINCIPAL DO <NOME-DO-MK> VOLTOU\""
/tool netwatch add comment="=== MONITORAMENTO DO LINK BACKUP ===" disabled=no down-script="ip route/disable [ find comment=\"=== MONITORAMENTO DO LINK BACKUP ===\"]  \r\
    \nlog/error \"LINK BACKUP DO <NOME-DO-MK> CAIU\"" host=192.5.5.241 http-codes="" interval=10s name="MONITORAMENTO DO LINK BACKUP" packet-count=10 packet-size=54 startup-delay=0s test-script="" thr-loss-percent=100% timeout=1s type=icmp up-script="ip route/enable [ find comment=\"=== MONITORAMENTO DO LINK BACKUP ===\"]  \r\
    \nlog/error message=\"LINK BACKUP DO <NOME-DO-MK> VOLTOU\""

============================================================================== FIREWALL v4 ========================================================================================

# Firewall v4 - Address List
/ip firewall address-list add address=192.168.1.0/24 list=rede-local-privada
/ip firewall address-list add address=10.0.3.0/24 list=rede-local-privada
/ip firewall address-list add address=10.0.2.0/24 list=rede-local-privada
/ip firewall address-list add address=10.0.1.0/30 list=rede-local-privada
/ip firewall address-list add address=10.0.4.0/24 list=rede-local-privada
/ip firewall address-list add address=10.0.1.0/30 list=rede-local-geral
/ip firewall address-list add address=10.0.3.0/24 list=rede-local-geral
/ip firewall address-list add address=10.0.2.0/24 list=rede-local-geral
/ip firewall address-list add address=0.0.0.0/8 list=BOGONS
/ip firewall address-list add address=10.0.0.0/8 comment="Activate if you are not using this network on your local network" disabled=yes list=BOGONS
/ip firewall address-list add address=100.64.0.0/10 list=BOGONS
/ip firewall address-list add address=127.0.0.0/8 list=BOGONS
/ip firewall address-list add address=169.254.0.0/16 list=BOGONS
/ip firewall address-list add address=172.16.0.0/12 comment="Activate if you are not using this network on your local network" list=BOGONS
/ip firewall address-list add address=192.0.0.0/24 list=BOGONS
/ip firewall address-list add address=192.0.2.0/24 list=BOGONS
/ip firewall address-list add address=192.168.0.0/16 comment="Activate if you are not using this network on your local network" disabled=yes list=BOGONS
/ip firewall address-list add address=198.18.0.0/15 list=BOGONS
/ip firewall address-list add address=198.51.100.0/24 list=BOGONS
/ip firewall address-list add address=203.0.113.0/24 list=BOGONS
/ip firewall address-list add address=224.0.0.0/3 list=BOGONS
/ip firewall address-list add address=<Network/Mask> list=rede-suporte
/ip firewall address-list add address=<Network/Mask> list=rede-local-geral
/ip firewall address-list add address=<Network/Mask> list=rede-local-privada
/ip firewall address-list add address=<Network/Mask> list=rede-corporativa
/ip firewall address-list add address=<Network/Mask> list=rede-visitante
/ip firewall address-list add address=<IP-PUBLICO> list="ACESSO-LIBERADO"

# Firewall v4 - Filter
/ip firewall filter add action=fasttrack-connection chain=forward comment="=== PERMITIR FASTTRACK ===" connection-state=established,related hw-offload=yes disabled=yes
/ip firewall filter add action=drop chain=output comment="=== MONITORAMENTO LINK PRINCIPAL ===" dst-address=199.7.83.42 out-interface=!pppoe-OP1 protocol=icmp
/ip firewall filter add action=drop chain=output comment="=== MONITORAMENTO LINK BACKUP ===" dst-address=192.5.5.241 out-interface=!pppoe-OP2 protocol=icmp
/ip firewall filter add action=accept chain=input comment="=== PERMITIR ESTABELECIDO, RELACIONADO, NO RASTREADO - INPUT ===" connection-state=established,related,untracked
/ip firewall filter add action=accept chain=forward comment="=== PERMITIR TRAFEGO ESTABELECIDO, RELACIONADO E NO RASTREADO - FORWARD ===" connection-state=established,related,untracked
/ip firewall filter add action=drop chain=input comment="=== BLOQUEAR TRAFEGO INVALIDO ===" connection-state=invalid
/ip firewall filter add action=add-src-to-address-list address-list=PORTSCAN address-list-timeout=4w2d chain=input comment="=== PORTSCAN TCP ===" in-interface-list=WAN protocol=tcp psd=21,3s,3,1
/ip firewall filter add action=add-src-to-address-list address-list=PORTSCAN address-list-timeout=4w2d chain=input comment="===  PORTSCAN UDP ===" in-interface-list=WAN protocol=udp psd=21,3s,3,1
/ip firewall filter add action=accept chain=input comment="=== PERMITIR WINBOX EXTERNO TCP ===" dst-port=<PORTA> in-interface-list=WAN log=yes log-prefix=ACESSO-EXTERNO-WINBOX_LIBERADO protocol=tcp src-address-list="ACESSO-LIBERADO"
/ip firewall filter add action=accept chain=input dst-port=<PORTA> protocol=tcp src-address-list=rede-suporte
/ip firewall filter add action=accept chain=input comment="=== PERMITIR SSH ===" dst-port=<PORTA> protocol=tcp src-address-list=rede-suporte
/ip firewall filter add action=accept chain=input comment="===ACEITAR DNS - UDP ===" port=53 protocol=udp
/ip firewall filter add action=accept chain=input comment="=== ACEITAR WIREGUARD VPN ===" dst-port=<PORTA> in-interface-list=WAN protocol=udp
/ip firewall filter add action=accept chain=input comment="=== PERMITIR SNMP - MONITORAMENTO ZABBIX ===" dst-port=<PORTA> protocol=udp src-address-list=MONITORAMENTO
/ip firewall filter add action=accept chain=input comment="=== ACEITAR OSPF PROTOCOL ===" protocol=ospf disabled=yes
/ip firewall filter add action=accept chain=input comment="=== PERMITIR DHCP PROTOCOL ===" dst-port=67,68 in-interface-list=REDE-LOCAL protocol=udp
/ip firewall filter add action=add-src-to-address-list address-list=FASE1 address-list-timeout=3s chain=input dst-port=<PORTA> protocol=tcp comment="=== ADD IP DE ORIGEM PARA LISTA FASE 1 ==="
/ip firewall filter add action=add-src-to-address-list address-list=FASE2 address-list-timeout=3s chain=input dst-port=<PORTA> protocol=tcp comment="=== ADD IP DE ORIGEM PARA LISTA FASE 2 ===" src-address-list=FASE1
/ip firewall filter add action=add-src-to-address-list address-list=LIBERADOS-PORT-KNOCK address-list-timeout=1d chain=input dst-port=<PORTA> comment="=== ADD IP DE ORIGEM PARA LISTA LIBERADOS-PORT-KNOCK ===" protocol=tcp src-address-list=FASE2
/ip firewall filter add action=jump chain=input comment="=== PASSAR PELO CONTROLE ICMP ===" jump-target=CONTROLE-ICMP protocol=icmp
/ip firewall filter add action=accept chain=CONTROLE-ICMP comment="=== ECHO REQUEST - AVOIDING PING FLOOD ===" icmp-options=8:0 limit=2,5:packet protocol=icmp
/ip firewall filter add action=accept chain=CONTROLE-ICMP comment="=== ECHO REPLY ===" icmp-options=0:0-255 limit=30/1m,5:packet protocol=icmp
/ip firewall filter add action=accept chain=CONTROLE-ICMP comment="=== TIME EXCEEDED ===" icmp-options=11:0-255 limit=30/1m,5:packet protocol=icmp
/ip firewall filter add action=accept chain=CONTROLE-ICMP comment="=== UNREACHABLE DESTINATION ===" icmp-options=3:0-255 limit=40/1m,5:packet protocol=icmp
/ip firewall filter add action=accept chain=CONTROLE-ICMP comment="=== PMTUD ===" icmp-options=3:4 protocol=icmp
/ip firewall filter add action=drop chain=CONTROLE-ICMP comment="=== ESQUECA O RESTO DO ICMP ===" protocol=icmp
/ip firewall filter add action=drop chain=input comment="=== BLOQUEIE TODO O OUTRO TRAFEGO DE ENTRADA ===" log-prefix="DROP GERAL"

# Firewall v4 - Mangle
/ip firewall mangle add action=mark-connection chain=input comment="=== MARCA_ENTRADA_MAIS-INTERNET ===" connection-mark=no-mark in-interface=pppoe-OP1 new-connection-mark=in-mais-internet
/ip firewall mangle add action=mark-routing chain=output comment="=== MARCA_SAIDA_MAIS-INTERNET ===" connection-mark=in-mais-internet new-routing-mark=main
/ip firewall mangle add action=mark-connection chain=input comment="=== MARCA_ENTRADA_VIVO ===" connection-mark=no-mark in-interface=pppoe-OP2 new-connection-mark=in-vivo
/ip firewall mangle add action=mark-routing chain=output comment="=== MARCA_SAIDA_VIVO ===" connection-mark=in-vivo new-routing-mark=rota-vivo

# Firewall v4 - NAT
/ip firewall nat add action=redirect chain=dstnat comment="=== DNS CACHE INTERNO ===" disabled=yes dst-port=53 in-interface-list=!WAN protocol=udp to-ports=53
/ip firewall nat add action=dst-nat chain=dstnat comment="=== DNS CACHE INTERNO ===" dst-port=53 in-interface-list=!WAN protocol=udp to-addresses=<GATEWAY-MK> to-ports=53
/ip firewall nat add action=dst-nat chain=dstnat comment="=== DNS CACHE INTERNO ===" dst-port=53 in-interface-list=!WAN protocol=tcp to-addresses=<GATEWAY-MK> to-ports=53
/ip firewall nat add action=src-nat chain=srcnat comment="=== NAT - MAIS INTERNET ===" out-interface=pppoe-OP1 src-address-list=rede-local-privada to-addresses=<IP-PUBLICO-FIXO>
/ip firewall nat add action=src-nat chain=srcnat comment="=== NAT - VIVO ===" out-interface=pppoe-OP2 src-address-list=rede-local-privada to-addresses=<IP-PUBLICO-FIXO>

# Firewall v4 - RAW
/ip firewall raw add action=accept chain=prerouting comment="=== ACEITA 50 PACOTES ICMP POR SEG ===" limit=50,5:packet protocol=icmp
/ip firewall raw add action=drop chain=prerouting comment="=== BLOCK PORTSCAN  ===" log-prefix="BLOCK PORTSCAN" src-address-list=PORTSCAN
/ip firewall raw add action=drop chain=prerouting comment="=== PORTSCAN TCP ===" dst-port=19-25,80,500,1701,1900,3389,4500,5900,8291,11211,32595 in-interface-list=WAN log-prefix=PORTSCAN protocol=tcp
/ip firewall raw add action=drop chain=prerouting comment="=== PORTSCAN UDP ===" dst-port=19-25,80,500,1701,1900,3389,4500,5900,8291,11211,32595 in-interface-list=WAN log-prefix=PORTSCAN protocol=udp
/ip firewall raw add action=drop chain=prerouting comment="=== BLOQUEAR DoS e DDoS ===" dst-address-list=ddosed src-address-list=ddoser
/ip firewall raw add action=drop chain=prerouting dst-port=53 in-interface-list=WAN protocol=tcp
/ip firewall raw add action=drop chain=prerouting dst-port=53 in-interface-list=WAN protocol=udp
/ip firewall raw add action=drop chain=prerouting in-interface-list=WAN protocol=ipsec-esp
/ip firewall raw add action=drop chain=prerouting in-interface-list=WAN protocol=ipsec-ah
/ip firewall raw add action=drop chain=prerouting comment="=== DROP IP-FLOOD ATTACK ===" protocol=icmp
/ip firewall raw add action=drop chain=prerouting comment="=== DROPAR REDE VISITANTE COM DESTINO A REDE CORPORATIVA ===" dst-address-list=rede-corporativa src-address-list=rede-visitante

# ip routes
/ip route add comment="=== MONITORAMENTO DO LINK PRINCIPAL ===" disabled=no distance=1 dst-address=199.7.83.42/32 gateway=<GATEWAY-OP1> routing-table=main scope=30 suppress-hw-offload=no target-scope=10
/ip route add comment="=== LINK BACKUP #2 ===" disabled=no distance=1 dst-address=0.0.0.0/0 gateway=<GATEWAY-OP2> routing-table=<FIB-OP2> scope=30 suppress-hw-offload=no target-scope=10
/ip route add comment="=== MONITORAMENTO DO LINK BACKUP ===" disabled=no distance=1 dst-address=192.5.5.241/32 gateway=<GATEWAY-OP2> routing-table=main scope=30 suppress-hw-offload=no target-scope=10
/ip route add comment="=== LINK PRINCIPAL #1 ===" disabled=no distance=1 dst-address=0.0.0.0/0 gateway=<GATEWAY-OP1> routing-table=<FIB-OP1> scope=30 suppress-hw-offload=no target-scope=10


============================================================================== FIREWALL v6 ========================================================================================

# Firewall v6 - Address List
/ipv6 firewall address-list add address=::/128 comment="=== unspecified address ===" list=bad_ipv6
/ipv6 firewall address-list add address=::1/128 comment="=== lo ===" list=bad_ipv6
/ipv6 firewall address-list add address=fec0::/10 comment="=== site-local ===" list=bad_ipv6
/ipv6 firewall address-list add address=::ffff:0.0.0.0/96 comment="=== ipv4-mapped ===" list=bad_ipv6
/ipv6 firewall address-list add address=::/96 comment="=== ipv4 compat ===" list=bad_ipv6
/ipv6 firewall address-list add address=100::/64 comment="=== discard only ===" list=bad_ipv6
/ipv6 firewall address-list add address=2001:db8::/32 comment="=== documentation ===" list=bad_ipv6
/ipv6 firewall address-list add address=2001:10::/28 comment="=== ORCHID ===" list=bad_ipv6
/ipv6 firewall address-list add address=3ffe::/16 comment="=== 6bone ===" list=bad_ipv6
/ipv6 firewall address-list add address=2001::/23 comment="=== RFC6890 ===" list=bad_ipv6
/ipv6 firewall address-list add address=100::/64 comment="=== RFC6890 Discard-only ===" list=not_global_ipv6
/ipv6 firewall address-list add address=2001::/32 comment="=== RFC6890 TEREDO ===" list=not_global_ipv6
/ipv6 firewall address-list add address=2001:2::/48 comment="=== RFC6890 Benchmark ===" list=not_global_ipv6
/ipv6 firewall address-list add address=fc00::/7 comment="=== RFC6890 Unique-Local ===" list=not_global_ipv6
/ipv6 firewall address-list add address=fe80::/10 comment="=== RFC6890 Linked-Scoped Unicast ===" list=no_forward_ipv6
/ipv6 firewall address-list add address=ff00::/8 comment="=== multicast ===" list=no_forward_ipv6

# Firewall v6 - Filter
/ipv6 firewall filter add action=accept chain=input comment="=== ICMPv6 ===" protocol=icmpv6
/ipv6 firewall filter add action=accept chain=input comment="=== Estabelecidas,relacionadas,untracked ===" connection-state=established,related,untracked
/ipv6 firewall filter add action=accept chain=input comment="=== UDP traceroute ===" port=33434-33534 protocol=udp
/ipv6 firewall filter add action=accept chain=input comment="=== DHCPv6-Client prefix delegation. ===" dst-port=546 protocol=udp src-address=fe80::/16
/ipv6 firewall filter add action=accept chain=input comment="=== IKE - IPSEC ===" dst-port=500,4500 protocol=udp
/ipv6 firewall filter add action=accept chain=input comment="===IPSec AH ===" protocol=ipsec-ah
/ipv6 firewall filter add action=accept chain=input comment="=== IPSec ESP ===" prot""ocol=ipsec-esp
/ipv6 firewall filter add action=accept chain=input comment="=== Winbox dst-port=<PORTA> ===" protocol=tcp
/ipv6 firewall filter add action=accept chain=input comment="=== SSH dst-port=<PORTA> ===" protocol=tcp
/ipv6 firewall filter add action=drop chain=input comment="=== DROP o restante exceto vindo da REDE-LOCAL ===" in-interface-list=!REDE-LOCAL
/ipv6 firewall filter add action=accept chain=forward comment="=== Estabelecidas,relacionadas,untracked ===" connection-state=established,related,untracked
/ipv6 firewall filter add action=drop chain=forward comment="=== Invalidas ===" connection-state=invalid
/ipv6 firewall filter add action=drop chain=forward comment="=== IPs que nao podem passar pelo router ===" src-address-list=no_forward_ipv6
/ipv6 firewall filter add action=drop chain=forward comment="=== IPs que nao podem passar pelo router ===" dst-address-list=no_forward_ipv6
/ipv6 firewall filter add action=drop chain=forward comment="=== rfc4890 drop hop-limit=1 ===" hop-limit=equal:1 protocol=icmpv6
/ipv6 firewall filter add action=accept chain=forward comment="=== ICMPv6 ===" protocol=icmpv6
/ipv6 firewall filter add action=accept chain=forward comment="=== HIP ===" protocol=139
/ipv6 firewall filter add action=accept chain=forward comment="=== IKE ===" dst-port=500,4500 protocol=udp
/ipv6 firewall filter add action=accept chain=forward comment="=== AH ===" protocol=ipsec-ah
/ipv6 firewall filter add action=accept chain=forward comment="=== IPSec ESP ===" protocol=ipsec-esp
/ipv6 firewall filter add action=accept chain=forward comment="=== IPSec policy ===" ipsec-policy=in,ipsec
/ipv6 firewall filter add action=drop chain=forward comment="=== DROP o resto exceto vindo da REDE-LOCAL ===" in-interface-list=!REDE-LOCAL

# Firewall v6 - RAW
/ipv6 firewall raw add action=accept chain=prerouting comment="=== enable for transparent firewall ===" disabled=yes
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" src-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" dst-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="===drop packets with bad SRC ipv6 ===" src-address-list=bad_src_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop packets with bad dst ipv6 ===" dst-address-list=bad_dst_ipv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== jump to ICMPv6 chain ===" protocol=icmpv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept local multicast scope ===" dst-address=ff02::/16
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop other multicast destinations ===" dst-address=ff00::/8
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop the rest ==="
/ipv6 firewall raw add action=accept chain=prerouting comment="=== enable for transparent firewall ===" disabled=yes
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" src-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" dst-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop packets with bad SRC ipv6 ===" src-address-list=bad_src_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop packets with bad dst ipv6 ===" dst-address-list=bad_dst_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop non global from WAN ===" in-interface-list=WAN src-address-list=not_global_ipv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== jump to ICMPv6 chain ===" protocol=icmpv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept local multicast scope ===" dst-address=ff02::/16
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop other multicast destinations ===" dst-address=ff00::/8
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept everything else from WAN ===" in-interface-list=WAN
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept everything else from REDE-LOCAL ===" in-interface-list=REDE-LOCAL
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop the rest ==="
/ipv6 firewall raw add action=accept chain=prerouting comment="=== enable for transparent firewall ===" disabled=yes
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" src-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop bogon IP's ===" dst-address-list=bad_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop packets with bad SRC ipv6 ===" src-address-list=bad_src_ipv6
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop packets with bad dst ipv6 ===" dst-address-list=bad_dst_ipv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== jump to ICMPv6 chain ===" protocol=icmpv6
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept local multicast scope ===" dst-address=ff02::/16
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop other multicast destinations ===" dst-address=ff00::/8
/ipv6 firewall raw add action=accept chain=prerouting comment="=== accept everything else from REDE-LOCAL ===" in-interface-list=REDE-LOCAL
/ipv6 firewall raw add action=drop chain=prerouting comment="=== drop the rest ==="
```
