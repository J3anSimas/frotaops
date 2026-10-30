# Pesquisa: funcionalidades do Módulo 1 (Fleet)

> Pesquisa de apoio ao [ROADMAP.md](../../ROADMAP.md), Módulo 1. Escopo: **cadastro e gestão de frota** (vehicle registry) de uma empresa de ônibus brasileira **fictícia**. Manutenção, escala e fretamento ficam de fora, exceto nas dependências entre módulos.
> Data de acesso das fontes: 2026-10-30. Citações no formato [n], listadas na seção 7. Tudo marcado como **não verificado** não foi confirmado na fonte dona.

## 1. Resumo

- O ROADMAP cobre bem o núcleo do Fleet: identificação, hodômetro com troca, máquina de estados e painel. As lacunas estão em **validação de identificadores**, **acessibilidade estruturada**, **documentos com vencimento** e **ciclo de vida/baixa**.
- A placa **não é imutável**. A Res. CONTRAN 969/2022 obriga a troca para o padrão Mercosul na mudança de categoria ou de município, e o cadastro precisa guardar a "placa anterior" [2]. Por isso, a identidade do veículo deve ser o prefixo ou um id interno, não a placa.
- Todo ônibus com mais de 10 lugares precisa de cronotacógrafo (CTB art. 105, II) [1], e o Inmetro exige verificação a cada 2 anos [8]. Esse é um documento com vencimento que vale para a frota inteira e deveria entrar antes do M1.5.
- "Acessível" não é um booleano. A NBR 14022 define 3 tipos de acesso, box para cadeira de rodas e ≥10% de assentos preferenciais [15]. O Planning e o GTFS [18] precisam de um derivado simples (sim/não), calculado a partir desses campos.
- Para fretamento interestadual (ANTT), não há limite de idade para ônibus. O limite de 15 anos vale só para micro-ônibus, e ônibus com mais de 15 anos passam a ter inspeção semestral [11]. A idade máxima de frota urbana é definida por contrato ou município, então deve ser um parâmetro.
- Recomendo **antecipar documentos e vencimentos (M1.5) para P1** e definir já no M1.2 a **média km/dia** (janela de 90 dias corridos), porque o Maintenance depende dela.
- Pneus, multas e abastecimento ficam fora do Fleet. A seção 3 justifica essa decisão.

## 2. Funcionalidades identificadas

Legenda de "Já no ROADMAP?": a seção citada é sempre do Módulo 1, salvo indicação.

| Funcionalidade | Descrição | Já no ROADMAP? | Fonte(s) | Prioridade |
|---|---|---|---|---|
| Prefixo (nº de frota) | Identificador operacional único e estável. É a identidade do veículo, porque a placa pode mudar. | sim (Entities: *fleet number*) | prática do setor; [2] (placa mutável) | P0 |
| Placa (PNU e Mercosul) | Aceitar os formatos `AAA9999` (PNU) e `LLLNLNN` (PIV Mercosul), além do histórico de placas com "placa anterior" pesquisável. | parcial (*license plate (unique)*, sem formato nem histórico) | [2] art. 2º §3, art. 56, art. 57 §1, Anexo I item 1.2 | P0 |
| Chassi/VIN | 17 caracteres em 3 seções (WMI/VDS/VIS). A norma brasileira **não exige dígito verificador**, então a validação deve ser de formato e não deve bloquear pelo DV. | parcial (*chassis/VIN (unique)*) | [3] art. 3º I, art. 5º | P0 |
| Nº da carroceria (NICV) | Ônibus têm identificador próprio da carroceria: o NICV, com 21 caracteres, atribuído pelo encarroçador. Campo opcional. | não | [3] art. 3º VIII, art. 4º V | P1 |
| RENAVAM | 11 dígitos. Códigos antigos de 9 dígitos recebem zeros à esquerda. DV por módulo 11 (**algoritmo não verificado em fonte primária**). | parcial (*registration number (RENAVAM)*, sem validação) | [24], [25] | P0 |
| Categoria do registro | Particular ou aluguel. O fretamento ANTT exige categoria aluguel, e a mudança de categoria gera novo CRV e nova placa. | não | [11] art. 15 par. único; [1] art. 123; [2] art. 56 §1 I a | P0 |
| Características técnicas | Marca/modelo do chassi e da carroceria, ano de fabricação e ano-modelo, combustível, tipo, capacidade sentados/em pé, ar-condicionado, nº de portas, PBT. | parcial (faltam AC, portas, PBT e classificação NBR) | [16] (classes NBR 15570) | P0 (AC, portas: P1) |
| Classificação NBR 15570 | Micro, mini, midi, básico, padron, articulado e biarticulado. Complementa o *type* do ROADMAP (urban/coach/minibus/school). | não | [16] (**valores não verificados na norma**) | P1 |
| Acessibilidade detalhada | Tipo de acesso (a: piso baixo; b: piso alto com plataforma de embarque; c: piso alto com plataforma elevatória), nº de boxes para cadeira de rodas, nº de assentos preferenciais, comunicação audiovisual e SIA. O flag "acessível" é derivado desses campos. | parcial (*wheelchair lift, low floor, wheelchair space*) | [15] seção 6, 6.2, 6.3, 7.2; [13] art. 38; [14] art. 48; [17]; [18]; [19] | P0 |
| Hodômetro monotônico | Leituras imutáveis, com origem, autor e alerta de salto anômalo. | sim (Business rules; M1.2) | [29] (fornecedor: validação contra leituras vizinhas e "void") | P0 |
| Troca de hodômetro | Registra o km final da unidade antiga e o km inicial da nova, preservando o km acumulado. | sim (M1.2) | [29] (fornecedor: reset de medidor) | P0 |
| Média km/dia | Janela, mínimo de amostras e fallback para veículo novo. Alimenta a preventiva. | parcial (pergunta aberta) | sem fonte normativa; ver seção 5 | P0 |
| Horímetro | Medidor secundário em horas (motor ou equipamento), com as mesmas regras do hodômetro. | não | [29] (fornecedor: *secondary meter*, unidade horas) | P2 |
| Máquina de estados | `in_service ⇄ reserve`, `→ in_maintenance`, `→ retired`, com timeline. | sim (M1.3) | [26] (fornecedor: status configurável) | P0 |
| Status em uma data | Consulta "qual era o status do veículo em D", usada pela escala de datas futuras e pelas métricas. | não (implícito em Module 3 §3.2) | ROADMAP Module 3 | P0 |
| Documentos com vencimento | CRLV/licenciamento, seguro RC, CSV/inspeção, certificado de verificação do cronotacógrafo, autorização de escolar, registro ANTT. | parcial (*Documents (later)*, M1.5) | [1] arts. 105, 130, 131, 136; [8]; [11] arts. 11, 16, 44 | P1 |
| Licenciamento por final de placa | Prazo-limite anual derivado do último dígito da placa. A UF pode antecipar. | não | [4]; [5] (tabela); [1] arts. 130, 131 §2, 230 V | P1 |
| Alertas de vencimento | Janela configurável (ex.: 30/15/7 dias), painel "a vencer/vencidos" e situação bloqueante ou de aviso. | parcial (*expiry alerts*, M1.5) | [28] (fornecedor: tipos de renovação) | P1 |
| Idade da frota | Idade pelo **ano de fabricação**, idade média por tipo e por garagem. | parcial (Fleet panel: *average age*) | [1] art. 114; [11] art. 15 | P0 (cálculo) / P1 (painel) |
| Idade máxima regulatória/contratual | Limite parametrizável por contexto: ANTT fretamento (micro-ônibus ≤15 anos), contrato municipal e internacional. Alerta quando o veículo se aproxima do limite. | não | [11] arts. 15, 16; [12] (**secundária**) | P1 |
| Vida útil de referência (ULB) | Anos e/ou km esperados por tipo, mais o KPI "% da frota que atingiu ou superou a vida útil". | não | [20] §625.5; [22] §625.43; [23] | P1 |
| Baixa (retirement) com motivo | Motivo (venda, sinistro/perda total, fim de vida útil, devolução de leasing, desmontagem), data, km final e valor. Prazos legais de comunicação. | parcial (*retired requires reason and date*) | [1] arts. 126, 134; [27] (fornecedor: *out-of-service date/meter*) | P1 |
| Data e km de entrada em serviço | Baseline para a preventiva e para a vida útil. | parcial (Module 2: "service entry date and initial km") | [27] (fornecedor) | P0 |
| Transferência entre garagens | Histórico de garagem com data efetiva. Não confundir com mudança de município de registro, que exige nova placa. | parcial (campo *depot* sem histórico) | [2] art. 56 §1 II; [1] art. 123 | P1 |
| Alocação/lotação | "Lotação" aqui tem dois sentidos: (a) capacidade legal (CRLV, art. 137) e (b) vínculo preferencial a linha ou contrato. | parcial (a: *capacity*; b: não) | [1] art. 137 | (a) P0 / (b) fora (Planning) |
| Histórico/auditoria | Toda alteração cadastral registrada (quem, quando, antes/depois). | parcial (Module 0: *Audit trail*) | ROADMAP Module 0 | P0 |
| Importação em lote de veículos | Planilha de cadastro inicial com validação linha a linha. O ROADMAP prevê importação só para hodômetro. | não (só *Odometer entry, bulk*) | [28] (fornecedor: import de renovações) | P1 |
| Anexos/fotos | PDF do CRLV-e, apólice, certificados e fotos do veículo. | não | [1] art. 131 (CRLV digital) | P1 |
| Equipamentos embarcados | Validador, câmeras, GPS/telemetria, cronotacógrafo, AC: nº de série, data de instalação/remoção. | não | [1] art. 105 II; [30], [31] (fornecedor) | P2 (tacógrafo como documento: P1) |
| Pneus | Controle por número de fogo, recapagem, sulco. | não | — | Fora (Maintenance §2.3) |
| Abastecimento/consumo | Litros, km/l. No Fleet, entra apenas como **origem** de leitura de hodômetro. | parcial (*source: fueling*) | [30], [31] (fornecedor) | P2 / outro módulo |
| Multas/infrações | Recebimento, indicação do condutor em 30 dias e NIC (2×) para pessoa jurídica. | não | [1] art. 257 §§7–8 | Fora (novo módulo; depende de M3.6) |
| Telemetria | Ingestão de odômetro/horímetro e posição via integração. | parcial (pergunta aberta; *source: telematics*) | [29], [30], [31] (fornecedor) | P2 |
| Painel da frota/KPIs | Por status, idade média, % acessível, % acima da vida útil, documentos vencendo, disponibilidade. | parcial (Fleet panel; M1.4) | [22]; ROADMAP Module 5 | P0 (básico) / P1 (extensões) |
| Filtros/busca | Status, tipo, garagem, acessibilidade, idade, situação documental; busca por prefixo, placa atual ou anterior, VIN, RENAVAM. | parcial (CRUD *with filters*) | [2] art. 57 §1 | P0 |
| Permissões por papel | `fleet_manager` escreve o cadastro; `maintenance` lança hodômetro/OS; `planning` e `read_only` leem. | parcial (Module 0 RBAC) | ROADMAP Module 0 | P0 |
| Avaliação de condição | Nota periódica de condição (ex.: escala TERM 1–5 do FTA). | não | [21] §625.25(b)(2); [23] | P2 |

## 3. Lista de prioridades

### P0: MVP do módulo (bloqueia Maintenance/Planning)

| Item | Justificativa | Dependências | Marco |
|---|---|---|---|
| Prefixo como identidade + placa com histórico (PNU/Mercosul) | A placa muda por lei [2], e prefixo e id não podem mudar junto. | Planning, Charter e Metrics referenciam o veículo | M1.1 |
| Validação de VIN (formato), RENAVAM (DV) e placa | Evita cadastro sujo desde o seed. Os identificadores têm unicidade exigida no M1.1. | — | M1.1 |
| Categoria do registro (aluguel/particular) | É requisito ANTT [11] e gatilho de troca de placa [2]. | Charter (rotas de fretamento) | M1.1 |
| Características técnicas + data/km de entrada em serviço | Baseline da preventiva (Module 2 §2.1) e requisitos de rota (tipo, capacidade). | Maintenance M2.4; Planning M3.4 | M1.1 |
| Acessibilidade estruturada + flag derivado | O Planning bloqueia a escala por acessibilidade (Module 3 §3.2), e a regra precisa de critério objetivo [15]. | Planning M3.4; Charter SLA (*non-compliant vehicle*) | M1.1 |
| Hodômetro monotônico, troca, alerta de salto | Já planejado. É a fonte de km do Maintenance e do Metrics. | Maintenance M2.1/M2.4/M2.6 | M1.2 |
| Média km/dia definida | A projeção "o que vencer primeiro" depende dela. | Maintenance M2.4 | M1.2 |
| Máquina de estados + "status em uma data" | O Planning valida escala em datas futuras e o Metrics calcula disponibilidade. | Maintenance M2.1; Planning M3.4 | M1.3 |
| Idade (cálculo pelo ano de fabricação) | Insumo do painel e da idade máxima. Custo baixo. | Metrics M5.1 | M1.1 |
| Auditoria, filtros/busca, permissões | Usam Module 0. Sem isso, o cadastro não é demonstrável. | M0.1, M0.3 | M1.1 |
| Painel básico | Já planejado. | Metrics M5.1 | M1.4 |

### P1: logo após o MVP (alto valor para demo)

| Item | Justificativa | Dependências | Marco |
|---|---|---|---|
| Documentos com vencimento + alertas + anexos | Cronotacógrafo e licenciamento valem para a frota inteira [1][8]. É a principal lacuna regulatória. | Planning M3.4 (bloqueio/aviso) | **M1.5, antecipar para logo após M1.4** |
| Licenciamento por final de placa | Gera vencimentos automaticamente, sem digitação [4][5]. | Documentos | M1.5 |
| Idade máxima por contexto + vida útil (ULB) + KPI "% acima da vida útil" | Mostra o planejamento de renovação da frota. Referência FTA [22][23]. | Charter (requisito de contrato) | M1.4 (KPI) / M1.6 |
| Baixa com motivo e prazos (CTB 126/134) | Fecha o ciclo de vida e alimenta o KPI de vida útil. | Metrics | M1.3 (estende `retired`) |
| Transferência entre garagens com histórico | Métricas por garagem ficam erradas sem a data efetiva. | Metrics (filtro *depot*) | **M1.6 (proposto)** |
| Importação em lote de veículos | Seed e onboarding. A demo fica mais crível. | M0.3 (seed) | **M1.6 (proposto)** |
| NICV, classificação NBR 15570, AC/portas | Enriquece requisitos de rota e contrato. | Planning, Charter | M1.1 (campos opcionais) ou M1.6 |

### P2: depois

| Item | Justificativa | Dependências | Marco |
|---|---|---|---|
| Horímetro | Pouco usado em preventiva de ônibus. Útil para AC/equipamentos. | Maintenance (planos por horas) | **M1.7 (proposto)** |
| Equipamentos embarcados (série, instalação/remoção) | Valor operacional, mas não bloqueia nada. | Maintenance (OS de equipamento) | M1.7 |
| Integração de telemetria/abastecimento como origem de leitura | Depende de decisão de integração (seção 5). | Fonte externa | **M1.8 (proposto)** |
| Avaliação de condição (TERM-like) | Boa prática de inventário de ativos [21], mas opcional. | Metrics | M1.8 |

### Fora de escopo / outro módulo

| Item | Destino | Justificativa |
|---|---|---|
| Pneus | Maintenance §2.3 (peça serializada) | O ciclo de vida do pneu (rodízio, recapagem) é de manutenção e estoque. O Fleet guarda só a configuração de eixos/medida, se necessário. |
| Multas/infrações | Novo módulo (ex.: "Compliance") | A indicação de condutor exige saber quem dirigia na data/hora, o que depende do M3.6. O prazo é de 30 dias, com multa 2× para PJ [1] art. 257 §§7–8. |
| Consumo/abastecimento (km/l, custo) | Metrics ou módulo próprio | No Fleet, só a leitura de hodômetro gerada pelo abastecimento. |
| Alocação de veículo a linha/contrato | Planning (Module 3) | Já coberto por blocos e escala. |
| Rastreamento em tempo real | Fora do produto | Não é cadastro. |

## 4. Regras de negócio sugeridas (não presentes no ROADMAP)

1. **Formato de placa**: aceitar `^[A-Z]{3}[0-9]{4}$` (PNU, "AAA-1111") e `^[A-Z]{3}[0-9][A-Z][0-9]{2}$` (PIV, sequência `LLLNLNN`) [2]. Normalizar para maiúsculas, sem hífen.
2. **Placa mutável com histórico**: a troca de placa é um evento (data, placa anterior, nova, motivo). As duas placas continuam pesquisáveis [2] art. 57 §1, e a unicidade vale contra todas as placas ativas e anteriores de outros veículos. A conversão PNU→PIV troca o 2º caractere numérico conforme o Anexo II [2] (**tabela de conversão não verificada no texto do Anexo II**).
3. **Troca obrigatória de placa**: mudança de categoria ou de município/UF de registro exige PIV Mercosul [2] art. 56 §1. O sistema deve sinalizar a pendência quando a categoria ou o município de registro mudar.
4. **VIN**: exigir exatamente 17 caracteres alfanuméricos [3] art. 3º I. **Não** bloquear por dígito verificador (posição 9), porque a Res. 968/2022 não o exige. Excluir I/O/Q (ISO 3779) só como aviso, já que a resolução não traz essa restrição.
5. **RENAVAM**: 11 dígitos, completar códigos antigos de 9 dígitos com zeros à esquerda [24], com validação do DV por módulo 11 (pesos 3,2,9,8,7,6,5,4,3,2; resto ×10 mod 11, 10→0) [25]. **Algoritmo não confirmado em fonte primária**: validar contra RENAVAMs reais de CRLV-e antes de tornar bloqueante.
6. **Idade**: calculada pelo **ano de fabricação** (gravado e inalterável, [1] art. 114), não pelo ano-modelo. Os limites ANTT também usam "anos de fabricação" [11] art. 15.
7. **Idade máxima por contexto**: parâmetro por contexto regulatório/contrato. Para ANTT fretamento, micro-ônibus ≤ 15 anos e ônibus sem limite, mas com CSV semestral acima de 15 anos e anual nos demais [11] arts. 15–16. Alerta configurável N meses antes de atingir o limite.
8. **Licenciamento por final de placa**: o vencimento anual é derivado do último dígito da placa, com limites máximos de setembro (1–2), outubro (3–5), novembro (6–8) e dezembro (9–0) [4][5]. Calendário parametrizável por UF, porque o DETRAN pode antecipar. Só conta como licenciado com débitos quitados [1] art. 131 §2. Circular sem licenciamento é infração gravíssima com remoção [1] art. 230 V.
9. **Cronotacógrafo obrigatório**: todo veículo com mais de 10 lugares ou de escolares [1] art. 105 II exige o documento "certificado de verificação do cronotacógrafo", com validade de 2 anos [8] (Portaria Inmetro 481/2021 [9]). Os requisitos técnicos do equipamento estão na Res. CONTRAN 938/2022 [7]. A lista vigente de equipamentos obrigatórios é a Res. 993/2023 [6], que revogou a 912/2022 ainda citada pelo Inmetro [10].
10. **Veículo escolar** (tipo `school`): exige autorização do órgão de trânsito afixada no veículo, inspeção **semestral**, faixa amarela, cronotacógrafo e cintos em número igual à lotação [1] arts. 136–137. A lotação não pode exceder a capacidade do fabricante [1] art. 137. O município pode exigir mais [1] art. 139.
11. **Acessibilidade derivada**: `acessivel_cadeira_rodas = tipo_acesso ∈ {a,b,c} ∧ boxes_cadeira ≥ 1` [15] §6, §6.3.1. Validar `assentos_preferenciais ≥ max(2, ceil(10% dos assentos))` [15] §6.2 como aviso de não conformidade. Mapear para GTFS `wheelchair_accessible` 1/2 [18]. Veículos exclusivos de fretamento/turismo estão fora da obrigação do Decreto 5.296 [13] art. 38 §5.
12. **Bloqueio de escala por documento vencido** (**afeta o Module 3, M3.4**): cada tipo de documento tem uma severidade parametrizável (`blocking` | `warning`), seguindo o padrão da preventiva vencida. Sugestão de default: CRLV vencido = bloqueante (art. 230 V). Autorização de escolar vencida = bloqueante para rotas escolares (art. 230 XX, gravíssima). CSV/seguro RC = bloqueante para viagens de fretamento ANTT (porte obrigatório, [11] art. 31 §3). Certificado do cronotacógrafo = aviso (a infração do art. 230 XIV trata de equipamento "viciado ou defeituoso", e não verifiquei a penalidade por verificação vencida).
13. **Baixa**: `retired` exige motivo de uma lista fechada. Motivo "irrecuperável/desmontagem" gera a pendência "solicitar baixa do registro" [1] art. 126. Motivo "venda" gera a pendência "comunicar venda ao DETRAN em até 60 dias" se o comprador não transferir em 30 [1] arts. 123 §1, 134.
14. **Transferência de garagem**: evento com data efetiva. Não altera o registro, a menos que o município de registro mude (nesse caso, ver regra 3).
15. **Leituras e baixa**: nenhuma leitura com data posterior à data de baixa, e nenhuma anterior à data de entrada em serviço (complementa "retired accepts no readings").

## 5. Respostas propostas às perguntas abertas do Módulo 1

**Média km/dia: janela de 30 ou 90 dias?** Não há fonte normativa. Isto é recomendação de projeto.
- **90 dias corridos, móvel, parametrizável.** Trinta dias oscilam demais em frota com contratos escolares (férias zeram a rodagem) e com períodos `in_maintenance`.
- Calcular como `(km_acumulado(hoje) − km_acumulado(hoje−90d)) / dias_corridos`, usando o **km acumulado**, para atravessar trocas de hodômetro sem distorção.
- Mínimo de 2 leituras cobrindo ≥ 14 dias. Abaixo disso (veículo novo), usar a média do mesmo tipo/garagem e marcar a projeção como "estimada".
- Dias em `in_maintenance` entram no denominador. A preventiva quer o ritmo real de acúmulo, paradas incluídas, e assim a projeção fica conservadora.
- O limiar do alerta de salto anômalo pode ser expresso como múltiplo dessa média, além do teto absoluto. O fornecedor Fleetio usa validação contra leituras vizinhas [29].

**Integração de telemetria/abastecimento ou só lançamento manual?**
- MVP: **manual (unitário + planilha)**. O enum `source` do ROADMAP já prevê `fueling` e `telematics`. Basta definir a porta de ingestão de leituras (interface própria do projeto), sem implementar adaptadores.
- Leituras de integração devem ser imutáveis e só "anuláveis" com justificativa, como no Fleetio [29], coerente com o princípio de histórico imutável.
- Abastecimento como entidade (litros, custo) fica fora do Fleet (seção 3).

## 6. Novas perguntas abertas para o humano

1. **Contextos regulatórios do seed**: a empresa fictícia opera urbano municipal, intermunicipal, fretamento interestadual (ANTT) e/ou escolar? Isso define quais documentos são obrigatórios por veículo.
2. **Severidade por documento vencido**: aceitar os defaults da regra 12 ou tratar tudo como aviso no MVP? (afeta M3.4)
3. **UF de registro**: uma UF só (um calendário de licenciamento) ou várias?
4. **Identidade do veículo**: confirmar prefixo (ou id interno) como chave, com placa mutável e histórico.
5. **Veículos arrendados/locados**: precisamos registrar propriedade (própria/leasing) e arrendante? A ANTT exige a anotação do arrendamento [11] art. 11 §1.
6. **Idade máxima urbana**: qual limite o seed usa (por contrato municipal)? Fica no contrato (Module 4) ou em parâmetro global?
7. **Acessibilidade**: guardar só o necessário para o flag derivado ou também os itens de comunicação (SIA, audiovisual) da NBR 14022 §7?
8. **Pneus e multas**: confirmar que pneus vão para o Maintenance e que multas viram módulo futuro (ou ficam fora).
9. **Capacidade**: usar a lotação do CRLV ou uma capacidade operacional menor definida em contrato?
10. **Vida útil de referência**: definir valores por tipo para o seed (referência FTA: ônibus 14 anos, *cutaway* 10 [23]) ou deixar só o parâmetro?

## 7. Fontes

### Primárias: legislação, regulação e normas

1. Brasil. Lei 9.503/1997, Código de Trânsito Brasileiro (texto compilado), arts. 104, 105 II, 114, 120, 123, 126, 130, 131, 133, 134, 136–139, 230, 231, 257. https://www.planalto.gov.br/ccivil_03/leis/l9503compilado.htm (acesso 2026-10-30)
2. CONTRAN. Resolução 969, de 20/06/2022 (Placas de Identificação Veicular, padrão Mercosul), cópia do DOU. https://www.detran.pr.gov.br/sites/default/arquivos_restritos/files/documento/2022-07/resolucao_969_2022-contran.pdf (acesso 2026-10-30)
3. CONTRAN. Resolução 968, de 20/06/2022 (identificação veicular: VIN, NIEV, NICV), reprodução LegisWeb. https://www.legisweb.com.br/legislacao/?id=433072 (acesso 2026-10-30)
4. CONTRAN. Resolução 110, de 24/02/2000 (calendário do licenciamento anual), reprodução LegisWeb. A tabela não foi exibida na reprodução. https://www.legisweb.com.br/legislacao/?id=97100 (acesso 2026-10-30)
5. CTB Digital. Comentário ao art. 130 (transcreve a tabela da Res. 110/2000). *Fonte secundária.* https://ctbdigital.com.br/comentario/comentario130/ (acesso 2026-10-30)
6. CONTRAN. Resolução 993, de 15/06/2023 (equipamentos obrigatórios; revoga a Res. 912/2022), reprodução Lex. Anexo I não lido. https://www.lex.com.br/resolucao-contran-no-993-de-15-de-junho-de-2023/ (acesso 2026-10-30)
7. CONTRAN. Resolução 938, de 28/03/2022 (requisitos técnicos do cronotacógrafo; revoga a Res. 92/1999). **Não lida integralmente.** https://www.legisweb.com.br/legislacao/?id=429779 (acesso 2026-10-30)
8. Inmetro. FAQ "Após a verificação inicial qual é a periodicidade de verificação dos cronotacógrafos?" ("a cada dois anos"). https://www.gov.br/inmetro/pt-br/acesso-a-informacao/perguntas-frequentes/metrologia-legal/cronotacografo/apos-a-verificacao-inicial-qual-e-a-periodicidade-de-verificacao-dos-cronotacografos (acesso 2026-10-30)
9. Inmetro. FAQ "Qual é a Portaria que trata dos prazos para a verificação de cronotacógrafos?" (Portaria Inmetro 481, de 06/12/2021). https://www.gov.br/inmetro/pt-br/acesso-a-informacao/perguntas-frequentes/metrologia-legal/cronotacografo/qual-e-a-portaria-que-trata-dos-prazos-para-a-verificacao-de-cronotacografos (acesso 2026-10-30)
10. Inmetro. FAQ "Quais os veículos devem possuir cronotacógrafo?" (cita a Res. CONTRAN 912/2022, já revogada pela 993/2023 [6]). https://www.gov.br/inmetro/pt-br/acesso-a-informacao/perguntas-frequentes/metrologia-legal/cronotacografo/quais-os-veiculos-devem-possuir-cronotacografo (acesso 2026-10-30)
11. ANTT. Resolução 4.777/2015 (fretamento interestadual e internacional), "vigente com alterações" (Res. 4.978/2015, 5.017/2016, 5.577/2017), arts. 3º, 11, 15, 16, 20, 31, 44, 45. https://anttlegis.antt.gov.br/action/ActionDatalegis.php?acao=abrirTextoAto&tipo=RES&numeroAto=00004777&seqAto=000&valorAno=2015&orgao=DG/ANTT/MT (acesso 2026-10-30)
12. Diário do Transporte. "ANTT publica novo marco do transporte internacional de passageiros…" (Res. ANTT 6.087/2026, transporte **internacional**). *Fonte secundária (imprensa); texto da resolução não verificado.* https://diariodotransporte.com.br/2026/08/17/antt-publica-novo-marco-do-transporte-internacional-de-passageiros-com-limite-de-idade-para-onibus-indice-de-qualidade-e-novas-regras-para-linhas-e-fretamento/ (acesso 2026-10-30)
13. Brasil. Decreto 5.296/2004, arts. 31–38 (acessibilidade no transporte coletivo rodoviário). https://www.planalto.gov.br/ccivil_03/_ato2004-2006/2004/decreto/d5296.htm (acesso 2026-10-30)
14. Brasil. Lei 13.146/2015 (Lei Brasileira de Inclusão), arts. 46 e 48. https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2015/lei/l13146.htm (acesso 2026-10-30)
15. ABNT. NBR 14022:2011, 4ª ed., *Acessibilidade em veículos de características urbanas para o transporte coletivo de passageiros*, seções 1, 6, 7 (cópia hospedada pelo MPRJ). Não verifiquei se existe edição posterior. https://www.mprj.mp.br/documents/20184/74453/nbr_14022_2011_onibus_ed4.pdf (acesso 2026-10-30)
16. Prefeitura de Ponta Grossa. Apêndice I.I, Especificações para os veículos (tabela de classes da NBR 15570:2021). *Documento de edital; valores não verificados na norma ABNT.* https://transportes.pontagrossa.pr.gov.br/wp-content/uploads/4.-APENDICE-I.I-ESPECIFICACOES-PARA-OS-VEICULOS-4.pdf (acesso 2026-10-30)
17. CONTRAN. Resolução 402/2012 (exige nota fiscal com atendimento às NBR 14022/15570 para urbanos fabricados a partir de 01/03/2009). **Visto apenas em resultado de busca; não lida.** https://www.normasbrasil.com.br/norma/resolucao-402-2012_240374.html (acesso 2026-10-30)

### Primárias: padrões abertos e referência internacional

18. GTFS Schedule Reference, `trips.txt` → `wheelchair_accessible` (0/1/2). A especificação oficial não tem arquivo de veículos. https://raw.githubusercontent.com/google/transit/master/gtfs/spec/en/reference.md (acesso 2026-10-30)
19. GTFS Realtime Reference, `VehicleDescriptor` (`id`, `label`, `license_plate`, `wheelchair_accessible`). https://raw.githubusercontent.com/google/transit/master/gtfs-realtime/spec/en/reference.md (acesso 2026-10-30)
20. EUA. 49 CFR §625.5, definições (*asset inventory*, *useful life benchmark*, *state of good repair*, *rolling stock*). https://www.law.cornell.edu/cfr/text/49/625.5 (acesso 2026-10-30)
21. EUA. 49 CFR §625.25, elementos do TAM plan (inventário, avaliação de condição). https://www.law.cornell.edu/cfr/text/49/625.25 (acesso 2026-10-30)
22. EUA. 49 CFR §625.43, medida de desempenho: % de veículos que atingiram ou superaram o ULB. https://www.law.cornell.edu/cfr/text/49/625.43 (acesso 2026-10-30)
23. FTA. *TAM ULB Cheat Sheet* (default ULB: BU 14, AB 14, CU 10 anos). **Acesso direto retornou 403; valores obtidos via resultado de busca.** https://www.transit.dot.gov/sites/fta.dot.gov/files/2021-11/TAM-ULB-CheatSheet.pdf (acesso 2026-10-30)

### Secundárias: identificadores

24. Portal do Trânsito. "Código do Renavam ganhará dois dígitos a partir de 1º de abril de 2013" (9→11 dígitos; zeros à esquerda; módulo de cálculo inalterado). *Imprensa.* https://www.portaldotransito.com.br/noticias/codigo-do-renavam-ganhara-dois-digitos-a-partir-de-1o-de-abril-de-2013/amp/ (acesso 2026-10-30)
25. VerificarAuto. "O que é RENAVAM… e como validar" (algoritmo do DV). *Site comercial; não é fonte oficial.* https://verificarauto.com.br/o-que-e-renavam/ (acesso 2026-10-30)

### Fontes de fornecedor (estado da arte; não normativas)

26. Fleetio. *Vehicle Statuses*. https://help.fleetio.com/en_US/vehicle-statuses (acesso 2026-10-30)
27. Fleetio. *Vehicle Replacement – Lifecycle Details*. https://help.fleetio.com/en_US/vehicle-replacement-lifecycle-details-5 (acesso 2026-10-30)
28. Fleetio. *Vehicle Renewal Types* (Emission Test, Inspection, Insurance, Registration). https://help.fleetio.com/en_US/vehicle-renewal-types (acesso 2026-10-30)
29. Fleetio. *Meter Overview*, *Meter Entry Conflicts* e *Reset Meter Values* (medidor primário/secundário, horas, *void*, validação contra leituras vizinhas). https://help.fleetio.com/en_US/meter-overview, https://help.fleetio.com/en_US/meter-entry-conflicts, https://help.fleetio.com/en_US/reset-meter-values (acesso 2026-10-30)
30. Infleet. Página inicial (módulos: Videotelemetria, Telemetria, Multas, Abastecimento, Manutenção, Checklist). https://infleet.com.br/ (acesso 2026-10-30)
31. Cobli. Página inicial (videotelemetria, telemetria via rede CAN, controle de manutenção, gestão de gastos/combustível). https://www.cobli.co/ (acesso 2026-10-30)
