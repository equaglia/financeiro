# Categorias iniciais (proposta para o casal ajustar)

Proposta de 2026-11-03. A exportação do Minhas Economias traz só o nome da categoria, sem o grupo (`docs/formatos/minhas-economias-exportacao.md`). Aqui as **71 categorias** encontradas na amostra do 1º trimestre de 2026 foram agrupadas em dois níveis (grupo > categoria). Esta lista é a lista inicial do app (PRD 003, US06) e o mapa da migração (PRD 005).

**Como ajustar:** mudar o grupo de uma categoria, renomear, juntar duas, ou riscar. As marcadas com **?** são as que mais pedem confirmação.

Legenda: **IR** = categoria dedutível no imposto de renda (saúde ou educação; regras confirmadas no PRD 010).

## Despesas

| Grupo | Categorias |
|---|---|
| Moradia | Aluguel, Condomínio, IPTU, Luz, Gás, Internet, TV por assinatura, Seguro Residencial, Manutenção da Casa, Móveis e decoração, Utensílios Domésticos, Empregada Doméstica |
| Alimentação | Supermercado, Padaria, Almoço, Jantar, Café/Lanches, Bar, Vinho, Nespresso, Herbalife **?** (ou Saúde) |
| Transporte | Táxi, Metrô/Ônibus, Combustível, Pedágio, Aluguel de carro, Manutenção do Carro |
| Saúde | Médico (IR), Dentista (IR), Psicólogo (IR), Exames (IR), Farmácia, Óculos |
| Educação | Faculdade (IR), Cursos, Vestibulares, Livros, Material de Artes, Atividades Extras |
| Lazer e viagens | Viagem, Passeios, Cinema/Teatro/Show, Clube, Festa, Fotografia, Material Esportivo |
| Cuidados pessoais | Roupas e Calçados, Salão de Beleza, Perfumaria |
| Filhos | Semanada Filhos |
| Animais | Ração, Veterinário |
| Compras e serviços | Eletrônicos, Software, Presentes, Doações, Profissional **?** (o que é?) |
| Impostos e tarifas | Impostos, INSS **?** (da empregada doméstica ou contribuição própria?), IOF, Tarifas bancárias |
| Outros | Outros Gastos |

## Receitas

| Grupo | Categorias |
|---|---|
| Rendimentos | Juros de Investimentos, **Lucros e dividendos** (nova: hoje é transferência vinda da conta PJ, que fica fora da 1ª versão) |
| Reembolsos | Reembolso **?** (do plano de saúde, da empresa ou outro? Reembolso de saúde abate a despesa dedutível no IR) |
| Outras receitas | PayPal **?** (entrada única na amostra; o que é?) |

## Não viram categoria

| Categoria no Minhas Economias | Destino no app |
|---|---|
| Transferência | vira **transferência** entre contas (os pares de linhas são unidos na migração) |
| Espólio (de parente) | vira **conta administrada** (PRD 002, US12); os lançamentos passam a ser transferências entre a conta do BB e a conta administrada |
| Sem Categoria | grupo **A classificar**, para o casal revisar depois da migração |
| Faturamento PJ, ISS, Contabilidade | ficam com as **contas PJ**, fora da 1ª versão (não migram) |

## Categorias novas sugeridas

| Grupo | Categoria | Por quê |
|---|---|---|
| Saúde | Plano de saúde (IR) **?** | dedutível no IR e não aparece na amostra: é pago por outra conta (PJ?) ou com outro nome? |
| Saúde | Hospital (IR) | dedutível no IR |
| Filhos | Despesas dos filhos **?** | hoje "Depósito <filho>" cai em "Outros Gastos" |
| (fixa) | Ajuste de saldo | conferência com o extrato (PRD 002); fora de despesas e receitas |

## Observações

- A amostra cobre só jan–mar de 2026. Categorias que existirem só em anos anteriores (desde abril de 2017) vão para **A classificar** na migração, e o casal as encaixa num grupo (PRD 005).
- "Material  de Artes" tem dois espaços no Minhas Economias; o nome é corrigido na migração.
