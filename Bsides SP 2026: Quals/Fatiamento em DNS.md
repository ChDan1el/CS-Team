# Enunciado: Analise o tráfego DNS, identifique as consultas legítimas e encontre a flag!

#### [Baixar Desafio](https://github.com/user-attachments/files/27715487/files0.zip)

Após baixar o arquivo .pcap, uso o comando *strings* para encontrar palavras no arquivo

<img width="627" height="463" alt="image" src="https://github.com/user-attachments/assets/ec09ead2-ca25-472a-bc0d-ccafd0a5f73e" />

Com isso, percebo que há um pequeno padrão, depois das palavras "exfil attacker", aparentemente tem strings codificadas em base64

Então agora vou analisar diretamente no wireshark. Após abri-lo, uso o filtro `dns.qry.name contains "exfil.attacker"` para filtrar as requisições DNS com o nome "exil attacker".

<img width="1919" height="450" alt="Captura de tela 2026-05-13 114545" src="https://github.com/user-attachments/assets/289d2c7b-ddf8-40b5-a702-59f9ad08b419" />

OLha aí! Encontrei as requisições DNS, onde estão ordenadas de c01 a c08 com strings em base64. 

Então aplico outro filtro, `dns.qry.name contains "exfil.attacker" && dns.flags.response == 0`, para ter uma saída mais limpa

<img width="1919" height="302" alt="Captura de tela 2026-05-13 114702" src="https://github.com/user-attachments/assets/4e277fdc-961f-4151-afd6-29ac75115d20" />

Com isso copio os endereços listados: 
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

Excluindo os prefixos c01 a c08 e o `sufixo .exfil.attacker.com`, consigo uma strings codificada em base

> SElLX2Ruc190dW5uZWxfcGllY2VzXzFhOWQ0ZTViMmM3ZjZhOGQzZTBiMWMyZDRmNWE2Yjdj

## FLAG: HIK_dns_tunnel_pieces_1a9d4e5b2c7f6a8d3e0b1c2d4f5a6b7c
