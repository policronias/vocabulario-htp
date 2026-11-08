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
