---
adr_number: "004"
status: proposto
created: 2026-11-03
supersedes: ""
superseded_by: ""
---

# ADR 004: Criptografia no aparelho, na rede de casa e na cópia de segurança, com chave de recuperação do casal

## Contexto

Dados financeiros em celulares (inclusive de adolescentes) e numa pasta do OneDrive. Decisões de produto: PIN ao entrar; dados no aparelho criptografados; aparelho perdido revogável (PRD 001); cópia de segurança criptografada antes de sair do computador, com chave de recuperação só do casal (PRD 011). O vínculo do aparelho é autorizado por um código mostrado no aparelho novo (PRD 001, US03).

## Alternativas Consideradas

- **Banco no aparelho:** SQLCipher (banco SQLite inteiro criptografado) — não há driver oficial para SQLDelight; existe driver comunitário em versão 0.1.x (verificado em 2026-11-03, fonte no TRD); **ou** criptografia por campo sensível com chave guardada no Android Keystore — sem dependência extra, mas protege só parte dos dados. A decidir.
- **Rede de casa:** HTTP sem criptografia (descartado: Wi-Fi compartilhado com visitas) **ou** TLS com certificado próprio do servidor, cuja impressão digital o aparelho aprende no vínculo (proposto).
- **Cópia de segurança:** sem criptografia (descartado) **ou** criptografia autenticada com chave derivada da chave de recuperação (proposto).

## Decisão

*(proposta — a confirmar na revisão do TRD; a forma da criptografia do banco no Android depende de avaliar o driver SQLCipher comunitário)*

- **Aparelho:** banco local criptografado; a chave fica protegida pelo armazenamento seguro do sistema (Android Keystore; no Windows, a proteção de dados do usuário) e só é liberada após o PIN.
- **Rede:** TLS com certificado autoassinado do servidor, fixado no aparelho durante o vínculo (o código de vínculo também confirma a identidade do servidor).
- **Cópia de segurança:** arquivo criptografado com criptografia autenticada, com chave derivada da **chave de recuperação** gerada na instalação (PRD 011, US04); sem a chave, a cópia não abre.
- Nenhuma chave vai para o repositório nem para serviços externos.

## Consequências

- **Positivas:** celular perdido e pasta do OneDrive não expõem os dados; visitas no Wi-Fi não leem a sincronização.
- **Negativas:** perder a chave de recuperação inutiliza as cópias antigas (mitigado no PRD 011); dependência de um driver comunitário, ou proteção parcial, no Android.
- **Neutras / trade-offs aceitos:** PIN de 6 dígitos com espera crescente (PRD 001) protege a abertura do app; a força real vem do armazenamento seguro do sistema.
