# Sistema de Captação e Qualificação de Leads B2B - TimeLabs

## Sumário Executivo

Este documento apresenta a arquitetura completa para um sistema de captação de leads B2B focado em empresas de Belo Horizonte, com extração de redes sociais e scoring inteligente.

---

## 1. Análise: N8N vs Alternativas

### Veredicto: **Continuar com N8N** (com otimizações)

| Ferramenta | Prós | Contras | Recomendação |
|------------|------|---------|--------------|
| **N8N** | Já conhece, visual, integrado, self-hosted | Limitado para scraping complexo | ✅ **Usar** |
| **Make/Integromat** | Mais estável | Pago, menos controle | ❌ Não |
| **Python puro** | Máximo controle | Curva de aprendizado alta | ❌ Não para iniciante |
| **Zapier** | Fácil | Caro, menos flexível | ❌ Não |

### Por que N8N é a escolha certa para você:
1. **Você já tem** - evita curva de aprendizado nova
2. **Visual** - ideal para vibe coding
3. **Gratuito** - self-hosted sem limites
4. **Integrações** - Airtable, OpenAI, HTTP nativas

### O que precisa mudar no seu workflow atual:
- ❌ Remover Apify Google Search (caro e instável)
- ❌ Remover LinkedIn Scraper Apify (bloqueios frequentes)
- ✅ Usar abordagem híbrida mais confiável
- ✅ Simplificar pipeline

---

## 2. Arquitetura Proposta

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SISTEMA DE LEADS TIMELABS v2                      │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   COLETA     │ ──▶ │ ENRIQUECIMENTO│ ──▶ │ QUALIFICAÇÃO │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     │
       ▼                    ▼                     ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Google Maps  │     │  Website     │     │   OpenAI     │
│   (Apify)    │     │  Scraping    │     │   Scoring    │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     │
       │              ┌─────┴─────┐               │
       │              ▼           ▼               │
       │        ┌─────────┐ ┌─────────┐          │
       │        │Instagram│ │LinkedIn │          │
       │        │  API    │ │Discovery│          │
       │        └─────────┘ └─────────┘          │
       │                                          │
       └──────────────────┬───────────────────────┘
                          ▼
                   ┌──────────────┐
                   │   AIRTABLE   │
                   │   (Output)   │
                   └──────────────┘
```

---

## 3. APIs e Ferramentas Recomendadas

### 3.1 Para Google Maps (MANTER)
**Apify - Google Maps Scraper**
- Actor: `nwua9Gu5YrADL7ZDj`
- Custo: ~$5 por 1000 empresas
- ✅ Funciona bem, manter

### 3.2 Para Redes Sociais (NOVA ABORDAGEM)

#### Instagram - 3 Opções (da melhor para a mais simples):

| Opção | Ferramenta | Custo | Dificuldade | Dados |
|-------|------------|-------|-------------|-------|
| **A** | Apify Instagram Scraper | ~$10/1000 | Fácil | Completo |
| **B** | RapidAPI - Instagram API | $0-50/mês | Médio | Básico |
| **C** | Extração do site | $0 | No workflow | Limitado |

**Recomendação: Opção C** (extrair do próprio site da empresa)
- Mais confiável
- Sem custo adicional
- Sem risco de bloqueio

#### LinkedIn - 3 Opções:

| Opção | Método | Custo | Confiabilidade |
|-------|--------|-------|----------------|
| **A** | Apify LinkedIn Scraper | ~$25/1000 | Baixa (bloqueios) |
| **B** | Busca Google site:linkedin.com | $5/1000 | Média |
| **C** | Padrão de URL previsível | $0 | Alta |

**Recomendação: Opção C + B como fallback**

```
LinkedIn URL Pattern:
linkedin.com/company/{nome-empresa-slug}

Exemplo:
"Clínica Saúde Total" → linkedin.com/company/clinica-saude-total
```

### 3.3 Tabela de APIs Final

| Serviço | API/Ferramenta | Custo Estimado |
|---------|---------------|----------------|
| Google Maps | Apify nwua9Gu5YrADL7ZDj | $5/1000 leads |
| Website Crawler | Apify aYG0l9s7dbB7j3gbS | $3/1000 páginas |
| LinkedIn Discovery | HTTP Request (padrão URL) | $0 |
| Instagram | Extrair do site | $0 |
| Qualificação | OpenAI GPT-4o-mini | $0.50/1000 leads |
| Storage | Airtable | $0 (free tier) |

**Custo total estimado: ~$8.50 por 1000 leads**

---

## 4. Lógica de Scoring de Leads (Melhorada)

### 4.1 Sistema de Pontuação (0-100)

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCORING DE LEADS v2                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DADOS BÁSICOS (máx 25 pts)                                    │
│  ├── Telefone válido ..................... +10 pts             │
│  ├── Email corporativo ................... +10 pts             │
│  └── Website funcional ................... +5 pts              │
│                                                                 │
│  PRESENÇA DIGITAL (máx 35 pts)                                 │
│  ├── LinkedIn encontrado ................. +15 pts             │
│  ├── Instagram encontrado ................ +10 pts             │
│  ├── WhatsApp Business ................... +5 pts              │
│  └── 3+ redes sociais .................... +5 pts (bônus)      │
│                                                                 │
│  REPUTAÇÃO (máx 20 pts)                                        │
│  ├── Rating Google ≥ 4.5 ................. +10 pts             │
│  ├── Rating Google ≥ 4.0 ................. +7 pts              │
│  ├── Rating Google ≥ 3.5 ................. +3 pts              │
│  ├── Reviews > 50 ....................... +5 pts              │
│  └── Reviews > 20 ....................... +3 pts              │
│                                                                 │
│  SEGMENTO (máx 20 pts)                                         │
│  ├── E-commerce/Loja Online .............. +20 pts             │
│  ├── Clínica/Consultório ................. +15 pts             │
│  ├── Escritório (Contab/Adv) ............. +15 pts             │
│  ├── Agência de Marketing ................ +10 pts             │
│  └── Outros segmentos .................... +5 pts              │
│                                                                 │
│  PENALIDADES                                                    │
│  ├── Sem telefone E sem email ............ -15 pts             │
│  ├── Rating < 3.0 ....................... -10 pts             │
│  └── Empresa fechada/inativa ............. -100 pts            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Classificação por Temperatura

| Score | Temperatura | Ação Recomendada |
|-------|-------------|------------------|
| 80-100 | 🔥 HOT | Contato imediato via LinkedIn/WhatsApp |
| 60-79 | 🌡️ WARM | Contato em até 48h via email |
| 40-59 | ❄️ COOL | Adicionar em campanha de nurturing |
| 0-39 | 🧊 COLD | Descartar ou revisar manualmente |

### 4.3 Código JavaScript para N8N (Scoring)

```javascript
// Colar no node "Code" do N8N
function calcularScore(lead) {
  let score = 0;
  let motivos = [];

  // DADOS BÁSICOS (25 pts max)
  if (lead.telefone && lead.telefone.length >= 10) {
    score += 10;
    motivos.push("+10: Telefone válido");
  }

  if (lead.email && lead.email.includes('@') && !lead.email.includes('exemplo')) {
    score += 10;
    motivos.push("+10: Email corporativo");
  }

  if (lead.website && lead.website.startsWith('http')) {
    score += 5;
    motivos.push("+5: Website funcional");
  }

  // PRESENÇA DIGITAL (35 pts max)
  if (lead.linkedin && lead.linkedin.includes('linkedin.com')) {
    score += 15;
    motivos.push("+15: LinkedIn encontrado");
  }

  if (lead.instagram && lead.instagram.includes('instagram.com')) {
    score += 10;
    motivos.push("+10: Instagram encontrado");
  }

  if (lead.whatsapp) {
    score += 5;
    motivos.push("+5: WhatsApp Business");
  }

  const redesSociais = [lead.linkedin, lead.instagram, lead.facebook, lead.twitter].filter(Boolean).length;
  if (redesSociais >= 3) {
    score += 5;
    motivos.push("+5: 3+ redes sociais");
  }

  // REPUTAÇÃO (20 pts max)
  const rating = parseFloat(lead.rating) || 0;
  if (rating >= 4.5) {
    score += 10;
    motivos.push("+10: Rating excelente (4.5+)");
  } else if (rating >= 4.0) {
    score += 7;
    motivos.push("+7: Rating bom (4.0+)");
  } else if (rating >= 3.5) {
    score += 3;
    motivos.push("+3: Rating aceitável (3.5+)");
  }

  const reviews = parseInt(lead.reviews) || 0;
  if (reviews > 50) {
    score += 5;
    motivos.push("+5: Muitas avaliações (50+)");
  } else if (reviews > 20) {
    score += 3;
    motivos.push("+3: Avaliações moderadas (20+)");
  }

  // SEGMENTO (20 pts max)
  const segmentosPrioritarios = {
    'e-commerce': 20,
    'loja online': 20,
    'clínica': 15,
    'consultório': 15,
    'escritório': 15,
    'contabilidade': 15,
    'advocacia': 15,
    'agência': 10,
    'marketing': 10
  };

  const segmento = (lead.categoria || lead.segmento || '').toLowerCase();
  let pontuacaoSegmento = 5; // default

  for (const [key, pts] of Object.entries(segmentosPrioritarios)) {
    if (segmento.includes(key)) {
      pontuacaoSegmento = pts;
      break;
    }
  }
  score += pontuacaoSegmento;
  motivos.push(`+${pontuacaoSegmento}: Segmento ${segmento || 'geral'}`);

  // PENALIDADES
  if (!lead.telefone && !lead.email) {
    score -= 15;
    motivos.push("-15: Sem contato direto");
  }

  if (rating > 0 && rating < 3.0) {
    score -= 10;
    motivos.push("-10: Rating baixo");
  }

  // Classificação
  let temperatura;
  if (score >= 80) temperatura = 'HOT';
  else if (score >= 60) temperatura = 'WARM';
  else if (score >= 40) temperatura = 'COOL';
  else temperatura = 'COLD';

  return {
    score: Math.max(0, Math.min(100, score)),
    temperatura,
    motivos: motivos.join(' | '),
    prioridade: temperatura === 'HOT' ? 'Alta' : temperatura === 'WARM' ? 'Média' : 'Baixa'
  };
}

// Processar cada item
return items.map(item => {
  const scoring = calcularScore(item.json);
  return {
    json: {
      ...item.json,
      ...scoring
    }
  };
});
```

---

## 5. Estrutura do Airtable

### Tabela: Leads_TimeLabs

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `nome` | Single line text | Nome da empresa |
| `telefone` | Phone | Telefone principal |
| `email` | Email | Email corporativo |
| `endereco` | Long text | Endereço completo |
| `bairro` | Single line text | Bairro em BH |
| `website` | URL | Site da empresa |
| `instagram` | URL | Link do Instagram |
| `linkedin` | URL | Link do LinkedIn |
| `whatsapp` | Phone | WhatsApp Business |
| `facebook` | URL | Link do Facebook |
| `categoria` | Single select | Segmento do negócio |
| `rating` | Number (1 decimal) | Nota no Google |
| `reviews` | Number | Quantidade de avaliações |
| `score` | Number | Score de qualificação (0-100) |
| `temperatura` | Single select | HOT/WARM/COOL/COLD |
| `prioridade` | Single select | Alta/Média/Baixa |
| `motivos_score` | Long text | Explicação do score |
| `data_coleta` | Date | Quando foi coletado |
| `status_contato` | Single select | Não contatado/Em progresso/Convertido |

---

## 6. Fluxo do Novo Workflow N8N

```
┌─────────────────────────────────────────────────────────────────┐
│                     WORKFLOW SIMPLIFICADO                        │
└─────────────────────────────────────────────────────────────────┘

[1] TRIGGER (Manual ou Agendado)
         │
         ▼
[2] CONFIGURAÇÃO
    • Lista de categorias
    • Cidade: Belo Horizonte
    • Max resultados: 20 por categoria
         │
         ▼
[3] GOOGLE MAPS SCRAPER (Apify)
    • Buscar empresas por categoria
    • Extrair: nome, telefone, endereço, site, rating
         │
         ▼
[4] FILTRAR EMPRESAS VÁLIDAS
    • Tem nome?
    • Está em BH?
    • Está aberta?
         │
         ▼
[5] TEM WEBSITE? ─────────────┐
    │                          │
    ▼ SIM                      ▼ NÃO
[6] CRAWLER DE WEBSITE    [7] PULAR CRAWLER
    • Extrair emails          • Manter dados básicos
    • Extrair redes sociais
    • Extrair WhatsApp
         │                     │
         └─────────┬───────────┘
                   ▼
[8] DESCOBERTA DE LINKEDIN
    • Gerar URL padrão: linkedin.com/company/{slug}
    • Verificar se existe (HTTP HEAD)
         │
         ▼
[9] DESCOBERTA DE INSTAGRAM
    • Buscar no HTML do site
    • Verificar padrão @handle
         │
         ▼
[10] CALCULAR SCORE (Code Node)
     • Aplicar lógica de pontuação
     • Classificar temperatura
         │
         ▼
[11] SALVAR NO AIRTABLE
     • Inserir ou atualizar registro
         │
         ▼
[12] NOTIFICAR HOT LEADS
     • Se score > 75, enviar email
```

---

## 7. Próximos Passos

### Imediato (Hoje)
1. ✅ Entender arquitetura (este documento)
2. 🔄 Importar novo workflow N8N
3. 🔄 Configurar credenciais (Apify, Airtable, OpenAI)

### Curto Prazo (Esta semana)
4. Criar base no Airtable com estrutura acima
5. Testar workflow com 1 categoria
6. Ajustar scoring conforme resultados

### Médio Prazo (Próximas semanas)
7. Adicionar mais categorias
8. Implementar deduplicação
9. Criar dashboard de métricas no Airtable

---

## 8. Custos Estimados (Mensal)

| Item | Uso Estimado | Custo |
|------|--------------|-------|
| Apify (Google Maps) | 2000 leads/mês | $10 |
| Apify (Crawler) | 2000 páginas/mês | $6 |
| OpenAI | 2000 leads/mês | $1 |
| Airtable | Free tier | $0 |
| N8N | Self-hosted | $0 |
| **TOTAL** | | **~$17/mês** |

---

## Contato e Suporte

Sistema desenvolvido para **TimeLabs** - Automação e IA
Documentação criada em: Janeiro 2026
