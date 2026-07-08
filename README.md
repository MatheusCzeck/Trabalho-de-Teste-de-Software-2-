# Estratégia de Qualidade e Governança em Plataformas de RH
## Documento: Plano de Testes Estratégico (Master Test Plan)
### Módulo de Geração de Laudos Comportamentais Automatizados — Plataforma SisTegra9

---

## 📑 Cabeçalho e Controle de Versões

* **Código de Identificação Corporativo:** `ST9-PLN-QA-2026-004-REV2`
* **Classificação de Segurança:** Altamente Confidencial (Restrito ao Conselho Diretor, Auditoria e Engenharia)
* **Sistema/Produto:** SisTegra9 — Plataforma Premium de Recrutamento e Seleção (R&S)
* **Módulo Alvo:** Motor de Inferência e Geração de Laudos Comportamentais Automatizados
* **Frameworks de Referência:** COBIT 2019 (Foco nos domínios APO e BAI) e ISO/IEC 29119 (Partes 1, 2, 3 e 4)

### Histórico da Revisão
| Versão | Data | Descrição da Alteração | Autor | Papel / Cargo | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1.0** | 08/07/2026 | Criação inicial do plano, definição do escopo de testes e alinhamento COBIT/ISO. | Engenharia de QA | Liderança de Análise de Sistemas e QA | *Substituído* |
| **1.1** | 08/07/2026 | Detalhamento exaustivo de cenários de arquitetura (Node.js/PostgreSQL), matrizes de risco e especificações técnicas de artefatos. | Engenharia de QA & Governança | Liderança de Análise de Sistemas e QA | **Aprovado** |

---

## 1. Introdução e Visão Geral do Projeto

### 1.1 Objeto de Teste (Resumo Executivo)
O objeto deste plano de testes é o **Módulo de Geração de Laudos Comportamentais Automatizados**, o componente mais crítico da arquitetura do **SisTegra9**. Este módulo processa as respostas fornecidas pelos candidatos em testes psicológicos estruturados (ex: mapeamento de competências, testes de perfil DISC e inventários de liderança), aplicando um algoritmo proprietário de inferência para consolidar essas respostas em relatórios analíticos de alta fidelidade técnica.

A arquitetura do ecossistema é composta por:
1. **Front-end (React):** Uma interface corporativa responsiva e com alta exigência de acessibilidade, responsável por capturar as interações do usuário em tempo real e renderizar gráficos dinâmicos e vetoriais (SVG) com os resultados comportamentais.
2. **Back-end (Node.js):** Uma API RESTful assíncrona orientada a eventos, responsável pelo processamento pesado, computação dos scores de aptidão e coordenação das regras de negócio.
3. **Banco de Dados (PostgreSQL):** Camada de persistência relacional responsável por armazenar os pesos das questões, o histórico de respostas, os logs de auditoria e os laudos gerados em formato JSONB estruturado.

O ciclo de validação detalhado neste documento foi concebido para auditar exaustivamente o fluxo de ponta a ponta, garantindo que o processamento matemático das regras de negócio seja imune a falhas técnicas de concorrência ou lógica. Sob a ótica da **ISO/IEC 29119**, o processo estabelece repetibilidade e robustez; sob as diretrizes do **COBIT**, atua como um controle corporativo rígido que mitiga o risco de desvios algorítmicos ou tomada de decisão equivocada em processos seletivos de alta liderança.

### 1.2 Objetivo do Documento
Este documento serve como a **Evidência Formal de Homologação e Mitigação de Riscos** exigida pelo conselho diretor do SisTegra9. Seus objetivos específicos são:
* **Prevenção de Riscos de Reputação:** Garantir que nenhum candidato receba um perfil comportamental incorreto devido a problemas de truncamento de dados, falhas de rede ou estouro de memória no back-end.
* **Prevenção de Riscos Financeiros e Jurídicos:** Blindar a plataforma e seus clientes corporativos contra processos judiciais decorrentes de discriminação algorítmica ou falhas de conformidade com as leis de proteção de dados sensíveis (LGPD/GDPR).
* **Alinhamento de Expectativas:** Unificar a linguagem entre o time de engenharia técnica (desenvolvedores, DevOps e testadores) e as partes interessadas de negócio (Diretores de RH, psicólogos organizacionais e auditores de conformidade), traduzindo métricas de software em indicadores de governança corporativa.

---

## 2. Alinhamento Estratégico com o Framework COBIT

A engenharia de qualidade do SisTegra9 não opera isolada; ela é desenhada para retroalimentar o Sistema de Governança Corporativa, transformando a atividade de testes em um centro gerador de conformidade e mitigação de riscos.

### 2.1 Objetivos COBIT Suportados (Contextualização Prática)

#### A. Redução de Riscos Operacionais e Tecnológicos (Domínio APO12 - Gerenciar Riscos)
Em plataformas de RH de alto volume, falhas técnicas sutis e não tratadas podem gerar impactos catastróficos. A tabela abaixo contextualiza como nossa estratégia de testes mapeia cenários reais de engenharia para evitar riscos de governança:

| Cenário de Risco Técnico | Impacto na Regra de Negócio | Estratégia de Mitigação no Ciclo de Testes |
| :--- | :--- | :--- |
| **Estouro de Concorrência no PostgreSQL:** Múltiplos candidatos salvando o teste exatamente no mesmo milissegundo, gerando condições de corrida (*race conditions*). | O banco pode registrar respostas trocadas ou falhar ao computar o peso de uma questão, gerando um laudo com o perfil comportamental de outro candidato ou corrompendo o *score* final. | **Testes de Carga e Concorrência:** Simulação automatizada via *k6* com mais de 5.000 usuários virtuais simultâneos batendo nos endpoints de escrita do PostgreSQL. Validação de isolamento de transações (`SERIALIZABLE`) para garantir o *rollback* automático e seguro caso haja conflito de dados. |
| **Tipagem Dinâmica Incorreta no Node.js:** Falha na conversão de strings para floats em cálculos matemáticos de percentis comportamentais. | Um candidato com alta aptidão em "Foco em Resultados" (ex: 92.5%) pode ter seu valor truncado para zero ou NaN, resultando em uma eliminação injusta do processo seletivo. | **Testes Unitários e Mutação:** Cobertura de 100% das funções do motor de inferência com *Jest* e testes de mutação com *Stryker*. Validação estrita de esquemas de dados com a biblioteca *Zod* em todos os contratos de entrada e saída da API. |
| **Erros de Desserialização no React:** O front-end falha ao parsear um objeto JSON complexo vindo do PostgreSQL contendo o mapeamento de competências. | O gráfico de competências do candidato é exibido de forma distorcida, em branco, ou omitindo traços críticos de liderança na tela do Diretor de RH. | **Testes de Integração de UI Front-end:** Implementação de testes baseados em contratos usando *Pact*, garantindo que qualquer alteração de esquema no back-end que quebre o front-end seja capturada no pipeline de CI/CD antes do deploy. |

#### B. Garantia de Conformidade e Transparência (Domínio MEA03 - Gerenciar Conformidade com Requisitos Externos)
Os laudos comportamentais lidam com dados altamente sensíveis de indivíduos. O ciclo de testes audita as regras de privacidade e conformidade legal:
* **Direito à Explicação:** Testes específicos de caixa-branca validam que o sistema mantém um log determinístico e auditável que explica exatamente o porquê daquele score ter sido gerado, permitindo que a plataforma atenda ao direito de revisão humana previsto na LGPD.
* **Segurança de Dados em Repouso e em Trânsito:** Varreduras de vulnerabilidades automatizadas (SAST/DAST) são integradas ao pipeline de testes para assegurar que os campos JSONB de laudos comportamentais no PostgreSQL estejam blindados contra injeções e vazamentos.

### 2.2 Geração de Valor para a Governança (Processo COBIT BAI07 - Gerenciar Aceitação e Transição de Mudanças)
O processo **BAI07** visa assegurar que os novos lançamentos de software ocorram de maneira controlada, previsível e sem impactos negativos na operação contínua do negócio. A nossa estratégia de qualidade gera valor direto para este processo através da aplicação de uma **Matriz de Rastreabilidade Bidirecional**.

```
[Requisito de RH] <=======> [História de Usuário / Regra] <=======> [Casos de Teste (QA)] <=======> [Métricas de Defeitos]
```

Esta correlação sistemática garante que:
1. **Transparência na Transição:** O comitê de liberação (*Release Gate*) possui um painel claro contendo o percentual exato de cobertura de testes de cada requisito regulatório.
2. **Análise de Impacto Rápida:** Caso um defeito seja reportado em ambiente de produção, a governança consegue rastrear instantaneamente quais regras de negócio correlacionadas estão sob risco, isolando o impacto operacional e permitindo acionar planos de contingência em minutos.
3. **Auditoria de Conformidade:** Sempre que um auditor questionar os critérios de validação do módulo, o SisTegra9 consegue exportar relatórios de execução de testes matematicamente amarrados aos requisitos de negócio correspondentes.

---

## 3. Conformidade com a ISO/IEC 29119

A adoção da norma internacional **ISO/IEC 29119** estabelece uma estrutura formal de processos de teste que substitui o empirismo técnico por engenharia previsível de alta performance.

### 3.1 Benefícios da Conformidade e Engenharia de Qualidade do Produto
Para uma plataforma classificada como *Premium*, a qualidade não pode ser um subproduto do acaso. A aderência à norma traz os seguintes benefícios estruturais para o SisTegra9:

* **Abordagem de Teste Baseada em Riscos (Risk-Based Testing):** Otimização de tempo e recursos. Concentramos os esforços de teste de regressão automatizados nos fluxos críticos (motor de cálculo do score) enquanto fluxos secundários (ex: exportação de relatórios secundários) seguem rotinas padrão, maximizando o ROI do time de QA.
* **Repetibilidade e Consistência:** Independentemente de quem seja o engenheiro executando a atividade, os procedimentos de teste seguem uma padronização universal (Definição, Especificação, Execução e Encerramento). Isso garante que novos desenvolvedores ou auditores externos consigam ler, compreender e replicar qualquer cenário de validação.
* **Redução do Custo de Falha:** Identificar desvios lógicos nas fases iniciais do ciclo de vida de desenvolvimento (através de revisões estáticas de requisitos alinhadas com a ISO 29119-2) reduz em até 40% o custo de correção de defeitos em comparação com a correção tradicional pós-implantação.

### 3.2 Detalhamento dos Artefatos Previstos (Entregáveis Oficiais de Engenharia)

Em total conformidade com a documentação prevista na **ISO/IEC 29119-3**, os seguintes artefatos técnicos serão obrigatoriamente produzidos, armazenados de forma imutável e vinculados ao pipeline de implantação:

#### A. Especificação de Casos de Teste (Exemplos de Padrão Técnico)

##### Identificador do Caso de Teste: `ST9-TC-REACT-012`
* **Componente:** `BehavioralChartRenderer.jsx` (Front-end React)
* **Objetivo:** Validar a resiliência da interface de usuário e a renderização correta de gráficos de competências comportamentais diante de payloads massivos ou corrompidos sem causar congelamento da aplicação (*App Crash*).
* **Pré-condições:** Usuário autenticado com perfil de Diretor de RH acessando a tela de visualização de laudos.
* **Procedimento de Execução:**
    1. Injetar (via interceptação de rede no Cypress) um payload JSON simulando um candidato com 50 competências comportamentais atípicas calculadas simultaneamente (cenário de estresse de dados).
    2. Acionar a renderização do gráfico vetorial.
    3. Verificar a legibilidade do texto e a ausência de sobreposição de elementos na UI usando testes de regressão visual automatizados (*Percy/Applitools*).
* **Resultado Esperado:** A interface deve adaptar dinamicamente o tamanho do gráfico usando paginação interna ou scroll acessível, sem estourar as margens da página ou exibir erros do tipo `Uncaught TypeError` ou tela branca no React.

##### Identificador do Caso de Teste: `ST9-TC-NODE-089`
* **Componente:** `ScoreCalculationEngine.js` (Back-end Node.js)
* **Objetivo:** Validar o cálculo preciso do percentil comportamental de liderança sob flutuação extrema de dados numéricos no PostgreSQL.
* **Pré-condições:** Massa de dados de testes comportamentais inserida na tabela `st9_candidate_responses`.
* **Procedimento de Execução:**
    1. Executar a chamada à API passando o `candidate_id` com valores de respostas limítrofes (todas as respostas zeradas vs. todas as respostas no valor máximo).
    2. Avaliar o tempo de resposta do endpoint (SLA esperado < 200ms).
    3. Comparar o valor retornado na API com o valor pré-calculado pela equipe estatística de psicologia através de asserções matemáticas rígidas.
* **Resultado Esperado:** O cálculo deve bater com exatidão de 4 casas decimais. Valores nulos ou inválidos devem acionar uma exceção tratada (`422 Unprocessable Entity`), registrando o evento no log do PostgreSQL sem expor segredos internos do sistema.

#### B. Matrizes de Rastreabilidade Baseadas na ISO/IEC 29119
Para assegurar a auditabilidade e comprovar a eficácia dos testes frente aos órgãos reguladores e ao conselho diretor, o projeto manterá a seguinte matriz operacionalizada dinamicamente via Jira/Xray:

| ID do Requisito de Negócio (RH) | Descrição do Requisito de Negócio | ID do Caso de Teste Associado | Tipo de Teste (ISO 29119-4) | Status da Automação | Módulo de Código Afetado |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-DISC-01** | O sistema deve calcular matematicamente a dominância, influência, estabilidade e conformidade do candidato com base em pesos dinâmicos. | `ST9-TC-NODE-089` | Teste Unitário / Regras de Negócio | 100% Automatizado (Jest) | `/services/inferenceEngine.js` |
| **REQ-DISC-02** | O laudo comportamental final deve ser gerado em formato PDF imutável e assinado digitalmente. | `ST9-TC-NODE-095` | Teste de Integração / Segurança | Automatizado (Supertest / PostgreSQL) | `/controllers/reportController.js` |
| **REQ-UX-05** | Gráficos do perfil comportamental devem ser acessíveis para leitores de tela e visualmente claros em resoluções Web e Mobile. | `ST9-TC-REACT-012` | Teste Funcional de UI / Regressão Visual | Automatizado (Cypress + Percy) | `/components/charts/` |
| **REQ-SEC-09** | Dados comportamentais sensíveis de candidatos reprovados devem seguir a política de expurgo automático de dados da LGPD. | `ST9-TC-DB-004` | Teste de Banco de Dados / Auditoria | Automatizado (PGTap / Scripts SQL) | `/db/migrations/purging_policy.sql` |

---
*Este documento estratégico é um ativo de propriedade intelectual corporativa do SisTegra9. Qualquer alteração ou desvio nos processos aqui descritos deve ser previamente aprovado pelo Comitê de Governança, Riscos e Conformidade (GRC).*
