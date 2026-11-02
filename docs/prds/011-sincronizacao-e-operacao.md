---
prd_number: "011"
status: rascunho
revisao: "1ª revisão aprovada pelo casal em 2026-11-03; falta a revisão final"
priority: crítica
created: 2026-11-03
issue: ""
depends_on: ["001"]
references:
  - docs/brainstorm-controle-financeiro.md
  - docs/prds/001-usuarios-perfis-e-acesso.md
---

# PRD 011: Sincronização e operação

## 1. Contexto

- **Produto/área**: controle financeiro da família. Base: `docs/brainstorm-controle-financeiro.md`.
- **Estado atual**: no brainstorm ficou decidido que os dados ficam só com a família: cada celular (Android) e computador (Windows) guarda a sua cópia e funciona sem internet; um **servidor num computador de casa**, que **nem sempre está ligado**, é o ponto de encontro; a sincronização acontece **só pelo Wi-Fi de casa**; **nenhum custo** adicional; **cópia de segurança no OneDrive ou em disco externo**. Quem instala e mantém o servidor é o marido, com alguma familiaridade técnica, e o desejo é **automatizar tudo o que der** e, no resto, ter **instruções claras**.
- **Problema**: sem um serviço na nuvem, a família vira o próprio "provedor": precisa que os aparelhos se encontrem sozinhos, que edições feitas em dois lugares não se percam, que um computador quebrado não leve 9 anos de histórico junto, e que nada disso exija conhecimento técnico no dia a dia.

> Este PRD descreve o comportamento visto pela família. Como sincronizar, criptografar, descobrir o servidor na rede e distribuir atualizações é decisão técnica: `docs/trd.md` e ADRs.

## 2. Solução Proposta

### Visão de produto

- O app funciona **sempre**, com ou sem servidor; ao chegar em casa, o aparelho **sincroniza sozinho**.
- Cada aparelho mostra **quando sincronizou** e **quantos lançamentos ainda não sincronizou**.
- Edições conflitantes feitas fora de casa são resolvidas por uma **regra simples e previsível**, e as que precisam de decisão aparecem para o casal.
- O servidor é **instalado por um assistente** no Windows e roda **em segundo plano**, iniciando com o computador.
- **Cópia de segurança automática**, criptografada antes de ir para o OneDrive ou para o disco externo, e uma **restauração guiada** para quando o computador quebrar.
- Um **painel** mostra a saúde do sistema: aparelhos, última sincronização de cada um, última cópia de segurança, versão.

### Decisões de produto

1. Offline primeiro: nenhum uso do app depende do servidor estar ligado. Motivo: servidor nem sempre ligado (brainstorm, rodada 2).
2. Sincronização só no Wi-Fi de casa, sem acesso remoto. Motivo: privacidade e custo zero (rodada 3).
3. Cópia de segurança criptografada antes de sair do computador. Motivo: validado na consolidação; o OneDrive não pode ler os dados.
4. A **chave de recuperação** da cópia de segurança fica só com o casal. Sem ela, ninguém (nem o próprio casal) consegue abrir a cópia. Motivo: consequência da decisão 3.
5. Tudo o que der é automático; o resto tem passo a passo escrito. Motivo: pedido do casal (rodada 6).

### Fora do escopo

- Acesso ao servidor fora de casa, por VPN ou túnel (decidido no brainstorm).
- Sincronização entre aparelhos sem o servidor (celular com celular).
- Mais de um servidor ou servidor sempre ligado na nuvem (custo zero, privacidade).
- Loja de aplicativos: o app Android é instalado direto (APK), sem Play Store (decisão técnica no TRD).

## 3. Funcionalidades

### US01: Usar o app sem o servidor

Como membro da família, quero lançar e consultar tudo fora de casa, para usar o app na hora em que gasto.

**Rules:**
- Todas as funções do app funcionam no aparelho sem o servidor, com os dados que o aparelho tem (filtrados pelo perfil, PRD 001).
- Exceções, que exigem o servidor: vincular aparelho, redefinir PIN de outro membro, revogar aparelho (PRD 001) e restaurar cópia de segurança (US07).
- O que foi feito offline fica guardado no aparelho até a próxima sincronização.

**Edge cases:**
- Aparelho sem sincronizar há **7 dias** → aviso discreto: "sincronize em casa; X lançamentos só estão neste aparelho" *(premissa — confirme ou corrija o prazo)*.
- Aparelho com pouco espaço → o app avisa antes de ficar sem espaço; nunca descarta lançamentos não sincronizados.

### US02: Sincronizar automaticamente em casa

Como membro da família, quero que meu aparelho sincronize sozinho quando chego em casa, para não precisar lembrar.

**Rules:**
- Com o aparelho no Wi-Fi de casa e o servidor ligado, a sincronização acontece sozinha: ao abrir o app e periodicamente com o app aberto (sincronizar com o app fechado depende de permissões do Android, ver TRD).
- Botão **sincronizar agora**.
- Cada aparelho mostra: última sincronização, lançamentos pendentes de envio, e se o servidor está acessível agora.
- Cada aparelho recebe só o que o membro pode ver (PRD 001, US05).
- A sincronização não interrompe o uso: o membro pode continuar lançando.

**Edge cases:**
- Servidor desligado ou fora do Wi-Fi de casa → nenhum erro incômodo; o app só indica "servidor não encontrado" no status.
- Sincronização interrompida no meio (saiu de casa) → nada se perde nem se duplica; continua na próxima vez.
- Aparelho revogado → servidor recusa e o app apaga os dados locais (PRD 001, US07).
- Primeira sincronização de um aparelho novo com 9 anos de histórico → mostra o progresso; o app pode ser usado com os dados já recebidos.

### US03: Resolver conflitos de edição

Como administrador, quero que edições feitas em dois aparelhos fora de casa se juntem sem perder nada, para que o casal possa lançar e corrigir cada um no seu celular.

**Rules:**
- Lançamentos **novos** feitos em aparelhos diferentes nunca conflitam: todos entram.
- O mesmo lançamento editado em dois aparelhos: as alterações em **campos diferentes** se juntam (ex.: um mudou a categoria, outro a observação).
- O **mesmo campo** alterado nos dois: vale a alteração do **administrador** sobre a do dependente (PRD 001, US06); entre dois administradores, vale a **mais recente**.
- Lançamento **excluído** num aparelho e **editado** no outro: prevalece a edição (não se perde dado), e o lançamento vai para a lista de conflitos para o casal decidir.
- Os casos resolvidos pela regra ficam numa **lista de conflitos**, onde o casal vê o que cada um fez e pode trocar a escolha.

**Edge cases:**
- Relógio errado num aparelho (data/hora atrasada) → a regra "mais recente" pode errar; a lista de conflitos permite corrigir (o TRD deve reduzir a dependência do relógio).
- Categoria juntada num aparelho enquanto o outro lançava nela → os lançamentos novos vão para a categoria que ficou.
- Mesma compra digitada pelos dois → não é conflito de sincronização: é duplicidade, tratada pela conciliação (PRD 004), que aponta prováveis duplicados.

### US04: Instalar o servidor no computador de casa

Como o administrador que cuida do servidor, quero instalá-lo com um assistente, para não precisar de conhecimento técnico.

**Rules:**
- Instalador para Windows com assistente em português: instala, configura para iniciar com o computador e rodar em segundo plano, e leva à criação da família (PRD 001, US01).
- O assistente cuida das permissões do Windows necessárias (ex.: liberar o acesso na rede de casa) explicando cada pedido.
- No fim, o assistente gera a **chave de recuperação** (US06) e pede para guardá-la (imprimir ou anotar), e configura o destino da cópia de segurança.
- O servidor funciona sempre que o computador estiver ligado, **mesmo sem ninguém logado** no Windows.
- Guia de instalação passo a passo, com imagens, em `docs/` (entregável deste PRD).

**Edge cases:**
- Computador entra em suspensão → o servidor fica indisponível enquanto o computador dorme; os aparelhos sincronizam quando ele acordar. O guia explica como ajustar a suspensão, se a família quiser.
- Computador usado por outras pessoas → o servidor não aparece no dia a dia; só um ícone discreto com o status.
- Antivírus ou firewall bloqueia → o assistente detecta que os aparelhos não conseguem chegar ao servidor e mostra o que fazer.

### US05: Atualizar o app e o servidor

Como o administrador que cuida do servidor, quero que as atualizações cheguem com o mínimo de trabalho, para manter tudo funcionando.

**Rules:**
- O servidor avisa quando há versão nova e se atualiza com um clique, sem perder dados, fazendo antes uma cópia de segurança.
- Os aparelhos recebem a versão nova do app **pelo servidor de casa** ao sincronizar, e o membro confirma a instalação (como o app não está em loja, a forma de distribuir é decisão técnica do TRD).
- Aparelho com versão antiga continua sincronizando enquanto as versões forem compatíveis; quando não forem, o app pede para atualizar antes de sincronizar.
- Instalação por cima, sem desinstalar: dados do aparelho preservados.

**Edge cases:**
- Atualização do servidor falha → volta à versão anterior automaticamente; os dados ficam como estavam.
- Aparelho fica meses sem atualizar → atualização direta para a versão atual.

### US06: Cópia de segurança automática

Como administrador, quero que os dados sejam copiados automaticamente para o OneDrive ou para um disco externo, para não perder o histórico se o computador quebrar.

**Rules:**
- Destino escolhido na instalação e alterável: **pasta do OneDrive** do computador ou **disco externo**; pode haver os dois.
- A cópia é feita **automaticamente** enquanto o servidor está ligado: uma por dia e logo após mudanças grandes (migração, importação).
- A cópia é **criptografada no computador** antes de ir para o destino, com a **chave de recuperação** do casal. Sem a chave, a cópia não pode ser aberta.
- Guarda as últimas **30 cópias diárias** e **12 mensais**.
- O painel (US08) e os aparelhos dos administradores mostram a data da última cópia bem-sucedida.

**Edge cases:**
- Disco externo desconectado ou OneDrive cheio → a cópia falha, o servidor tenta de novo e avisa os administradores no próximo uso.
- Nenhuma cópia há **7 dias** → aviso destacado aos administradores.
- Chave de recuperação perdida → as cópias existentes ficam inúteis; o assistente permite gerar uma chave nova (com o servidor funcionando), e as próximas cópias usam a nova. O guia explica guardar a chave em dois lugares.

### US07: Restaurar depois de perder o computador do servidor

Como o administrador que cuida do servidor, quero reinstalar o servidor num computador novo e recuperar tudo, para que a perda do computador não seja a perda dos dados.

**Rules:**
- No assistente de instalação, opção **restaurar de uma cópia de segurança**: escolhe a cópia (OneDrive ou disco), informa a chave de recuperação, e o servidor volta com a família, membros, aparelhos e todos os dados.
- Depois de restaurar, os aparelhos já vinculados reconhecem o servidor novo e sincronizam; o que tinham de mais novo que a cópia é enviado e se junta (US03).
- **Sem cópia de segurança**: um aparelho de **administrador** sincronizado tem os dados da família inteira e pode reconstruir o servidor.
- Recuperação quando os dois administradores esqueceram o PIN (PRD 001, US04): no computador do servidor, com a chave de recuperação.
- Guia de recuperação passo a passo em `docs/` (entregável deste PRD), incluindo um **teste de restauração** recomendado uma vez por ano.

**Edge cases:**
- Cópia mais recente corrompida → usa a anterior; o assistente lista as cópias disponíveis com data.
- Restauração de cópia antiga (ex.: 1 semana) → lançamentos dessa semana voltam pelos aparelhos que os tinham; o que só existia no servidor se perde, e o assistente avisa.

### US08: Ver a saúde do sistema

Como administrador, quero um painel com o estado do servidor, dos aparelhos e das cópias, para saber se está tudo bem sem precisar entender os detalhes.

**Rules:**
- No computador do servidor e nos aparelhos dos administradores: versão do servidor, aparelhos vinculados com a última sincronização de cada um, última cópia de segurança e destino, espaço em disco, avisos pendentes.
- Indicador geral simples: **tudo certo**, **atenção** (ex.: cópia atrasada, aparelho sem sincronizar há mais de 7 dias) ou **problema** (ex.: cópias falhando).
- Cada aviso diz o que fazer, em linguagem simples.

**Edge cases:**
- Painel aberto num aparelho fora de casa → mostra o estado da última sincronização, com a data.
- Aparelho de dependente → não vê o painel.

### US09: Exportar todos os dados

Como administrador, quero exportar todos os dados da família num formato aberto, para nunca ficar preso ao app, como aconteceu com o Minhas Economias.

**Rules:**
- Exportação completa (contas, cartões, lançamentos, categorias, orçamentos, investimentos) em formato aberto e legível por planilha *(premissa — confirme ou corrija os formatos)*.
- Feita no computador, salva onde o administrador escolher; nada é enviado para fora.

**Edge cases:**
- Exportação de 9 anos → completa, sem cortar.
- Exportação no aparelho do dependente → não disponível.

## 4. Fluxo de Negócio

```
Fora de casa                         Em casa (Wi-Fi), servidor ligado
   lança e consulta no aparelho  ──▶    sincroniza sozinho
   (pendentes ficam no aparelho)          │
                                          ├── conflito? ─▶ regra (campo, perfil, mais recente)
                                          │                  └─▶ lista de conflitos para o casal
                                          ▼
                                   servidor atualizado ──▶ cópia diária criptografada
                                                              ├─▶ OneDrive
                                                              └─▶ disco externo

Computador do servidor quebrou
   novo computador ──▶ instalador: restaurar ──▶ cópia + chave de recuperação
                                     └── sem cópia ──▶ reconstruir a partir de um aparelho de administrador
```

## 5. Critérios de Aceite

### 5a. Critérios de aceite da feature

| Critério | Razão de negócio | Como verificar (observável) |
|----------|------------------|-----------------------------|
| Todas as funções do dia a dia funcionam com o servidor desligado | servidor nem sempre ligado | desligar o servidor; lançar, consultar relatórios e orçamento: tudo funciona |
| Ao chegar em casa, o aparelho sincroniza sem ação do usuário | automação pedida pelo casal | lançar fora, entrar no Wi-Fi e abrir o app: lançamento aparece no outro aparelho após ele sincronizar |
| Sincronização interrompida não perde nem duplica lançamentos | confiança nos números | cortar o Wi-Fi no meio: depois de completar, contagem de lançamentos correta |
| Edições em campos diferentes do mesmo lançamento se juntam; mesmo campo segue a regra e vai para a lista de conflitos | o casal edita cada um no seu celular | editar categoria num e observação no outro: as duas ficam; editar a categoria nos dois: regra aplicada e conflito listado |
| Instalação do servidor sem passos técnicos além do assistente | quem cuida tem alguma familiaridade técnica | instalar num Windows limpo seguindo só o assistente e o guia |
| Cópia de segurança diária e criptografada no destino escolhido | não perder dados; OneDrive não lê | verificar o arquivo do dia no OneDrive; tentar abrir sem a chave: impossível |
| Restaurar num computador novo recupera família, membros, aparelhos e dados | computador quebrado não leva o histórico | restaurar a cópia num segundo computador: aparelhos sincronizam e os dados batem |
| Painel mostra última sincronização de cada aparelho e última cópia | saber se está tudo bem | abrir o painel: datas corretas; parar as cópias 7 dias: estado "atenção" |
| Atualizar o servidor e o app não perde dados e não exige desinstalar | manutenção sem risco | atualizar com lançamentos pendentes em um aparelho: nada se perde |
| Exportação completa abre numa planilha com todos os lançamentos | não ficar preso ao app | exportar e comparar a contagem de lançamentos com a do app |

### 5b. Métricas de sucesso

| Métrica | Baseline (fonte) | Meta | Prazo | Mín. aceitável | Responsável |
|---------|-------------------|------|-------|-----------------|-------------|
| Dias sem cópia de segurança bem-sucedida | não se aplica | 0 a 1 (dias com o servidor ligado) | contínuo | 7 | marido (servidor) |
| Lançamentos perdidos por falha de sincronização | não se aplica | 0 | contínuo | 0 | casal |
| Teste de restauração feito | não se aplica | 1 por ano | anual | 1 antes do primeiro ano completo | marido (servidor) |
| Intervenções técnicas no servidor por mês (fora atualizações) | não se aplica | 0 | a partir do 2º mês | 1 | marido (servidor) |

## 6. Milestones

### Milestone 1: Família sincronizada

**Por que é um marco:** os aparelhos da família compartilham os mesmos dados sem nuvem, e cada um funciona fora de casa.

**Funcionalidades:** US01, US02, US03, US04

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Todas as funções do dia a dia funcionam com o servidor desligado
- [ ] Ao chegar em casa, o aparelho sincroniza sem ação do usuário
- [ ] Sincronização interrompida não perde nem duplica lançamentos
- [ ] Edições em campos diferentes do mesmo lançamento se juntam; mesmo campo segue a regra e vai para a lista de conflitos
- [ ] Instalação do servidor sem passos técnicos além do assistente

**Aprovador:** casal

### Milestone 2: Dados protegidos contra perda

**Por que é um marco:** 9 anos de histórico deixam de depender de um único computador.

**Funcionalidades:** US06, US07, US08

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Cópia de segurança diária e criptografada no destino escolhido
- [ ] Restaurar num computador novo recupera família, membros, aparelhos e dados
- [ ] Painel mostra última sincronização de cada aparelho e última cópia

**Aprovador:** casal

### Milestone 3: Manutenção sem esforço

**Por que é um marco:** atualizar o sistema e levar os dados para fora dele passam a ser tarefas de um clique.

**Funcionalidades:** US05, US09

**Checklist de aceite** (marcado pelo Aprovador após a implementação):
- [ ] Atualizar o servidor e o app não perde dados e não exige desinstalar
- [ ] Exportação completa abre numa planilha com todos os lançamentos

**Aprovador:** casal

## 7. Riscos e Dependências

| Risco | Impacto | Mitigação | Status |
|-------|---------|-----------|--------|
| Chave de recuperação perdida torna as cópias inúteis | Alto | guardar em dois lugares (guia); gerar chave nova enquanto o servidor funciona; reconstrução a partir de aparelho de administrador | Pendente |
| Computador do servidor desligado por longos períodos: aparelhos acumulam pendências e não há cópia | Médio | avisos de 7 dias (US01, US06); aparelhos guardam tudo até sincronizar | Pendente |
| Android restringe sincronização em segundo plano e com o app fechado | Médio | sincroniza ao abrir o app em casa; detalhe no TRD | Pendente |
| Conflitos frequentes confundem o casal | Baixo | lançamentos novos nunca conflitam; regra simples; lista de conflitos | Pendente |
| Atualização quebrar a compatibilidade entre aparelhos e servidor | Médio | cópia antes de atualizar; volta automática; aviso para atualizar o app (US05) | Pendente |

**Dependências:**

| Dependência | Tipo | Status | Impacto se bloqueado |
|-------------|------|--------|----------------------|
| PRD 001: perfis, vínculo de aparelho, revogação, PIN | Interna | rascunho (1ª revisão aprovada) | todos |
| TRD: mecanismo de sincronização, criptografia, descoberta do servidor na rede, distribuição do app e das atualizações | Interna | a escrever | todos |
| Guias de instalação e de recuperação | Interna | a escrever junto com a implementação | Milestones 1 e 2 |

## 8. Referências

- `docs/brainstorm-controle-financeiro.md` — servidor nem sempre ligado, sincronização em casa, custo zero, OneDrive ou disco externo, operação pelo marido.
- `docs/prds/001-usuarios-perfis-e-acesso.md` — vínculo de aparelho, filtro por perfil, revogação, PIN.

## 9. Registro de Decisões

- **2026-11-03:** Offline primeiro; sincronização automática só no Wi-Fi de casa. Motivo: decisões do brainstorm (rodadas 2 e 3).
- **2026-11-03:** Cópia de segurança criptografada com chave de recuperação só do casal. Motivo: validado na consolidação; o OneDrive não pode ler os dados.
- **2026-11-03:** Regra de conflito: campos diferentes se juntam; mesmo campo, administrador sobre dependente, e entre administradores o mais recente; exclusão x edição, prevalece a edição. Motivo: não perder dado e ser previsível (premissas a confirmar).
- **2026-11-03:** Exportação completa em formato aberto. Motivo: a família ficou presa ao Minhas Economias quando ele mudou as regras.
- **2026-11-03:** `depends_on: ["001"]`. Critério: vínculo de aparelho, filtro por perfil e revogação vêm do PRD 001. Os demais PRDs dependem deste para funcionar em vários aparelhos, mas este não depende do conteúdo deles.
- **2026-11-03:** 1ª revisão do casal: as 20 premissas foram aceitas como estão (inclusive a regra de conflitos, a chave de recuperação só com o casal, cópias diárias com 30 diárias e 12 mensais, atualização do app pelo servidor de casa, avisos de 7 dias). Marcas removidas; as notas que apontam para o TRD foram mantidas. Status mantido em `rascunho` até a revisão final.
