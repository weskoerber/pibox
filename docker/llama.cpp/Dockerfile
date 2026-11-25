FROM debian:stable-slim

ARG DEBIAN_FRONTEND=noninteractive

RUN apt update && apt upgrade -y

# Must-haves
RUN apt install -y curl ca-certificates tar

# Download and extract tarballs early on in the build stage

# ROCm
RUN <<EOF
curl -fsSL -o /tmp/rocm.tar.gz https://stable.repo.amd.com/rocm/core/tarball/therock-dist-linux-multiarch-10.2.0.tar.gz
mkdir /opt/rocm
tar -xzf /tmp/rocm.tar.gz -C /opt/rocm
rm /tmp/rocm.tar.gz
EOF

# llama.cpp
RUN <<EOF
curl -fsSL -o /tmp/llama.cpp.tar.gz https://github.com/ggml-org/llama.cpp/archive/refs/tags/v1.0.0.tar.gz
tar -xzf /tmp/llama.cpp.tar.gz -C /opt/
rm /tmp/llama.cpp.tar.gz
EOF

# ROCm runtime dependencies
RUN apt install -y libatomic1 libquadmath0 python3

# Add ROCm to path
ENV PATH="/opt/rocm/bin:${PATH}"
ENV LD_LIBRARY_PATH="/opt/rocm/lib"

# llama.cpp build dependencies
RUN apt install -y build-essential cmake clang libssl-dev git

ARG HIPCXX=/opt/rocm/lib/llvm/bin/clang
ARG HIP_PATH=/opt/rocm

RUN cmake -S /opt/llama.cpp-1.0.0 -B /opt/llama.cpp-1.0.0/build -DGGML_HIP=ON -DLLAMA_BUILD_UI=OFF -DLLAMA_USE_PREBUILT_UI=OFF -DGPU_TARGETS=gfx1100 -DCMAKE_BUILD_TYPE=Release
RUN cmake --build /opt/llama.cpp-1.0.0/build --config Release -- -j 16

ENV PATH="/opt/llama.cpp-1.0.0/build/bin:${PATH}"
