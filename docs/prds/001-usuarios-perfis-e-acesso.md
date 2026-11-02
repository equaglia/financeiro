---
prd_number: "001"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: crítica
created: 2026-11-02
issue: ""
depends_on: []
references:
  - docs/brainstorm-controle-financeiro.md
---

# PRD 001: Usuários, perfis e acesso

## 1. Contexto

- **Produto/área**: controle financeiro de uma família (casal e dois filhos), usado em celulares Android e computadores Windows, com um servidor num computador de casa. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: o casal controla receitas e despesas numa planilha simples, compartilhada só entre os dois. Os filhos não participam. Antes usavam o Minhas Economias, abandonado quando passou a exigir Open Finance.
- **Problema**: o app novo vai ter dados de toda a família em vários aparelhos, inclusive nos celulares dos filhos. Sem perfis e controle de acesso, um filho veria as finanças da família, e um celular perdido exporia tudo. Este PRD define quem usa o app, o que cada um vê e faz, e como o acesso é protegido. Todos os outros PRDs dependem disso.

> Contexto técnico (como os dados são filtrados, criptografia, sincronização) fica em `docs/trd.md` e nos ADRs.

## 2. Solução Proposta

### Visão de produto

- Uma família, dois perfis: **administrador** (o casal) e **dependente** (os filhos).
- O casal vê e administra tudo: as finanças da família e as contas dos filhos.
- Cada filho vê só o que é dele: a própria conta individual (e contas próprias futuras) e todas as compras do próprio cartão adicional.
- Cada aparelho é **vinculado** à família por um administrador, na rede de casa, e o app pede **PIN** ao entrar.
- Um aparelho perdido pode ser **revogado** pelo casal; o que estiver nele fica protegido pelo PIN.

### Decisões de produto

1. Dois perfis fixos (administrador e dependente), sem permissões configuráveis item a item. Motivo: família de 4 pessoas; as regras foram definidas no brainstorm e não precisam de flexibilidade.
2. O que o dependente não pode ver **nem chega ao aparelho dele** (não basta esconder na tela). Motivo: celular de adolescente é emprestado, perdido e mexido; esconder na tela não protege.
3. PIN obrigatório para todos os perfis. Motivo: dados financeiros e celulares dos filhos.
4. Vincular um aparelho exige um administrador e a rede de casa. Motivo: o servidor só é acessível em casa (sincronização pelo Wi-Fi de casa) e ninguém de fora deve conseguir entrar na família.
5. Usuários são desativados, não apagados. Motivo: o histórico (quem lançou o quê, contas dos filhos) continua valendo.

### Fora do escopo

- Mais de uma família no mesmo servidor, cadastro aberto ou convite por e-mail. Sistema de uma família só.
- Permissões configuráveis por conta ou por funcionalidade.
- Entrar com digital ou reconhecimento facial no lugar do PIN.
- Acesso fora de casa ao servidor (decidido no brainstorm).
- Separação de finanças entre os dois administradores: os dois veem e podem alterar tudo.

## 3. Funcionalidades

### US01: Criar a família no primeiro uso

Como administrador, quero configurar a família na primeira vez que instalo o servidor, para começar a usar o app.

**Rules:**
- No primeiro uso, o servidor pede o nome da família e cria o **primeiro administrador** (nome e PIN).
- Enquanto a família não existe, nenhum aparelho consegue se vincular.
- A configuração inicial só pode ser feita **no próprio computador do servidor**.
- O servidor mostra, ao final, as instruções para vincular o primeiro celular (US03).

**Edge cases:**
- Configuração interrompida no meio (computador desligado) → na próxima vez recomeça do zero, sem família pela metade.
- Tentativa de criar uma segunda família no mesmo servidor → não é permitida; o servidor tem uma família só.
- Servidor reinstalado num computador novo → a família é recuperada da cópia de segurança (PRD 011), não recriada.

### US02: Cadastrar e gerenciar os membros da família

Como administrador, quero cadastrar o outro administrador e os filhos, para que cada um tenha o próprio acesso.

**Rules:**
- Cada membro tem nome, perfil (administrador ou dependente) e PIN.
- Pode haver **até 2 administradores** e qualquer número de dependentes.
- Para dependente, o cadastro diz quais contas e cartões adicionais são dele (ligação feita nos cadastros de contas e cartões, PRD 002).
- Para dependente e para o outro administrador, o cadastro também registra **CPF e data de nascimento**, usados no apoio ao IR (PRD 010).
- Um administrador pode **desativar** um membro: ele perde o acesso em todos os aparelhos; o histórico e as contas continuam.
- Um administrador pode reativar um membro desativado.
- Um administrador pode mudar o perfil de um membro (ex.: filho que vira adulto).

**Edge cases:**
- Tentativa de desativar o último administrador ativo → bloqueada; a família não pode ficar sem administrador.
- Dois membros com o mesmo nome → permitido, mas o app pede um apelido para distinguir na tela.
- Dependente desativado com contas próprias → as contas continuam visíveis ao casal, marcadas como de membro desativado.

### US03: Vincular um aparelho à família

Como membro da família, quero vincular meu celular ou computador, para usar o app nele.

**Rules:**
- O aparelho precisa estar no **Wi-Fi de casa**, com o servidor ligado.
- O vínculo é autorizado por um **administrador**: o app mostra um código no aparelho novo e o administrador confirma no servidor ou no próprio aparelho dele.
- Cada aparelho pertence a **um membro**. Exceção: um computador pode ter **os dois administradores**, cada um entrando com o próprio PIN.
- Um dependente não pode ser vinculado num aparelho que também tenha um administrador. Motivo: o aparelho guardaria dados da família.
- Depois de vinculado, o aparelho recebe os dados que o membro pode ver (PRD 011) e passa a funcionar também fora de casa.
- O casal vê a lista de aparelhos vinculados: membro, nome do aparelho, última sincronização.

**Edge cases:**
- Código de vínculo expirado ou digitado errado → vínculo recusado; gerar código novo. O código vale **10 minutos**.
- Aparelho fora do Wi-Fi de casa ou servidor desligado → o app explica que o vínculo só é feito em casa, com o servidor ligado.
- Aparelho já vinculado a outro membro → precisa ser desvinculado antes (os dados locais do membro anterior são apagados).

### US04: Entrar com PIN

Como membro da família, quero que o app peça meu PIN ao abrir, para que ninguém mais veja meus dados no meu aparelho.

**Rules:**
- O PIN tem **6 dígitos**.
- O app pede o PIN ao abrir e de novo depois de **5 minutos** sem uso ou em segundo plano.
- O PIN funciona também fora de casa, sem o servidor (o app funciona offline).
- O membro pode trocar o próprio PIN informando o atual.
- Após **5 PINs errados seguidos**, o app espera um tempo crescente antes de aceitar nova tentativa (1 min, 5 min, 30 min…).

**Edge cases:**
- Membro esqueceu o PIN → um administrador cria um PIN provisório para ele, na rede de casa; no primeiro uso o membro troca.
- Administrador esqueceu o PIN → o outro administrador redefine. Os dois esqueceram → recuperação no computador do servidor, com instruções (PRD 011).
- PIN trocado num aparelho → vale nos outros aparelhos do mesmo membro depois da próxima sincronização.
- PIN óbvio (000000, 123456) → recusado.

### US05: Ver só o que o perfil permite

Como filho, quero ver minha conta e as compras do meu cartão adicional, para acompanhar meus gastos; como administrador, quero ver tudo, para administrar as finanças da família.

**Rules:**
- **Administrador** vê: todas as contas, cartões, lançamentos, orçamentos, relatórios e investimentos da família, e as contas dos filhos (saldo e movimentação, separadas dos relatórios da família).
- **Dependente** vê:
  - a própria conta individual e as contas próprias que vier a ter;
  - todas as compras do próprio cartão adicional, com o consumo do ciclo contra o limite dele (PRD 009);
  - as categorias da família, para categorizar as próprias compras.
- **Dependente não vê**: contas, cartões e lançamentos da família; compras do titular e do outro filho; total da fatura do cartão; limite total do cartão; orçamento, relatórios, patrimônio e investimentos da família; dados de IR.
- O que o dependente não pode ver não é copiado para o aparelho dele.

**Edge cases:**
- Compra no cartão adicional do filho importada pelo casal → aparece para o filho depois da próxima sincronização dele.
- Compra lançada no cartão errado (titular em vez do adicional do filho) e depois corrigida pelo casal → passa a aparecer para o filho; se for o contrário, some do aparelho dele na próxima sincronização.
- Conta que passa da família para um filho (ou o contrário) → a visibilidade muda na próxima sincronização, nos dois sentidos.

### US06: Ações permitidas ao dependente

Como filho, quero registrar, anotar e categorizar as minhas compras, para que meus pais saibam com o que gastei.

**Rules:**
- O dependente pode **registrar** uma compra no próprio cartão adicional ou um movimento na própria conta (conciliado depois com a importação, PRD 004).
- Pode **categorizar** e **anotar** (texto livre) qualquer lançamento do próprio cartão adicional e da própria conta, inclusive os importados pelo casal.
- Não pode **excluir** lançamentos importados nem lançamentos feitos pelo casal.
- Pode excluir ou corrigir o que ele mesmo registrou, enquanto não foi conciliado com a importação.
- Não pode criar nem editar categorias, contas ou cartões.
- O casal pode corrigir qualquer categorização feita pelo filho.

**Edge cases:**
- Filho e casal mudam a categoria do mesmo lançamento enquanto estão fora de casa → vale a alteração do administrador (regra geral de conflitos no PRD 011).
- Filho tenta registrar compra em cartão que não é dele → não é possível; só o próprio cartão adicional aparece.
- Filho desativado → as anotações e categorizações dele continuam no histórico.

### US07: Revogar um aparelho perdido ou trocado

Como administrador, quero revogar um aparelho, para que ele não receba mais dados da família.

**Rules:**
- Qualquer administrador revoga qualquer aparelho da lista (US03), inclusive o próprio.
- Aparelho revogado não sincroniza mais: o servidor recusa.
- Se um aparelho revogado voltar a se conectar ao servidor, o app **apaga os dados locais** e volta para a tela de vínculo.
- Os dados no aparelho perdido continuam protegidos pelo PIN e pela criptografia (TRD); a revogação não apaga nada à distância, porque o servidor só alcança aparelhos dentro de casa.
- Lançamentos feitos no aparelho e ainda não sincronizados no momento da perda são perdidos. O app mostra quantos lançamentos ainda não foram sincronizados, para lembrar o membro de sincronizar.

**Edge cases:**
- Celular perdido é encontrado depois de revogado → precisa ser vinculado de novo (US03); o que não tinha sido sincronizado se perde.
- Administrador revoga o único aparelho do outro administrador por engano → o outro vincula de novo (US03).
- Aparelho revogado ainda com dados não sincronizados → o app avisa antes de apagar que esses dados serão perdidos.

### US08: Saber quem lançou ou alterou

Como administrador, quero ver quem criou e quem alterou cada lançamento, para entender diferenças entre o que cada um registrou.

**Rules:**
- Todo lançamento guarda quem o criou, quando, por qual meio (digitado, importado, migrado) e quem fez a última alteração.
- O casal vê essa informação no detalhe do lançamento.
- O dependente vê essa informação só nos lançamentos que ele pode ver.

**Edge cases:**
- Lançamento migrado do Minhas Economias → autor "migração", sem membro.
- Membro desativado → o nome continua aparecendo no histórico.

## 4. Fluxo de Negócio

```
Aparelho novo instalado
   │
   ▼
Está no Wi-Fi de casa e o servidor está ligado?
   ├── não ──▶ "Vincule em casa, com o servidor ligado"
   └── sim ──▶ Administrador autoriza o vínculo?
                 ├── não ──▶ Vínculo recusado
                 └── sim ──▶ Membro escolhido é dependente?
                               ├── sim ──▶ Recebe só a conta dele e o cartão adicional dele
                               └── não ──▶ Recebe todos os dados da família
                                              │
                                              ▼
                                   App pede PIN a cada abertura
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Um aparelho só se vincula com autorização de um administrador, no Wi-Fi de casa | ninguém de fora entra na família | tentar vincular sem autorização e fora da rede de casa: ambos recusados |
| O aparelho do dependente não contém dados da família | celular de adolescente é emprestado, perdido e mexido | vincular um filho, examinar os dados guardados no aparelho: só a conta e o cartão adicional dele |
| Dependente vê todas as compras do próprio cartão adicional e a própria conta | combinado com o casal no brainstorm | importar uma fatura com compras do titular e dos dois filhos: cada filho vê só as dele, todas elas |
| App pede PIN ao abrir e após 5 min sem uso | dados sensíveis em celular | abrir o app; deixar parado 5 min; voltar: PIN pedido nas duas vezes |
| Após 5 PINs errados, nova tentativa só depois de espera crescente | impedir adivinhar o PIN | errar 5 vezes: o app bloqueia por 1 min |
| Aparelho revogado não sincroniza e apaga os dados locais ao se reconectar | celular perdido não continua recebendo dados | revogar, reconectar em casa: sincronização recusada e dados apagados |
| Família nunca fica sem administrador | sem administrador ninguém gerencia | tentar desativar o último administrador: bloqueado |
| Todo lançamento mostra quem criou e quem alterou por último | casal entende diferenças entre os registros | criar num aparelho, alterar no outro: o detalhe mostra os dois |
| O app funciona sem o servidor depois de vinculado (PIN incluído) | servidor nem sempre ligado | desligar o servidor, sair de casa: entrar com PIN e lançar funcionam |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Membros com aparelho vinculado e usando o app | 0 (app não existe) | 4 (casal e filhos) | 1 mês após a entrega | 2 (o casal) | casal |
| Compras do cartão adicional categorizadas pelo próprio filho | 0 (filhos não participam hoje) | metade das compras do mês | 3 meses após a entrega | alguma participação de cada filho | casal |
| Incidentes de dado da família visto por um filho | não se aplica | 0 | contínuo | 0 | casal |

## 6. Milestones

### Milestone 1: Casal usando o app com acesso protegido

**Por que é um marco:** o casal sai da planilha e usa o app nos próprios aparelhos, com PIN, vendo os mesmos dados.

**Funcionalidades:** US01, US02, US03, US04, US08

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Um aparelho só se vincula com autorização de um administrador, no Wi-Fi de casa
- [ ] App pede PIN ao abrir e após 5 min sem uso
- [ ] Após 5 PINs errados, nova tentativa só depois de espera crescente
- [ ] Família nunca fica sem administrador
- [ ] Todo lançamento mostra quem criou e quem alterou por último
- [ ] O app funciona sem o servidor depois de vinculado (PIN incluído)

**Aprovador:** casal

### Milestone 2: Filhos no app, vendo só o que é deles

**Por que é um marco:** os filhos passam a participar (registrar e categorizar as próprias compras) sem acesso às finanças da família.

**Funcionalidades:** US05, US06

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] O aparelho do dependente não contém dados da família
- [ ] Dependente vê todas as compras do próprio cartão adicional e a própria conta

**Aprovador:** casal

### Milestone 3: Aparelho perdido sob controle

**Por que é um marco:** perder um celular deixa de ser um risco para os dados da família.

**Funcionalidades:** US07

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Aparelho revogado não sincroniza e apaga os dados locais ao se reconectar

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Filhos não aderem ao app | Médio | uso dos filhos é desejável, não obrigatório: o app funciona sem eles (o casal categoriza) | Monitorando |
| Esquecer o PIN e ficar sem acesso | Médio | redefinição pelo outro administrador; recuperação documentada no servidor (PRD 011) | Pendente |
| Celular perdido com lançamentos não sincronizados | Baixo | aviso de lançamentos pendentes de sincronização (US07) | Pendente |
| PIN de 6 dígitos fraco contra quem tem o aparelho e tempo | Médio | espera crescente após erros; criptografia dos dados no aparelho (TRD) | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 002 (contas e cartões): dono de cada conta e cartão adicional | Interna | a escrever | Milestone 2 (visibilidade do dependente) |
| PRD 011 (sincronização e operação): sincronização filtrada, conflitos, recuperação | Interna | a escrever | Milestones 1, 2 e 3 |
| TRD: criptografia no aparelho, forma do vínculo | Interna | a escrever | Milestones 1 e 3 |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — decisões sobre perfis, permissões dos filhos, PIN, sincronização em casa e aparelho perdido.

## 9. Registro de Decisões

- **2026-11-02:** Dois perfis fixos (administrador, dependente). Motivo: família pequena, regras fechadas no brainstorm.
- **2026-11-02:** Casal vê as contas dos filhos; cada filho vê só a própria conta e todas as compras do próprio cartão adicional. Motivo: decisão do casal na rodada 5 do brainstorm.
- **2026-11-02:** O que o dependente não pode ver não chega ao aparelho dele. Motivo: validado na consolidação do brainstorm (esconder na tela não protege).
- **2026-11-02:** PIN ao entrar. Motivo: decisão do casal na rodada 6.
- **2026-11-02:** Aparelho perdido: revogação pelo casal e dados protegidos por PIN e criptografia; sem apagamento remoto. Motivo: validado na consolidação; o servidor só alcança aparelhos em casa.
- **2026-11-02:** `depends_on: []`. Critério: é o primeiro PRD; os outros dependem dele. As ligações com os PRDs 002 e 011 estão na seção 7 como dependências de entrega, porque ainda não foram escritos.
- **2026-11-03:** As 23 premissas do rascunho foram aceitas pelo casal como estão; marcas removidas. Status mantido em `rascunho` até a revisão final de todos os PRDs (1ª revisão aprovada).
