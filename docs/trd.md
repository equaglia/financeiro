# TRD — Technical Requirements Document

> Documento técnico global do projeto. Criado e atualizado via skill `escrever-trd`.
> Carregado como contexto global na implementação. Granularidade baixa: cobre o que é global e estável. Regras finas ficam em ADRs (`docs/adrs/`).
>
> Criado em 2026-11-03 a partir de `docs/brainstorm-controle-financeiro.md` e dos PRDs 001–003 e 005–011 (o PRD 004, importação, ainda não foi escrito). Projeto ainda **sem código**. Itens de versão trazem a fonte e a data da verificação; "a verificar" significa que a fonte não é oficial ou não confirmou o número.

---

## Stack

| Dimensão | Valor |
|---|---|
| Linguagem principal | Kotlin 2.4.21 (mesma versão que já compila no `contagem_tm` neste ambiente; ADR 002) |
| Runtime / plataforma | Android (app do celular); JVM no Windows 10 ou mais novo (app do computador e servidor). Compose para desktop exige JDK 11+, e o empacotamento nativo JDK 17+ ([kotlinlang.org](https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html), consultado em 2026-11-03) |
| Framework principal | Compose Multiplatform 1.12.2 (baseado em Jetpack Compose 1.12.1; [kotlinlang.org](https://kotlinlang.org/docs/multiplatform/compose-compatibility-and-versioning.html), 2026-11-03) para Android e Windows; Ktor 3.4.0 no servidor (a verificar: [blog da JetBrains](https://blogs.jetbrains.com/ktor/category/releases/), 2026-11-03, sem data da versão) |
| Banco de dados | SQLite em cada aparelho e no servidor, com SQLDelight 2.3.2 (a verificar: [dev.co](https://dev.co/databases/open-source/sqldelight), 2026-11-03). Criptografia do banco: ADR 004 |
| Ferramentas de build | Gradle com Android Gradle Plugin 9.4.0 (mesma versão do `contagem_tm`); empacotamento do Windows em .zip montado na nuvem (ADR 005) |
| Gerenciador de pacotes | Gradle (catálogo de versões `gradle/libs.versions.toml`) |

Outras peças já escolhidas:

| Peça | Escolha | Fonte |
|---|---|---|
| Serviço do Windows | WinSW (MIT; linha 2.x estável, 3.x em pré-lançamento) | [github.com/winsw/winsw](https://github.com/winsw/winsw), 2026-11-03 (versão exata a verificar) |
| Sincronização em segundo plano no Android | WorkManager, tarefa periódica (mínimo 15 min) com condição de rede sem cobrança (Wi-Fi) | [developer.android.com](https://developer.android.com/reference/androidx/work/PeriodicWorkRequest), 2026-11-03 |
| Criptografia do SQLite no Android | a decidir (ADR 004): driver SQLCipher comunitário `io.github.yet300:sqlcipher-driver` 0.1.x ou criptografia por campo | [mvnrepository](https://mvnrepository.com/artifact/io.github.yet300/sqlcipher-driver), 2026-11-03 |

---

## Arquitetura

### Padrão arquitetural

**Local-first com servidor doméstico** (ADR 001), organizado como **monólito modular em Kotlin Multiplatform**: domínio e regras de negócio num módulo comum, usado pelo app Android, pelo app Windows e pelo servidor.

Por quê: a privacidade exige que os dados fiquem só com a família, e o servidor nem sempre está ligado; com as regras num só módulo, regime de caixa, conciliação, orçamento, rendimento e IR têm uma única implementação testada.

### Estrutura de pastas dominante

Proposta para a primeira fatia de implementação (pode ser ajustada no plano da fatia):

```
financeiro/
├── shared/             # Kotlin Multiplatform: domínio, regras de negócio, esquema SQLDelight, cliente de sincronização, telas Compose comuns
├── app-android/        # app Android (APK assinado com chave fixa)
├── app-desktop/        # app Windows (Compose desktop)
├── servidor/           # servidor Ktor (JVM), roda como serviço do Windows
├── empacotamento/      # montagem do .zip do Windows: runtime Java, WinSW, scripts de instalação
├── scripts/            # preparação do ambiente de nuvem (Android SDK, espelho do Maven), como no contagem_tm
├── gradle/             # catálogo de versões
└── docs/               # brainstorm, PRDs, TRD, ADRs, formatos, guias
```

### Módulos / camadas principais

| Módulo | Responsabilidade |
|---|---|
| `shared` — domínio | entidades (conta, cartão, fatura, lançamento, categoria, aplicação, orçamento) e regras: mês de referência (regime de caixa), transferências, previstos, rendimento, patrimônio, apoio ao IR |
| `shared` — dados | banco local SQLite (SQLDelight), registro de alterações para sincronização (ADR 003) |
| `shared` — sincronização | cliente: descobrir o servidor na rede de casa, enviar e receber alterações, aplicar a regra de conflito |
| `shared` — interface | telas Compose comuns a Android e Windows |
| `app-android` | ponto de entrada Android, armazenamento seguro de chaves, WorkManager, instalação de atualizações do APK |
| `app-desktop` | ponto de entrada Windows, assistente de instalação e de restauração, painel de saúde |
| `servidor` | vínculo de aparelhos, autorização por perfil, ordenação e junção das alterações, filtro por perfil, cópias de segurança, distribuição do APK |
| `empacotamento` | gera o .zip do Windows com runtime Java, WinSW e scripts |

Fluxo principal:

```
[celular]  ─┐                         ┌─▶ cópia criptografada ─▶ pasta do OneDrive
[celular]  ─┼─ Wi-Fi de casa (TLS) ─▶ [servidor no PC] ─┤
[computador]┘   (só quando em casa)   └─▶ cópia criptografada ─▶ disco externo
   cada aparelho: banco local completo para o seu perfil, funciona offline
```

---

## Requisitos Não-Funcionais

| Dimensão | Requisito |
|---|---|
| Performance | Relatórios com 9 anos de dados abrem no celular em até 3 s (PRD 008); lançar uma despesa conhecida em até 15 s (PRD 003); prévia da migração de 9 anos em até 1 min no computador (PRD 005) |
| Disponibilidade / SLA | O app funciona 100 % offline nos aparelhos; o servidor pode ficar desligado. RPO: 1 dia de dados no servidor (cópia diária), com os aparelhos guardando o que não foi sincronizado; RTO: algumas horas para restaurar num computador novo (PRD 011) |
| Escalabilidade | Uma família: até 4 usuários e poucos aparelhos; ~20 mil lançamentos no histórico desde 2017 e ~3 mil por ano (estimativa a partir da amostra de 648 lançamentos num trimestre) |
| Segurança | Nenhum dado financeiro sai da rede de casa, exceto a cópia de segurança criptografada. Banco do aparelho criptografado, chave liberada pelo PIN; TLS na rede de casa com certificado fixado no vínculo; cópia de segurança com chave de recuperação do casal (ADR 004). O aparelho do dependente não recebe dados da família (PRD 001) |
| Observabilidade | Sem telemetria nem envio de erros para fora. Log local no servidor com rotação; painel de saúde (PRD 011, US08) mostra sincronizações, cópias e avisos |
| Custo | Zero: sem serviço pago, sem loja, sem nuvem paga (restrição declarada) |

---

## Dependências Externas

| Serviço / Sistema | Tipo | Constraint relevante | Dono |
|---|---|---|---|
| OneDrive (cliente instalado no Windows) | pasta local sincronizada | o app só grava a cópia criptografada na pasta; a Microsoft guarda dados que não consegue ler; limitado pelo espaço da conta | externo (Microsoft), conta da família |
| Disco externo | sistema de arquivos | pode estar desconectado: a cópia falha, tenta de novo e avisa (PRD 011) | família |
| Wi-Fi de casa | rede local | descoberta automática do servidor pode ser bloqueada pelo roteador (isolamento de clientes, multicast desligado); prever IP informado à mão como alternativa | família |
| Android (restrições de segundo plano) | sistema | tarefa periódica no mínimo a cada 15 min e sujeita a atrasos por economia de bateria; sincroniza também ao abrir o app | externo (Google) |
| Windows (serviço, firewall, suspensão) | sistema | firewall pede liberação; servidor indisponível com o computador suspenso | externo (Microsoft), PC da família |
| GitHub | repositório privado | só código e documentos, nunca dados da família. GitHub Actions com Windows: cota gratuita para repositório privado **a verificar** (a [página de preços](https://github.com/pricing) cita 2.000 min/mês, possivelmente desatualizada; multiplicador do Windows não confirmado; relatos divergentes sobre limite de gasto zero, [discussão](https://github.com/orgs/community/discussions/210874), 2026-11-03) | externo |
| Regras da Receita Federal | documentação, sem integração | limites e cotações do IR são informados pelo casal a cada ano (PRD 010) | externo |
| Bancos (BB, Santander, XP, BBPrev, exterior) | arquivos de extrato e fatura, sem integração | formatos a definir no PRD 004; mudam sem aviso | externo |

---

## Padrões

### Testes

| Item | Valor |
|---|---|
| Framework | kotlin.test no módulo comum; JUnit nos módulos JVM e Android |
| Comando completo | `./gradlew test` (a confirmar quando o projeto Gradle existir) |
| Cobertura mínima | não definida; toda regra de negócio dos PRDs com critério de aceite numérico tem teste no módulo comum |
| Estratégia | unitários no domínio (regime de caixa, conciliação, rendimento, orçamento, IR); integração da sincronização (dois aparelhos simulados e servidor, conflitos e interrupções); migração testada contra uma exportação sintética do Minhas Economias (nunca a real no repositório) |

### Estilo de código

- **Linter:** a definir na primeira fatia (sugestão: ktlint; versão a verificar).
- **Formatter:** o do linter escolhido.
- **Convenções de nomenclatura:** padrão Kotlin (camelCase para funções e propriedades, PascalCase para classes). Termos de domínio em português, como nos PRDs (`Lancamento`, `Fatura`, `mesReferencia`), para manter a rastreabilidade com a especificação.

### Error handling

Erros de regra de negócio são resultados explícitos do domínio (ex.: lançamento inválido, partes da divisão que não fecham o total) e viram mensagens claras na tela. Falhas de infraestrutura (servidor fora, disco desconectado, OneDrive cheio) nunca perdem dados: a operação fica pendente, é repetida e vira aviso no painel (PRD 011). Valores monetários nunca usam ponto flutuante (centavos inteiros ou decimal exato).

### Logging

- **Formato:** texto estruturado em arquivo local no servidor, com rotação; nos aparelhos, só log local de depuração.
- **Nível padrão:** INFO no servidor; DEBUG só em build de desenvolvimento.
- **Biblioteca:** a definir na primeira fatia.
- **Regra:** logs nunca contêm valores, descrições de lançamentos, CPF/CNPJ nem nomes; só identificadores e contagens.

### Autenticação / autorização

Vínculo de aparelho autorizado por administrador com código temporário (PRD 001, US03); cada aparelho recebe uma credencial própria, revogável. O PIN libera a chave do banco local no aparelho. A autorização por perfil (administrador, dependente) é aplicada **no servidor**, que filtra o que cada aparelho recebe e recusa alterações fora do que o perfil pode fazer (PRD 001, US05 e US06).

---

## Decisões Globais (ADRs)

| # | Título | Data | Status | Link |
|---|--------|------|--------|------|
| 001 | Dados locais em cada aparelho, com servidor doméstico e sincronização só na rede de casa | 2026-11-03 | aceito | [adrs/001](adrs/001-local-first-com-servidor-domestico.md) |
| 002 | Kotlin com Compose Multiplatform (Android e Windows), servidor Ktor e SQLite | 2026-11-03 | aceito | [adrs/002](adrs/002-kotlin-compose-multiplatform-ktor-sqlite.md) |
| 003 | Sincronização por registro de alterações, junção campo a campo e relógio lógico híbrido | 2026-11-03 | proposto | [adrs/003](adrs/003-sincronizacao-por-registro-de-alteracoes.md) |
| 004 | Criptografia no aparelho, na rede de casa e na cópia de segurança, com chave de recuperação do casal | 2026-11-03 | proposto | [adrs/004](adrs/004-criptografia-e-chave-de-recuperacao.md) |
| 005 | Servidor como serviço do Windows, pacote montado na nuvem e app Android distribuído pelo servidor de casa | 2026-11-03 | aceito | [adrs/005](adrs/005-servico-windows-e-distribuicao.md) |
