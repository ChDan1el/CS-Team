# Introdução

Neste desafio, foi fornecido um arquivo `.pcap` contendo tráfego de rede. O objetivo era analisar a captura, identificar uma transferência FTP e recuperar o arquivo baixado durante a comunicação para encontrar a flag.

##### [Download do Desafio](https://github.com/user-attachments/files/27812843/files1.zip)

## 1) Analisando o Arquivo

Após baixar o arquivo, uso o seguinte comando para ler as principais strings contidas nele:

```bash
strings ftp-passive-cache.pcap
```

Principais informações da saída:

```text
220 ftp-cache.internal FTP service ready
USER backupsvc
PASS N0tTheRightOne
530 Login incorrect.
USER monitor
PASS monitor
530 Login incorrect.
USER backupsvc
PASS C@ririCache2026!
230 Login successful.
PASV
227 Entering Passive Mode (10,60,4,10,7,233)
RETR cache_bundle.zip
150 Opening BINARY mode data connection for cache_bundle.zip
226 Transfer complete.
```

A partir dessas informações, percebo que houve uma conexão FTP e que o arquivo `cache_bundle.zip` foi baixado com sucesso.

### 2) Entendendo o FTP passivo

No tráfego FTP, encontro o comando:

```text
PASV
```

Esse comando indica que o cliente solicitou uma conexão em modo passivo. Logo em seguida, o servidor responde:

```
227 Entering Passive Mode (10,60,4,10,7,233)
```

No FTP passivo, os dois últimos números indicam a porta usada para a conexão de dados.

A porta é calculada assim:

```text
porta = 7 * 256 + 233
porta = 1792 + 233
porta = 2025
```

Portanto, o arquivo foi transferido pela porta TCP 2025.

### 3) Analisando pelo Wireshark

No Wireshark, uso o seguinte filtro para encontrar os pacotes relacionados à transferência do arquivo:

```
tcp.port == 2025
```

<img width="1238" height="305" alt="image" src="https://github.com/user-attachments/assets/3e232371-74af-4252-9443-13d4af92e39a" />

Entre os pacotes filtrados, é possível ver dados sendo enviados do servidor para o cliente, como:

```text
2025 → 51301 [ACK] Len=120
2025 → 51301 [ACK] Len=160
2025 → 51301 [PSH, ACK] Len=92
```

Isso confirma que houve transferência de conteúdo nessa conexão.

### 4) Extraindo o Arquivo

Para recuperar o arquivo transferido, clico em um dos pacotes da conexão e seleciono a opção: `Follow > TCP Stream`

Depois, altero a opção: `Show data as: ASCII` para `Raw`

Faço isso para conseguir salvar o conteúdo como um arquivo ZIP sem corrompê-lo.

Em seguida, salvo o conteúdo como: `cache_bundle.zip`

<img width="1324" height="866" alt="image" src="https://github.com/user-attachments/assets/fdbc60e2-25ed-453e-a79c-b6add3f7dd2a" />

Depois de salvar o arquivo, faço a descompactação e recebo alguns diretórios relacionados a cache:

<img width="519" height="240" alt="image" src="https://github.com/user-attachments/assets/56b9838d-1840-4974-9daa-dd598165263b" />

### 5) Lendo a flag

Ao analisar os arquivos extraídos, encontro um caminho que contém o nome *flag*. Então leio o arquivo com o comando:

```bash
cat cache/.state/.flag.note
```

<img width="488" height="71" alt="image" src="https://github.com/user-attachments/assets/e5cc63c4-5652-42a1-8bbd-1ee4ce1a3ddb" />

Com isso, encontro a flag do desafio:

## FLAG: HIK_ftp-passive-cache_1e48f3c20a6d59b7ce14af9283bd4061

## Conclusão

O desafio consistia em analisar uma captura de tráfego FTP e recuperar um arquivo transferido durante a comunicação. A parte principal foi identificar que o FTP estava usando modo passivo, calcular a porta da conexão de dados a partir da resposta `227 Entering Passive Mode` e filtrar essa porta no Wireshark.

Após seguir o fluxo TCP da porta 2025, consegui salvar o conteúdo transferido como um arquivo ZIP. Dentro dele havia um arquivo oculto chamado `.flag.note`, que continha a flag final.
