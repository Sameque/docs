# Integração Azure DevOps ↔ Microsoft Teams
## Notificações de Pull Requests para a Branch `develop`

---

**Versão:** 1.0  
**Data:** Agosto/2026  
**Escopo:** Envio automático de notificações ao Microsoft Teams sempre que um Pull Request (PR) for aberto direcionado à branch `develop` no Azure DevOps.

---

## 📋 Sumário

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Pré-requisitos](#2-pré-requisitos)
3. [Configuração no Microsoft Teams](#3-configuração-no-microsoft-teams)
   - 3.1 [Criando um Workflow de Webhook de Entrada](#31-criando-um-workflow-de-webhook-de-entrada)
   - 3.2 [Obtendo a URL do Webhook](#32-obtendo-a-url-do-webhook)
   - 3.3 [Testando o Webhook](#33-testando-o-webhook)
4. [Configuração no Azure DevOps](#4-configuração-no-azure-devops)
   - 4.1 [Criando um Personal Access Token (PAT)](#41-criando-um-personal-access-token-pat)
   - 4.2 [Configurando Service Hooks](#42-configurando-service-hooks)
   - 4.3 [Filtrando por Branch `develop`](#43-filtrando-por-branch-develop)
5. [Formato do Payload e Adaptive Cards](#5-formato-do-payload-e-adaptive-cards)
   - 5.1 [Estrutura do Evento `git.pullrequest.created`](#51-estrutura-do-evento-gitpullrequestcreated)
   - 5.2 [Exemplo de Adaptive Card para Teams](#52-exemplo-de-adaptive-card-para-teams)
6. [Implementação com Middleware (Recomendado)](#6-implementação-com-middleware-recomendado)
   - 6.1 [Por que usar um Middleware?](#61-por-que-usar-um-middleware)
   - 6.2 [Exemplo em Python (Azure Function)](#62-exemplo-em-python-azure-function)
   - 6.3 [Exemplo em Node.js](#63-exemplo-em-nodejs)
7. [Testes e Validação](#7-testes-e-validação)
8. [Troubleshooting](#8-troubleshooting)
9. [Boas Práticas e Segurança](#9-boas-práticas-e-seurança)
10. [Referências Oficiais](#10-referências-oficiais)

---

## 1. Visão Geral da Arquitetura

A integração entre **Azure DevOps** e **Microsoft Teams** para notificações de PRs segue o seguinte fluxo:

```
┌─────────────────┐     PR criado para      ┌──────────────────┐
│   Azure DevOps  │ ──── branch develop ───▶ │  Service Hook    │
│   (Repositório) │                          │  (Webhook HTTP)  │
└─────────────────┘                          └────────┬─────────┘
                                                      │
                                                      ▼
┌─────────────────┐                          ┌──────────────────┐
│  Canal/Grupo    │ ◀── Adaptive Card ────── │  Middleware      │
│    Teams        │     (JSON formatado)     │  (Opcional)      │
└─────────────────┘                          └──────────────────┘
```

**Fluxo detalhado:**
1. Um desenvolvedor cria um PR com target branch = `develop`
2. O Azure DevOps dispara o evento `git.pullrequest.created`
3. O Service Hook envia um payload JSON para um endpoint HTTP
4. *(Opcional)* Um middleware recebe o payload, valida que o target é `refs/heads/develop`, formata a mensagem
5. A mensagem formatada (Adaptive Card) é enviada para o webhook do Teams
6. O Teams exibe a notificação no canal/grupo configurado

> **Nota importante:** Os conectores do Office 365 no Teams estão sendo descontinuados. A Microsoft recomenda o uso de **Workflows** (Power Automate) para criar webhooks de entrada. Webhooks criados via conectores antigos continuarão funcionando até dezembro de 2025, mas não é mais possível criar novos.

---

## 2. Pré-requisitos

| Componente | Requisito |
|------------|-----------|
| **Azure DevOps** | Organização e projeto ativos com repositório Git |
| **Permissões Azure DevOps** | Membro do grupo **Project Administrators** ou permissões *Edit subscriptions* e *View subscriptions* em Service Hooks |
| **Microsoft Teams** | Canal de equipe com permissão para criar Workflows |
| **Permissões Teams** | Permissão para adicionar apps/workflows em canais |
| **PAT (Personal Access Token)** | Token com escopo *Code (read)* para validações via API (opcional) |

---

## 3. Configuração no Microsoft Teams

### 3.1 Criando um Workflow de Webhook de Entrada

O método recomendado atualmente é utilizar **Workflows** no Teams, pois os conectores tradicionais estão sendo aposentados.

**Passo a passo:**

1. No Microsoft Teams, navegue até o **canal** onde deseja receber as notificações
2. Clique nos **três pontos** (⋮) ao lado do nome do canal
3. Selecione **Workflows** (ou **Fluxos de trabalho**)
4. Escolha o modelo **"Postar em um canal quando um webhook for solicitado"** (ou similar)
   - Em inglês: *"Post to a channel when a webhook request is received"*
5. Clique em **Próximo** ou **Configurar**
6. Dê um nome descritivo ao workflow, por exemplo:
   - `Azure DevOps - PRs para Develop`
7. Selecione o **canal de destino** (deve ser o mesmo canal onde você está criando)
8. Clique em **Adicionar workflow** ou **Criar**

> **Dica:** Se a opção "Workflows" não aparecer no menu do canal, verifique se você tem permissões suficientes ou se o administrador do Teams habilitou Workflows para a equipe.

### 3.2 Obtendo a URL do Webhook

Após criar o workflow:

1. O Teams exibirá uma tela com a **URL do webhook**
2. A URL terá o seguinte formato:
   ```
   https://<tenant>.webhook.office.com/webhookb2/<guid1>/IncomingWebhook/<guid2>/<guid3>
   ```
3. **Copie e guarde esta URL** — ela será usada no Azure DevOps
4. Clique em **Concluir** ou **Fechar**

> **⚠️ Atenção:** A URL do webhook é uma credencial sensível. Qualquer pessoa com acesso a ela pode postar mensagens no canal. Armazene-a em um cofre de segredos (Azure Key Vault, GitHub Secrets, etc.).

### 3.3 Testando o Webhook

Antes de configurar o Azure DevOps, valide que o webhook funciona:

**Usando cURL:**
```bash
curl -X POST <URL_DO_WEBHOOK>   -H "Content-Type: application/json"   -d '{
    "type": "message",
    "attachments": [{
      "contentType": "application/vnd.microsoft.card.adaptive",
      "content": {
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "type": "AdaptiveCard",
        "version": "1.4",
        "body": [
          {
            "type": "TextBlock",
            "text": "✅ Teste de Webhook",
            "weight": "Bolder",
            "size": "Large",
            "color": "Good"
          },
          {
            "type": "TextBlock",
            "text": "Se você está vendo esta mensagem, o webhook está configurado corretamente!",
            "wrap": true
          }
        ]
      }
    }]
  }'
```

Se a mensagem aparecer no canal do Teams, o webhook está funcionando.

---

## 4. Configuração no Azure DevOps

### 4.1 Criando um Personal Access Token (PAT)

Um PAT é necessário se você for criar ou gerenciar Service Hooks via API REST. Para configuração via interface web, não é obrigatório, mas é recomendado para operações automatizadas.

**Passo a passo:**

1. No Azure DevOps, clique no ícone do usuário (canto superior direito)
2. Selecione **Configurações de usuário** → **Personal access tokens**
3. Clique em **+ Novo Token**
4. Preencha:
   - **Nome:** `Teams-Integration-PR-Develop`
   - **Organização:** Selecione sua organização
   - **Expiração:** Defina conforme sua política de segurança (recomendado: 90 dias)
   - **Escopos:**
     - **Code:** `Read` (mínimo necessário)
     - **Service Hooks:** `Read and write` (se for gerenciar via API)
5. Clique em **Criar**
6. **Copie o token imediatamente** — ele não será exibido novamente

### 4.2 Configurando Service Hooks

Service Hooks são o mecanismo nativo do Azure DevOps para enviar notificações a serviços externos quando eventos ocorrem.

**Passo a passo via Interface Web:**

1. No seu projeto Azure DevOps, acesse **Configurações do projeto** (Project Settings)
2. No menu lateral, selecione **Service hooks**
3. Clique em **Criar assinatura** (Create subscription)
4. Na lista de serviços, selecione **Web Hooks**
5. Clique em **Próximo**

**Configuração do Evento (Trigger):**

6. Em **Trigger on this type of event**, selecione:
   - **Pull request created** (ou `git.pullrequest.created`)
7. Em **Filters**, configure:
   - **Repository:** Selecione o repositório desejado
   - **Branch:** `develop` *(este filtro restringe a branch de destino — veja seção 4.3)*
   - **Pull request created by:** *(opcional)* Restrinja por usuário/grupo
   - **Pull request reviewers contains:** *(opcional)* Restrinja por revisores

**Configuração da Ação (Action):**

8. Em **Action**, configure:
   - **URL:** Cole a URL do webhook do Teams obtida na seção 3.2
   - **HTTP headers:**
     - `Content-Type: application/json`
   - **Resource details to send:** Selecione **All**
   - **Messages to send:** Selecione **All**
   - **Detailed messages to send:** Selecione **All**
9. Clique em **Testar** para validar a conexão
10. Se o teste for bem-sucedido, clique em **Concluir**

> **Nota sobre o filtro de branch:** O campo `branch` no Service Hook de PR filtra eventos onde a branch especificada está envolvida. No contexto de PRs, este filtro se aplica à branch de destino (target branch) quando o evento é de criação de PR.

### 4.3 Filtrando por Branch `develop`

A filtragem específica para a branch `develop` pode ser feita de duas formas:

#### Opção A: Filtro Nativo do Service Hook (Recomendado)

Durante a criação da assinatura (seção 4.2), no campo **Branch**, informe:
```
develop
```

Ou, se o filtro exigir o formato completo:
```
refs/heads/develop
```

Isso garante que apenas PRs com target branch `develop` disparem o webhook.

#### Opção B: Filtro via Middleware (Mais Flexível)

Se precisar de lógica mais complexa (ex: múltiplas branches, padrões regex), configure o Service Hook sem filtro de branch e faça a validação no middleware:

```python
# Exemplo de validação no middleware
target_branch = event["resource"]["targetRefName"]  # ex: "refs/heads/develop"

if target_branch == "refs/heads/develop":
    # Envia notificação para o Teams
    send_to_teams(formatted_message)
else:
    # Ignora o evento
    return {"status": "ignored", "reason": "Target branch is not develop"}
```

**Campos relevantes no payload do PR:**

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| `resource.sourceRefName` | Branch de origem | `refs/heads/feature/nova-funcionalidade` |
| `resource.targetRefName` | **Branch de destino** | `refs/heads/develop` |
| `resource.title` | Título do PR | `Implementa autenticação OAuth2` |
| `resource.description` | Descrição do PR | `...` |
| `resource.createdBy.displayName` | Nome do autor | `João Silva` |
| `resource.pullRequestId` | ID do PR | `42` |
| `resource.status` | Status do PR | `active` |
| `resource.repository.name` | Nome do repositório | `meu-projeto` |

---

## 5. Formato do Payload e Adaptive Cards

### 5.1 Estrutura do Evento `git.pullrequest.created`

Quando um PR é criado, o Azure DevOps envia um payload JSON similar a:

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "eventType": "git.pullrequest.created",
  "publisherId": "tfs",
  "message": {
    "text": "João Silva created pull request 42 (Implementa autenticação OAuth2)",
    "html": "João Silva created pull request <a href='...'>42</a> (Implementa autenticação OAuth2)",
    "markdown": "João Silva created pull request [42](...) (Implementa autenticação OAuth2)"
  },
  "detailedMessage": {
    "text": "João Silva created pull request 43...",
    "html": "...",
    "markdown": "..."
  },
  "resource": {
    "repository": {
      "id": "repo-guid",
      "name": "meu-projeto",
      "url": "https://dev.azure.com/org/proj/_apis/git/repositories/repo-guid",
      "project": {
        "name": "Meu Projeto",
        "id": "proj-guid"
      }
    },
    "pullRequestId": 42,
    "codeReviewId": 42,
    "status": "active",
    "createdBy": {
      "displayName": "João Silva",
      "id": "user-guid",
      "uniqueName": "joao.silva@empresa.com",
      "imageUrl": "https://dev.azure.com/org/_apis/GraphProfile/MemberAvatars/..."
    },
    "creationDate": "2026-08-14T10:30:00.000Z",
    "title": "Implementa autenticação OAuth2",
    "description": "Este PR adiciona suporte a autenticação OAuth2...",
    "sourceRefName": "refs/heads/feature/oauth2-login",
    "targetRefName": "refs/heads/develop",
    "mergeStatus": "succeeded",
    "mergeId": "merge-guid",
    "lastMergeSourceCommit": {
      "commitId": "abc123...",
      "url": "..."
    },
    "lastMergeTargetCommit": {
      "commitId": "def456...",
      "url": "..."
    },
    "reviewers": [
      {
        "displayName": "Maria Santos",
        "id": "reviewer-guid",
        "vote": 0,
        "isRequired": true
      }
    ],
    "url": "https://dev.azure.com/org/proj/_apis/git/repositories/repo-guid/pullRequests/42"
  },
  "resourceContainers": {
    "collection": {"id": "coll-guid"},
    "account": {"id": "acct-guid"},
    "project": {"id": "proj-guid"}
  },
  "createdDate": "2026-08-14T10:30:00.000Z"
}
```

### 5.2 Exemplo de Adaptive Card para Teams

Para enviar uma notificação rica ao Teams, utilize o formato **Adaptive Card**:

```json
{
  "type": "message",
  "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.adaptive",
      "content": {
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "type": "AdaptiveCard",
        "version": "1.4",
        "body": [
          {
            "type": "TextBlock",
            "text": "🔀 Novo Pull Request para Develop",
            "weight": "Bolder",
            "size": "Large",
            "color": "Accent"
          },
          {
            "type": "TextBlock",
            "text": "Implementa autenticação OAuth2",
            "weight": "Bolder",
            "size": "Medium",
            "wrap": true
          },
          {
            "type": "FactSet",
            "facts": [
              {
                "title": "Autor:",
                "value": "João Silva"
              },
              {
                "title": "Branch Origem:",
                "value": "feature/oauth2-login"
              },
              {
                "title": "Branch Destino:",
                "value": "develop"
              },
              {
                "title": "Repositório:",
                "value": "meu-projeto"
              },
              {
                "title": "Revisores:",
                "value": "Maria Santos (required)"
              }
            ]
          },
          {
            "type": "TextBlock",
            "text": "Este PR adiciona suporte a autenticação OAuth2...",
            "wrap": true,
            "isSubtle": true,
            "maxLines": 3
          }
        ],
        "actions": [
          {
            "type": "Action.OpenUrl",
            "title": "🔗 Revisar PR",
            "url": "https://dev.azure.com/org/proj/_git/meu-projeto/pullrequest/42"
          },
          {
            "type": "Action.OpenUrl",
            "title": "📋 Ver Diff",
            "url": "https://dev.azure.com/org/proj/_git/meu-projeto/pullrequest/42?_a=files"
          }
        ]
      }
    }
  ]
}
```

---

## 6. Implementação com Middleware (Recomendado)

### 6.1 Por que usar um Middleware?

Enviar o payload bruto do Azure DevOps diretamente para o Teams **não é recomendado** porque:
- O payload do Azure DevOps não está no formato esperado pelo Teams
- Não há filtragem avançada de branch
- Não é possível formatar a mensagem com Adaptive Cards
- Dificuldade em adicionar lógica de retry, logs e tratamento de erros

Um middleware (Azure Function, AWS Lambda, ou serviço próprio) atua como tradutor entre os dois serviços.

### 6.2 Exemplo em Python (Azure Function)

```python
import azure.functions as func
import requests
import json
import os
import logging

TEAMS_WEBHOOK_URL = os.environ["TEAMS_WEBHOOK_URL"]

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info("Recebendo evento do Azure DevOps")

    try:
        event = req.get_json()
        event_type = event.get("eventType", "")

        # Valida o tipo de evento
        if event_type != "git.pullrequest.created":
            return func.HttpResponse(
                json.dumps({"status": "ignored", "eventType": event_type}),
                status_code=200,
                mimetype="application/json"
            )

        resource = event.get("resource", {})
        target_branch = resource.get("targetRefName", "")

        # Filtra apenas PRs para develop
        if target_branch != "refs/heads/develop":
            return func.HttpResponse(
                json.dumps({
                    "status": "ignored",
                    "reason": f"Target branch is {target_branch}, not develop"
                }),
                status_code=200,
                mimetype="application/json"
            )

        # Extrai informações do PR
        pr_id = resource.get("pullRequestId")
        title = resource.get("title", "Sem título")
        description = resource.get("description", "Sem descrição")
        author = resource.get("createdBy", {}).get("displayName", "Desconhecido")
        source_branch = resource.get("sourceRefName", "").replace("refs/heads/", "")
        repo_name = resource.get("repository", {}).get("name", "")
        project_name = event.get("resourceContainers", {}).get("project", {}).get("name", "")

        # Obtém revisores
        reviewers = resource.get("reviewers", [])
        reviewers_text = ", ".join([
            f"{r['displayName']} {'(required)' if r.get('isRequired') else ''}"
            for r in reviewers
        ]) if reviewers else "Nenhum revisor atribuído"

        # Constrói a URL do PR
        org_url = event.get("resourceContainers", {}).get("account", {}).get("baseUrl", "")
        pr_url = f"https://dev.azure.com/{project_name}/_git/{repo_name}/pullrequest/{pr_id}"

        # Monta o Adaptive Card
        card = {
            "type": "message",
            "attachments": [{
                "contentType": "application/vnd.microsoft.card.adaptive",
                "content": {
                    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                    "type": "AdaptiveCard",
                    "version": "1.4",
                    "body": [
                        {
                            "type": "TextBlock",
                            "text": "🔀 Novo PR para Develop",
                            "weight": "Bolder",
                            "size": "Large",
                            "color": "Accent"
                        },
                        {
                            "type": "TextBlock",
                            "text": title,
                            "weight": "Bolder",
                            "size": "Medium",
                            "wrap": True
                        },
                        {
                            "type": "FactSet",
                            "facts": [
                                {"title": "Autor:", "value": author},
                                {"title": "Origem:", "value": source_branch},
                                {"title": "Destino:", "value": "develop"},
                                {"title": "Repositório:", "value": repo_name},
                                {"title": "Revisores:", "value": reviewers_text}
                            ]
                        },
                        {
                            "type": "TextBlock",
                            "text": description[:300] + "..." if len(description) > 300 else description,
                            "wrap": True,
                            "isSubtle": True,
                            "maxLines": 3
                        }
                    ],
                    "actions": [
                        {
                            "type": "Action.OpenUrl",
                            "title": "🔗 Revisar PR",
                            "url": pr_url
                        }
                    ]
                }
            }]
        }

        # Envia para o Teams
        response = requests.post(
            TEAMS_WEBHOOK_URL,
            json=card,
            headers={"Content-Type": "application/json"},
            timeout=10
        )
        response.raise_for_status()

        return func.HttpResponse(
            json.dumps({"status": "sent", "prId": pr_id}),
            status_code=200,
            mimetype="application/json"
        )

    except Exception as e:
        logging.error(f"Erro ao processar evento: {str(e)}")
        return func.HttpResponse(
            json.dumps({"status": "error", "message": str(e)}),
            status_code=500,
            mimetype="application/json"
        )
```

**Arquivo `function.json`:**
```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

### 6.3 Exemplo em Node.js

```javascript
const express = require('express');
const axios = require('axios');

const app = express();
app.use(express.json());

const TEAMS_WEBHOOK_URL = process.env.TEAMS_WEBHOOK_URL;

app.post('/webhook/azuredevops', async (req, res) => {
  const event = req.body;
  const eventType = event.eventType;

  console.log(`Evento recebido: ${eventType}`);

  // Ignora eventos que não são criação de PR
  if (eventType !== 'git.pullrequest.created') {
    return res.status(200).json({ status: 'ignored', eventType });
  }

  const resource = event.resource;
  const targetBranch = resource.targetRefName;

  // Filtra apenas PRs para develop
  if (targetBranch !== 'refs/heads/develop') {
    return res.status(200).json({
      status: 'ignored',
      reason: `Target branch is ${targetBranch}`
    });
  }

  const pr = {
    id: resource.pullRequestId,
    title: resource.title,
    author: resource.createdBy.displayName,
    sourceBranch: resource.sourceRefName.replace('refs/heads/', ''),
    targetBranch: 'develop',
    repo: resource.repository.name,
    description: resource.description || 'Sem descrição',
    url: resource._links?.web?.href || 
         `https://dev.azure.com/${event.resourceContainers.project.name}/_git/${resource.repository.name}/pullrequest/${resource.pullRequestId}`,
    reviewers: (resource.reviewers || []).map(r => r.displayName).join(', ') || 'Nenhum'
  };

  const card = {
    type: 'message',
    attachments: [{
      contentType: 'application/vnd.microsoft.card.adaptive',
      content: {
        $schema: 'http://adaptivecards.io/schemas/adaptive-card.json',
        type: 'AdaptiveCard',
        version: '1.4',
        body: [
          {
            type: 'TextBlock',
            text: '🔀 Novo PR para Develop',
            weight: 'Bolder',
            size: 'Large',
            color: 'Accent'
          },
          {
            type: 'TextBlock',
            text: pr.title,
            weight: 'Bolder',
            size: 'Medium',
            wrap: true
          },
          {
            type: 'FactSet',
            facts: [
              { title: 'Autor:', value: pr.author },
              { title: 'Origem:', value: pr.sourceBranch },
              { title: 'Destino:', value: pr.targetBranch },
              { title: 'Repositório:', value: pr.repo },
              { title: 'Revisores:', value: pr.reviewers }
            ]
          },
          {
            type: 'TextBlock',
            text: pr.description.substring(0, 300) + (pr.description.length > 300 ? '...' : ''),
            wrap: true,
            isSubtle: true,
            maxLines: 3
          }
        ],
        actions: [
          {
            type: 'Action.OpenUrl',
            title: '🔗 Revisar PR',
            url: pr.url
          }
        ]
      }
    }]
  };

  try {
    await axios.post(TEAMS_WEBHOOK_URL, card, {
      headers: { 'Content-Type': 'application/json' },
      timeout: 10000
    });
    res.status(200).json({ status: 'sent', prId: pr.id });
  } catch (error) {
    console.error('Erro ao enviar para Teams:', error.message);
    res.status(500).json({ status: 'error', message: error.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

---

## 7. Testes e Validação

### Teste 1: Validação do Webhook do Teams

```bash
curl -X POST <URL_DO_WEBHOOK>   -H "Content-Type: application/json"   -d '{"text": "Teste de conectividade"}'
```

**Resultado esperado:** Mensagem "Teste de conectividade" aparece no canal.

### Teste 2: Simulação de Evento do Azure DevOps

Você pode reenviar eventos do histórico do Service Hook:

1. No Azure DevOps, acesse **Project Settings → Service hooks**
2. Encontre sua assinatura e clique nela
3. Na aba **History**, visualize entregas anteriores
4. Clique em **Redeliver** (Reenviar) para testar novamente

### Teste 3: Criar um PR Real

1. Crie uma branch a partir de `develop`:
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/teste-integracao
   ```
2. Faça uma alteração qualquer e commit:
   ```bash
   echo "teste" >> teste.txt
   git add teste.txt
   git commit -m "Teste de integração Teams"
   git push origin feature/teste-integracao
   ```
3. Crie um PR via portal Azure DevOps com target = `develop`
4. Verifique se a notificação aparece no Teams em até 30 segundos

### Teste 4: PR para Outra Branch (Teste Negativo)

1. Crie um PR com target = `main` (ou outra branch)
2. **Resultado esperado:** Nenhuma notificação deve ser enviada ao Teams

---

## 8. Troubleshooting

| Problema | Causa Provável | Solução |
|----------|---------------|---------|
| **Mensagem não chega ao Teams** | URL do webhook incorreta | Verifique se a URL está completa e sem espaços |
| **Erro 400 Bad Request** | Payload mal formatado | Verifique se o JSON está válido e se o Adaptive Card segue o schema 1.4 |
| **Cards aparecem como texto puro** | Formato MessageCard vs Adaptive Card | Webhooks modernos do Teams usam Adaptive Cards. Não use o formato legado `@type: MessageCard` |
| **Erro 429 (Rate Limit)** | Muitas requisições | O Teams limita a ~4 mensagens/segundo. Implemente retry com backoff exponencial |
| **Webhook retorna 404** | Workflow foi removido ou reconfigurado | Recrie o workflow no Teams e atualize a URL no Azure DevOps |
| **Service Hook não dispara** | Filtros muito restritivos | Verifique se o filtro de branch está correto (`develop` ou `refs/heads/develop`) |
| **Evento dispara para qualquer branch** | Filtro de branch não configurado | Adicione o filtro `branch: develop` no Service Hook ou valide no middleware |
| **Permissão negada no Azure DevOps** | Usuário sem permissões | Conceda as permissões *Edit subscriptions* e *View subscriptions* em Service Hooks |
| **Mensagem chega mas sem formatação** | Content-Type incorreto | Certifique-se de enviar `Content-Type: application/json` |

### Como verificar o histórico de entregas do Service Hook

1. Acesse **Project Settings → Service hooks**
2. Clique na assinatura criada
3. Vá para a aba **History**
4. Você verá uma lista com:
   - Data/hora da tentativa
   - Status HTTP (200, 400, 500, etc.)
   - Tempo de resposta
   - Botão para reenviar (Redeliver)

---

## 9. Boas Práticas e Segurança

### 🔐 Segurança

1. **Proteja a URL do webhook:** Trate a URL como uma credencial. Armazene em:
   - Azure Key Vault
   - GitHub Secrets / GitLab CI Variables
   - Azure App Configuration
   - **Nunca** commite a URL em repositórios públicos

2. **Use HTTPS sempre:** O Azure DevOps só permite webhooks HTTPS

3. **Valide a origem do payload:** Em produção, valide que o request realmente vem do Azure DevOps verificando headers ou implementando HMAC (se disponível)

4. **Roteie por canal:** Use webhooks diferentes para diferentes tipos de eventos (PRs, builds, deploys) em canais separados

5. **Renove PATs periodicamente:** Configure expiração de 90 dias para tokens de acesso

### ✅ Boas Práticas

1. **Filtre agressivamente na origem:** Configure filtros no Service Hook para reduzir tráfego desnecessário
2. **Inclua links diretos:** Todo card deve ter um botão `Action.OpenUrl` levando direto ao PR
3. **Código por cor:**
   - 🟢 `Good` — Sucesso, PR aprovado
   - 🔴 `Attention` — Erro, build falhou
   - 🟡 `Warning` — Aprovação pendente
4. **Limite descrições:** Trunque descrições longas para evitar cards gigantes
5. **Monitore falhas:** Implemente logging e um endpoint `/health` no middleware
6. **Agrupe notificações:** Se múltiplos PRs forem criados em curto intervalo, considere agrupar em um único card
7. **Mencione revisores:** Use `@mentions` no Teams quando possível para notificar revisores diretamente

---

## 10. Referências Oficiais

| Recurso | Link |
|---------|------|
| **Service Hooks - Azure DevOps** | https://learn.microsoft.com/en-us/azure/devops/service-hooks/overview |
| **Eventos de Service Hook** | https://learn.microsoft.com/en-us/azure/devops/service-hooks/events |
| **Integração Azure DevOps + Teams** | https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/teams |
| **Webhooks e Connectors - Teams** | https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/what-are-webhooks-and-connectors |
| **Adaptive Cards Documentation** | https://adaptivecards.io/explorer/ |
| **Aposentadoria de Office 365 Connectors** | https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using |
| **Azure DevOps REST API - Subscriptions** | https://learn.microsoft.com/en-us/rest/api/azure/devops/hooks/subscriptions/create |
| **Pull Requests - Azure Repos** | https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests |
| **Branch Policies** | https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies |

---

*Documentação elaborada com base na documentação oficial da Microsoft. Para dúvidas ou atualizações, consulte sempre os links oficiais acima.*
