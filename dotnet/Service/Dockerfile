FROM mcr.microsoft.com/dotnet/aspnet:7.0-jammy AS base
WORKDIR /app
ENV TZ=Europe/Amsterdam
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN DEBIAN_FRONTEND=noninteractive apt-get update -y -qq
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -qq
EXPOSE 9001

#FROM mcr.microsoft.com/dotnet/sdk:8.0.100-rc.1-jammy-amd64 AS build
FROM mcr.microsoft.com/dotnet/sdk:7.0.401-jammy-amd64 AS build
RUN DEBIAN_FRONTEND=noninteractive apt-get update -y -qq
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -qq

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -qq \
    cmake \
    git \
    build-essential \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libgif-dev

RUN rm -rf /var/lib/apt/lists/*
WORKDIR /dotnetsrc
COPY *.sln .
COPY ./src ./src/
RUN dotnet restore
WORKDIR /dotnetsrc/src/kernel-memory/dotnet
RUN dotnet build Service -c Release --no-restore --property:BuildDir=/app/build

FROM build AS development
#RUN echo "$(ls -la)"
EXPOSE 9001
WORKDIR /dotnetsrc
CMD dotnet run --project src/kernel-memory/dotnet/Service --no-restore --cert /https/aspnetapp.pfx --password "password" dev-certs https --check

FROM build AS publish
WORKDIR /dotnetsrc
RUN dotnet publish src/kernel-memory/dotnet/Service -c Release --property:PublishDir=/app/publish --no-restore

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
CMD dotnet Service.dll