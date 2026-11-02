# Brainstorm: controle financeiro familiar

**Lente:** especificação de projeto de desenvolvimento
**Última sessão:** 2026-11-02 (consolidação validada pelo casal; PRDs em andamento)

Documento de trabalho: guarda decisões, premissas, alternativas descartadas e pendências. É a base para os PRDs (`docs/prds/`) e o TRD (`docs/trd.md`), não a especificação final. Uma sessão nova retoma daqui (seção "Estados" e "Pendências").

## Problema

- A família usava o app Minhas Economias, com todos os lançamentos digitados à mão. Na última atualização o app passou a exigir conexão por Open Finance (open banking), e o casal não aceita entregar o acesso às contas a terceiros (risco de perda de privacidade).
- Hoje usam uma planilha simples de receitas e despesas, sem gráficos, relatórios nem análises.
- Decidido: o controle financeiro também deve **ajudar no preenchimento da declaração anual do imposto de renda**.
- **O que se quer:** o controle que o app dava (contas, cartões, investimentos, lançamentos manuais) com análise e relatórios melhores que a planilha, **sem que os dados financeiros passem por serviços de terceiros**.

**Reframing:** o diferencial do sistema não é ter funcionalidades novas, é a privacidade. Toda decisão de arquitetura (onde os dados ficam, como os dois celulares/computadores se sincronizam, se há cotação de mercado externa) é julgada primeiro por esse critério.

## Requisitos de negócio

### Usuários e perfis

| Perfil | Quem | O que faz |
|---|---|---|
| Administrador | o casal (2 pessoas) | lança, importa, categoriza, analisa, administra contas, cartões e investimentos |
| Dependente | 2 filhos | anota e categoriza as próprias compras, acompanha limite ou mesada; vê **todas as compras do próprio cartão adicional** e **a própria conta individual**; não vê nada além do que é dele |

- Todas as despesas são comuns: não há separação "dinheiro dele / dela" nem rateio.
- Contas correntes, contas de investimento e cartões de crédito são conjuntos.
- Os filhos têm cartões adicionais (dependentes) no cartão do casal. Hoje não se interessam pelo app; o casal gostaria que passassem a usar.
- Decidido: hoje cada filho já tem **conta individual própria**, onde recebe a **mesada por Pix**, e o cartão adicional tem **limite mensal**.
- Decidido: **as contas dos filhos entram no app**. No futuro podem ter também **contas correntes e de investimento próprias**.
- Decidido: **o que o filho faz com o dinheiro da conta dele é dele**: a movimentação das contas dos filhos não entra nos relatórios da família.
- Decidido: **o casal vê as contas dos filhos** (saldo e movimentação), separadas dos relatórios da família. Consequência: cada conta tem um dono (o casal ou um filho), e o que cada usuário vê depende disso. Os relatórios da família não incluem as contas dos filhos.

### Capacidades

- Cadastrar e acompanhar **contas correntes**, **cartões de crédito** (com cartões adicionais) e **contas de investimento**.
- Decidido (nível b): **investimentos por aplicação** (CDB, fundo, ações, Tesouro etc.) com aportes, resgates e rendimento. Sem cotação automática: o valor atual é informado à mão ou importado. Descartados: (a) só saldo por conta, pouco para análise; (c) cotação automática, dependência externa.
- **Dependentes:** anotar e categorizar as próprias compras no cartão adicional; acompanhar o consumo do mês contra o limite do cartão adicional; acompanhar a mesada.
  - Decidido (consolidação): o filho pode **registrar a compra no celular na hora**; quando o casal importa a fatura, a compra importada é conciliada com a registrada pelo filho (identificada pelo cartão adicional). Se errado: o filho só categoriza depois da importação.
  - Decidido (consolidação): o limite mensal do cartão adicional é **cadastrado pelo casal** no app; o app mostra quanto já foi usado e avisa perto do limite. O app não bloqueia nada (quem bloqueia é o banco).
  - Decidido: o **Pix da mesada é despesa da família** (não é transferência interna) e receita na conta do filho.
  - Decidido: **as compras no cartão adicional são despesas da família** e entram nos relatórios do casal (o casal paga a fatura).
- **Lançamento manual** de receitas e despesas (como no Minhas Economias).
- **Importação opcional** de extratos e faturas em **CSV e PDF**.
- Decidido: **relatórios e análises**: gastos por categoria no mês; evolução ao longo dos meses; orçamento planejado x realizado; patrimônio total.
- Decidido: **orçamento** por categoria (planejado), comparado com o realizado.
- Decidido: **contas no exterior com controle simplificado**: lançamento a partir do extrato mensal, saldo em **dólar**; no patrimônio total, convertido pelo **câmbio do fechamento do mês**.
- Decidido: **previdência** (BBPrev) entra como tipo de conta, acompanhada pelo saldo e pelas contribuições do extrato, e somada ao patrimônio.
- Decidido: **migração do histórico** a partir de uma **exportação do Minhas Economias** (período a verificar pelo casal; amostra a pedir mais tarde).
- Decidido: **apoio à declaração anual do IR**, com **relatórios para ajudar a preencher** (não gera o arquivo oficial da Receita nem transmite a declaração; confirmado na rodada 6):
  - **Despesas dedutíveis** (saúde, educação, previdência): o lançamento guarda **CPF ou CNPJ do prestador** e **quem foi atendido** (titular, cônjuge ou filho).
  - **Bens e direitos**: saldo de cada conta e aplicação em 31/12 (inclusive exterior).
  - **Declaração conjunta**, com os **filhos como dependentes**.
  - Fora: rendimentos (isentos, exclusivos) e ganhos em renda variável / DARF (não escolhidos na rodada 6).
- Decidido: **categorias com subcategorias**; as importadas do Minhas Economias são aproveitadas e podem ser **editadas**, e o casal **cria novas**.
- Decidido: **contas fixas (recorrentes)** cadastradas e **parcelas futuras** visíveis. Decidido (consolidação): a partir delas o app mostra o **comprometimento dos próximos meses** e os lançamentos previstos ficam marcados como "previsto" até se realizarem.

### Regras de negócio levantadas (a confirmar nos PRDs)

- Decidido: **transferência entre contas próprias não é receita nem despesa** (pagamento da fatura do cartão, aplicação/resgate de investimento, transferência entre correntes). Motivo: senão os relatórios contam o mesmo gasto duas vezes.
- Decidido: **um lançamento digitado e o mesmo lançamento vindo de uma importação são reconhecidos como um só** (conciliação). Motivo: quem digita na hora e depois importa a fatura não pode duplicar gastos.
- Decidido: **regime de caixa para o cartão**: a compra no cartão conta no **mês em que a fatura é paga**, não no mês da compra. Cada parcela conta no mês da fatura em que é paga. O pagamento da fatura em si é transferência (não duplica). Decidido (consolidação): a compra guarda a data real, para consulta, e o cartão tem fatura por ciclo (fechamento e vencimento).
- Decidido (confirmado na rodada 5): o **limite mensal do cartão adicional** é acompanhado por **ciclo de fatura** (é assim que o banco controla), mesmo que o relatório use o mês de pagamento.
- Decidido (confirmado na rodada 5): **patrimônio total da família** soma correntes, investimentos, previdência e exterior convertido; **não inclui as contas dos filhos** (coerente com "o que o filho faz é dele").
- Decidido (consolidação): cada compra no cartão guarda **qual cartão (titular ou adicional de qual filho) fez a compra**, o que permite ver os gastos de cada filho.

### Fora de escopo

- Decidido: **conexão direta com bancos (Open Finance / open banking)**. Motivo: privacidade, razão de existir do projeto.
- Decidido (consolidação): venda ou uso por outras famílias (sistema para uma família só). Se errado: exigiria isolamento entre famílias, cadastro, cobrança.
- Decidido (consolidação): separação de finanças por pessoa / rateio entre o casal (todas as despesas são comuns).

## Requisitos técnicos

<!-- cada um aponta o requisito de negócio que serve -->

- Decidido (consolidação): **os dados financeiros não podem ser enviados a serviços de terceiros que possam lê-los** (serve: Problema/privacidade). Resolvido nas rodadas 2 e 3: servidor num computador de casa, sincronização só na rede de casa.
- Decidido (consolidação): **os dois administradores precisam ver os mesmos dados**, lançando cada um do seu aparelho (serve: casal lança gastos). Isso obriga algum tipo de sincronização ou servidor; é a principal tensão com a privacidade.
- Decidido: **celulares Android e computadores Windows** (serve: casal lança gastos onde estiver). Sem iPhone: a questão de custo da App Store e dos limites do PWA no iOS deixa de existir.
- Decidido: **servidor num dos computadores Windows de casa, que pode não ficar ligado o tempo todo**, e **nenhum custo adicional** para rodar (serve: privacidade; restrição declarada).
- Decidido (consolidação) (consequência das duas acima): **o app funciona sem o servidor** (offline-first / local-first). Cada aparelho guarda sua cópia, lança offline e sincroniza quando o servidor está acessível. O servidor é o ponto de encontro e a cópia de referência, não um requisito para usar. Se errado (app só funciona com servidor ligado): ninguém lança quando o servidor está desligado.
- Decidido: **sincronização só pelo Wi-Fi de casa** (serve: privacidade, custo zero). O servidor não fica exposto à internet; não há acesso remoto, túnel nem serviço de terceiro. Consequência aceita: o que um lança na rua só aparece para os outros depois que os dois aparelhos sincronizarem em casa.
- Decidido (consolidação): o servidor roda **em segundo plano no Windows, iniciando com o computador**, e os aparelhos **descobrem e sincronizam sozinhos** ao entrar no Wi-Fi de casa (serve: operação automática). Se errado: alguém precisa abrir o servidor ou apertar "sincronizar".
- Decidido (consolidação): como dois aparelhos podem alterar o mesmo dado offline, a sincronização precisa de **regra de conflito** definida (ex.: mesma transação editada nos dois celulares).
- Decidido (consolidação): **o servidor filtra o que cada usuário recebe** (serve: filhos não veem além do que é deles). Matriz decidida na rodada 5: casal vê tudo (família e filhos); cada filho vê só o próprio cartão adicional e a própria conta. O aparelho de um filho não pode receber a base da família inteira e esconder na tela; recebe só os dados dele. Restringe o desenho da sincronização (não é cópia integral para todos).
- Decidido (consolidação): **operação o mais automática possível**: instalação simples, atualização e cópia de segurança sem intervenção (serve: pedido do casal, servidor doméstico).
- Decidido: **quem instala e mantém o servidor é o marido**, com alguma familiaridade técnica; desejável **automatizar tudo o que der** e, no resto, **instruções claras passo a passo** (serve: operação). Decidido (consolidação): a especificação inclui um guia de instalação e de recuperação (o que fazer se o computador do servidor quebrar).
- Decidido: **cópia de segurança no OneDrive ou em disco externo** (serve: não perder dados, custo zero). Decidido (consolidação): **automática e periódica**, e **criptografada antes de sair do computador** quando for para o OneDrive (a Microsoft guarda só dados cifrados). Decidido (consolidação): a restauração é testável e documentada. Se errado (cópia sem criptografia no OneDrive): os dados financeiros ficariam legíveis por terceiro, contra o requisito central.
- Decidido: **o app pede PIN ao entrar** (serve: dados sensíveis, celulares dos filhos). Decidido (consolidação): PIN por usuário em cada aparelho; os dados guardados no aparelho são **criptografados**; se um celular for perdido, o casal **revoga o aparelho** no servidor. Se errado: quem pegar o celular perdido lê os dados copiando os arquivos do app.
- Decidido (consolidação): no celular, app Android **instalado direto (APK)**, sem loja; no computador, app Windows ou web local. A escolha entre app nativo, multiplataforma ou web fica para o TRD (consultar versões atuais).
- Decidido: **instituições hoje**: Banco do Brasil (corrente e investimento), Santander (corrente e investimento; cartões Visa e Mastercard), XP Investimentos (2 contas, cada uma com cartão Visa), BBPrev (previdência), contas no exterior (extrato mensal). A importação é especificada **por instituição e tipo de documento**. A verificar: que formatos (CSV, OFX, PDF) cada uma oferece hoje.
- Decidido (consolidação): **multimoeda mínima**: cada conta tem moeda (real ou dólar); conversão só para o patrimônio, pelo câmbio de fechamento mensal. Decidido: **câmbio digitado pelo casal** no fechamento do mês (sem busca automática; nenhuma integração externa). Decidido: hoje **só dólar** no exterior.
- A verificar (TRD/PRD do IR): a regra da Receita para converter saldos no exterior na declaração (cotação e data) pode diferir do câmbio de fechamento usado no patrimônio.
- Decidido (consolidação): **importação de PDF depende do layout de cada banco**; o suporte é por banco/emissor, e um layout novo pode quebrar a importação (serve: importação opcional). CSV/OFX é mais estável quando o banco oferece.

## Não-funcionais

| Item | Estado | Nota |
|---|---|---|
| Volume e escala | Decidido (consolidação) | 4 usuários no máximo, poucos milhares de lançamentos por ano; escala não é problema |
| Tempo de resposta | Decidido (consolidação) | tudo pode ser imediato; importação pode levar alguns segundos |
| Autenticação e autorização | Decidido | administrador vê tudo da família; dependente só o que é dele, filtrado no servidor |
| Dados sensíveis | Decidido | dados financeiros da família; privacidade é o requisito central |
| Disponibilidade e falha | Decidido (parcial) | servidor pode ficar desligado: app funciona offline e sincroniza depois. Não pode perder dados: cópia de segurança automática e sem custo (destino em aberto) |
| Integrações externas | Decidido | nenhuma: sem bancos, sem cotação |
| Onde roda | Decidido | celulares Android, computadores Windows; servidor num computador Windows de casa, nem sempre ligado; sincronização só na rede de casa |
| Quem opera | Decidido (parcial) | a família, querendo o mínimo de manutenção: tudo automático |
| Restrições declaradas | Decidido | sem Open Finance; nenhum custo adicional para rodar |

## Estados

| Item | Estado | Fonte / premissa | Data |
|---|---|---|---|
| Idioma português, moeda real (R$), regras brasileiras (Pix, fatura, parcelas) | Decidido (consolidação) | contexto do usuário; não contestado | 2026-11-02 |
| Uso privado de uma família | Decidido (consolidação) | não contestado | 2026-11-02 |
| Sem Open Finance | Decidido | "não ligado direto pelo Open Finance" | 2026-11-02 |
| Entrada manual + importação CSV/PDF opcional | Decidido | resposta 3, rodada 1 | 2026-11-02 |
| Contas correntes, cartões, investimentos | Decidido | resposta 1, rodada 1 | 2026-11-02 |
| Casal administra; todas as despesas comuns; contas conjuntas | Decidido | resposta 2, rodada 1 | 2026-11-02 |
| Filhos com cartão adicional; desejável que usem o app | Decidido | resposta 2, rodada 1 | 2026-11-02 |
| Transferência entre contas próprias não conta como despesa | Decidido | confirmado na rodada 3 | 2026-11-02 |
| Conciliação de digitado x importado | Decidido | confirmado na rodada 3 | 2026-11-02 |
| Despesa do cartão na data da compra, com fatura e parcelas | Substituído | trocado por regime de caixa na rodada 4 | 2026-11-02 |
| Compra identifica o cartão (titular / adicional) | Decidido (consolidação) | permite ver gastos de cada filho | 2026-11-02 |
| Celulares e computadores | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Servidor próprio, nem sempre ligado | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Nenhum custo adicional | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Operação o mais automática possível | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| App funciona offline e sincroniza com o servidor (local-first) | Decidido (consolidação) | consequência de servidor nem sempre ligado | 2026-11-02 |
| Servidor filtra os dados por usuário | Decidido (consolidação) | consequência de "filhos não veem além do que é deles" | 2026-11-02 |
| Filhos anotam/categorizam, acompanham limite ou mesada, só veem o que é deles | Decidido | resposta 2, rodada 2 | 2026-11-02 |
| Filhos poderão ter contas próprias no futuro | Decidido | resposta 2, rodada 2 | 2026-11-02 |
| Investimentos por aplicação, sem cotação | Decidido | resposta 3, rodada 2 (opção b) | 2026-11-02 |
| Android e Windows (sem iPhone) | Decidido | resposta 1, rodada 3 | 2026-11-02 |
| Servidor num computador Windows de casa | Decidido | resposta 1, rodada 3 | 2026-11-02 |
| Sincronização só no Wi-Fi de casa, sem acesso remoto | Decidido | resposta 2, rodada 3 | 2026-11-02 |
| Servidor em segundo plano; sincronização automática ao chegar em casa | Decidido (consolidação) | operação automática | 2026-11-02 |
| Filhos já têm conta própria (mesada por Pix) e limite mensal no cartão adicional | Decidido | resposta 3, rodada 3 | 2026-11-02 |
| Filho registra a compra na hora; conciliada com a fatura importada | Decidido (consolidação) | estende a conciliação ao filho | 2026-11-02 |
| Limite do cartão adicional cadastrado pelo casal; app avisa, não bloqueia | Decidido (consolidação) | o bloqueio é do banco | 2026-11-02 |
| Contas dos filhos entram no app; movimentação delas fora dos relatórios da família | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Mesada: despesa da família, receita do filho | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Compras do cartão adicional: despesa da família | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Relatórios: categoria no mês, evolução, orçamento x realizado, patrimônio | Decidido | resposta 2, rodada 4 | 2026-11-02 |
| Cartão em regime de caixa (mês do pagamento da fatura) | Decidido | resposta 2, rodada 4 (substitui a inferência "data da compra") | 2026-11-02 |
| Limite do cartão adicional por ciclo de fatura | Decidido | dedução confirmada na rodada 5 | 2026-11-02 |
| Patrimônio da família sem as contas dos filhos | Decidido | dedução confirmada na rodada 5 | 2026-11-02 |
| Instituições: BB, Santander, XP (2), BBPrev, exterior | Decidido | resposta 3, rodada 4 | 2026-11-02 |
| Formatos de exportação de cada instituição | Em aberto | verificar (fonte e data) ou amostras do casal | 2026-11-02 |
| Exterior: extrato mensal em dólar, câmbio de fechamento no patrimônio | Decidido | resposta 3, rodada 4 | 2026-11-02 |
| Previdência BBPrev: saldo e contribuições, no patrimônio | Decidido | dedução confirmada na rodada 5 | 2026-11-02 |
| Casal vê as contas dos filhos; filho vê o próprio cartão adicional e a própria conta | Decidido | resposta 1, rodada 5 | 2026-11-02 |
| Câmbio digitado à mão; só dólar | Decidido | resposta 2, rodada 5 | 2026-11-02 |
| Histórico: exportação do Minhas Economias; período a verificar | Decidido / Em aberto | resposta 3, rodada 5; casal verifica o período, amostra depois | 2026-11-02 |
| Apoio à declaração do IR | Decidido | rodada 5 | 2026-11-02 |
| App gera relatórios de apoio, não o arquivo oficial da Receita | Decidido (consolidação) | escopo controlado | 2026-11-02 |
| Regra de conversão de saldos no exterior para o IR | Em aberto | verificar fonte oficial (Receita) com data | 2026-11-02 |
| App gera relatórios de apoio ao IR, não o arquivo oficial | Decidido | confirmado na rodada 6 | 2026-11-02 |
| IR: despesas dedutíveis com CPF/CNPJ e paciente; bens e direitos | Decidido | resposta 1, rodada 6 | 2026-11-02 |
| IR: declaração conjunta; filhos dependentes | Decidido | resposta 1, rodada 6 | 2026-11-02 |
| IR: rendimentos e renda variável fora | Decidido (consolidação) | não escolhidos na rodada 6 | 2026-11-02 |
| Categorias com subcategorias; importadas, editáveis, novas | Decidido | resposta 2, rodada 6 | 2026-11-02 |
| Cópia de segurança no OneDrive ou disco externo | Decidido | resposta 2, rodada 6 | 2026-11-02 |
| Cópia automática e criptografada antes do OneDrive | Decidido (consolidação) | privacidade | 2026-11-02 |
| Contas fixas e parcelas futuras; comprometimento futuro | Decidido / Decidido (consolidação) | resposta 3, rodada 6; comprometimento é dedução | 2026-11-02 |
| PIN ao entrar | Decidido | resposta 3, rodada 6 | 2026-11-02 |
| Dados criptografados no aparelho; revogar aparelho perdido | Decidido (consolidação) | consequência do PIN e dos celulares dos filhos | 2026-11-02 |
| Marido instala e mantém; automatizar e documentar passo a passo | Decidido | resposta 3, rodada 6 | 2026-11-02 |
| Investimentos: valor atual digitado ou importado do extrato da corretora; rendimento = variação do saldo descontados aportes e resgates | Decidido (consolidação) | não perguntado; nível (b) da rodada 2 | 2026-11-02 |
| Há histórico a migrar | Decidido | resposta 3, rodada 4 | 2026-11-02 |

## Alternativas descartadas

- **Continuar no Minhas Economias com Open Finance:** descartado pela privacidade.
- **Dados só num servidor sempre ligado (cliente-servidor clássico):** descartado; o servidor pode ficar desligado.
- **Servidor alugado (VPS) ou nuvem paga:** descartado; nenhum custo adicional.
- **IR: rendimentos, ganhos em renda variável e DARF:** não escolhidos na rodada 6.
- **Gerar o arquivo oficial da declaração:** descartado; só relatórios de apoio.
- **Câmbio automático (PTAX do Banco Central):** descartado na rodada 5; o casal digita o câmbio.
- **Regime de competência no cartão (compra no mês da compra):** descartado na rodada 4; vale o mês do pagamento.
- **Mesada como transferência interna:** descartada na rodada 4; é despesa da família.
- **Acesso remoto ao servidor (túnel/VPN gratuito de terceiro):** descartado na rodada 3; sincronização só em casa.
- **App para iPhone / PWA:** desnecessário; a família usa só Android.
- **Cotação automática de investimentos:** descartada na rodada 2 (opção b).
- **Melhorar a planilha:** não discutido como opção; a ausência de análises e gráficos é a queixa. (Registrar se o casal quiser reconsiderar.)

## Pendências

Aguardando o casal:

- Período do histórico no Minhas Economias.
- Amostras de extratos e faturas (BB, Santander, XP x2, BBPrev, exterior) e da exportação do Minhas Economias, com dados pessoais apagados. Ficam só na sessão; no repositório entra apenas a descrição do formato. Pedir quando chegar o PRD de importação.

A verificar em fonte, com data (no PRD ou TRD correspondente):

- Formatos de exportação de cada instituição (CSV, OFX, PDF).
- Regra da Receita para converter saldos no exterior em bens e direitos.
- Versões atuais da stack (TRD).

## Consolidação (2026-11-02, validada pelo casal)

Todas as deduções (antes marcadas como inferidas) foram confirmadas pelo casal na validação da consolidação; a divisão em PRDs também.

Proposta de divisão em PRDs (`docs/prds/`), na ordem de dependência:

| PRD | Escopo |
|---|---|
| 001 Usuários, perfis e acesso | casal e filhos, o que cada um vê e faz, PIN, aparelho perdido |
| 002 Contas e cartões | correntes, cartões com adicionais e fatura, investimentos, previdência, exterior em dólar; dono de cada conta |
| 003 Lançamentos e categorias | manual, categorias e subcategorias, transferências, parcelas, recorrentes, regime de caixa, dados de IR no lançamento |
| 004 Importação e conciliação | extratos e faturas CSV/PDF por instituição, conciliação com o digitado |
| 005 Migração do Minhas Economias | histórico, categorias |
| 006 Investimentos e previdência | aplicações, aportes, resgates, rendimento, saldo |
| 007 Orçamento | planejado por categoria x realizado |
| 008 Relatórios e patrimônio | categoria no mês, evolução, patrimônio com câmbio de fechamento |
| 009 Filhos: limite e mesada | limite do cartão adicional por ciclo, mesada, aviso |
| 010 Apoio ao imposto de renda | despesas dedutíveis, bens e direitos, declaração conjunta |
| 011 Sincronização e operação | comportamento offline, sincronização em casa, conflitos, cópia de segurança, instalação e recuperação (lado do usuário) |

Depois: `docs/trd.md` e ADRs (local-first com servidor doméstico; stack Android/Windows; criptografia no aparelho e na cópia; distribuição por APK) e `docs/glossario.md`.
