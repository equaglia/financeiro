# Brainstorm: controle financeiro familiar

**Lente:** especificação de projeto de desenvolvimento
**Última sessão:** 2026-11-02 (rodada 4 de perguntas em andamento)

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
- Decidido: no futuro os filhos podem ter **contas correntes e de investimento próprias**. Consequência: cada conta tem um dono (o casal ou um filho), e o que cada usuário vê depende disso. Os relatórios da família (casal) incluem as contas dos filhos? Em aberto.

### Capacidades

- Cadastrar e acompanhar **contas correntes**, **cartões de crédito** (com cartões adicionais) e **contas de investimento**.
- Decidido (nível b): **investimentos por aplicação** (CDB, fundo, ações, Tesouro etc.) com aportes, resgates e rendimento. Sem cotação automática: o valor atual é informado à mão ou importado. Descartados: (a) só saldo por conta, pouco para análise; (c) cotação automática, dependência externa.
- **Dependentes:** anotar e categorizar as próprias compras no cartão adicional; acompanhar o consumo do mês contra o limite do cartão adicional; acompanhar a mesada.
  - Inferido: o filho pode **registrar a compra no celular na hora**; quando o casal importa a fatura, a compra importada é conciliada com a registrada pelo filho (identificada pelo cartão adicional). Se errado: o filho só categoriza depois da importação.
  - Inferido: o limite mensal do cartão adicional é **cadastrado pelo casal** no app; o app mostra quanto já foi usado e avisa perto do limite. O app não bloqueia nada (quem bloqueia é o banco).
  - Inferido: a **mesada é despesa da família** (categoria própria) e **receita do filho**. Se as contas dos filhos entrarem no app, o Pix da mesada vira transferência entre contas da família? Em aberto (rodada 4), muda os relatórios.
- **Lançamento manual** de receitas e despesas (como no Minhas Economias).
- **Importação opcional** de extratos e faturas em **CSV e PDF**.
- **Relatórios gráficos e análises** (detalhar quais na rodada 3).

### Regras de negócio levantadas (a confirmar nos PRDs)

- Decidido: **transferência entre contas próprias não é receita nem despesa** (pagamento da fatura do cartão, aplicação/resgate de investimento, transferência entre correntes). Motivo: senão os relatórios contam o mesmo gasto duas vezes.
- Decidido: **um lançamento digitado e o mesmo lançamento vindo de uma importação são reconhecidos como um só** (conciliação). Motivo: quem digita na hora e depois importa a fatura não pode duplicar gastos.
- Inferido: a despesa no cartão é registrada **na data da compra**, e o cartão tem **fatura por ciclo** (fechamento e vencimento) e **compras parceladas**. Se errado: muda como o orçamento do mês é calculado (competência x caixa); ver Pendências.
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
| Despesa do cartão na data da compra, com fatura e parcelas | Inferido | confirmar (competência x caixa) | 2026-11-02 |
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
| Mesada: despesa da família, receita do filho | Inferido | depende de as contas dos filhos entrarem no app | 2026-11-02 |

## Alternativas descartadas

- **Continuar no Minhas Economias com Open Finance:** descartado pela privacidade.
- **Dados só num servidor sempre ligado (cliente-servidor clássico):** descartado; o servidor pode ficar desligado.
- **Servidor alugado (VPS) ou nuvem paga:** descartado; nenhum custo adicional.
- **Acesso remoto ao servidor (túnel/VPN gratuito de terceiro):** descartado na rodada 3; sincronização só em casa.
- **App para iPhone / PWA:** desnecessário; a família usa só Android.
- **Cotação automática de investimentos:** descartada na rodada 2 (opção b).
- **Melhorar a planilha:** não discutido como opção; a ausência de análises e gráficos é a queixa. (Registrar se o casal quiser reconsiderar.)

## Pendências

Rodada 4 (perguntas feitas em 2026-11-02):

1. **Contas dos filhos no app.** As contas individuais (mesada) entram no app agora? Os relatórios do casal incluem os gastos dos filhos (cartão adicional e conta própria) ou a mesada é a despesa da família e o resto é só do filho? Destrava: regra da mesada, perfis, relatórios.
2. **Relatórios, análises e orçamento**, e **competência x caixa** (compra no cartão conta no mês da compra ou no mês em que a fatura é paga). Destrava: PRD de relatórios e orçamento.
3. **Bancos, cartões e corretoras** usados, e **histórico a migrar** (planilha atual; exportação do Minhas Economias, se existir). Destrava: PRD de importação e de migração.

Para rodadas seguintes:

- Destino da cópia de segurança, sem custo (disco externo, nuvem que já usam, com os dados criptografados).
- Investimentos: como entra o valor atual (à mão, importação de extrato da corretora), quais rendimentos acompanhar, imposto de renda.
- Categorias: aproveitar as do Minhas Economias ou criar do zero.
- Lançamentos recorrentes (contas fixas) e parcelas futuras.
- Conhecimento técnico do casal (quem instala e mantém o servidor).
- Senha/bloqueio do app nos celulares; o que acontece se um celular for perdido.
