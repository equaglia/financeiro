---
prd_number: "005"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: alta
created: 2026-11-03
issue: ""
depends_on: ["001", "002", "003"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/formatos/minhas-economias-exportacao.md
  - docs/categorias-iniciais.md
  - docs/prds/002-contas-e-cartoes.md
  - docs/prds/003-lancamentos-e-categorias.md
---

# PRD 005: Migração do histórico do Minhas Economias

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: a família tem o histórico de lançamentos no Minhas Economias **desde abril de 2017**, exportado em arquivos `.xls` por período (ex.: `Extrato_20260101_20260331.xls`), todos com as mesmas 5 colunas: data, descrição, valor, categoria e conta (`docs/formatos/minhas-economias-exportacao.md`).
- **Problema**: sem o histórico, os relatórios de evolução e as comparações começam do zero, e quase 9 anos de registros ficam presos num app abandonado. A exportação, porém, é pobre: não traz grupos de categoria, faturas, cartão adicional, parcelas, moeda nem tipo de conta. Ela também mistura com as da família as contas de uma PJ (fora da 1ª versão) e o dinheiro que o casal administra para parentes. A migração precisa traduzir tudo isso para o modelo do app sem distorcer os números.

> Contas, donos e contas administradas: PRD 002. Lançamentos, transferências e categorias: PRD 003. Contexto técnico: `docs/trd.md`.

## 2. Solução Proposta

### Visão de produto

- Um **assistente de migração**, usado por um administrador no computador, que lê os arquivos `.xls` de todos os anos de uma vez.
- O assistente pede três mapeamentos: **contas** do Minhas Economias → contas do app; **categorias** → grupo e categoria (proposta pronta em `docs/categorias-iniciais.md`); **casos especiais** (PJ, espólio, carteiras por pessoa).
- Antes de gravar, uma **prévia** mostra totais por ano e por conta, comparáveis com o Minhas Economias. Só depois de conferida a migração é confirmada, e pode ser **desfeita** como um todo.
- O histórico entra com autor "migração" (PRD 001) e passa a valer nos relatórios como se tivesse sido lançado no app.

### Decisões de produto

1. Migrar o histórico completo, desde abril de 2017. Motivo: decisão do casal ("temos histórico para trazer").
2. Contas da PJ não migram, mas o que entrou da PJ nas contas da família vira receita. Motivo: PJ fora da 1ª versão (decisão do casal, 2026-11-03); os lucros e dividendos são renda da família.
3. O dinheiro de terceiros (espólio, conta do parente) migra para **contas administradas**, fora dos números da família. Motivo: decisão do casal, 2026-11-03 ("aparece como nossa e isso não é bom").
4. Migração em lote, com prévia e desfazer. Motivo: são milhares de linhas; errar um mapeamento não pode exigir apagar lançamento por lançamento.
5. A migração vai até uma **data de corte**; depois dela, os lançamentos são feitos só no app. Motivo: evitar o mesmo gasto vindo do Minhas Economias e do app.

### Fora do escopo

- Migrar de outros apps ou da planilha atual.
- Recuperar os dados de IR (CPF/CNPJ, atendido) das despesas antigas: os anos passados já foram declarados.
- Descobrir no histórico qual cartão adicional fez cada compra: a exportação não traz essa informação.
- Migrar as contas e os lançamentos da PJ.
- Sincronização contínua com o Minhas Economias: a migração é feita uma vez (pode ser refeita, US07).

## 3. Funcionalidades

### US01: Carregar os arquivos de exportação

Como administrador, quero carregar de uma vez todos os arquivos exportados do Minhas Economias, para migrar o histórico inteiro.

**Rules:**
- Aceita vários arquivos `.xls` no formato descrito em `docs/formatos/minhas-economias-exportacao.md`.
- O assistente mostra o resumo: período coberto, total de linhas, linhas por ano, contas e categorias encontradas.
- Arquivos com **períodos sobrepostos** → as linhas do período repetido são consideradas uma vez só: vale o arquivo mais recente.
- Buracos no período (meses sem nenhum arquivo) são apontados.
- A migração é feita no computador (os arquivos estão no Dropbox local) e só por administrador.

**Edge cases:**
- Arquivo com colunas diferentes ou outra planilha → recusado, com a indicação do que difere.
- Linha com data, valor ou conta vazios → listada como problema; não bloqueia o resto.
- Arquivo `.xlsx` ou `.csv` salvo a partir do `.xls` → aceito se tiver as mesmas colunas.

### US02: Mapear as contas

Como administrador, quero dizer a que conta do app corresponde cada conta do Minhas Economias, para que o histórico caia no lugar certo.

**Rules:**
- Para cada conta encontrada, o administrador escolhe: **ligar a uma conta já cadastrada**, **criar uma conta nova** (tipo, dono e moeda, PRD 002) ou **não migrar** (ex.: PJ).
- O assistente sugere o tipo pelo nome (ex.: "Cartão", "Visa", "Mastercard" → cartão; "Cash" → dinheiro em espécie; "Inv", "Fundos" → investimento).
- Dinheiro em espécie por pessoa ("Cash - <pessoa>") → uma carteira por pessoa, com o dono certo (família ou filho, PRD 002).
- Conta em nome de parente → **conta administrada** (PRD 002, US12).
- Cartão → ligado a um cartão cadastrado (com dia de fechamento e vencimento), necessário para US05.
- Conta ligada a uma conta em dólar → os valores são tratados como dólar, porque a exportação não tem moeda.

**Edge cases:**
- Conta que trocou de nome no Minhas Economias ao longo dos anos (aparece com dois nomes) → os dois nomes podem ser ligados à mesma conta do app.
- Conta de filho no histórico (ex.: carteira do filho) → os lançamentos ficam fora dos relatórios da família, inclusive os antigos (PRD 002, US04).
- Conta não migrada com lançamentos ligados a contas migradas (transferências) → tratado em US04.

### US03: Mapear as categorias

Como administrador, quero que as categorias do Minhas Economias sejam agrupadas e ligadas às do app, para que os relatórios por categoria usem todo o histórico.

**Rules:**
- O assistente parte da proposta em `docs/categorias-iniciais.md`, já ajustada pelo casal, e liga cada categoria do arquivo a uma categoria do app.
- Categoria do arquivo que não está na proposta (ex.: só existe em anos antigos) → vai para o grupo **A classificar**; o administrador pode encaixá-la já no assistente ou depois (PRD 003, US06).
- "Sem Categoria" → **A classificar**.
- "Transferência" não vira categoria: vira transferência (US04).
- Categorias da PJ (Faturamento PJ, ISS, Contabilidade) só aparecem em contas não migradas; se aparecerem numa conta da família, vão para **A classificar**.
- Nomes são corrigidos (espaços duplos, maiúsculas) sem perder a ligação.

**Edge cases:**
- Duas categorias do arquivo que significam a mesma coisa (ex.: com e sem acento) → podem ser ligadas à mesma categoria do app.
- Categoria cujo sinal do valor não bate com o tipo (receita numa categoria de despesa) → mantida como estorno ou devolução da despesa (PRD 003, US09).

### US04: Converter transferências e casos especiais

Como administrador, quero que transferências, dinheiro da PJ e do espólio sejam convertidos corretamente, para que despesas e receitas da família não fiquem infladas.

**Rules:**
- **Pares de transferência**: duas linhas "Transferência" com a mesma data, a mesma descrição e valores opostos viram **uma transferência** entre as duas contas do app.
- **Pagamento de fatura** (transferência para a conta de um cartão) vira pagamento da fatura (PRD 002, US07).
- **Transferência com uma conta não migrada (PJ)**: só o lado da família entra. Entrada vinda da PJ → **receita** "Lucros e dividendos"; saída para a PJ → o administrador escolhe a categoria (sugestão: A classificar).
- **Espólio**: lançamentos da categoria do espólio numa conta da família viram **transferências** entre essa conta e a conta administrada do espólio (PRD 002, US12).
- **Transferência para conta de filho** (ex.: carteira do filho) → despesa da família, categoria Semanada ou outra escolhida, conforme a regra do PRD 003.
- **Transferência para conta administrada** (ex.: investimento do parente) → continua transferência, fora dos números da família.

**Edge cases:**
- Linha "Transferência" sem par (só um lado) → listada para o administrador escolher: transferência com a outra conta indicada, despesa ou receita.
- Dois pares iguais no mesmo dia (mesma descrição e valor) → unidos dois a dois; o assistente não perde nenhum.
- Transferência entre duas contas não migradas → ignorada.

### US05: Reconstruir as faturas dos cartões

Como administrador, quero que as compras antigas no cartão sejam agrupadas em faturas, para que o histórico siga o regime de caixa como o resto do app.

**Rules:**
- As compras de cada cartão são distribuídas nas faturas pelo **dia de fechamento** cadastrado (PRD 002).
- Os **pagamentos** migrados (US04) marcam o mês de cada fatura; as compras contam no mês do pagamento.
- Para cada fatura reconstruída, o assistente compara o total das compras com o pagamento e mostra a diferença.
- O dia de fechamento pode ter mudado ao longo dos anos: o administrador pode informar **períodos com outro dia de fechamento**.
- Compras do histórico ficam atribuídas ao **titular** do cartão (a exportação não identifica o adicional); os filhos não veem compras anteriores à migração no cartão deles.
- Parcelas: cada parcela já vem como uma linha no Minhas Economias e é migrada como compra própria naquela fatura, sem formar série.

**Edge cases:**
- Diferença entre compras e pagamento numa fatura antiga → aceitável; o administrador pode ignorar diferenças de anos anteriores e corrigir só as recentes.
- Fatura sem pagamento encontrado → as compras ficam no mês do vencimento previsto, marcadas para revisão.
- Cartão que existiu e foi cancelado → criado como cartão encerrado (PRD 002, US09).

### US06: Definir os saldos iniciais

Como administrador, quero que o saldo de cada conta migrada bata com o banco hoje, para que o histórico e o presente fiquem coerentes.

**Rules:**
- A exportação não traz saldos. Para cada conta migrada, o administrador informa o **saldo atual** (do extrato) numa data; o app calcula o saldo inicial em abril de 2017 (ou na primeira data da conta) subtraindo o histórico.
- Alternativa por conta: informar o saldo numa data antiga conhecida.
- A conta fica conferida na data informada (PRD 002, US08).

**Edge cases:**
- Saldo inicial calculado improvável (ex.: conta corrente com R$ −50 mil em 2017) → indica lançamentos faltando no Minhas Economias; o assistente avisa, e o administrador ajusta ou cria um ajuste de saldo.
- Conta encerrada há anos → saldo final zero na data do encerramento.

### US07: Prever, conferir, confirmar e desfazer

Como administrador, quero ver o resultado antes de gravar e poder desfazer, para não estragar os dados do app com um mapeamento errado.

**Rules:**
- A **prévia** mostra, por ano e por conta: número de lançamentos, total de despesas, total de receitas, transferências e o que não foi migrado (PJ). Para conferir com o Minhas Economias, mostra também a soma por conta no mesmo critério do arquivo.
- Pendências listadas: transferências sem par, linhas com problema, categorias em A classificar, faturas com diferença.
- A migração só é confirmada com as pendências **resolvidas ou aceitas**.
- Confirmada, a migração forma um **lote**; desfazer o lote apaga tudo o que ele criou (lançamentos, e contas e categorias criadas por ele, se não tiverem sido usadas depois).
- Os mapeamentos ficam guardados: refazer a migração (ex.: com um arquivo corrigido) reaproveita as escolhas.

**Edge cases:**
- Lançamentos feitos no app em cima de lançamentos migrados (ex.: categoria corrigida) e depois o lote é desfeito → o app avisa quantos lançamentos alterados serão perdidos.
- Migração interrompida no meio → nada é gravado; o lote só existe quando confirmado.

### US08: Data de corte e não duplicar

Como administrador, quero definir até quando vale o Minhas Economias, para que nada seja contado duas vezes.

**Rules:**
- O administrador escolhe a **data de corte**: lançamentos até ela vêm da migração; depois dela, só do app (digitados ou importados).
- Sugestão: o último dia do último arquivo exportado.
- Lançamentos do arquivo depois da data de corte são ignorados e listados.
- Lançamentos feitos no app antes da data de corte → o assistente aponta possíveis duplicados (mesma conta, data e valor) para o administrador decidir.

**Edge cases:**
- O casal começa a usar o app antes de exportar o último trimestre do Minhas Economias → a data de corte é a do dia em que o app passou a ser usado; o período entre o último arquivo e essa data fica como buraco apontado (US01).
- Importação de extrato do banco (PRD 004) cobrindo período anterior à data de corte → os lançamentos migrados são tratados como já conciliáveis (sem duplicar).

## 4. Fluxo de Negócio

```
Arquivos .xls (2017 → hoje)
   │
   ▼
Resumo: período, linhas, contas, categorias, buracos
   │
   ▼
Mapear contas ──▶ PJ? ─sim─▶ não migra (entradas da PJ viram receita na família)
   │              parente/espólio? ─sim─▶ conta administrada
   │              "Cash - pessoa"? ─sim─▶ carteira da pessoa
   ▼
Mapear categorias (proposta) ──▶ desconhecida ──▶ A classificar
   │
   ▼
Converter transferências (pares) e reconstruir faturas dos cartões
   │
   ▼
Saldos iniciais (a partir do saldo atual)
   │
   ▼
Prévia com totais e pendências ──▶ ok? ─não─▶ ajustar mapeamentos
   │ sim
   ▼
Confirmar lote (pode ser desfeito)
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Todos os arquivos de abril de 2017 até a data de corte são lidos de uma vez | histórico completo | carregar todos: resumo mostra cada ano e nenhum arquivo recusado |
| Soma por conta na prévia igual à soma do arquivo no mesmo critério | confiar nos números migrados | comparar a prévia com a soma da coluna Valor por conta no `.xls` |
| Todos os pares de transferência viram uma transferência | não inflar despesas e receitas | na amostra de 2026, 44 pares viram 44 transferências; nenhuma sobra |
| Entradas vindas da PJ viram receita "Lucros e dividendos"; contas PJ não aparecem no app | PJ fora da 1ª versão | migrar a amostra: nenhuma conta PJ; as entradas da PJ estão como receita |
| Lançamentos do espólio ficam fora das despesas e receitas da família | dinheiro do parente não é da família | após migrar, relatório do mês não inclui o espólio; a conta administrada mostra o saldo dele |
| Compras de cartão do histórico contam no mês do pagamento da fatura | regime de caixa também no histórico | compra de 25/03 com fatura paga em 10/04 aparece em abril |
| Saldo atual de cada conta migrada igual ao do extrato | coerência entre histórico e presente | após informar o saldo atual, conta conferida com diferença zero |
| Desfazer o lote deixa o app como antes da migração | errar o mapeamento não pode ser irreversível | confirmar e desfazer: contagem de lançamentos volta ao valor anterior |
| Prévia de 9 anos de histórico fica pronta em até 1 minuto no computador | o administrador refaz a prévia várias vezes ao ajustar mapeamentos | cronometrar com todos os arquivos |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Anos de histórico disponíveis no app | 0 | desde abril de 2017 | na entrega da migração | últimos 3 anos | casal |
| Lançamentos migrados em "A classificar" | A levantar (depende dos anos antigos) | 0 | 1 mês após a migração | menos de 1 % | casal |
| Contas migradas conferidas com o saldo atual | 0 | todas | na migração | correntes e cartões | casal |

## 6. Milestones

### Milestone 1: Histórico do Minhas Economias no app

**Por que é um marco:** quase 9 anos de registros passam a alimentar os relatórios do app, com o dinheiro da PJ e dos parentes separado do da família.

**Funcionalidades:** US01, US02, US03, US04, US06, US07, US08

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Todos os arquivos de abril de 2017 até a data de corte são lidos de uma vez
- [ ] Soma por conta na prévia igual à soma do arquivo no mesmo critério
- [ ] Todos os pares de transferência viram uma transferência
- [ ] Entradas vindas da PJ viram receita "Lucros e dividendos"; contas PJ não aparecem no app
- [ ] Lançamentos do espólio ficam fora das despesas e receitas da família
- [ ] Saldo atual de cada conta migrada igual ao do extrato
- [ ] Desfazer o lote deixa o app como antes da migração
- [ ] Prévia de 9 anos de histórico fica pronta em até 1 minuto no computador

**Aprovador:** casal

### Milestone 2: Histórico dos cartões em regime de caixa

**Por que é um marco:** a evolução dos gastos ao longo dos anos fica comparável com os meses novos, todos pelo mês do pagamento da fatura.

**Funcionalidades:** US05

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Compras de cartão do histórico contam no mês do pagamento da fatura

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Exportações antigas com problemas (contas renomeadas, categorias extintas, buracos) | Médio | mapeamento por nome, A classificar, aviso de buracos (US01–US03) | Pendente |
| Faturas antigas não batem com os pagamentos (fechamento mudou) | Baixo | períodos de fechamento e aceitar diferenças antigas (US05) | Pendente |
| Saldos iniciais estranhos por lançamentos que faltaram no Minhas Economias | Médio | saldo a partir do saldo atual e ajuste de saldo (US06) | Pendente |
| Dinheiro de terceiros que passou pelas contas da família sem categoria própria (só o espólio tem categoria) | Médio | reclassificar na prévia; lançamentos podem ser corrigidos depois | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 002: contas, cartões, contas administradas | Interna | rascunho (1ª revisão; US12 aguardando) | Milestones 1 e 2 |
| PRD 003: lançamentos, transferências, categorias | Interna | rascunho (aguardando 1ª revisão) | Milestone 1 |
| `docs/categorias-iniciais.md` ajustada pelo casal | Interna | proposta | Milestone 1 (US03) |
| Arquivos de exportação de 2017 a 2025 | Externa (casal) | com o casal | Milestone 1 |

## 8. Referências

- `docs/formatos/minhas-economias-exportacao.md` — formato do arquivo (colunas, transferências em pares, o que não vem).
- `docs/categorias-iniciais.md` — proposta de agrupamento das 71 categorias.
- `docs/prds/002-contas-e-cartoes.md` — tipos de conta, donos, contas administradas (US12), faturas.
- `docs/prds/003-lancamentos-e-categorias.md` — transferências, semanada, categorias.

## 9. Registro de Decisões

- **2026-11-03:** Migrar desde abril de 2017; exportações com as mesmas 5 colunas. Motivo: informação do casal.
- **2026-11-03:** PJ não migra; entradas da PJ nas contas da família viram receita "Lucros e dividendos". Motivo: PJ fora da 1ª versão (decisão do casal).
- **2026-11-03:** Espólio e conta do parente viram contas administradas. Motivo: decisão do casal; hoje aparecem como da família.
- **2026-11-03:** Hierarquia de categorias proposta por Claude e ajustada pelo casal. Motivo: listar no Minhas Economias é trabalhoso (casal).
- **2026-11-03:** PRD 005 escrito antes do 004. Motivo: a exportação do Minhas Economias já estava em mãos; as amostras dos bancos (PRD 004) chegam depois.
- **2026-11-03:** `depends_on: ["001", "002", "003"]`. Critério: autor "migração" e permissão de administrador (001); contas, cartões, faturas e contas administradas (002); lançamentos, transferências e categorias (003). Não depende do 004: a migração não usa a importação de extratos.
- **2026-11-03:** 1ª revisão do casal: as 14 premissas foram aceitas como estão (inclusive: a planilha atual não é migrada; dados de IR das despesas antigas não são recuperados). Marcas removidas; status mantido em `rascunho` até a revisão final.
