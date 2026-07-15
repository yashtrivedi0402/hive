# -----------------------------------------------------------------------------
# Aden Tools MCP Server
# Exposes tools via Model Context Protocol (MCP)
# -----------------------------------------------------------------------------

FROM python:3.11-slim

# Python runtime configuration
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# -----------------------------------------------------------------------------
# Install system dependencies and Google Chrome
# -----------------------------------------------------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        wget \
        gnupg && \
    mkdir -p /etc/apt/keyrings && \
    wget -q -O /etc/apt/keyrings/google-chrome.asc \
        https://dl.google.com/linux/linux_signing_key.pub && \
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/google-chrome.asc] \
        http://dl.google.com/linux/chrome/deb/ stable main" \
        > /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends google-chrome-stable && \
    apt-get purge -y wget gnupg && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# Copy dependency metadata first to maximize Docker layer caching
# -----------------------------------------------------------------------------
COPY pyproject.toml README.md ./

# Upgrade pip and install dependencies
RUN python -m pip install --upgrade pip && \
    pip install --no-cache-dir -e .

# Copy application source
COPY src ./src
COPY mcp_server.py .

# -----------------------------------------------------------------------------
# Create non-root user
# -----------------------------------------------------------------------------
RUN useradd --create-home --uid 1001 appuser && \
    mkdir -p /app/workdir/workspaces && \
    chown -R appuser:appuser /app

USER appuser

# Persist workspace data
VOLUME ["/app/workdir/workspaces"]

# OCI image metadata
LABEL org.opencontainers.image.title="Aden Tools MCP Server" \
      org.opencontainers.image.description="Model Context Protocol server exposing Aden tools"

EXPOSE 4001

# -----------------------------------------------------------------------------
# Health Check
# -----------------------------------------------------------------------------
HEALTHCHECK --interval=30s \
            --timeout=5s \
            --start-period=10s \
            --retries=3 \
CMD python -c "import httpx; httpx.get('http://localhost:4001/health').raise_for_status()"

# Start MCP server
CMD ["python", "mcp_server.py"]
