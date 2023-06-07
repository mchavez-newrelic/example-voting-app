# because of dotnet, we always build on amd64, and target platforms in cli
# dotnet doesn't support QEMU for building or running. 
# (errors common in arm/v7 32bit) https://github.com/dotnet/dotnet-docker/issues/1537
# https://hub.docker.com/_/microsoft-dotnet
# hadolint ignore=DL3029
# to build for a different platform than your host, use --platform=<platform>
# for example, if you were on Intel (amd64) and wanted to build for ARM, you would use:
# docker buildx build --platform "linux/arm64/v8" .
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:7.0 as build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

WORKDIR /source
COPY *.csproj .
RUN case ${TARGETPLATFORM} in \
         "linux/amd64")  ARCH=x64  ;; \
         "linux/arm64")  ARCH=arm64  ;; \
         "linux/arm64/v8")  ARCH=arm64  ;; \
         "linux/arm/v7") ARCH=arm  ;; \
    esac \
    && dotnet restore -r linux-${ARCH}

COPY . .
RUN  case ${TARGETPLATFORM} in \
         "linux/amd64")  ARCH=x64  ;; \
         "linux/arm64")  ARCH=arm64  ;; \
         "linux/arm64/v8")  ARCH=arm64  ;; \
         "linux/arm/v7") ARCH=arm  ;; \
    esac \
    && dotnet publish -c release -o /app -r linux-${ARCH} --self-contained false --no-restore

# app image
FROM mcr.microsoft.com/dotnet/runtime:7.0

RUN apt-get update && apt-get install -y wget ca-certificates gnupg \
&& echo 'deb http://apt.newrelic.com/debian/ newrelic non-free' | tee /etc/apt/sources.list.d/newrelic.list \
&& wget https://download.newrelic.com/548C16BF.gpg \
&& apt-key add 548C16BF.gpg \
&& apt-get update \
&& apt-get install -y newrelic-dotnet-agent \
&& rm -rf /var/lib/apt/lists/*

ENV CORECLR_ENABLE_PROFILING=1 \
CORECLR_PROFILER={36032161-FFC0-4B61-B559-F6C5D41BAE5A} \
CORECLR_NEWRELIC_HOME=/usr/local/newrelic-dotnet-agent \
CORECLR_PROFILER_PATH=/usr/local/newrelic-dotnet-agent/libNewRelicProfiler.so \
NEW_RELIC_LICENSE_KEY=YOUR_LICENSE_KEY \
NEW_RELIC_APP_NAME=example-voting-app-worker

WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]