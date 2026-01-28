# Guia de Configuracao - Lead Collector TimeLabs v2

## Passo a Passo Completo

---

## 1. Configurar Airtable

### 1.1 Criar Base
1. Acesse [airtable.com](https://airtable.com)
2. Clique em "Add a base" > "Start from scratch"
3. Nomeie: `TimeLabs Leads`

### 1.2 Criar Tabela
Renomeie a tabela para `Leads` e adicione os campos:

| Campo | Tipo | Configuracao |
|-------|------|--------------|
| nome | Single line text | Primary field |
| telefone | Phone | - |
| email | Email | - |
| endereco | Long text | - |
| bairro | Single line text | - |
| website | URL | - |
| instagram | URL | - |
| linkedin | URL | - |
| whatsapp | Phone | - |
| facebook | URL | - |
| categoria | Single select | Adicionar opcoes depois |
| rating | Number | Precision: 1.0 |
| reviews | Number | Integer |
| score | Number | Integer |
| temperatura | Single select | HOT, WARM, COOL, COLD |
| prioridade | Single select | Alta, Media, Baixa |
| motivos_score | Long text | - |
| data_coleta | Date | Include time |
| status_contato | Single select | Nao contatado, Em progresso, Convertido, Descartado |

### 1.3 Obter Credenciais
1. Clique no seu perfil > "Developer hub"
2. Clique "Create new token"
3. Nome: `N8N Integration`
4. Scopes: `data.records:read`, `data.records:write`
5. Access: Selecione sua base `TimeLabs Leads`
6. **Copie o token** (so aparece uma vez!)

### 1.4 Obter IDs
1. Abra sua base no Airtable
2. A URL sera: `https://airtable.com/appXXXXXXXXXXXXXX/tblYYYYYYYYYYYYYY/...`
3. `appXXXXXXXXXXXXXX` = Base ID
4. `tblYYYYYYYYYYYYYY` = Table ID

---

## 2. Configurar Apify

### 2.1 Criar Conta
1. Acesse [apify.com](https://apify.com)
2. Crie conta gratuita (vem com $5 de credito)

### 2.2 Obter API Token
1. Va em Settings > Integrations
2. Copie seu "Personal API Token"

---

## 3. Configurar N8N

### 3.1 Importar Workflow
1. Abra seu N8N
2. Va em "Workflows" > "Import from file"
3. Selecione `workflows/lead_collector_v2.json`

### 3.2 Configurar Credenciais

#### Apify
1. Clique no node "Google Maps Scraper"
2. Em "Credential to connect with", clique "Create New"
3. Cole seu API Token
4. Salve

#### Airtable
1. Clique no node "Salvar no Airtable"
2. Em "Credential", clique "Create New"
3. Cole seu Personal Access Token
4. Salve
5. **IMPORTANTE**: Edite o node e preencha:
   - Base ID: `appXXXXXXXXXXXXXX` (seu ID)
   - Table ID: `tblYYYYYYYYYYYYYY` (seu ID)

#### Email (Opcional)
1. Clique no node "Notificar Hot Lead"
2. Configure SMTP ou use Gmail:
   - Host: smtp.gmail.com
   - Port: 465
   - User: seu-email@gmail.com
   - Password: [App Password do Google]

---

## 4. Testar o Workflow

### 4.1 Teste Inicial
1. No node "Configuracoes", mude `maxResultadosPorCategoria` para `1`
2. Clique "Execute Workflow"
3. Observe cada node executar
4. Verifique se dados chegaram no Airtable

### 4.2 Ajustar e Expandir
1. Se funcionou, aumente `maxResultadosPorCategoria` para `10`
2. Execute novamente
3. Verifique qualidade dos dados

---

## 5. Agendar Execucao Automatica

### 5.1 Adicionar Trigger de Agendamento
1. Delete o node "Iniciar Coleta"
2. Adicione node "Schedule Trigger"
3. Configure:
   - Trigger Times: `0 8 * * 1` (toda segunda 8h)
   - Ou escolha intervalo visual

### 5.2 Ativar Workflow
1. Clique no toggle "Active" no canto superior direito
2. O workflow agora roda automaticamente

---

## 6. Personalizacoes

### 6.1 Adicionar/Remover Categorias
No node "Gerar Termos de Busca", edite a lista:

```javascript
const categorias = [
  'e-commerce',
  'loja de roupas',
  // Adicione ou remova aqui
  'sua nova categoria'
];
```

### 6.2 Mudar Cidade
No node "Configuracoes", mude:
```
cidade: "Sao Paulo"
```

### 6.3 Ajustar Scoring
No node "Calcular Score", modifique os pesos:

```javascript
// Exemplo: dar mais peso para LinkedIn
if (lead.linkedin) {
  score += 20; // era 15
}
```

---

## 7. Solucao de Problemas

### Erro: "Rate limit exceeded" no Apify
- **Causa**: Muitas requisicoes
- **Solucao**: Reduza `maxResultadosPorCategoria` ou adicione delays entre buscas

### Erro: "Invalid credentials" no Airtable
- **Causa**: Token expirou ou permissoes erradas
- **Solucao**: Gere novo token com permissoes corretas

### Leads sem Instagram/LinkedIn
- **Causa**: Site nao tem links ou usa formato diferente
- **Solucao**: Normal, nem toda empresa tem redes sociais no site

### Score muito baixo para todos
- **Causa**: Pesos podem nao estar adequados ao seu mercado
- **Solucao**: Ajuste os pesos no node "Calcular Score"

---

## 8. Custos Estimados

| Cenario | Leads/Mes | Custo Apify | Custo Total |
|---------|-----------|-------------|-------------|
| Teste | 100 | $0.50 | ~$0.50 |
| Pequeno | 500 | $2.50 | ~$3 |
| Medio | 2000 | $10 | ~$12 |
| Grande | 5000 | $25 | ~$30 |

---

## 9. Proximos Passos

1. [ ] Configurar Airtable
2. [ ] Configurar Apify
3. [ ] Importar workflow no N8N
4. [ ] Configurar credenciais
5. [ ] Testar com 1 resultado por categoria
6. [ ] Verificar dados no Airtable
7. [ ] Aumentar para 10-20 resultados
8. [ ] Agendar execucao automatica
9. [ ] Comecar a contatar leads!

---

## Suporte

- Documentacao N8N: [docs.n8n.io](https://docs.n8n.io)
- Documentacao Apify: [docs.apify.com](https://docs.apify.com)
- Documentacao Airtable: [support.airtable.com](https://support.airtable.com)
