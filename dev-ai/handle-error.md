# Diretriz de Registro de Erros

## Objetivo

Todo erro identificado durante o desenvolvimento deve ser documentado para criar uma base histórica de problemas e soluções.

A documentação deve ocorrer independentemente de o erro ter sido:

- Encontrado pelo agente durante análise de código;
- Encontrado durante compilação;
- Encontrado durante execução;
- Encontrado durante testes;
- Introduzido por alteração realizada pelo próprio agente;
- Inferido pelo agente como causa provável de falha;
- Relatado por usuários ou outros desenvolvedores.

---

# Local de Armazenamento

Todos os registros devem ser armazenados no diretório:

```text
docs/erros/
```

---

# Nomenclatura dos Arquivos

Os arquivos devem seguir o padrão:

```text
NUMERO-descricao_curta_do_erro.md
```

A descrição deve usar apenas letras minúsculas e algarismos, sem acentos ou espaços, com as palavras separadas por underscore (`_`).

Exemplos:

```text
1-erro_ao_salvar_registro_pessoa.md

2-falha_compilacao_servico_autenticacao.md

3-null_reference_ao_carregar_cliente.md

4-timeout_consulta_api_externa.md
```

---

# Numeração

Antes de criar um novo registro:

1. Verificar os arquivos existentes no diretório `docs/erros` (na branch `develop` atualizada).
2. Identificar o último número utilizado.
3. Criar o próximo número sequencial.
4. Nunca reutilizar números removidos.
5. Nunca alterar a numeração de registros já mesclados na `develop`.

**Conflitos entre branches:** como branches diferentes podem criar registros em paralelo, dois registros podem acabar recebendo o mesmo número. Ao atualizar a branch com a `develop` (ou ao abrir o Pull Request), quem estiver com a branch mais nova deve renumerar apenas o(s) próprio(s) arquivo(s) novo(s) para o próximo número livre. Nunca renumerar um registro que já esteja na `develop`.

Exemplo:

```text
1-erro_x.md
2-erro_y.md
3-erro_z.md
```

Próximo arquivo:

```text
4-novo_erro.md
```

---

# Estrutura Obrigatória do Documento

Todo documento de erro deve conter os metadados iniciais (Data e Status) e exatamente as seguintes três seções:

## 1. Problema

Descrever de forma objetiva o impacto do problema.

Exemplos:

- Erro de compilação.
- Erro durante execução.
- Falha ao salvar registro.
- Exceção não tratada.
- Timeout em integração.
- Dados inconsistentes.

---

## 2. Onde estava o problema

Descrever precisamente onde o problema foi identificado.

Sempre informar quando possível:

- Arquivo
- Classe
- Método
- Função
- Endpoint
- Procedure
- Query
- Componente de UI

Exemplo:

```text
Arquivo:
PessoaService.cs

Classe:
PessoaService

Método:
SalvarPessoa()

Linha aproximada:
145
```

---

## 3. Resolução

Preencher apenas quando o Status (ver template) for `Resolvido`. Um registro com Status `Em aberto` deve conter somente as seções 1 e 2, sendo atualizado depois — conforme a Regra 4 — quando a correção for de fato aplicada.

Descrever claramente a causa identificada e as ações executadas para correção.

A descrição deve permitir que outro desenvolvedor compreenda:

- O motivo do erro;
- Como ele foi identificado;
- O que foi alterado;
- Como evitar recorrência.

---

# Template Obrigatório

Utilizar o seguinte modelo para todos os registros:

```markdown
# [TÍTULO DO ERRO]

**Data:** DD/MM/AAAA  
**Status:** Resolvido | Em aberto

## 1. Problema

Descreva brevemente o que ocorreu e qual o impacto observado.

---

## 2. Onde estava o problema

**Arquivo:**  
Nome do arquivo

**Classe:**  
Nome da classe

**Método/Função:**  
Nome do método ou função

**Outros identificadores (se aplicável):**  
Endpoint, Procedure, Query ou Componente de UI envolvido

**Observações:**  
Informações adicionais relevantes.

---

## 3. Resolução

*(Preencher apenas se Status = Resolvido)*

### Causa Raiz

Descrição da causa identificada.

### Correção Aplicada

Descrição técnica da correção.

### Validação

Como foi validado que o problema foi resolvido.
```

---

# Exemplo

Arquivo:

```text
docs/erros/1-erro_ao_salvar_registro_pessoa.md
```

Conteúdo:

```markdown
# Erro ao salvar registro de pessoa

**Data:** 15/03/2026  
**Status:** Resolvido

## 1. Problema

Ao tentar salvar um novo cadastro de pessoa, a aplicação lançava uma exceção e impedia a gravação do registro.

---

## 2. Onde estava o problema

**Arquivo:**  
PessoaService.cs

**Classe:**  
PessoaService

**Método/Função:**  
SalvarPessoa()

**Outros identificadores (se aplicável):**  
Não se aplica.

**Observações:**  
A propriedade CPF era enviada nula para a camada de persistência.

---

## 3. Resolução

### Causa Raiz

A validação de CPF não era executada antes da chamada ao repositório.

### Correção Aplicada

Foi adicionada validação obrigatória do CPF antes da persistência dos dados.

### Validação

- Compilação executada com sucesso.
- Teste manual realizado.
- Cadastro concluído sem erros.
- Testes automatizados aprovados.
```

---

# Regras Adicionais para o Agente

1. Todo erro resolvido deve gerar documentação.
2. A documentação deve ser criada ou atualizada no mesmo commit em que o erro é identificado ou corrigido, conforme o caso.
3. Nenhum Pull Request deve ser finalizado sem registrar os erros encontrados durante o desenvolvimento (ver seção "Pull Requests" em `developing.md`).
4. Caso o erro já exista documentado, atualizar o registro existente adicionando novas informações — inclusive para mudar o Status de `Em aberto` para `Resolvido`.
5. Sempre registrar a causa raiz, não apenas o sintoma.
6. Sempre registrar a forma de validação da correção.
7. A documentação deve ser escrita em linguagem objetiva e técnica.
8. Evitar descrições genéricas como "erro corrigido" ou "ajustado código".
9. O histórico de erros constitui uma base de conhecimento do projeto e nunca deve ser removido sem justificativa formal.
10. Ao mudar o Status de um erro para `Resolvido`, atualizar também o arquivo de lições aprendidas (ver seção "Lições Aprendidas" abaixo).

---

# Lições Aprendidas

Além do registro individual de cada erro, mantenha um arquivo único e consolidado com orientações objetivas de "o que fazer" e "o que evitar", extraídas dos erros já resolvidos.

## Objetivo

Permitir consultar rapidamente, antes de escrever código, um resumo do que já causou problemas no projeto — sem precisar ler todo o histórico em `docs/erros/`.

## Local de Armazenamento

```text
docs/diretrizes/lessons-learned.md
```

## Quando Atualizar

Sempre que o Status de um registro de erro mudar para `Resolvido`:

1. Verificar se já existe uma orientação equivalente no arquivo de lições aprendidas.
2. Se existir, refinar a orientação existente com o que foi aprendido (nunca duplicar).
3. Se não existir, adicionar uma nova entrada.

Erros com Status `Em aberto` não geram entrada aqui — a causa raiz e a forma de evitar recorrência precisam estar confirmadas primeiro.

## Formato das Entradas

Organizar as entradas por categoria (ex.: Validação de Dados, Persistência/Banco de Dados, Programação Assíncrona, Configuração, Testes), para manter o arquivo navegável mesmo com o crescimento.

Cada entrada deve seguir o modelo:

```markdown
### [Categoria] Título curto da orientação

- **Fazer:** ação recomendada.
- **Evitar:** o que causou o erro.
- **Referência:** docs/erros/N-descricao_curta.md
```

Exemplo:

```markdown
### [Persistência] Validar CPF antes de persistir

- **Fazer:** validar o CPF antes de qualquer chamada ao repositório.
- **Evitar:** confiar que a camada de persistência vai rejeitar dados inválidos.
- **Referência:** docs/erros/1-erro_ao_salvar_registro_pessoa.md
```

## Diferença em Relação ao Histórico de Erros

Ao contrário do histórico em `docs/erros/` (Regra 9 — nunca remover sem justificativa formal), o arquivo de lições aprendidas deve ser mantido enxuto: entradas duplicadas devem ser mescladas, entradas obsoletas (ex.: referentes a uma biblioteca ou abordagem que o projeto não usa mais) podem ser removidas, e o texto pode ser reescrito para maior clareza. O histórico completo de cada caso continua disponível no registro de erro referenciado.
