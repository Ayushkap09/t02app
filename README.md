# Project 2: Containerize and Run Locally

This project is a small website that runs inside Docker. It has three parts:

- A web page that is built with a two-step (multi-stage) Docker build and served by nginx.
- A Redis cache that runs next to the website, started with Docker Compose.
- A small AWS function that runs on your own computer inside a container.

You do not need an AWS account or any cloud account to run any of this.

## What you need first

- Docker (with the Compose plugin)
- git
- AWS SAM CLI

Type all commands into an Ubuntu (WSL) terminal, not PowerShell.
Port 8080 on your computer needs to be free.

## Step 1: Download the project

    git clone https://github.com/Ayushkap09/t02app.git t02app
    cd t02app

## Step 2: Check the files are set up correctly

    grep -c "^FROM" Dockerfile
    docker compose config --services

The first command should print 2. This means the Dockerfile has two stages.
The second command should print two names: cache and web.

## Step 3: Build the website image

    docker build -t t02app:multi .

## Step 4: Start the website and the cache

    docker compose up -d
    docker compose ps

You should see both services running. The cache should say "(healthy)".

## Step 5: Check that both services work

    curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080
    docker compose exec cache redis-cli ping

The first command should print 200, which means the website loaded.
The second command should print PONG, which means the cache is answering.

## Step 6: Stop everything

    docker compose down

## Step 7: Run the function

    cd local-fn
    sam local invoke HelloWorldFunction
    sam local invoke HelloWorldFunction -e events/event.json
    cd ..

Both commands should show "statusCode": 200 in the output.

The first time you run this, it downloads a Lambda image, so it may take a
few minutes. After that it is fast.

You may also see a line that starts with "[ERROR] ... Failed to get next
invocation". This is normal. It shows up after the result, when the test
container shuts down. Nothing actually went wrong.

## What is in this folder

| File or folder     | What it does                                                     |
|--------------------|------------------------------------------------------------------|
| Dockerfile         | Builds the site in one stage, then copies only the result into a small nginx image |
| Dockerfile.single  | A one-stage version, used only to compare image sizes            |
| compose.yaml       | Starts the website and the Redis cache together, in the right order |
| .dockerignore      | Stops extra files (like local-fn and .git) from going into the image |
| local-fn/          | The AWS SAM "hello world" function                               |
| RATIONALE.md       | Explains why I picked these images and services                  |
