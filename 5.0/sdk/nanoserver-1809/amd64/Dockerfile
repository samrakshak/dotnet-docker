# escape=`

# Installer image
FROM mcr.microsoft.com/windows/servercore:1809 AS installer

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Retrieve .NET Core SDK
RUN $dotnet_sdk_version = '5.0.100-preview.1.20118.1'; `
    Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.azureedge.net/dotnet/Sdk/$dotnet_sdk_version/dotnet-sdk-$dotnet_sdk_version-win-x64.zip; `
    $dotnet_sha512 = 'b6779084c887ca1cb358e2191a60da79c068f31b34896072c2d4057477af8c4379a60489b0585dbcca90dcd5697037081d8c9718e5f13380962d041ee5eba357'; `
    if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    Expand-Archive dotnet.zip -DestinationPath dotnet; `
    Remove-Item -Force dotnet.zip

# Install PowerShell global tool
RUN $powershell_version = '7.0.0-rc.3'; `
    Invoke-WebRequest -OutFile PowerShell.Windows.x64.$powershell_version.nupkg https://pwshtool.blob.core.windows.net/tool/$powershell_version/PowerShell.Windows.x64.$powershell_version.nupkg; `
    $powershell_sha512 = 'be5790718b6d5e47b2389ee42bf758784d17b697e28f94dc391f2e4d26d5f0e1249ccd17014a3604223fe4ff2bbb8166cea6257159b1f193606f49d143115b80'; `
    if ((Get-FileHash PowerShell.Windows.x64.$powershell_version.nupkg -Algorithm sha512).Hash -ne $powershell_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    \dotnet\dotnet tool install --add-source . --tool-path \powershell --version $powershell_version PowerShell.Windows.x64; `
    \dotnet\dotnet nuget locals all --clear; `
    Remove-Item -Force PowerShell.Windows.x64.$powershell_version.nupkg; `
    Remove-Item -Path \powershell\.store\powershell.windows.x64\$powershell_version\powershell.windows.x64\$powershell_version\powershell.windows.x64.$powershell_version.nupkg -Force

# SDK image
FROM mcr.microsoft.com/windows/nanoserver:1809

ENV `
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip `
    # PowerShell telemetry for docker image usage
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetCoreSDK-NanoServer-1809

# In order to set system PATH, ContainerAdministrator must be used
USER ContainerAdministrator
RUN setx /M PATH "%PATH%;C:\Program Files\dotnet;C:\Program Files\powershell"
USER ContainerUser

COPY --from=installer ["/dotnet", "/Program Files/dotnet"]

COPY --from=installer ["/powershell", "/Program Files/powershell"]

# Trigger first run experience by running arbitrary cmd
RUN dotnet help