# Orientação Geral do Agente

Este arquivo é o ponto de entrada para qualquer agente de IA (Claude Code, Codex CLI, Cursor, GitHub Copilot, Windsurf ou outro) que trabalhe neste projeto. Leia-o por completo antes de iniciar qualquer tarefa de desenvolvimento.

---

## Sobre o Projeto

**[PREENCHER — substituir por informações do projeto atual antes de usar este arquivo]**

- **Nome do projeto:**
- **Descrição:** o que o sistema faz e para quem.
- **Stack e versões:** linguagens, frameworks, banco de dados.
- **Arquitetura:** padrão utilizado (ex.: Clean Architecture, DDD, MVC, microsserviços).
- **Estrutura de pastas:** organização principal do repositório.
- **Como rodar localmente:** comandos de setup, build e execução.
- **Comandos comuns:** build, testes, lint, migrations.
- **Contexto de domínio:** termos de negócio e regras específicas que não são óbvias pelo código.

---

## Guidelines Obrigatórias

Antes de escrever ou alterar qualquer código, leia integralmente os quatro arquivos abaixo e siga tudo o que eles determinam. Eles têm prioridade sobre qualquer convenção genérica do agente. Se o seu agente não os carregar automaticamente, leia-os manualmente como primeiro passo de qualquer tarefa:

- `docs/guidelines/developing.md` — fluxo de trabalho: branches, commits, Pull Requests, revisão de código, critérios de merge e deploy.
- `docs/guidelines/dotnet.md` — padrões de código C#/.NET: princípios (KISS, DRY, SOLID, YAGNI), estilo, nulidade, tratamento de exceções, testes, bibliotecas.
- `docs/guidelines/handle-error.md` — registro de erros: todo erro identificado, inferido ou corrigido deve ser documentado em `docs/erros/`, seguindo o template definido nesse arquivo.
- `docs/lessons-learned.md` — orientações de "fazer" e "evitar" extraídas de erros já resolvidos; consultar antes de implementar, para não repetir problemas conhecidos.

---

## Em Caso de Dúvida ou Conflito

- Dúvida sobre regra de negócio, mudança arquitetural ou nova biblioteca: parar e perguntar ao responsável pelo projeto antes de implementar — nunca presumir aprovação tácita (ver `dotnet.md`, seção "Regras de Ouro").
- Conflito entre este arquivo e uma das guidelines obrigatórias: as guidelines obrigatórias prevalecem, exceto quando a seção "Sobre o Projeto" acima registrar explicitamente uma exceção para este projeto.
- Conflito entre uma instrução direta dada na conversa e qualquer guideline: a instrução direta do responsável pelo projeto prevalece.

---

## Marcações de Ferramentas de IA

Nunca incluir marcações, assinaturas ou menções de LLM, agente de IA ou ferramenta de IA em qualquer artefato de texto produzido para o projeto: Pull Requests (título, descrição ou comentários), mensagens de commit, comentários no código-fonte, documentação, ou qualquer outro texto gerado como parte do trabalho.

Exemplos do que evitar:

- Rodapés como "Co-Authored-By: Claude" ou similares.
- Assinaturas do tipo "Generated with [ferramenta]" ou "Written by AI".
- Emojis ou marcas d'água que identifiquem a ferramenta usada.
- Comentários no código como `// gerado por IA` ou `// revisado pelo agente`.

Esta regra vale independentemente de qual agente ou ferramenta esteja sendo usado.
