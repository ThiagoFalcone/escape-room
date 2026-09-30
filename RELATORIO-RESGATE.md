# Relatório de Resgate
- Equipe: Thiago, João e Gabriel
- Branch de trabalho: resgate/discente-thiago-joao-gabriel
## Diagnóstico
- Porta 1 (compilação): `a70ee84` chamou `new Mercadoria(...)` sem o endereço e `repository.gravar` (método inexistente); `64f88f6` apagou `Validador.java`, usado por `EntregaService`.
- Porta 2: arquivo excluído = `util/Validador.java` (commit `64f88f6`).
- Porta 3: `9a6d3b0` trocou `&&` por `||` e `SENHA.equals` por `senha == SENHA`, aceitando qualquer usuário `admin` sem senha.
- Porta 4: `0cd80f6` reduziu o README a uma frase.
- Porta 5: `6572d8a` versionou `config/application.properties` com `db.password` e `api.token`.
## Comandos Git utilizados
- `git log --oneline --all --graph`: mapear histórico e branches.
- `git show <hash>`: ver o que cada commit alterou.
- `git checkout -b resgate/discente-thiago-joao-gabriel`: branch de trabalho.
- `git revert <hash>`: desfaz um commit gerando novo commit rastreável (64f88f6, a70ee84, 9a6d3b0, 0cd80f6, 6572d8a).
- `git merge --no-ff`: integrar o resgate na `main`.
## Commits relevantes
- Revertidos: `64f88f6`, `a70ee84`, `9a6d3b0`, `0cd80f6`, `6572d8a`.
- O segredo continua no histórico (`6572d8a`); na versão atual foi removido e `config/application.properties` entrou no `.gitignore`. As credenciais devem ser rotacionadas; limpar o histórico exigiria `git filter-repo`.
## Validação final
- Compilação: `javac` em todos os `.java` sem erros (Maven não está instalado nesta máquina; `pom.xml` inalterado).
- Login: `admin/12345678` autoriza; `admin/errada` e `x/12345678` retornam "Acesso negado".
- Cadastro: saída mostra a mercadoria com "Entrega: Av. Goiás, 1000...".
- README: instruções de compilar/executar restauradas.
- Segurança: `git grep -i "senha123\|token" -- config` não retorna nada na versão atual.

## Autores dos commits problemáticos
- `9a6d3b0` e `6572d8a`: Felipe Rocha (login quebrado e credenciais versionadas).
- `0cd80f6`: Gustavo Melo (README reduzido).
- `a70ee84`: Henrique Nunes (cadastro sem endereço, chamada a método inexistente).
- `64f88f6`: Igor Reis (exclusão de `Validador.java`).

## Recuperação e comparações
- Arquivo excluído: `git log --diff-filter=D --summary` localizou `Validador.java`; recuperado via `git revert 64f88f6`.
- Comparação de arquivo: `git diff v1.0.0-funcional -- src/main/java/br/edu/entregas/service/LoginService.java`.
- README: versão útil vinda do histórico (`git show 356ec6f:README.md` / branch `docs-readme`).

## Integração
- `git merge --no-ff` na `main`: commits `73e69bf` e `b204842` (visíveis em `git log --oneline --all --graph --decorate`).

## Pergunta de encerramento
O controle de configuração registra quem mudou o quê, quando e por quê. Isso reduz riscos porque cada regressão é localizável e reversível (`git revert`) sem perder evidências; facilita auditorias porque autores, hashes e diffs são rastreáveis (foi assim que achamos o segredo em `6572d8a`); e permite recuperar o projeto porque tags como `v1.0.0-funcional` e branches guardam versões boas. Ressalva: segredos já commitados permanecem no histórico, então é preciso rotacioná-los.
