## Application Commands, EntryPoint and Arguments in Pod defination

### Docker

Difference Between `ENTRYPOINT` and `CMD` in Docker and use case.

- **`CMD`** is used to run the command when the container is started.
  - The `CMD` instruction specifies the default command or arguments that will be executed by the ENTRYPOINT if no other command is provided when running the container.
  - If you use both `ENTRYPOINT` and `CMD` in a Dockerfile, the CMD instruction will provide the default arguments for the `ENTRYPOINT`.
- **`ENTRYPOINT`** is used to run the command when the container is started.
  - It is the preferred way to define the main command for a Docker container.
  - If you provide additional arguments when running the container, they will be appended to the `ENTRYPOINT` instruction.
  - The ENTRYPOINT cannot be overridden by arguments provided during container runtime unless you use the `--entrypoint` flag.

Command:
`docker run ubuntu [COMMAND]`

![alt text](image-5.png)

**Example:**

1.

```Dockerfile
# Example 1: Using ENTRYPOINT
FROM ubuntu
ENTRYPOINT ["sleep"]

# If you run: docker run my-image 10
# It will execute: sleep 10

# If you run: docker run my-image sleep2.0 10
# It willnot execute: sleep 10 (cannot replace sleep the default ENTRYPOINT) need to use `--entrypoint sleep2.0` to replace it in commandline

```

`docker run --entrypoint sleep2.0 my-image 10`

2.

```Dockerfile
# Example 2: Using CMD
FROM ubuntu
CMD ["sleep", "5"]

# If you run: docker run my-image
# It will execute: sleep 5

# If you run: docker run my-image 10
# It will execute: sleep 10 (overriding the default CMD)

# If you run: docker run my-image sleep2.0 10
# It will execute: sleep2.0 10 (replace the default CMD)

```

3.

```Dockerfile
# Example 3: Using ENTRYPOINT and CMD together
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

# If you run: docker run my-image
# It will execute: sleep 5 (CMD provides the default argument)

# If you run: docker run my-image 10
# It will execute: sleep 10 (additional arguments are appended)
```
