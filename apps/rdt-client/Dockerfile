# Stage 1 - Build the frontend
FROM node:18-alpine3.20 AS node-build-env

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
   dotnet publish RdtClient.Web/RdtClient.Web.csproj -c Release -o /appserver/out/web && \
   dotnet publish RdtClient.Service/RdtClient.Service.csproj -c Release -o /appserver/out/service && \
   dotnet publish RdtClient.Data/RdtClient.Data.csproj -c Release -o /appserver/out/data && \
   ls -la /appserver/out/web  # Debug to confirm the DLL is here

# Stage 3 - Build runtime image
FROM alpine:3.20

RUN apk add --no-cache bash icu-libs krb5-libs libgcc libintl libssl3 libstdc++ zlib curl

RUN mkdir -p /usr/share/dotnet && \
    wget https://download.visualstudio.microsoft.com/download/pr/7aa33fc7-07fe-48c2-8e44-a4bfb4928535/3b96ec50970eee414895ef3a5b188bcd/aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz && \
    tar zxf aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz -C /usr/share/dotnet && \
    ls -la /usr/share/dotnet

ENV PATH "$PATH:/usr/share/dotnet"

# Create the /config directory for the PVC and /app for the application
RUN mkdir -p /config/data /app /data && \
    chown -R nobody:nogroup /config /app /data

WORKDIR /app

# Copy backend output
COPY --from=dotnet-build-env /appserver/out/web /app
COPY --from=dotnet-build-env /appserver/out/service /app
COPY --from=dotnet-build-env /appserver/out/data /app

# Copy frontend output
COPY --from=node-build-env /appclient/client/out /app/wwwroot

USER nobody:nogroup

# Use /app for the .dll entry point
ENTRYPOINT ["dotnet", "/app/RdtClient.Web.dll"]

# Expose the port used by the application
EXPOSE 6500
