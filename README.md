the docker compose final file is 
version: '3.3'
services:
  server:
    build:
      context: .
    ports:
      - 9000:80
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
    environment:
      - PASSWORD_FILE_PATH=/run/secrets/db-password
      - DB_HOST=db
      - DB_NAME=example
      - DB_USER=root
#for dynamic Update of Services###

    develop:
      watch:
        - action: sync
          path: ./src
          target: /var/www/html

  db:
    image: mariadb
    restart: always
    user: root
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/mysql
    environment:
      - MARIADB_ROOT_PASSWORD_FILE=/run/secrets/db-password
      - MARIADB_DATABASE=example
    expose:
      - 3306
    healthcheck:
      test:  ["CMD", "/usr/local/bin/healthcheck.sh", "--su-mysql", "--connect",  "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
#### delete if issues ###
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8080:80
    depends_on:
      - db
    environment:
      - PMA_HOST=db

volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
