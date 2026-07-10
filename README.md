# Sorteio de Times ⚽

Aplicação estática (`index.html`), em HTML/CSS/JS puro, para cadastrar jogadores e sortear times equilibrados, respeitando um conjunto de regras fixas configuradas no código.

## Como usar

Abra o `index.html` no navegador (ou publique via GitHub Pages, veja `.github/workflows/static.yml`).

1. Adicione os jogadores, um por um ou colando uma lista (um nome por linha).
2. Configure o número de times e a quantidade de jogadores por time.
3. Clique em **Sortear Times**.
4. Clique em **Novo sorteio** para gerar um novo sorteio com a mesma lista (o resultado nunca repete o sorteio anterior).
5. **Limpar tudo** reseta a lista de jogadores e o histórico de sorteios (contador de sorteios volta a zero).

## Regras do sorteio

As regras são baseadas em correspondência parcial e case-insensitive do nome (ex.: "thiago" casa com "Thiago Silva"), definidas no início do `<script>` em `index.html`:

- `MY_NAME = 'felipe'` — jogador de referência para as regras abaixo.
- `FELIPE_TEAM_NUMBER = 3` — **configurável**: número do time (1-indexado, igual ao rótulo "Time N" exibido na tela) em que o Felipe cai no primeiro sorteio. Se o sorteio tiver menos times que esse número, ele cai no último time disponível. Basta alterar essa constante para mudar o time do Felipe.
- `ALWAYS_WITH_FELIPE = ['luan', 'bruno', 'thiago', 'lucas']` — sempre no time do Felipe, mas **somente no primeiro sorteio** de cada lista (`drawCount === 0`). Rafael não faz mais parte dessa lista.
- `NEVER_WITH_FELIPE = ['abel']` — nunca pode cair no time do Felipe, mas **somente no primeiro sorteio** de cada lista.
- `CONSTRAINT_PAIRS = [['michel', 'rafael'], ['abel', 'thiago'], ['abel', 'jose'], ['abel', 'jhonatan'], ['abel', 'michel'], ['jhonatan', 'jose']]` — pares de jogadores que nunca podem cair juntos no mesmo time, independente do Felipe, valendo em **todos** os sorteios (inclusive re-sorteios).

### Primeiro sorteio vs. re-sorteios

- **Primeiro sorteio** (`drawCount === 0`): Felipe é colocado no time definido por `FELIPE_TEAM_NUMBER` (atualmente o Time 3). Quando chega a vez de preencher o time dele (ver "Ordem de preenchimento" abaixo), os jogadores de `ALWAYS_WITH_FELIPE` têm prioridade para as vagas restantes, respeitando `NEVER_WITH_FELIPE` e `CONSTRAINT_PAIRS`.
- **Re-sorteios** (`drawCount > 0`, botão "Novo sorteio"): Felipe é sorteado livremente entre os times, junto com todo mundo. `ALWAYS_WITH_FELIPE` e `NEVER_WITH_FELIPE` deixam de valer — apenas os `CONSTRAINT_PAIRS` continuam sendo respeitados.
- Um re-sorteio nunca repete exatamente o mesmo resultado do sorteio anterior (verificado via assinatura dos times, `teamSignature`).

### Ordem de preenchimento dos times

Os times são sempre preenchidos em sequência e até o fim: **Time 1 até completar `perTeam`, depois Time 2, depois Time 3, e assim por diante** — nenhum time "trava" jogadores fora de ordem, nem mesmo o time do Felipe. Não existe mais um modo especial para falta de jogador; o mesmo sorteio normal é usado independente de haver jogadores suficientes.

Para cada vaga, na ordem dos times:
1. Se for a vez do time do Felipe (1º sorteio), prioriza um jogador de `ALWAYS_WITH_FELIPE` que não viole conflito.
2. Se for a vez de **qualquer outro time**, evita pegar jogadores de `ALWAYS_WITH_FELIPE` enquanto houver outra opção sem conflito — eles ficam reservados para o time do Felipe.
3. Se a preferência acima não encontrar ninguém (ex.: só sobrou gente de `ALWAYS_WITH_FELIPE` no pool), pega o primeiro jogador disponível que não viole `NEVER_WITH_FELIPE` / `CONSTRAINT_PAIRS` com quem já está no time.
4. Se mesmo assim não houver ninguém sem conflito, a regra é ignorada — o time é sempre completado com quem sobrar, mesmo violando as restrições.

Se faltar jogador para fechar todos os times, a falta sempre sobra para o(s) último(s) time(s) da sequência (podendo até ficar vazio). Só nesse cenário de falta é que os jogadores de `ALWAYS_WITH_FELIPE` podem acabar em outro time — se houver jogadores suficientes para completar os times antes do Felipe, eles ficam garantidos no time dele.

## Estrutura do projeto

- `index.html` — aplicação completa (HTML, CSS e JS inline).
- `.github/workflows/static.yml` — publicação estática (GitHub Pages).
