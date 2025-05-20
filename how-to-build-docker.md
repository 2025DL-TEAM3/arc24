1. `docker build -f docker/Dockerfile -t arc24 .`

2. `docker run --gpus all -it --rm -v $(pwd):/workspace -w /workspace arc24`

