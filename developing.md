# Diretrizes de Desenvolvimento e Gestão de Código

## Objetivo

Garantir padronização, rastreabilidade, qualidade e segurança no ciclo de desenvolvimento de software.

---

# Fluxo de Trabalho

## 1. Antes de iniciar qualquer desenvolvimento

### Atualizar a branch develop

Sempre sincronize sua cópia local com a branch `develop`.

Exemplo:

```bash
git checkout develop
git pull origin develop
```

### Criar uma branch de desenvolvimento

Crie uma nova branch a partir da branch `develop`.

Padrão de nomenclatura:

```text
feature/NOME_DA_FUNCIONALIDADE
bugfix/NOME_DO_BUG
hotfix/NOME_DO_HOTFIX
refactor/NOME_DO_REFACTOR
```

Exemplo:

```bash
git checkout -b feature/cadastro-clientes
```

### Publicar a branch

Publique imediatamente a branch remota para garantir backup e rastreabilidade.

```bash
git push -u origin feature/cadastro-clientes
```

---

# Desenvolvimento

## 2. Boas práticas de implementação

### Sempre

- Desenvolver uma única responsabilidade por branch.
- Criar código simples e de fácil manutenção.
- Seguir os padrões de arquitetura definidos pelo projeto.
- Respeitar convenções de nomenclatura existentes.
- Evitar código duplicado.
- Reutilizar componentes, funções e serviços existentes sempre que possível.
- Manter métodos pequenos e objetivos.
- Escrever código legível antes de otimizar.

### Nunca

- Alterar código sem entender seu impacto.
- Comentar código morto.
- Fazer commit de arquivos temporários.
- Fazer commit de credenciais, tokens ou senhas.
- Adicionar dependências sem justificativa técnica.

---

# Gestão de Commits

## 3. Commits

Os commits devem ser pequenos, atômicos e descritivos.
Não coloque e-mail ou qualquer marcação da LLM, Agente ou Ferramenta de AI.

### Padrão recomendado

```text
feat: adiciona cadastro de clientes

fix: corrige validação de CPF

refactor: simplifica regra de cálculo

test: adiciona testes da API de autenticação

docs: atualiza documentação de instalação
```

### Evitar

```text
ajustes
correções
update
teste
commit final
```

---

# Testes

## 4. Antes de finalizar o desenvolvimento

Executar:

- Testes unitários
- Testes de integração
- Testes de regressão relevantes
- Análise estática de código
- Linters do projeto

Checklist:

- [ ] Build executa com sucesso
- [ ] Testes executam com sucesso
- [ ] Não existem warnings críticos
- [ ] Não existem vulnerabilidades conhecidas nas dependências

---

# Revisão de Código

## 5. Antes de abrir Pull Request

Realize uma auto revisão:

### Verificar

- Código duplicado
- Possíveis bugs
- Tratamento de exceções
- Logs adequados
- Nomes claros para métodos e variáveis
- Remoção de código não utilizado
- Impactos de performance

### Perguntas obrigatórias

- O código resolve o problema solicitado?
- Existe uma solução mais simples?
- Os testes cobrem os cenários principais?
- A alteração pode gerar regressão?

---

# Pull Requests

## 6. Criação do Pull Request

O Pull Request deve conter:

### Título

```text
feat: cadastro de clientes
```

### Descrição

```text
## Objetivo

Adicionar funcionalidade de cadastro de clientes.

## Alterações realizadas

- Inclusão da API de cadastro
- Criação da tela de cadastro
- Implementação da validação de CPF

## Como testar

1. Acessar menu Clientes
2. Clicar em Novo Cliente
3. Preencher os campos obrigatórios
4. Salvar

## Evidências

Anexar prints ou vídeos.
```

---

# Revisão por Pares

## 7. Code Review

Toda alteração deve ser revisada por pelo menos outro desenvolvedor.

O revisor deve avaliar:

- Qualidade do código
- Arquitetura
- Segurança
- Performance
- Cobertura de testes
- Conformidade com os padrões do projeto

---

# Segurança

## 8. Regras obrigatórias

Nunca:

- Comitar senhas
- Comitar tokens
- Comitar arquivos de configuração sensíveis
- Armazenar segredos em código-fonte

Sempre:

- Utilizar variáveis de ambiente
- Sanitizar entradas de usuários
- Validar dados de entrada
- Aplicar princípio do menor privilégio

---

# Banco de Dados

## 9. Alterações de banco

Toda alteração estrutural deve possuir:

- Migration versionada
- Script reversível (rollback)
- Compatibilidade com ambientes existentes

Antes de realizar deploy:

- Validar impacto em performance
- Avaliar bloqueios de tabelas
- Verificar volume de dados

---

# Observabilidade

## 10. Logs e Monitoramento

Toda nova funcionalidade deve considerar:

- Logs apropriados
- Tratamento de erros
- Monitoramento
- Métricas quando aplicável

Evitar:

- Logs excessivos
- Logs contendo dados sensíveis

---

# Critérios para Merge

O merge somente poderá ser realizado quando:

- [ ] Branch atualizada com develop
- [ ] Build aprovado
- [ ] Testes aprovados
- [ ] Pull Request revisado
- [ ] Sem conflitos
- [ ] Sem vulnerabilidades críticas
- [ ] Documentação atualizada quando necessário

---

# Critérios para Deploy

Antes do deploy:

- [ ] Merge aprovado
- [ ] Pipeline concluída com sucesso
- [ ] Banco validado
- [ ] Plano de rollback definido
- [ ] Evidências de testes disponíveis

---

# Regra Principal

Qualidade, simplicidade, segurança e rastreabilidade devem sempre ter prioridade sobre velocidade de entrega.
