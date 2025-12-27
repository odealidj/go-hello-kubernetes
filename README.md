# go-hello-kubernetes

A simple Go REST API connected to MariaDB, containerized with Docker, and ready to be deployed with Docker Compose. Kubernetes guidance is outlined for future deployment.

## Overview

This repository contains two main components:

- `hello-world`: A Go HTTP server exposing REST endpoints.
- `mariadb`: Database initialization script (`schema.sql`) to create the `hello_world` database and `users` table.

The API reads environment variables for configuration and uses the official MySQL driver.

## Stack

- Go (module `hello-world`, `net/http`)
- MariaDB 10.1
- Docker + Docker Compose

## Services

Defined in [docker-compose.yml](docker-compose.yml):

- `service-hello-world`: Builds from [hello-world/Dockerfile](hello-world/Dockerfile), exposes HTTP on `8083`.
- `service-mariadb`: Runs `mariadb:10.1`, initializes schema from [mariadb/schema.sql](mariadb/schema.sql) and persists data to a Windows volume.

## Environment Variables

- `PORT` (required): Port for the HTTP server (default used in Compose: `8083`).
- `MYSQL_CONN_STRING` (required): MySQL DSN; example used in Compose: `root@tcp(service-mariadb:3306)/hello_world?parseTime=true`.
- `INSTANCE_ID` (optional): If set, appends instance info to root endpoint response.

## API Endpoints

- `GET /`: Returns `"hello world"` (optionally `"hello world. from <INSTANCE_ID>"`).
- `GET /user`: Returns all users from DB.
- `POST /user`: Creates a new user.

Request body for `POST /user` (note: field names are PascalCase per struct definitions):

```json
{
	"FirstName": "Jane",
	"LastName": "Doe",
	"Birth": "1994-05-10T00:00:00Z"
}
```

`Birth` should be in RFC3339 format (e.g., `YYYY-MM-DDTHH:MM:SSZ`).

Sample response shape:

```json
{
	"Status": 200,
	"Data": [
		{
			"ID": 1,
			"FirstName": "Jane",
			"LastName": "Doe",
			"Birth": "1994-05-10T00:00:00Z"
		}
	],
	"Message": ""
}
```

## Run Locally with Docker Compose (Windows)

Prerequisites:

- Docker Desktop installed and running.
- Ensure the local data directory exists for MariaDB volume: `D:\docker-volumes\hello-world`.

From the repository root:

```powershell
# Build images
docker compose build

# Start services
docker compose up -d

# Tail app logs (optional)
docker compose logs -f service-hello-world
```

Test the API (PowerShell using `curl.exe` to avoid alias issues):

```powershell
curl.exe http://localhost:8083/
curl.exe http://localhost:8083/user

# Create a user (use RFC3339 for Birth)
curl.exe -X POST http://localhost:8083/user ^
	-H "Content-Type: application/json" ^
	-d "{\"FirstName\":\"Jane\",\"LastName\":\"Doe\",\"Birth\":\"1994-05-10T00:00:00Z\"}"
```

To stop and remove containers:

```powershell
docker compose down
```