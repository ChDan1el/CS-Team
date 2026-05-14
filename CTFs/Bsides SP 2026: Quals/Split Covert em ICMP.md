# Introdução

O desafio fornece um arquivo `.pcap` contendo tráfego ICMP. O objetivo é analisar os pacotes, identificar dados escondidos no tráfego e reconstruir a flag a partir das partes encontradas.

##### [Download do Desafio](https://github.com/user-attachments/files/27772552/files3.zip)

## 1) Analisando o arquivo

Primeiramente, utilizei o comando strings para procurar textos legíveis dentro do arquivo .pcap:

```bash
strings icmp-covert-split.pcap
```

<img width="549" height="488" alt="image" src="https://github.com/user-attachments/assets/8c04f6ec-941b-4928-9027-8d4a10a08ea2" />

Ao analisar a saída, encontrei alguns registros contendo o campo stream. Cada registro armazenava uma parte de uma string codificada em Base64.

Depois de organizar os registros pela ordem indicada no campo part, obtive o seguinte:

```text
part=01|host=wkst-07|stream=SElLX2ljbX
part=02|host=wkst-07|stream=AtY292ZXJ0
part=03|host=wkst-07|stream=LXNwbGl0Xz
part=04|host=wkst-07|stream=BmM2M2YTlk
part=05|host=wkst-07|stream=NDFiMjVlNz
part=06|host=wkst-07|stream=hhYzRkMWZl
part=07|host=wkst-07|stream=NjkzNzBiMj
part=08|host=wkst-07|stream=U0
```

### 2) Analisando com o Wireshark

Também é possível confirmar essas informações diretamente pelo Wireshark.

Para isso, filtrei os pacotes ICMP que continham a palavra `part`:

```
icmp contains "part"
```

<img width="1919" height="784" alt="image" src="https://github.com/user-attachments/assets/bf4aff6e-9601-4e46-9b5d-166549d06950" />

Com esse filtro, foi possível visualizar os pacotes que carregavam as partes da string codificada. Isso confirma que os dados estavam sendo transmitidos de forma dividida dentro do tráfego ICMP, caracterizando um possível canal encoberto, também conhecido como `covert channel`.

### 3) Reconstruindo a string em Base64

Após identificar as partes, removi os prefixos e mantive apenas o conteúdo do campo `stream`:

```text
SElLX2ljbX
AtY292ZXJ0
LXNwbGl0Xz
BmM2M2YTlk
NDFiMjVlNz
hhYzRkMWZl
NjkzNzBiMj
U0
```

Em seguida, juntei todos os trechos em uma única linha:

>SElLX2ljbXAtY292ZXJ0LXNwbGl0XzBmM2M2YTlkNDFiMjVlNzhhYzRkMWZlNjkzNzBiMjU0

### 4) Decodificando a flag

Como a string estava em Base64, utilizei o comando abaixo para decodificá-la:

```bash
echo 'SElLX2ljbXAtY292ZXJ0LXNwbGl0XzBmM2M2YTlkNDFiMjVlNzhhYzRkMWZlNjkzNzBiMjU0' | base64 -d
```

<img width="630" height="76" alt="image" src="https://github.com/user-attachments/assets/f8985585-2c80-4f42-9172-6326abebf2b3" />

## FLAG: HIK_icmp-covert-split_0f3c6a9d41b25e78ac4d1fe69370b254

### Conclusão

Neste desafio, a flag estava dividida em várias partes dentro de pacotes ICMP. A análise inicial com `strings` revelou os trechos codificados em Base64, e o Wireshark confirmou que esses dados estavam presentes no tráfego ICMP. Após ordenar corretamente as partes, juntar os valores do campo `stream` e decodificar a string em Base64, foi possível recuperar a flag com sucesso.
