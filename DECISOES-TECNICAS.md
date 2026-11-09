# Decisões técnicas

Registro das mudanças técnicas na plataforma do Vocabulário de História do Tempo Presente (data, o que foi pedido, o que foi alterado e por quê).

## 2026-11-08. Interface em português do Brasil

- **Pedido:** colocar a interface em pt-BR e trocar o título do site.
- **Alterado:** em `quartz.config.yaml`, `locale` de `en-US` para `pt-BR` e `pageTitle` de "Quartz 5" para "Vocabulário de História do Tempo Presente".
- **Por quê:** o Quartz já traz tradução pt-BR; o título identifica o projeto.
- **Pendente:** textos ainda em inglês (rodapé, "Home", busca, "Properties") e correções na tradução ("items", "gráfico").

## 2026-11-08. Arquivo .claude/launch.json (pré-visualização local)

- **Pedido:** conferir como ficou o site depois da mudança de idioma e de título.
- **Alterado:** criado o arquivo `.claude/launch.json`, que diz ao app do Claude como iniciar o site local (`npx quartz build --serve`, porta 8080).
- **Por quê:** o app só abre a pré-visualização a partir dessa configuração. O arquivo não altera o site, o conteúdo nem a publicação, e não foi incluído nos commits.
- **Observação:** foi criado além do que estava pedido naquele momento, por ser necessário para a conferência; fica registrado aqui por transparência.

## 2026-11-08. "Home" vira "Início" e rodapé com "Código-fonte" (configuração)

- **Pedido:** segunda rodada de tradução da interface, com os termos da súmula (itens 5 e 9 da proposta).
- **Alterado:** em `quartz.config.yaml`:
  - plugin `breadcrumbs`: acrescentada a opção `rootName: Início` (antes aparecia "Home");
  - plugin `footer`: os links "GitHub" (apontava para o Quartz original) e "Discord Community" foram removidos, e entrou "Código-fonte" apontando para https://github.com/policronias/vocabulario-htp.
- **Por quê:** as duas opções existem na documentação oficial do plugin e resolvem o texto sem mexer em código.
- **ATENÇÃO, versão em inglês:** esses dois textos vêm da configuração do plugin, não da tradução do idioma, então NÃO mudam quando o `locale` muda. No teste com `locale: en-US` continuaram "Início" e "Código-fonte". Se o site ganhar uma versão em inglês, cada um terá de ter um equivalente em inglês ("Home" e "Source code", por exemplo), e a configuração terá de ser separada por idioma.

## 2026-11-08. Remendos de tradução dos plugins, fase 1 (patch-package)

- **Pedido:** trocar, só em português, textos que ficam dentro de plugins (itens 1, 2, 4 e 6 da proposta), sem afetar uma eventual versão em inglês.
- **Problema:** cada plugin traz sua própria tabela de tradução, em `node_modules/@quartz-community/`. Não há opção na configuração para mudar esses textos, e editar `node_modules` direto se perde a cada `npm ci` (que a publicação faz).
- **Solução:** ferramenta `patch-package`. Os remendos ficam na pasta `patches/` (versionada no Git) e são reaplicados automaticamente depois de cada instalação.
- **Exceções às regras do projeto, autorizadas pelo coordenador:** (1) o conteúdo de `node_modules` passa a ser alterado por meio dos arquivos de remendo; (2) mudanças em `package.json`.
- **Alterado:**
  - `package.json`: dependência `patch-package` (^8.0.1), em `dependencies` para a instalação nunca falhar se as ferramentas de desenvolvimento forem puladas, e script `"postinstall": "patch-package"`.
  - `package-lock.json`: 22 pacotes novos (os do patch-package); 7 pacotes existentes perderam a marca "dev" por passarem a ser usados também por ele. Nenhuma versão existente mudou.
  - `patches/`: cinco arquivos, um por plugin e versão (o nome carrega a versão).
- **Textos (sempre nas DUAS cópias de cada plugin, `dist/index.js` e `dist/components/index.js`):**

  | Plugin (versão) | Antes (pt-BR) | Depois (pt-BR) |
  |---|---|---|
  | graph 1.0.1 | Visão de gráfico | Grafo conceitual |
  | backlinks 1.0.1 | Backlinks | Verbetes que remetem a este |
  | backlinks 1.0.1 | Sem backlinks encontrados | Nenhum verbete remete a este |
  | tag-page 1.0.1 | N items com esta tag | N itens com esta tag |
  | folder-page 1.0.1 | N items neste arquivo | N itens neste arquivo |
  | note-properties 1.1.0 | (não tinha pt-BR; aparecia "Properties") | Propriedades (entrada pt-BR acrescentada e registrada no mapa de idiomas) |

- **Regra seguida:** só o bloco pt-BR foi tocado. Nenhum texto em inglês foi alterado; no `note-properties`, que só tinha tabela em inglês, foi ACRESCENTADA uma entrada pt-BR. Conferido nos arquivos de remendo: só há linhas pt-BR mudadas ou acrescentadas, mais uma vírgula na linha do mapa de idiomas.
- **Não aparecem na tela hoje (conferido só nos arquivos):** "Nenhum verbete remete a este" (o plugin esconde o bloco de backlinks quando não há links, que é o padrão) e "itens neste arquivo" (o conteúdo não tem pastas).
- **Fora da fase 1, pendentes (texto fixo no código, igual em qualquer idioma):** "Global Graph" (item 3), "Tag suggestions" (7), "Search results" (8) e as mensagens da busca ("No results.", "Try another search term?" etc.).
- **Manutenção:** se a versão de um desses plugins for atualizada, o remendo falha com aviso na instalação (não muda texto em silêncio) e precisa ser refeito. Por isso o nome do arquivo tem a versão.

### Teste do ciclo completo (feito no computador do coordenador)

1. Apagada a pasta `node_modules`.
2. `npm ci`: terminou sem erro e o `postinstall` aplicou os cinco remendos (backlinks, folder-page, graph, note-properties, tag-page, todos com ✔).
3. `npx quartz plugin install --from-config`: respondeu "All configured plugins are already installed" e não desfez nenhum remendo (conferido nos arquivos depois).
4. `npx quartz build` (`locale: pt-BR`): sem erro, 94 arquivos. No HTML gerado, na página de verbete: "Grafo conceitual", "Verbetes que remetem a este", "Propriedades", "Início" e "Código-fonte" presentes; "Visão de gráfico", "Backlinks", "Properties", "Home" e "Discord" ausentes. Nas páginas de tag: "itens com esta tag" (e nenhum "items").
5. Teste com `locale: en-US` (trocado só para o teste, gerado numa pasta temporária fora do projeto e depois restaurado para `pt-BR`): a interface ficou em inglês ("Graph View", "Backlinks", "Properties", "items with this tag", "Created with", "Table of Contents"), e nenhum texto novo em português apareceu nos plugins. Únicas exceções, esperadas: "Início" e "Código-fonte" (ver a entrada acima).
- **Observação:** durante `npm install` e `npm ci`, o npm avisou que dois pacotes (`@parcel/watcher` e `esbuild`) têm scripts de instalação ainda não aprovados pelo recurso `allowScripts`. O build funcionou normalmente; fica anotado caso algum dia apareça erro ligado a esses pacotes.
- **Ainda não feito:** ver a página no navegador depois da fase 1 (conferido apenas no HTML gerado), e testar a publicação real no GitHub Actions, que só acontece depois do commit e do push.

## 2026-11-08. Grafo: cores e formas por tipo de verbete, legenda e fim das bolinhas de tag (patch-package)

- **Pedido:** colorir os nós do grafo por tipo de verbete (tag do cabeçalho), com legenda visível, sem depender só da cor.
- **TIPOLOGIA PROVISÓRIA. Depende de confirmação da coordenação.** Os quatro tipos abaixo, as cores e as formas são propostas de trabalho e podem mudar (ver "Como mudar a tipologia depois").

  | Tag | Legenda | Cor | Forma | Contraste no fundo claro (#faf8f8) / escuro (#161618) |
  |---|---|---|---|---|
  | `conceito` | Conceitos e categorias | azul #0072B2 | círculo | 4,9 / 3,5 |
  | `tema` | Temas | vermelhão #D55E00 | quadrado | 3,7 / 4,7 |
  | `referencia` | Referências | verde #009E73 | triângulo | 3,2 / 5,3 |
  | `interface` | Interfaces | violeta #9B59B6 | losango | 4,4 / 3,9 |
  | (sem tipo) | Sem tipo | cinza #767676 | círculo | 4,3 / 4,0 |

  Todos acima de 3 para 1 (mínimo da norma de acessibilidade para elementos gráficos). Paleta de base: Okabe-Ito (própria para daltonismo), com o violeta no lugar do rosa, que falhava no fundo claro. A informação não depende só da cor: cada tipo tem forma própria e há legenda com os nomes.
- **Decisões tomadas, todas autorizadas pelo coordenador:**
  1. **Caminho do código:** patch-package (como na fase 1), em vez de plugin próprio. Se vierem muitas outras personalizações do grafo, vale reavaliar um plugin próprio.
  2. **Tabela de tipos num único lugar:** `quartz/styles/custom.scss`, em variáveis `--tipo-N-tag`, `-rotulo`, `-cor`, `-forma`. O grafo e a legenda leem essas mesmas variáveis. O bloco tem comentário explicando como editar.
  3. **Bolinhas de tag removidas** (`showTags: false`, no grafo local e no global). **Motivo:** elas criavam ligações entre verbetes que não foram definidas pela coordenação. O grafo passa a mostrar só as ligações escritas nos textos (links `[[ ]]`).
  4. **Páginas de tag escondidas no grafo global.** Sem esse ajuste, com `showTags: false` as páginas `tags/...` apareceriam como círculos soltos.
  5. **Página aberta:** indicada por um anel ao redor do nó. **Páginas visitadas** deixam de mudar de cor.
  6. **Verbete com mais de uma tag de tipo:** vale a primeira escrita no cabeçalho; o navegador registra um aviso no console com o nome do verbete.
  7. **Verbete sem tag de tipo, ou com tag desconhecida:** círculo cinza. "Sem tipo" entra na legenda só quando existe algum verbete assim naquele grafo (os quatro tipos aparecem sempre), para o aviso de erro de cadastro não virar ruído.
  8. **Raio mínimo de 5 pixels** (variável `--grafo-raio-minimo`), porque com o raio original (cerca de 3) quadrado, triângulo e losango se confundem. O tamanho continua crescendo com o número de ligações.
  9. **Página inicial:** círculo cinza oco, sem entrada na legenda (não é verbete). Páginas de tag, se aparecerem, têm o mesmo visual.
  10. **Legenda:** montada pelo próprio grafo a partir da tabela. No grafo pequeno fica embaixo da caixa; no grafo global, no canto inferior esquerdo. A aparência está em `custom.scss`.
- **Exceção às regras do projeto, autorizada pelo coordenador só para este remendo:** edição de `node_modules/@quartz-community/graph` (o código foi gerado por script e aplicado ali; o resultado foi capturado com `npx patch-package @quartz-community/graph`).
- **Alterado:**
  - `patches/@quartz-community+graph+1.0.1.patch`: **substituído**; o novo contém também a troca de título "Grafo conceitual" da fase 1. A cópia do remendo anterior foi guardada fora do projeto antes. Os outros quatro arquivos de `patches/` (backlinks, folder-page, note-properties, tag-page) **não mudaram** (conferido pelas somas de verificação SHA-256 antes e depois).
  - `quartz/styles/custom.scss`: tabela de tipos e estilo da legenda.
  - `quartz.config.yaml`: opção `showTags: false` no plugin do grafo (local e global). O restante do arquivo não mudou.
  - `content/Verbete Teste 04, 07, 08, 10, 11 e 12.md`: **só o valor de `tags`** no cabeçalho. Texto e links `[[ ]]` intocados (conferido pelo diff: exatamente 6 linhas trocadas). Conteúdo fictício. Nova distribuição: conceito = 01, 02, 05; tema = 03, 06, 09; referencia = 04, 07, 10; interface = 08, 11, 12.
- **Como mudar a tipologia depois (esforço):** renomear rótulo ou trocar cor/forma: mudar um valor em `custom.scss` (cerca de 1 minuto). Acrescentar tipo: copiar um bloco de 4 linhas e mudar o número (cerca de 5 minutos). Remover tipo: apagar o bloco. Não é preciso mexer no remendo nem refazê-lo. Única exceção: uma forma nova, além de círculo, quadrado, triângulo e losango, exige alterar o remendo. Depois de qualquer mudança, é preciso gerar o site de novo (`npx quartz build`). Lembrar de mudar também as tags dos verbetes quando um tipo for renomeado.
- **Testes feitos:**
  - Sintaxe do código do grafo (`node --check`), no arquivo do plugin e no script embutido: sem erro.
  - Lógica de classificação testada isoladamente (sem tag, tag desconhecida, duas tags de tipo, maiúsculas e espaços, página inicial, páginas de tag, buraco na numeração dos tipos): resultado como combinado.
  - Navegador (`quartz build --serve`): grafo pequeno e grafo global com as quatro formas e cores, legenda com os quatro nomes (sem "Sem tipo"), anel na página aberta, página inicial como círculo oco, nenhuma bolinha de tag, modos escuro e claro.
  - Ciclo completo da fase 1: `node_modules` apagado, `npm ci` (os cinco remendos aplicados, todos com ✔), `npx quartz plugin install --from-config` ("All configured plugins are already installed", remendo intacto), `npx quartz build` em pt-BR (textos da fase 1 presentes; sem erro, 96 arquivos) e em en-US (gerado em pasta temporária, depois configuração restaurada e conferida byte a byte): interface em inglês, nenhum texto novo em português nos plugins.
- **Limitação conhecida (en-US):** os rótulos da legenda ficam em `custom.scss` e portanto em português mesmo se o `locale` for en-US; numa futura versão em inglês a tabela terá de ser separada por idioma (mesma situação de "Início" e "Código-fonte").
- **Não testado:** celular (a conferência será no site publicado, depois do merge) e a publicação real no GitHub Actions.
- **Manutenção:** se o plugin `@quartz-community/graph` for atualizado, o remendo falha com aviso na instalação e precisa ser refeito (o nome do arquivo carrega a versão).

### PENDÊNCIA: páginas institucionais não são verbetes

- A página inicial e as futuras páginas institucionais (apresentação, equipe, critérios editoriais, nota sobre o uso de IA) **não são verbetes** e hoje apareceriam no grafo como "Sem tipo" (cinza cheio), o que faria o aviso de erro de cadastro perder o sentido. A página inicial já é tratada como círculo oco; as outras ainda não.
- **Investigação (sem alterar nada):**
  - O plugin do grafo não tem opção para excluir páginas por pasta ou por campo do cabeçalho. A opção `removeTags` só vale para bolinhas de tag. O índice que o grafo lê (`contentIndex.json`) só traz título, links, tags e texto.
  - O Quartz tem `unlisted: true` no cabeçalho (plugin `unlisted-pages`, já instalado e ativado neste projeto): a página continua publicada e acessível pelo endereço, mas some do grafo, **e também da busca, do explorador, dos "verbetes que remetem a este", das listas de tag, do mapa do site e do RSS**. Serve se a página puder ficar fora da busca.
  - `ignorePatterns` (por pasta ou nome) tira a página do site por completo: não serve.
  - Para sumir **só do grafo**, sem perder a busca, seria preciso uma regra no remendo (por exemplo, uma tag `institucional` ou uma pasta própria, que o grafo trataria como página oca ou oculta). É uma alteração pequena, na mesma linha da tabela de tipos. Decisão da coordenação.

### Observações fora do escopo (anotadas, não corrigidas)

- No modo escuro, as linhas de ligação do grafo são muito pouco visíveis. Isso já ocorre no site publicado, com o grafo original (cor `lightgray` do tema sobre fundo escuro). Vale avaliar quando se tratar do destaque das relações.
- O grafo local casa a página aberta com o índice pelo endereço em minúsculas. Abrir `/Verbete-Teste-01` (com maiúsculas) mostra um nó solto; `/verbete-teste-01` funciona. Nos links do site os endereços já são em minúsculas.
- O item 2 do pedido (destaque por clique pressionado e toque longo) segue aguardando o teste do coordenador no celular, no site publicado.

## 2026-11-08. Grafo: linhas sempre visíveis; coluna da direita só com o grafo

- **Pedido:** (1) ver as linhas (arestas) do grafo sem precisar passar o cursor, para não parecer que há nós soltos; (2) deixar na área da direita só o grafo, sem "Sumário" e sem "Verbetes que remetem a este".
- **Diagnóstico das linhas:** elas **já eram desenhadas**, mas na cor `lightgray` do tema (cinza #282828 no modo escuro, #e5e5e5 no claro), quase igual ao fundo. Acontece também no grafo original, no site publicado. Não era erro de dados: as ligações existem e o grafo as mostra quando a cor tem contraste.
- **Alterado (linhas):**
  - Remendo `patches/@quartz-community+graph+1.0.1.patch`, **regerado** (substitui o anterior; contém também a tipologia e a troca de título). A linha comum passa a ter cor e espessura lidas de variáveis. Ao passar o mouse, as linhas do nó em foco ficam na cor de texto do tema (`--dark`), para se distinguirem das demais (antes ficavam em `--gray`, que no modo claro ficaria mais fraca que as linhas comuns). Os outros quatro arquivos de `patches/` não mudaram (conferido por SHA-256 antes e depois).
  - `quartz/styles/custom.scss`: `--grafo-cor-ligacao: #808080` (contraste 3,7 no fundo claro #faf8f8 e 4,6 no escuro #161618) e `--grafo-largura-ligacao: 1.5`. Ficam junto da tabela de tipos e podem ser ajustadas ali.
- **Alterado (coluna da direita), em `quartz.config.yaml`:**
  - plugin `table-of-contents` (Sumário): `enabled: false`.
  - plugin `backlinks` ("Verbetes que remetem a este"): `position` de `right` para `afterBody`, isto é, **movido para o fim do verbete, não removido**.
- **DECISÃO A CONFIRMAR PELA COORDENAÇÃO (backlinks):** a súmula exige, na página de cada verbete, "verbetes relacionados (links bidirecionais)" e uma alternativa textual ao grafo. Por isso a lista foi movida para o fim do texto, em vez de apagada. Para remover de vez: `enabled: false` no plugin `backlinks`. Para devolver à direita: `position: right`.
- **Observação:** sem o Sumário, verbetes longos perdem a navegação por títulos. Para devolver: `enabled: true` no plugin `table-of-contents`.
- **Testes feitos:** `node --check` do plugin e do script embutido (sem erro); navegador: linhas visíveis no grafo global (modo escuro), coluna da direita só com o grafo, "Verbetes que remetem a este" no fim da página com os quatro verbetes que citam o 01; ciclo completo (`npm ci` com os cinco remendos ✔, `plugin install --from-config` sem desfazer nada, arquivo do plugin idêntico byte a byte antes e depois da reinstalação; build em pt-BR e em en-US com os textos esperados, configuração restaurada e conferida). Não testado: celular, modo claro depois desta mudança e a publicação no GitHub Actions.
- **Observação sobre os testes:** durante os testes havia outro servidor de pré-visualização ocupando a porta 8080 (processo `node` sem janela, de testes anteriores); os testes desta etapa usaram a porta 8081. Ao final, as portas 8080, 8081, 3001 e 3002 estavam livres. Se `npx quartz build --serve` avisar que uma porta está em uso, é um servidor antigo: encerre-o ou use `--port 8081 --wsPort 3002`.

## 2026-11-08. Sumário de volta; grafo, nós e títulos maiores

- **Pedido:** manter "Verbetes que remetem a este" no fim da página (decisão confirmada); devolver o "Sumário" à coluna da direita; aumentar o grafo da coluna da direita e o grafo global, porque nós e títulos estavam pequenos demais para ler. As linhas visíveis foram aprovadas como estão.
- **Decisão confirmada pelo coordenador:** "Verbetes que remetem a este" fica no fim da página (`position: afterBody`). Fica registrado que isso atende à exigência da súmula de verbetes relacionados em cada verbete.
- **Alterado, `quartz.config.yaml`:**
  - plugin `table-of-contents` (Sumário): `enabled: true` de novo, na coluna da direita abaixo do grafo (posição e prioridade originais).
  - plugin do grafo: `fontSize: 0.9` (antes 0,6) nos grafos local e global, o que leva os títulos de cerca de 8 para cerca de 12 pixels no grafo pequeno e de 10 para 15 no global; `linkDistance: 65` (antes 30) só no grafo pequeno, para os nós ocuparem a caixa em vez de ficarem agrupados no centro.
- **Alterado, `quartz/styles/custom.scss`** (valores em variáveis no bloco `:root`, perto da tabela de tipos):
  - `--grafo-raio-minimo` de 5 para **7** (nós maiores; os símbolos de cada tipo ficam mais legíveis).
  - `--grafo-altura-local: 360px` (antes 250) em telas de 1200 pixels ou mais; `--grafo-altura-local-estreita: 300px` abaixo disso.
  - Coluna da direita mais larga em telas grandes: `--coluna-direita-1200: 380px` (de 1200 a 1599 pixels) e `--coluna-direita-1600: 460px` (de 1600 pixels em diante); antes 320px. A coluna da esquerda não mudou. Os pontos de corte 1200 e 1600 estão escritos nas regras `@media` porque variáveis não funcionam dentro dessa condição.
  - Grafo global (tela cheia): 90% da largura e da altura da tela (antes 80%).
  - Em telas estreitas (tablet e celular), o limite de 24rem de altura que o Quartz põe nos blocos da coluna lateral foi retirado, porque com o grafo maior e a legenda o limite sobreporia a legenda (31 pixels de excesso medidos no celular simulado).
- **Remendo do grafo:** **não mudou nesta etapa** (nenhuma alteração em `patches/` além das já registradas). Por isso o ciclo `npm ci` não foi repetido.
- **Testes feitos (navegador, `quartz build --serve`, porta 8081):** em 1500 pixels de largura, coluna da direita com 380px, grafo de 316 por 360 pixels, "Grafo conceitual" seguido de "Sumário"; grafo global com 90% da tela, nós maiores e conectados; tamanho do texto dos títulos lido direto do código em execução (13,5 antes da divisão pelo zoom, contra 9 antes); em 375 pixels (celular simulado) e 900 pixels (tablet): sem rolagem horizontal, legenda inteira e sem sobreposição com o rodapé.
- **Não testado:** o aspecto do título ao passar o mouse sobre um nó (o painel de teste não repassa o movimento do mouse ao grafo; só o tamanho foi conferido no código), toque no celular real, e a publicação no GitHub Actions.
- **Como ajustar depois:** tamanho dos nós, altura do grafo e largura da coluna em `custom.scss` (variáveis); tamanho dos títulos em `quartz.config.yaml` (`fontSize`).

## 2026-11-08. Destaque por clique pressionado e toque longo: teste no celular (item 2)

- **Pedido original:** ao manter o clique pressionado no computador (ou passar o mouse) e ao fazer toque longo no celular, destacar o verbete e os ligados a ele, esmaecendo os demais; o clique simples continua abrindo o verbete.
- **Teste feito pelo coordenador, no celular, no site publicado** (grafo original do plugin, sem nenhuma alteração desta etapa): resultado informado como "todo correto".
- **Decisão:** **nenhuma alteração de código foi necessária para o item 2.** O comportamento do plugin original já atende ao pedido no computador e no celular. O remendo do grafo desta etapa não mexe nesse comportamento.
- **Observação:** o teste foi feito no grafo original, que ainda tinha as bolinhas de tag e o tamanho antigo dos nós. Como o remendo e o tamanho novos não alteram o tratamento de toque, não se espera diferença; vale repetir o toque longo uma vez no site publicado depois do merge, junto com a conferência das novidades.

## 2026-11-08. Sumário removido da coluna da direita (de novo)

- **Pedido:** a coordenação decidiu que, ao menos neste momento, não quer a área "Sumário" na parte inferior da coluna da direita.
- **Alterado, `quartz.config.yaml`:** plugin `table-of-contents` (Sumário): `enabled: false` (era `true`). Só essa linha mudou.
- **Por que assim:** desligar o plugin remove o Sumário de todas as páginas sem apagar nada. A decisão é reversível.
- **Observação:** sem o Sumário, verbetes longos perdem a navegação por títulos. Para devolver: `enabled: true` no plugin `table-of-contents`. A coluna da direita fica só com o grafo; "Verbetes que remetem a este" continua no fim da página.
- **Testes feitos (navegador, `quartz build --serve`, porta 8080):** o build rodou sem erro; na página do Verbete Teste 01 a coluna da direita tem apenas "Grafo conceitual", sem nenhum elemento ou texto de Sumário (conferido no código da página). Nenhum arquivo de `content/` ou de `patches/` foi tocado.
- **Não testado:** celular real e publicação no GitHub Actions (nada foi enviado ao GitHub).

## 2026-11-08. Grafo: rótulos aparecem com menos zoom e distribuição dos nós mais aberta

- **Pedido:** como no Obsidian, os títulos dos nós deveriam aparecer conforme o zoom (e não só com zoom máximo), e nós e linhas deveriam ficar melhor distribuídos.
- **Diagnóstico (lido no código do plugin, sem alterá-lo):**
  - **Rótulos:** a transparência do título é `(zoom × opacityScale − 1) / 3,75`. Com `opacityScale` = 1 (padrão), o título é invisível em zoom 1 ou menos e só chega a 80% no zoom máximo (4). Por isso só aparecia "com muito zoom".
  - **Distribuição:** no grafo global, o padrão do plugin liga uma força radial (`enableRadial: true`) que empurra todos os nós para um anel. A repulsão padrão também é fraca. O Obsidian não usa anel.
- **Alterado, só em `quartz.config.yaml` (opções do plugin do grafo; o remendo em `patches/` não mudou):**
  - grafo local: `opacityScale: 3`
  - grafo global: `opacityScale: 3`, `enableRadial: false`, `repelForce: 1.2` (era 0,5), `centerForce: 0.1` (era 0,2), `linkDistance: 80` (era 30)
- **Efeito do `opacityScale: 3`:** título a ~13% de opacidade em zoom 0,5; ~53% em zoom 1; 100% a partir de zoom ~1,6.
- **Como ajustar depois:** mais rótulos visíveis = subir `opacityScale`; menos = descer. Nós mais afastados = subir `repelForce` ou `linkDistance`; mais agrupados = descer. Para voltar ao comportamento anterior: apagar as linhas acrescentadas.
- **Limitação conhecida (não resolvida):** ao abrir o grafo, os títulos ainda começam invisíveis e aparecem na primeira rolada de zoom, porque o código do plugin só calcula a transparência quando ocorre um evento de zoom. Resolver exige alterar o remendo do grafo (uma linha), o que depende de autorização da coordenação.
- **Testes feitos (navegador, `quartz build --serve`, porta 8080, janela de 1500 px):** grafo global sem anel, nós espalhados de modo irregular; com uma rolada de zoom, todos os títulos aparecem. Os valores foram conferidos no `data-cfg` do grafo local em execução.
- **Não testado:** grafo local com zoom (o painel de teste corta a coluna da direita), celular real (pinça), efeito com o número real de verbetes (hoje são 12 de teste; a repulsão pode precisar de novo ajuste com 100 ou mais), e publicação no GitHub Actions.
