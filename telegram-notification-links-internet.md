# Scripts de Monitoramento MikroTik com Alertas via Telegram

Este repositório contém um conjunto de scripts para o sistema MikroTik RouterOS que monitoram a estabilidade de links de internet (um principal e um de backup) e enviam notificações em tempo real para um grupo ou canal do Telegram quando um link cai ou é restabelecido.

## Funcionalidades

-   **Monitoramento Duplo:** Acompanha um link principal (PPPoE) e um link de backup (Ethernet).
-   **Alertas Instantâneos:** Notifica imediatamente sobre quedas e restabelecimentos.
-   **Mensagens Detalhadas:** Os alertas incluem o nome do roteador, o link afetado, o status e o horário exato do evento.
-   **Fácil de Adaptar:** As variáveis principais (tokens, IDs e nomes) estão no início de cada script para fácil personalização.

---

## Pré-requisitos

Antes de começar, você precisará de:

1.  **Um roteador MikroTik** com acesso à internet.
2.  **Um Bot no Telegram:**
    * Converse com o **[@BotFather](https://t.me/BotFather)** no Telegram.
    * Use o comando `/newbot` para criar um novo bot.
    * O BotFather fornecerá um **Token de Acesso (Bot Token)**. Guarde-o.
3.  **O ID do Chat do Telegram (Chat ID):**
    * Adicione seu bot recém-criado a um grupo do Telegram.
    * Envie qualquer mensagem nesse grupo.
    * Acesse a seguinte URL no seu navegador, substituindo `<SEU_BOT_TOKEN>` pelo token que você guardou:
        `https://api.telegram.org/bot<SEU_BOT_TOKEN>/getUpdates`
    * Procure por `"chat":{"id":-123456789,...}` no texto que aparecer. O número negativo é o seu **Chat ID**.

---

## Passo a Passo da Implementação

### Passo 1: Personalizar os Scripts

Antes de subir os scripts para o seu MikroTik, você precisa editar as informações de acordo com a sua necessidade. Altere as 3 primeiras linhas de cada um dos 4 arquivos de script:

-   `:local botToken "..."`: Coloque o token do seu bot aqui.
-   `:local chatId "..."`: Coloque o ID do seu chat aqui.
-   `:local routerName "..."`: Defina um nome para identificar seu roteador nos alertas.

### Passo 2: Adicionar os Scripts ao MikroTik

1.  Acesse seu MikroTik (via WinBox ou WebFig).
2.  Navegue até **System > Scripts**.
3.  Crie 4 novos scripts, um para cada arquivo deste repositório. Dê a eles nomes fáceis de identificar, como os nomes dos arquivos (`telegram-RedLink-DOWN`, etc.).
4.  Para cada script, copie e cole o conteúdo correspondente no campo **Source**.

#### Scripts
```
Script 1: telegram-RedLink-DOWN (com Horário) -> LINK PPPoE Client

:local botToken "<SEU_BOT_TOKEN>" 
:local chatId "-123456789"
:local routerName "<NOME-DO-SEU-ROTEADOR-MK>"

# Mensagem
:local message ("🚨 *Alerta em " . $routerName . "*: O link <NOME-DA-INTERFACE-PPPOE-CLIENT> caiu!")

# Monta a URL base 
:local url "https://api.telegram.org/bot$botToken/sendMessage"

# Monta o corpo da requisição (os dados que enviaremos)
:local data "chat_id=$chatId&parse_mode=Markdown&text=$message"

# Envia a notificação usando o método POST 
/tool fetch url=$url http-method=post http-data=$data keep-result=no


Script 2: telegram-RedLink-UP (com Horário) -> LINK PPPoE Client

:local botToken "<SEU_BOT_TOKEN>" 
:local chatId "-123456789"
:local routerName "<NOME-DO-SEU-ROTEADOR-MK>"
:local linkName "<NOME-DA-INTERFACE-PPPOE-CLIENT>"

# --- Captura data e hora do evento ---
:local eventDate [/system clock get date]
:local eventTime [/system clock get time]

# Mensagem multi-linha formatada (usando %0A para quebra de linha)
:local message "✅ *Normalização de Link*%0A%0A"
:set message ($message . "*Roteador:* `" . $routerName . "`%0A")
:set message ($message . "*Link:* `" . $linkName . "`%0A")
:set message ($message . "*Status:* RESTABELECIDO%0A")
:set message ($message . "*Horário:* " . $eventDate . " às " . $eventTime)

# Monta a URL base
:local url "https://api.telegram.org/bot$botToken/sendMessage"

# Monta o corpo da requisição
:local data "chat_id=$chatId&parse_mode=Markdown&text=$message"

# Envia a notificação usando o método POST
/tool fetch url=$url http-method=post http-data=$data keep-result=no


Script 3: telegram-Vivo-DOWN (com Horário) -> LINK DMZ

:local botToken "<SEU_BOT_TOKEN>" 
:local chatId "-123456789"
:local routerName "<NOME-DO-SEU-ROTEADOR-MK>"
:local linkName "<NOME-DA-INTERFACE-LINK-DMZ>"

# --- Captura data e hora do evento ---
:local eventDate [/system clock get date]
:local eventTime [/system clock get time]

# Mensagem multi-linha formatada (usando %0A para quebra de linha)
:local message "🚨 *Alerta de Link de Backup*%0A%0A"
:set message ($message . "*Roteador:* `" . $routerName . "`%0A")
:set message ($message . "*Link:* `" . $linkName . "`%0A")
:set message ($message . "*Status:* CAIU%0A")
:set message ($message . "*Horário:* " . $eventDate . " às " . $eventTime)

# Monta a URL base
:local url "https://api.telegram.org/bot$botToken/sendMessage"

# Monta o corpo da requisição
:local data "chat_id=$chatId&parse_mode=Markdown&text=$message"

# Envia a notificação usando o método POST
/tool fetch url=$url http-method=post http-data=$data keep-result=no


Script 4: telegram-Vivo-UP (com Horário) -> LINK DMZ

:local botToken "<SEU_BOT_TOKEN>" 
:local chatId "-123456789"
:local routerName "<NOME-DO-SEU-ROTEADOR-MK>"
:local linkName "<NOME-DA-INTERFACE-LINK-DMZ>"

# --- Captura data e hora do evento ---
:local eventDate [/system clock get date]
:local eventTime [/system clock get time]

# Mensagem multi-linha formatada (usando %0A para quebra de linha)
:local message "✅ *Normalização de Link de Backup*%0A%0A"
:set message ($message . "*Roteador:* `" . $routerName . "`%0A")
:set message ($message . "*Link:* `" . $linkName . "`%0A")
:set message ($message . "*Status:* RESTABELECIDO%0A")
:set message ($message . "*Horário:* " . $eventDate . " às " . $eventTime)

# Monta a URL base
:local url "https://api.telegram.org/bot$botToken/sendMessage"

# Monta o corpo da requisição
:local data "chat_id=$chatId&parse_mode=Markdown&text=$message"

# Envia a notificação usando o método POST
/tool fetch url=$url http-method=post http-data=$data keep-result=no
```

### Passo 3: Configurar o Netwatch (O Gatilho)

O Netwatch será o responsável por monitorar os links e disparar os scripts. Vamos criar duas regras, uma para cada link.

#### Monitorando o Link Principal (RedLink - PPPoE)

1.  Vá para **Tools > Netwatch** e clique no `+`.
2.  **Aba Host:**
    * **Host:** `8.8.8.8` (ou um IP confiável que você saiba que só é acessível pelo link principal).
    * **Interval:** `00:00:10` (a cada 10 segundos).
3.  **Aba Down:** Cole o nome do seu script de queda: `telegram-RedLink-DOWN`
4.  **Aba Up:** Cole o nome do seu script de restabelecimento: `telegram-RedLink-UP`
5.  Clique em **OK**.

#### Monitorando o Link de Backup (Vivo - Ethernet)

1.  Clique no `+` novamente no Netwatch.
2.  **Aba Host:**
    * **Host:** `1.1.1.1` (use um IP diferente do primeiro para garantir o monitoramento independente).
    * **Interval:** `00:00:10`.
3.  **Aba Down:** Cole o nome do seu script de queda: `telegram-Vivo-DOWN`
4.  **Aba Up:** Cole o nome do seu script de restabelecimento: `telegram-Vivo-UP`
5.  Clique em **OK**.

Pronto! Seu sistema de monitoramento via Telegram está ativo. Ao desconectar um dos cabos ou quando um dos links falhar, o Netwatch detectará a falha e executará o script correspondente, enviando o alerta para seu Telegram.
