# ===============================
# Stage 1: Base dependencies
# ===============================
FROM node:20-bookworm AS base

RUN apt-get update && apt-get install -y \
    openssl \
    ca-certificates \
    --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

RUN npm install -g pnpm
WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --ignore-scripts

# ===============================
# Stage 2: Build
# ===============================
FROM base AS builder

ARG COMMIT_SHA
ARG SENTRY_DSN

ENV COMMIT_SHA=${COMMIT_SHA}
ENV SENTRY_DSN=${SENTRY_DSN}

COPY . .
RUN pnpm exec prisma generate
RUN pnpm run build

# ===============================
# Stage 2.5: Prod deps only
# ===============================
FROM base AS prod-deps

COPY . .
# Fresh prod-only install in a clean stage
RUN pnpm install --frozen-lockfile --ignore-scripts --prod
# Re-run prisma generate so the client exists in this prod node_modules
RUN pnpm add prisma --save-dev && pnpm exec prisma generate && pnpm remove prisma

# ===============================
# Stage 3: Runtime
# ===============================
FROM node:20-bookworm AS runtime

RUN apt-get update && apt-get install -y \
    openssl \
    ca-certificates \
    --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

RUN npm install -g pnpm
WORKDIR /app

RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --ingroup appgroup appuser

COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/prisma ./prisma
COPY --from=builder --chown=appuser:appgroup /app/package.json ./package.json
COPY --from=builder --chown=appuser:appgroup /app/pnpm-lock.yaml ./pnpm-lock.yaml

# Copy prod-only node_modules (with generated prisma client)
COPY --from=prod-deps --chown=appuser:appgroup /app/node_modules ./node_modules

ARG COMMIT_SHA
ENV COMMIT_SHA=${COMMIT_SHA}

# Create temp storage dir with correct ownership before dropping to appuser
# RUN mkdir -p /app/storage/temp && \
#     chown -R appuser:appgroup /app/storage
RUN mkdir -p /app/storage/temp && \
    touch /app/swagger.json && \
    chown -R appuser:appgroup /app/storage && \
    chown appuser:appgroup /app/swagger.json

USER appuser

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', r => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["node", "dist/main"]