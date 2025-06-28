FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/node:20-slim

WORKDIR /app

# Configure apt sources for faster downloads in China (清华大学源 - 使用HTTP避免SSL证书问题)
RUN echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list && \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware" >> /etc/apt/sources.list

# Copy package files first for better layer caching
COPY package*.json ./

# Configure npm to use Chinese mirror (阿里云源)
RUN npm config set registry https://registry.npmmirror.com

# Install build dependencies for node-gyp
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    make \
    g++ \
    build-essential \
    pkg-config \
    libcairo2-dev \
    libpango1.0-dev \
    libjpeg-dev \
    libgif-dev \
    librsvg2-dev \
    && rm -rf /var/lib/apt/lists/*

RUN npm install

# Copy the frontend code
COPY . .

# Build arguments for environment variables (will be passed from docker-compose)
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ARG NEXT_PUBLIC_BACKEND_URL=http://localhost:8000/api
ARG NEXT_PUBLIC_URL=http://localhost:3000
ARG NEXT_PUBLIC_ENV_MODE=production

# Set environment variables for build
ENV NEXT_PUBLIC_VERCEL_ENV=production
ENV NEXT_PUBLIC_SUPABASE_URL=${NEXT_PUBLIC_SUPABASE_URL}
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=${NEXT_PUBLIC_SUPABASE_ANON_KEY}
ENV NEXT_PUBLIC_BACKEND_URL=${NEXT_PUBLIC_BACKEND_URL}
ENV NEXT_PUBLIC_URL=${NEXT_PUBLIC_URL}
ENV NEXT_PUBLIC_ENV_MODE=${NEXT_PUBLIC_ENV_MODE}

# Set fallback values to prevent build failures when env vars are missing
ENV NEXT_PUBLIC_SUPABASE_URL=${NEXT_PUBLIC_SUPABASE_URL:-http://localhost:54321}
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=${NEXT_PUBLIC_SUPABASE_ANON_KEY:-dummy-key-for-build}

RUN npm run build

EXPOSE 3000

# Default command is dev, but can be overridden in docker-compose
CMD ["npm", "start"]