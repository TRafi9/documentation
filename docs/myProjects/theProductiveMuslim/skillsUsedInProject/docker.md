# Docker

3 different docker images were used in this project.

- Official image for SQL Cloud proxy
- My frontend image
- My backend image

Heres an example of a docker file for my backend service that I was using for local testing

```docker
FROM golang:1.21

WORKDIR /app
COPY . .

RUN go build tpm

EXPOSE 8080

CMD ["sh", "-c", "./tpm"]

```

## Docker compose

I also made the use of docker compose files to speed up development and launch both the frontend and backend docker files at the same time, calling env variables from .env files so I could change configuration easily as I went along, here's an example:

```
version: "3"

services:
  tpm-backend:
    image: tpm-backend
    build:
      context: .
      dockerfile: Dockerfile
    env_file:
      - .env
    ports:
      - "8080:8080"

  tpm-frontend:
    image: tpm-frontend
    build:
      context: ./frontend/tpm-frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
```

## Passing env vars

### Locally

There were 2 ways I was passing env vars in for local testing:

1. Mounting env vars using the `env_file` flag in my docker-compose file
2. When running docker images independantly, docker images were passed using env variables using the `-e` flag to pass env variables when the docker run command was executed, I used this to keep secrets outside my code base and pass them into the scripts.

### In Production

In the production environment, my app was running in kubernetes, so I loaded the env vars in through a kubernetes yaml manifest from secrets I stored in my cluster by using `kubectl create secret`
