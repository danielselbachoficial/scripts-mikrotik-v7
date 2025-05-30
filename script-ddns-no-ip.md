# Atualização automática do IP Público com DDNS No-IP no MikroTik RouterOS v7
> Desenvolvido por: **Gilson Telecom**.
>
> Atualizador por: **Daniel Selbach**
>
> Instagram - Daniel Selbach: https://www.instagram.com/danielselbach.com.br/
>
> YouTube - Daniel Selbach: https://www.youtube.com/@danielselbachoficial


## Script de Atualização automática do IP Público com DDNS No-IP
```bash
# Alterar as informacoes desta secao conforme os dados do seu login e host no-ip
:local noipuser "usuario"
:local noippass "senha"
:local noiphost "nomedomeuhost.no-ip.org"

# Nome da interface que devera ter o endereco IP vinculado ao host do no-ip
:local inetinterface "pppoe-out1"

:global previousIP

:if ([/interface get $inetinterface value-name=running]) do={
# Obtendo informacao sobre o IP atual
   :local currentIP [/ip address get [find interface="$inetinterface" disabled=no] address]
   :for i from=( [:len $currentIP] - 1) to=0 do={
       :if ( [:pick $currentIP $i] = "/") do={ 
           :set currentIP [:pick $currentIP 0 $i]
       } 
   }



  :if ($currentIP != $previousIP) do={
       :log info "No-IP: IP atual $currentIP diferente do IP anterior, atualizando."
       :set previousIP $currentIP

       # Enviando o novo IP via http
       :log info "No-IP: Atualizando o host $noiphost"
       /tool fetch mode=http user=$noipuser password=$noippass url="http://dynupdate.no-ip.com/nic/update?hostname=dominio&myip=IP" keep-result=no
       :log info "No-IP: Host $noiphost atualizado no No-IP = $currentIP"
   }
} else={
   :log info "No-IP: $inetinterface desconectada. Impossivel atualizar No-IP."
}
```
