# Relatorio de evidencias - Trabalho Pratico Integrador 01-06

Nome: Mateus Pinto Magalhães  Data: 24/11/2026
URL do repositorio Git: ____________________________

## E01 - Ambiente (1 valor)
Comandos e versoes observadas:

PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker --version
Docker version 29.9.2, build a82e528
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose version
Docker Compose version v5.6.1
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git --version
git version 2.56.1.windows.1
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker info
Server Version: 29.9.2
OSType: linux

API = app.py, 
Testes = tests/test_app.py, 
Compose = compose.yaml, 
Pipeline = Jenkinsfile.

## E02 - Dockerfile e container (3 valores)
Tag da imagem; docker ps; HTTP /health; utilizador efetivo do container:

PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker build -t natixis-support:1.0 .
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker run -d --name tp-api -p 127.0.0.1:18080:8000 natixis-support:1.0
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                       NAMES
ce4af62f2965   natixis-support:1.0   "python -m uvicorn a…"   2 minutes ago   Up 2 minutes   127.0.0.1:18080->8000/tcp   tp-api
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/health
{"status":"ok","service":"support-api","version":"1.0"}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker logs --tail 15 tp-api
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     172.17.0.1:45306 - "GET /health HTTP/1.1" 200 OK
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker exec tp-api id
uid=10001(appuser) gid=10001(appuser) groups=10001(appuser)
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker rm -f tp-api
tp-api

Tag: "natixis-support:1.0"
Portas: host "127.0.0.1:18080" - container "8000"
Utilizador efetivo: "appuser" (uid 10001), não é o root.

Neste exercicio a imagem foi construida e o container respondeu HTTP 200 com status="ok",  o processo corre com um utilizador sem privilegios (appuser) que é dono de "/data", e do ".dockerignore" que por sua vez impede que os ficheiros .git, .venv, caches, .env e evidencias entrem no contexto de build.


## E03 - Compose, DNS, healthcheck e volume (4 valores)
Resultado de docker compose config; health; teste checker -> api; contador antes/depois da recriacao:

PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose config --quiet
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose up -d --build
[+] Building 103.7s (15/15) FINISHED
[...] 
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose ps
NAME                       IMAGE                   COMMAND                  SERVICE   CREATED          STATUS                    PORTS
natixis-integrador-api-1   natixis-support:local   "python -m uvicorn a…"   api       28 seconds ago   Up 26 seconds (healthy)   127.0.0.1:18080->8000/tcp
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose --profile diagnostico run --rm checker -fsS http://api:8000/health
[...]
{"status":"ok","service":"support-api-integrador","version":"1.0"}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/visits
{"visits":0}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe -X POST http://127.0.0.1:18080/visits
{"visits":1}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/visits
{"visits":1}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose up -d --force-recreate
[+] up 1/1
 ✔ Container natixis-integrador-api-1 Started                                                                                                                                                                         3.4s
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose ps
NAME                       IMAGE                   COMMAND                  SERVICE   CREATED          STATUS                   PORTS
natixis-integrador-api-1   natixis-support:local   "python -m uvicorn a…"   api       11 seconds ago   Up 8 seconds (healthy)   127.0.0.1:18080->8000/tcp
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/visits
{"visits":1}

O Compose é válido, o servico "api" fica "healthy" e o "checker" so arranca depois disso ("depends_on: condition: service_healthy"). O "checker" chegou á API pelo nome "api", o que prova a resolucao DNS interna da rede do Compose.
O contador nao se perdeu porque o ficheir "/data/visitas.txt" esta guardado no volume com o nome de "app-data", que existe independentemente do container, recriar o container apaga apenas a camada de escrita dele, e o volume é montado de novo no container novo não perdendo assim os seus dados.


## E04 - Incidente Docker (3 valores)
Sintoma, comandos de investigacao, causa-raiz, alteracao realizada, prova da recuperacao e contador preservado:


## SINTOMA: 
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose -f compose.yaml -f incidente.yaml ps
NAME                       IMAGE                   COMMAND                  SERVICE   CREATED         STATUS                     PORTS
natixis-integrador-api-1   natixis-support:local   "python -m uvicorn a…"   api       2 minutes ago   Up 2 minutes (unhealthy)   127.0.0.1:18080->8000/tcp
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/health
curl: (52) Empty reply from server
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose -f compose.yaml -f incidente.yaml logs --tail 30 api
api-1  | INFO:     Started server process [1]
api-1  | INFO:     Waiting for application startup.
api-1  | INFO:     Application startup complete.
api-1  | INFO:     Uvicorn running on http://0.0.0.0:8001 (Press CTRL+C to quit)

O container ficou unhealthy e curl a /health falha.

## DIAGNOSTICO/CAUSA RAIZ: 
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose -f compose.yaml -f incidente.yaml config
name: natixis-integrador
[...]
- --port
- "8001"
[...]
target: 8000
[...]
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker inspect --format "{{json .State.Health}}" natixis-integrador-api-1
{"Status":"unhealthy"}[...]

O overlay póe o unicorn na porta 8001 mas o mapeaomento passa pela porta 8000 e o healthcheck como usa a porta 8000 e não está la nada dá erro.  

## CORREÇÃO: 
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose -f compose.yaml -f incidente.yaml exec api python -c "import urllib.request;print(urllib.request.urlopen('http://127.0.0.1:8001/health').read())"
b'{"status":"ok","service":"support-api-integrador","version":"1.0"}'
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose up -d --force-recreate api
[+] up 1/1
 ✔ Container natixis-integrador-api-1 Started                                                                                                                                                                         3.7s

Regressar a configuracao base (sem o overlay), recriando so o servico "api" e sem remover o volume.

## VALIDAÇÃO 
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> docker compose ps
NAME                       IMAGE                   COMMAND                  SERVICE   CREATED          STATUS                    PORTS
natixis-integrador-api-1   natixis-support:local   "python -m uvicorn a…"   api       35 seconds ago   Up 31 seconds (healthy)   127.0.0.1:18080->8000/tcp
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/health
{"status":"ok","service":"support-api-integrador","version":"1.0"}
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> curl.exe http://127.0.0.1:18080/visits
{"visits":1}

O contador manteve o valor anterior porque o volume "app-data" nao foi removido e o status já demonstra o valor healthy outra vez. 

## E05 - Git (3 valores)
Branch, hashes de commits, merge e tag v1.0.0:

PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git switch -c fix/health-info
Switched to a new branch 'fix/health-info'
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git commit -m "fix: acrescentar identificacao no health"
[fix/health-info 3fa2f76] fix: acrescentar identificacao no health
 2 files changed, 3 insertions(+), 1 deletion(-)
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git switch main
Switched to branch 'main'
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git merge --no-ff fix/health-info -m "merge: integrar fix/health-info"
Merge made by the 'ort' strategy.
 app.py            | 3 ++-
 tests/test_app.py | 1 +
 2 files changed, 3 insertions(+), 1 deletion(-)
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git tag -a v1.0.0 -m "Release integrador 1.0.0"
PS C:\Users\MateusMagalhães\Desktop\Mateus\DevOps\TP_Integrador_Kit_Formandos> git log --oneline --graph --decorate -6
*   fd5f4d7 (HEAD -> main, tag: v1.0.0) merge: integrar fix/health-info
|\  
| * 3fa2f76 (fix/health-info) fix: acrescentar identificacao no health
|/  
* 5d6a4ab feat: ambiente integrador funcional

## E06 - Jenkins SUCCESS (4 valores)
URL/numero do build, etapas e identificador da imagem construida:

## E06 - Jenkins SUCCESS (4 valores)
URL/numero do build, etapas e identificador da imagem construida:

- Job: tp-integrador-MateusMagalhaes
- Repositorio: https://github.com/K4MIS4MA/tp-integrador
- Build: #1 - http://localhost:8080/job/tp-integrador-MateusMagalhaes/1/
- Etapas: Checkout SUCCESS | Tests SUCCESS (2 passed) | Docker Build SUCCESS
- Imagem: natixis-support:ci-1
- ID da imagem (docker image inspect): sha256:35922b49635a856b8481708e609f1b36c0c9a81cb03313a048fdb3144964776c
- Captura: evidencias/SucessJenkins.png

Console Output JOB 1:

Running on ci-python-docker
Checking out Revision 6ab00d8df2bd977c6f370fd579aba77d79df2c91 (refs/remotes/origin/main)
+ .venv/bin/python -m pytest -q
2 passed, 1 warning in 0.79s
+ docker build -t natixis-support:ci-1 .
Successfully tagged natixis-support:ci-1
+ docker image inspect natixis-support:ci-1 --format {{.Id}}
sha256:35922b49635a856b8481708e609f1b36c0c9a81cb03313a048fdb3144964776c
Finished: SUCCESS


A pipeline obtem o codigo do Git (Checkout), corre os testes num venv isolado no agente ci-python-docker e depois constroi a imagem com uma tag unica por execucao (ci-${BUILD_NUMBER}= ci-1), confirmada com docker image inspect.

## E07 - Jenkins FAILURE e recuperacao (2 valores)
Build FAILURE, etapa que falhou e excerto relevante do log; build SUCCESS apos correcao:

Como mudei o assert response.json()["status"] para offline, o job agora vai dar failure. 

Console output Jenkins JOB 2: 
[...]
FAILED tests/test_app.py::test_health - AssertionError: assert 'ok' == 'offline'
[...]
ERROR: script returned exit code 1
Finished: FAILURE

Para voltar a coorer bem o job basta mudar o assert response.json()["status"] (que se encontra em tests/test_app.py) para ok e dar commit para o github. 
Console output Jenkins JOB 3: 
Successfully built 35922b49635a
Successfully tagged natixis-support:ci-3
+ docker image inspect natixis-support:ci-3 --format {{.Id}}
sha256:35922b49635a856b8481708e609f1b36c0c9a81cb03313a048fdb3144964776c
Finished: SUCCESS

A assercao alterada esperava status="offline" mas a API devolve "ok", por isso o pytest falhou e a pipeline parou na etapa Tests, impedindo a construcao de uma imagem com testes a falhar. Depois de repor o assert, o build voltou a SUCCESS.


## Verificacao final
[ ] Dockerfile [ ] compose.yaml [ ] app.py [ ] tests/ [ ] Jenkinsfile
[ ] Historico Git [ ] Tag v1.0.0 [ ] Evidencias E01-E07
