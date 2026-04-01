
```docker-compose
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_test
    restart: unless-stopped
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: qwe123
      POSTGRES_DB: test
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d test"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    name: postgres_test_data
```