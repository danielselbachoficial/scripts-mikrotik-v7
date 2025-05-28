# Script firewall IPv4 Básico - MikroTik RouterOS v7

# Address-list

/ip firewall address-list add address=192.168.10.0/24 list=rede-local-privada

# Firewall Filter

/ip firewall filter add action=accept chain=input comment="PERMITIR CONEXOES ESTABELECIDAS, RELACIONADAS E UNTRACKED" connection-state=established,related,untracked/
/ip firewall filter add action=accept chain=forward comment="PERMITIR CONEXOES ESTABELECIDAS, RELACIONADAS E UNTRACKED - FORWARD" connection-state=established,related,untracked
/ip firewall filter add action=drop chain=input comment="BLOQUEAR CONEXOES INVALIDAS" connection-state=invalid
/ip firewall filter add action=add-src-to-address-list address-list=port-scanners address-list-timeout=4w2d chain=input comment="=== PEGA MALANDRO - TCP ===" dst-port=3389,21-25 in-interface=ether1 protocol=tcp
/ip firewall filter add action=add-src-to-address-list address-list=port-scanners address-list-timeout=4w2d chain=input comment="=== PEGA MALANDRO - UDP ===" dst-port=3389,21-25 in-interface=ether1 protocol=udp
/ip firewall filter add action=accept chain=input comment="PERMITIR SSH" dst-port=5000 protocol=tcp
/ip firewall filter add action=accept chain=input comment="PERMITIR WINBOX" dst-port=5100 protocol=tcp
/ip firewall filter add action=accept chain=input comment="PERMTIR DHCP" dst-port=67,68 protocol=udp src-address-list=rede-local-privada
/ip firewall filter add action=add-src-to-address-list address-list=port-scanners address-list-timeout=2w chain=input comment="=== PORTSCAN TCP === " in-interface=ether1 protocol=tcp psd=21,3s,3,1
/ip firewall filter add action=add-src-to-address-list address-list=port-scanners address-list-timeout=4w2d chain=input comment="=== PORTSCAN UDP ===" in-interface=ether1 protocol=udp psd=21,3s,3,1
/ip firewall filter add action=drop chain=input comment="PROTECAO - DNS UDP" in-interface=ether1 protocol=udp
/ip firewall filter add action=drop chain=input comment="PROTECAO - DNS TCP" in-interface=ether1 protocol=tcp
/ip firewall filter add action=accept chain=input comment="PERMITIR DNS" dst-port=53 protocol=udp
/ip firewall filter add action=drop chain=input comment="DROP GERAL"

# Firewall NAT

/ip firewall nat add action=masquerade chain=srcnat comment=NAT out-interface=ether1

# Firewall RAW

/ip firewall raw add action=drop chain=prerouting comment="=== BLOQUEAR PORTSCAN ===" log-prefix=PORTSCAN src-address-list=PORTSCAN
/ip firewall raw add action=drop chain=prerouting comment="=== PORTSCAN TCP ===" dst-port=19-25,50,80,443,500,1701,1900,3389,4500,5900,8291,11211,32595 in-interface=ether1 log-prefix=PORTSCAN protocol=tcp
/ip firewall raw add action=drop chain=prerouting comment="=== PORTSCAN UDP ===" dst-port=19-25,50,80,443,500,1701,1900,3389,4500,5900,8291,11211,32595 in-interface=ether1 log-prefix=PORTSCAN protocol=udp
/ip firewall raw add action=accept chain=prerouting comment="=== ACEITAR 50 PACOTES ICMP POR SEGUNDO ===" limit=50,5:packet protocol=icmp
/ip firewall raw add action=drop chain=prerouting comment="=== DROPAR ATAQUE DDoS ===" dst-port=53 in-interface=ether1 protocol=tcp
/ip firewall raw add action=drop chain=prerouting dst-port=53 in-interface=ether1 protocol=udp
/ip firewall raw add action=drop chain=prerouting comment="=== DROPAR ATAQUE IP-FLOOD ===" protocol=icmp
