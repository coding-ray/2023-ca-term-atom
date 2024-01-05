default:
	@echo "With .env set, use:\n  make [start|attach|stop]"

start:
	docker compose up --build --detach

attach:
	docker exec -it $$(grep "^CONTAINER_NAME=" .env | awk -F'=' '{printf $$2}') /bin/bash

stop:
	docker compose down