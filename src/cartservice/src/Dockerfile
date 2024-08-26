# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list

FROM mcr.microsoft.com/dotnet/sdk:6.0.425@sha256:6df1177e48b55272316d08f19cb383483af82aca5cdc67a76c414bc200847624 as builder

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends \
        make \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY cartservice.csproj .
RUN dotnet restore cartservice.csproj -r linux-musl-x64
COPY . .
RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore

# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
FROM mcr.microsoft.com/dotnet/runtime-deps:6.0.6-alpine3.14-amd64@sha256:f5402e55b58904ab2e94ec1a04d3b214ea653e0b97a0e876343ec90e77727f40
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.6 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe
WORKDIR /app
COPY --from=builder /cartservice .
ENV ASPNETCORE_URLS http://*:7070
ENTRYPOINT ["/app/cartservice"]
