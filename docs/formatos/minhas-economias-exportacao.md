# Formato: exportação do Minhas Economias

Descrição do arquivo de exportação usado na migração do histórico (PRD 005). Analisado em 2026-11-03 a partir de uma amostra real (1º trimestre de 2026, 648 linhas). **A amostra não fica no repositório**; aqui só o formato, sem nomes nem valores.

## Arquivo

- Excel antigo (`.xls`, formato binário CDFV2). Nome: `Extrato_AAAAMMDD_AAAAMMDD.xls` (período exportado).
- Duas planilhas: `Transações` (os dados) e `Filtros ativos` (vazia na amostra).
- Uma linha de cabeçalho; uma linha por lançamento.

## Colunas da planilha `Transações`

| Coluna | Conteúdo | Observações |
|---|---|---|
| `Data Ocorrência` | data do lançamento | data do Excel com hora fixa 12:00; sem hora real |
| `Descrição` | texto livre | parcelas aparecem só no texto, ex.: `Loja (3 / 3)` |
| `Valor` | número com sinal | negativo = saída, positivo = entrada; nenhum zero na amostra |
| `Categoria` | nome da categoria | **um nível só** (o nome da subcategoria, sem o grupo); `Sem Categoria` quando não categorizado |
| `Conta` | nome da conta ou cartão | nomes livres dados pelo usuário (ex.: prefixos `01.`, `02.`, `03.` para ordenar) |

## O que o formato traz e o que não traz

- **Transferência:** categoria `Transferência`, em **duas linhas** (saída na origem, entrada no destino) com a mesma data, a mesma descrição e valores opostos. Na amostra, todas formam par (44 pares). O pagamento de fatura é uma transferência para a "conta" do cartão.
- **Cartão:** o cartão aparece como uma conta. As compras têm a data da compra; **não há** fatura, fechamento, vencimento nem cartão adicional (quem fez a compra).
- **Parcelas:** não há estrutura de parcelamento; só o texto `(n / N)` na descrição, quando o usuário escreveu.
- **Categorias:** a hierarquia (grupo > subcategoria) **não vem** na exportação; precisa ser reconstruída na migração.
- **Moeda:** não há coluna de moeda.
- **Tipos de conta:** não há coluna de tipo; corrente, cartão, investimento e dinheiro só se distinguem pelo nome.
- Não há CPF/CNPJ, favorecido separado da descrição, nem observação.

## Contas e categorias encontradas (tipos, sem nomes)

- Contas correntes (2 bancos), cartões (4), investimentos (vários, inclusive fundos), dinheiro em espécie **por pessoa** (`Cash - <pessoa>`), **conta de pessoa jurídica** e investimento da PJ, e investimento em nome de **outra pessoa da família** (fora do casal).
- 71 categorias, entre elas: `Transferência`, `Reembolso`, `Juros de Investimentos`, `Semanada Filhos`, categorias de PJ (`Faturamento PJ`, `ISS`, `INSS`, `Contabilidade`, `Impostos`) e uma de **espólio** de parente.

## A confirmar com o casal

- O formato é o mesmo nas exportações dos anos anteriores?
- Período total do histórico.
