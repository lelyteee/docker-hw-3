# Docker hw

## setup

```sh
cp .env.example .env
mkdir -p logs
```

set `APP_PORT` and `GREETING` in `.env`
after starting a container, wait a few seconds before running curl

## single container

```sh
docker build -t igor-counter .
docker run -d --name igor-counter-demo -p 8080:8000 -e 'GREETING=Hello, Igor!' igor-counter
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
docker logs igor-counter-demo > logs/4-container.log 2>&1
docker stop igor-counter-demo
docker rm igor-counter-demo
```

## SQLite

```sh
docker compose -f compose.sqlite.yaml up -d
```

wait until the app is healthy: `docker compose -f compose.sqlite.yaml ps`

```sh
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
docker compose -f compose.sqlite.yaml down
docker compose -f compose.sqlite.yaml up -d
```

wait until the app is healthy again

```sh
curl http://localhost:8080/
docker compose -f compose.sqlite.yaml down
```

## Redis

```sh
docker compose up -d
```

wait until both services are healthy: `docker compose ps`

```sh
curl -X POST http://localhost:8080/increment
curl http://localhost:8080/
docker compose down
docker compose up -d
```

wait until both services are healthy again

```sh
curl http://localhost:8080/
docker compose stop redis
curl -i http://localhost:8080/healthz
curl -i http://localhost:8080/readyz
curl -i -X POST http://localhost:8080/increment
docker compose start redis
docker compose ps
```

wait until both services are healthy

```sh
curl http://localhost:8080/
curl -X POST http://localhost:8080/increment
docker compose down
```

## res

saved output: `logs/4-container.log`, `logs/6-check.txt`, `logs/8-check.txt`

```text
container: 0 -> 1 -> 1, greeting works
sqlite after down/up: 1 -> 1
redis after down/up: 1 -> 1
redis stopped: healthz 200, readyz 503, increment 503
redis back: count 1, increment 2, app not restarted
```
