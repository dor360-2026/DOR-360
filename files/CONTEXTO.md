# D'OR 360 — Contexto do Projeto para Claude Code

## Visão Geral

App de visita técnica de imóveis para a **Rede D'Or — Diretoria de Engenharia / Projetos Especiais**.  
Responsável: **Maria Giovanardi** — Coordenadora de Projetos Especiais.

O app é um **único arquivo HTML standalone** (`dor360-projetos-especiais.html`), sem backend, sem dependências de servidor. Roda inteiramente no navegador do celular (Edge, Safari ou Chrome). Não pode ser aberto dentro do app do Claude — precisa de um navegador real com acesso a câmera, arquivos e impressão.

---

## Arquivo Principal

```
dor360-projetos-especiais.html   ← app completo (405KB)
```

Tudo está embutido neste único arquivo: CSS, JS, ícones em base64, fontes via CDN (Inter do Google Fonts, Tabler Icons).

---

## Brand Guideline

| Elemento | Valor |
|---|---|
| Azul principal | `#003DA5` |
| Azul claro | `#71C5E8` |
| Azul escuro | `#002855` |
| Fonte do app | Inter (Google Fonts) |
| Fonte do PDF | Inter (via @import no PCSS) |
| Ícone do app | DOR_360.png embutido em base64 (favicon + apple-touch-icon + PWA manifest) |

---

## Estrutura do App — 15 Seções + Resumo

| ID | Seção | Destaques |
|---|---|---|
| s1 | Dados Gerais | Nome, endereço, data, responsável, tipo EAS (CEMED/COLETA/IMUNO/NEFROLOGIA/ONCOLOGIA/SADT/OUTRO), hospital referência |
| s2 | Localização | Tipo de via, tipo de imóvel (IMÓVEL DE RUA/MONOUSUÁRIO/LAJE CORPORATIVA/LOJA EM MALL/SALAS/OUTRO) |
| s3 | Características Físicas | Estado geral, área permeável, recuos, cobertura, piso externo, fachada, obstáculos, convenção de condomínio; pavimentos dinâmicos com nome+pé-direito+área+acabamentos (piso/parede/forro) por pavimento |
| s4 | Acessos e Circulação | Acessos existentes, acessibilidade, rampa de veículos, escadas, elevadores (com aproveitável+detalhes), observações |
| s5 | Áreas de Apoio | Resíduos, DML, copa (SIM→localização), vestiários (SIM→localização), área técnica |
| s6 | Vagas | Tipo, demarcadas, embarque/desembarque, ambulância (SIM→percurso/NÃO→alternativas) |
| s7 | Estrutura | Pilares, vigas, vedações, tipo de estrutura do piso, conservação vigas/pilares, piso elevado (SIM→material+abrangência) |
| s8 | Elétrica | Entrada, medidores, bitola, proteção, tensão, gerador (SIM→potência+aproveitável+tanque diesel), SPDA |
| s9 | Hidráulica | Reservatório (SIM→cap/material+aproveitável), hidrômetro (SIM→número+aproveitável), pluvial, esgoto/gordura |
| s10 | Gases Medicinais | Rede (SIM→tipos+aproveitável), central (SIM→localização+aproveitável) |
| s11 | HVAC e Exaustão | Sistema HVAC (SIM→tipo+aproveitável), exaustão fachada, renovação de ar, sistema de exaustão — cada item com aproveitável individual |
| s12 | Combate a Incêndio | Regular Bombeiros, instalações (SIM→tipos+aproveitável), sinalização, rota de fuga, portas, corrimão (SIM→material+aproveitável), guarda-corpo (SIM→material+aproveitável), largura escada |
| s13 | Aspectos Legais | Tombamento/Patrimônio Histórico, zoneamento, uso de saúde permitido, alvará, habite-se, pendências normativas, restrições condomínio |
| s14 | Análise Técnica | Pontos positivos (bullets via Enter), pontos negativos (bullets via Enter), intervenção estimada (POUCA/MÉDIA/ALTA com cores verde/amarelo/vermelho), justificativa, programa pretendido (bullets via Enter) |
| s15 | Fotos | Dois botões: câmera (capture=environment) e upload (multiple). Compressão automática: max 1280px, JPEG 62%. Sem limite de fotos. |
| s-resumo | Resumo | Exibe todos os dados preenchidos. No final: "Relatório pronto para envio?" SIM→botão Gerar PDF / NÃO→salvar+exportar rascunho |

---

## Lógica de Chips (Seleção Rápida)

Todos os campos de seleção usam chips (botões de toque). Duas classes: `.chip` (maior) e `.mchip` (menor, inline).

**Regras universais:**
- Clicar em qualquer chip do grupo sempre reavalia todos os campos condicionais daquele grupo
- `data-g="grupo"` — identifica o grupo
- `data-show="id-do-cond-field"` — abre esse container ao selecionar
- `data-excl="1"` — exclusivo (SIM/NÃO/N/A); sem isso é multi-seleção
- `data-multi="1"` — permite selecionar múltiplos do mesmo grupo
- `data-other="id-do-input"` — abre campo de texto livre ao selecionar "OUTRO"
- NÃO/N/A ou desmarcar: fecha o cond-field E limpa o conteúdo interno (chips desmarcados, textos apagados)

**Funções JS relevantes:**
```js
updateCondGroup(g)      // esconde/mostra cond-fields do grupo
updateOtherGroup(g)     // esconde/mostra campos "OUTRO" do grupo
clearCondFieldContents(container)  // limpa chips e textos dentro de cond-field fechado
collectG(g)             // salva seleção do grupo em D[g]
```

---

## Persistência de Dados

**Duas camadas:**

1. **localStorage automático** (`dor360_rascunho_v1`): salva ao navegar entre seções e com debounce de 1.2s no input. Ao abrir o app, detecta rascunho salvo e pergunta se quer continuar.

2. **Exportar/importar arquivo .json**: botão "Exportar rascunho" usa `navigator.share()` (mobile) com fallback de download (desktop). Importar via `<input type="file" accept=".json">` na S1.

**Funções JS:**
```js
capturarEstadoCompleto()    // retorna objeto com D, camposTexto, chipsSelecionados, fotos, curSecao
aplicarEstadoCompleto(estado)  // restaura estado completo no DOM
salvarRascunhoAutomatico()  // grava no localStorage
exportarRascunho()          // compartilha/baixa arquivo .json
importarRascunho(file)      // lê arquivo .json e restaura
```

---

## Geração de PDF

**Método:** overlay full-screen + `window.print()` nativo do navegador.

- `gerarPDF()` — valida campos obrigatórios (cada seção s1-s15 precisa ter ao menos 1 campo preenchido), monta o HTML do relatório e injeta em `#pdf-overlay-content`
- O CSS `@media print` corrige o container do overlay: `position:static`, `overflow:visible`, `height:auto` (resolve o bug clássico de só a capa ser impressa quando o container é `position:fixed + overflow:auto`)
- O app de formulário (`.app`, `.nav`) fica oculto durante a impressão
- Botões de "Voltar" e "Salvar PDF" somem no PDF final (ocultos via `@media print`)
- Cores de intervenção: `.abox-pouca` (verde `#1a7c40`), `.abox-media` (amarelo `#b8860b`), `.abox-alta` (vermelho `#c0392b`)

---

## Pavimentos Dinâmicos

Gerados dinamicamente via `rebuildPavSection()`. Cada pavimento tem:
- Nome (input text) — `pav-name-${p}`
- Pé-direito (input number) — `pd-pav${p}`
- Área em m² (input number) — `area-pav${p}`
- Acabamentos por elemento (piso/parede/forro): tipo, condição (RUIM→REGULAR→BOA), aproveitável (SIM/PARCIAL/NÃO)
- Observações (textarea) — `obs-ac-p${p}`

IDs dos chips de acabamento: `ac-p${p}-${tipo}-tipo`, `ac-p${p}-${tipo}-cond`, `ac-p${p}-${tipo}-aprov`

---

## Variáveis JS Globais Principais

```js
const D = {}          // todos os valores do formulário (texto + chips)
const fotos = []      // array de {id, label, dataUrl} — fotos comprimidas
let cur = 0           // índice da seção atual
const SECS = [...]    // array de IDs das seções em ordem
const LBLS = [...]    // labels de exibição de cada seção
const STORAGE_KEY = 'dor360_rascunho_v1'
const BULLET_TEXTAREAS = ['positivos', 'negativos', 'capacidade']  // Enter vira bullet nesses
```

---

## Validação antes de gerar PDF

```js
function validarCamposObrigatorios()
```
Verifica seções s1 a s15. Para cada seção:
- Texto: qualquer `input[type=text/number/date]` ou `textarea` com valor
- Chips: qualquer `.chip.sel` ou `.mchip.sel` dentro da seção
- Fotos (s15): `fotos.length > 0`

Retorna array com nomes das seções faltantes. Se não vazio, mostra `alert()` e bloqueia o PDF.

---

## Auto-resize de Textareas

```js
function autoResizeTextarea(el)   // ajusta height = scrollHeight
function autoResizeAllTextareas() // aplica em todas as textareas do DOM
```
Chamada no evento `input` e ao navegar entre seções. Min-height: 64px via CSS.

---

## Distribuição

- **Opção 1 (atual):** arquivo HTML distribuído por e-mail/mensagem. Guia de instalação: `guia_instalacao_dor360.docx`
- **Opção 2 (pendente):** hospedagem em URL fixa (ex: `dor360.rededor.com.br`). Especificação técnica: `especificacao_hospedagem_dor360.docx`. Conector Cloudflare MCP conectado no Claude.ai mas só leitura — deploy precisa ser feito via painel Cloudflare Pages (interface visual, gratuito) ou Cloudflare Workers via wrangler CLI. Maria ainda não decidiu entre fazer ela mesma ou repassar ao TI.

---

## Dispositivos e Ambiente

- **iPhone corporativo:** só Edge disponível (política/disponibilidade). Funciona bem.
- **iPhone pessoal:** Chrome instalado mas não aparece como opção para abrir .html (limitação do iOS). Edge ou Safari funcionam.
- **Abrir SEMPRE pelo navegador** (Edge/Safari/Chrome) — nunca dentro do app do Claude, que bloqueia câmera, upload e salvamento de PDF.
- Para adicionar à tela inicial: Safari → compartilhar → "Adicionar à Tela de Início" / Edge → menu → "Adicionar à tela inicial"

---

## Pendências / Próximos Passos

- [ ] Teste de campo real em visita técnica (Maria vai testar e reportar ajustes necessários)
- [ ] Decisão de hospedagem: painel Cloudflare Pages (Maria faz sozinha) ou repassar ao TI com a especificação já entregue
- [ ] Possível ajuste fino de campos após uso em campo

---

## Convenções de Código

- Todo texto de usuário é salvo em **maiúsculas** (`el.value.toUpperCase()`)
- Campos condicionais usam classe `.cond-field` com `style="display:none"` inicial
- `D[grupo]` guarda valor de chips; `D[id]` guarda valor de campos de texto
- Funções de coleta: `collectG(g)` para chips, `collectText()` para inputs/textareas
- O PCSS (CSS do PDF) é uma string JS injetada via `<style id="pdf-report-style">` no `<head>` ao gerar o relatório
