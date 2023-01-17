build:
	docker-compose build nginx

run:
	docker-compose up

down:
	docker-compose down -v

strace:
	docker-compose exec nginx strace -p 1 -s 10000 -v -f

reload:
	docker-compose exec nginx kill -HUP 1
