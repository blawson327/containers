# Stage 1 - Build the frontend
FROM node:18-alpine3.18 AS node-build-env

RUN mkdir /appclient
WORKDIR /appclient

RUN apk add --no-cache git python3 py3-pip make g++

RUN \
   git clone https://github.com/rogerfar/rdt-client.git . && \
   cd client && \
   npm ci && \
   npx ng build --output-path=out

# Stage 2 - Build the backend
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS dotnet-build-env

RUN mkdir /appserver
WORKDIR /appserver

RUN apk add --no-cache git

RUN \
   git clone https://github.com/rogerfar/rdt-client.git . && \
   cd server && \
   dotnet restore --no-cache RdtClient.sln && \
   dotnet publish --no-restore -c Release -o out

# Stage 3 - Build runtime image
FROM alpine:3.18

RUN apk add --no-cache bash icu-libs krb5-libs libgcc libintl libssl3 libstdc++ zlib curl

RUN mkdir -p /usr/share/dotnet && \
    wget https://download.visualstudio.microsoft.com/download/pr/7aa33fc7-07fe-48c2-8e44-a4bfb4928535/3b96ec50970eee414895ef3a5b188bcd/aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz && \
    tar zxf aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz -C /usr/share/dotnet && \
    ls -la /usr/share/dotnet

ENV PATH "$PATH:/usr/share/dotnet"

# Create the /config directory
RUN mkdir -p /config/downloads /config/db

WORKDIR /app
COPY --from=dotnet-build-env /appserver/out /app
COPY --from=node-build-env /appclient/client/out ./wwwroot

# Move app and db files to /config
RUN mv /app /config/app && \
    mv /data/db /config/db && \
    chown -R 1000:1000 /config && \
    chmod -R 770 /config

USER nobody:nogroup

# Use /config/app for the .dll entry point
ENTRYPOINT ["dotnet", "/config/app/RdtClient.dll"]

# ports and volumes
EXPOSE 6500
VOLUME [ "/config" ]

# Healthcheck
HEALTHCHECK --interval=30s --timeout=30s --start-period=30s --retries=3 CMD curl --fail http://localhost:6500 || exit 1
