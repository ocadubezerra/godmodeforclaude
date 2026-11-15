# Changelog

## 1.1.0 (2026-11-15)
- Redação revisada e simplificada nas instruções do orquestrador, do executor, do `CLAUDE.md`, do modelo de skill e da seção do `/goal`. Configurações, caminhos e argumentos sem alteração.
- Esclarece que `Agent(executor)` limita a delegação porque o orquestrador roda como sessão principal, e que `/effort max` vale só para a sessão.

## 1.0.0 (2026-11-15)
- Primeira versão: orquestrador (Opus, xhigh), executor (Haiku), `CLAUDE.md`, `.claude/settings.json`, `memoria/plano.md`, modelo de skill que se aprimora e guia do `/goal`.
- Nunca sobrescreve arquivos que já existem (acrescenta, mescla ou pergunta).
- Argumentos opcionais: `60` (compactação aos 60%) e `high` (esforço high).
