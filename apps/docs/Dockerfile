# Use a node base image with Alpine for smaller image size
FROM node:18-alpine AS base

# The builder stage to install dependencies and prepare build artifacts
FROM base AS builder
RUN apk update
RUN apk add --no-cache libc6-compat
# Set the working directory for the app
WORKDIR /app

# Install turbo globally using npm (instead of yarn)
RUN npm install -g turbo

# Copy the entire repository to the container
COPY . .

# Generate a partial monorepo with a pruned lockfile for a target workspace.
RUN turbo prune docs --docker

# The installer stage to install dependencies
FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Copy .gitignore and other needed files
COPY .gitignore .gitignore

# Copy the pruned lockfile and other necessary files from the builder stage
COPY --from=builder /app/out/json/ .

# Install dependencies using npm
RUN npm install

# Build the project using turbo (run `npm run build` in your case)
COPY --from=builder /app/out/full/ .
RUN npm run build

# The runner stage to create a production-ready image
FROM base AS runner
WORKDIR /app

EXPOSE 3000

# Don't run production as root for security reasons
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

# Copy the necessary files from the installer stage
COPY --from=installer /app/apps/docs/next.config.js .
COPY --from=installer /app/apps/docs/package.json .

# Automatically leverage output traces to reduce image size
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/standalone ./ 
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/static ./apps/docs/.next/static 
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/public ./apps/docs/public

# Print out the environment variables for debugging
RUN echo "PORT: $PORT"

# Run the default Next.js start command (using the PORT environment variable)
CMD ["npm", "run", "start"]
