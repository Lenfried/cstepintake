# syntax=docker/dockerfile:1
ARG PYTHON_VERSION=3.13
FROM plone/server-builder:uv-${PYTHON_VERSION} AS builder

COPY . /src
WORKDIR /src

# Install package
RUN --mount=type=cache,target=/root/.cache \
    --mount=type=bind,source=mx.ini,target=mx.ini \
    uv tool run mxdev[uv] -c mx.ini && \
    uv sync \
        --no-dev \
        --no-group test \
        --group container \
        --no-editable

# Move skeleton files to /app
RUN <<EOT
    mv /app_skeleton/* /app
    rm -Rf /app_skeleton
    chmod +x /app/docker-entrypoint.sh
    mv scripts/create_site.py /app/scripts/create_site.py
EOT

# Compile translation files
RUN <<EOT
    /app/bin/python /compile_mo.py
EOT

FROM plone/server-prod-config:uv-${PYTHON_VERSION}

LABEL maintainer="Zihao <zihao.chen23@login.cuny.edu>" \
      org.label-schema.name="c-step-intake-backend" \
      org.label-schema.description="c-step-intake backend image." \
      org.label-schema.vendor="Zihao"

# Copy the pre-built `/app` directory to the runtime container
# and change the ownership to user app and group app in one step.
COPY --from=builder --chown=500:500 /app /app

RUN <<EOT
    ln -s /data /app/var
    chown -R 500:500 /data
EOT
