# Brainstorm: controle financeiro familiar

**Lente:** especificação de projeto de desenvolvimento
**Última sessão:** 2026-11-02 (rodada 5 de perguntas em andamento)

Documento de trabalho: guarda decisões, premissas, alternativas descartadas e pendências. É a base para os PRDs (`docs/prds/`) e o TRD (`docs/trd.md`), não a especificação final. Uma sessão nova retoma daqui (seção "Estados" e "Pendências").

## Problema

- A família usava o app Minhas Economias, com todos os lançamentos digitados à mão. Na última atualização o app passou a exigir conexão por Open Finance (open banking), e o casal não aceita entregar o acesso às contas a terceiros (risco de perda de privacidade).
- Hoje usam uma planilha simples de receitas e despesas, sem gráficos, relatórios nem análises.
- **O que se quer:** o controle que o app dava (contas, cartões, investimentos, lançamentos manuais) com análise e relatórios melhores que a planilha, **sem que os dados financeiros passem por serviços de terceiros**.

**Reframing:** o diferencial do sistema não é ter funcionalidades novas, é a privacidade. Toda decisão de arquitetura (onde os dados ficam, como os dois celulares/computadores se sincronizam, se há cotação de mercado externa) é julgada primeiro por esse critério.

## Requisitos de negócio

### Usuários e perfis

| Perfil | Quem | O que faz |
|---|---|---|
| Administrador | o casal (2 pessoas) | lança, importa, categoriza, analisa, administra contas, cartões e investimentos |
| Dependente | 2 filhos | anota e categoriza as próprias compras, acompanha limite ou mesada; **não vê nada além do que é dele** |

- Todas as despesas são comuns: não há separação "dinheiro dele / dela" nem rateio.
- Contas correntes, contas de investimento e cartões de crédito são conjuntos.
- Os filhos têm cartões adicionais (dependentes) no cartão do casal. Hoje não se interessam pelo app; o casal gostaria que passassem a usar.
- Decidido: hoje cada filho já tem **conta individual própria**, onde recebe a **mesada por Pix**, e o cartão adicional tem **limite mensal**.
- Decidido: **as contas dos filhos entram no app**. No futuro podem ter também **contas correntes e de investimento próprias**.
- Decidido: **o que o filho faz com o dinheiro da conta dele é dele**: a movimentação das contas dos filhos não entra nos relatórios da família. Consequência: cada conta tem um dono (o casal ou um filho), e o que cada usuário vê depende disso. Os relatórios da família (casal) incluem as contas dos filhos? Em aberto.

### Capacidades

- Cadastrar e acompanhar **contas correntes**, **cartões de crédito** (com cartões adicionais) e **contas de investimento**.
- Decidido (nível b): **investimentos por aplicação** (CDB, fundo, ações, Tesouro etc.) com aportes, resgates e rendimento. Sem cotação automática: o valor atual é informado à mão ou importado. Descartados: (a) só saldo por conta, pouco para análise; (c) cotação automática, dependência externa.
- **Dependentes:** anotar e categorizar as próprias compras no cartão adicional; acompanhar o consumo do mês contra o limite do cartão adicional; acompanhar a mesada.
  - Inferido: o filho pode **registrar a compra no celular na hora**; quando o casal importa a fatura, a compra importada é conciliada com a registrada pelo filho (identificada pelo cartão adicional). Se errado: o filho só categoriza depois da importação.
  - Inferido: o limite mensal do cartão adicional é **cadastrado pelo casal** no app; o app mostra quanto já foi usado e avisa perto do limite. O app não bloqueia nada (quem bloqueia é o banco).
  - Decidido: o **Pix da mesada é despesa da família** (não é transferência interna) e receita na conta do filho.
  - Decidido: **as compras no cartão adicional são despesas da família** e entram nos relatórios do casal (o casal paga a fatura).
- **Lançamento manual** de receitas e despesas (como no Minhas Economias).
- **Importação opcional** de extratos e faturas em **CSV e PDF**.
- Decidido: **relatórios e análises**: gastos por categoria no mês; evolução ao longo dos meses; orçamento planejado x realizado; patrimônio total.
- Decidido: **orçamento** por categoria (planejado), comparado com o realizado.
- Decidido: **contas no exterior com controle simplificado**: lançamento a partir do extrato mensal, saldo em **dólar**; no patrimônio total, convertido pelo **câmbio do fechamento do mês**.
- Decidido: **previdência** (BBPrev) entra como tipo de conta; inferido: acompanhada pelo saldo e pelas contribuições do extrato, e somada ao patrimônio.
- Decidido: **migração do histórico** existente (formato e período: rodada 5).

### Regras de negócio levantadas (a confirmar nos PRDs)

- Decidido: **transferência entre contas próprias não é receita nem despesa** (pagamento da fatura do cartão, aplicação/resgate de investimento, transferência entre correntes). Motivo: senão os relatórios contam o mesmo gasto duas vezes.
- Decidido: **um lançamento digitado e o mesmo lançamento vindo de uma importação são reconhecidos como um só** (conciliação). Motivo: quem digita na hora e depois importa a fatura não pode duplicar gastos.
- Decidido: **regime de caixa para o cartão**: a compra no cartão conta no **mês em que a fatura é paga**, não no mês da compra. Cada parcela conta no mês da fatura em que é paga. O pagamento da fatura em si é transferência (não duplica). Inferido: a compra guarda a data real, para consulta, e o cartão tem fatura por ciclo (fechamento e vencimento).
- Inferido: o **limite mensal do cartão adicional** é acompanhado por **ciclo de fatura** (é assim que o banco controla), mesmo que o relatório use o mês de pagamento.
- Inferido: **patrimônio total da família** soma correntes, investimentos, previdência e exterior convertido; **não inclui as contas dos filhos** (coerente com "o que o filho faz é dele"). Se errado: patrimônio da família incluiria saldo dos filhos.
- Inferido: cada compra no cartão guarda **qual cartão (titular ou adicional de qual filho) fez a compra**, o que permite ver os gastos de cada filho.

### Fora de escopo

- Decidido: **conexão direta com bancos (Open Finance / open banking)**. Motivo: privacidade, razão de existir do projeto.
- Inferido: venda ou uso por outras famílias (sistema para uma família só). Se errado: exigiria isolamento entre famílias, cadastro, cobrança.
- Inferido: separação de finanças por pessoa / rateio entre o casal (todas as despesas são comuns).

## Requisitos técnicos

<!-- cada um aponta o requisito de negócio que serve -->

- Inferido: **os dados financeiros não podem ser enviados a serviços de terceiros que possam lê-los** (serve: Problema/privacidade). Em aberto o quanto isso restringe: só nos aparelhos da família, servidor próprio, nuvem com criptografia (rodada 2).
- Inferido: **os dois administradores precisam ver os mesmos dados**, lançando cada um do seu aparelho (serve: casal lança gastos). Isso obriga algum tipo de sincronização ou servidor; é a principal tensão com a privacidade.
- Decidido: **celulares Android e computadores Windows** (serve: casal lança gastos onde estiver). Sem iPhone: a questão de custo da App Store e dos limites do PWA no iOS deixa de existir.
- Decidido: **servidor num dos computadores Windows de casa, que pode não ficar ligado o tempo todo**, e **nenhum custo adicional** para rodar (serve: privacidade; restrição declarada).
- Inferido (consequência das duas acima): **o app funciona sem o servidor** (offline-first / local-first). Cada aparelho guarda sua cópia, lança offline e sincroniza quando o servidor está acessível. O servidor é o ponto de encontro e a cópia de referência, não um requisito para usar. Se errado (app só funciona com servidor ligado): ninguém lança quando o servidor está desligado.
- Decidido: **sincronização só pelo Wi-Fi de casa** (serve: privacidade, custo zero). O servidor não fica exposto à internet; não há acesso remoto, túnel nem serviço de terceiro. Consequência aceita: o que um lança na rua só aparece para os outros depois que os dois aparelhos sincronizarem em casa.
- Inferido: o servidor roda **em segundo plano no Windows, iniciando com o computador**, e os aparelhos **descobrem e sincronizam sozinhos** ao entrar no Wi-Fi de casa (serve: operação automática). Se errado: alguém precisa abrir o servidor ou apertar "sincronizar".
- Inferido: como dois aparelhos podem alterar o mesmo dado offline, a sincronização precisa de **regra de conflito** definida (ex.: mesma transação editada nos dois celulares).
- Inferido: **o servidor filtra o que cada usuário recebe** (serve: filhos não veem além do que é deles). O aparelho de um filho não pode receber a base da família inteira e esconder na tela; recebe só os dados dele. Restringe o desenho da sincronização (não é cópia integral para todos).
- Inferido: **operação o mais automática possível**: instalação simples, atualização e cópia de segurança sem intervenção (serve: pedido do casal, servidor doméstico).
- Inferido: no celular, app Android **instalado direto (APK)**, sem loja; no computador, app Windows ou web local. A escolha entre app nativo, multiplataforma ou web fica para o TRD (consultar versões atuais).
- Decidido: **instituições hoje**: Banco do Brasil (corrente e investimento), Santander (corrente e investimento; cartões Visa e Mastercard), XP Investimentos (2 contas, cada uma com cartão Visa), BBPrev (previdência), contas no exterior (extrato mensal). A importação é especificada **por instituição e tipo de documento**. A verificar: que formatos (CSV, OFX, PDF) cada uma oferece hoje.
- Inferido: **multimoeda mínima**: cada conta tem moeda (real ou dólar); conversão só para o patrimônio, pelo câmbio de fechamento mensal. Origem do câmbio (à mão ou PTAX do Banco Central): rodada 5.
- Inferido: **importação de PDF depende do layout de cada banco**; o suporte é por banco/emissor, e um layout novo pode quebrar a importação (serve: importação opcional). CSV/OFX é mais estável quando o banco oferece.

## Não-funcionais

| Item | Estado | Nota |
|---|---|---|
| Volume e escala | Inferido | 4 usuários no máximo, poucos milhares de lançamentos por ano; escala não é problema |
| Tempo de resposta | Inferido | tudo pode ser imediato; importação pode levar alguns segundos |
| Autenticação e autorização | Decidido (parcial) | administrador vê tudo da família; dependente só o que é dele, filtrado no servidor |
| Dados sensíveis | Decidido | dados financeiros da família; privacidade é o requisito central |
| Disponibilidade e falha | Decidido (parcial) | servidor pode ficar desligado: app funciona offline e sincroniza depois. Não pode perder dados: cópia de segurança automática e sem custo (destino em aberto) |
| Integrações externas | Decidido | nenhuma: sem bancos, sem cotação |
| Onde roda | Decidido | celulares Android, computadores Windows; servidor num computador Windows de casa, nem sempre ligado; sincronização só na rede de casa |
| Quem opera | Decidido (parcial) | a família, querendo o mínimo de manutenção: tudo automático |
| Restrições declaradas | Decidido | sem Open Finance; nenhum custo adicional para rodar |

## Estados

| Item | Estado | Fonte / premissa | Data |
|---|---|---|---|
| Idioma português, moeda real (R$), regras brasileiras (Pix, fatura, parcelas) | Inferido | contexto do usuário; não contestado | 2026-11-02 |
| Uso privado de uma família | Inferido | não contestado | 2026-11-02 |
| Sem Open Finance | Decidido | "não ligado direto pelo Open Finance" | 2026-11-02 |
| Entrada manual + importação CSV/PDF opcional | Decidido | resposta 3, rodada 1 | 2026-11-02 |
| Contas correntes, cartões, investimentos | Decidido | resposta 1, rodada 1 | 2026-11-02 |
| Casal administra; todas as despesas comuns; contas conjuntas | Decidido | resposta 2, rodada 1 | 2026-11-02 |
| Filhos com cartão adicional; desejável que usem o app | Decidido | resposta 2, rodada 1 | 2026-11-02 |
| Transferência entre contas próprias não conta como despesa | Decidido | confirmado na rodada 3 | 2026-11-02 |
| Conciliação de digitado x importado | Decidido | confirmado na rodada 3 | 2026-11-02 |
| Despesa do cartão na data da compra, com fatura e parcelas | Substituído | trocado por regime de caixa na rodada 4 | 2026-11-02 |
| Compra identifica o cartão (titular / adicional) | Inferido | permite ver gastos de cada filho | 2026-11-02 |
| Celulares e computadores | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Servidor próprio, nem sempre ligado | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Nenhum custo adicional | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| Operação o mais automática possível | Decidido | resposta 1, rodada 2 | 2026-11-02 |
| App funciona offline e sincroniza com o servidor (local-first) | Inferido | consequência de servidor nem sempre ligado | 2026-11-02 |
| Servidor filtra os dados por usuário | Inferido | consequência de "filhos não veem além do que é deles" | 2026-11-02 |
| Filhos anotam/categorizam, acompanham limite ou mesada, só veem o que é deles | Decidido | resposta 2, rodada 2 | 2026-11-02 |
| Filhos poderão ter contas próprias no futuro | Decidido | resposta 2, rodada 2 | 2026-11-02 |
| Investimentos por aplicação, sem cotação | Decidido | resposta 3, rodada 2 (opção b) | 2026-11-02 |
| Android e Windows (sem iPhone) | Decidido | resposta 1, rodada 3 | 2026-11-02 |
| Servidor num computador Windows de casa | Decidido | resposta 1, rodada 3 | 2026-11-02 |
| Sincronização só no Wi-Fi de casa, sem acesso remoto | Decidido | resposta 2, rodada 3 | 2026-11-02 |
| Servidor em segundo plano; sincronização automática ao chegar em casa | Inferido | operação automática | 2026-11-02 |
| Filhos já têm conta própria (mesada por Pix) e limite mensal no cartão adicional | Decidido | resposta 3, rodada 3 | 2026-11-02 |
| Filho registra a compra na hora; conciliada com a fatura importada | Inferido | estende a conciliação ao filho | 2026-11-02 |
| Limite do cartão adicional cadastrado pelo casal; app avisa, não bloqueia | Inferido | o bloqueio é do banco | 2026-11-02 |
| Contas dos filhos entram no app; movimentação delas fora dos relatórios da família | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Mesada: despesa da família, receita do filho | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Compras do cartão adicional: despesa da família | Decidido | resposta 1, rodada 4 | 2026-11-02 |
| Relatórios: categoria no mês, evolução, orçamento x realizado, patrimônio | Decidido | resposta 2, rodada 4 | 2026-11-02 |
| Cartão em regime de caixa (mês do pagamento da fatura) | Decidido | resposta 2, rodada 4 (substitui a inferência "data da compra") | 2026-11-02 |
| Limite do cartão adicional por ciclo de fatura | Inferido | é como o banco controla | 2026-11-02 |
| Patrimônio da família sem as contas dos filhos | Inferido | coerente com "o que o filho faz é dele" | 2026-11-02 |
| Instituições: BB, Santander, XP (2), BBPrev, exterior | Decidido | resposta 3, rodada 4 | 2026-11-02 |
| Formatos de exportação de cada instituição | Em aberto | verificar (fonte e data) ou amostras do casal | 2026-11-02 |
| Exterior: extrato mensal em dólar, câmbio de fechamento no patrimônio | Decidido | resposta 3, rodada 4 | 2026-11-02 |
| Previdência BBPrev: saldo e contribuições, no patrimônio | Inferido | detalhe não discutido | 2026-11-02 |
| Há histórico a migrar | Decidido | resposta 3, rodada 4 | 2026-11-02 |

## Alternativas descartadas

- **Continuar no Minhas Economias com Open Finance:** descartado pela privacidade.
- **Dados só num servidor sempre ligado (cliente-servidor clássico):** descartado; o servidor pode ficar desligado.
- **Servidor alugado (VPS) ou nuvem paga:** descartado; nenhum custo adicional.
- **Regime de competência no cartão (compra no mês da compra):** descartado na rodada 4; vale o mês do pagamento.
- **Mesada como transferência interna:** descartada na rodada 4; é despesa da família.
- **Acesso remoto ao servidor (túnel/VPN gratuito de terceiro):** descartado na rodada 3; sincronização só em casa.
- **App para iPhone / PWA:** desnecessário; a família usa só Android.
- **Cotação automática de investimentos:** descartada na rodada 2 (opção b).
- **Melhorar a planilha:** não discutido como opção; a ausência de análises e gráficos é a queixa. (Registrar se o casal quiser reconsiderar.)

## Pendências

Rodada 5 (perguntas feitas em 2026-11-02):

1. **O casal vê as contas dos filhos?** E os filhos veem o próprio cartão adicional inteiro (todas as compras dele) e a própria conta? Destrava: matriz de permissões.
2. **Câmbio do fechamento mensal:** digitado à mão ou buscado automaticamente (PTAX do Banco Central, dado público; nada da família é enviado, mas o servidor precisa de internet). Outras moedas além do dólar? Investimentos no exterior também? Destrava: integração externa, multimoeda.
3. **Histórico a migrar:** o que é (planilha: formato e colunas; exportação do Minhas Economias: formato), de quando até quando. E se o casal pode fornecer **amostras de extratos e faturas** de cada instituição (anonimizadas; onde guardá-las, já que o repositório está no GitHub). Destrava: PRDs de migração e importação.

Para rodadas seguintes (ou inferir na consolidação):

- Destino da cópia de segurança, sem custo (disco externo, nuvem que já usam, com os dados criptografados).
- Investimentos: como entra o valor atual (à mão, importação de extrato da corretora), quais rendimentos acompanhar, imposto de renda.
- Categorias: aproveitar as do Minhas Economias ou criar do zero; subcategorias.
- Lançamentos recorrentes (contas fixas) e parcelas futuras.
- Conhecimento técnico do casal (quem instala e mantém o servidor).
- Senha/bloqueio do app nos celulares; o que acontece se um celular for perdido.
