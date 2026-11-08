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
