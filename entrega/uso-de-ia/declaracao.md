# Declaração de uso de IA

## Ferramenta e modelo

- **Ferramenta:** Claude, da Anthropic (interface de chat, com execução em ambiente de trabalho próprio da ferramenta).
- **Modelos:** Claude Opus 5.5 e Claude Sonnet 5.5. O modelo foi alternado ao longo da mesma conversa: as primeiras respostas (leitura do README e comandos de PowerShell) vieram do Opus 5.5 e as demais, do Sonnet 5.5. [CONFERIR com o seletor de modelo da interface antes de entregar]
- **Registro da sessão:** `sessao.md` (conversa completa, prompts e respostas), nesta mesma pasta.

## Como a IA foi usada

Usei a IA como apoio ao aprendizado e à discussão, pois sou iniciante em programação e queria entender o que estava corrigindo. A IA não teve acesso ao repositório: viu apenas o que eu anexei ou colei na conversa (o README do desafio, o arquivo de teste de câmbio, a mensagem de erro do CLI, saídas do terminal e trechos de código).

## O que a IA fez

- Explicou o README e o objetivo do projeto: regras de negócio, contrato do CLI, formato do JSON, `Decimal` x `float`, arredondamento half-up e códigos de saída.
- Traduziu os comandos de Bash para PowerShell (ambiente virtual, variável `PYTHONPATH`, quebra de linha, busca recursiva com `Select-String`).
- Orientou nomenclatura de branch (`fix/...`) e mensagens de commit (Conventional Commits).
- Explicou o erro de import relativo ao rodar `cli.py` diretamente (o CLI deve ser executado com `python -m onfly_report`).
- A cada teste que falhava, ajudou a interpretar a saída do `pytest` e a deduzir a causa a partir dos números (`118.45`, `-28`, `120.00`, ordem `H04, H03, H02`).
- Indicou comandos de busca para localizar as linhas no código e leu os trechos que colei (`currency_converter.py`, a ordenação de `top_hotels`), apontando onde estava o defeito.
- Sugeriu, em texto, a forma de cada correção (multiplicar pela taxa mantendo `Decimal`; subtrair datas completas; aceitar apenas `CONFIRMED`; ordenar por valor decrescente com desempate por `hotel_id` crescente).
- Fez um primeiro rascunho do `README-ENTREGA.md` e desta declaração, que revisei e ajustei.
- Gerou o `sessao.md` a partir do registro da conversa. Também sugeriu, em texto, ideias de testes adicionais; elas não foram adotadas, e nenhum teste novo faz parte da entrega.

## O que a IA não fez

- Durante a investigação e a correção dos defeitos, não acessou, editou nem executou nada no meu código, nos testes, no terminal ou no Git; os resultados do `pytest` e do CLI citados foram os que colei da minha máquina. Os defeitos foram corrigidos por mim.
- Só depois de os defeitos estarem corrigidos a IA passou a ter acesso ao repositório (no Claude Code), onde gerou o `sessao.md` e editou os documentos de `entrega/`.
- Não alterou os testes fornecidos nem o código do programa, e não adicionou testes.
- Não fez commits nem publicou nada no GitHub.

## O que eu fiz

- Montei o ambiente (Python, venv, dependências) e executei todos os comandos na minha máquina (Windows/PowerShell).
- Rodei os testes um a um, li as mensagens de erro e colei os resultados para discussão.
- Procurei os trechos no código, li os arquivos e identifiquei onde cada defeito estava.
- Apliquei as correções no código e confirmei, rodando os testes, que os defeitos foram resolvidos e todos os testes passaram.
- No defeito de status, decidi a forma final da alteração depois de discutir a abordagemm, mantendo apenas os dados do status CONFIRMED.
- Revisei o `README-ENTREGA.md`, completei os pontos pendentes e sou responsável pelo conteúdo entregue, pelo versionamento (branch e commits) e pela publicação no GitHub.
