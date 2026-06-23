# Análise de Satisfação de Clientes - Olist E-commerce

Análise exploratória dos dados públicos da Olist com foco em satisfação do cliente, tempo de entrega e desempenho por categoria de produto.

---

## Sobre o projeto

A Olist é uma plataforma brasileira de e-commerce que conecta pequenos lojistas a grandes marketplaces. Este projeto analisa **99.225 avaliações de clientes** para identificar os principais fatores que influenciam a satisfação.

**Ferramentas utilizadas:** Databricks, SQL, Delta Lake  
**Arquitetura:** Medallion (bronze → silver → gold)  
**Dados:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) - Kaggle

---

## Principais insights

### 1. Tempo de entrega é o principal fator de satisfação

Clientes que deram nota 5 receberam seus pedidos em média **10.6 dias** e **13.4 dias antes do prazo**. Já clientes que deram nota 1 esperaram **21.3 dias** e o pedido chegou apenas **4.1 dias antes do prazo**.

| Nota | Total pedidos | Média dias entrega | Dias antes do prazo |
|------|--------------|-------------------|---------------------|
| 1    | 9.409        | 21.3 dias         | 4.1 dias            |
| 2    | 2.941        | 16.6 dias         | 8.6 dias            |
| 3    | 7.962        | 14.2 dias         | 10.8 dias           |
| 4    | 18.987       | 12.3 dias         | 12.4 dias           |
| 5    | 57.060       | 10.6 dias         | 13.4 dias           |

> Conclusão: não é apenas sobre atrasar, chegar bem antes do prazo é o que gera nota máxima.

---

### 2. Categorias de móveis concentram as piores avaliações

As 5 categorias com pior média de satisfação (mínimo 100 reviews):

| Categoria          | Total reviews | Nota média |
|--------------------|--------------|------------|
| office_furniture   | 1.687        | 3.49       |
| fashion_male_clothing | 131       | 3.64       |
| fixed_telephony    | 262          | 3.68       |
| home_confort       | 435          | 3.83       |
| audio              | 361          | 3.83       |

> Conclusão: móveis e produtos grandes têm maior risco de danos no transporte e atrasos, explicando as notas mais baixas.

---

### 3. Eventos sazonais impactam diretamente a satisfação

A média geral da plataforma é **4.09**, mas meses de alto volume ou feriados prolongados registram quedas expressivas:

- **Dezembro 2017:** nota 3.95 com 7.668 reviews —> reflexo da Black Friday e natal
- **Março 2018:** pior mês do período com nota **3.76** e 7.475 reviews —> impacto logístico do Carnaval, com pedidos feitos em fevereiro chegando atrasados

> Conclusão: a logística não acompanha os picos de demanda, e feriados prolongados agravam os atrasos.

---

## Arquitetura do projeto

```
bronze (dado bruto)
│
├── bronze_orders
├── bronze_reviews
├── bronze_order_items
├── bronze_products
└── bronze_category_translation
        │
        ▼
silver (dado limpo)
│
├── silver_orders        → datas convertidas para TIMESTAMP
└── silver_reviews       → review_score convertido para INT, datas limpas
        │
        ▼
gold (dado agregado)
│
├── gold_delivery_vs_satisfaction   → atraso por nota
├── gold_satisfaction_by_category   → nota média por categoria
└── gold_satisfaction_over_time     → evolução mensal da satisfação
```

---

## Como reproduzir

1. Baixe os dados em [kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
2. Crie uma conta gratuita no [Databricks Community Edition](https://community.cloud.databricks.com)
3. Suba os CSVs via **Data → Add data → Upload file**
4. Execute os notebooks na ordem: `bronze → silver → gold`

---

## Aprendizados técnicos

- Uso de `TRY_CAST` para tratar dados sujos sem quebrar a pipeline
- Arquitetura Medallion para organizar camadas de dados
- `DATEDIFF` para calcular tempo entre eventos
- `COALESCE` para fallback entre colunas
- `HAVING` para filtrar grupos após agregação
- `LEFT JOIN` para preservar registros sem correspondência
