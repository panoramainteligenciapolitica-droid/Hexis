# Héxis · mapa do código
Héxis 2.6 · completo/index.html SHA-256 a75dada9ed5c37772a4554c2dd5b973668943407e793201d14c735c0bc9ac713

Procure antes de abrir: `grep -w fichaTeto mapa/funcoes.tsv` dá parte, linha, seção, descrição e onde é chamada.
Depois leia só o trecho: `sed -n '324,332p' partes/p09e-*.txt` (use `linha_na_parte`).
- `mapa/<parte>.md`: seções e funções da parte, com testes que citam cada uma.
- `mapa/dados.tsv`: caminhos do banco (`meta/ficha`, `dieta/<id>`…) e as funções que os usam.
- `mapa/ids.tsv`: ids de HTML criados no código. `mapa/testes.tsv`: o que cada roteiro confere. `mapa/bugs.tsv`: bugs por função.
Refaça com `python3 ferramentas/mapa.py` depois de juntar as partes; `node testes/mapa.js ../completo/index.html` acusa mapa velho.

| Parte | Linhas no completo | Funções | Conteúdo |
|---|---|---|---|
| `p01` | C:1–772 | 0 | estilos |
| `p02` | C:773–1914 | 0 | estilos liquid glass telas e sec01 parametros |
| `p03` | C:1915–2820 | 68 | secoes 02 a 06 ficha motor observacoes tempo plano |
| `p04` | C:2821–3476 | 40 | secoes 07 a 08 perfil e objetivo |
| `p05` | C:3477–3836 | 26 | secoes 09 a 10 explicacoes e estado |
| `p06` | C:3837–4411 | 37 | secoes 11 a 13 semana proxima progresso |
| `p07` | C:4412–5087 | 60 | secoes 14 a 17 corpo cardio plano unilateral |
| `p08` | C:5088–5960 | 78 | secoes 18 a 22 cronometro resumo inicio navegacao fotos |
| `p09` | C:5961–6843 | 42 | secoes 23 a 24 hormonios e edicao |
| `p09b` | C:6844–7425 | 41 | secao 28 profissional |
| `p09c` | C:7426–7487 | 5 | secao 29 niveis de uso |
| `p09d` | C:7488–7531 | 4 | secao 30 fale com o dev |
| `p09e` | C:7532–7982 | 36 | secao 31 ficha por conta e gerador |
| `p09f` | C:7983–8300 | 29 | secao 32 treinador nutricionista endocrino |
| `p10` | C:8301–9434 | 70 | secoes 25 a 27 conexao dieta partida |

Por assunto (arquivo de `mapa/` onde procurar):
- cronômetro (série, descanso, par): `mapa/p08.md` (30 funções)
- ficha de treino, gerador e tempo: `mapa/p09e.md`, `mapa/p09f.md` (40 funções)
- bi-set e cartão do exercício: `mapa/p03.md`, `mapa/p06.md` (6 funções)
- dieta: `mapa/p05.md`, `mapa/p10.md` (44 funções)
- login e fila de gravação: `mapa/p08.md`, `mapa/p10.md` (13 funções)
- profissional, vínculo e orientações: `mapa/p09b.md` (40 funções)
- treinador, nutricionista, endócrino: `mapa/p09f.md` (13 funções)
- níveis de uso: `mapa/p09c.md` (5 funções)
- perfil, objetivo e cadastro: `mapa/p04.md` (10 funções)
- corpo e bioimpedância (laudo InBody): `mapa/p07.md` (19 funções)
- hormônios, diário e laudo em PDF: `mapa/p09.md` (29 funções)
- fotos: `mapa/p08.md` (9 funções)
- semana, progressão e plano: `mapa/p03.md`, `mapa/p04.md`, `mapa/p05.md`, `mapa/p06.md`, `mapa/p07.md` (8 funções)
- Fale com o dev: `mapa/p09d.md` (4 funções)
- estilos e telas (CSS), versão e parâmetros (`APP_VERSAO`, seção 1): `mapa/p01.md`, `mapa/p02.md` (só seções)
- build do artefato (fora das partes): `ferramentas/build_artefato.py`; testes `testes/bug-2.js`, `testes/bug-14.js`
