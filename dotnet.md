# Diretrizes de Desenvolvimento C#/.NET

## Linguagem

- Utilizar .NET 10 e a versão do C# suportada pelo projeto.
- Sempre utilizar recursos modernos da linguagem quando aumentarem a legibilidade e simplicidade do código.
- Seguir as especificações ECMA-334 (C#) e ECMA-335 (CLI) como referência.

---

## Princípios Fundamentais

### KISS (Keep It Simple, Stupid)

- Priorizar soluções simples.
- Evitar abstrações desnecessárias.
- Não criar complexidade sem necessidade comprovada.

### DRY (Don't Repeat Yourself)

- Evitar duplicação de código.
- Centralizar regras compartilhadas.
- Reutilizar componentes, serviços e métodos existentes.

### SOLID

Aplicar os princípios SOLID sempre que fizer sentido:

- Single Responsibility Principle (SRP)
- Open/Closed Principle (OCP)
- Liskov Substitution Principle (LSP)
- Interface Segregation Principle (ISP)
- Dependency Inversion Principle (DIP)

### YAGNI (You Aren't Gonna Need It)

- Não implementar funcionalidades especulativas.
- Desenvolver apenas o necessário para atender ao requisito atual.
- Evitar generalizações prematuras.

---

## Código

### Estilo de Codificação

- Utilizar nomenclaturas em inglês para classes, métodos, propriedades, variáveis, interfaces e namespaces.
- Exceções podem ser feitas quando o domínio de negócio exigir terminologia específica.
- Os nomes devem ser autoexplicativos.
- O nome de uma classe, método, propriedade ou variável deve comunicar claramente sua responsabilidade.
- Seguir a regra: tão longo quanto necessário para ser claro e tão curto quanto possível para manter a legibilidade.
- Utilizar PascalCase para classes, métodos, propriedades, enums e records.
- Utilizar camelCase para parâmetros e variáveis locais.
- Interfaces devem iniciar com o prefixo "I".
- Evitar abreviações não padronizadas.

---

### Métodos

- Cada método deve possuir apenas uma responsabilidade.
- Métodos devem ser pequenos e objetivos.
- Sempre preferir extração de métodos quando um bloco possuir responsabilidade distinta.
- Evitar múltiplos níveis de aninhamento.
- Utilizar guard clauses para reduzir complexidade.
- Métodos privados devem ser utilizados para melhorar a legibilidade e organização.

---

### Organização

- Cada classe deve possuir seu próprio arquivo.
- O nome do arquivo deve ser igual ao nome da classe principal.
- Organizar pastas conforme a arquitetura do projeto.
- Evitar classes excessivamente grandes (God Classes).
- Manter alta coesão e baixo acoplamento.

---

### Injeção de Dependências

- Utilizar injeção de dependências via construtor.
- Realizar validação de null para todas as dependências injetadas.
- Utilizar ArgumentNullException.ThrowIfNull().
- Evitar Service Locator.
- Remover dependências não utilizadas.

---

### Mapeamento de Classes

- Separar a lógica de mapeamento das regras de negócio.
- Sempre que possível utilizar métodos Create() e From().
- Utilizar classes de mapeamento específicas quando houver muitas conversões.
- Evitar duplicação de código de mapeamento.
- Garantir reutilização entre camadas da aplicação.

---

### Tratamento de Exceções

- Nunca utilizar blocos catch vazios.
- Registrar erros relevantes em logs.
- Não utilizar exceções para controle de fluxo.
- Capturar exceções apenas quando existir uma ação de recuperação.
- Preservar a stack trace original.

---

### Programação Assíncrona

- Utilizar async/await para operações de I/O.
- Evitar .Result e .Wait().
- Utilizar CancellationToken quando aplicável.
- Métodos assíncronos devem possuir o sufixo Async.

---

### Código Limpo

- Remover código morto.
- Remover comentários obsoletos.
- Evitar uso excessivo de #region.
- Não deixar TODOs sem justificativa.
- Utilizar readonly sempre que possível.
- Declarar variáveis com o menor escopo possível.
- Priorizar legibilidade sobre otimizações prematuras.

---

### Testes

- Toda regra de negócio deve possuir testes automatizados quando viável.
- Priorizar testes de comportamento.
- Corrigir testes quebrados antes de finalizar uma implementação.
- Garantir que novas funcionalidades não introduzam regressões.

---

### Logs

- Registrar apenas informações relevantes.
- Nunca registrar senhas, tokens ou dados sensíveis.
- Utilizar níveis adequados de log (Information, Warning, Error e Critical).

---

## Bibliotecas

- Avaliar primeiro as funcionalidades nativas do .NET.
- Não adicionar bibliotecas externas sem justificativa técnica.
- Priorizar bibliotecas amplamente utilizadas e mantidas.
- Antes de adicionar uma biblioteca, apresentar justificativa e solicitar aprovação.
- Evitar desenvolver soluções próprias para problemas já resolvidos de forma estável e consolidada pelo ecossistema.

---

## Regras de Ouro

- Priorizar simplicidade, legibilidade e manutenção.
- Não realizar mudanças arquiteturais sem aprovação prévia.
- Não adicionar bibliotecas, frameworks ou infraestruturas sem aprovação.
- Em caso de dúvida sobre regras de negócio, solicitar esclarecimentos antes da implementação.
- Para decisões técnicas de baixo impacto, seguir as diretrizes deste documento e os padrões já existentes no projeto.
- Sempre considerar KISS, DRY, SOLID e YAGNI antes de implementar qualquer solução.
