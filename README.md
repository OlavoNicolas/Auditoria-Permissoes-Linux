# Auditoria de Permissões Linux

## 1. Visão geral

- **Categoria:** Linux Security · Authorization · Access Control
- **Ambiente:** Ubuntu Linux — Bash Shell
- **Usuário analisado:** `researcher2`
- **Grupo associado:** `research_team`
- **Diretório auditado:** `/home/researcher2/projects`

### Objetivo

Analisar e ajustar as permissões de arquivos e diretórios em um ambiente Linux, aplicando o **Princípio do Menor Privilégio (Principle of Least Privilege)**.

A atividade consiste em auditar o diretório de uma equipe de pesquisa, identificar permissões excessivas ou indevidas e aplicar as correções necessárias para restringir acessos não autorizados.

Para isso, são utilizados comandos básicos da linha de comando para inspecionar permissões, identificar arquivos ocultos e modificar os privilégios de acesso de usuários, grupos e outros usuários do sistema.

---

## 2. Contexto de segurança

A autorização determina quais recursos um usuário pode acessar e quais operações pode executar sobre eles. No Linux, as permissões tradicionais permitem definir privilégios distintos para três categorias:

- **User (`u`):** proprietário do arquivo ou diretório.
- **Group (`g`):** usuários pertencentes ao grupo associado ao recurso.
- **Others (`o`):** demais usuários do sistema que não se enquadram nas categorias anteriores.

Permissões excessivas podem permitir a leitura ou modificação indevida de informações, comprometendo a confidencialidade e a integridade dos dados.

Neste cenário, o usuário `researcher2`, integrante do grupo `research_team`, é responsável pelos recursos do diretório `projects`. A atividade consiste em verificar se as permissões existentes respeitam os requisitos de acesso estabelecidos e corrigir as configurações incompatíveis.

---

## 3. Fundamentos das permissões Linux

O comando `ls -l` apresenta informações sobre os arquivos e diretórios, incluindo uma sequência de dez caracteres que representa o tipo do recurso e suas permissões.

### 3.1. Permissões básicas

| Permissão | Significado | Arquivos | Diretórios |
|---|---|---|---|
| `r` | Read | Permite ler o conteúdo | Permite listar os nomes, conforme as demais permissões |
| `w` | Write | Permite modificar o conteúdo | Permite criar, remover ou renomear entradas, conforme as demais restrições |
| `x` | Execute | Permite executar o arquivo, quando aplicável | Permite atravessar o diretório e acessar recursos por seu caminho |

Essas permissões são definidas separadamente para o proprietário, o grupo e os outros usuários.

### 3.2. Interpretando uma string de permissões

Exemplo:

```text
drwxr-xr--
```

A sequência pode ser dividida em quatro partes:

| Posições | Valor | Descrição |
|---|---|---|
| 1º caractere | `d` | Indica que o recurso é um diretório |
| 2º ao 4º | `rwx` | O proprietário pode ler, escrever e atravessar o diretório |
| 5º ao 7º | `r-x` | O grupo pode ler e atravessar o diretório, mas não escrever |
| 8º ao 10º | `r--` | Outros usuários podem ler, mas não escrever nem atravessar o diretório |

O primeiro caractere identifica o tipo de recurso: `d` representa um diretório, `-` representa um arquivo comum e `l` representa um link simbólico.

**Observação:** a interpretação das permissões depende do tipo de recurso. Em diretórios, a permissão `x` está relacionada à travessia e ao acesso aos conteúdos, e não à execução de um programa.

---

## 4. Inspeção inicial do ambiente

O primeiro estágio consistiu em navegar até o diretório de trabalho e examinar os arquivos, diretórios, proprietários, grupos e permissões existentes.

Comandos utilizados:

```bash
cd /home/researcher2/projects
ls -l
ls -la
```

O comando `ls -l` permite inspecionar os recursos visíveis, enquanto `ls -la` também apresenta arquivos e diretórios ocultos.

### 4.1. Arquivos identificados

As permissões iniciais dos arquivos eram:

| Recurso | Proprietário | Grupo | Outros | Permissão inicial |
|---|---|---|---|---|
| `project_k.txt` | Leitura e escrita | Leitura e escrita | Leitura e escrita | `-rw-rw-rw-` (666) |
| `project_m.txt` | Leitura e escrita | Leitura | Nenhuma | `-rw-r-----` (640) |
| `project_r.txt` | Leitura e escrita | Leitura e escrita | Leitura | `-rw-rw-r--` (664) |
| `project_t.txt` | Leitura e escrita | Leitura e escrita | Leitura | `-rw-rw-r--` (664) |
| `.project_x.txt` | Leitura e escrita | Escrita | Nenhuma | `-rw--w----` (620) |

Além dos arquivos, foi identificado o subdiretório `drafts`, com as seguintes permissões iniciais:

| Proprietário | Grupo | Outros | Permissão inicial |
|---|---|---|---|
| Leitura, escrita e execução | Execução | Nenhuma | `drwx--x---` (710) |

---

## 5. Requisitos de autorização e análise

A análise foi conduzida conforme os requisitos definidos no cenário do laboratório.

### 5.1. Arquivo `project_k.txt`

**Constatação:** o arquivo permitia que outros usuários lessem e escrevessem em seu conteúdo.

**Requisito:** outros usuários não devem possuir permissão de escrita nos arquivos do diretório.

**Risco identificado:** a permissão de escrita concedida a outros usuários possibilitava alterações indevidas no conteúdo do arquivo.

**Ação executada:**

```bash
chmod o-w project_k.txt
```

**Resultado esperado:** remoção da permissão de escrita para outros usuários, mantendo as demais permissões existentes.

**Permissão final esperada:** `-rw-rw-r--` (664).

### 5.2. Arquivo `project_m.txt`

**Constatação:** o arquivo concedia permissão de leitura ao grupo associado.

**Requisito:** somente o proprietário deve possuir permissões de leitura e escrita nesse arquivo.

**Risco identificado:** a leitura concedida ao grupo era incompatível com a política de acesso restrito definida para o recurso.

**Ação executada:**

```bash
chmod g-rw project_m.txt
```

**Resultado esperado:** remoção das permissões de leitura e escrita do grupo. Outros usuários já não possuíam permissões concedidas nesse arquivo.

**Permissão final esperada:** `-rw-------` (600).

### 5.3. Arquivos `project_r.txt` e `project_t.txt`

**Constatação:** ambos permitiam leitura pelo grupo e por outros usuários, mas não permitiam escrita por outros usuários.

**Requisito:** verificar as permissões existentes e identificar eventuais permissões de escrita indevidas para outros usuários.

**Ação executada:** inspeção das permissões, sem alteração.

**Resultado:** ambos foram mantidos com as permissões originais.

**Permissão final esperada:** `-rw-rw-r--` (664) para os dois arquivos.

A manutenção dessas permissões é compatível com o requisito específico de não permitir escrita por outros usuários. A adequação da leitura concedida ao grupo e aos demais usuários depende da política de autorização aplicável a cada arquivo.

### 5.4. Arquivo oculto `.project_x.txt`

**Constatação:** o arquivo arquivado concedia permissão de escrita ao proprietário e ao grupo.

**Requisito:** o arquivo deve continuar legível pelo proprietário e pelo grupo, mas não pode ser escrito por nenhum deles.

**Risco identificado:** a permissão de escrita permitia modificar um arquivo que deveria permanecer protegido contra alterações.

**Ação executada:**

```bash
chmod ug-w .project_x.txt
```

**Resultado esperado:** remoção da permissão de escrita do proprietário e do grupo, mantendo suas permissões de leitura. Outros usuários continuam sem permissões concedidas.

**Permissão final esperada:** `-r--r-----` (440).

### 5.5. Diretório `drafts/`

**Constatação:** o diretório concedia ao grupo a permissão de execução, necessária para atravessá-lo e acessar seus conteúdos.

**Requisito:** somente o usuário `researcher2` deve possuir permissão de execução no diretório.

**Risco identificado:** a permissão de travessia concedida ao grupo não estava alinhada à restrição de acesso estabelecida.

**Ação executada:**

```bash
chmod g-x drafts
```

**Resultado esperado:** remoção da permissão de execução do grupo, mantendo as demais permissões existentes.

**Permissão final esperada:** `drwx------` (700).

> **Observação:** as permissões finais apresentadas são os resultados esperados após a aplicação dos comandos, considerando o estado inicial informado. A confirmação do estado efetivo depende de uma nova inspeção do diretório.

---

## 6. Resumo das alterações

| Recurso | Ação aplicada | Permissão inicial | Permissão final esperada |
|---|---|---|---|
| `project_k.txt` | `chmod o-w` | 666 | 664 |
| `project_m.txt` | `chmod g-rw` | 640 | 600 |
| `project_r.txt` | Nenhuma alteração | 664 | 664 |
| `project_t.txt` | Nenhuma alteração | 664 | 664 |
| `.project_x.txt` | `chmod ug-w` | 620 | 440 |
| `drafts/` | `chmod g-x` | 710 | 700 |

Os ajustes foram realizados com permissões simbólicas, removendo apenas os privilégios incompatíveis com os requisitos definidos no cenário.

---

## 7. Comandos e conceitos aplicados

Durante o laboratório, foram utilizados os seguintes comandos e conceitos:

| Comando | Finalidade |
|---|---|
| `cd` | Navegação entre diretórios |
| `ls -l` | Inspeção de arquivos, diretórios e permissões |
| `ls -la` | Inspeção que inclui recursos ocultos |
| `chmod o-w` | Remoção da escrita concedida a outros usuários |
| `chmod g-rw` | Remoção das permissões de leitura e escrita do grupo |
| `chmod ug-w` | Remoção da escrita do proprietário e do grupo |
| `chmod g-x` | Remoção da permissão de execução/travessia do grupo |

Os comandos demonstram a aplicação de permissões simbólicas para adequar os acessos aos requisitos definidos no cenário.

---

## 8. Conclusão

A atividade envolveu três etapas principais:

- **Mapeamento e auditoria:** identificação de proprietários, grupos e permissões de leitura, escrita e execução com `ls -l` e `ls -la`.
- **Identificação de riscos:** análise de permissões excessivas que poderiam permitir leitura ou alteração indevida de arquivos.
- **Ajuste e remediação:** utilização de `chmod` para remover permissões incompatíveis com os requisitos de acesso estabelecidos.

O principal aprendizado foi compreender como permissões aparentemente simples podem influenciar a confidencialidade e a integridade dos dados, além de praticar a aplicação do princípio do menor privilégio em um ambiente Linux.

Esses fundamentos contribuem para a administração segura de sistemas e podem apoiar atividades mais amplas de auditoria e segurança da informação. Entretanto, este laboratório isolado não comprova conformidade com a LGPD nem previne, por si só, movimentação lateral em incidentes de segurança.

### Próximos passos

- Aprofundar o uso de **ACLs (Access Control Lists)** para controle de acesso mais granular.
- Desenvolver scripts em Python para identificar arquivos e diretórios com permissões potencialmente excessivas, como `777` e `666`.
- Expandir as práticas de auditoria e validação de permissões em ambientes Linux.

---

**Tecnologias e conceitos:** Linux · Ubuntu · Bash · `chmod` · `ls` · File Permissions · Access Control · Least Privilege
