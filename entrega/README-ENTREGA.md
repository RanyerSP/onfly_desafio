# Entrega — Consolidador de reservas de hotel (Onfly)

CLI em Python que gera o relatório JSON consolidado das reservas de hotel. O código original entregava números errados; este documento registra os defeitos encontrados, a causa raiz de cada um, como foram investigados e como validar.

## Resumo dos defeitos corrigidos

| # | Defeito | Causa raiz (uma frase) | Arquivo |
|---|---------|------------------------|---------|
| 1 | Valores em moeda estrangeira incorretos e perda de precisão no dinheiro | `convert_to_brl` dividia o valor pela taxa em vez de multiplicar e ainda passava os valores por `float` antes de voltar a `Decimal`. | `services/currency_converter.py` |
| 2 | Noites negativas ou erradas na virada de mês/ano | O cálculo subtraía apenas o dia do mês (`checkout.day - checkin.day`) em vez da diferença entre as datas completas. | [CONFERIR: arquivo do cálculo de noites] |
| 3 | Reservas `NO_SHOW` somadas nos valores faturáveis | O filtro excluía apenas `CANCELLED`, deixando `NO_SHOW` entrar nos totais; só `CONFIRMED` é faturável. | [CONFERIR: arquivo do filtro, provavelmente `services/report_builder.py`] |
| 4 | Ranking de `top_hotels` em ordem inversa | A ordenação era crescente por valor, de modo que o `--top N` devolvia os N hotéis de menor valor. | `services/report_builder.py` |

Obs: O test_regressions_report.py ao chegar nele para sua correção já não apresentava mais o erro. Que seria o relatório completo.

## Detalhamento

### 1. Câmbio: divisão no lugar de multiplicação, e uso de `float`

- **Sintoma:** o teste de câmbio esperava `642.00` e o CLI devolveu `118.45`.
- **Causa raiz:** a função fazia `Decimal(str(float(amount) / float(rate)))`. Dois problemas na mesma linha: a operação era divisão (o README manda `amount * rate_to_brl`) e os valores passavam por `float`, que não representa decimais com exatidão.
- **Correção:** `amount * rate`, mantendo `Decimal` do início ao fim, sem arredondar na conversão (o arredondamento é feito só na serialização).

### 2. Noites: diferença calculada só com o dia do mês

- **Sintoma:** check-in 30/01 e check-out 02/02 resultava em `-28` noites; o esperado era `3`.
- **Causa raiz:** `checkout.day - checkin.day` (`2 - 30 = -28`) ignora mês e ano. Só funcionava quando as duas datas estavam no mesmo mês.
- **Correção:** `(checkout - checkin).days * rooms`. A subtração de datas completas gera um `timedelta` que já trata meses de tamanhos diferentes, virada de ano e anos bissextos.

### 3. Status: só `CONFIRMED` é faturável

- **Sintoma:** com reservas `CONFIRMED` (100.00), `CANCELLED` (50.00) e `NO_SHOW` (20.00), o total veio `120.00` em vez de `100.00`.
- **Causa raiz:** o filtro barrava apenas `CANCELLED`; o `NO_SHOW` (20.00) era somado ao `CONFIRMED` (100.00), o que explica o `120.00`.
- **Correção:** o filtro passou a considerar somente `CONFIRMED` nos valores. As contagens por status já estavam corretas (feitas antes do filtro) e não foram alteradas.

### 4. Ranking de hotéis em ordem inversa

- **Sintoma:** com 4 hotéis (H01 a H04, do maior para o menor valor) e `top=3`, o CLI devolveu `['H04', 'H03', 'H02']` em vez de `['H01', 'H02', 'H03']`.
- **Causa raiz:** a chave de ordenação era `(amount, hotel_id)`, crescente nos dois critérios; o corte do `--top` pegava os primeiros da lista, ou seja, os de menor valor.
- **Correção:** a chave passou a ser `(-amount, hotel_id)`: valor decrescente e, em caso de empate, `hotel_id` crescente, como pede o README. Não foi usado `reverse=True`, que inverteria também o desempate.

## Como investiguei

1. Rodei cada arquivo de teste separadamente (`pytest -q tests/test_xxx.py`) e tratei um erro por vez.
2. Antes de abrir o código, reconstruí a conta por trás do valor errado, o que apontou a causa em todos os casos:
   - câmbio: `100 ÷ 5.42 + 100 = 118.45` (o código dividia);
   - noites: `2 − 30 = −28` (o código usava só o dia do mês);
   - status: `100 + 20 = 120` (o `NO_SHOW` entrava na soma);
   - ranking: `H04, H03, H02` são os três menores em ordem crescente (a ordenação estava invertida).
3. Localizei a linha no código com busca recursiva (`Get-ChildItem -Recurse | Select-String`) e li o fluxo em camadas (CLI → repositório de leitura → serviços de regra de negócio).
4. Corrigi um defeito por vez, rodei o teste isolado e depois o `pytest -q` completo, e fiz um commit por defeito (mensagens no padrão Conventional Commits, em branch `fix/...`).
5. Descartei suspeitos lendo o código: o `CsvRateRepository` já lia as taxas corretamente como `Decimal` a partir de texto, então o defeito estava na camada de serviço.

Usei o Claude (IA) como apoio para entender os conceitos (`Decimal` x `float`, `timedelta`, ordenação estável, import relativo ao rodar o CLI) e para discutir cada diagnóstico. A leitura do código, a localização das linhas e as alterações foram feitas por mim. Detalhes em `entrega/uso-de-ia/`.

## Como validar

**PowerShell (Windows)**

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements-dev.txt

$env:PYTHONPATH = "src"
pytest -q
python -m onfly_report --input data/reservas_hoteis.csv --rates data/taxas_cambio.csv --output output/report.json
python -m onfly_report --input data/exemplo-validacao.csv --rates data/taxas_cambio.csv --output output/exemplo.json
```

**Bash (Linux/macOS)**

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

PYTHONPATH=src pytest -q
PYTHONPATH=src python -m onfly_report --input data/reservas_hoteis.csv --rates data/taxas_cambio.csv --output output/report.json
PYTHONPATH=src python -m onfly_report --input data/exemplo-validacao.csv --rates data/taxas_cambio.csv --output output/exemplo.json
```

Conferência da amostra: comparar `output/exemplo.json` com a tabela do README do desafio (12 reservas, 9 confirmadas, 28 noites, `total_amount_brl` = `7566.00`, média `270.21`).

## Resultado do `pytest`

```
(.venv-1) PS C:\Users\Ranyer\Downloads\onfly_desafio> pytest -q 
.....                                                                                                                                                                    [100%]
5 passed in 0.71s
(.venv-1) PS C:\Users\Ranyer\Downloads\onfly_desafio> 

```

## Testes adicionais



## Uso de IA

Foi utilizada IA como apoio ao aprendizado e à discussão dos diagnósticos. A exportação da sessão e a declaração estão em `entrega/uso-de-ia/` (`sessao.md` e `declaracao.md`).
