# Stage 1: Download all dependencies
FROM ubuntu:20.04 AS dependencies

# Need latest cmake for ARM64 support
RUN apt-get update && apt-get install -y wget
RUN wget -qO- "https://cmake.org/files/v3.21/cmake-3.21.2-linux-aarch64.tar.gz" | \
  tar --strip-components=1 -xz -C /usr/local

RUN apt-get update && apt-get install -y --no-install-recommends cmake ninja-build

RUN apt-get update && apt-get install -y --no-install-recommends git

RUN apt-get install -y --no-install-recommends \
	build-essential ca-certificates curl zip unzip tar pkg-config

RUN apt-get install -y --no-install-recommends libluajit-5.1-dev unzip

WORKDIR /opt
RUN git clone https://github.com/microsoft/vcpkg
RUN ./vcpkg/bootstrap-vcpkg.sh

WORKDIR /opt/vcpkg
COPY vcpkg.json /opt/vcpkg/
# Required for ARM64 support
ENV VCPKG_FORCE_SYSTEM_BINARIES=1
RUN /opt/vcpkg/vcpkg --feature-flags=binarycaching,manifests,versions install

# Stage 2: create build
FROM dependencies AS build

COPY cmake /srv/cmake
COPY CMakeLists.txt vcpkg.json /srv/
COPY src /srv/src

WORKDIR /srv/build

RUN cmake -DCMAKE_TOOLCHAIN_FILE=/opt/vcpkg/scripts/buildsystems/vcpkg.cmake .. && make -j`nproc`

# Stage 3: load data and execute
FROM build

COPY --from=build /srv/build/bin/otbr /bin/otbr
COPY LICENSE *.sql key.pem /otbr/
COPY data /otbr/data
COPY config.lua.dist /otbr/config.lua

WORKDIR /otbr

RUN sed -i '/mysqlHost = .*$/c\mysqlHost = "mysql"' config.lua \
  && sed -i '/mysqlUser = .*$/c\mysqlUser = "otserver"' config.lua \
  && sed -i '/mysqlPass = .*$/c\mysqlPass = "otserver"' config.lua \
  && sed -i '/mysqlDatabase = .*$/c\mysqlDatabase = "otserver"' config.lua \
  && unzip -o data/world/world.zip -d data/world/

VOLUME /otbr

ENTRYPOINT ["/bin/otbr"]

