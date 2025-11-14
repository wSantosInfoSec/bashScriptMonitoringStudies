Este repositório contém um script Bash projetado para automatizar o processo de coleta, filtragem, anonimização e arquivamento de logs de uma aplicação fictícia, foi criado durante a formação de Linux da Alura, durante o ano de 2025, utilizando WSL 2 dentro do Windows 11.

## 🤔 O que o Script Faz?
**Preparação**: Cria os diretórios necessários (logs, logs-processados, logs-temp).

**Busca**: Encontra todos os arquivos .log no diretório de origem.

**Filtragem**: Para cada log, extrai apenas as linhas que contêm "ERROR" ou "SENSITIVE_DATA".

**Anonimização (Redact)**: Remove dados sensíveis (senhas, tokens, cartões) substituindo-os por "REDACTED".

**Limpeza:** Ordena (sort) e remove linhas duplicadas (uniq) do log filtrado.

**Estatísticas**: Gera um arquivo de estatísticas (log_stats_...txt) com a contagem de linhas e palavras de cada log processado.

**Consolidação**: Combina todos os logs limpos em um único arquivo (logs_combinados_...log), adicionando um prefixo ([FRONTEND], [BACKEND]) com base no nome do arquivo.

**Ordenação Final**: Ordena o log combinado pela segunda coluna.

**Arquivamento**: Move os arquivos processados para um diretório temporário e cria um arquivo .tar.gz (compactado) deles.

**Limpeza Final**: Remove o diretório temporário.


## 🎚️ Comandos, Conceitos e Opções Aprendidos

### Manipulação de Diretórios e Arquivos
mkdir -p [diretorio]
Comando: mkdir (criar diretório).
Opção -p: "Parents". Cria toda a estrutura de diretórios pai, se ela não existir, e não reclama se o diretório já existe.

### mv [origem] [destino]
Comando: mv (mover/renomear). Usado para mover os arquivos finais para o diretório temporário antes de compactar.

### rm -r [diretorio]
Comando: rm (remover).
Opção -r: "Recursivo". Necessário para remover um diretório e todo o seu conteúdo.

>[!WARNING]
> O comando rm -r "$TEMP_DIR" ao final do script é destrutivo e excluirá permanentemente o diretório temporário e todos os arquivos de log processados que ele contém. Certifique-se de que o arquivo .tar.gz foi criado corretamente antes de rodar o script em produção.

### cat [arquivo]
Comando: cat (concatenar). Ler o conteudo do arquivo

### find [caminho] -name "[padrão]" -print0
Comando: find (buscar arquivos).
Opção -name: Filtra a busca por nome. O "*" é um curinga (glob) que significa "qualquer coisa".
Opção -print0: Uma boa prática crucial. Faz o find separar os nomes dos arquivos com um caractere nulo (em vez de quebra de linha), o que previne erros caso um arquivo tenha espaços no nome.

### grep "[padrão]" [arquivo]
Comando: grep (buscar texto dentro de arquivos). Usado para extrair apenas as linhas de "ERROR" e "SENSITIVE_DATA".

### sed -i 's/busca/substitui/g' [arquivo]
Comando: sed (Stream Editor). Usado para fazer a anonimização (redact).
Opção -i: "In-place". Modifica o arquivo original diretamente, em vez de apenas imprimir o resultado no terminal.
Sintaxe s/ / /g: O comando de substituição. O g no final significa "global" (substitui todas as ocorrências na linha, não apenas a primeira).

### Regex .*
: (Expressão Regular) Significa "qualquer caractere (.) repetido zero ou mais vezes (*)" — usado para apagar tudo após o dado sensível.
Regex ^: (Expressão Regular) Significa "início da linha". Usado para adicionar os prefixos [FRONTEND] e [BACKEND].

>[!CAUTION]
>A opção -i no comando sed (edição "in-place") modifica o arquivo diretamente no disco. Se a sua expressão regular (s/.../.../g) estiver incorreta, você pode corromper ou apagar dados permanentemente do arquivo .filtrado sem um backup.

### sort -k2 [arquivo]
Comando: sort (ordenar linhas).
Opção -k2: "Key 2". Ordena o arquivo usando a segunda coluna (ou campo) como chave. Por padrão, campos são separados por espaços.
Opção -o [arquivo_saida]: "Output". Salva o resultado ordenado em um arquivo de saída (neste script, foi usado para sobrescrever o próprio arquivo).

### uniq [arquivo]
Comando: uniq (remover duplicatas). Remove apenas linhas duplicadas adjacentes.

>[!TIP]
> Ao usar uniq para remover linhas duplicadas, sempre execute sort antes (como feito no script). O comando uniq só consegue identificar e remover linhas duplicadas que estão adjacentes (uma logo após a outra).

### wc -w [arquivo] e wc -l [arquivo]
Comando: wc (Word Count).
Opção -w: Conta o número de words (palavras).
Opção -l: Conta o número de lines (linhas).

### basename [caminho_completo]
Comando: basename (nome base). Remove o caminho do diretório, deixando apenas o nome do arquivo (ex: de /logs/app.log para app.log).

## Compactação
tar -czf [arquivo.tar.gz] -C [diretorio_origem] .
Comando: tar (Tape Archive). Usado para agrupar e compactar arquivos.
Opão -c: cria um novo arquivo.
Opção -z: Compactar usando gzip (resultando em .tar.gz).
Opção -f: "File". Indica que o próximo argumento é o nome do file de saída.
Opção -C [dir]: "Change directory". Muda para o diretório especificado antes de adicionar os arquivos.
Argumento . (ponto): Significa "adicionar tudo que está no diretório atual" (que, graças ao -C, é o $TEMP_DIR).

## Estruturas e Conceitos do Shell Bash
Shebang (#!/bin/bash): A primeira linha, que diz ao sistema operacional para executar este arquivo usando o interpretador /bin/bash.

Variáveis: Definição e uso (ex: LOG_DIR="..." e $LOG_DIR).

Pipes (|): O "cano". Envia a saída padrão de um comando (esquerda) para ser a entrada padrão do próximo comando (direita). Ex: find ... | while ....

Redirecionamento de Saída (> e >>):

> (Sobrescrever): Cria um novo arquivo ou apaga o conteúdo de um existente.

>> (Anexar/Append): Adiciona o novo conteúdo ao final do arquivo, sem apagar o que já existe.

>[!IMPORTANT]
> No script, o primeiro grep (para "ERROR") usa o operador > (sobrescrever) para criar o arquivo .filtrado. Todos os comandos seguintes (grep "SENSITIVE_DATA") devem usar >> (anexar/append) para adicionar conteúdo sem apagar os resultados anteriores.

Redirecionamento de Entrada (<):

Usado com wc (ex: wc -w < "arquivo.unificado") para enviar o conteúdo do arquivo como entrada padrão, evitando que o wc imprima o nome do arquivo na saída.

Substituição de Comando ($(...)):

Executa o comando dentro dos parênteses e "cola" a saída dele naquele ponto.

Ex: num_linhas=$(wc -l < ...) captura a saída do wc e a armazena na variável num_linhas.

>[!NOTE]
>$(date +%F) é um exemplo de "substituição de comando". O shell executa o comando date +%F primeiro e, em seguida, "cola" o resultado (ex: 2025-11-14) no nome do arquivo. Isso é usado para criar logs com nomes dinâmicos.

## Loop while read:
processar a saída de outro comando (como find) linha por linha (ou, neste caso, item por item, graças ao -print0 e read -d '').
Condicionais (if [[ ... ]]; then ... fi):
Permite que o script tome decisões.
Teste [[ "$var" == *padrão* ]]: O == dentro de colchetes duplos [[ ... ]] permite o uso de globbing (o curinga *). Isso verifica se a variável contém a palavra "frontend" em qualquer lugar do nome.
