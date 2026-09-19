ARG ODOO_VERSION=19.0
FROM odoo:${ODOO_VERSION}

LABEL org.opencontainers.image.title="noden · Odoo Module Migration Workspace" \
      org.opencontainers.image.description="Test environment for AI-assisted Odoo module migrations (v12 → v19)" \
      org.opencontainers.image.vendor="Noden Labs" \
      org.opencontainers.image.url="https://sentilis.me/noden" \
      org.opencontainers.image.source="https://github.com/nodenlabs/devx"

USER root

RUN mkdir -p /mnt/extra-addons && chown -R odoo:odoo /mnt/extra-addons

USER odoo
