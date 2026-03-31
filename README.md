# Projeto Vendas — Veloura Beauty
![BOSS BI Framework](https://img.shields.io/badge/Powered%20by-BOSS%20BI%20Framework-black?style=for-the-badge&logo=powerbi&logoColor=yellow)  

## 📊 Visão Geral

Este projeto apresenta um **dashboard de análise de vendas desenvolvido em Power BI**, criado para monitorar a performance comercial da empresa fictícia **Veloura Beauty**.

A solução permite acompanhar indicadores estratégicos de vendas, identificar padrões de comportamento, avaliar a eficiência de produtos, vendedores e regiões, além de apoiar decisões estratégicas orientadas por dados.

🔎 **[Dashboard Interativo](https://app.powerbi.com/view?r=eyJrIjoiYjI5ZjFkOTEtNmI5YS00OGRlLThjMDQtNzliNzczNWY1MGJiIiwidCI6IjIzY2FjN2VlLWYxZDgtNDMzOS1hYTdiLTc4MWFhOWY5MjI1YiJ9)**  

---

## 🧠 Contexto do Problema

A operação logística da **TransFlow Logistics** enfrentava desafios no monitoramento de:

- pedidos entregues no prazo
- desempenho de motoristas
- controle de devoluções
- visibilidade sobre clientes e volume de pedidos

Essas limitações dificultavam a identificação rápida de problemas operacionais e impactavam diretamente a eficiência da cadeia logística e a experiência do cliente.

---

## 🎯 Abordagem Estratégica

Para resolver esses desafios, foi desenvolvida uma solução analítica utilizando **Power BI**, estruturada com **modelagem dimensional** e definição de indicadores estratégicos de performance.

O dashboard foi projetado para oferecer:

- leitura executiva clara
- monitoramento operacional detalhado
- navegação intuitiva entre métricas logísticas  

### KPIs principais

- Volume de Pedidos
- Volume de Produtos Transportados
- Base de Motoristas Ativos

---

## 🧠 Metodologia Aplicada — BOSS BI Framework

> Este projeto foi desenvolvido utilizando o BOSS BI Framework (Business-Oriented Smart Solutions), uma metodologia proprietária desenvolvida para estruturar projetos de Business Intelligence e Analytics, focada na geração de valor estratégico, consistência analítica e suporte à tomada de decisão.

## 🔷 Fluxo do BOSS BI Framework

```mermaid
flowchart LR
    A[Business Understanding] --> B[Data Understanding]
    B --> C[Data Preparation]
    C --> D[Data Modeling]
    D --> E[Analytical Exploration]
    E --> F[Data Visualization]
    F --> G[Insights & Decision Support]
    G --> H[Deployment]
    H --> I[Continuous Improvement]
    I --> A
```

## 📌 Detalhamento das Etapas

### 🔹 1. Business Understanding
Definição do problema analítico e alinhamento com os objetivos estratégicos do negócio, garantindo que a solução gere valor real e mensurável.

---

### 🔹 2. Data Understanding
Mapeamento das fontes de dados e análise inicial para compreensão da estrutura, qualidade e granularidade das informações disponíveis.

---

### 🔹 3. Data Preparation
Tratamento, limpeza e transformação dos dados, assegurando consistência, padronização e confiabilidade para análise.

---

### 🔹 4. Data Modeling
Estruturação do modelo de dados utilizando boas práticas de modelagem dimensional, com foco em performance e escalabilidade.

---

### 🔹 5. Analytical Exploration
Exploração dos dados para identificação de padrões, tendências, correlações e possíveis anomalias relevantes ao negócio.

---

### 🔹 6. Data Visualization
Desenvolvimento de dashboards e relatórios interativos, aplicando princípios de visualização e Data Storytelling.

---

### 🔹 7. Insights & Decision Support
Geração de insights acionáveis e recomendações estratégicas para apoiar a tomada de decisão baseada em dados.

---

### 🔹 8. Deployment
Publicação e disponibilização da solução analítica, garantindo acesso, atualização e governança dos dados.

---

### 🔹 9. Continuous Improvement
Monitoramento contínuo e evolução da solução, adaptando-se às mudanças e novas necessidades do negócio.

---

## 📈 Impactos e Resultados

A solução permite:

- identificar rapidamente **gargalos na operação logística**
- analisar **motoristas com maior índice de atraso**
- detectar **padrões de devoluções**
- monitorar **clientes com maior volume de pedidos**

Com isso, gestores conseguem tomar decisões mais rápidas e baseadas em dados.

---

## 🧩 Estrutura do Dashboard

### 📊 **Indicadores Principais**

O dashboard apresenta três cartões principais:

### Volume de Pedidos

- quantidade total de pedidos
- quantidade e percentual de entregas realizadas no prazo

### Volume de Produtos Transportados

- quantidade total de produtos transportados
- quantidade e percentual de produtos devolvidos

### Base de Motoristas Ativos

- quantidade total de motoristas ativos

---

## 📊 Visualizações Analíticas

### 🚚 **Performance de Entregas**

Gráfico de rosca mostrando:

- pedidos entregues no prazo
- pedidos entregues atrasados
- quantidade e percentual de cada categoria

---

### 👥 **Pedidos por Cliente**

Gráfico de barras horizontais que apresenta:

- quantidade de pedidos por cliente
- identificação dos clientes com maior volume de pedidos

---

### ⏱️ **Atrasos por Motorista**

Gráfico de barras horizontais mostrando:

- número de atrasos registrados por motorista
- identificação de padrões de performance

---

### 🔁 **Motivos de Devolução por Motorista**

Gráfico de barras verticais empilhadas que apresenta:

- percentual de motivos de devolução
- relação entre devolução e motorista responsável

---

### 📦 **Distribuição de Devoluções**

Gráfico **Treemap** exibindo:

- quantidade de devoluções por motivo
- impacto relativo de cada causa

---

## 🎛️ Filtros Interativos

O dashboard permite análise dinâmica por:

- **Ano**
- **Estado**

Esses filtros permitem explorar diferentes cenários operacionais.

---

## 🎨 Experiência de Navegação

O dashboard inclui recursos de usabilidade e design:

- 🌙 **Modo Dark (padrão)**
- ☀️ **Modo Light (opcional)**
- 🔎 botão **Analisar**
- 🏠 botão **Home**

Esses elementos melhoram a experiência de exploração dos dados.

---

## 🛠️ Stack Técnica

- Excel
- Microsoft Power BI
- Power Query
- DAX (Data Analysis Expressions)
- Modelagem Dimensional
- Storytelling com Dados
- PowerPoint

---

## 🧱 Modelagem de Dados

⭐ **Star Schema**

Neste projeto, foi adotado o modelo Star Schema como padrão de modelagem dimensional, priorizando performance analítica, simplicidade estrutural e eficiência no processamento de dados.  

A utilização de dimensões desnormalizadas e relacionamentos diretos com a tabela fato reduz a complexidade de junções, melhora a compressão de dados no mecanismo VertiPaq e garante maior previsibilidade no comportamento dos filtros e medidas.

Essa abordagem é amplamente recomendada em soluções de Business Intelligence, especialmente no Power BI, por proporcionar melhor desempenho e facilitar a construção de análises escaláveis e intuitivas.

### **Tabelas Fato**

- pedidos realizados

### **Tabelas Dimensão**

- calendário
- clientes
- motoristas
- destino
- status
- motivo de devolução

Com isso, a solução proporciona maior visibilidade operacional, permitindo a identificação proativa de gargalos, melhoria no desempenho logístico e suporte mais assertivo à tomada de decisão estratégica.

## 🗂️ Modelo de Dados

![Modelo de Dados](images/modelo-dados-veloura-beauty.png)  

O modelo foi projetado para suportar evolução futura, incluindo a possibilidade de integração com tabelas fato adicionais ou resolução de cenários de muitos-para-muitos por meio de tabelas bridge.

---

## 📸 Preview do Dashboard

![Dashboard Preview](images/vendas-veloura-beauty.png)

## Documentação das Medidas

Para consultar a documentação das medidas deste projeto, suas fórmulas e descrições, acesse a **[Documentação das Medidas](docs/medidas-documentacao.md)**.

## 👨‍💻 Autor

Projeto desenvolvido como parte do meu portfólio profissional em **Business Intelligence e Data Analytics**, destacando habilidades avançadas e aplicáveis a diversos cenários analíticos:

- Desenvolvimento de **dashboards executivos e painéis estratégicos**, focados em insights acionáveis e tomada de decisão baseada em dados  
- **Modelagem dimensional e relacional**, aplicando corretamente **cardinalidade, granularidade** e hierarquias entre tabelas para garantir consistência e integridade dos dados  
- **Transformação de dados com Power Query e Linguagem M**, criando pipelines eficientes, automatizados e auditáveis  
- Criação de **KPIs estratégicos e métricas customizadas em DAX**, para análise de performance e comparações confiáveis  
- **Integração de múltiplas fontes de dados** (Excel, SQL, APIs, arquivos planos), padronizando e validando informações críticas  
- **Data storytelling e visualizações interativas**, com cores, hierarquias, filtros e destaque de insights, para facilitar interpretação e engajamento do usuário  
- **Análises estatísticas e preditivas**, usando Python, R, regressões, teste de hipóteses, séries temporais e técnicas de Machine Learning para identificação de tendências e padrões  
- **Automatização e otimização de processos analíticos**, incluindo ETL, scripts e compressão de dados, garantindo performance e escalabilidade dos relatórios  
- **Documentação detalhada de medidas, tabelas, modelos e processos**, permitindo reprodutibilidade, transparência e governança dos dados  
- Aplicação de **boas práticas de engenharia de dados**, integrando análise, estatística, IA e visualização para soluções analíticas completas e confiáveis  
- Domínio completo de **Power BI, DAX, Power Query, Python e R**, com foco em performance, qualidade e entrega de insights estratégicos

---

<div align="center">
  
**Portfólio de Business Intelligence & Data Analytics**  

<img src="images/assinatura.png" width="400">

---

💼 Aberto a oportunidades em Business Intelligence & Data Analytics

| [LinkedIn](https://www.linkedin.com/in/rogério-clynton-ribeiro/) | [Portfólio](https://clyntonboss.github.io/) | [e-Mail](mailto:clyntonribeiror@gmail.com) | [WhatsApp](https://wa.me/5524999240768) |

</div>
