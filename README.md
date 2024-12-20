# judge-images
```YAML
name: Update Image

on:
  push:
    branches:
      - main  # 僅在 main 分支觸發
  workflow_dispatch:  # 手動觸發
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build the Server Docker image
      run: |
        docker build --pull --no-cache -f dockerfile.api -t tsukisama9292/server:api .
        docker build --pull --no-cache -f dockerfile.nginx -t tsukisama9292/server:nginx .
        docker build --pull --no-cache -f dockerfile.redis -t tsukisama9292/server:redis .

    - name: Push the Docker image to Docker Hub
      run: |
        docker push tsukisama9292/server:api
        docker push tsukisama9292/server:nginx
        docker push tsukisama9292/server:redis
    # 清理 Docker Builder 緩存
    - name: Prune Docker builder cache
      run: docker builder prune -a -f

    # 清理所有 Docker 映像檔
    - name: Prune all Docker resources
      run: docker system prune -a -f
        
  pull-image:
    runs-on: self-hosted
    needs: build
    if: success()
    steps:
    - name: Pull the Docker image
      run: |
        docker pull tsukisama9292/ubuntu-jammy-dind:1.16.0-latest
```