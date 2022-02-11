# Exchange Rate App

# Demo

- http://test.serhatkaya.tk

# Development

- You should create a file named .contentful.json for required datas.
- Use .contentful-sample.json as schema.

# Build with docker

docker build . -t <YOUR_DOCKER_HUB_USERNAME>/exchange-app

# Run

docker run -p 2771:2771 <YOUR_DOCKER_HUB_USERNAME>/exchange-app

# Push container to docker hub

docker push <YOUR_DOCKER_HUB_USERNAME>/exchange-app
