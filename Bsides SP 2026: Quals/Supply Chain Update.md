### Contexto: Um ataque de supply chain acontece quando o atacante compromete algo que parece confiável, como uma atualização de software.

### Em vez de atacar diretamente a vítima, ele entrega um “update” falso.

## PASSO A PASSO

Após baixar o arquivo .pcap, uso o comando strings para encontrar palavras no arquivo

<img width="360" height="438" alt="image" src="https://github.com/user-attachments/assets/57bdc2cc-3336-494c-a793-49d2b7585ac8" />

Encontrei domínios legítimos:

```
updates.microsoft.com
repo.debian.org
nfe.fazenda.gov.br
ocsp.globalsign.com
cdp.microsoft.com
download.windowsupdate.com
api.nuget.org
packages.microsoft.com
```

E domínios suspeitos: 

```
updates.ssin-cloud.com
cdn.ssin-cloud.net
updates.ssin-cloud.net
static.ssin-cloud.net
```

Com isso, o tráfego suspeito está relacionado com o `ssin-cloud`

## Filtrando os SNIs TLS

Após abrir o wireshark, uso o filtro ```tls.handshake.extensions_server_name contains "ssin-cloud"``` para encontrar as requisições com ssin-cloud

<img width="1110" height="287" alt="image" src="https://github.com/user-attachments/assets/22181993-defe-4cfc-8e3d-05630252993f" />

Com esse filtro aparece várias conversas parecidas com:

```
10.8.6.20 -> 200.160.2.44  updates.ssin-cloud.com
10.8.6.21 -> 200.160.2.44  cdn.ssin-cloud.net
10.8.6.22 -> 200.160.2.44  updates.ssin-cloud.net
10.8.6.23 -> 200.160.2.44  static.ssin-cloud.net
10.8.6.23 -> 200.160.2.44  updates.ssin-cloud.net
```

Mas o mais suspeito é o `10.8.6.23 -> 200.160.2.44  updates.ssin-cloud.net`

## Seguir a conversa:

Agora vou seguir a conversa desse domínio, usando a opição: ```Follow > TCP Stream```, para saber melhor o seu fluxo de requisições

<img width="1252" height="831" alt="image" src="https://github.com/user-attachments/assets/b5463889-d81f-4ad6-945b-b75adc47ba79" />

Coletando as informações desses fluxos, consigo: 

```
name=nfe-signer.dylib
{"siem":"supply_chain_update","ja3":"suspicious","pkg":"nfe-signer.dylib","c2":"200.160.2.44","result":"benign"}
SURICATA[SUPPLY_CHAIN]: tls_update pkg=nfe-signer.dylib src=10.8.6.23 result=clean done=true
```

##### Explicação:
> nfe-signer.dylib -> Pacote/Arquivo baixado

> src=10.8.6.23 -> Quem baixou

> 200.160.2.44 e ja3="suspicious" -> Servidor suspeito

A conexão TLS foi marcada como suspeita. Mesmo aparecendo: 

```
result="benign"
result=clean
status=allowed
```

isso é provavelmente isca do desafio. O importante é que o próprio tráfego entrega o contexto:

```
supply_chain_update
pkg=nfe-signer.dylib
c2=200.160.2.44
```

Logo, temos a lógica do ataque:

> Arquivo suspeito: nfe-signer.dylib

> Vindo de: updates.ssin-cloud.net

> Falando com: 200.160.2.44

### Foramção da FLAG:

A flag tem esse formato: `HIK_supply-chain-update_HASH`

O hash é feito com os principais indicadores encontrados: 

```nfe-signer.dylib|updates.ssin-cloud.net|200.160.2.44```

Então vou transformar isso em um hash MD5, com o comando: 

```echo -n 'nfe-signer.dylib|updates.ssin-cloud.net|200.160.2.44' | md5sum```

<img width="616" height="67" alt="image" src="https://github.com/user-attachments/assets/c734997b-c664-4a5f-b276-e850c452535b" />

## FLAG: HIK_supply-chain-update_44948940e9d2d52119fc8593335b96ed
