ARG REPO=mcr.microsoft.com/dotnet/aspnet
# Installer image
FROM $REPO:8.0.18-alpine3.21-arm64v8 AS installer

# Install .NET SDK
RUN dotnet_sdk_version=8.0.412 \
    && dotnet_version=8.0.18 \
    && wget \
        https://builds.dotnet.microsoft.com/dotnet/Sdk/$dotnet_sdk_version/dotnet-sdk-$dotnet_sdk_version-linux-musl-arm64.tar.gz \
        https://builds.dotnet.microsoft.com/dotnet/checksums/$dotnet_version-sha.txt \
    && awk -v file="dotnet-sdk-$dotnet_sdk_version-linux-musl-arm64.tar.gz" '{gsub(/\r/, "")} $2 == file' $dotnet_version-sha.txt | sha512sum -c \
    && mkdir --parents /dotnet \
    && tar --gzip --extract --no-same-owner --file dotnet-sdk-$dotnet_sdk_version-linux-musl-arm64.tar.gz --directory /dotnet ./packs ./sdk ./sdk-manifests ./templates ./LICENSE.txt ./ThirdPartyNotices.txt \
    && rm \
        dotnet-sdk-$dotnet_sdk_version-linux-musl-arm64.tar.gz \
        $dotnet_version-sha.txt


# .NET SDK image
FROM $REPO:8.0.18-alpine3.21-arm64v8

ENV \
    # Do not generate certificate
    DOTNET_GENERATE_ASPNET_CERTIFICATE=false \
    # Do not show first run text
    DOTNET_NOLOGO=true \
    # SDK version
    DOTNET_SDK_VERSION=8.0.412 \
    # Disable the invariant mode (set in base image)
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip \
    # PowerShell telemetry for docker image usage
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetSDK-Alpine-3.21-arm64

RUN apk add --upgrade --no-cache \
        curl \
        git \
        icu-data-full \
        icu-libs \
        tzdata

COPY --from=installer ["/dotnet", "/usr/share/dotnet"]

# Trigger first run experience by running arbitrary cmd
RUN dotnet help
