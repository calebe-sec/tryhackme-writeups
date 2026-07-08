# TryHackMe — MITRE ATT&CK: Write-Up

**Author:** [Calebe Araújo]
**Platform:** TryHackMe
**Room:** MITRE (ATT&CK, Navigator, CAR, D3FEND, AADAPT, ATLAS)
**Category:** Threat Intelligence / Blue Team
**Difficulty:** Easy–Medium

---

## Table of Contents

1. [Overview](#overview)
2. [Task 1 — Introdução ao MITRE ATT&CK](#task-1--introdução-ao-mitre-attck)
3. [Task 3 — Investigando um Grupo APT](#task-3--investigando-um-grupo-apt)
4. [Task 4 — Encontrando Grupos por Setor](#task-4--encontrando-grupos-por-setor)
5. [Task 5 — MITRE CAR](#task-5--mitre-car)
6. [Task 6 — MITRE D3FEND](#task-6--mitre-d3fend)
7. [Task 7 — Outros Projetos MITRE](#task-7--outros-projetos-mitre)
8. [Key Takeaways](#key-takeaways)

---

## Overview

Este write-up documenta a exploração do ecossistema de frameworks mantidos pela MITRE, com foco no ATT&CK Matrix, no ATT&CK Navigator, no Cyber Analytics Repository (CAR), no D3FEND e em projetos complementares como AADAPT e ATLAS.

O objetivo da sala é desenvolver a capacidade de navegar por esses frameworks para mapear táticas, técnicas, sub-técnicas, grupos de ameaça (APTs), softwares associados, mitigações e estratégias de detecção — habilidades centrais para funções de SOC, Threat Intelligence e Detection Engineering.

---

## Task 1 — Introdução ao MITRE ATT&CK

### Objetivo

Entender como o framework ATT&CK é organizado e como localizar táticas, técnicas e sub-técnicas.

### Conceitos

O ATT&CK organiza o comportamento dos atacantes em três níveis hierárquicos:

| Nível | Definição |
|---|---|
| **Tactic** | O objetivo do atacante ("por quê?") |
| **Technique** | Como esse objetivo é alcançado |
| **Sub-technique** | Uma forma específica de executar a técnica |

As táticas aparecem na parte superior da matriz, enquanto as técnicas ficam listadas abaixo de cada uma delas.

### Questão 1 — Tactic da técnica Hide Artifacts

**Como encontrei:**
1. Acessei a matriz do ATT&CK.
2. Pesquisei pela técnica *Hide Artifacts*.
3. Abri a página da técnica.
4. No topo da página, no campo **Tactic**, a informação estava disponível.

**Resposta:**
```
Defense Evasion
```

### Questão 2 — ID da técnica Create Account

**Como encontrei:**
1. Pesquisei por *Create Account* no ATT&CK.
2. Abri a página da técnica.
3. Logo abaixo do nome, o identificador da técnica é exibido.

**Resposta:**
```
T1136
```

---

## Task 3 — Investigando um Grupo APT

### Objetivo

Aprender como investigar um grupo de ameaça (APT) utilizando o ATT&CK Navigator.

### Questão 1 — País de origem do Mustang Panda

**Como encontrei:**
1. Acessei a página do grupo **Mustang Panda (G0129)**.
2. A descrição inicial do grupo informa sua origem.

**Resposta:**
```
China
```

### Questão 2 — Técnica de Reconnaissance do Mustang Panda

**Como encontrei:**
1. Entrei na página do grupo Mustang Panda.
2. Abri a matriz (Navigator Layer).
3. Localizei a coluna **Reconnaissance**.
4. Observei quais técnicas estavam destacadas e cliquei na correspondente.

**Resposta:**
```
T1598
```

### Questão 3 — Software usado para Access Token Manipulation

**Como encontrei:**
1. Na página do Mustang Panda, procurei a técnica **Access Token Manipulation**.
2. Na seção **Procedure Examples**, o MITRE lista os softwares utilizados pelo grupo.

**Resposta:**
```
Cobalt Strike
```

---

## Task 4 — Encontrando Grupos por Setor

### Objetivo

Aprender a pesquisar grupos APT utilizando os filtros disponíveis no ATT&CK.

### Questão 1 — Grupo ativo desde 2013 que ataca o setor de aviação

**Como encontrei:**
1. Acessei a página **Groups**.
2. Procurei por grupos relacionados ao setor **Aviation**.
3. Verifiquei a descrição de cada grupo até encontrar a correspondência.

**Resposta:**
```
APT33
```

### Questão 2 — Sub-técnica de preocupação para ambientes Office 365

**Como encontrei:**
1. Abri a página do APT33.
2. Analisei sua matriz de técnicas.
3. Procurei técnicas relacionadas ao Microsoft 365.

**Resposta:**
```
Cloud Accounts
```

### Questão 3 — Ferramenta associada ao grupo e à sub-técnica

**Como encontrei:**
1. Cliquei na sub-técnica **Cloud Accounts**.
2. Fui até **Procedure Examples**.
3. Localizei o grupo **APT33** e observei o software utilizado.

**Resposta:**
```
Ruler
```

### Questão 4 — Mitigação para remoção de contas inativas

**Como encontrei:**

Na página da técnica, seção **Mitigations**.

**Resposta:**
```
User Account Management
```

### Questão 5 — Detection Strategy ID

**Como encontrei:**

Na página da técnica, seção **Detections**.

**Resposta:**
```
DET0546
```

---

## Task 5 — MITRE CAR

### Objetivo

Conhecer o **Cyber Analytics Repository (CAR)**, repositório de analytics de detecção mantido pela MITRE.

### Questão 1 — Tactic associada ao CAR-2019-07-001

**Como encontrei:**
1. Abri o analytic **CAR-2019-07-001**.
2. No topo da página consta a associação com o ATT&CK.

**Resposta:**
```
Defense Evasion
```

### Questão 2 — Analytic Type de Access Permission Modification

**Como encontrei:**
1. Acessei a lista de Analytics.
2. Pesquisei **Access Permission Modification**.
3. Observei a coluna **Analytic Type**.

**Resposta:**
```
Situational Awareness
```

---

## Task 6 — MITRE D3FEND

### Objetivo

Aprender como o D3FEND complementa o ATT&CK, apresentando técnicas defensivas contrapostas às técnicas ofensivas.

### Questão 1 — Sub-técnica de User Behavior Analysis

**Como encontrei:**
1. Pesquisei **User Behavior Analysis**.
2. Abri a técnica e analisei suas sub-técnicas.

**Resposta:**
```
User Geolocation Logon Pattern Analysis
```

### Questão 2 — Artefato digital analisado pela sub-técnica

**Como encontrei:**

Na mesma página, seção **Digital Artifact Relationships**.

**Resposta:**
```
Network Traffic
```

---

## Task 7 — Outros Projetos MITRE

### Questão 1 — Technique ID de Scrape Blockchain Data (AADAPT)

**Como encontrei:**
1. Abri o framework **AADAPT** (voltado a ativos digitais/blockchain).
2. Pesquisei por **Scrape Blockchain Data**.

**Resposta:**
```
ADT3025
```

### Questão 2 — Tactic de LLM Prompt Obfuscation (ATLAS)

**Como encontrei:**
1. Abri o framework **ATLAS** (ameaças a sistemas de IA).
2. Pesquisei **LLM Prompt Obfuscation** e observei o campo **Tactic**.

**Resposta:**
```
Defense Evasion
```

---

## Key Takeaways

| Framework | Foco | Uso principal |
|---|---|---|
| **ATT&CK** | TTPs de adversários reais | Base comum de linguagem entre analistas |
| **Navigator** | Visualização de grupos/técnicas | Análise de cobertura defensiva |
| **CAR** | Analytics de detecção | Criação de regras para SIEM (ex.: Splunk) |
| **D3FEND** | Técnicas defensivas | Mapeamento de contramedidas às TTPs do ATT&CK |
| **AADAPT** | Ativos digitais / blockchain | Extensão de cobertura para ameaças financeiras digitais |
| **ATLAS** | Sistemas de IA | Ameaças direcionadas a modelos e pipelines de ML |

> **Nota:** Dominar o ecossistema MITRE é essencial para quem atua ou pretende atuar em SOC e Threat Intelligence — esses frameworks fornecem uma linguagem comum entre analistas, facilitam a criação de regras de detecção, apoiam a resposta a incidentes e permitem mapear ameaças reais de forma estruturada.

---

*Write-up produced as part of an ongoing offensive/defensive security portfolio. All exercises were conducted within the TryHackMe learning platform's isolated lab environments.*
