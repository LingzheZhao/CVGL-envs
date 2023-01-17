# hloc

## Code

https://github.com/cvg/Hierarchical-Localization

# Run

```bash
docker build -t hloc:latest .
docker run -it --rm -p 8888:8888 --gpus all hloc:latest jupyter lab --ip 0.0.0.0 --port 8888 --no-browser --allow-root
```

Or directly use the pre-built image:

```bash
docker run -it --rm -p 8888:8888 --gpus all harbor.cvgl.lab/library/hloc:latest jupyter lab --ip 0.0.0.0 --port 8888 --no-browser --allow-root
```
