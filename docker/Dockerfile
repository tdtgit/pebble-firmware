ARG PYTHON_VERSION=3.13
ARG EM_VERSION=4.0.1
ARG EM_NODE_VERSION=20.18.0_64bit
ARG ARM_GNU_TOOLCHAIN_VERSION=14.2.rel1
ARG ARM_GNU_TOOLCHAIN_PLATFORM=x86_64-arm-none-eabi
ARG PEBBLE_BOARD_LIST="snowy_bb2 spalding_bb2 silk_bb2 asterix_evt1"

FROM python:${PYTHON_VERSION} AS base
RUN apt update && apt install -y gcc-multilib gettext clang libusb-1.0-0-dev

FROM base AS arm-toolchain
ARG ARM_GNU_TOOLCHAIN_VERSION
ARG ARM_GNU_TOOLCHAIN_PLATFORM
RUN curl -L "https://developer.arm.com/-/media/Files/downloads/gnu/${ARM_GNU_TOOLCHAIN_VERSION}/binrel/arm-gnu-toolchain-${ARM_GNU_TOOLCHAIN_VERSION}-${ARM_GNU_TOOLCHAIN_PLATFORM}.tar.xz" -o arm-gnu-toolchain.tar.xz && \
    tar xf arm-gnu-toolchain.tar.xz -C /opt && \
    mv /opt/arm-gnu-toolchain-${ARM_GNU_TOOLCHAIN_VERSION}-${ARM_GNU_TOOLCHAIN_PLATFORM} /opt/arm-gnu-toolchain

FROM base AS emsdk
ARG EM_VERSION
RUN git clone https://github.com/emscripten-core/emsdk.git /emsdk --depth 1 && \
    cd /emsdk && \
    ./emsdk install ${EM_VERSION} && \
    ./emsdk activate ${EM_VERSION}

FROM base AS openocd
RUN git clone https://github.com/pebble/openocd.git /openocd --depth 1 && \
    cd /openocd && \
    ./bootstrap && \
    ./configure --disable-werror --disable-ewarning && \
    make && \
    make install

FROM base AS runtime
COPY --from=arm-toolchain /opt/arm-gnu-toolchain /opt/arm-gnu-toolchain
COPY --from=openocd /usr/local /usr/local
COPY --from=emsdk /emsdk /emsdk
ARG EM_NODE_VERSION
ENV PATH="/emsdk:/emsdk/upstream/emscripten:/emsdk/node/${EM_NODE_VERSION}/bin:/opt/arm-gnu-toolchain/bin:${PATH}"

WORKDIR /app
COPY requirements.txt .
COPY python_libs/ python_libs/
RUN pip install -r requirements.txt

COPY . .

ENTRYPOINT ["/bin/bash", "-c"]