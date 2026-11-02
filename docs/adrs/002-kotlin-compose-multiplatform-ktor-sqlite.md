---
adr_number: "002"
status: aceito
created: 2026-11-03
supersedes: ""
superseded_by: ""
---

# ADR 002: Kotlin com Compose Multiplatform (Android e Windows), servidor Ktor e SQLite

## Contexto

O app precisa rodar em celulares Android e computadores Windows, com um servidor no Windows (ADR 001). A família não usa iPhone. O autor já desenvolve em Kotlin no projeto `contagem_tm` (app Android de coleta, módulo Kotlin puro), com APK gerado em sessão de nuvem do Claude e assinado com chave fixa; Kotlin 2.4.21 e Android Gradle Plugin 9.4.0 já compilam nesse ambiente. Escolha do casal em 2026-11-03.

## Alternativas Consideradas

- **Kotlin + Compose Multiplatform** — uma linguagem e uma interface para Android e Windows; servidor na mesma linguagem; experiência e fluxo de build já existentes no `contagem_tm`. Escolhido.
- **Flutter (Dart)** — atende Android e Windows; mas é linguagem e ecossistema novos para o autor, e o servidor seria em outra linguagem ou em Dart com ecossistema menor.
- **App web servido pelo computador de casa (PWA)** — sem instalação; mas o funcionamento offline exige HTTPS válido na rede local, difícil de manter, e o armazenamento do navegador pode ser apagado. Desaconselhado.

## Decisão

- Linguagem: **Kotlin** em todas as partes.
- Interface: **Compose Multiplatform** para Android e Windows (desktop JVM), com o código de tela e de domínio compartilhado.
- Servidor: **Ktor** em JVM, rodando como serviço do Windows (ADR 005).
- Banco: **SQLite** em cada aparelho e no servidor, com **SQLDelight** para consultas tipadas e compartilhadas.
- Versões de referência em `docs/trd.md` (seção Stack), com fonte e data da verificação.

## Consequências

- **Positivas:** regras de negócio (regime de caixa, conciliação, rendimento, orçamento) escritas uma vez e testadas no módulo comum; mesmo fluxo de build e assinatura do `contagem_tm`.
- **Negativas:** o instalador nativo do Windows (`jpackage`) só é gerado no Windows (ADR 005); criptografia do SQLite no Android não tem driver oficial para SQLDelight (risco tratado no ADR 004).
- **Neutras / trade-offs aceitos:** a interface no Windows segue o visual do Compose, não o visual nativo do Windows.
