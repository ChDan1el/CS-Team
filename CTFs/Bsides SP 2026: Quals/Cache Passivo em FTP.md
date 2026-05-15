##### [Download do Desafio](https://github.com/user-attachments/files/27812843/files1.zip)

## 1) Analisando o Arquivo

Após baixar o arquivo, uso o seguinte comando para poder ler as principais strings contidas nele

```bash
strings ftp-passive-cache.pcap
```
Principais informções da saída:

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

A partir dessas informações, percebo que houve uma conexão FTP e que o arquivo cache_bundle.zip foi baixado com sucesso.

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

Usando o seguinte filtro, consigo encontrar as requisiçoes que passam o arquivo baixado

```
tcp.port == 2025
```

<img width="1238" height="305" alt="image" src="https://github.com/user-attachments/assets/3e232371-74af-4252-9443-13d4af92e39a" />

Entre eles, é possível ver pacotes com dados sendo enviados do servidor para o cliente, como:

```text
2025 → 51301 [ACK] Len=120
2025 → 51301 [ACK] Len=160
2025 → 51301 [PSH, ACK] Len=92
```

Isso confirma que houve transferência de conteúdo nessa conexão.

### 4) Extraindo o Arquivo

Para recuperar o arquivo, seleciono a opção `Follow > TCP Stream` e depois mudo a opção `Show data as: ASCII` para `Raw`, para eu poder baixar o arquivo em zip sem corrompe-lo.

Salvando como `cache_bundle.zip`

<img width="1324" height="866" alt="image" src="https://github.com/user-attachments/assets/fdbc60e2-25ed-453e-a79c-b6add3f7dd2a" />

Em seguida, descompacto o arquivo, recebendo diretorios de cache

<img width="519" height="240" alt="image" src="https://github.com/user-attachments/assets/56b9838d-1840-4974-9daa-dd598165263b" />

### 5) Lendo a flag

Ao ler o caminho que contem o nome *flag*, recebo a flag do desafio


<img width="488" height="71" alt="image" src="https://github.com/user-attachments/assets/e5cc63c4-5652-42a1-8bbd-1ee4ce1a3ddb" />




