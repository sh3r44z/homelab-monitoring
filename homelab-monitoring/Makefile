.PHONY: up down restart logs ps pull reload-prometheus

up:
	docker compose up -d

down:
	docker compose down

restart:
	docker compose restart

logs:
	docker compose logs -f

ps:
	docker compose ps

pull:
	docker compose pull

reload-prometheus:
	curl -X POST http://localhost:9090/-/reload
