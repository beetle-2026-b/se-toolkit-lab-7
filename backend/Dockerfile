# Stage 1: Build
ARG REGISTRY_PREFIX=harbor.pg.innopolis.university/docker-hub-cache/
FROM ${REGISTRY_PREFIX}astral/uv:python3.14-bookworm-slim AS builder
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy UV_NO_DEV=1 UV_PYTHON_DOWNLOADS=0

WORKDIR /app

# Copy all pyproject.toml files first
COPY pyproject.toml uv.lock ./
COPY backend/pyproject.toml ./backend/

# Run uv lock on the container to ensure lockfile matches
# This avoids --locked errors due to stale host uv.lock
RUN uv lock

# Install dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --no-install-project

# Copy backend code
COPY backend /app/backend

# Final uv sync to ensure environment is ready
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --no-install-project

# Stage 2: runtime
FROM ${REGISTRY_PREFIX}python:3.14.2-slim-bookworm
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1 LANG=C.UTF-8

RUN groupadd --system --gid 999 nonroot \
    && useradd --system --gid 999 --uid 999 --create-home nonroot

COPY --from=builder --chown=nonroot:nonroot /app /app
ENV PATH="/app/.venv/bin:$PATH"

USER nonroot
WORKDIR /app

CMD ["python", "backend/app/run.py"]
