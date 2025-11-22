# Guia Completo: Postagens no LinkedIn via n8n (HTTP Nodes)

> **Autor:** Script7  
> **Data:** Novembro 2024  
> **Objetivo:** Ensinar como fazer postagens automáticas no LinkedIn (perfil pessoal e páginas empresariais) usando n8n com nodes HTTP, sem depender do node nativo do LinkedIn.

---

## 📋 Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Configuração Inicial no LinkedIn Developers](#configuração-inicial-no-linkedin-developers)
3. [Configuração da Autenticação OAuth2 no n8n](#configuração-da-autenticação-oauth2-no-n8n)
4. [Obtendo seu ID de Perfil Pessoal](#obtendo-seu-id-de-perfil-pessoal)
5. [Postagem Simples no Perfil Pessoal](#postagem-simples-no-perfil-pessoal)
6. [Postagem com Imagem no Perfil Pessoal](#postagem-com-imagem-no-perfil-pessoal)
7. [Obtendo ID da Página Empresarial](#obtendo-id-da-página-empresarial)
8. [Postagem Simples na Página](#postagem-simples-na-página)
9. [Postagem com Imagem na Página](#postagem-com-imagem-na-página)
10. [Troubleshooting (Resolução de Problemas)](#troubleshooting)
11. [Dicas e Boas Práticas](#dicas-e-boas-práticas)

---

## 🎯 Pré-requisitos

Antes de começar, você precisará de:

- ✅ Conta no LinkedIn (pessoal)
- ✅ Página empresarial no LinkedIn (se quiser postar em páginas)
- ✅ Acesso administrativo à página (role: ADMINISTRATOR)
- ✅ n8n instalado e rodando (cloud ou self-hosted)
- ✅ Conhecimentos básicos de APIs REST e JSON

---

## 🔧 Configuração Inicial no LinkedIn Developers

### Passo 1: Criar um App no LinkedIn

1. Acesse: [LinkedIn Developers](https://www.linkedin.com/developers/apps)
2. Clique em **"Create app"**
3. Preencha as informações:
   - **App name:** n8n (ou o nome que preferir)
   - **LinkedIn Page:** Associe a uma página (se tiver)
   - **App logo:** Faça upload de um logo
   - **Legal agreement:** Aceite os termos
4. Clique em **"Create app"**

### Passo 2: Configurar Redirect URLs

1. No seu app, vá em **Auth** → **OAuth 2.0 settings**
2. Em **Redirect URLs**, adicione:
   ```
   https://seu-n8n.com/rest/oauth2-credential/callback
   ```
   ⚠️ **Importante:** Substitua `seu-n8n.com` pela URL real do seu n8n

3. Clique em **"Update"**

### Passo 3: Anotar Client ID e Client Secret

1. Na aba **Auth**, copie:
   - **Client ID** (exemplo: `77pqnghwiotpkb`)
   - **Client Secret** (clique em "Show" para visualizar)

2. ⚠️ **Guarde essas informações em local seguro!**

### Passo 4: Solicitar Acesso aos Produtos

1. Vá na aba **Products**
2. Localize **"Share on LinkedIn"** e clique em **"Request access"** (se não estiver aprovado)
3. Aguarde aprovação (geralmente é automática)

#### Para Postagens em Páginas:
Se você quiser postar em páginas empresariais, precisa também solicitar:
- **"Community Management API"** ou
- **"Marketing Developer Platform"**

⚠️ **Nota:** A aprovação para produtos de páginas pode demorar dias/semanas.

### Passo 5: Verificar OAuth 2.0 Scopes

1. Na aba **Auth**, role até **OAuth 2.0 scopes**
2. Verifique se estão disponíveis:
   - ✅ `openid`
   - ✅ `profile`
   - ✅ `email`
   - ✅ `w_member_social` (essencial para posts no perfil)
   - ✅ `w_organization_social` (necessário para posts em páginas)
   - ✅ `r_organization_social` (necessário para ler dados de páginas)

---

## 🔐 Configuração da Autenticação OAuth2 no n8n

### Passo 1: Criar Credencial OAuth2

1. No n8n, vá em **Credentials** → **Add Credential**
2. Procure por **"OAuth2 API"**
3. Configure os campos:

**Grant Type:**
```
Authorization Code
```

**Authorization URL:**
```
https://www.linkedin.com/oauth/v2/authorization
```

**Access Token URL:**
```
https://www.linkedin.com/oauth/v2/accessToken
```

**Client ID:**
```
[Cole o Client ID do seu app LinkedIn]
```

**Client Secret:**
```
[Cole o Client Secret do seu app LinkedIn]
```

**Scope:**
```
openid profile email w_member_social w_organization_social r_organization_social
```

**Auth URI Query Parameters:**
```
response_type=code
```

**Authentication:**
```
Body
```
⚠️ **IMPORTANTE:** Selecione "Body", não "Header"! Isso é crucial para funcionar.

### Passo 2: Conectar a Conta

1. Clique em **"Connect my account"** ou **"Save"**
2. Uma janela do LinkedIn será aberta
3. Autorize o acesso ao app
4. Você será redirecionado de volta ao n8n
5. Verifique se aparece **"Account connected"** ✅

### Passo 3: Nomear a Credencial

Dê um nome fácil de identificar, como:
```
LinkedIn Auth
```

---

## 👤 Obtendo seu ID de Perfil Pessoal

Antes de fazer postagens, você precisa do seu ID de perfil pessoal (URN).

### Node HTTP Request - GET userinfo

**Configuração:**
- **Method:** `GET`
- **URL:** `https://api.linkedin.com/v2/userinfo`
- **Authentication:** Selecione a credencial "LinkedIn Auth" criada anteriormente

**Resultado Esperado:**
```json
{
  "sub": "0E_xMnNp_J",
  "name": "Seu Nome",
  "given_name": "Seu",
  "family_name": "Nome",
  "email": "seu@email.com"
}
```

📌 **Anote o valor do campo `sub`** - esse é o seu ID de perfil!

No exemplo acima: `0E_xMnNp_J`

Seu URN completo será: `urn:li:person:0E_xMnNp_J`

---

## 📝 Postagem Simples no Perfil Pessoal

Agora vamos criar uma postagem de texto no seu perfil.

### Node HTTP Request - POST ugcPosts

**Configuração:**
- **Method:** `POST`
- **URL:** `https://api.linkedin.com/v2/ugcPosts`
- **Authentication:** Selecione "LinkedIn Auth"

**Headers:**
1. **Header 1:**
   - Name: `Content-Type`
   - Value: `application/json`

2. **Header 2:**
   - Name: `X-Restli-Protocol-Version`
   - Value: `2.0.0`

**Body:**
- **Body Content Type:** JSON
- **Specify Body:** Using JSON

**JSON:**
```json
{
  "author": "urn:li:person:SEU_ID_AQUI",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "Minha primeira postagem automatizada via n8n! 🚀"
      },
      "shareMediaCategory": "NONE"
    }
  },
  "visibility": {
    "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
  }
}
```

⚠️ **IMPORTANTE:** Substitua `SEU_ID_AQUI` pelo ID que você obteve no passo anterior (exemplo: `0E_xMnNp_J`)

### Resultado Esperado

Você receberá uma resposta com o ID da postagem:
```json
{
  "id": "urn:li:share:7395917808657772544"
}
```

✅ Acesse seu perfil no LinkedIn e verifique se a postagem apareceu!

### 🔄 Automatizando com Variáveis

Para não precisar copiar e colar o ID manualmente, use este workflow:

**Node 1 - HTTP Request (GET userinfo):**
- Busca seu ID

**Node 2 - Set:**
- Nome: `Preparar URN`
- Adicione este campo:
  - **Name:** `authorUrn`
  - **Value:** `urn:li:person:{{ $json.sub }}`

**Node 3 - HTTP Request (POST ugcPosts):**
- No JSON, use: `"author": "{{ $node['Preparar URN'].json.authorUrn }}"`

---

## 📸 Postagem com Imagem no Perfil Pessoal

Para postar com imagem, o processo tem **3 etapas**:

1. Registrar o upload
2. Fazer upload da imagem
3. Criar a postagem com a imagem

### ETAPA 1: Registrar Upload da Imagem

**Node HTTP Request 1:**
- **Method:** `POST`
- **URL:** `https://api.linkedin.com/v2/assets?action=registerUpload`
- **Authentication:** LinkedIn Auth

**Headers:**
1. Name: `Content-Type`, Value: `application/json`
2. Name: `X-Restli-Protocol-Version`, Value: `2.0.0`

**Body (JSON):**
```json
{
  "registerUploadRequest": {
    "recipes": [
      "urn:li:digitalmediaRecipe:feedshare-image"
    ],
    "owner": "urn:li:person:SEU_ID_AQUI",
    "serviceRelationships": [
      {
        "relationshipType": "OWNER",
        "identifier": "urn:li:userGeneratedContent"
      }
    ]
  }
}
```

**Resultado Esperado:**
```json
{
  "value": {
    "uploadMechanism": {
      "com.linkedin.digitalmedia.uploading.MediaUploadHttpRequest": {
        "uploadUrl": "https://api.linkedin.com/mediaUpload/..."
      }
    },
    "asset": "urn:li:digitalmediaAsset:XXXXXX"
  }
}
```

📌 **Importante:** Você vai precisar do `uploadUrl` e do `asset` nos próximos passos.

### ETAPA 2: Upload da Imagem

**Node HTTP Request 2:**
- **Method:** `POST`
- **URL:** Use expressão: `{{ $json.value.uploadMechanism["com.linkedin.digitalmedia.uploading.MediaUploadHttpRequest"].uploadUrl }}`
- **Authentication:** LinkedIn Auth

**Headers:**
- Name: `Content-Type`
- Value: `image/jpeg` (ou `image/png` se sua imagem for PNG)

**Body:**
- **Body Content Type:** Raw/Custom
- **Content Type:** `application/octet-stream`
- **Body:** Selecione **"Binary Data"**
- **Input Binary Field:** `data` (ou o nome do campo que contém sua imagem)

💡 **Como obter a imagem no n8n:**
- Use um node **"Read Binary File"** antes, ou
- Use um node **"HTTP Request"** para baixar de uma URL, ou
- Receba via **Webhook**

### ETAPA 3: Criar Post com Imagem

**Node HTTP Request 3:**
- **Method:** `POST`
- **URL:** `https://api.linkedin.com/v2/ugcPosts`
- **Authentication:** LinkedIn Auth

**Headers:**
1. Name: `Content-Type`, Value: `application/json`
2. Name: `X-Restli-Protocol-Version`, Value: `2.0.0`

**Body (JSON):**
```json
{
  "author": "urn:li:person:SEU_ID_AQUI",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "Postagem com imagem via n8n! 📸🚀"
      },
      "shareMediaCategory": "IMAGE",
      "media": [
        {
          "status": "READY",
          "description": {
            "text": "Descrição da imagem"
          },
          "media": "{{ $node['HTTP Request 1'].json.value.asset }}",
          "title": {
            "text": "Título da imagem"
          }
        }
      ]
    }
  },
  "visibility": {
    "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
  }
}
```

✅ Execute o workflow e verifique a postagem no seu perfil!

---

## 🏢 Obtendo ID da Página Empresarial

Se você tem uma página empresarial no LinkedIn, pode obter o ID dela de duas formas:

### Método 1: Via URL (Mais Simples)

1. Acesse sua página no LinkedIn
2. Vá em **Admin tools** → **Admin center**
3. A URL será algo como: `https://www.linkedin.com/company/110037454/admin/dashboard/`
4. O número **110037454** é o ID da sua página!

Seu URN completo será: `urn:li:organization:110037454`

### Método 2: Via API

**Node HTTP Request:**
- **Method:** `GET`
- **URL:** `https://api.linkedin.com/v2/organizationAcls?q=roleAssignee&projection=(elements*(organization~(localizedName),roleAssignee,role))`
- **Authentication:** LinkedIn Auth

**Headers:**
- Name: `X-Restli-Protocol-Version`
- Value: `2.0.0`

**Resultado:**
```json
{
  "elements": [
    {
      "organization~": {
        "localizedName": "Nome da Sua Página"
      },
      "organization": "urn:li:organization:110037454",
      "role": "ADMINISTRATOR"
    }
  ]
}
```

📌 Use o valor do campo `organization`

⚠️ **Nota:** Este endpoint só funciona se você tiver o scope `r_organization_social` habilitado.

---

## 📝 Postagem Simples na Página

O processo é **idêntico** ao perfil pessoal, apenas mudando o `author`.

### Node HTTP Request - POST ugcPosts

**Configuração:**
- **Method:** `POST`
- **URL:** `https://api.linkedin.com/v2/ugcPosts`
- **Authentication:** LinkedIn Auth

**Headers:**
1. Name: `Content-Type`, Value: `application/json`
2. Name: `X-Restli-Protocol-Version`, Value: `2.0.0`

**Body (JSON):**
```json
{
  "author": "urn:li:organization:110037454",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "Postagem da nossa página via n8n! 🚀"
      },
      "shareMediaCategory": "NONE"
    }
  },
  "visibility": {
    "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
  }
}
```

⚠️ **Diferença:** Use `"author": "urn:li:organization:ID_DA_PAGINA"` ao invés de `person`

---

## 📸 Postagem com Imagem na Página

O processo é **idêntico** ao perfil pessoal, mudando apenas o `owner` e `author` para a organização.

### ETAPA 1: Registrar Upload (PARA PÁGINA)

**Body (JSON):**
```json
{
  "registerUploadRequest": {
    "recipes": [
      "urn:li:digitalmediaRecipe:feedshare-image"
    ],
    "owner": "urn:li:organization:110037454",
    "serviceRelationships": [
      {
        "relationshipType": "OWNER",
        "identifier": "urn:li:userGeneratedContent"
      }
    ]
  }
}
```

⚠️ **Mudança:** `"owner": "urn:li:organization:ID_DA_PAGINA"`

### ETAPA 2: Upload da Imagem

**(Processo idêntico ao perfil pessoal - sem mudanças)**

### ETAPA 3: Criar Post com Imagem (NA PÁGINA)

**Body (JSON):**
```json
{
  "author": "urn:li:organization:110037454",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": {
        "text": "Post com imagem da nossa página! 📸"
      },
      "shareMediaCategory": "IMAGE",
      "media": [
        {
          "status": "READY",
          "description": {
            "text": "Descrição da imagem"
          },
          "media": "{{ $node['HTTP Request 1'].json.value.asset }}",
          "title": {
            "text": "Título da imagem"
          }
        }
      ]
    }
  },
  "visibility": {
    "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
  }
}
```

⚠️ **Mudança:** `"author": "urn:li:organization:ID_DA_PAGINA"`

---

## 🔧 Troubleshooting

### Erro: "Forbidden - perhaps check your credentials?"

**Causa:** Problema com autenticação ou scopes.

**Soluções:**
1. Verifique se a credencial está conectada (botão verde)
2. Reconecte a conta
3. Verifique se os scopes estão corretos
4. No OAuth2, certifique-se que "Authentication" está como **"Body"**, não "Header"

### Erro: "Not enough permissions to access: userinfo.GET.NO_VERSION"

**Causa:** Falta o scope `profile` ou `openid`.

**Solução:**
1. Adicione os scopes: `openid profile email w_member_social`
2. Reconecte a conta

### Erro: "Content is a duplicate of urn:li:share:XXXXX"

**Causa:** Você está tentando postar texto idêntico a um post anterior.

**Solução:**
- Altere o texto da postagem
- O LinkedIn bloqueia posts duplicados

### Erro: "Field Value validation failed in REQUEST_BODY: /author"

**Causa:** ID do autor está incorreto ou você não tem permissão.

**Soluções:**
1. Verifique se o URN está correto: `urn:li:person:ID` ou `urn:li:organization:ID`
2. Para páginas, certifique-se que tem o scope `w_organization_social`
3. Verifique se você é administrador da página

### Erro: "Not enough permissions to access: organizationalEntityAcls"

**Causa:** Falta o scope `r_organization_social` para ler dados de páginas.

**Solução:**
1. Adicione o scope: `r_organization_social`
2. Reconecte a conta
3. Solicite acesso ao produto "Marketing Developer Platform" ou "Community Management API"

### Postagem criada mas não aparece no LinkedIn

**Causas possíveis:**
1. Aguarde 1-2 minutos (delay de processamento)
2. Atualize a página do LinkedIn
3. Acesse diretamente seu perfil/página (não o feed)
4. A postagem pode estar em revisão (comum em contas novas usando API)

**Como verificar:**
Busque a postagem pelo ID retornado:
```
GET https://api.linkedin.com/v2/ugcPosts/urn:li:share:ID_DA_POSTAGEM
```

### Erro ao fazer upload de imagem (ETAPA 2)

**Causas possíveis:**
1. Content-Type incorreto (use `image/jpeg` ou `image/png`)
2. Imagem muito grande (máximo ~10MB)
3. Formato não suportado (use JPEG ou PNG)

**Soluções:**
- Verifique o tipo MIME correto da imagem
- Reduza o tamanho da imagem se necessário
- Certifique-se que está enviando Binary Data

---

## 💡 Dicas e Boas Práticas

### 1. Evite Posts Duplicados

O LinkedIn bloqueia postagens com texto idêntico. Para automatizar:

**Adicione timestamp:**
```json
"text": "Minha postagem - {{ $now.format('DD/MM/YYYY HH:mm') }}"
```

**Use variáveis dinâmicas:**
```json
"text": "{{ $json.conteudo }} #{{ $json.hashtag }}"
```

### 2. Visibilidade das Postagens

Opções de visibilidade:

**Público (qualquer pessoa):**
```json
"visibility": {
  "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
}
```

**Apenas conexões:**
```json
"visibility": {
  "com.linkedin.ugc.MemberNetworkVisibility": "CONNECTIONS"
}
```

### 3. Boas Práticas para Imagens

- **Tamanho recomendado:** 1200x627 pixels
- **Formato:** JPEG ou PNG
- **Tamanho máximo:** ~10MB
- **Sempre adicione:** `description` e `title` nas imagens

### 4. Workflow Reutilizável

Crie um workflow que funciona tanto para perfil quanto para página:

**Node 1 - Set (Configuração):**
```json
{
  "postType": "person",  // ou "organization"
  "authorId": "0E_xMnNp_J",  // ou "110037454"
  "message": "Conteúdo da postagem"
}
```

**Node 2 - Function:**
```javascript
const type = $json.postType;
const id = $json.authorId;
const authorUrn = `urn:li:${type}:${id}`;

return {
  authorUrn: authorUrn,
  message: $json.message
};
```

**Node 3 - HTTP Request:**
Use: `{{ $json.authorUrn }}` no campo `author`

### 5. Rate Limits

O LinkedIn tem limites de requisições:
- **Perfil pessoal:** ~100 posts por dia
- **Páginas:** Varia conforme o plano

**Dica:** Adicione delays entre postagens se estiver fazendo múltiplos posts.

### 6. Teste em Ambiente Controlado

Antes de automatizar completamente:
1. Teste com posts manuais
2. Verifique se as postagens aparecem corretamente
3. Teste com diferentes tipos de conteúdo (texto, imagem, links)

### 7. Monitoramento

Crie um sistema de log para suas postagens:
- Salve o ID retornado pelo LinkedIn
- Registre data/hora da postagem
- Armazene o status (sucesso/erro)

### 8. Múltiplas Imagens (Carrossel)

Para postar múltiplas imagens, adicione mais objetos no array `media`:

```json
"media": [
  {
    "status": "READY",
    "media": "urn:li:digitalmediaAsset:ASSET_1",
    "description": { "text": "Imagem 1" }
  },
  {
    "status": "READY",
    "media": "urn:li:digitalmediaAsset:ASSET_2",
    "description": { "text": "Imagem 2" }
  }
]
```

⚠️ **Nota:** Você precisará fazer upload de cada imagem separadamente (Etapas 1 e 2 para cada uma).

### 9. Agendamento de Posts

Use o node **Schedule Trigger** do n8n para agendar postagens:
- Define horários específicos
- Configure dias da semana
- Automatize totalmente o processo

### 10. Hashtags

Adicione hashtags diretamente no texto:
```json
"text": "Minha postagem sobre #lowcode #nocode #automação 🚀"
```

O LinkedIn reconhece automaticamente.

---

## 📊 Resumo das Diferenças: Perfil vs Página

| Aspecto | Perfil Pessoal | Página Empresarial |
|---------|----------------|-------------------|
| **URN Format** | `urn:li:person:ID` | `urn:li:organization:ID` |
| **Scope Necessário** | `w_member_social` | `w_organization_social` |
| **Owner (Upload)** | `urn:li:person:ID` | `urn:li:organization:ID` |
| **Author (Post)** | `urn:li:person:ID` | `urn:li:organization:ID` |
| **Produto LinkedIn** | "Share on LinkedIn" | "Marketing Developer Platform" ou "Community Management API" |
| **Aprovação** | Geralmente automática | Pode demorar dias/semanas |

---

## 🎓 Próximos Passos

Agora que você domina o básico, explore:

1. **Posts com Links:** Adicione preview de links
2. **Posts com Vídeo:** Upload de vídeos (processo similar a imagens, mas com `digitalmediaRecipe:feedshare-video`)
3. **Análises:** Use a API do LinkedIn para buscar métricas das suas postagens
4. **Comentários:** Automatize respostas a comentários
5. **Mensagens Diretas:** Integre com LinkedIn Messaging API

---

## 📚 Recursos Adicionais

- [LinkedIn API Documentation](https://docs.microsoft.com/en-us/linkedin/)
- [n8n Documentation](https://docs.n8n.io/)
- [LinkedIn Developer Portal](https://www.linkedin.com/developers/)
- [OAuth 2.0 Specification](https://oauth.net/2/)

---

## 🤝 Suporte

Se você encontrar problemas:

1. **Revise este guia** do início ao fim
2. **Verifique a seção Troubleshooting**
3. **Confira os logs do n8n** para erros detalhados
4. **Teste com cURL** para isolar se o problema é no n8n ou na API
5. **Entre em contato** com a comunidade Made In Low Code

---

## 📝 Changelog

- **v1.0 (Nov 2024):** Versão inicial do guia
  - Configuração OAuth2
  - Posts simples em perfil e página
  - Posts com imagem em perfil e página
  - Troubleshooting completo

---

## ⚖️ Licença e Uso

Este guia foi criado por **Script7** para uso educacional da comunidade **Made In Low Code**.

Você é livre para:
- ✅ Usar em seus projetos
- ✅ Compartilhar com seus alunos
- ✅ Adaptar conforme necessário
- ✅ Contribuir com melhorias

---

## 🎉 Conclusão

Parabéns! Você agora sabe como:
- ✅ Configurar autenticação OAuth2 com LinkedIn
- ✅ Fazer postagens de texto no perfil pessoal
- ✅ Fazer postagens com imagem no perfil pessoal
- ✅ Fazer postagens de texto em páginas
- ✅ Fazer postagens com imagem em páginas
- ✅ Resolver problemas comuns

Use esse conhecimento para criar automações incríveis e compartilhe seu aprendizado com a comunidade! 🚀

---

**Bons estudos e boas automações!** 💙

Script7 | Made In Low Code Community
