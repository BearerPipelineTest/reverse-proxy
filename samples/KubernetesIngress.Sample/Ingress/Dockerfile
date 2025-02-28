#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM amd64/buildpack-deps:jammy-curl AS publish
WORKDIR /src
COPY ["global.json", ""]
RUN sed 's/\r$//' global.json > global-lf.json
RUN curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --jsonfile global-lf.json
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT 1

# Copy csproj files and other files needed for restoring (to build a nuget cache layer to speed up rebuilds)
COPY ["samples/KubernetesIngress.Sample/Ingress/Yarp.Kubernetes.Ingress.csproj", "samples/KubernetesIngress.Sample/Ingress/"]
COPY ["src/ReverseProxy/Yarp.ReverseProxy.csproj", "src/ReverseProxy/"]
COPY ["src/Kubernetes.Controller/Yarp.Kubernetes.Controller.csproj", "src/Kubernetes.Controller/"]
COPY ["src/Directory.Build.props", "src/"]
COPY ["Directory.Build.*", "./"]
COPY ["NuGet.config", ""]

# Build a cache layer with all of the nuget packages
RUN /root/.dotnet/dotnet restore samples/KubernetesIngress.Sample/Ingress/Yarp.Kubernetes.Ingress.csproj

# Copy the remaining source files
WORKDIR /src
COPY . .

WORKDIR /src/samples/KubernetesIngress.Sample/Ingress/
RUN /root/.dotnet/dotnet publish -c Release --no-restore -o /app/publish -f net6.0

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Yarp.Kubernetes.Ingress.dll"]
