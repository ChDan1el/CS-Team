# Introdução

Neste desafio, o objetivo era analisar arquivos relacionados ao histórico e ao cache de um navegador para encontrar uma informação escondida. A flag estava armazenada dentro de um dos arquivos de cache, codificada em Base64.

##### [Download do Desafio](https://github.com/user-attachments/files/27739817/files6.zip)

## 1) Extraindo o arquivo ZIP

Após baixar o desafio, extraí o arquivo `.zip`.

<img width="298" height="110" alt="image" src="https://github.com/user-attachments/assets/acfbaded-a6ee-4ef5-9f13-bd0078d5d5d3" />

Depois da extração, foram encontrados os seguintes arquivos:

- um banco SQLite chamado `History`;
- um arquivo `Cache_Index.json`;
- dois arquivos de cache.

Ao abrir o arquivo `Cache_Index.json`, percebi que ele fazia referência aos dois arquivos de cache presentes no desafio.

<img width="523" height="219" alt="image" src="https://github.com/user-attachments/assets/d63301f4-46cc-46c7-bd35-6a4422c4be40" />

Isso indicava que os arquivos de cache deveriam ser analisados com mais atenção, pois provavelmente continham o conteúdo necessário para chegar à flag.

### 2) Examinando os caches

Comecei analisando o primeiro arquivo de cache. Nele, encontrei apenas um comando simples de `print` em JavaScript, sem nenhuma informação relevante para a flag.

<img width="236" height="39" alt="image" src="https://github.com/user-attachments/assets/ff729585-b31b-49dc-8b80-59b1d4679314" />

Em seguida, analisei o segundo arquivo de cache. Nele, encontrei uma variável chamada `archivedBlob`, contendo uma string codificada em Base64:

> SElLX2hpc3RvcnktY2FjaGUtaWR4XzdlMTNjNGFkMjBmOTViNjhhYzE0ZGUzN2ZiOTI2NTAx

<img width="623" height="88" alt="image" src="https://github.com/user-attachments/assets/dd3ff7e5-cf2c-46b3-9ed5-58a70eb75e2c" />

Como a string parecia estar em Base64, usei o seguinte comando para decodificá-la, achando a flag:
```bash
echo 'SElLX2hpc3RvcnktY2FjaGUtaWR4XzdlMTNjNGFkMjBmOTViNjhhYzE0ZGUzN2ZiOTI2NTAx' | base64 -d
```

<img width="625" height="48" alt="image" src="https://github.com/user-attachments/assets/12260684-5447-4a9d-98bd-73dd9001a5f4" />

## FLAG: HIK_history-cache-idx_7e13c4ad20f95b68ac14de37fb926501

### Conclusão
A resolução do desafio consistiu em analisar os arquivos extraídos do pacote, identificar a relação entre o `Cache_Index.json` e os arquivos de cache, e inspecionar o conteúdo armazenado neles. O primeiro cache não continha informações úteis, mas o segundo possuía uma variável com uma string em Base64. Após decodificá-la, a flag foi revelada com sucesso.
