# PostgreSQL Development Environment

This repository contains the configuration files and setup instructions for a PostgreSQL development environment using Docker and Visual Studio Code (VSCode).

## Starting the Dev Container

Open VSCode Command Palette (F1) and select:

-   `Dev Container: Reopen in Container`
-   OR
-   `Dev Container: Rebuild Without Cache and Reopen in Container`

## Prerequisites

Ensure you have the following installed:

-   [Docker](https://www.docker.com/get-started)
-   [Docker Compose](https://docs.docker.com/compose/install/)
-   [Visual Studio Code](https://code.visualstudio.com/)
-   [Remote - Containers VSCode Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Directory Structure

The project directory should look like this:

```plaintext
.
├── .devcontainer
│   └── devcontainer.json
├── database
│   └── init.sql
├── Dockerfile
└── docker-compose.yml
```

## Configuration Files

### .devcontainer/devcontainer.json

Configures the development environment:

```json
{
	"name": "Postgres Development Container",
	"dockerComposeFile": "../docker-compose.yml",
	"service": "db",
	"workspaceFolder": "/",
	"customizations": {
		"vscode": {
			"extensions": ["mtxr.sqltools", "mtxr.sqltools-driver-pg"]
		}
	},
	"settings": {
		"sqltools.connections": [
			{
				"name": "PostgreSQL Container",
				"driver": "PostgreSQL",
				"server": "localhost",
				"port": 5432,
				"database": "mydb",
				"username": "admin",
				"password": "admin"
			}
		]
	},
	"forwardPorts": ["5432:5432", "5050:5050"],
	"postCreateCommand": "psql --version"
}
```

### Dockerfile

Sets up a PostgreSQL database container:

```dockerfile
FROM postgres:latest

ENV POSTGRES_USER=admin
ENV POSTGRES_PASSWORD=admin
ENV POSTGRES_DB=mydb

COPY ./database/init.sql /docker-entrypoint-initdb.d/
EXPOSE 5432
```

### docker-compose.yml

Builds and runs PostgreSQL and pgAdmin containers:

```yaml
services:
    db:
        build: .
        container_name: postgres-db
        ports:
            - "5432:5432"
        environment:
            POSTGRES_USER: admin
            POSTGRES_PASSWORD: admin
            POSTGRES_DB: mydb
        volumes:
            - db-data:/var/lib/postgresql/data
        healthcheck:
            test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
            interval: 10s
            timeout: 5s
            retries: 5

    pgadmin:
        image: dpage/pgadmin4
        container_name: pgadmin
        environment:
            PGADMIN_DEFAULT_EMAIL: admin@admin.com
            PGADMIN_DEFAULT_PASSWORD: admin
        ports:
            - "5050:5050"
        depends_on:
            db:
                condition: service_healthy

volumes:
    db-data:
        driver: local
```

## Database Initialization Script

### database/init.sql

Creates tables and inserts sample data:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) VALUES
('john_doe', 'john@example.com'),
('jane_smith', 'jane@example.com');

INSERT INTO products (name, description, price) VALUES
('Laptop', 'High-performance laptop', 999.99),
('Smartphone', 'Latest model smartphone', 599.99),
('Headphones', 'Noise-cancelling headphones', 199.99);

INSERT INTO orders (user_id, total_amount, status) VALUES
(1, 1599.98, 'completed'),
(2, 199.99, 'pending');
```

## Accessing the Database

### pgAdmin

Access pgAdmin at [http://localhost:5050](http://localhost:5050) with credentials:

-   Email: `admin@admin.com`
-   Password: `admin`

### PostgreSQL Client Tools

Install the [PostgreSQL Client Tools for VSCode](https://marketplace.visualstudio.com/items?itemName=ms-ossdata.vscode-postgresql).

Open the Command Palette (Ctrl + Shift + P) and select `PostgreSQL: New Query`.

### Connect to Docker Container

```sh
# Get the container ID
docker ps

# Open a shell in the container
docker exec -it container_id sh

# Log in to PostgreSQL
psql -h localhost -U admin -d mydb

# List tables
\dt

# Query the table to verify the data
SELECT * FROM users;
```

## Spring Boot Application

Configure `application.properties` or `application.yml`:

```properties
spring.datasource.url=jdbc:postgresql://postgres-db:5432/mydb
spring.datasource.username=admin
spring.datasource.password=admin
```

Replace `postgres-db` with the actual name or network alias of your PostgreSQL container.

## References

-   [PostgreSQL Docker Official Image](https://hub.docker.com/_/postgres)
-   [Creating a PostgreSQL Docker Container](https://forums.docker.com/t/how-to-make-a-docker-file-for-your-own-postgres-container/126526)
-   [Using the PostgreSQL Docker Image](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/)
-   [Setting Up PostgreSQL with Docker](https://www.dbvis.com/thetable/how-to-set-up-postgres-using-docker/)
-   [Creating a PostgreSQL Database with Docker](https://dev.to/andre347/how-to-easily-create-a-postgres-database-in-docker-4moj)

For more information, check the [awesome-compose repository](https://github.com/docker/awesome-compose/blob/master/postgresql-pgadmin/README.md).
