# Hexis · documentação do código

Atualizada em 18/nov/2026 (revisão de segurança, conferência dos pedidos e melhorias; ver a seção 11). A tabela de parâmetros e o índice de funções deste documento são tirados dos comentários do próprio código, então cada linha daqui tem um comentário igual no ponto correspondente do `index.html`.

## 1. Onde fica cada coisa

| O quê | Onde fica | Para que serve |
| --- | --- | --- |
| Site | https://panoramainteligenciapolitica-droid.github.io/Hexis/ | O app com login, aberto a qualquer pessoa |
| Repositório | github.com/panoramainteligenciapolitica-droid/Hexis | Arquivos do site: `index.html`, `manifest.webmanifest`, `icons/`, `README.md` e este documento |
| Banco do site | Supabase, projeto `hexis` (região sa-east-1) | Login por e-mail e senha e a tabela `hexis_docs`, com os dados de cada pessoa |
| Versão de origem | App privado, fora do repositório | É onde o app é alterado primeiro. Tem armazenamento próprio, separado do banco do site |
| Gerador do site, testes e cópias de segurança | Área de trabalho de desenvolvimento, fora do repositório | **Desatualizado desde 18/nov: não use** (ver seção 8). `build_web.py` transformava a versão de origem no `index.html` do site; os testes conferem cada função antes de publicar |

Pontos de atenção:

- A versão de origem e o site **não compartilham dados**. O que é anotado em um não aparece no outro.
- O gerador do site e os testes antigos (`run10`…`run38`, `tbudget`, `cmp.sh`, `inv.py`) ficaram na área de desenvolvimento e não vieram no pacote. Desde 18/nov o pacote traz a pasta `testes/` (seção 9), que roda contra o `completo/index.html`. Quem alterar o `index.html` direto no GitHub muda só o site.

## 2. Como o `index.html` é organizado

É um arquivo só, com três partes:

1. **Estilos** (`<style>`): a aparência, em 21 blocos numerados. O bloco 21 (Liquid Glass) vem por último e prevalece sobre os anteriores: cores, vidro, cápsulas, tela travada e quadros fixos das séries.
2. **Telas** (`<main>` e as barras): a estrutura vazia de cada aba.
3. **Programa** (`<script>`): 27 seções numeradas, que preenchem as telas e guardam os dados.

Para achar um trecho, procure no arquivo pelo número da seção: `===== 18.` leva ao cronômetro no programa; `---- 12.` leva aos estilos do cronômetro. O começo do programa tem um mapa com as 27 seções e um resumo de segurança.

### Estilos (21 blocos)

1. Cores e fontes (tema claro; o escuro vem logo abaixo)
2. Base e cabeçalho
3. Cardio: caixas de pace, velocidade e calorias
4. Telas, avisos e barra da semana
5. Cartão do exercício e tabela de séries
6. Botões
7. Campos, painéis, blocos de número, tabelas e gráficos
8. Barra de salvar, botão "i" e pesagem rápida
9. Janela de explicação
10. Perfil e Objetivo (formulários)
11. Aquecimento, Feito, intervalo e blocos de observação
12. Cronômetro (barra da série, barra fixa e aviso)
13. Unilateral
14. Navegação (barra de baixo e subabas)
15. Início
16. Transições e animações
17. Resumo do dia
18. Série: execução, parciais, descanso e aviso das observações
19. Dieta: diário de alimentos
20. Corpo: evolução corporal (exames de bioimpedância)
21. Liquid Glass: vidro em camadas, cantos concêntricos, barras flutuantes; tela travada e quadros fixos

### Telas

Cada aba é uma `<section class="view">`. Só a aba aberta fica visível.

- `v-inicio`: Início (preenchida por renderHome): o que é o app
- `v-perfil`: Perfil
- `v-objetivo`: Objetivo
- `v-semana`: Semana (treino do dia)
- `v-resumo`: Resumo do dia
- `v-proxima`: Próxima semana
- `v-progresso`: Progresso (gráficos)
- `v-corpo`: Corpo e dieta
- `v-dieta`: Dieta (meta do dia e diário de alimentos com a tabela TACO)
- `v-diario`: Hormônios e peptídeos (referência para estudo, controle de doses, calendário e exames)
- `v-plano`: Plano e regras

Fora das abas ficam: o cabeçalho com o logo, o botão **Hoje** (`#hojeBtn`) e as subabas (`#subnav`), o painel flutuante Hoje (`#hoje`, com o treino do dia, os números principais e a pesagem diária), a barra de navegação de baixo (`#bnav`), a barra de salvar com o cronômetro (`.savebar`, `#clkbar`) e a janela de explicação (`#pop`).

### Navegação

A barra de baixo tem 5 grupos; cada grupo pode ter subabas no topo. Tudo sai da lista `NAVG` (seção 21):

| Grupo | Abas |
| --- | --- |
| Início | Início |
| Treino | Semana · Resumo do dia · Próxima |
| Dieta | Dieta |
| Evolução | Progresso · Corpo · Hormônios |
| Você | Perfil · Objetivo · Plano e regras |

## 3. Como os dados são guardados

Cada item é um documento separado, com um caminho:

| Caminho | O que guarda |
| --- | --- |
| `semanas/w01-A` | Um dia de treino: `{ semana, dia, nota, tags, cardio, ex }` |
| `corpo/2026-11-10` | Pesagem e medidas de um dia |
| `meta/perfil` | Perfil e Objetivo |
| `meta/ajustes` | Deload e semanas leves escolhidos, palavras-chave criadas, quais exercícios são unilaterais e as cargas de cada máquina (`maq[nome do exercício] = { tipo, placas }`) |
| `diario/sub-…`, `diario/ap-…`, `diario/ex-…` | Hormônios e peptídeos: substância, aplicação e exame |
| `dieta/2026-11-15` | Alimentos de um dia: `itens[] = { r: refeição, a: alimento, n: nome, q: quantidade, u, kcal, p, c, g }` |
| `exames/2026-11-13` | Exame de bioimpedância: composição, análise segmentar (`seg`), índices e faixas normais do laudo (`faixas`); `soMes` quando só o mês é conhecido |
| `dieta/meus` | Alimentos fora da tabela cadastrados pela pessoa |
| `semanas/w01-DOM` | Domingo (dia livre): só o registro de cardio ou atividade |
| `fotos/2026-11-12-…` | Registro de uma foto: data, tipo (`treino` ou `evolucao`) e onde o arquivo está |

Dentro de um dia, `ex` tem um registro por exercício:

- `ex.A1 = { st, sets, nota, tags, trans, startAt, doneAt, prev, aq, aqT, aqD }`
  - `st`: `"feito"`, `"nao"` ou vazio. `trans`: intervalo antes do exercício, em segundos. `startAt` e `doneAt`: horários usados para descobrir a ordem feita. `prev`: o que estava previsto quando o exercício foi feito. `aq`: séries de aquecimento marcadas. `aqT`: tempo cronometrado do aquecimento (séries e trocas de peso), em segundos. `aqD`: descanso entre o fim do aquecimento e a 1ª série válida, em segundos (0 quando esse descanso já está dentro de `aqT`).
- `sets[i] = { kg, reps, rir, parc, exec, desc, lados, dif }`
  - `parc`: repetições parciais, em % do movimento. `exec`: tempo de execução (s). `desc`: descanso depois da série (s).
  - `lados`: dados de cada lado (`e`, `d`) no exercício unilateral. `dif`: lado mais difícil.

O cronômetro em andamento fica só no aparelho (parâmetro `CLK_KEY`), para continuar contando se a página for fechada. A preferência de tela acesa também (`WAKE_KEY`).

No diário e nas fotos, apagar é gravar o documento como `{ apagado: true }`; o app deixa de mostrá-lo.

**No site**, cada documento é uma linha da tabela `hexis_docs`:

| Coluna | Conteúdo |
| --- | --- |
| `user_id` | Dono da linha (o usuário logado) |
| `path` | O caminho do documento (`semanas/w01-A`) |
| `data` | O documento, em JSON |
| `updated_at` | Data da última gravação |

A chave é `(user_id, path)`. As regras de acesso da tabela (RLS) só deixam cada pessoa ler e gravar as linhas com o próprio `user_id`.

Os arquivos das fotos ficam no armazenamento `fotos` do Supabase, que é privado: cada pessoa tem uma pasta com o próprio `user_id` e só ela consegue enviar, abrir e apagar o que está lá. A foto é reduzida para 1.280 px antes do envio e aberta por um link temporário de 1 hora.

## 4. Como uma mudança percorre o app

1. Você toca ou digita (seção 24).
2. O registro muda na memória (seção 10) e o salvamento é agendado para 1,2 s depois (seção 24).
3. O motor refaz todas as semanas a partir dos registros (seção 3). Nada da prescrição é gravado: ela é sempre recalculada.
4. A tela é redesenhada (seções 11 a 23).

Por isso corrigir um registro antigo corrige sozinho todas as semanas seguintes.

## 5. Parâmetros ajustáveis

São 53 parâmetros, todos na seção 1 do programa. Mudar um valor ali muda todos os lugares que o usam.

**Plano**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `MAXW` | `60` | quantas semanas o app calcula (até o fim de 2027) |
| `BLOCK_W` | `9` | semanas do primeiro bloco ("Semana N de 9") e de cada bloco de quem não segue o plano anual |
| `PLUS_MAX` | `2` | no máximo 2 séries a mais por exercício quando o músculo é prioritário |
| `REFASE_MIN` | `2` | a carga é reajustada entre blocos quando o meio da faixa muda 2 repetições ou mais |
| `TEMPO_MAX_DEF` | `120` | limite de tempo por dia (min), se o Perfil não disser outro |
| `CARDIO_DEF` | `{ min: 40, vezes: 6, incl: 15, vel: 5.5, tipo: "caminhada" }` | cardio quando ainda não há plano |

**Motor de progressão**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `BACKOFF` | `0.88` | back-off = 88% da série pesada (12% a menos) |
| `BELOW` | `0.975` | série "abaixo do previsto": força menor que 97,5% do piso da faixa |
| `PROGRESSO` | `1.005` | progresso = força pelo menos 0,5% acima da melhor marca |
| `CUT_DOR` | `0.9` | dor forte, ou dor com série abaixo: a carga cai 10% |
| `CUT_QUEDA` | `0.95` | 1ª semana abaixo da faixa, ou 2 semanas sem fazer: a carga cai 5% |
| `RESET_ONDA` | `[0.9, 0.95, 1]` | reset em onda: fases 1, 2 e 3, como fração da carga travada |
| `AJUSTE_PISO` | `0.85` | ajuste entre séries: no máximo 15% abaixo do previsto |
| `MAQ_TOL` | `[0.10, 2.5, 7.5]` | placas fixas: uma carga vale como "a placa mais próxima" da prevista quando a diferença é de até 10% (no mínimo 2,5 kg, no máximo 7,5 kg) |
| `MAQ_JUNTA` | `0.75` | placas fixas: cargas anotadas a menos disso uma da outra contam como a mesma placa |
| `MAQ_PLACA` | `[4, 15]` | placas fixas: a máquina é reconhecida sozinha quando todas as distâncias entre as cargas anotadas ficam nesta faixa de kg (e valem pelo menos 2 saltos da ficha) |
| `ELO_SEG` | `0.95` | conversa entre treinos: margem de 5% na carga trazida de outro dia quando a faixa de repetições é diferente |
| `ELO_LIMIAR` | `1` | conversa entre treinos: a diferença contra o previsto precisa valer pelo menos este tanto de saltos de carga para mexer na carga do outro dia |
| `PARC_OPTS` | `[25, 50, 75]` | botões de repetição parcial: % do movimento feito |
| `PARC_CREDITO` | `0.5` | parcial vale metade da fração feita (50% do movimento = 0,25 repetição) |
| `DELOAD_DIAS` | `2` | dias abaixo da faixa na semana 4 que ligam o deload automático da semana 5 |

**Tempo, descanso e recuperação**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `EXEC_DEF` | `40` | segundos de execução por série, quando não anotado |
| `TRANS_DEF` | `90` | segundos de intervalo entre exercícios, quando não anotado |
| `TRANS_JUNTO` | `30` | troca quando o exercício é na mesma máquina do anterior (só muda a carga); na ficha, junto: true usa este valor e junto: 100 usa 100 s (mesma polia, troca de puxador) |
| `RITMO_MIN` | `8` | tempos cronometrados necessários para o app passar a usar o seu ritmo no lugar do padrão |
| `RITMO_N` | `60` | o ritmo olha os tempos mais recentes (até este número de cada tipo) |
| `RITMO_FOLGA` | `10` | descanso real mais de 10 s acima do previsto, em média: o painel mostra quanto tempo isso custa |
| `AQUEC_SEC` | `300` | aquecimento geral no começo do treino |
| `WARM_EXEC` | `20` | execução de uma série de aquecimento (carga leve ou poucas repetições) |
| `WARM_PAUSA` | `[[0.6, 30], [1, 60]]` | pausa depois de uma série de aquecimento, pela fração da carga do dia: até 60%, só a troca de peso (30 s); acima, 1 min |
| `WARM_ANTES` | `[[0.6, 60], [1, 120]]` | descanso entre a última série de aquecimento e a 1ª válida, pela fração da última: leve 1 min; pesada 2 min |
| `CLK_WARM_MAX` | `900` | aquecimento acima de 15 min no cronômetro é tratado como esquecido |
| `REST_OPTS` | `[30, 45, 60, 90, 120, 150, 180, 240]` | botões de descanso entre séries |
| `TRANS_OPTS` | `[60, 90, 120, 180, 240, 300, 420]` | botões de intervalo entre exercícios |
| `REST_CURTO` | `0.75` | descanso "curto": abaixo de 75% do previsto |
| `REST_LONGO` | `1.5` | descanso "longo": acima de 150% do previsto |
| `REC_BOA` | `0.95` | recuperação "boa": a série seguinte rende 95% ou mais do esperado |
| `REC_REGULAR` | `0.9` | "regular" até 90%; abaixo disso, "incompleta" |

**Cronômetro**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `CLK_KEY` | `"hexis.clk.v1"` | nome com que o cronômetro fica guardado no aparelho |
| `CLK_ALERT` | `300` | avisa a cada 5 minutos rodando |
| `CLK_EXEC_MAX` | `150` | execução acima de 2:30 é tratada como esquecida |
| `FOTO_MAX` | `1280` | lado maior da foto enviada, em pixels (fotos maiores são reduzidas) |
| `DJ_ATRAS` | `60, DJ_FRENTE = 30` | curva do diário: dias para trás e para a frente |
| `DJ_EQUIL` | `5` | meias-vidas até o nível estabilizar com doses repetidas (e até a substância praticamente sair depois da última dose) |
| `DJ_HT_ALERTA` | `54` | hematócrito (%) acima do qual a diretriz de reposição de testosterona manda suspender e reavaliar |
| `KEEP_KEY` | `"hexis.abertos"` | blocos recolhíveis abertos ou fechados por você, guardados no aparelho       // preferência "tela acesa durante o cronômetro", guardada no aparelho |
| `CLK_SEG_REP` | `4` | segundos por repetição, para a barra de execução |
| `UNI_TROCA` | `30` | troca de lado na série unilateral: referência da barra e tempo previsto (o que você levar é anotado) |

**Resumo do dia**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `PERF_ACIMA` | `0.02` | "acima do previsto" a partir de +2% |
| `PERF_ABAIXO` | `-0.025` | "abaixo do previsto" a partir de −2,5% |
| `PERF_DESTAQUE` | `0.04` | entra nos destaques a partir de +4% |

**Unilateral**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `ASSIM_EQUIL` | `3` | diferença entre os lados abaixo de 3%: "força equilibrada" |
| `ASSIM_ALERTA` | `10` | a partir de 10%: orienta começar pelo lado mais fraco |

**Lados juntos** (pedido de 18/nov): o exercício é unilateral, mas cada série tem uma linha só e um tempo só, que inclui os dois lados e a troca. A série fica marcada com `uj: 1`. Conta como unilateral no tempo previsto (`dayTime`, via `isUni`) e no ritmo (`ritmo`, via `serieUni`); o cronômetro aceita até `2 × CLK_EXEC_MAX + UNI_TROCA` (`clkExecMax`, 5:30) e espera o dobro do tempo (`clkGo`); o Resumo do dia mostra "(2 lados)"; o gráfico de execução por série conta a metade (um lado). Teste: `testes/unijunto.js`.

Os números do treino em si não estão nesta tabela, e sim na ficha (seção 2): séries, faixa de repetições, carga inicial, descanso previsto e quanto a carga sobe em cada exercício (lista `EX`), e as metas de RIR de cada semana (lista `WT`).

## 6. Índice por seção

São 383 funções no programa (contadas em 18/nov, depois da 2ª banca). Cada uma tem, no código, um comentário com o mesmo texto da tabela. Para conferir se alguma ficou fora do índice, procure cada `function nome(` do código neste arquivo.

### 1. Parâmetros ajustáveis

Os números que regulam o app (tabela acima).


### 2. Ficha

Dias, exercícios, regras de cada semana do primeiro bloco e os blocos do resto do ano.

| Dado | O que guarda |
| --- | --- |
| `PLAN_START` | segunda da semana 1 (vem do Perfil) |
| `DAYS` | Dias de treino: código, dia da semana (0 = segunda), nome curto e título |
| `EX` | A ficha: todos os exercícios, na ordem em que aparecem em cada dia |
| `EXBY` | Exercício pelo código (EXBY.A1) |
| `WT` | Semanas do bloco: RIR-alvo [mín, máx] da série pesada (top), dos compostos (comp) e dos isoladores (iso), séries extras de core e a alavanca da semana |
| `WT_NORMAL` | Semana comum, usada quando a semana 7 ou 8 não cai nas festas do calendário |
| `RIR_FORCA`, `RIR_HIPER`, `RIR_PICO` | Rampa de RIR por semana do bloco, na ordem [série pesada, compostos, isoladores]; ao chegar ao fim, recomeça |
| `BLOCOS_ANO` | Blocos do plano anual, cada um a partir de uma segunda-feira (ini) até o começo do próximo. top, back = faixa da série pesada e dos back-offs (sem isso, 6–8 e 8–10) · rir = rampa de RIR · core = séries extras de core plus = músculos prioritários: ganham n0 séries na 1ª semana e +1 por semana até n1 · leve = semanas leves · teste = semana de teste |
| `ACT` | Ações que o motor pode prescrever: símbolo, texto e cor |
| `FIXNAME` | Nomes das técnicas de destrave |
| `MES` | Meses abreviados |

| Função | O que faz |
| --- | --- |
| `ex()` | Monta um exercício da ficha: k = código, d = dia, n = nome, m = músculo. O resto tem padrão: sc ("st" séries iguais, "top" 1 pesada + back-offs), sets, faixa lo–hi, kg inicial, inc e mic (salto e microcarga), step (arredondamento), t (comp, iso ou core), rest, obs, bw (peso do corpo), opt (opcional) |
| `contaEx()` | Séries de core por semana na ficha (sem os opcionais), antes do acréscimo da fase Conta para o dia e a semana: não é opcional e já estava na ficha naquela semana (exercício novo não vira pendência nas semanas anteriores) |
| `dayEx()` | Exercícios de um dia, na ordem da ficha |
| `exShort()` | Nome curto do exercício (sem o que vem entre parênteses) |
| `isFestas()` | Semana com 24, 25 ou 31/dez ou 1º/jan |
| `blocoDe()` | Bloco do plano anual em que a semana cai (só depois do primeiro bloco e para quem segue o plano anual) |
| `wtAno()` | Regras de uma semana dentro de um bloco do plano anual (k = semana do bloco) |
| `wtCiclo()` | Quem não segue o plano anual: depois do primeiro bloco, o ciclo de 9 semanas recomeça a partir das cargas atingidas |
| `wt()` | Regras da semana w: primeiro bloco pela tabela WT (ajustada às festas do calendário); depois, pelo bloco do plano anual ou pelo ciclo |
| `plusSets()` | Séries a mais de um exercício quando o músculo é prioritário no bloco: as n séries da semana são repartidas entre os exercícios do músculo, na ordem da ficha |
| `rephase()` | Mudança de faixa da série pesada entre blocos (ex.: 6–8 → 4–6): ajusta a carga pela força estimada, para o mesmo esforço com outra quantidade de repetições |
| `rnd()` | Arredonda x para o múltiplo de step |
| `fmt()` | Número com vírgula, até 2 casas; vazio vira — |
| `e1()` | Força estimada (1RM pela fórmula de Epley): carga × (1 + repetições ÷ 30) |
| `wkKey()` | Chave da semana nos registros: w01, w02… |
| `weekStart()` | Data da segunda-feira da semana w |
| `isoDate()` | Data como AAAA-MM-DD |
| `weekLabel()` | Período da semana por extenso: 9–15/nov |
| `weekOfDateStr()` | Número da semana do plano em que cai uma data |
| `dayDate()` | Data de um dia de treino (A, B…) na semana w |

### 3. Motor de progressão

Decide carga, repetições e técnica de cada exercício em cada semana, a partir dos registros.

| Dado | O que guarda |
| --- | --- |
| `mid` | Meio de uma faixa [mín, máx] |
| `MAQ` | Mapa das máquinas da última conta feita (usado na prescrição e no ajuste entre séries) |
| `ELOS`, `DOW`, `EX_ORD` | Irmãos de cada exercício (mesmo nome em outro dia), a posição de cada dia na semana e a ficha em ordem de dia |

| Função | O que faz |
| --- | --- |
| `baseSets()` | Séries-base do exercício: 1 pesada (6–8) + 2 back-offs (8–10), ou N séries iguais na faixa da ficha |
| `rirFor()` | RIR-alvo da série conforme o tipo (pesada, composto ou isolador) e a semana |
| `prescribe()` | Prescrição de uma semana a partir do estado |
| `setsStr()` | Séries feitas em texto: 52×7, 46×9+1p (p = parciais) |
| `parcCredit()` | Parciais valem meia repetição na fração feita (ex.: 50% = 0,25): contam para o progresso, não para a faixa |
| `transition()` | Transição: o que a semana w ensinou para a semana w+1 |
| `transitionCore()` | Regras da progressão, na ordem em que valem: sem registro → não feito → carga nova → semana leve → dor → ajuda → carga fixa → pesada demais → semana fora do normal → fácil → reset em onda → topo da faixa → destrave funcionou → progrediu → abaixo da faixa → platô (destraves) |
| `belowSet()` | Série abaixo do previsto: força da série feita < 97,5% da força exigida pelo piso da faixa na carga prevista |
| `suggestSets()` | Ajuste fino entre séries do mesmo exercício (autorregulação por RIR, como no powerlifting). A nota aparece sempre que a série anterior foi anotada: ▲ subir, ▼ baixar, ● manter |
| `metricOf()` | Medida de desempenho do exercício: melhor força estimada entre as séries (peso do corpo: total de repetições) |
| `missed()` | Exercício não feito: repete a prescrição; na 2ª semana seguida, volta com 5% a menos |
| `howFor()` | Texto de como aplicar cada técnica de destrave |
| `initState()` | Estado inicial do exercício: carga calculada pelo Perfil (1RM), carga da ficha, ou a definir |
| `maqTol()` | Tolerância, em kg, para uma carga contar como a placa vizinha da prevista |
| `maqMin()` | Peso base da máquina (a carga com que ela começa, sem nenhuma placa): nenhuma carga prevista fica abaixo dele. 0 = sem peso base |
| `maqAll()` | Cargas conhecidas de cada máquina, por nome do exercício: primeiro as declaradas, depois as anotadas nos registros |
| `maqPick()` | Escolhe uma carga conhecida da máquina: a mais próxima de L (dir 0), a seguinte acima de base (dir 1) ou a mais próxima de L abaixo de base (dir -1). Sem candidata, null |
| `maqEntrada()` | Carga que veio da ficha, de outra faixa ou de outro treino (ou qualquer carga fora de placa, em máquina de placas fixas): se há uma placa conhecida perto, usa a placa |
| `maqDir()` | Depois de uma subida ou descida de carga: troca o valor calculado pela placa conhecida seguinte |
| `maqNotas()` | Sinal de placa fixa para a progressão: máquina declarada ou reconhecida como fixa, ou carga usada diferente da prevista mas vizinha dela |
| `eloSets()` | Séries de um registro que servem de sinal: com carga e repetições |
| `eloFonte()` | A sessão mais recente de um irmão feita depois da última sessão do próprio exercício: nesta semana em dia anterior, ou na semana passada em dia posterior |
| `eloAjuste()` | Aplica a conversa ao estado do exercício antes de prescrever a semana w: traz a carga (exercício ainda sem carga) ou move 1 salto (o irmão rendeu bem acima ou bem abaixo do previsto) |
| `replay()` | Replay de todas as semanas, do início até W |

### 4. Observações

Transforma o texto e os blocos de observação em sinais que o motor usa na semana seguinte.

| Dado | O que guarda |
| --- | --- |
| `NOTE_PARTS` | Partes do corpo que o app reconhece no texto e nos blocos de dor: [nome, expressão que encontra a palavra] |
| `CAP` | Blocos de observação: os do app e os criados pelo usuário (com o efeito que ele escolheu) Temas com sub-seleções |
| `T_DOR`, `T_DOR_DIA`, `T_EXEC`, `T_CARGA`, `T_EQUIP`, `T_EQUIP_DIA`, `T_DISP`, `T_TEMPO` | Temas dos blocos de observação, cada um com suas subopções [código, texto]: dor, execução, carga, equipamento, disposição e tempo |
| `TREE_EX` | Temas dos blocos de observação: do exercício e do dia |
| `TREE_DIA` | Temas das observações do dia |
| `TAG_LABEL` | Texto de cada bloco, pelo código |
| `OLD_TAGS` | Blocos da versão antiga → código atual (registros antigos continuam valendo) |
| `TAG_EFS` | Efeitos que uma palavra-chave criada pelo usuário pode ter |

| Função | O que faz |
| --- | --- |
| `readNote()` | Lê o texto livre da observação e devolve os sinais encontrados (dor e onde, equipamento, cansaço, tempo, ajuda, fácil, carga fixa) |
| `userTags()` | Palavras-chave criadas pelo usuário |
| `applyEf()` | Aplica o efeito de uma palavra-chave nos sinais da observação |
| `tagFlags()` | Blocos marcados → sinais para o motor (dor, forte, ajuda, fixa, fácil, pesada, equip, cansaço, tempo) |
| `mergeFlags()` | Junta dois conjuntos de sinais (soma os locais de dor) |
| `noteFlags()` | Junta a observação do exercício e a do dia (a do dia vale para dor só quando diz onde dói) |
| `noteTxt()` | Sinais da observação em texto, para a explicação da semana seguinte |
| `noteEffect()` | O que a observação muda na semana seguinte (texto para o aviso abaixo do campo) |

### 5. Descanso, tempo de treino e ordem dos exercícios

Descanso previsto e feito, recuperação entre séries, duração do treino e ordem em que os exercícios foram feitos.

| Função | O que faz |
| --- | --- |
| `restSec()` | Descanso previsto em segundos (bi-set: o descanso do par) |
| `dayTime()` | Tempo do treino de um dia: previsto (no seu ritmo, ou no padrão com poucos tempos anotados) e registrado; os opcionais ficam à parte (`optSec`) |
| `mmss()` | Segundos em minutos, para o selo do campo: 70 → "1:10 min" |
| `fmtSec()` | Segundos em texto curto: 45 → "45 s", 90 → "1:30" |
| `recov()` | Recuperação entre duas séries: força estimada da seguinte ÷ da anterior (com o RIR, se anotado) (normalizada pelo previsto: back-off mais leve ou com mais folga já é esperado) |
| `recovTxt()` | Recuperação em texto: 96% · boa |
| `restFlags()` | Descanso médio do exercício e se foi curto (abaixo de 75% do previsto) |
| `warmDone()` | Quantas séries de aquecimento foram marcadas (null = nenhuma marcação feita) |
| `warmGap()` | Pausa em segundos depois de uma série de aquecimento feita com a fração f da carga (tab = WARM_PAUSA ou WARM_ANTES) |
| `warmTime()` | Tempo do aquecimento de um exercício: serie = execuções e pausas entre elas; antes = descanso até a 1ª válida. marks = séries marcadas (null = todas) |
| `pausaTxt()` | Pausa curta por extenso: 30 s, 1 min, 1:30, 2 min |
| `ritmo()` | Tempo do treino de um dia: previsto (descanso e execução padrão) e registrado Ritmo real do treino: médias do que foi cronometrado (execução por série, segundos de descanso além do previsto, troca de exercício). Com poucos tempos anotados, cada item fica null e o tempo previsto usa o padrão |
| `dayOrder()` | Ordem em que os exercícios foram feitos: hora do "Feito" (ou da 1ª série anotada); empate = ordem da ficha |
| `recOf()` | Registro de um exercício em uma semana |
| `weekStruggle()` | Em quantos dias da semana houve série abaixo do previsto (decide o deload automático) |

### 6. Plano anual

Tabela do plano do ano, mês a mês: fase, calorias, macros, cardio e refeição livre.

| Dado | O que guarda |
| --- | --- |
| `YEAR` | Uma linha por mês: fase, calorias, proteína (p), carboidrato (c), gordura (g), cardio, refeição livre e peso no início e no fim (w0, w1) |

| Função | O que faz |
| --- | --- |
| `monthName()` | "2027-03" → "mar/27" |

### 7. Perfil e Objetivo: campos e cálculos

Campos do Perfil e do Objetivo e os cálculos que saem deles: gordura, metabolismo, calorias, macros, prazo, fases e cargas iniciais.

| Dado | O que guarda |
| --- | --- |
| `MEDIDAS` | Medidas de fita: [chave, nome] |
| `DOBRAS` | Dobras cutâneas (mm) |
| `METAS_MED` | Metas de medidas: chave da meta -> medida |
| `OPT` | Opções de um campo de escolha, com a opção vazia na frente |
| `PF_PERFIL` | Campos da aba Perfil, por seção: [chave, rótulo, tipo, opções] |
| `PF_OBJ` | Campos da aba Objetivo |
| `OWNER_PERFIL` | Perfil inicial do dono do app (usado só antes do primeiro salvamento) |
| `OBJ_NOME` | Nome da fase conforme o objetivo |
| `num` | Converte para número; vazio ou inválido vira null |
| `r50` | Arredonda para múltiplos de 50 |
| `r5` | Arredonda para múltiplos de 5 |

| Função | O que faz |
| --- | --- |
| `mondayOf()` | Segunda-feira da semana de uma data |
| `dstr()` | Milissegundos → AAAA-MM-DD |
| `dfmt()` | Milissegundos → 10/jan/27 |
| `ageOf()` | Idade, a partir do Perfil |
| `cardioSessionKcal()` | Gasto estimado de uma sessão de cardio (esteira, bicicleta ou elíptico) |
| `bfDobras()` | % de gordura por dobras cutâneas: 7 dobras ou 3 dobras (densidade corporal -> %) |
| `computePlan()` | Calcula ponto de partida e plano com o que estiver preenchido; marca o que é estimativa |
| `medidaAtual()` | Medida atual: registro mais recente da aba Corpo que tenha o campo; senão, o perfil |
| `medidasRef()` | Medidas: atual, referência, sua meta e estimativa na meta |
| `buildPhases()` | Fases do plano: plano anual (dono) ou geradas pelo objetivo |
| `phaseAt()` | Fase do plano em uma data |
| `userPlanPoints()` | Pontos do peso planejado ao longo das fases (linha "Plano" do gráfico) |
| `loadFromMark()` | Carga da semana 1 a partir da carga máxima e das repetições válidas (1RM estimado) |
| `markOf()` | Marca do exercício no Perfil: carga máxima e repetições válidas |
| `computeStartLoads()` | Cargas da semana 1 pelas marcas do Perfil (nada a calcular se a pessoa mantém as cargas da ficha) |
| `applyProfile()` | Aplica o Perfil: data de início, plano, fases, cargas iniciais e semana atual; depois redesenha |

### 8. Telas: Perfil e Objetivo

Formulários e resultados das abas Perfil e Objetivo.

| Dado | O que guarda |
| --- | --- |
| `PFBY` | Campo do formulário pela chave |

| Função | O que faz |
| --- | --- |
| `fieldHtml()` | Um campo do formulário (texto, número, data, escolha, lista de marcar ou texto longo) |
| `pfConfere()` | Confere o rascunho do Perfil antes de salvar: cada campo numérico tem faixa (`PF_FAIXA`; medidas 10–250 cm; sem faixa própria, até `PF_FAIXA_MAX`) e as datas têm de ser plausíveis. Devolve a mensagem do 1º campo errado |
| `sectionsHtml()` | Seções do formulário, que abrem e fecham |
| `cargaRows()` | Linhas da tabela de cargas: um exercício por nome, agrupado pelo treino em que aparece primeiro |
| `rmCells()` | Células de 1RM estimado e carga da semana 1 de um exercício |
| `cargasHtml()` | Tabela de cargas máximas e repetições válidas (1RM) |
| `renderPerfil()` | Desenha o formulário do Perfil |
| `renderObjetivo()` | Desenha o formulário do Objetivo |
| `avisosHtml()` | Lista de avisos (atenção ou informação) |
| `renderPerfilResult()` | Resultado do Perfil: ponto de partida, medidas e o que falta preencher |
| `renderObjResult()` | Resultado do Objetivo: meta, prazo, calorias, macros, cardio e fases |
| `savePerfil()` | Salva Perfil e Objetivo, registra peso e medidas do dia e recalcula tudo |
| `maxMin()` | Limite de tempo por dia, em minutos |
| `ownerPlan()` | A pessoa segue o plano anual? |
| `renderPerfilTab()` | Abre a aba Perfil sem refazer o formulário que já está na tela |
| `renderObjTab()` | Abre a aba Objetivo sem refazer o formulário que já está na tela |
| `pfInput()` | Guarda no rascunho o que foi digitado no Perfil ou no Objetivo e atualiza os resultados |
| `refreshRm()` | Atualiza a coluna de 1RM da tabela de cargas |

### 9. Explicações (botões "i")

Textos das explicações (botões "i") e a janela que os mostra.

| Dado | O que guarda |
| --- | --- |
| `LEGEND` | Legenda das ações: quando aparece e o que fazer |
| `GLOSS` | Textos dos botões "i": chave → [título, explicação] |
| `FITA` | Regra geral da fita métrica, repetida nas medidas |
| `DOBRA_REGRA` | Regra geral das dobras cutâneas, repetida em cada dobra |
| `INFO_COMUM` | Botão "i" que abre a explicação de uma chave Termos de uso comum em prescrição de treino, dieta e medidas: ficam sem o botão "i" (a explicação continua no Glossário, em Plano e regras) |
| `INFO_ACT_COMUM` | Ações óbvias pelo próprio nome (subir, manter, reduzir…) também ficam sem "i"; destrave, nova faixa, conversa entre treinos e festas mantêm |
| `popReturn` | Botão que abriu a janela de explicação (o foco volta para ele ao fechar) |

| Função | O que faz |
| --- | --- |
| `infoRaro()`, `infoBtn()` | O termo merece o botão "i"? |
| `glossEntry()` | Explicação de uma chave (inclui as geradas na hora: ações e técnicas) |
| `openInfo()` | Abre a janela de explicação |
| `closeInfo()` | Fecha a janela e devolve o foco ao botão que abriu |

### 10. Estado do app e acesso aos registros

O que o app guarda na memória e como encontra o registro de um dia ou de um exercício.

| Dado | O que guarda |
| --- | --- |
| `app` | Estado do app: conexão, registros, perfil, plano, o que está aberto na tela e o cronômetro |

| Função | O que faz |
| --- | --- |
| `todayStr()` | Hoje como AAAA-MM-DD (horário do aparelho) |
| `numBr()` | Número digitado à brasileira, com regra estrita: "72,5", "72.5", "1.000", "1.000,5" e "120" valem; "0x10", "1e2", "80O" e letras dão NaN |
| `dataReal()` | Data de registro plausível: existe, é de 2000 em diante e não passa de amanhã (Corpo, exames, CSV, laudo) |
| `msgFalha()` | Escreve a mensagem de falha de gravação e marca o elemento (`data-falha`); quando a fila esvazia, `syncEstado` troca por "Salvo ✓" |
| `initDates()` | Define a semana e o dia de hoje |
| `esc()` | Protege um texto antes de colocá-lo no HTML |
| `actChip()` | Selo de ação (▲ Subir carga…) com o botão "i" |
| `rirTxt()` | Faixa de RIR em texto: 2 ou 2–3 |
| `computeWeeks()` | Prescrição de todas as semanas até upTo, refeita a partir dos registros |
| `docId()` | Código do registro de um dia: w01-A |
| `getDoc()` | Registro do dia (cria, se pedido) |
| `entryFor()` | Registro de um exercício no dia (cria, se pedido) |
| `setStatus()` | Situação de cada série feita contra o previsto |
| `updateHints()` | Atualiza dicas das séries seguintes sem redesenhar o cartão (mantém o foco do teclado) |
| `fillDone()` | "Feito": completa as séries em branco com o previsto (já ajustado pelas séries anteriores) |

### 11. Tela: Semana

A aba Semana: treino do dia, cartão do exercício, aquecimento, parciais, blocos de observação, descanso e tempo.

| Dado | O que guarda |
| --- | --- |
| `WARM_BY` | Aquecimento: séries leves antes da primeira válida (não contam para volume nem progressão) |
| `LADO` | Nome dos lados no unilateral |

| Função | O que faz |
| --- | --- |
| `medeQuadros()` | Desenha a aba Semana: título, alvos de RIR, avisos, dias e o treino do dia Quadros das séries com largura fixa: mede a carga e as repetições mais longas já anotadas (e as da ficha) e fixa as colunas para todos os cartões |
| `renderWeek()` | Desenha a aba Semana: título, alvos de RIR, avisos, dias e o treino do dia |
| `dayBtns()` | Botões dos dias da semana, com a situação de cada um (usados em Semana e em Resumo) |
| `dayStatus()` | Situação do dia: feito, parcial, não feito ou pendente |
| `renderDay()` | Treino do dia: cartões dos exercícios, intervalos, cardio, tempo e observações do dia |
| `warmPlan()` | Séries de aquecimento do exercício: [fração da carga, repetições]; nenhuma se o músculo já foi aquecido antes |
| `warmHtml()` | Aquecimento do cartão, com controle próprio: cada série é um botão de marcar, seguido da pausa indicada; o bloco tem cronômetro e a contagem do descanso antes da 1ª válida |
| `parcHtml()` | Repetições parciais da série: chips (toque remove) e botões de 25/50/75% |
| `tagsHtml()` | Blocos de observação: tocar seleciona; "+ palavra-chave" cria um bloco novo com efeito escolhido |
| `exFlags()` | Sinais das observações do exercício (texto, blocos e descanso) |
| `dayFlags()` | Sinais das observações do dia |
| `refreshTags()` | Redesenha os blocos de observação e o aviso do que o app entendeu |
| `restHtml()` | Descanso entre a série i e a seguinte: tempos fixos; o previsto fica marcado |
| `dayTimeHtml()` | Painel "Tempo do treino": previsto, registrado, descanso médio, recuperação e ordem feita |
| `ritmoHtml()` | Linha do ritmo real, abaixo dos quadros de tempo: o que o cronômetro mediu, quanto custa descansar além do previsto e quanto somam os opcionais |
| `updateDayTime()` | Atualiza o painel de tempo sem redesenhar a tela |
| `ordemLine()` | O que o app aprendeu sobre trocar a ordem deste exercício |
| `transHtml()` | Intervalo entre exercícios (troca de máquina, ajuste, água): fica no exercício que começa depois |
| `setCells()` | Células de carga, repetições e RIR de uma série (side "e"/"d": um dos lados, no unilateral) |
| `execField()` | Campo do tempo de execução, com o selo que converte para minutos |
| `exCard()` | Cartão do exercício: prescrição, motivo, aquecimento, séries, observações e Feito / Não fiz |
| `keepOpen()` | Bloco recolhível que lembra se estava aberto (chave em data-keep); "padrao" abre na primeira vez |

### 12. Tela: Próxima semana

A aba Próxima semana.

| Função | O que faz |
| --- | --- |
| `renderNext()` | Desenha a aba Próxima semana: o que muda em cada exercício e por quê |

### 13. Tela: Progresso

A aba Progresso e seus gráficos.

| Dado | O que guarda |
| --- | --- |
| `dsLine`, `dsDash`, `dsBar` | Estilos padrão das séries: linha cheia, linha tracejada e barra |

| Função | O que faz |
| --- | --- |
| `css()` | Valor de uma cor do tema |
| `chartBase()` | Opções comuns dos gráficos (cores do tema) |
| `mkChart()` | Cria ou recria um gráfico |
| `actGroup()` | Agrupa as ações para o gráfico de prescrições |
| `weekAgg()` | Números de cada semana: séries, volume, exercícios feitos, índice de força, ações, séries por músculo e cardio |
| `weekExtras()` | Números extras de cada semana para os gráficos: execução média por série (s), séries com parciais e diferença entre os lados (% por exercício; positivo = direito mais forte) |
| `nf()` | Número inteiro com separador de milhar |
| `renderProgress()` | Desenha a aba Progresso: blocos de número e gráficos |
| `tile()` | Bloco de número: rótulo, valor e detalhe |
| `renderExChart()` | Gráfico da força estimada de um exercício ao longo das semanas |

### 14. Tela: Corpo e dieta

A aba Corpo e dieta e a pesagem rápida.

| Dado | O que guarda |
| --- | --- |
| `BIO_CAMPOS` | Campos do exame: [chave, rótulo, unidade, grupo]. Grupo "c" = composição; "i" = índices do laudo |
| `BIO_SEG` | Segmentos do corpo na análise segmentar |
| `BIO_FAIXA` | Medidas que trazem faixa normal no laudo (os números entre parênteses) |
| `BIO_REF` | Padrão para estimar as faixas quando o laudo não traz: IMC ideal, parte de gordura ideal e faixa normal de gordura (%), por sexo |
| `BIO_IMC` | faixa normal de IMC |
| `BIO_ZONA` | onde começa e termina a faixa "normal" nas barras, em % da largura |
| `BIO_COL` | largura de cada exame no histórico (px) |
| `BIO_LIN`, `BIO_HIS` | Linhas do histórico e da análise músculo-gordura: [chave, rótulo, unidade, cor, menos é melhor] |
| `BIO_CSV` | Colunas do arquivo exportado pelo aplicativo do InBody: [campo do exame, expressão que reconhece o título da coluna] |
| `BIO_CSV_SEG` | Colunas por segmento do corpo (massa muscular e massa gorda de cada um) |

| Função | O que faz |
| --- | --- |
| `medIdeal()` | Meta ou referência de uma medida, para a linha do gráfico |
| `bioCsvLinha()` | Divide uma linha do CSV respeitando aspas ("72,5" entre aspas, com vírgula como separador) |
| `renderGoal()` | Desenha a aba Corpo e dieta: fase, calorias e macros, peso, ritmo, gordura e medidas |
| `initBodyForm()` | Monta os campos de medidas do formulário de corpo |
| `weightStats()` | Pesagens: média de 7 dias, média anterior e ritmo por semana |
| `dm()` | "2026-11-10" → "10/nov" |
| `updateQW()` | Atualiza a pesagem rápida (média, ritmo e peso do plano hoje) |
| `saveQW()` | Salva a pesagem rápida |
| `bioList()` | Exames salvos, do mais antigo ao mais novo |
| `bioMes()`, `bioDia()` | Mês e ano curtos do exame ("nov/26") e a data por extenso (só mês e ano quando o dia não é conhecido) |
| `bioVal()` | Valor de um campo do exame; o que falta e dá para deduzir é calculado (massa magra = peso − gordura; IMC pela altura do Perfil) |
| `bioFaixas()` | Faixas normais: as do laudo mais recente que as traz; sem laudo, estimadas pela altura e pelo sexo do Perfil |
| `bioN()`, `bioN3()` | Número com casas fixas e vírgula ("22,0"); nos segmentos, três algarismos como no laudo ("3,80", "28,7") |
| `bioPos()` | Posição de um valor na barra (0 a 100%): abaixo, dentro e acima da faixa normal ocupam zonas fixas |
| `bioFig()` | Figura do corpo (desenho simples) com os segmentos pintados na cor do cartão |
| `bioHtml()` | Painel "Evolução corporal": gordura contra a meta, análise músculo-gordura, análise segmentar e histórico |
| `BIO_PESO`, `BIO_GORD`, `BIO_CSV_MAX` | Faixas aceitas do peso (20–300 kg) e da gordura (2–70 %) dos exames, e o tamanho máximo do arquivo (2 MB) |
| `bioCsv()` | Lê o arquivo do InBody e devolve um exame por linha (colunas sem valor ou desconhecidas ficam de fora). Aceita data DD/MM/AAAA ou AAAA-MM-DD e recusa data que não existe; o nome do aparelho fica só com letras, números, espaço, ponto e hífen (40 caracteres); peso fora de `BIO_PESO` recusa a linha, gordura fora de `BIO_GORD` é descartada, outros valores só de 0 a 1000. Arquivo maior que `BIO_CSV_MAX` (2 MB) é recusado e só os 500 últimos exames entram |
| `bioImporta()` | Importa o arquivo: exame novo entra; exame que já existe é atualizado, guardando o que só o laudo tem (% do padrão e faixas) |
| `bioEspelha()` | O peso e a gordura de um exame do período do plano entram também nas medidas do dia |
| `bioFormHtml()` | Formulário do exame: os campos seguem a ordem do laudo |
| `bioGrava()` | Grava (ou apaga) um exame |
| `bioSalva()` | Lê o formulário e salva o exame; o peso e a gordura do exame entram também nas medidas do dia, quando ele é do período do plano |
| `renderBio()` | Desenha o painel e, se ninguém está digitando nele, o formulário do exame |

### 15. Cardio

Cardio: previsto, registro, pace, velocidade e calorias.

| Função | O que faz |
| --- | --- |
| `cardioPlan()` | Cardio previsto na semana: minutos, vezes, inclinação, velocidade e tipo |
| `lastWeight()` | Último peso registrado (ou o do Perfil) |
| `cardioKcal()` | Equações de gasto da caminhada e da corrida (VO2 em ml/kg/min, v em m/min, inclinação em fração): caminhada = 0,1·v + 1,8·v·incl + 3,5 · corrida = 0,2·v + 0,9·v·incl + 3,5 · gasto líquido ≈ 5 kcal por litro de O2 |
| `parseTempo()` | Tempo da esteira: "45", "45,5" ou "45:30" -> minutos |
| `cardioEff()` | Valores efetivos: o digitado; velocidade = digitada, senão distância ÷ tempo, senão a prevista |
| `paceOf()` | Pace em min/km: tempo ÷ distância; sem distância, 60 ÷ velocidade digitada |
| `paceFmt()` | Pace em min:seg |
| `cardioCard()` | Cartão do cardio do dia |
| `paceInfo()` | Pace automático: tempo ÷ distância; sem distância, pela velocidade digitada; sem nada, o previsto |
| `cardioOutHtml()` | Caixas de pace, velocidade e calorias |
| `updateCardioOut()` | Atualiza as caixas do cardio enquanto a pessoa digita |
| `cardioEntry()` | Registro do cardio do dia (cria, se preciso) |
| `fillCardio()` | "Feito" no cardio: completa os campos em branco com o previsto |
| `cardioWeek()` | Cardio da semana: minutos, dias, distância, pace e calorias |
| `planAt()` | Peso planejado em uma data |

### 16. Tela: Plano e regras

A aba Plano e regras.

| Função | O que faz |
| --- | --- |
| `gloHtml()` | Desenha a aba Plano e regras: legenda, destraves, semanas do bloco e plano anual Glossário: todos os termos do app, com busca; cada um abre a mesma explicação do botão "i" |
| `renderPlan()` | Desenha a aba Plano e regras: legenda, destraves, semanas do bloco e plano anual |

### 17. Exercício unilateral

Modo unilateral: dados de cada lado e qual lado conta para a progressão.

| Função | O que faz |
| --- | --- |
| `uniDefault()` | Unilateral por padrão? (pelo nome, ou pela observação "cada perna / cada lado") |
| `uniModo()` | Modo do exercício: `bi` (bilateral), `lados` (unilateral, cada lado anotado) ou `junto` (unilateral com os lados juntos). A escolha do usuário (`settings.uni[k]`: `false`, `true` ou `"junto"`) vale mais que o padrão |
| `isUni()` | Unilateral de qualquer jeito (cada lado ou lados juntos): conta no tempo previsto como unilateral |
| `uniLados()` | Unilateral com cada lado anotado à parte (linhas de esquerdo e direito, cronômetro por lado) |
| `uniJunto()` | Unilateral com os lados juntos: uma linha por série; o tempo anotado é o dos dois lados e a troca |
| `serieUni()` | A série foi feita como unilateral? (lados anotados, marca `uj` da série ou exercício hoje em lados juntos) |
| `uniLinkHtml()` | Botãozinho 🔗 à vista no cartão dos unilaterais: liga ou separa os lados |
| `uniSum()` | Soma de um campo dos dois lados (ex.: tempo de execução) |
| `uniKg()` | Carga de um lado; se estiver vazia, a do outro |
| `uniScore()` | Força estimada de um lado na série |
| `uniAgg()` | A série vale pelo lado mais fraco (empate: o que você marcou como mais difícil) |
| `maqBoxHtml()` | Bloco "Cargas da máquina" do cartão: as placas conhecidas, o tipo (automático, placas fixas, carga livre) e o campo para declarar as placas |
| `maqSet()` | Guarda o tipo ou as placas da máquina (vale para o mesmo exercício em todos os dias) |
| `uniToggleHtml()` | Botões Bilateral / Unilateral (cada lado) / Unilateral (lados juntos) do cartão |
| `uniLine()` | Diferença entre os lados no exercício de hoje |
| `uniDifHtml()` | Botões "Mais difícil: Esquerdo / Igual / Direito" |
| `uniRows()` | Linhas da série unilateral: esquerdo e direito com carga, reps e RIR; depois cronômetro, execução, parciais e dificuldade |

### 18. Cronômetro

Cronômetro de execução, descanso e intervalo, com aviso de esquecimento e animação.

| Dado | O que guarda |
| --- | --- |
| `clkAudio` | Saída de som do aviso (criada no primeiro toque, exigência dos navegadores) |
| `clkRaf` | Pedido de quadro da animação em andamento (null = parada) |
| `wakeLock` | Tela acesa enquanto o cronômetro roda (para o aviso de 5 minutos tocar). A preferência fica no aparelho. |

| Função | O que faz |
| --- | --- |
| `clkLoad()` | Lê o cronômetro guardado no aparelho |
| `clkSave()` | Guarda o cronômetro no aparelho |
| `clkEl()` | Segundos decorridos da fase atual |
| `clkFmt()` | Segundos como m:ss |
| `parseMmss()` | "1:30" ou "90" → segundos |
| `clkRecord()` | Anota um tempo no registro: exec (execução), desc (descanso), troca (entre os lados) ou trans (intervalo entre exercícios) |
| `clkLeft()` | Exercícios do dia que ainda faltam (fora o atual) |
| `clkUnlock()` | Libera o som no primeiro toque (exigência dos navegadores) |
| `clkBeep()` | Aviso sonoro e vibração |
| `clkToast()` | Mensagem curta acima da barra do cronômetro |
| `clkHandoff()` | Fecha a fase em andamento quando você começa a série (m, j) e anota o que couber |
| `clkExecMax()` | Teto da execução cronometrada: nos lados juntos cabe o tempo dos dois lados e a troca (5:30); nos outros, `CLK_EXEC_MAX` |
| `clkGo()` | Começa a cronometrar a execução da série (m = exercício, j = série) |
| `clkAfterExec()` | Depois da execução: unilateral → troca de lado (com barra de descanso própria); os dois lados feitos → descanso da série |
| `clkToRest()` | Descanso entre séries; depois da última série, intervalo até o próximo exercício (ou fim do dia) |
| `clkStop()` | Para a execução, anota o tempo e passa para a fase seguinte |
| `clkOnDone()` | "Feito" no cartão: o intervalo até o próximo exercício começa (do fim da última série, se cronometrada) |
| `clkOnInput()` | Anotou carga ou reps em outro exercício durante o intervalo: você já está no próximo |
| `clkAct()` | Trata os botões do cronômetro: go, warm, first, stop, side, skip, next, end, see, keep, drop, fix e fixok |
| `clkRefresh()` | Redesenha as barras do cronômetro |
| `clkRing()` | Animação do tempo passando: barra enchendo até o previsto e ponteiro de segundos |
| `clkPlan()` | Tempo previsto da fase atual, para a barra de progresso |
| `clkWarmRec()` | Grava o tempo do aquecimento do exercício; quem não marcou nenhuma série fica com todas marcadas. antes = 0 quando o descanso até a 1ª válida já está dentro desse tempo |
| `clkPreRec()` | Grava o descanso entre o fim do aquecimento e a 1ª série válida |
| `clkWarm()` | Começa a cronometrar o aquecimento de um exercício (fecha o intervalo ou a fase que estiver rodando) |
| `clkFrame()` | Um quadro da animação: enche a barra e gira o ponteiro |
| `clkAnim()` | Liga a animação se houver cronômetro rodando |
| `clkSetHtml()` | Barra embaixo de cada série (unilateral: um lado, troca de lado com contagem própria, o outro lado, descanso) |
| `clkBar()` | Barra fixa embaixo da tela enquanto o cronômetro roda |
| `clkTick()` | A cada meio segundo: atualiza os números e dispara o aviso de 5 minutos |
| `wakeOn()` | A pessoa quer a tela acesa? (padrão: sim) |
| `wakeSync()` | Liga ou solta o bloqueio de tela conforme o cronômetro, a preferência e a página visível |
| `wakeHtml()` | Botões Sim/Não da preferência, no painel de tempo do treino (só onde o navegador tem o recurso) |
| `clkDock()` | Mostra a barra de baixo fora da aba Semana enquanto o cronômetro roda |

### 19. Tela: Resumo do dia

A aba Resumo do dia: métricas, direcionamentos e exercício por exercício.

| Função | O que faz |
| --- | --- |
| `sumNext()` | O que o motor prevê para a próxima vez do exercício |
| `sumEx()` | Números de cada exercício do dia: séries, volume, desempenho, RIR, descanso, execução, sinais, ordem e lados |
| `pctTxt()` | Fração como porcentagem com sinal: +2,5% |
| `kgTxt()` | Quilos sem casas decimais: 1.240 kg |
| `dayStats()` | Números do dia, usados pelo resumo e pela imagem de compartilhar: exercícios, séries, volume, desempenho, tempo e cardio |
| `recordes()` | Recordes do dia: exercícios em que a melhor série (1RM estimado por `e1`) passou da melhor de todas as semanas anteriores, contando o mesmo exercício em qualquer dia (mesmo nome). A primeira vez de um exercício não é recorde; carga zero fica de fora. Aparece nos Direcionamentos como "🏆 Recorde" |
| `daySummaryHtml()` | Monta o resumo de um dia: resultado, blocos de número, direcionamentos e exercício por exercício |
| `renderResumo()` | Desenha a aba Resumo do dia |

### 20. Tela: Início

A página inicial (o que é o app) e o painel flutuante Hoje (treino do dia, números e pesagem).

| Função | O que faz |
| --- | --- |
| `todayPlan()` | Semana e treino de hoje |
| `homeToday()` | Card de hoje e números principais |
| `homeLogo()` | Logo grande, com os traços animados |
| `hojeFill()` | Painel "Hoje": abre, fecha e atualiza (treino do dia, números e pesagem). O rótulo do botão mostra o andamento do dia. |
| `hojeOpen()` | Abre o painel Hoje |
| `hojeClose()` | Fecha o painel Hoje (com a animação de saída) |
| `renderHome()` | Desenha o Início: apresentação do app, o nome, por que registrar, como funciona e atalhos |

### 21. Navegação e animações

Troca de abas, barra de baixo, subabas e animações de entrada.

| Dado | O que guarda |
| --- | --- |
| `NAVG` | Grupos da barra de baixo: nome, subabas e ícone |
| `TABS` | Todas as abas |
| `RM` | O aparelho pede menos movimento? |

| Função | O que faz |
| --- | --- |
| `setTab()` | Troca de aba: marca a navegação, mostra a tela e anima a entrada |
| `renderOnb()` | Aviso de boas-vindas para quem ainda não preencheu o Perfil |
| `render()` | Redesenha a aba atual (espera a pessoa terminar de digitar, se for o caso) |
| `routeFromHash()` | Lê a aba no endereço (#semana, #resumo…) |
| `navGroup()` | Grupo a que uma aba pertence |
| `renderNav()` | Desenha a barra de baixo e as subabas |
| `subnavInd()` | Indicador que desliza até a subaba escolhida |
| `animSwap()` | Anima a entrada de um bloco |
| `countUp()` | Números que contam até o valor (só na entrada da tela) |
| `revealAll()` | Blocos que aparecem ao rolar |

### 22. Fotos e compartilhar

Fotos do treino e de evolução, a imagem do resumo para compartilhar e a gravação dos registros avulsos.

| Função | O que faz |
| --- | --- |
| `putDoc()` | Guarda um documento avulso (diário, fotos): aparece na hora e segue para o armazenamento. Apagar = gravar { apagado: true }. |
| `subExtras()` | Liga as coleções avulsas ao armazenamento: o diário e os registros das fotos |
| `isoLocal()` | Data local (AAAA-MM-DD) e data e hora local (AAAA-MM-DDTHH:MM) de um instante |
| `nowLocal()` | Data e hora local de agora, no formato dos campos de data e hora |
| `dCurta()` | "2026-11-12" → "12/nov" |
| `fotoShrink()` | Reduz a foto para no máximo FOTO_MAX px de lado e devolve um JPEG |
| `fotoAdd()` | Envia a foto escolhida e guarda o registro: tipo "treino" (com semana e dia) ou "evolucao" |
| `fotosDe()` | Fotos de um tipo, da mais nova para a mais antiga (f = filtro opcional) |
| `fotoThumbs()` | Miniaturas. O endereço da imagem é um link temporário que chega depois: fotoFill preenche cada <img> |
| `fotoUrl()` | Link temporário de uma foto (pedido uma vez e reaproveitado) |
| `fotoFill()` | Preenche as miniaturas que ainda estão sem imagem |
| `fotoDiaHtml()` | Painel "Foto do treino" do dia aberto na aba Semana |
| `fotoEvoHtml()` | Painel "Fotos de evolução" da aba Corpo e dieta: primeira contra a mais recente, e todas |
| `fotoDel()` | Apagar foto: o primeiro toque pede confirmação no próprio botão; o segundo apaga |
| `loadImg()` | Carrega uma imagem para desenhar no canvas |
| `shareDraw()` | Desenha a imagem (1080 × 1350) com os números do dia e, se houver, a foto do treino no topo |
| `sharePrep()` | 1º toque: prepara a imagem e mostra a prévia. 2º toque (botões da prévia): envia ou salva. O envio precisa de um toque próprio para o celular permitir. Salvar usa app.salvar, ligado na conexão; onde ele não existe, a saída é segurar o dedo sobre a imagem. |
| `shareAct()` | Botões de compartilhar: preparar a prévia, fechar, enviar e salvar |

### 23. Hormônios e peptídeos

A aba Hormônios e peptídeos: dados de referência para estudo, substâncias, aplicações, calendário, controle de doses, curva pela meia-vida e exames.

| Dado | O que guarda |
| --- | --- |
| `DJ_MARC` | Marcadores comuns de exame e a unidade usual. A faixa de referência é a do laudo, digitada por quem registra. |
| `DJ_GRUPOS` | Grupos da referência: chave, título e nota do grupo |
| `DJ_T` | Hormônios com dados de bula. n nome · g grupo · un unidade usual · via · mv meia-vida em dias, só quando a fonte traz um valor único · mvT meia-vida por extenso · bula intervalo na indicação aprovada · nota · fonte |
| `DJ_FICHAS` | Fichas do material de estudo: [ficha, nome, grupo, classe, via, frequência ou uso descrito, meia-vida descrita, meia-vida em dias quando há um valor único] |
| `DJ_BULA` | O que foi conferido em bula ou estudo para substâncias que também têm ficha: entra por cima do dado da ficha |
| `DJ_M` | Citados só nas tabelas do material, sem ficha: [nome, classe, via, padrão descrito] |
| `DJ_REF` | Lista final da referência: hormônios, fichas (com o conferido em bula por cima) e os citados sem ficha |
| `DJ_LOCAIS` | locais de aplicação oferecidos na lista |
| `DJ_PDF` | leitor de PDF, carregado só quando um laudo é escolhido |
| `DJ_PDF_SHA` | Impressão digital (SHA-384) de cada arquivo do leitor de PDF, tirada do pacote oficial no npm. Ao trocar a versão do leitor, troque também estes valores |
| `DJ_LER` | Como cada marcador aparece nos laudos: [marcador, expressão que acha o nome] |
| `DJ_LER_JANELA` | caracteres depois do nome do marcador em que o valor é procurado |

| Função | O que faz |
| --- | --- |
| `djSlug()`, `djNota()` | Anotação do usuário sobre uma substância da referência (posologia, correções): fica na conta dele, não no código |
| `djEscala()` | Exame de contagem em "mil/mm³" (plaquetas): valor e faixa de referência sempre nessa escala (número acima de 1.000 é /mm³ e é dividido). Usada ao ler o laudo, ao salvar à mão e ao mostrar, então registros antigos em duas escalas aparecem certos sem regravar |
| `djNotaSave()` | Salva a anotação digitada no cartão |
| `djVia()` | Via da referência → opção do cadastro |
| `djDias()` | Tempo por extenso a partir de dias: 0,43 → "10 h"; 40 → "40 dias" |
| `djRefHtml()` | Painel da referência: busca, um bloco recolhível por grupo com um cartão por substância, e as notas de leitura, exames e regras |
| `djCtrlHtml()` | Controle de doses: por substância, a última aplicação, os totais de 7 e 30 dias e o estimado hoje pela meia-vida |
| `djIntSet()` | Intervalo do cadastro: a escala fixa ou, em "outro", o número de dias digitado |
| `djApGuia()` | Guia da aplicação: unidade da substância, sua posologia anotada, a última aplicação e as doses que você já registrou (toque para repetir) |
| `djExGuia()` | Guia do exame: unidade e referência do último exame daquele marcador já vêm preenchidas |
| `djTipo()` | Selo do tipo da substância, quando ela veio da lista de referência: hormônio (grupo "t") ou peptídeo |
| `djList()` | Itens do diário de um tipo ("sub" substância, "ap" aplicação, "ex" exame), em ordem de data (ou de nome) |
| `djLevel()` | Quantidade estimada de uma substância no instante t: soma das doses, cada uma caindo pela metade a cada meia-vida informada. Modelo didático, não é nível no sangue. |
| `djNext()` | Próxima aplicação prevista: a última mais o intervalo informado (dias) |
| `djCalHtml()` | Calendário do mês: ponto cheio = aplicação feita; ponto vazado = próxima prevista |
| `djDelBtn()` | Botão de apagar com confirmação no segundo toque |
| `renderDiario()` | Desenha a aba Hormônios e peptídeos: referência, substâncias, calendário, aplicações, controle de doses, curva estimada e exames |
| `djAct()` | Ações do diário: trocar o mês do calendário, copiar da referência para o cadastro, apagar (com confirmação) e salvar substância, aplicação ou exame |
| `djNum()` | Número escrito à brasileira ("1.234,5" ou "45,2") para número |
| `djRefTxt()` | Faixa de referência escrita no laudo: prefere o trecho do sexo do Perfil e a faixa na mesma unidade do resultado. Devolve [mínimo, máximo]; um deles pode faltar |
| `djAcha()` | Procura os marcadores no texto de um laudo. Primeiro os laudos em blocos (título do exame, "Método" e uma linha "RESULTADO: valor unidade"); o que não veio em bloco é procurado no texto corrido (tabelas, como a do hemograma): o primeiro número depois do nome é o valor |
| `djPdfArq()` | Baixa um arquivo do leitor de PDF, confere a impressão digital (`DJ_PDF_SHA`) e devolve um endereço local (blob:). Arquivo diferente do conferido não roda |
| `djPdfTexto()` | Texto de um PDF, linha por linha (as palavras de mesma altura na página formam uma linha). Carrega o leitor uma vez por sessão (`djPdfLib`), com `isEvalSupported: false` |
| `djLerHtml()` | Lista de conferência do que foi lido no laudo: cada marcador com valor, unidade e referência editáveis |
| `djLerArquivo()` | Lê o laudo escolhido e mostra a lista de conferência |

### 24. Edição e salvamento

Tudo o que acontece quando você toca ou digita, e a gravação dos dados.

| Dado | O que guarda |
| --- | --- |
| `settingsSaving` | Ajustes: um salvamento por vez |
| `settingsPending` | Ficou um salvamento de ajustes na fila enquanto outro estava em curso |

| Função | O que faz |
| --- | --- |
| `ctx()` | Exercício pronto para editar: definição, prescrição da semana e registro (i = garante que a série i existe; W = outra semana) |
| `snapPrev()` | Guarda no registro o que estava previsto, enquanto o exercício não foi fechado (force: sempre) |
| `sideOf()` | Dados de um lado (e/d) da série unilateral |
| `currentPres()` | Prescrição do exercício na semana aberta |
| `touch()` | Marca o dia como alterado e agenda o salvamento (1,2 s depois da última mudança) |
| `flush()` | Salva o registro de um dia; se falhar, tenta de novo |
| `saveErr()` | Mensagem de erro ao salvar (retry: o app tenta de novo sozinho) |
| `setSaveMsg()` | Mensagem da barra de salvar Situação do salvamento: aparece na barra de baixo só enquanto importa (salvando, erro); "Salvo" some sozinho |
| `tapSel()` | Seletor do botão tocado, para reencontrá-lo depois de redesenhar |
| `playTap()` | Efeito de toque no botão |
| `flashBtn()` | Mostra "✓ Salvo" no botão por um instante |
| `parcEdit()` | Parciais de uma série: abre o campo, inclui (pv = % do movimento) ou remove (pv = posição); ps = lado, no unilateral |
| `saveSettings()` | Salva os ajustes (deload escolhido, palavras-chave e exercícios unilaterais) |

### 25. Conexão com o armazenamento

Login e ligação com o banco de dados: carrega os dados da pessoa e grava cada mudança.

| Função | O que faz |
| --- | --- |
| Dado | O que guarda |
| --- | --- |
| `SB_URL`, `SB_KEY` | Endereço do projeto e a chave **pública** do Supabase (feita para ficar no navegador; quem protege os dados são as regras RLS) |
| `SENHA_MIN` | Tamanho mínimo da senha ao **criar** conta (8). Contas antigas com 6 continuam entrando. O mínimo que vale de verdade é o do painel do Supabase |

| Função | O que faz |
| --- | --- |
| `makeDb()` | Mesma interface de coleções e documentos que o app usa. **Carga**: lê `hexis_docs` de 1.000 em 1.000, só as linhas do próprio `user_id` (`.eq("user_id", uid)`, além do RLS) e só caminhos válidos (recusa `__proto__`). **Gravação**: uma fila por documento (duas mudanças seguidas chegam na ordem e a última vence). A fila também fica **guardada no aparelho** (`FILA_KEY` + id da pessoa) até o servidor confirmar: se o celular fechar o app sem sinal, o que faltava volta na próxima abertura e é enviado logo depois da carga. Falha de rede: nova tentativa em 3 s, dobrando até 30 s, e na hora em que a rede volta (`online`). Sessão expirada ou recusa do servidor: a gravação **continua na fila** (tentativas até 5 min) e segue assim que a sessão renova (`retenta`). A primeira falha é devolvida para a tela avisar. Antes de a carga terminar, recusa gravar (`nao_carregou`). Valores da fila valem mais que os recém-carregados. Métodos extras: `pronto()`, `pendentes()`, `agora()`, `retenta()`, `descarta()` (Sair: tira a fila do aparelho), `vazia(ms)` (espera a fila esvaziar) e `dump()` |
| `syncEstado()` | Cabeçalho: "Sincronizado", "Salvando…", "Não salvo · tentando de novo" ou "Não salvo · entre de novo" (sessão recusada). Quando a fila esvazia depois de uma falha, troca por "Salvo ✓" as mensagens de erro que ficaram nas telas (`msgFalha`), na barra, na dieta e na bioimpedância |
| `dataOk()` | Confere se uma data AAAA-MM-DD existe de verdade (recusa 2026-13-45). Usada no CSV, nos exames, no Corpo e na carga |
| `corpoRuimAviso()` | Na carga, registros de Corpo com data inválida ficam fora das contas (`app.corpoRuim`) e aparecem num aviso com botão para apagar |
| `gravaJa()` | Ao esconder ou fechar o app (`visibilitychange`, `pagehide`), grava na hora o que esperava os 1,2 s. Com gravação pendente, fechar a aba pede confirmação (`beforeunload`) |
| `numCampos()` | Troca os campos `type=number` por texto com `inputmode="decimal"`: o teclado numérico continua e a vírgula vira ponto ("72,5" → 72,5). Um observador repete a troca nos campos que as telas criam depois |
| `lgSay()`, `lgErr()` | Mensagem da tela de login e o erro do Supabase em português |
| `showLogin()`, `hideLogin()` | Mostram e escondem a tela de login |
| `lgGo()` | Entrar ou criar conta com e-mail e senha |
| `exportaDados()` | Seus dados (Plano e regras): baixa a cópia de segurança completa em JSON ou as séries em planilha CSV (separador `;`, abre no Excel com acentos). Tudo é gerado no aparelho |
| `connect()` | Liga no Supabase e espera a sessão. `detectSessionInUrl: false`: um link com sessão no endereço não troca a conta (o app só entra por e-mail e senha) e o endereço é limpo. As gravações feitas com a tela escondida saem com `keepalive`, para o navegador não cortá-las. Sessão renovada reenvia a fila; sessão de outra conta ou encerrada em outra aba recarrega |
| `startSession()` | Sessão aberta: deixa a tela em espera (`body.carregando`, nada é editável) até os dados chegarem; se a carga falha, mostra o aviso com o botão "Tentar de novo" (`#connRetry`) e continua sem gravar nada. Liga as fotos (só caminhos da pasta da própria pessoa, links de 10 min) e busca de novo ao voltar para o app só se não houver gravação pendente. Se a sessão acaba em outra aba (`SIGNED_OUT`), recarrega |

**Sair** (2ª banca): primeiro tenta gravar o que falta (até 4 s; sem rede, não espera). Se ainda sobrar algo, pergunta "N registros ainda não foram gravados… Sair mesmo assim?". Depois apaga do aparelho a fila, o cronômetro, os blocos abertos e a tela acesa (`FILA_KEY`, `CLK_KEY`, `KEEP_KEY`, `WAKE_KEY`), encerra a sessão **no aparelho** (`signOut({ scope: "local" })`, que funciona sem internet; o token é apagado à mão se a biblioteca não responder em 2,5 s) e recarrega uma vez só (`app.saindo`).

O cadastro responde a mesma frase quando o e-mail já tem conta ou é novo ("Se este e-mail ainda não tinha conta, enviamos um link de confirmação. Se já tinha, toque em Entrar."), também quando o erro vem do servidor (`lgErr`), para não revelar quem usa o app. Senha recusada pelo servidor diz o mínimo de `SENHA_MIN`. E-mail sem @ recebe "Confira o e-mail".

**Onde cada gravação avisa a falha** (todas passam por `saveErr()`, que escreve a barra e o cabeçalho): treino (`flush`), dieta (`dtGrava`), ajustes (`saveSettings`), documentos avulsos como diário e fotos (`putDoc`, devolve verdadeiro ou falso), exames de bioimpedância (`bioGrava`, "Exame salvo ✓" só depois de gravar) e o espelho em Corpo (`bioEspelha`).

### 26. Dieta: diário de alimentos com a tabela TACO

A aba Dieta: a tabela TACO embutida, o registro de alimentos por refeição e as comparações por refeição e por dia da semana contra a meta.

| Dado | O que guarda |
| --- | --- |
| `TACO_CAT`, `TACO` | Tabela TACO (Tabela Brasileira de Composição de Alimentos, 4ª edição revisada e ampliada, NEPA/UNICAMP, 2011). Cada linha vale para 100 g de parte comestível: [nome, grupo, kcal, proteína g, carboidrato g, gordura g, fibra g]. kcal −1: a tabela não traz a composição desse alimento |
| `DT_REF` | Refeições padrão conforme o número de refeições do Perfil: nome e parte das calorias do dia (%) |
| `DT_RES` | alimentos mostrados na busca |
| `DT_FOLGA` | até 10% acima ou abaixo do proposto conta como "dentro" |
| `DT_PER`, `DT_SEM` | períodos (dias) da comparação por refeição |
| `DT_GRAMAS` | quantidades prontas (g) na escolha do alimento |
| `DT_PORCOES` | quantidades prontas para alimento por porção |
| `DT_FREQ` | alimentos frequentes mostrados antes de digitar |
| `DT_HORA` | Horário típico de cada refeição pelo nome (horas do dia), para a refeição já vir escolhida |

| Função | O que faz |
| --- | --- |
| `semAcento()` | Texto sem acento e em minúsculas, para a busca |
| `dtS()` | Estado da aba Dieta: menu aberto, dia mostrado, refeição recebendo alimento, busca e alimento escolhido |
| `dtRefAgora()` | Refeição da hora: a do horário típico mais próximo; nomes sem horário conhecido dividem o dia (5h às 23h) em partes iguais |
| `dtCurto()` | Nome curto da refeição para os botões |
| `dtAdd()` | Soma dias a uma data AAAA-MM-DD |
| `dtDow()` | Dia da semana de uma data (0 = segunda) |
| `dtKc()` | Calorias arredondadas, com ponto de milhar |
| `dtMeta()` | Meta do dia pela fase do plano: calorias (primeiro número do texto da fase) e macros |
| `dtRefs()` | Refeições do dia: as que você definiu ou o padrão pelo número de refeições do Perfil |
| `dtPartes()` | Parte de cada refeição no dia, ajustada para as partes somarem 1 |
| `dtMeus()` | Alimentos cadastrados por você (fora da tabela) |
| `dtFood()` | Alimento pela chave: número = linha da TACO; texto = alimento seu |
| `dtCalc()` | Calorias e macros de uma quantidade (gramas, ou número de porções quando o alimento é por porção) |
| `dtSum()` | Soma de calorias e macros de uma lista de itens |
| `dtItens()` | Alimentos registrados em um dia |
| `dtBusca()` | Busca: todas as palavras digitadas têm de aparecer no nome. Ordem: seus alimentos; a forma do dia a dia da palavra comum (`DT_PREF`, por exemplo "arroz" → "Arroz, tipo 1, cozido"); o nome-base exato ("Maçã, …"); a 1ª palavra igual; o que começa pela palavra; o resto. Desempate pelo nome mais curto |
| `DT_PREF` | Palavra comum → começo do nome na TACO que deve vir primeiro (leite, arroz, feijão, ovo, frango, pão, batata, banana, patinho, aveia, iogurte, queijo). A TACO não tem os valores do leite de vaca integral líquido: crie em "Meus alimentos" |
| `dtUsados()` | Alimentos que você mais registrou (aparecem antes de digitar), cada um com a última quantidade usada |
| `dtUltQ()` | Última quantidade usada de um alimento (ou 100 g / 1 porção) |
| `dtQtd()` | Quantidade com unidade: "150 g" ou "1,5 porção" |
| `dtGrava()` | Grava um registro da dieta (um dia ou a lista dos seus alimentos) |
| `dtPoe()` | Grava a lista de alimentos de um dia |
| `dtSit()` | Situação contra o proposto: dentro (até 10%), acima ou abaixo, com a cor de cada uma |
| `dtBar()` | Barra de progresso: ingerido contra o proposto |
| `dtDif()` | Diferença com sinal e situação: "+120 · acima" |
| `DT_GASTO_DIAS`, `DT_GASTO_MIN`, `DT_GASTO_PESOS`, `DT_KCAL_KG` | Gasto real estimado: janela de 21 dias terminando ontem; mínimo de 10 dias com alimentos e 4 pesagens; 7.700 kcal por kg de peso |
| `dtGasto()` | Gasto real estimado pelos registros: média das calorias dos dias anotados menos a variação de peso (reta pelas pesagens da janela) × 7.700. `null` com poucos dados |
| `dtGastoHtml()` | Bloco "Gasto real estimado" no alto da visão "Por dia", com a comparação com a meta da fase. Só informa: não muda a meta |
| `renderDieta()` | Desenha a aba Dieta: meta da fase, menu e a visão escolhida |
| `dtDiaHtml()` | Visão "Registrar": o dia, o total contra a meta e os alimentos de cada refeição |
| `dtHeroHtml()` | Cartão do total do dia: anel das calorias contra a meta, o que falta e os macros |
| `dtItemHtml()` | Linha de um alimento registrado: nome, quantidade (editável), calorias e remover |
| `dtPrevTxt()` | Texto da prévia: calorias e macros da quantidade digitada |
| `dtResHtml()` | Lista da busca (ou os mais usados, antes de digitar) |
| `dtAddHtml()` | Caixa de adicionar alimento: busca, escolha e quantidade; ou o formulário de alimento fora da tabela |
| `dtRefHtml()` | Visão "Por refeição": média de cada refeição nos dias com registro, contra o proposto |
| `dtSemHtml()` | Visão "Por dia da semana": barras do total de cada dia contra a meta e a tabela refeição × dia |
| `dtCfgHtml()` | Bloco "Refeições e proposto": nomes e parte de cada refeição nas calorias do dia |
| `dtRefsSalva()` | Guarda as refeições definidas (ou volta ao padrão) nos ajustes |
| `dtOk()` | Adiciona o alimento escolhido à refeição aberta |
| `dtPoeItem()` | Põe um alimento na refeição escolhida e deixa a busca pronta para o próximo |
| `dtForaSalva()` | Cadastra um alimento fora da tabela e já deixa escolhido para informar a quantidade |
| `dtClick()` | Toques na aba Dieta |
| `dtInput()` | Digitação: a busca atualiza só a lista; a quantidade atualiza a prévia |
| `dtChange()` | Campos alterados: dia, quantidade de um item e nomes ou partes das refeições |

### 27. Partida

Ordem em que o app liga.

| Função | O que faz |
| --- | --- |
| `travaTela()` | O app fica nos limites da tela: sem zoom de pinça nem de toque duplo, sem zoom ao entrar num campo e sem rolagem para os lados |
| `largos()` | Elementos que passam da largura da tela (conferência) |

## 7. Como fazer as mudanças mais comuns

**Depois de qualquer mudança no programa** (desde 18/nov): `python3 testes/atualiza-csp.py partes`, depois `cat partes/p*.txt > completo/index.html` e `testes/rodar-tudo.sh`. Sem o primeiro passo, a CSP recusa o programa e o app não abre.

**Trocar ou incluir um exercício.** Seção 2, lista `EX`. Cada linha é `ex("código", "dia", "Nome", "Músculo", { opções })`. O que não for informado nas opções usa o padrão:

| Opção | Padrão | Significado |
| --- | --- | --- |
| `sc` | `"st"` | `"st"`: séries iguais. `"top"`: 1 série pesada + back-offs |
| `sets` | `3` | Número de séries |
| `lo`, `hi` | `8`, `10` | Faixa de repetições |
| `kg` | vazio | Carga inicial. Vazio = "definir carga" na primeira semana |
| `inc` | `2.5` | Quanto a carga sobe quando progride (kg) |
| `mic` | `1.25` | Microcarga, usada no destrave |
| `step` | `0.5` | Arredondamento da carga |
| `t` | `"iso"` | `"iso"` isolador ou `"comp"` composto (muda o RIR-alvo) |
| `rest` | `"2 min"` | Descanso previsto: `"90 s"` ou `"2 min"`. **Máximo de 2 min** (regra do usuário, 18/nov). Uma faixa como `"2–3 min"` conta pelo maior número (`restSec`) e o teste `banca.js` acusa qualquer texto acima de 2 min |
| `obs` | vazio | Observação fixa do exercício |
| `bw` | não | Exercício com o peso do corpo |
| `opt` | não | Opcional (não conta para fechar o dia) |

O código (`A1`, `C2f`) identifica o exercício nos registros. Trocar o código de um exercício que já tem registros faz o app deixar de enxergá-los.

**Mudar o descanso previsto.** Mesma lista, opção `rest` do exercício.

**Mudar as semanas do primeiro bloco.** Seção 2, lista `WT` (metas de RIR e séries de core por semana) e o parâmetro `BLOCK_W`.

**Mudar os blocos do resto do ano.** Seção 2, lista `BLOCOS_ANO`: cada linha tem a segunda-feira em que o bloco começa, o nome, a faixa da série pesada, a rampa de RIR, os músculos prioritários e as semanas leves. O parâmetro `MAXW` diz até que semana o app calcula.

**Criar um bloco de observação.** Seção 4: inclua a subopção no tema certo (`T_DOR`, `T_EXEC`, `T_CARGA`, `T_EQUIP`, `T_DISP`, `T_TEMPO`), o texto em `TAG_LABEL` e, se o bloco deve mudar o treino, o efeito em `tagFlags()`.

**Criar uma explicação "i".** Seção 9: `GLOSS.chave = ["Título", "<p>texto</p>"]`. Para mostrar o botão, use `infoBtn("chave")` no ponto da tela.

**Criar uma aba.** Seção 21: inclua a aba em um grupo de `NAVG`. No HTML, crie `<section class="view" id="v-nome">`. No programa, escreva a função que desenha a tela e inclua uma linha para ela em `render()`.

**Mudar um aviso do Resumo do dia.** Seção 19, função `daySummaryHtml()`: cada direcionamento é uma linha `dir.push([...])`. Os limites ("acima", "abaixo", descanso curto) são parâmetros da seção 1.

## 8. Versão web: como é gerada e publicada

> **Atenção (18/nov):** as correções das revisões de 18/nov (segurança, gravação, bancas) foram feitas direto no `index.html` deste pacote (`partes/`). A versão de origem e o `build_web.py` **não** as têm. **Não gere o site de novo pelo `build_web.py`**: isso desfaria todas as correções. A fonte da verdade agora são as `partes/` deste pacote; o texto abaixo descreve como o site foi gerado antes.

O site é gerado da versão de origem pelo `build_web.py`, que faz o seguinte:

1. Ajusta o cabeçalho da página para funcionar como app instalável: `manifest.webmanifest`, ícones e cor do tema.
2. Inclui a biblioteca do Supabase.
3. Inclui a tela de login (entrar ou criar conta com e-mail e senha) e o botão de sair.
4. Remove o perfil inicial do dono do app, para que dados pessoais não fiquem no site público.
5. Troca a seção 25 (conexão com o armazenamento) por uma que lê e grava na tabela `hexis_docs` e liga o armazenamento de fotos.
6. Confere que não sobrou nenhuma referência à plataforma de origem e interrompe a geração se sobrar.

Publicar: gerar o `index.html`, copiar para o repositório, fazer o commit e enviar para o ramo `main`. O GitHub Pages atualiza o site em cerca de um minuto.

## 9. Testes

Rodam antes de cada publicação, na área de desenvolvimento. Cada um abre o app em um navegador simulado, executa as ações e confere o resultado.

| Teste | O que confere |
| --- | --- |
| `run10` | Botões, estados de selecionado e marcação de feito |
| `run11` | Perfil e Objetivo, para usuário novo e para o dono |
| `run12` | Versão web: login e gravação |
| `run13` | Repetições parciais |
| `run14` | Leitura das observações em texto |
| `run15` | Descanso entre séries, temas de observação e efeito no motor |
| `run16` | Intervalo entre exercícios e ordem feita |
| `run17` | Conversão do tempo de execução para minutos |
| `run18` | Cronômetro completo e modo unilateral |
| `run19` | Resumo do dia |
| `run20` | Página inicial, painel Hoje com a pesagem e navegação |
| `run21` | Feito durante o descanso, mensagens de erro e parâmetros |
| `run22` | Parcial com porcentagem livre, descanso previsto e tela acesa |
| `run23` | Blocos do ano: semanas, ajuste de carga na troca de faixa, séries dos prioritários e semanas leves |
| `run25` | Gráficos de execução, parciais, lados e descanso por exercício |
| `run26` | Hormônios e peptídeos: substâncias, aplicações, curva, exames e apagar |
| `run27` | Aquecimento: marcar séries, cronômetro e tempo do treino |
| `run28` | Aquecimento: pausas indicadas, descanso antes da 1ª série (anotar, encerrar, corrigir) e tempo do treino |
| `run29` | Ajuste de carga entre as séries: subir, baixar, manter e preenchimento ao marcar Feito |
| `run30` | Referência de hormônios e peptídeos, cópia para o cadastro, controle de doses e alerta de hematócrito |
| `run31` | Modo unilateral: troca de lado cronometrada e descanso depois dos dois lados |
| `run32` | Anotações livres e posologia preenchida pelo usuário na aba de hormônios e peptídeos |
| `run33` | Conversa entre treinos: carga trazida do exercício irmão e ajuste pelo desempenho no outro dia |
| `run34` | Cargas da máquina: placas guardadas, reconhecimento de placa fixa, placas deduzidas, declaração manual e carga livre |
| `run36` (Dieta) | Também cobre a inserção rápida: refeição da hora, busca, escolha, quantidade pronta e alimento fora da tabela |
| `run38` | Hormônios: fontes num menu recolhido, seletor e edição no cadastro, selo de tipo; leitura do texto de um laudo; importação do arquivo do InBody conferida com o arquivo real |
| `run37` | Exames de bioimpedância: painel Evolução corporal com 11 exames reais, faixas do laudo e estimadas, formulário (salvar, editar, só o mês, excluir) e espelho nas medidas do dia |
| `run36` | Opcionais e exercícios novos fora das pendências; domingo como dia livre; aba Dieta: busca na TACO, registro, alimento fora da tabela, comparação por refeição e por dia |
| `run35` | Tempo no ritmo real (semana 1 de verdade): todos os dias do ano cabem no limite; abdômen abre o treino; panturrilha no leg press; opcionais fora do previsto |
| `tbudget` | Duração prevista de cada dia de treino |
| `tbudget2` | Maior duração prevista de cada dia ao longo das 60 semanas |
| `cmp.sh` | Compara a saída de todos os testes antes e depois de uma reorganização do código |
| `inv.py` | Lista funções, textos e estilos sem uso |

Na revisão de 18/nov os testes rodaram em Chromium a 390 px, com o Supabase simulado e as bibliotecas servidas das cópias oficiais do npm:

| Teste | O que confere |
| --- | --- |
| Injeção de HTML | Textos maliciosos em todos os campos de texto livre (perfil, observações, palavras-chave, substâncias, aplicações, exames, alimentos, refeições, bioimpedância, laudo lido) e em todas as abas: nada pode virar HTML nem rodar |
| Política de conteúdo | Nenhum bloqueio da CSP com o app em uso (abas, gráficos, leitor de PDF, exportação) |
| Leitor de laudo | PDF gerado com hematócrito e testosterona: o leitor baixa, confere a impressão digital e acha os dois |
| Recorde, gasto real e exportação | Recorde no Resumo, gasto com 21 dias de dieta e 7 pesagens, JSON com todos os documentos e CSV das séries |
| Tempo do dia | Todas as semanas de 1 a 60: o maior dia prevê 99 min com o cardio (limite 120) |
| Largura | `largos()` vazio em todas as abas, com um e-mail comprido no cabeçalho |

### Pasta `testes/` (no pacote desde 18/nov)

Os roteiros da revisão de 18/nov e os das duas bancas estão no pacote, na pasta `testes/`, com `README.md`, `preparar.sh` (baixa as bibliotecas do npm uma vez) e `rodar-tudo.sh`. Os roteiros antigos da tabela de cima (`run10`…`run38`, `tbudget`, `cmp.sh`, `inv.py`) **não** vieram no pacote: ficaram na área de desenvolvimento original. Os do pacote rodam com Node e Playwright, contra qualquer `index.html`, sem tocar no banco real, e cada um termina com código 1 quando algo falha (o `rodar-tudo.sh` também).

| Arquivo | O que confere |
| --- | --- |
| `banca2.js` | 34 conferências dos achados da 2ª banca: Sair sem rede (sai, apaga sessão e fila, avisa o que se perderia); série guardada no aparelho sem sinal e enviada ao reabrir; sessão recusada mantém a gravação e mostra "Não salvo"; link com sessão no endereço é ignorado; números estritos nas séries; desmarcar Feito; faixas do Perfil e datas absurdas; dieta ("1.000" g, toque duplo, alimento próprio); aplicação sem duplicar com rede falhando; plaquetas na mesma escala; tentativas com espera crescente; foto com ".."; CSP com hash, quadro de outro site e cadastro neutro |
| `atualiza-csp.py` | Recalcula o hash do programa na CSP (rode depois de qualquer mudança; com `--confere` só verifica) |
| `banca.js` | Conferências dos achados da banca: CSV malicioso ou com data e números impossíveis; tela travada durante a carga e sem gravar quando a carga falha; aviso e nova tentativa na dieta e nos ajustes; duas inclusões seguidas na dieta com rede lenta; "72,5" lido como 72,5; repetições fora da faixa recusadas; toque duplo em Feito; laudo com plaquetas e sinal "<"; "250.000" gravado como 250000; busca da TACO; descanso máximo de 2 min; bi-set só bíceps+tríceps na polia |
| `xss.js`, `xss2.js` | Injeção de HTML em todos os campos e no CSV |
| `func.js` | Exportação, leitor de laudo, dieta, Sair e CSP |
| `tempo.js` | Duração prevista de todos os dias das 60 semanas |
| `largos.js` | Largura de todas as abas a 390 px |

## 10. Limites conhecidos

- **Ainda dependem do painel do Supabase** (o código não consegue confirmar): as regras RLS de `hexis_docs` e do bucket `fotos` (consulta na seção 11), o cadastro aberto, a senha mínima de 8 no servidor e a proteção contra senhas vazadas.
- **Teste no celular real ainda falta**: vírgula nos números (`numCampos`), gravação ao travar a tela (`keepalive`) e Sair sem sinal foram conferidos só no Chromium de teste.
- `numCampos` troca o tipo dos campos de número depois de desenhados, com um observador na página inteira. Funciona, mas o certo seria desenhar os campos já como texto (dívida técnica, junto com as funções grandes e os nomes curtos).
- Os estilos ainda aceitam `'unsafe-inline'` na CSP. Não permite rodar programa, mas um texto que escapasse ainda poderia mudar a aparência.
- O leitor de laudo entende primeiro os laudos em blocos (título do exame, "Método" e uma linha "RESULTADO:", como os do Sabin) e depois procura no texto corrido o que faltou. A referência escolhida é a do sexo do Perfil; quando o laudo só traz faixas por idade sem linha de adulto, fica em branco. Conferido com dois laudos reais.
- Nos formulários de hormônios, o intervalo é uma escala fixa, o local tem lista, e a dose, o lote, a unidade e a referência do exame são guiados pelo último registro da própria pessoa.
- A **leitura de laudo em PDF** usa a biblioteca pdf.js, baixada na hora (versão fixa). Ela só acha os marcadores de `DJ_LER` e nada é salvo sem conferência. A **importação do InBody** reconhece as colunas pelo título (`BIO_CSV`).
- O painel **Evolução corporal** (aba Corpo) sai dos exames de bioimpedância: o pico é o exame de maior gordura; as faixas de peso, massa magra e massa gorda vêm do laudo ou são estimadas pela altura (peso ideal = IMC 22 no homem e 21,5 na mulher; faixa de 85% a 115%).
- A **tabela TACO** (4ª edição, NEPA/UNICAMP, 597 alimentos) fica embutida no programa, conferida linha a linha contra a planilha original. Seis linhas não têm composição na tabela e ficam fora da busca. A meta por refeição é a meta do dia vezes a parte de cada refeição (`settings.dieta.refs`).
- O **domingo** (`DOM`, `livre: true`) não tem exercícios: só registra atividade, entra no cardio da semana e não conta como falta.
- Um exercício só conta como pendente a partir da semana em que entrou na ficha (`desde`); opcional (`opt`) nunca conta.
- O **tempo previsto** usa o ritmo medido (`ritmo`): média da execução por série, segundos de descanso além do previsto e troca de exercício, a partir de 8 tempos de cada tipo. Antes disso vale o padrão. Exercícios opcionais (`opt`) ficam fora do previsto; exercício na mesma máquina do anterior (`junto`) conta 30 s de troca.
- As **placas deduzidas** de uma máquina partem do passo entre as cargas já usadas; com só duas cargas o passo pode estar errado. O campo "Placas desta máquina" corrige.
- O app calcula **60 semanas**, até o fim de 2027 (`MAXW`). Os blocos seguem a lista `BLOCOS_ANO`; a troca de variações de exercício entre blocos ainda é manual.
- As fotos só existem no site, porque dependem do armazenamento de arquivos do Supabase.
- O aviso do cronômetro só toca com o app aberto na tela. Por padrão o app mantém a tela acesa enquanto o cronômetro roda; aviso com a tela bloqueada exigiria notificações enviadas por um servidor.
- O site não tem recuperação de senha por e-mail.
- O **gasto real estimado** depende de registrar o dia inteiro: dias com alimentos faltando puxam o número para baixo.
- O **recorde** compara exercícios pelo nome. O abdominal de quinta (D6) ainda tem a carga da polia na semana 1, então o abdominal na máquina só marca recorde quando passar dela (ver as pendências no contexto).
- No plano gratuito do Supabase, um projeto sem uso por uma semana é pausado e precisa ser reativado no painel.
- A versão de origem e o site têm dados separados.


## Desenho (Liquid Glass) e tela

- **Camadas**: fundo em degradê fixo (`body::before`); conteúdo em vidro claro (`--surface`, borda de luz `--edge`); navegação em vidro com desfoque (`--glass`, `--blur`): cabeçalho, subabas, barra de abas flutuante, barra do cronômetro e janelas.
- **Tela travada**: `travaTela()` (seção 27) fixa o viewport sem zoom, bloqueia a pinça no iPhone e desfaz o zoom ao sair de um campo; todos os campos têm 16 px; `largos()` lista o que passar da largura (conferência).
- **Quadros fixos das séries**: `medeQuadros()` mede a carga e as repetições mais longas já anotadas e fixa as colunas (`--kgc`, `--rpc`); a tabela usa `table-layout: fixed`; cada série tem uma linha de altura fixa para situação e ajuste (`.sline`, texto curto `curto` da sugestão; o completo fica no toque); descanso e intervalo em grades fixas (`.rgrid`).
- **Informação em camadas**: blocos recolhíveis `details.mais` lembram se estavam abertos (`data-keep`, guardado no aparelho em `KEEP_KEY`); "Por que" do cartão, observações e ajustes, metas da semana, tempo do treino, opções do dia, grupos de gráficos de Progresso e Corpo.
- **Botão "i"**: só nos termos menos comuns (`INFO_COMUM`, `INFO_ACT_COMUM`, `infoRaro`); todos os termos ficam no Glossário (Plano e regras, `gloHtml`).
- **Dieta**: aba própria; refeição escolhida pelo horário (`dtRefAgora`, `DT_HORA`); frequentes com a última quantidade (`dtUsados`, `dtUltQ`) adicionados em um toque (`dtPoeItem`); quantidades prontas (`DT_GRAMAS`, `DT_PORCOES`); repetir a refeição de ontem; desfazer o último.
- **Peso base da máquina**: `settings.maq[nome].min`; `maqMin` impede carga prevista, back-off, ajuste entre séries e aquecimento abaixo dele.

## 11. Segurança

O que protege os dados de saúde de cada pessoa, e onde fica cada proteção:

| Proteção | Onde | O que faz |
| --- | --- | --- |
| Regras de acesso (RLS) | Painel do Supabase: tabela `hexis_docs` e armazenamento `fotos` | Cada pessoa só lê e grava as linhas com o próprio `user_id` e só mexe na própria pasta de fotos. A chave no código é a pública: sem as regras, qualquer pessoa leria tudo. **Conferir no painel** (consulta abaixo) |
| Escape do texto | Código, `esc()` (seção 10) | Todo texto digitado ou importado passa por `esc()` antes de virar HTML. A revisão de 18/nov achou e corrigiu um ponto sem escape: o nome do aparelho no painel Evolução corporal (`bioHtml`), que também vem do CSV do InBody |
| Integridade das bibliotecas | `<script ... integrity>` (Chart.js e Supabase) e `DJ_PDF_SHA` (leitor de PDF) | O navegador só roda o arquivo se ele for idêntico ao conferido. O Supabase passou a usar `supabase.js` (o `supabase.min.js` não existe no pacote e era gerado na hora pelo CDN) |
| Política de conteúdo (CSP) | `<meta http-equiv="Content-Security-Policy">` no `<head>` | Programas: só o próprio programa embutido, conferido pelo **hash** (`sha256`, sem `'unsafe-inline'`), e as bibliotecas das **versões exatas** do CDN (Chart.js 4.4.1 e supabase-js 2.119.0). Conexões: só o Supabase do app e a pasta do leitor de PDF (pdfjs-dist 4.10.38). Um texto malicioso que escape para a tela não roda. **Depois de qualquer mudança no programa, rode `python3 testes/atualiza-csp.py partes` e junte as partes de novo; sem isso o app não abre** (o navegador recusa o programa com hash diferente). Os estilos ainda aceitam `'unsafe-inline'` (o app usa `style=` em muitos pontos) |
| Sem referência ao endereço de origem | `<meta name="referrer" content="no-referrer">` | Os pedidos às bibliotecas e às fontes não levam o endereço do app |
| Aparelho compartilhado | Botão Sair (`#logout`) | Apaga o que o app guardou no aparelho antes de sair. A sessão encerrada em outra aba recarrega esta |
| Dentro de quadro de outro site | Seção 27 (partida) | O app não abre dentro de um `iframe` (a regra `frame-ancestors` não vale em `<meta>`) |
| Login por link | `connect()` | `detectSessionInUrl: false`: um endereço com a sessão de outra pessoa não troca a conta logada |
| Fila no aparelho | `makeDb()`, `FILA_KEY` | Só o que ainda não chegou ao servidor fica no aparelho, e sai assim que o servidor confirma ou ao tocar em Sair. É o preço de não perder série anotada sem sinal |
| Isolamento entre contas no código | `makeDb()` e `startSession()` | Além do RLS, a leitura filtra pelo `user_id` da sessão e as fotos só são pedidas ou apagadas se o caminho for exatamente `<pasta da pessoa>/<arquivo>`, sem subpastas nem `..` |
| Nada gravado antes da carga | `makeDb().pronto()` e `body.carregando` | Evita sobrescrever o servidor com dados vazios quando a carga é lenta ou falha |
| Arquivo importado | `bioCsv()`, `BIO_CSV_MAX` | Limite de tamanho, datas e números conferidos, nome do aparelho limpo e escapado |
| Faixas dos números | `djAct`, eventos de digitação, Corpo, dieta | Carga 0–1000 kg, repetições 0–100, RIR 0–10, peso 20–300 kg, gordura 2–70 %, medidas 10–250 cm, alimento até 5.000 g ou 50 porções, dose abaixo de 1 milhão. Fora da faixa, o campo fica vermelho e nada é gravado |
| Senha de conta nova | `SENHA_MIN` | Pede 8 caracteres ao criar conta. O mínimo de verdade é o do painel (Authentication → Policies) |
| Dados pessoais | Regra fixa | Perfil, laudos, cargas e peso base ficam só na conta; o código não tem nome, documento nem laudo. **Exceção ainda aberta**: a tabela `YEAR` (seção 6) traz a trajetória de peso, calorias e eventos mês a mês do plano do dono, e `PLAN_START` tem a data de início. Tirar isso depende de levar esses dados para a conta (ver pendências no contexto) |

Consulta para conferir as regras no painel do Supabase (SQL Editor, só leitura):

```sql
select schemaname, tablename, policyname, cmd, roles, qual, with_check
from pg_policies where tablename = 'hexis_docs' or schemaname = 'storage';
select relname, relrowsecurity from pg_class where relname = 'hexis_docs';
```

O certo é: `relrowsecurity = true`; em `hexis_docs`, as regras de leitura, inclusão, alteração e exclusão comparam `auth.uid() = user_id` (a de inclusão e a de alteração também em `with_check`); no armazenamento, as regras do `fotos` comparam a primeira pasta do caminho com `auth.uid()`.


## Ajustes do celular e versão do artefato (18/nov)

- **Bloco de estilos 22** (fim do `<style>`): palavras só quebram inteiras (`overflow-wrap: break-word`, sem hífen automático); botões nunca quebram letra a letra e, nas tabelas, ficam em uma linha com fonte de 12,5 px; campos `date` e `datetime-local` sem a largura própria do iPhone (`min-width: 0`, `appearance: none`), em colunas `minmax(0, 1fr)`; no celular (até 640 px) a conta sai do cabeçalho (`.acct`) e aparece no topo do Perfil (`#acctBox`, preenchido em `startSession`; o botão `.sair` usa o mesmo fluxo do `#logout`); `#djSubs` e `#djApList` viram cartões.
- **Versão do artefato**: `ferramentas/build_artefato.py <site> <saída>` gera o arquivo do artefato a partir do site. Troca a seção 25 pela conexão com o armazenamento do artefato (mesma interface `doc`, `collection`, `pronto`, `vazia`, `pendentes`, `dump`), tira a CSP, o manifesto, a biblioteca e a tela de login do Supabase e o bloqueio de quadro (o artefato roda dentro de um quadro). Mantém `dataOk`, `corpoRuimAviso` e "Seus dados".
