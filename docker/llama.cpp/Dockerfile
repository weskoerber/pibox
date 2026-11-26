ARG LLAMA_SRCDIR="/usr/src/llama.cpp-1.0.0"
ARG LLAMA_BUILDDIR="/usr/src/llama.cpp-1.0.0/build"
ARG LLAMA_INSTALLDIR="/usr/src/llama.cpp-1.0.0/install"

FROM debian:stable-slim AS build
ARG DEBIAN_FRONTEND=noninteractive

RUN apt update

# Must-haves
RUN apt install -y --no-install-recommends wget ca-certificates tar gzip

# llama.cpp build dependencies
RUN apt install -y --no-install-recommends cmake build-essential clang libssl-dev

# ROCm
RUN wget -qO /tmp/rocm.tar.gz https://repo.amd.com/rocm/tarball/therock-dist-linux-gfx110X-all-7.13.0.tar.gz
RUN <<EOF
mkdir /opt/rocm
tar -xzf /tmp/rocm.tar.gz -C /opt/rocm
rm /tmp/rocm.tar.gz
EOF

# llama.cpp
RUN wget -qO /tmp/llama.cpp.tar.gz https://github.com/ggml-org/llama.cpp/archive/refs/tags/v1.0.0.tar.gz
RUN <<EOF
tar -xzf /tmp/llama.cpp.tar.gz -C /usr/src/
rm /tmp/llama.cpp.tar.gz
EOF

ARG HIPCXX=/opt/rocm/lib/llvm/bin/clang
ARG HIP_PATH=/opt/rocm

ARG LLAMA_SRCDIR
ARG LLAMA_BUILDDIR
ARG LLAMA_INSTALLDIR

RUN \
    cmake \
        -S ${LLAMA_SRCDIR} \
        -B ${LLAMA_BUILDDIR} \
        -DGGML_HIP=ON \
        -DGGML_BLAS=OFF \
        -DGGML_CUDA=OFF \
        -DGGML_VULKAN=OFF \
        -DGGML_OPENCL=OFF \
        -DGGML_OPENMP=OFF \
        -DGGML_METAL=OFF \
        -DGGML_SYCL=OFF \
        -DGGML_MUSA=OFF \
        -DGPU_TARGETS=gfx1100 \
        -DLLAMA_BUILD_TESTS=OFF \
        -DLLAMA_BUILD_EXAMPLES=OFF \
        -DLLAMA_BUILD_UI=OFF \
        -DLLAMA_USE_PREBUILT_UI=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=${LLAMA_INSTALLDIR} && \
    cmake --build ${LLAMA_BUILDDIR} --config Release -j 16 && \
    cmake --install ${LLAMA_BUILDDIR} --config Release

FROM debian:stable-slim

ARG LLAMA_INSTALLDIR

COPY --from=build /opt/rocm/ /opt/rocm/
COPY --from=build ${LLAMA_INSTALLDIR}/bin/ /usr/bin/
COPY --from=build ${LLAMA_INSTALLDIR}/include/ /usr/include/
COPY --from=build ${LLAMA_INSTALLDIR}/lib/ /usr/lib/x86_64-linux-gnu/

RUN apt update

# ROCm runtime dependencies
RUN apt install -y --no-install-recommends libatomic1 libquadmath0 python3

RUN echo '/opt/rocm/lib' >> /etc/ld.so.conf && ldconfig

CMD ["llama"]
