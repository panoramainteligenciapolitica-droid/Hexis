# Hexis · documentação do código

Atualizada em 12/nov/2026. A tabela de parâmetros e o índice de funções deste documento são tirados dos comentários do próprio código, então cada linha daqui tem um comentário igual no ponto correspondente do `index.html`.

## 1. Onde fica cada coisa

| O quê | Onde fica | Para que serve |
| --- | --- | --- |
| Site | https://panoramainteligenciapolitica-droid.github.io/Hexis/ | O app com login, aberto a qualquer pessoa |
| Repositório | github.com/panoramainteligenciapolitica-droid/Hexis | Arquivos do site: `index.html`, `manifest.webmanifest`, `icons/`, `README.md` e este documento |
| Banco do site | Supabase, projeto `hexis` (região sa-east-1) | Login por e-mail e senha e a tabela `hexis_docs`, com os dados de cada pessoa |
| Versão de origem | App privado, fora do repositório | É onde o app é alterado primeiro. Tem armazenamento próprio, separado do banco do site |
| Gerador do site, testes e cópias de segurança | Área de trabalho de desenvolvimento, fora do repositório | `build_web.py` transforma a versão de origem no `index.html` do site; os testes conferem cada função antes de publicar |

Pontos de atenção:

- A versão de origem e o site **não compartilham dados**. O que é anotado em um não aparece no outro.
- O gerador do site e os testes ainda não estão no repositório. Quem alterar o `index.html` direto no GitHub muda só o site.

## 2. Como o `index.html` é organizado

É um arquivo só, com três partes:

1. **Estilos** (`<style>`): a aparência, em 18 blocos numerados.
2. **Telas** (`<main>` e as barras): a estrutura vazia de cada aba.
3. **Programa** (`<script>`): 26 seções numeradas, que preenchem as telas e guardam os dados.

Para achar um trecho, procure no arquivo pelo número da seção: `===== 18.` leva ao cronômetro no programa; `---- 12.` leva aos estilos do cronômetro. O começo do programa tem um mapa com as 26 seções.

### Estilos (18 blocos)

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
- `v-diario`: Hormônios e peptídeos (referência para estudo, controle de doses, calendário e exames)
- `v-plano`: Plano e regras

Fora das abas ficam: o cabeçalho com o logo, o botão **Hoje** (`#hojeBtn`) e as subabas (`#subnav`), o painel flutuante Hoje (`#hoje`, com o treino do dia, os números principais e a pesagem diária), a barra de navegação de baixo (`#bnav`), a barra de salvar com o cronômetro (`.savebar`, `#clkbar`) e a janela de explicação (`#pop`).

### Navegação

A barra de baixo tem 5 grupos; cada grupo pode ter subabas no topo. Tudo sai da lista `NAVG` (seção 21):

| Grupo | Abas |
| --- | --- |
| Início | Início |
| Treino | Semana · Próxima semana |
| Resumo | Resumo do dia |
| Evolução | Progresso · Corpo e dieta · Hormônios |
| Você | Perfil · Objetivo · Plano e regras |

## 3. Como os dados são guardados

Cada item é um documento separado, com um caminho:

| Caminho | O que guarda |
| --- | --- |
| `semanas/w01-A` | Um dia de treino: `{ semana, dia, nota, tags, cardio, ex }` |
| `corpo/2026-11-10` | Pesagem e medidas de um dia |
| `meta/perfil` | Perfil e Objetivo |
| `meta/ajustes` | Deload e semanas leves escolhidos, palavras-chave criadas e quais exercícios são unilaterais |
| `diario/sub-…`, `diario/ap-…`, `diario/ex-…` | Hormônios e peptídeos: substância, aplicação e exame |
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

São 44 parâmetros, todos na seção 1 do programa. Mudar um valor ali muda todos os lugares que o usam.

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
| `PARC_OPTS` | `[25, 50, 75]` | botões de repetição parcial: % do movimento feito |
| `PARC_CREDITO` | `0.5` | parcial vale metade da fração feita (50% do movimento = 0,25 repetição) |
| `DELOAD_DIAS` | `2` | dias abaixo da faixa na semana 4 que ligam o deload automático da semana 5 |

**Tempo, descanso e recuperação**

| Parâmetro | Valor | O que regula |
| --- | --- | --- |
| `EXEC_DEF` | `40` | segundos de execução por série, quando não anotado |
| `TRANS_DEF` | `90` | segundos de intervalo entre exercícios, quando não anotado |
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
| `WAKE_KEY` | `"hexis.wake"` | preferência "tela acesa durante o cronômetro", guardada no aparelho |
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

Os números do treino em si não estão nesta tabela, e sim na ficha (seção 2): séries, faixa de repetições, carga inicial, descanso previsto e quanto a carga sobe em cada exercício (lista `EX`), e as metas de RIR de cada semana (lista `WT`).

## 6. Índice por seção

São 267 funções. Cada uma tem, no código, um comentário com o mesmo texto da tabela.

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
| `mmss()` | Segundos em minutos, para o selo do campo: 70 → "1:10 min" |
| `fmtSec()` | Segundos em texto curto: 45 → "45 s", 90 → "1:30" |
| `recov()` | Recuperação entre duas séries: força estimada da seguinte ÷ da anterior (com o RIR, se anotado) (normalizada pelo previsto: back-off mais leve ou com mais folga já é esperado) |
| `recovTxt()` | Recuperação em texto: 96% · boa |
| `restFlags()` | Descanso médio do exercício e se foi curto (abaixo de 75% do previsto) |
| `warmDone()` | Quantas séries de aquecimento foram marcadas (null = nenhuma marcação feita) |
| `warmGap()` | Pausa em segundos depois de uma série de aquecimento feita com a fração f da carga (tab = WARM_PAUSA ou WARM_ANTES) |
| `warmTime()` | Tempo do aquecimento de um exercício: serie = execuções e pausas entre elas; antes = descanso até a 1ª válida. marks = séries marcadas (null = todas) |
| `pausaTxt()` | Pausa curta por extenso: 30 s, 1 min, 1:30, 2 min |
| `dayTime()` | Tempo do treino de um dia: previsto (descanso e execução padrão) e registrado |
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
| `medidaAtual()` | Medida atual: registro mais recente do Corpo e dieta que tenha o campo; senão, o perfil |
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
| `popReturn` | Botão que abriu a janela de explicação (o foco volta para ele ao fechar) |

| Função | O que faz |
| --- | --- |
| `infoBtn()` | Botão "i" que abre a explicação de uma chave |
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
| `updateDayTime()` | Atualiza o painel de tempo sem redesenhar a tela |
| `ordemLine()` | O que o app aprendeu sobre trocar a ordem deste exercício |
| `transHtml()` | Intervalo entre exercícios (troca de máquina, ajuste, água): fica no exercício que começa depois |
| `setCells()` | Células de carga, repetições e RIR de uma série (side "e"/"d": um dos lados, no unilateral) |
| `execField()` | Campo do tempo de execução, com o selo que converte para minutos |
| `exCard()` | Cartão do exercício: prescrição, motivo, aquecimento, séries, observações e Feito / Não fiz |

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

| Função | O que faz |
| --- | --- |
| `medIdeal()` | Meta ou referência de uma medida, para a linha do gráfico |
| `renderGoal()` | Desenha a aba Corpo e dieta: fase, calorias e macros, peso, ritmo, gordura e medidas |
| `initBodyForm()` | Monta os campos de medidas do formulário de corpo |
| `weightStats()` | Pesagens: média de 7 dias, média anterior e ritmo por semana |
| `dm()` | "2026-11-10" → "10/nov" |
| `updateQW()` | Atualiza a pesagem rápida (média, ritmo e peso do plano hoje) |
| `saveQW()` | Salva a pesagem rápida |

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
| `renderPlan()` | Desenha a aba Plano e regras: legenda, destraves, semanas do bloco e plano anual |

### 17. Exercício unilateral

Modo unilateral: dados de cada lado e qual lado conta para a progressão.

| Função | O que faz |
| --- | --- |
| `uniDefault()` | Unilateral por padrão? (pelo nome, ou pela observação "cada perna / cada lado") |
| `isUni()` | Unilateral: a escolha do usuário vale mais que o padrão |
| `uniSum()` | Soma de um campo dos dois lados (ex.: tempo de execução) |
| `uniKg()` | Carga de um lado; se estiver vazia, a do outro |
| `uniScore()` | Força estimada de um lado na série |
| `uniAgg()` | A série vale pelo lado mais fraco (empate: o que você marcou como mais difícil) |
| `uniToggleHtml()` | Botões Bilateral / Unilateral do cartão |
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
| `DJ_GRUPOS` | Grupos da referência: chave, título e a situação comum do grupo |
| `DJ_REF` | Referência para estudo. n nome · g grupo · un unidade usual · via · mv meia-vida em dias, só quando a bula traz um valor único (senão null) · mvT meia-vida por extenso · bula intervalo na indicação aprovada · nota · fonte |

| Função | O que faz |
| --- | --- |
| `djDias()` | Tempo por extenso a partir de dias: 0,43 → "10 h"; 40 → "40 dias" |
| `djRefHtml()` | Painel da referência: um bloco recolhível por grupo, com um cartão por substância, mais as notas de leitura, exames e regras |
| `djCtrlHtml()` | Controle de doses: por substância, a última aplicação, os totais de 7 e 30 dias e o estimado hoje pela meia-vida |
| `djList()` | Itens do diário de um tipo ("sub" substância, "ap" aplicação, "ex" exame), em ordem de data (ou de nome) |
| `djLevel()` | Quantidade estimada de uma substância no instante t: soma das doses, cada uma caindo pela metade a cada meia-vida informada. Modelo didático, não é nível no sangue. |
| `djNext()` | Próxima aplicação prevista: a última mais o intervalo informado (dias) |
| `djCalHtml()` | Calendário do mês: ponto cheio = aplicação feita; ponto vazado = próxima prevista |
| `djDelBtn()` | Botão de apagar com confirmação no segundo toque |
| `renderDiario()` | Desenha a aba Hormônios e peptídeos: referência, substâncias, calendário, aplicações, controle de doses, curva estimada e exames |
| `djAct()` | Ações do diário: trocar o mês do calendário, copiar da referência para o cadastro, apagar (com confirmação) e salvar substância, aplicação ou exame |

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
| `setSaveMsg()` | Mensagem da barra de salvar |
| `tapSel()` | Seletor do botão tocado, para reencontrá-lo depois de redesenhar |
| `playTap()` | Efeito de toque no botão |
| `flashBtn()` | Mostra "✓ Salvo" no botão por um instante |
| `parcEdit()` | Parciais de uma série: abre o campo, inclui (pv = % do movimento) ou remove (pv = posição); ps = lado, no unilateral |
| `saveSettings()` | Salva os ajustes (deload escolhido, palavras-chave e exercícios unilaterais) |

### 25. Conexão com o armazenamento

Login e ligação com o banco de dados: carrega os dados da pessoa e grava cada mudança.

| Função | O que faz |
| --- | --- |
| `connect()` | Dono: dados na raiz (como antes). Outras pessoas: espaço privado data/users/<id>/app/, que nem o dono lê. |

### 26. Partida

Ordem em que o app liga.

## 7. Como fazer as mudanças mais comuns

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
| `rest` | `"2 min"` | Descanso previsto: `"90 s"`, `"2 min"`, `"2–3 min"` |
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
| `tbudget` | Duração prevista de cada dia de treino |
| `tbudget2` | Maior duração prevista de cada dia ao longo das 60 semanas |
| `cmp.sh` | Compara a saída de todos os testes antes e depois de uma reorganização do código |
| `inv.py` | Lista funções, textos e estilos sem uso |

## 10. Limites conhecidos

- O app calcula **60 semanas**, até o fim de 2027 (`MAXW`). Os blocos seguem a lista `BLOCOS_ANO`; a troca de variações de exercício entre blocos ainda é manual.
- As fotos só existem no site, porque dependem do armazenamento de arquivos do Supabase.
- O aviso do cronômetro só toca com o app aberto na tela. Por padrão o app mantém a tela acesa enquanto o cronômetro roda; aviso com a tela bloqueada exigiria notificações enviadas por um servidor.
- O site não tem recuperação de senha por e-mail.
- No plano gratuito do Supabase, um projeto sem uso por uma semana é pausado e precisa ser reativado no painel.
- A versão de origem e o site têm dados separados.
