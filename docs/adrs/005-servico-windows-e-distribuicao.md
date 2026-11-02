---
adr_number: "005"
status: aceito
created: 2026-11-03
supersedes: ""
superseded_by: ""
---

# ADR 005: Servidor como serviço do Windows, pacote montado na nuvem e app Android distribuído pelo servidor de casa

## Contexto

O servidor deve funcionar sempre que o computador estiver ligado, mesmo sem ninguém logado (PRD 011, US04), com instalação simples feita por quem tem alguma familiaridade técnica. O desenvolvimento acontece em sessões de nuvem do Claude (Linux). O instalador nativo do Windows gerado por `jpackage` só pode ser produzido no próprio Windows. O app Android não vai para a Play Store (PRD 011) e, como no `contagem_tm`, é instalado por APK assinado com chave fixa. Escolhas do casal em 2026-11-03.

## Alternativas Consideradas

- **Servidor:** serviço do Windows com WinSW (programa livre e mantido que transforma um processo Java em serviço) — roda sem login; escolhido. **Ou** o app do computador como servidor — mais simples, mas só com alguém logado; descartado.
- **Empacotamento do Windows:** (a) GitHub Actions com máquina Windows — instalador nativo automático; condições da cota gratuita para repositório privado **a verificar**; (b) script de build no computador do marido; (c) **pacote .zip montado na nuvem**, com o Java para Windows, o WinSW e um script de instalação — escolhido por ora.
- **App Android:** Play Store (descartado: exige conta paga e publicação) **ou** APK distribuído pelo servidor de casa (escolhido).

## Decisão

- O servidor (Ktor, ADR 002) roda como **serviço do Windows via WinSW**, iniciando com o computador.
- O app de Windows e o servidor são entregues num **pacote .zip montado na sessão de nuvem**, com runtime Java para Windows incluído e um script de instalação que registra o serviço, cria atalhos e abre o assistente (PRD 011, US04). Plano B: script de build no computador de casa. Plano C, a avaliar: GitHub Actions com Windows.
- O **APK** é assinado com **chave fixa** guardada no repositório privado (mesma disciplina do `contagem_tm`: nunca trocar a chave, porque isso obrigaria desinstalar e perder dados não sincronizados) e distribuído aos celulares **pelo servidor de casa**, que oferece a atualização na sincronização (PRD 011, US05).

## Consequências

- **Positivas:** tudo é gerado na nuvem, sem depender do Windows para compilar; o servidor funciona sem login; atualização dos celulares sem loja.
- **Negativas:** o .zip com script é menos polido que um instalador nativo; o Windows pode exibir avisos de segurança ao rodar um programa não assinado; o celular precisa permitir a instalação de apps desta origem.
- **Neutras / trade-offs aceitos:** a troca para instalador nativo (planos B ou C) pode ser feita depois sem mudar o código, só o empacotamento.
