# ------------------------
# Builder stage
# ------------------------
# Prisma 7 requires Node >= 20.19 (recommends 22.x).
FROM node:22-slim AS builder
WORKDIR /app

# Install OpenSSL (required by Prisma)
RUN apt-get update -y \
  && apt-get install -y openssl ca-certificates \
  && rm -rf /var/lib/apt/lists/*

COPY package*.json ./
COPY tsconfig*.json ./
COPY nest-cli.json .swcrc ./
RUN npm ci
COPY . .

# Dummy DB URL for Prisma generate. prisma.config.ts reads DB_URL (the app's
# datasource env), so both are set so config load + generate don't fail at build.
ENV DATABASE_URL="postgresql://user:pass@localhost:5432/dummy"
ENV DB_URL="postgresql://user:pass@localhost:5432/dummy"
RUN npx prisma generate
RUN npm run build

# Extract ffprobe binary before pruning devDeps
RUN node -e "const p=require('ffprobe-static');require('fs').cpSync(p.path,'/ffprobe');require('fs').chmodSync('/ffprobe',0o755)"
RUN npm prune --omit=dev

# ------------------------
# Runtime stage
# ------------------------
FROM node:22-slim
WORKDIR /app

# Install OpenSSL in runtime too
RUN apt-get update -y \
  && apt-get install -y openssl ca-certificates \
  && rm -rf /var/lib/apt/lists/*

ENV FFPROBE_PATH=/usr/local/bin/ffprobe
COPY --from=builder /ffprobe /usr/local/bin/ffprobe
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./
COPY package*.json ./

EXPOSE 5000
CMD ["node", "dist/main.js"]