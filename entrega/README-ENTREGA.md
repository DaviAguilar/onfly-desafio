# Entrega — Consolidador de reservas de hotel

Corrigi os 5 defeitos que faziam o CLI gerar números errados. Os 5 testes fornecidos passam sem alteração, e a amostra de validação reproduz exatamente os valores do README. Cada correção está em um commit próprio no histórico.

## Defeitos corrigidos

| # | Arquivo | Causa raiz | Correção |
|-|-|-|-|
| 1 | `repositories/csv_reservation_repository.py` | A leitura do CSV parava ao atingir 200 linhas e descartava as demais sem aviso. | Removi o `if len(rows) == 200: break`. |
| 2 | `services/stay_calculator.py` | As noites eram calculadas subtraindo só o dia do mês (`.day`), o que gerava valores negativos na virada de mês ou de ano. | Usei a diferença real entre as datas: `(checkout - checkin).days * rooms`. |
| 3 | `services/status_policy.py` | A política faturava todo status diferente de `CANCELLED`, o que incluía `NO_SHOW` nos totais. | Passei a faturar só `status == "CONFIRMED"`. |
| 4 | `services/currency_converter.py` | A conversão dividia o valor pela taxa em vez de multiplicar e passava por `float`, perdendo a exatidão do `Decimal`. | Passei a calcular `amount * rate`, inteiramente em `Decimal`. |
| 5 | `services/report_builder.py` | O ranking ordenava os hotéis por valor crescente e exibia os de menor faturamento. | Ordenei por `(-amount, hotel_id)`: valor decrescente, com desempate por `hotel_id` crescente. |

### Histórico de commits

```
db11743 Ordena top hotéis por valor decrescente
72ff41f Converte para BRL multiplicando pela taxa em Decimal
86a4a26 Fatura apenas reservas CONFIRMED
e6566c5 Calcula noites pela diferença entre datas
4dad511 Remove limite de 200 linhas na leitura de reservas
20eb387 Código original do desafio Onfly
```

O primeiro commit guarda o código exatamente como foi recebido, para que cada correção apareça isolada no diff.

## Como investiguei

1. **Comparei o README com o código.** Li as regras de negócio (status, noites, câmbio, exatidão, ordenação) e conferi cada uma contra o módulo que deveria implementá-la. O pipeline segue a ordem descrita no README (`ler → converter tipos → contar status → filtrar CONFIRMED → agregar`), então cada etapa foi verificada separadamente.
2. **Medi o estado inicial.** Rodei o `pytest` e o CLI antes de mexer no código. Os 5 testes falhavam, e a base real gerava valores claramente impossíveis: 200 reservas num arquivo de 300 linhas e um total de -1393 noites.
3. **Perfilei os dados.** O README afirma que os dados estão consistentes. Mesmo assim, verifiquei:
   - status, moedas e quantidade de quartos distintos;
   - IDs duplicados;
   - checkout anterior ao check-in;
   - caracteres ocultos.

   Não encontrei nada fora do padrão, o que confirmou que os defeitos estavam no CLI. O perfil também mostrou que 71 reservas confirmadas atravessam a virada de mês, o que explicava as noites negativas.
4. **Corrigi um defeito por vez.** Depois de cada correção, rodei os testes e o CLI na amostra e verifiquei qual teste passava a funcionar. Assim nenhuma correção mascarava outra. Por exemplo, a amostra chegou a mostrar 29 noites em vez de 28 até a correção de status, por causa de uma reserva `NO_SHOW` de 1 noite.
5. **Testei casos que os testes fornecidos não cobrem:**
   - virada de ano: 30/12 a 02/01 com 2 quartos dá 6 noites;
   - ano bissexto: 28/02 a 01/03/2024 dá 2 noites;
   - desempate do ranking: com dois hotéis em 500.00, o `H01` vem antes do `H03`;
   - ausência de empates no valor arredondado entre os hotéis da base real.
6. **Recalculei os totais por fora do CLI.** Fiz um script independente que lê o CSV direto e recalcula os totais da base real. Os resultados bateram com os do CLI.

Usei IA nesta investigação. Os detalhes estão em `entrega/uso-de-ia/declaracao.md`.

## Como validar

Requisitos: Python 3.11+ e `pip install -r requirements-dev.txt`.

```bash
# testes fornecidos
pytest -q

# amostra de validação (comparar com a tabela do README)
PYTHONPATH=src python -m onfly_report \
  --input data/exemplo-validacao.csv \
  --rates data/taxas_cambio.csv \
  --output output/exemplo.json

# relatório da base completa
PYTHONPATH=src python -m onfly_report \
  --input data/reservas_hoteis.csv \
  --rates data/taxas_cambio.csv \
  --output output/report.json
```

### Amostra de validação

Todos os valores conferem com o README.

| Campo | Obtido | Esperado |
|-|-|-|
| `total_reservations` | 12 | 12 |
| `confirmed` / `cancelled` / `no_show` | 9 / 2 / 1 | 9 / 2 / 1 |
| `total_room_nights` | 28 | 28 |
| `total_amount_brl` | 7566.00 | 7566.00 |
| `average_nightly_rate_brl` | 270.21 | 270.21 |
| `by_month` 2025-01 | 8 reservas, 25 noites, 6684.00 | 8 reservas, 25 noites, 6684.00 |
| `by_month` 2025-02 | 1 reserva, 3 noites, 882.00 | 1 reserva, 3 noites, 882.00 |
| `top_hotels` | H01, H02, H12, H10, H09 | H01, H02, H12, H10, H09 |

### Base completa (`data/reservas_hoteis.csv`)

| Campo | Antes | Depois |
|-|-|-|
| `total_reservations` | 200 | 300 |
| `confirmed_reservations` | 153 | 235 |
| `total_room_nights` | -1393 | 1347 |
| `total_amount_brl` | 193218.53 | 396170.00 |
| `average_nightly_rate_brl` | — | 294.11 |

O top 5 de hotéis ficou assim:

| Posição | Hotel | Total BRL |
|-|-|-|
| 1 | H06 Hotel Salvador Centro | 45165.46 |
| 2 | H10 Hotel Madrid, Gran Vía | 45041.69 |
| 3 | H12 Hotel Miami Beach | 41647.69 |
| 4 | H07 Hotel Brasília Tower | 39920.52 |
| 5 | H11 Hotel Buenos Aires | 37455.69 |

Os valores do "Depois" foram conferidos com o script independente do passo 6.

## Resultado do `pytest`

```
$ pytest -q
.....                                                                    [100%]
5 passed in 1.38s
```

Rodei os testes com Python 3.13.3. As correções não usam nenhum recurso posterior ao Python 3.11, mas não testei especificamente no 3.11.

## Limitações conhecidas (fora do escopo)

Identifiquei três pontos de robustez. Os dados fornecidos não os disparam, então optei por não alterá-los:

- **Moeda sem taxa na tabela:** gera `KeyError` e faz o CLI sair com código `1` ("erro inesperado"), quando o contrato pede `2` para entrada inválida.
- **Status fora de `CONFIRMED`/`CANCELLED`/`NO_SHOW`:** entra em `total_reservations`, mas não aparece em nenhuma contagem por status nem gera aviso.
- **Linhas incoerentes:** checkout ≤ check-in, `rooms` ≤ 0 ou `amount` negativo não são rejeitados.
