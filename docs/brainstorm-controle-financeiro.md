# Brainstorm: controle financeiro familiar

**Lente:** especificação de projeto de desenvolvimento
**Última sessão:** 2026-11-02 (rodada 2 de perguntas em andamento)

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
| Dependente | 2 filhos | detalhar na rodada 2 (ver Pendências) |

- Todas as despesas são comuns: não há separação "dinheiro dele / dela" nem rateio.
- Contas correntes, contas de investimento e cartões de crédito são conjuntos.
- Os filhos têm cartões adicionais (dependentes) no cartão do casal. Hoje não se interessam pelo app; o casal gostaria que passassem a usar.

### Capacidades

- Cadastrar e acompanhar **contas correntes**, **cartões de crédito** (com cartões adicionais) e **contas de investimento**.
- **Lançamento manual** de receitas e despesas (como no Minhas Economias).
- **Importação opcional** de extratos e faturas em **CSV e PDF**.
- **Relatórios gráficos e análises** (detalhar quais na rodada 3).

### Regras de negócio levantadas (a confirmar nos PRDs)

- Inferido: **transferência entre contas próprias não é receita nem despesa** (pagamento da fatura do cartão, aplicação/resgate de investimento, transferência entre correntes). Se errado: os relatórios contariam o mesmo gasto duas vezes (na compra e no pagamento da fatura).
- Inferido: **um lançamento digitado e o mesmo lançamento vindo de uma importação devem ser reconhecidos como um só** (conciliação). Se errado: quem digita na hora e depois importa a fatura duplica gastos.
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
- Inferido: **importação de PDF depende do layout de cada banco**; o suporte é por banco/emissor, e um layout novo pode quebrar a importação (serve: importação opcional). CSV/OFX é mais estável quando o banco oferece.

## Não-funcionais

| Item | Estado | Nota |
|---|---|---|
| Volume e escala | Inferido | 4 usuários no máximo, poucos milhares de lançamentos por ano; escala não é problema |
| Tempo de resposta | Inferido | tudo pode ser imediato; importação pode levar alguns segundos |
| Autenticação e autorização | Em levantamento | 2 perfis (administrador, dependente); o que o dependente vê e faz: rodada 2 |
| Dados sensíveis | Decidido | dados financeiros da família; privacidade é o requisito central |
| Disponibilidade e falha | Inferido | não pode perder dados (cópia de segurança obrigatória); ficar fora do ar algumas horas é aceitável |
| Integrações externas | Decidido (parcial) | nenhuma com bancos; cotação de investimentos: rodada 2 |
| Onde roda | Em levantamento | rodada 2 |
| Quem opera | Em levantamento | rodada 2 |
| Restrições declaradas | Decidido | sem Open Finance |

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
| Transferência entre contas próprias não conta como despesa | Inferido | regra usual; confirmar no PRD | 2026-11-02 |
| Conciliação de digitado x importado | Inferido | consequência de ter as duas entradas | 2026-11-02 |
| Despesa do cartão na data da compra, com fatura e parcelas | Inferido | confirmar (competência x caixa) | 2026-11-02 |
| Compra identifica o cartão (titular / adicional) | Inferido | permite ver gastos de cada filho | 2026-11-02 |

## Alternativas descartadas

- **Continuar no Minhas Economias com Open Finance:** descartado pela privacidade.
- **Melhorar a planilha:** não discutido como opção; a ausência de análises e gráficos é a queixa. (Registrar se o casal quiser reconsiderar.)

## Pendências

Rodada 2 (perguntas feitas em 2026-11-02):

1. **Onde os dados podem ficar** e em quais aparelhos o app é usado (celular, computador). Destrava: arquitetura inteira (local, servidor em casa, servidor alugado, nuvem criptografada).
2. **O que os filhos fariam no app.** Destrava: perfil dependente, permissões.
3. **Profundidade dos investimentos** (só saldo por conta ou ativos, rentabilidade, cotações). Destrava: se há integração externa de cotações e o tamanho do módulo.

Para rodadas seguintes:

- Bancos e emissores de cartão usados (quais extratos/faturas importar).
- Migração do histórico (planilha atual, exportação do Minhas Economias).
- Relatórios e análises desejados; orçamento por categoria e metas.
- Categorias: aproveitar as do Minhas Economias ou criar do zero.
- Competência x caixa no orçamento do mês.
- Quem mantém o sistema depois de pronto; conhecimento técnico do casal.
