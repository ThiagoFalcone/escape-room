# Relatório de Resgate
- Equipe: Thiago, João e Gabriel
- Branch de trabalho: resgate/discente-thiago-joao-gabriel
## Diagnóstico
- Porta 1 (compilação): `a70ee84` chamou `new Mercadoria(...)` sem o endereço e `repository.gravar` (método inexistente); `64f88f6` apagou `Validador.java`, usado por `EntregaService`. Mensagens exatas do Maven na seção "Erro de compilação (Etapa 3)".
- Porta 2: arquivo excluído = `util/Validador.java` (commit `64f88f6`).
- Porta 3: `9a6d3b0` trocou `&&` por `||` e `SENHA.equals` por `senha == SENHA`: com `||`, qualquer senha passa se o usuário for `admin`; e `==` compara referência, não conteúdo.
- Porta 4: `0cd80f6` reduziu o README a uma frase.
- Porta 5: `6572d8a` versionou `config/application.properties` com `db.password` e `api.token`.
## Investigação inicial (Etapa 2)
- Branches (`git branch -a`): 8 — `main`, `release-dev`, `backup-antigo`, `docs-readme`, `feature-cadastro`, `hotfix-login`, `teste-login-descartavel` e a branch de resgate da equipe (`resgate/discente-thiago-joao-gabriel`).
- Branch inicialmente ativa: `release-dev` (topo `799c5ec`, "chore: adicionar instrucoes do resgate", que traz o `DESAFIO.md`).
- Tags: `v1.0.0-funcional` (`356ec6f`) e `commit-perigoso` (`6572d8a`).
- Commits relacionados aos problemas: `64f88f6`, `a70ee84` (compilação), `9a6d3b0` (login), `0cd80f6` (README), `6572d8a` (credenciais). Autores na seção "Autores dos commits problemáticos".
- Branches/tags com versões úteis: tag `v1.0.0-funcional` e `backup-antigo` (ambas em `356ec6f`, versão que compilava e autenticava corretamente) e `docs-readme` (README com requisitos e execução, mais a seção "Fluxo recomendado").
- Branches que não resolvem nenhuma porta: `teste-login-descartavel` (aponta para `9a6d3b0`, o commit que quebrou o login), `hotfix-login` (só adiciona `NOTA-HOTFIX.txt`, sem corrigir código) e `feature-cadastro` (só adiciona a validação de `status` em `EntregaService`).
## Erro de compilação (Etapa 3)
Reproduzido com `mvn clean package` no commit `64f88f6`, extraído com `git archive 64f88f6` para uma pasta temporária (a branch de trabalho não foi alterada; o `src/` é idêntico ao de `release-dev`). `...` abrevia o caminho absoluto:
```
[ERROR] COMPILATION ERROR :
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[6,28] package br.edu.entregas.util does not exist
[ERROR] .../EntregaService.java:[14,9] cannot find symbol
  symbol:   variable Validador
  location: class br.edu.entregas.service.EntregaService
  (idem nas linhas [15,9], [16,9] e [17,9])
[ERROR] .../EntregaService.java:[19,33] constructor Mercadoria in class br.edu.entregas.model.Mercadoria cannot be applied to given types;
  required: long,java.lang.String,java.lang.String,double,double,java.lang.String,br.edu.entregas.model.Endereco
  found:    long,java.lang.String,java.lang.String,double,double,java.lang.String
  reason: actual and formal argument lists differ in length
[ERROR] .../EntregaService.java:[20,19] cannot find symbol
  symbol:   method gravar(br.edu.entregas.model.Mercadoria)
  location: variable repository of type br.edu.entregas.repository.MercadoriaRepository
[INFO] 7 errors
[INFO] BUILD FAILURE
```
| Linhas de `EntregaService.java` | Causa | Commit (autor) | Correção |
|---|---|---|---|
| 6, 14–17 | `util/Validador.java` foi apagado | `64f88f6` (Igor Reis) | `git revert 64f88f6` → `e9cd868` |
| 19–20 | construtor chamado sem `endereco` e `repository.gravar` (o método existente é `salvar`) | `a70ee84` (Henrique Nunes) | `git revert a70ee84` → `bb941b6` |
## Comandos Git utilizados
- `git status` e `git branch -a`: estado do repositório e lista de branches.
- `git log --oneline --all --graph`: mapear histórico e branches.
- `git show <hash>`: ver o que cada commit alterou.
- `git diff v1.0.0-funcional -- <arquivo>`: comparar `LoginService.java` e `EntregaService.java` com a versão funcional.
- `git log --diff-filter=D --summary`: localizar a exclusão de `Validador.java`.
- `git log --all -- README.md` e `git show docs-readme:README.md`: localizar versões do README.
- `git log --all --oneline -- config/application.properties` e `git show commit-perigoso`: auditoria do segredo.
- `git switch -c resgate/discente-thiago-joao-gabriel` (equivalente a `git checkout -b`): branch de trabalho.
- `git revert <hash>`: desfaz um commit gerando novo commit rastreável (64f88f6, a70ee84, 9a6d3b0, 0cd80f6, 6572d8a).
- `git merge --no-ff`: integrar o resgate na `main`.
- `git archive 64f88f6`: extrair o commit quebrado para uma pasta temporária e reproduzir o erro do Maven sem mexer na branch.
- `mvn clean package` e `java -cp target/classes br.edu.entregas.Main`: compilar e executar.
## Commits relevantes
- Revertidos: `64f88f6`, `a70ee84`, `9a6d3b0`, `0cd80f6`, `6572d8a`.
- O segredo continua no histórico (`6572d8a`); na versão atual foi removido e `config/application.properties` entrou no `.gitignore`. As credenciais devem ser rotacionadas; limpar o histórico exigiria `git filter-repo`.
## Validação final
Executada na `main` (commit `248b291`) em 05/10/2026, com Maven 3.8.8 e JDK 21.0.7. `git diff v1.0.0-funcional main --stat -- src README.md` não retorna nada: o código e o README da `main` são idênticos aos da tag funcional.
- Compilação: `mvn clean package` → `BUILD SUCCESS` (7 arquivos compilados, `projeto-entregas-1.0.0.jar` gerado em `target/`, "No tests to run", 19,2 s).
- Login (`java -cp target/classes br.edu.entregas.Main`): `admin/12345678` → "Acesso autorizado. Mercadorias cadastradas:"; `admin/errada` → "Acesso negado."; `x/12345678` → "Acesso negado.".
- Cadastro: `#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000`.
- README: instruções de compilar/executar restauradas (`mvn clean package` e `java -cp target/classes br.edu.entregas.Main`).
- Segurança: `git grep -i "senha123\|token" -- config` não retorna nada e `git ls-files config` está vazio na versão atual.

## Autores dos commits problemáticos
- `9a6d3b0` e `6572d8a`: Felipe Rocha (login quebrado e credenciais versionadas).
- `0cd80f6`: Gustavo Melo (README reduzido).
- `a70ee84`: Henrique Nunes (cadastro sem endereço, chamada a método inexistente).
- `64f88f6`: Igor Reis (exclusão de `Validador.java`).

## Recuperação e comparações
- Arquivo excluído: `git log --diff-filter=D --summary` localizou `Validador.java` (`64f88f6`, Igor Reis); recuperado via `git revert 64f88f6` (`e9cd868`). A outra exclusão listada, `792e2d1`, é o nosso revert que removeu o arquivo com segredos.
- Comparação de arquivo: `git diff v1.0.0-funcional -- src/main/java/br/edu/entregas/service/LoginService.java` (feito antes da correção; `git diff v1.0.0-funcional origin/release-dev -- <mesmo arquivo>` ainda mostra a diferença).
- README: recuperado com `git revert 0cd80f6` (`149f56a`), sem reescrever; o conteúdo ficou idêntico ao de `git show 356ec6f:README.md` (tag `v1.0.0-funcional`). A branch `docs-readme` (`git show origin/docs-readme:README.md`) tem o mesmo texto mais a seção "Fluxo recomendado", e não foi integrada.

## Integração
- `git merge --no-ff` na `main`: commits `73e69bf` e `b204842` (visíveis em `git log --oneline --all --graph --decorate`).

## Pergunta de encerramento
O controle de configuração registra quem mudou o quê, quando e por quê. Isso reduz riscos porque cada regressão é localizável e reversível (`git revert`) sem perder evidências; facilita auditorias porque autores, hashes e diffs são rastreáveis (foi assim que achamos o segredo em `6572d8a`); e permite recuperar o projeto porque tags como `v1.0.0-funcional` e branches guardam versões boas. Ressalva: segredos já commitados permanecem no histórico, então é preciso rotacioná-los.
