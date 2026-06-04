# --- Etapa 1: Compilación ---
FROM node:22-alpine AS builder

RUN npm install -g pnpm
WORKDIR /app

# Copiamos los archivos de configuración
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY tsconfig.json nest-cli.json ./

# Copiamos el código fuente
COPY src ./src

# Instalamos todas las dependencias y compilamos el proyecto
RUN pnpm install --frozen-lockfile
RUN pnpm build

# --- Etapa 2: Imagen final de ejecución (Producción) ---
FROM node:22-alpine AS runner

RUN npm install -g pnpm
WORKDIR /app

EXPOSE 3000
ENV PORT=3000

# Copiamos las configuraciones necesarias para instalar en producción
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./

# Instalamos únicamente las dependencias de producción directamente en la etapa final
RUN pnpm install --prod --frozen-lockfile

# Copiamos la aplicación ya compilada desde la etapa anterior
COPY --from=builder /app/dist ./dist

CMD ["node", "dist/main.js"]