# Commands and Arguments

- `command` in the Pod spec = `ENTRYPOINT` in the Dockerfile.
- `args` in the Pod spec = `CMD` in the Dockerfile.
- If `args` is not passed but `command` is, both `CMD` and `ENTRYPOINT` from the Dockerfile are ignored.
