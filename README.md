# Regras da Comunidade

Nesta página será postada um conjunto de padrões necessários para ter um projeto perfeito, um projeto delicioso, um projeto nos trinques!

## Padrão de Branch
Todas as branches precisam estar no padrão [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).
**Exemplo**: `feat/TASK00-login-page`
Substituir `TASK00` pelo código da **issue**, geralmente é a sigla do projeto seguida de um número de dois dígitos.


## Estilo do Código
Para projetos javascript, utilizar o **ESLint** e o **Prettier**, além de colocar o seguinte código no seu arquivo `.vscode/settings.json`:
```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "eslint.validate": ["javascript", "typescript"],
  "javascript.suggestionActions.enabled": false,
  "typescript.suggestionActions.enabled": false,
  "typescript.preferences.quoteStyle": "single",
  "javascript.preferences.quoteStyle": "single"
}
```

## Git Ignore
Adicionar o seguinte código no seu arquivo `.gitignore`:
```bash
# dependencies
/node_modules
package-lock.json

# testing
/coverage
/reports

# production
/build

# stryker temp files
/stryker-tmp
```

## Automatização de Commits
Utilizar o **husky** e o **commitlint**, caso queira, usar o **commitizen** também, mas ele é opcional.
### Husky
Rodar no terminal:
```bash
npm install --save-dev husky
npx husky install
```
### Commitlint
Rodar no terminal:
```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

Criar o arquivo `commitlint.config.js` com:
```javascript
module.exports = {
    extends: ['@commitlint/config-conventional'],
};
```

Criar o arquivo `commit-msg` na pasta `.husky/` com:
```bash
npx commitlint --edit $1
```

### Commitizen
Rodar no terminal:
```bash
npm install --save-dev commitizen cz-conventional-changelog
```

Adicionar depois das dependencias no `package.json`:
```json
{
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```
Para criar o commit:
```bash
npx cz
```

## Layout de Merge Request
O título do merge request segue o mesmo padrão de commit, deve seguir o seguinte padrão:
`feat(TASK00): Atividade`
Onde `feat` é tipo, `TASK00` é o código da issue e `Atividade` o resumo do que foi feito.

Também é interessante inserir uma tag para cada tipo:
**enhancement**: feat (coisa nova);
**bug**: fix (arrumando algo);
**documentation**: docs (documentação).

Adicione a seguinte informação no arquivo `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md:`

```markdown
# Descrição

<!--
Descreva brevemente as alterações realizadas neste pull request. Se houver uma issue associada, mencione-a aqui.
-->

Resolve o problema de desempenho na página de login (#123).

<!--
Este 123 é o código que aparece na url da issue, exemplo:
https://github.com/magas-xlr/projeto/issues/123
-->

# Mudanças

<!--
Lite o que foi feito.
-->

- Alterações no componente `login.component.ts` para otimizar o tempo de carregamento;
- Atualização do serviço de autenticação para utilizar um novo endpoint.

# Passos para Testar

<!--
Enumere como testar a mudança.
-->

1. Certifique-se de que o projeto está rodando localmente;
2. Navegue até a página de login;
3. Verifique se o tempo de carregamento foi reduzido;
4. Teste a funcionalidade de login com credenciais válidas e inválidas.

# Checklist

- [ ] Testes foram escritos e passam com sucesso;
- [ ] A esteira passou com sucesso.

# Prints
<!--  
Adicione somente se for algo visual.
-->

Não se aplica

```

## Testes Unitários, de Mutação e End-to-End
Deve escrever testes unitários e de mutação para cada funcionalidade, cobrindo 100% dos casos.
O teste unitário pode escolher qualquer biblioteca, porém os de mutação precisam ser feitos com o **stryker** e os E2E com **cypress*.

## Documentação
Se for uma API, será necessário implementar o **swagger**, caso seja uma biblioteca, deve colocar a forma de importação do pacote. Se for uma aplicação frontend, deve ter **prints** do aplicativo.
Para todos os tipos de projetos, precisa ter os seguintes tópicos no `README.md`:
- Título;
- Badges de tag, framework e cobertura de testes (shields.io);
- Descrição; <!-- Resumo -->
- Funcionalidades; <!-- O que o projeto faz -->
- Tecnologias; <!-- O que foi usado -->
- Como rodar. <!-- Os comandos necessários -->

## Automatização do Deploy
Todo deploy precisa gerar uma tag e um pacote.
Para aplicações, precisa utilizar o **docker**.
Seja ela uma aplicação ou uma biblioteca, precisa publicá-lo no **Github Packages**.
Será necessário adicionar também a cobertura de código no **codecov**.
Para gerar uma tag automaticamente, precisa utilizar o **standard-version** no **husky** para cada push.
Além disso, deve-se adicionar isso no seu arquivo `.github/workflows/release.yml`:
```yml
name: Publish Product

on:
  push:
    branches:
      - main

permissions:
  contents: write
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Build project
        run: npm run build

      - name: Run mutation tests
        run: npm run mutation

      - name: Run unit tests
        run: npm test --coverage

      - name: Upload Coverage Reports to CodeCov
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-coverage
          fail_ci_if_error: true

      - name: Log in to GitHub Package Registry
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Configure Git user
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "github-actions[bot]@users.noreply.github.com"

      # Coloque aqui o passo de criação de pacote no Github Package (docker, package.json, pom.xml, etc)

      - name: Create Tag
        run: |
          VERSION=$(node -p "require('./package.json').version")
          git tag "v$VERSION"
          git push origin "v$VERSION"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Para projetos embrionários, não precisam de muito dessas coisas aqui citadas, deve ser considerado apenas em projetos próximos ao passo de finalização (quando acaba todas as issues).