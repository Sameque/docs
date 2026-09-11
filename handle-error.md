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

1. Verificar os arquivos existentes no diretório `docs/erros`.
2. Identificar o último número utilizado.
3. Criar o próximo número sequencial.
4. Nunca reutilizar números removidos.
5. Nunca alterar a numeração de registros históricos.

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

Todo documento de erro deve conter exatamente as seguintes seções:

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

**Observações:**  
Informações adicionais relevantes.

---

## 3. Resolução

Descreva as ações executadas para corrigir o problema.

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
2. A documentação deve ser criada no mesmo commit da correção.
3. Nenhum Pull Request deve ser finalizado sem registrar erros encontrados durante o desenvolvimento.
4. Caso o erro já exista documentado, atualizar o registro existente adicionando novas informações.
5. Sempre registrar a causa raiz, não apenas o sintoma.
6. Sempre registrar a forma de validação da correção.
7. A documentação deve ser escrita em linguagem objetiva e técnica.
8. Evitar descrições genéricas como "erro corrigido" ou "ajustado código".
9. O histórico de erros constitui uma base de conhecimento do projeto e nunca deve ser removido sem justificativa formal.
