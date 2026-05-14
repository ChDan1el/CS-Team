## Introdução

Neste desafio, o objetivo era analisar uma estrutura de arquivos de e-mail no formato **Maildir**. Após descompactar os arquivos fornecidos, foi necessário navegar pelos diretórios da caixa de e-mail e inspecionar as mensagens armazenadas até encontrar a flag.

##### [Download do Desafio](https://github.com/user-attachments/files/27740800/files8.zip)

## 1) Descompactando os arquivos
Após baixar o arquivo `.zip` do desafio, comecei descompactando-o.

Ao extrair o `.zip`, encontrei outro arquivo compactado, desta vez no formato `.tar`. Depois de descompactar novamente esse arquivo, foi gerada uma pasta com os dados do desafio.

<img width="375" height="59" alt="image" src="https://github.com/user-attachments/assets/807b6776-b23b-4001-b47a-b8742461d000" />

Entrando na pasta extraída, encontrei três diretórios:

<img width="384" height="57" alt="image" src="https://github.com/user-attachments/assets/e94ce4ee-5767-4ebf-a7d6-6cf583f69083" />

Esses diretórios seguem a estrutura típica de um **Maildir**, que normalmente contém pastas como:

```text
cur
new
tmp
```

Nesse formato, os e-mails podem ficar armazenados como arquivos individuais dentro desses diretórios.

## Analisando o diretório `cur`

Comecei a investigação pelo diretório cur, onde geralmente ficam mensagens já entregues e processadas pela caixa de e-mail.

Dentro dele, encontrei dois arquivos. Ao abrir o primeiro, não identifiquei nenhuma informação relevante para o desafio.

Em seguida, analisei o segundo arquivo. Nele, encontrei diretamente uma linha contendo a flag:

<img width="609" height="410" alt="Captura de tela 2026-05-14 002603" src="https://github.com/user-attachments/assets/653ae188-e942-4eb1-9f36-6d986a5c2673" />

```text
flag=HIK_maildir-headerlog_5d28c7ae10f34b69dc14e37af9265081
```

## FLAG: HIK_maildir-headerlog_5d28c7ae10f34b69dc14e37af9265081

### Conclusão

A resolução do desafio consistiu em extrair os arquivos compactados e reconhecer a estrutura de diretórios no formato Maildir. Ao inspecionar os arquivos presentes no diretório cur, foi possível localizar uma mensagem que continha a flag diretamente em seu conteúdo. Assim, a análise dos arquivos de e-mail revelou a flag do desafio.
