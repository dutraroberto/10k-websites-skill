# 10K Websites (skill para Claude Code)

Skill que constrói e publica um site cinematográfico guiado pelo scroll: um vídeo gerado por IA avança e volta conforme a rolagem, com legendas, seções reais e uma chamada para ação.

As imagens e vídeos podem ser gerados por dois caminhos, à escolha do usuário:

- **Higgsfield:** o Claude gera tudo pela CLI do Higgsfield, mostrando o preço de cada etapa antes de gastar créditos.
- **Manual:** o Claude escreve os prompts prontos para colar; você gera as imagens no ChatGPT e os vídeos no Google Flow (Veo).

## Instalar

Em um projeto:

    mkdir -p .claude/skills
    cp -r 10k-websites .claude/skills/

Para todos os projetos:

    cp -r 10k-websites ~/.claude/skills/

Depois, no Claude Code, digite `/10k-websites`.

## Requisitos

- ffmpeg e Node.js (a skill instala se faltarem)
- Caminho Higgsfield: `npm i -g @higgsfield/cli` e `higgsfield auth login`
- Caminho manual: conta no ChatGPT e acesso ao Google Flow (labs.google/flow)
