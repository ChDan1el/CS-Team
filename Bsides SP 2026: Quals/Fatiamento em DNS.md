# Introdução

O desafio fornece um arquivo `.pcap` com tráfego DNS. O objetivo é analisar as consultas, separar o que é legítimo do que é suspeito e reconstruir a flag escondida nos domínios consultados.

## Enunciado: 
Analise o tráfego DNS, identifique as consultas legítimas e encontre a flag!

#### [Baixar Desafio](https://github.com/user-attachments/files/27715487/files0.zip)

## Analisando o arquivo

Após baixar e extrair o arquivo `.pcap`, usei o comando `strings` para procurar textos legíveis dentro da captura:

```strings dns-tunnel-pieces.pcap```

<img width="627" height="463" alt="image" src="https://github.com/user-attachments/assets/ec09ead2-ca25-472a-bc0d-ccafd0a5f73e" />

Ao observar a saída, percebi um padrão interessante: apareciam várias ocorrências relacionadas a `exfil.attacker`, junto com trechos que pareciam estar codificados em Base64.

Isso já é um forte indício de que os dados podem estar sendo enviados em partes por meio de consultas DNS.

### Analisando no Wireshark

Para investigar melhor, abri o arquivo .pcap no Wireshark.

Usei o seguinte filtro para exibir apenas consultas DNS relacionadas ao domínio suspeito:

```dns.qry.name contains "exfil.attacker"```

<img width="1919" height="450" alt="Captura de tela 2026-05-13 114545" src="https://github.com/user-attachments/assets/289d2c7b-ddf8-40b5-a702-59f9ad08b419" />

Com esse filtro, foi possível encontrar várias consultas DNS para subdomínios de `exfil.attacker.com`

Os domínios seguiam um padrão bem claro:

```
c01.<pedaço>.exfil.attacker.com
c02.<pedaço>.exfil.attacker.com
...
```

Os prefixos `c01`, `c02` etc. indicam a ordem correta dos pedaços.

### Filtrando apenas as queries DNS

Como o filtro anterior também mostrava respostas DNS, apliquei um filtro mais específico para exibir somente as consultas, ou seja, os pacotes `Standard query`:

```dns.qry.name contains "exfil.attacker" && dns.flags.response == 0```

<img width="1919" height="302" alt="Captura de tela 2026-05-13 114702" src="https://github.com/user-attachments/assets/4e277fdc-961f-4151-afd6-29ac75115d20" />

Agora a visualização ficou mais limpa, mostrando apenas os domínios que continham os pedaços da informação escondida.

### Extraindo os pedaços

As consultas encontradas foram:
```
c01.SElLX2Ruc1.exfil.attacker.com
c02.90dW5uZWxf.exfil.attacker.com
c03.cGllY2VzXz.exfil.attacker.com
c04.FhOWQ0ZTVi.exfil.attacker.com
c05.MmM3ZjZhOG.exfil.attacker.com
c06.QzZTBiMWMy.exfil.attacker.com
c07.ZDRmNWE2Yj.exfil.attacker.com
c08.dj.exfil.attacker.com
```

Removendo os prefixos `c01`, `c02`, ..., `c08` e o sufixo `.exfil.attacker.com`, ficam apenas os pedaços codificados:

```
SElLX2Ruc1
90dW5uZWxf
cGllY2VzXz
FhOWQ0ZTVi
MmM3ZjZhOG
QzZTBiMWMy
ZDRmNWE2Yj
dj
```

Juntando tudo na ordem indicada pelos prefixos:

> SElLX2Ruc190dW5uZWxfcGllY2VzXzFhOWQ0ZTViMmM3ZjZhOGQzZTBiMWMyZDRmNWE2Yjdj

### Decodificando a string

Para decodificar, usei o comando:

```echo 'SElLX2Ruc190dW5uZWxfcGllY2VzXzFhOWQ0ZTViMmM3ZjZhOGQzZTBiMWMyZDRmNWE2Yjdj' | base64 -d```

<img width="643" height="81" alt="Captura de tela 2026-05-13 115432" src="https://github.com/user-attachments/assets/dc6941ec-bf92-40ee-bf31-28e57afa8a1f" />

## FLAG: HIK_dns_tunnel_pieces_1a9d4e5b2c7f6a8d3e0b1c2d4f5a6b7c

### Conclusão

O desafio utilizava uma técnica simples de DNS tunneling, onde a flag foi dividida em pequenos pedaços e enviada dentro de subdomínios.

O ponto principal da resolução foi identificar o padrão nos domínios:

> cXX.<base64>.exfil.attacker.com

Depois disso, bastou ordenar os pedaços pelo prefixo c01 até c08, juntar as partes codificadas e decodificar o resultado em Base64.
