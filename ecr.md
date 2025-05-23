To utilise docker layer caching in ECR (inline) you must enable docker buildkit. E.g. in a github action that attempts to build from a cache with 
```zsh
export DOCKER_BUILDKIT=1
docker build . \
	--file Dockerfile \
	--build-arg BUILDKIT_INLINE_CACHE=1 \
	--cache-from=type=registry,ref=$IMAGE_URI:latest \
	--tag $IMAGE_URI:$IMAGE_TAG \
	--tag $IMAGE_URI:latest
```
