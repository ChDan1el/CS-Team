## Introdução

Neste desafio, o objetivo era analisar um arquivo .pcap e identificar um possível ataque de supply chain, no qual um atacante utiliza uma atualização aparentemente legítima para entregar um arquivo malicioso ou suspeito. A partir da análise do tráfego de rede, foi necessário encontrar os principais indicadores do ataque e, com eles, montar a flag.

## Contexto:
Um ataque de supply chain acontece quando o atacante compromete algo que parece confiável, como uma atualização de software, um pacote, uma dependência ou um servidor de distribuição.

Em vez de atacar diretamente a vítima, ele entrega um “update” falso ou adulterado, fazendo com que a própria vítima baixe o conteúdo comprometido acreditando ser legítimo.

##### [Download do Desafio](https://github.com/user-attachments/files/27737641/supply-chain-update.zip)

### 1) Analisando o Arquivo

Após baixar o arquivo `.pcap`, comecei utilizando o comando `strings` para procurar palavras legíveis dentro do tráfego capturado

```bash
strings -a supply-chain-update.pcap
```

<img width="360" height="438" alt="image" src="https://github.com/user-attachments/assets/57bdc2cc-3336-494c-a793-49d2b7585ac8" />

Durante essa análise, encontrei alguns domínios aparentemente legítimos:

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

Porém, também apareceram domínios suspeitos relacionados a `ssin-cloud`:

```
updates.ssin-cloud.com
cdn.ssin-cloud.net
updates.ssin-cloud.net
static.ssin-cloud.net
```

Com isso, ficou claro que o tráfego suspeito provavelmente estava relacionado aos domínios `ssin-cloud`

### 2) Filtrando os SNIs TLS

Depois disso, abri o arquivo no Wireshark e utilizei o seguinte filtro para encontrar conexões TLS que continham `ssin-cloud` no campo SNI:

```bash
tls.handshake.extensions_server_name contains "ssin-cloud"
```

<img width="1110" height="287" alt="image" src="https://github.com/user-attachments/assets/22181993-defe-4cfc-8e3d-05630252993f" />

Com esse filtro, apareceram várias conexões parecidas com estas:

```
10.8.6.20 -> 200.160.2.44  updates.ssin-cloud.com
10.8.6.21 -> 200.160.2.44  cdn.ssin-cloud.net
10.8.6.22 -> 200.160.2.44  updates.ssin-cloud.net
10.8.6.23 -> 200.160.2.44  static.ssin-cloud.net
10.8.6.23 -> 200.160.2.44  updates.ssin-cloud.net
```

Entre elas, a conexão mais suspeita foi: ```10.8.6.23 -> 200.160.2.44  updates.ssin-cloud.net```

Isso chamou atenção porque o domínio parecia simular um servidor de atualização, o que combina diretamente com o contexto de ataque de supply chain.

### 3) Seguindo a conversa TCP: 

Para entender melhor o fluxo dessa comunicação, selecionei a conexão suspeita no Wireshark e usei a opção: ```Follow > TCP Stream```

<img width="1252" height="831" alt="image" src="https://github.com/user-attachments/assets/b5463889-d81f-4ad6-945b-b75adc47ba79" />

Ao analisar os dados encontrados nos fluxos, consegui identificar as seguintes informações:

```
name=nfe-signer.dylib
{"siem":"supply_chain_update","ja3":"suspicious","pkg":"nfe-signer.dylib","c2":"200.160.2.44","result":"benign"}
SURICATA[SUPPLY_CHAIN]: tls_update pkg=nfe-signer.dylib src=10.8.6.23 result=clean done=true
```

##### Explicação:
> nfe-signer.dylib -> Pacote/Arquivo baixado

> src=10.8.6.23 -> Quem baixou

> 200.160.2.44 -> Servidor externo envolvido na comunicação

> ja3="suspicious" -> Tráfego TLS marcado como suspeito com base no fingerprint JA3

A conexão TLS foi marcada como suspeita. Mesmo aparecendo: 

```
result="benign"
result=clean
status=allowed
```

Isso é provavelmente isca do desafio. O importante é que o próprio tráfego entrega o contexto:

```
supply_chain_update
pkg=nfe-signer.dylib
c2=200.160.2.44
```

Com isso, os principais indicadores encontrados foram:

> Arquivo suspeito: nfe-signer.dylib

> Vindo de: updates.ssin-cloud.net

> Falando com: 200.160.2.44

### 4) Formação da FLAG:

A flag segue o seguinte formato: `HIK_supply-chain-update_HASH`

O hash é feito com os principais indicadores encontrados, separados por `|`: 

```nfe-signer.dylib|updates.ssin-cloud.net|200.160.2.44```

Para gerar o hash MD5, usei o comando:

```bash
echo -n 'nfe-signer.dylib|updates.ssin-cloud.net|200.160.2.44' | md5sum
```
> Resultado: 44948940e9d2d52119fc8593335b96ed

<img width="616" height="67" alt="image" src="https://github.com/user-attachments/assets/c734997b-c664-4a5f-b276-e850c452535b" />

## FLAG: HIK_supply-chain-update_44948940e9d2d52119fc8593335b96ed

### Conclusão

Neste desafio, a análise começou com a busca por strings dentro do `.pcap`, o que revelou domínios suspeitos relacionados a `ssin-cloud`. Em seguida, filtrando os SNIs TLS no Wireshark, foi possível identificar uma comunicação suspeita entre a máquina `10.8.6.23` e o servidor `200.160.2.44`.

Ao seguir o fluxo TCP, foram encontrados os principais indicadores do ataque: o pacote `nfe-signer.dylib`, o domínio `updates.ssin-cloud.net` e o IP externo `200.160.2.44`. Esses elementos confirmaram o cenário de supply chain update e foram usados para gerar o hash MD5 que compõe a flag final.
