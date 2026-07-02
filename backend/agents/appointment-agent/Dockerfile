# Appointment Scheduling Capability Agent
# Lightweight container for Strands A2A agent with Appointment API integration
#
# Build: docker build -t appointment-agent .
# Run:   docker run -p 8000:8000 -e APPOINTMENT_API_URL=... appointment-agent

# Amazon Linux 2023 eliminates Debian CVEs (gnutls28, openssl, glibc, nghttp2, libcap2)
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN yum install -y \
    python3.13 \
    python3.13-pip \
    shadow-utils \
    && yum clean all

RUN alternatives --install /usr/bin/python3 python3 /usr/bin/python3.13 1 \
    && alternatives --set python3 /usr/bin/python3.13

# Non-root user
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN python3 -m pip install --no-cache-dir -r requirements.txt

# Copy application
COPY appointment_client.py .
COPY main.py .

# Ownership
RUN chown -R appuser:appuser /app
USER appuser

# Default port for A2A server
EXPOSE 8000

# Health check via the A2A Agent Card endpoint (curl-minimal is pre-installed on AL2023)
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8000/.well-known/agent-card.json || exit 1

CMD ["python3", "main.py"]
