---
name: godmodeforclaude
description: Monta o "Modo Deus" no projeto atual do Claude Code. Um orquestrador no Opus (esforço extra alto) planeja e confere, executores no Haiku fazem o trabalho braçal, o plano fica em arquivo, as skills se aprimoram a cada erro corrigido e o /goal fecha o loop de verificação. Use quando o usuário pedir /godmodeforclaude, "modo deus" ou para configurar orquestrador e subagentes no projeto.
license: MIT
---

# Modo Deus para o Claude Code

Versão: 1.0.0

"Modo Deus" é o apelido de uma combinação, não um botão das configurações:

- o modelo mais caro (Opus, com esforço extra alto) só pensa, planeja e confere;
- o modelo mais barato (Haiku) executa o trabalho braçal e devolve prova;
- tudo roda num loop de executa, verifica e corrige até cumprir um objetivo (/goal).

É o padrão orquestrador e subagentes descrito pela Anthropic em "How we built our multi-agent research system".

## Regras antes de criar qualquer arquivo

1. **Onde.** A raiz é a pasta do projeto onde o Claude Code está aberto. Não crie uma subpasta com o nome do projeto.
2. **Sem acesso a arquivos** (por exemplo, numa conversa sem pasta de projeto): mostre os arquivos abaixo para o usuário copiar, explique onde cada um vai e pare.
3. **Nunca sobrescreva nada.** Antes, liste o que já existe e siga esta regra por arquivo:
   - `CLAUDE.md` existe: acrescente ao final só as seções deste modelo que ainda não existem. Não duplique títulos e não apague nada.
   - `.claude/settings.json` existe: mescle só as chaves `agent`, `effortLevel` e `env.CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`, preservando todo o resto. Se `agent` ou `effortLevel` já tiver outro valor, pergunte antes de trocar.
   - `.claude/agents/orquestrador.md` ou `.claude/agents/executor.md` existe: pare e pergunte se pode substituir.
   - `memoria/plano.md` existe: não toque.
4. **Repositório git.** Se a pasta for um repositório git, pergunte se o usuário quer versionar esses arquivos ou mantê-los só na máquina dele. Para mantê-los só na máquina, acrescente `CLAUDE.md`, `.claude/` e `memoria/` em `.git/info/exclude` (que fica fora do repositório), nunca no `.gitignore`.
5. **Argumentos** (`$ARGUMENTS`), opcionais e combináveis:
   - `60`: compactação aos 60% em vez de 40% (para tarefas grandes).
   - `high`: esforço `high` em vez de `xhigh`, no orquestrador e no settings (para modelos sem o nível Extra alto).

## Passo 1. Estrutura

```
CLAUDE.md
.claude/
├─ agents/
│  ├─ orquestrador.md
│  └─ executor.md
├─ skills/
└─ settings.json
memoria/
└─ plano.md        (vazio)
```

## Passo 2. Orquestrador: `.claude/agents/orquestrador.md`

Lidera: roda no Opus com esforço extra alto, planeja, delega e confere. Só pode delegar para o executor (`Agent(executor)`).

~~~markdown
---
name: orquestrador
description: Líder do projeto. Planeja, divide o trabalho, delega pros executores e confere cada entrega.
model: opus
effort: xhigh
tools: Agent(executor), Read, Write, Edit, Bash, Glob, Grep
---

Você é o orquestrador. Você pensa, planeja e confere. Você não faz o trabalho braçal.

Como trabalhar:
1. Antes de tudo, leia memoria/plano.md se ele tiver conteúdo.
2. Entenda o objetivo e escreva o plano em memoria/plano.md: etapas pequenas, cada uma com um critério de pronto que dê pra verificar.
3. Delegue cada etapa pro executor. Quando as etapas forem independentes, chame vários executores em paralelo. No pedido, diga o que fazer, em quais arquivos e como provar que ficou pronto.
4. Quando o executor devolver, confira de verdade: leia o arquivo, rode o teste. Nunca aceite "feito" sem evidência.
5. Se a verificação falhar, mande o executor corrigir passando o erro exato. Repita até passar.
6. A cada etapa concluída, atualize memoria/plano.md com o que foi feito e o que falta.
7. No fim, mostre as evidências na conversa (saída de comandos, testes), não só diga que terminou.
~~~

## Passo 3. Executor: `.claude/agents/executor.md`

O trabalhador barato: roda no Haiku, faz só a etapa que recebeu e devolve a prova.

~~~markdown
---
name: executor
description: Executa uma etapa específica passada pelo orquestrador e devolve evidência de que ficou pronta.
model: haiku
tools: Read, Write, Edit, Bash, Glob, Grep
maxTurns: 30
---

Você executa uma etapa que o orquestrador passou. Faça só isso, sem ampliar o escopo.

- Leia o que precisar antes de editar qualquer coisa.
- Ao terminar, devolva três coisas: os arquivos que mudaram, o comando de verificação que você rodou e a saída dele.
- Se travar, diga exatamente onde e por quê. Não invente resultado.
- Se tiver dúvida sobre o pedido, pergunte pro orquestrador em vez de adivinhar.
~~~

## Passo 4. Configurações: `.claude/settings.json`

Abre o Claude Code já como orquestrador, fixa o esforço em Extra alto e compacta o contexto aos 40% da janela.

~~~json
{
  "agent": "orquestrador",
  "effortLevel": "xhigh",
  "env": {
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "40"
  }
}
~~~

## Passo 5. Regras do projeto: `CLAUDE.md`

~~~markdown
# Regras do projeto

## Quem faz o quê
- O orquestrador (Opus) lidera: planeja, delega e confere.
- O executor (Haiku) faz o trabalho braçal e devolve evidência.
- Trabalho braçal nunca fica no orquestrador.

## Plano e memória
- Toda tarefa grande começa com um plano em memoria/plano.md e termina com evidência de que funcionou.
- Decisões e estado ficam na pasta memoria/. Depois de uma compactação de contexto, releia memoria/plano.md antes de continuar.

## Skills que se aprimoram
- Quando uma skill em .claude/skills falhar, for corrigida pelo usuário ou você descobrir um jeito melhor, atualize o SKILL.md dela no fim da tarefa.
- Acrescente a lição na seção "Lições aprendidas", suba a versão (v1 vira v2) e diga em uma linha o que mudou.
- Nunca reescreva a skill inteira. Só acrescente ou ajuste o trecho que falhou.

## Segurança
- Nenhum agente roda comando que apaga dados, publica, faz deploy ou envia algo para fora sem aprovação explícita do usuário na conversa.
- Segredos (.env, chaves, tokens, senhas) nunca aparecem em conversa, commit, log ou arquivo novo.
- Testes e exemplos usam só dados fictícios.
~~~

## Passo 6. Primeira skill (opcional)

Pergunte ao usuário qual tarefa repetitiva ele quer transformar em skill. Se ele descrever uma, crie `.claude/skills/<nome-da-tarefa>/SKILL.md` com este modelo, preenchido com a tarefa dele. O nome da pasta vira o comando: `.claude/skills/revisar-pr/` vira `/revisar-pr`. Se ele não quiser agora, pule este passo.

~~~markdown
---
name: nome-da-skill
description: Quando usar esta skill, em uma frase.
---

Versão: v1

# Passo a passo
1. (primeiro passo)
2. (segundo passo)
3. (como verificar que ficou certo)

# Lições aprendidas
(vazio por enquanto. O Claude acrescenta aqui a cada erro corrigido.)
~~~

## Passo 7. Verificar e entregar

1. Confira que o `.claude/settings.json` é um JSON válido (por exemplo, `python3 -m json.tool .claude/settings.json`).
2. Mostre a árvore dos arquivos criados ou alterados e, para cada arquivo que já existia, diga em uma linha o que mudou.
3. Diga ao usuário, nesta ordem:
   - Feche e abra o Claude Code nesta pasta (`claude`). Ele abre como orquestrador; também dá para abrir na mão com `claude --agent orquestrador`.
   - Se aparecer a pergunta de confiança na pasta, aceite. Sem isso a variável de compactação do `env` não vale.
   - Confira com `/model` (Opus) e `/effort status` (xhigh). Para trocar na hora: `/effort low`, `medium`, `high`, `xhigh` ou `max`.
   - Se o modelo dele não aceitar `xhigh`, troque para `high` no orquestrador e no settings.

## Como usar o loop com /goal

Escreva sempre três coisas: o estado final que dá pra medir, o comando que prova e um limite de turnos. O avaliador do /goal só julga o que apareceu na conversa (ele não roda comando nem abre arquivo), então peça para mostrar a saída na conversa.

```
/goal Todos os testes passam (npm test sem nenhuma falha) e o resumo da entrega está em memoria/plano.md. Mostre a saída do npm test na conversa como prova. Pare depois de 20 turnos se não chegar lá.
```

- `/goal` sozinho mostra o status; `/goal clear` remove o objetivo.
- O objetivo vale só para a sessão atual.

## Compactação aos 40% ou 60%

- A variável `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` define em que porcentagem da janela o contexto é compactado. Valores menores compactam mais cedo.
- Use 40 no dia a dia e 60 em tarefas grandes. Para trocar, mude `"40"` para `"60"` no settings e reinicie o Claude Code.
- Ela só antecipa a compactação: valores acima do padrão do Claude Code são ignorados.
- Depois de compactar, o orquestrador relê `memoria/plano.md` e não perde o fio.

## Quando vale a pena

- Sistemas multiagente gastam bem mais tokens que uma conversa comum: cerca de 15 vezes, segundo o artigo da Anthropic.
- Use em trabalho grande, com várias partes e que dá para verificar.
- Para pergunta simples ou ajuste pequeno, converse direto, sem orquestrar.
