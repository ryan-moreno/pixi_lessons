# Defines an existing linux container that we're building on top of.
FROM ghcr.io/prefix-dev/pixi:noble AS build

# akes the directory if it doesn't already exit and set working directory
WORKDIR /app
# Copy everything from outside the container into the working directory in the docker file (e.g. pixi.toml etc.)
COPY . .
# Override cuda virtual package (it won't have GPU) and set particular version
ENV CONDA_OVERRIDE_CUDA=12
# Install locked version of the pixi environment
RUN pixi install --locked --environment gpu
# Want ability to interact with the pixi environment without interacting with pixi
# Creating a tiny script that calls the shell hook at entrypoint.sh
RUN echo "#!/bin/bash" > /app/entrypoint.sh && \
    pixi shell-hook --environment gpu -s bash >> /app/entrypoint.sh && \
    echo 'exec "$@"' >> /app/entrypoint.sh

# Best to have build and production stages separated
# Example: might not need build tools (like compilers) in production
FROM ghcr.io/prefix-dev/pixi:noble AS production

WORKDIR /app
COPY --from=build /app/.pixi/envs/gpu /app/.pixi/envs/gpu
COPY --from=build /app/pixi.toml /app/pixi.toml
COPY --from=build /app/pixi.lock /app/pixi.lock
# The ignore files are needed for 'pixi run' to work in the container
COPY --from=build /app/.pixi/.gitignore /app/.pixi/.gitignore
COPY --from=build /app/.pixi/.condapackageignore /app/.pixi/.condapackageignore
COPY --from=build --chmod=0755 /app/entrypoint.sh /app/entrypoint.sh
COPY ./app /app/src

ENTRYPOINT [ "/app/entrypoint.sh" ]