#FROM mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim AS base
FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
EXPOSE 40386

#FROM mcr.microsoft.com/dotnet/sdk:5.0-buster-slim AS build
FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["EmployeeProject.csproj", "EmployeeProject/"]

ARG sonarscan=yes
RUN dotnet restore "EmployeeProject/EmployeeProject.csproj"
COPY  . "EmployeeProject/"
WORKDIR "/src/EmployeeProject"
RUN dotnet build "EmployeeProject.csproj" -c Release -o /app
RUN dotnet new tool-manifest
RUN pwd \
    && dotnet tool install --local --version 6.2.2 Swashbuckle.AspNetCore.Cli \
    && find / -name dotnet-tools.json \
    && cat /src/EmployeeProject/.config/dotnet-tools.json
    
RUN dotnet tool run swagger tofile --output swagger.json bin/Debug/netcoreapp3.1/EmployeeProject.dll v1
RUN dotnet tool install -g dotnet-sonarscanner --version 5.5.3 && dotnet tool install -g dotnet-reportgenerator-globaltool --version 5.1.1 \
ENV PATH "${PATH}:/root/.dotnet/tools" \
RUN if [ "$sonarscan" = "yes" ] ; then \
    
    	 dotnet sonarscanner begin \
	/k:testimplementation \
        /name:EmployeeProject \
        /d:sonar.host.url=http://3.109.121.132:9000/ \
        /d:sonar.branch.name=dindlocaldb \
	/d:sonar.login=b4254a0e97265d862b24b657ad70a784062719a3\
	/d:sonar.coverageReportPaths="coverage/SonarQube.xml" \
    && dotnet test --collect:"XPlat Code Coverage" --results-directory ./coverage  \
      && reportgenerator "-reports:./coverage/*/coverage.cobertura.xml" "-targetdir:coverage" "-reporttypes:SonarQube" ; exit 0 \
    && dotnet sonarscanner end 	/d:sonar.login=b4254a0e97265d862b24b657ad70a784062719a3 ; \
    else \
    echo "sonarscanning skipped"; \
    fi


FROM build AS publish
RUN dotnet publish "EmployeeProject.csproj" -c Release -o /app --no-restore

FROM base AS final
WORKDIR /app

COPY --from=publish /app .

ENTRYPOINT ["dotnet", "EmployeeProject.dll"]
