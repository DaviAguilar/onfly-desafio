# Declaração de uso de IA

## Ferramenta e modelo

- **Ferramenta:** Claude Code (extensão para VS Code), da Anthropic
- **Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
- **Sessão:** uma única sessão, em 14/11/2026. A exportação está em `entrega/uso-de-ia/`.

## Resumo da divisão de trabalho

| Etapa | IA | Eu |
|-|-|-|
| Encontrar os defeitos | Fez a varredura e apontou os 5 defeitos | Revisei os achados |
| Corrigir o código | Só ajustou o estilo das aspas em uma linha | Escrevi as 5 correções |
| Validar as correções | Rodou testes, CLI e casos de borda a cada correção | Rodei os testes e decidi aceitar cada correção |
| Git e GitHub | Criou o repositório, escreveu as mensagens de commit, fez os pushes | Escolhi o nome e aprovei cada commit e push |
| Documentação | Escreveu esta declaração | Revisei a declaração |

## O que a IA fez

### 1. Varredura inicial do projeto

A meu pedido, a IA leu todo o projeto (código, testes, configuração, dependências e os três CSVs) procurando código que pudesse causar prejuízo. Ela:

- verificou que não havia código malicioso, dependência suspeita, credencial exposta, caractere oculto ou tentativa de injeção nos CSVs;
- rodou o CLI e o `pytest` para medir o impacto. Os 5 testes falhavam, e a base real gerava 200 reservas e -1393 noites;
- apontou os 5 defeitos, com arquivo e linha de cada um:
  1. leitura do CSV interrompida em 200 linhas (`csv_reservation_repository.py`);
  2. câmbio dividindo pela taxa em vez de multiplicar, e passando por `float` (`currency_converter.py`);
  3. `NO_SHOW` tratado como faturável (`status_policy.py`);
  4. noites calculadas pelo dia do mês, negativas na virada de mês (`stay_calculator.py`);
  5. ranking de hotéis em ordem crescente (`report_builder.py`);
- apontou também pontos de robustez que os dados atuais não disparam: moeda sem taxa gera código de saída 1 em vez de 2, status desconhecido não é sinalizado e linhas incoerentes não são validadas.

### 2. Ambiente de desenvolvimento

- Diagnosticou o erro `ImportError: attempted relative import with no known parent package` que aparecia no VS Code. A causa era rodar um arquivo do pacote diretamente, e não um defeito do código.
- Criou `.vscode/launch.json` e `.vscode/settings.json` para rodar o CLI como módulo (`-m onfly_report`). Esses arquivos ficaram fora do repositório entregue.

### 3. Repositório Git e GitHub

- Iniciou o repositório git e criou o repositório público `DaviAguilar/onfly-desafio` usando a CLI `gh`.
- Escreveu as mensagens de todos os commits e fez os pushes, sempre após a minha aprovação.
- Reescreveu o primeiro commit com `git push --force-with-lease` para corrigir o histórico. A minha primeira correção tinha entrado por engano no commit "Código original". O commit foi refeito com o código realmente original e a correção passou para um commit próprio. Nessa reescrita, a IA restaurou temporariamente as duas linhas originais e depois as removeu de novo. Não houve mudança de código além disso.

### 4. Validação das minhas correções

Para cada uma das 5 correções, a IA:

- leu o diff;
- rodou o `pytest`;
- rodou o CLI na amostra `data/exemplo-validacao.csv` e comparou com os valores esperados do README;
- testou casos de borda que os testes fornecidos não cobrem: virada de ano, ano bissexto, desempate do ranking por `hotel_id` e empates na base real;
- comentou se a solução era correta e eficiente.

### 5. Única alteração de código feita pela IA

Em `status_policy.py`, trocou `'CONFIRMED'` por `"CONFIRMED"` para seguir o padrão de aspas duplas do projeto, a meu pedido. A lógica não mudou.

### 6. Documentação

Escreveu esta declaração.

## O que eu fiz

- Pedi a varredura inicial e revisei os defeitos apontados.
- **Escrevi as 5 correções de código:**
  - removi o limite de 200 linhas;
  - troquei `.day` por `(checkout - checkin).days`;
  - restringi o faturamento a `CONFIRMED`;
  - troquei a conversão para `amount * rate` em `Decimal`;
  - inverti a ordenação com `-amount` e desempate por `hotel_id`.
- Rodei os testes após cada correção.
- Decidi o nome e a visibilidade do repositório, a organização dos commits (um por defeito) e aprovei cada commit e push.
- Decidi **não** implementar duas sugestões da IA: os testes adicionais e o tratamento de moeda sem taxa.
- Revisei esta declaração e a exportação da sessão antes de publicá-las.

## Resultado

Todas as correções estão no histórico do repositório, um commit por defeito. O `pytest -q` passa com os 5 testes fornecidos sem alteração (`5 passed`). A amostra de validação reproduz exatamente os valores esperados do README.
