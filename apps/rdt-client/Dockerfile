# Stage 1 - Build the frontend
FROM node:18-alpine3.18 AS node-build-env
ARG TARGETPLATFORM
ENV TARGETPLATFORM=${TARGETPLATFORM:-linux/amd64}
ARG BUILDPLATFORM
ENV BUILDPLATFORM=${BUILDPLATFORM:-linux/amd64}

RUN mkdir /appclient
WORKDIR /appclient

RUN apk add --no-cache git python3 py3-pip make g++

RUN \
   echo "**** Cloning Source Code ****" && \
   git clone https://github.com/rogerfar/rdt-client.git . && \
   cd client && \
   echo "**** Building Code  ****" && \
   npm ci && \
   npx ng build --output-path=out

RUN ls -FCla /appclient/root

# Stage 2 - Build the backend
FROM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS dotnet-build-env
ARG TARGETPLATFORM
ENV TARGETPLATFORM=${TARGETPLATFORM:-linux/amd64}
ARG BUILDPLATFORM
ENV BUILDPLATFORM=${BUILDPLATFORM:-linux/amd64}

RUN mkdir /appserver
WORKDIR /appserver

RUN apk add --no-cache git

RUN \
   echo "**** Cloning Source Code ****" && \
   git clone https://github.com/rogerfar/rdt-client.git . && \
   echo "**** Building Source Code for $TARGETPLATFORM on $BUILDPLATFORM ****" && \
   cd server && \
   if [ "$TARGETPLATFORM" = "linux/arm/v7" ] ; then \
      echo "**** Building $TARGETPLATFORM arm v7 version" && \
      dotnet restore --no-cache -r linux-arm RdtClient.sln && dotnet publish --no-restore -r linux-arm -c Release -o out ; \
   elif [ "$TARGETPLATFORM" = "linux/arm/v8" ] ; then \
      echo "**** Building $TARGETPLATFORM arm v8 version" && \
      dotnet restore --no-cache -r linux-arm64 RdtClient.sln && dotnet publish --no-restore -r linux-arm64 -c Release -o out ; \
   else \
      echo "**** Building $TARGETPLATFORM x64 version" && \
      dotnet restore --no-cache RdtClient.sln && dotnet publish --no-restore -c Release -o out ; \
   fi

# Stage 3 - Build runtime image
FROM alpine:3.20
ARG TARGETPLATFORM
ENV TARGETPLATFORM=${TARGETPLATFORM:-linux/amd64}
ARG BUILDPLATFORM
ENV BUILDPLATFORM=${BUILDPLATFORM:-linux/amd64}

# set version label
ARG BUILD_DATE
ARG VERSION
LABEL build_version="Custom version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="ravensorb"

# set environment variables
ENV XDG_CONFIG_HOME="/config/xdg"
ENV RDTCLIENT_BRANCH="main"

# Install dependencies
RUN \
    apk add --no-cache bash icu-libs krb5-libs libgcc libintl libssl3 libstdc++ zlib curl

# Install .NET runtime
RUN \
   mkdir -p /usr/share/dotnet && \
   if [ "$TARGETPLATFORM" = "linux/arm/v7" ] ; then \
      wget https://download.visualstudio.microsoft.com/download/pr/c3bf3103-efdb-42e0-af55-bbf861a4215b/dc22eda8877933b8c6569e3823f18d21/aspnetcore-runtime-8.0.0-linux-musl-arm64.tar.gz && \
      tar zxf aspnetcore-runtime-8.0.0-linux-musl-arm64.tar.gz -C /usr/share/dotnet ; \
   elif [ "$TARGETPLATFORM" = "linux/arm/v8" ] ; then \
      wget https://download.visualstudio.microsoft.com/download/pr/c3bf3103-efdb-42e0-af55-bbf861a4215b/dc22eda8877933b8c6569e3823f18d21/aspnetcore-runtime-8.0.0-linux-musl-arm64.tar.gz && \
      tar zxf aspnetcore-runtime-8.0.0-linux-musl-arm64.tar.gz -C /usr/share/dotnet ; \
   else \
      wget https://download.visualstudio.microsoft.com/download/pr/7aa33fc7-07fe-48c2-8e44-a4bfb4928535/3b96ec50970eee414895ef3a5b188bcd/aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz && \
      tar zxf aspnetcore-runtime-8.0.0-linux-musl-x64.tar.gz -C /usr/share/dotnet ; \
   fi

# Set environment variables for the app
ENV PATH "$PATH:/usr/share/dotnet"

# Copy files for app
WORKDIR /app
COPY --from=dotnet-build-env /appserver/server/out .
COPY --from=node-build-env /appclient/client/out ./wwwroot
COPY --from=node-build-env /appclient/root/ /

# ports and volumes
EXPOSE 6500
VOLUME [ "/data" ]


ENTRYPOINT ["dotnet", "/app/RdtClient.Web.dll"]

