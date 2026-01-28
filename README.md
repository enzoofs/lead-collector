# Lead Collector - TimeLabs

Sistema de captacao e qualificacao de leads B2B para empresas de automacao e IA.

## Visao Geral

```
Google Maps → Website Scraping → Scoring → Airtable
     ↓              ↓               ↓
  Empresas    Instagram/LinkedIn   HOT/WARM/COLD
```

## Arquivos do Projeto

```
lead-collector/
├── docs/
│   ├── ARQUITETURA_SISTEMA.md   # Documentacao tecnica completa
│   └── GUIA_CONFIGURACAO.md     # Passo a passo de setup
├── workflows/
│   └── lead_collector_v2.json   # Workflow N8N otimizado
└── README.md
```

## Quick Start

1. **Configure Airtable** - Crie base com campos especificados
2. **Configure Apify** - Obtenha API token
3. **Importe workflow** - `workflows/lead_collector_v2.json` no N8N
4. **Configure credenciais** - Apify, Airtable, Email
5. **Execute** - Teste com 1 resultado, depois expanda

## Dados Coletados

| Campo | Fonte |
|-------|-------|
| Nome, Endereco, Telefone | Google Maps |
| Rating, Reviews | Google Maps |
| Email | Website da empresa |
| Instagram | Website da empresa |
| LinkedIn | Website + padrao URL |
| WhatsApp | Website da empresa |
| Score (0-100) | Algoritmo interno |

## Sistema de Scoring

- **HOT (80-100)**: Contato imediato
- **WARM (60-79)**: Contato em 48h
- **COOL (40-59)**: Nurturing
- **COLD (0-39)**: Descartar

## Custo Estimado

~$8-15/mes para 2000 leads

## Documentacao

- [Arquitetura Completa](docs/ARQUITETURA_SISTEMA.md)
- [Guia de Configuracao](docs/GUIA_CONFIGURACAO.md)

---

Desenvolvido para **TimeLabs** - Automacao e IA
