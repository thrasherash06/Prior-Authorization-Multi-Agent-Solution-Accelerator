# =============================================================================
# Synthesis Decision Agent — MAF Hosted Agent container
# =============================================================================
# Built with --platform linux/amd64 via 'az acr build --platform linux/amd64'
# (Foundry Hosted Agents run on Linux AMD64 infrastructure)
# See: https://aka.ms/foundry-hosted-agents
FROM python:3.12-slim

# Install curl for health check
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy agent code and skills
COPY main.py ./
COPY schemas.py ./
COPY skills/ ./skills/

# Foundry hosting runtime (ResponsesHostServer) persists function-approval state
# and workflow checkpoints to HARD-CODED absolute paths /.function_approvals and
# /.checkpoints. A non-root user cannot create these under /, so any request that
# triggers a tool/skill approval (e.g. the SkillsProvider `load_skill` call) fails
# with "[Errno 13] Permission denied: '/.function_approvals'" and the agent returns
# empty output. Pre-create them world-writable (sticky bit, like /tmp) so the
# runtime can write regardless of the UID the platform assigns.
RUN mkdir -p /.function_approvals /.checkpoints && chmod 1777 /.function_approvals /.checkpoints

# Non-root user
RUN useradd --create-home appuser && chown -R appuser:appuser /app
USER appuser

ENV PYTHONUNBUFFERED=1

EXPOSE 8088

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
    CMD curl -f http://localhost:8088/readiness || exit 1

CMD ["python", "main.py"]
