# MCP Server Dockerfile for Penpot
# This Dockerfile builds and runs the Penpot MCP Server

# Use Node.js 22 LTS (standard image for better compatibility)
FROM node:22 AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y python3 make g++ && rm -rf /var/lib/apt/lists/*

# Enable corepack for pnpm
RUN corepack enable && corepack prepare pnpm@10.31.0 --activate

# Set working directory
WORKDIR /app

# Copy package files first for better caching
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY packages/common/package.json ./packages/common/
COPY packages/server/package.json ./packages/server/
COPY packages/plugin/package.json ./packages/plugin/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy the rest of the application
COPY . .

# Build all components
RUN pnpm run build

# Production stage
FROM node:22 AS production

# Enable corepack for pnpm
RUN corepack enable && corepack prepare pnpm@10.31.0 --activate

# Set working directory
WORKDIR /app

# Copy package files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY packages/common/package.json ./packages/common/
COPY packages/server/package.json ./packages/server/
COPY packages/plugin/package.json ./packages/plugin/

# Copy built artifacts from builder
COPY --from=builder /app/packages/server/dist ./packages/server/dist
COPY --from=builder /app/packages/plugin/dist ./packages/plugin/dist
COPY --from=builder /app/packages/common/dist ./packages/common/dist

# Copy server data files
COPY --from=builder /app/packages/server/data ./packages/server/data

# Copy plugin source files needed for vite build --watch
COPY --from=builder /app/packages/plugin/vite.config.ts ./packages/plugin/
COPY --from=builder /app/packages/plugin/src ./packages/plugin/src/
COPY --from=builder /app/packages/plugin/index.html ./packages/plugin/
COPY --from=builder /app/packages/plugin/public ./packages/plugin/public/

# Install all dependencies (including devDependencies) - required for plugin
RUN pnpm install --frozen-lockfile

# Expose the necessary ports
# 4400 - Plugin web server
# 4401 - MCP HTTP/SSE server
# 4402 - WebSocket server (plugin connection)
# 4403 - REPL server (development/debugging)
EXPOSE 4400 4401 4402 4403

# Set environment variables
ENV NODE_ENV=production
ENV PENPOT_MCP_SERVER_HOST=0.0.0.0
ENV PENPOT_MCP_SERVER_PORT=4401
ENV PENPOT_MCP_WEBSOCKET_PORT=4402
ENV WS_URI=ws://penpot-mcp:4402

# Start both servers
CMD ["pnpm", "run", "start"]
