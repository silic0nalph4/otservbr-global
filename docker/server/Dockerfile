# Stage 1: Download all dependencies
FROM ubuntu:20.04 AS dependencies

RUN apt-get install -y --no-install-recommends \
	git build-essential ca-certificates curl zip unzip tar pkg-config libluajit-5.1-dev wget

# vcpkg cannot yet bootstrap aarch64 binaries so we need system installs
# https://github.com/microsoft/vcpkg/discussions/17738
ENV VCPKG_FORCE_SYSTEM_BINARIES=1
RUN apt-get update && apt-get install -y ninja-build
# Manually install aarch64 cmake (this command not good for other platforms)
RUN wget -qO- "https://cmake.org/files/v3.21/cmake-3.21.2-linux-aarch64.tar.gz" | \
  tar --strip-components=1 -xz -C /usr/local

WORKDIR /opt
RUN git clone https://github.com/microsoft/vcpkg
RUN ./vcpkg/bootstrap-vcpkg.sh

WORKDIR /opt/vcpkg
COPY vcpkg.json /opt/vcpkg/
RUN /opt/vcpkg/vcpkg --feature-flags=binarycaching,manifests,versions install

# Stage 2: create build
FROM dependencies AS build

# TODO Will this work with system cmake?
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
# Use local config rather than .dist
COPY config.lua /otbr/config.lua

WORKDIR /otbr

# Configure config for docker-compose database
RUN sed -i '/mysqlHost = .*$/c\mysqlHost = "mysql"' config.lua \
  && sed -i '/mysqlUser = .*$/c\mysqlUser = "otserver"' config.lua \
  && sed -i '/mysqlPass = .*$/c\mysqlPass = "otserver"' config.lua \
  && sed -i '/mysqlDatabase = .*$/c\mysqlDatabase = "otserver"' config.lua \
  && unzip -o data/world/world.zip -d data/world/

VOLUME /otbr

ENTRYPOINT ["/bin/otbr"]
