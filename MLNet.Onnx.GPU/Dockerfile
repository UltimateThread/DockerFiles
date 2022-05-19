# WINDOWS 10/11/Server BEFORE ANYTHING ELSE YOU NEED TO SET UP DOCKER ALLOW ACCESS YOUR NVIDIA GPU
# Here is a good tutorial on how to get it set up
# https://gregbouwens.com/docker-with-gpu-enabled-on-windows/

# The Tensorrt NVIDIA base image has the CUDA and cuDNN versions needed for Microsoft.ML.OnnxRuntime.Gpu (1.10.0)
# https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html#requirements
# CUDA 11.4
# cuDNN 8.2.4 (Linux)
# https://docs.nvidia.com/deeplearning/tensorrt/container-release-notes/rel_21-10.html#rel_21-10
FROM nvcr.io/nvidia/tensorrt:21.10-py3

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        \
        # .NET dependencies
        libc6 \
        libgcc1 \
        libgssapi-krb5-2 \
        libicu66 \
        libssl1.1 \
        libstdc++6 \
        zlib1g \
        libgdiplus \
        wget \
        software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# Install .NET SDK and runtimes silently
RUN curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin -Channel 6.0 -InstallDir /usr/share/dotnet \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet

WORKDIR /src
COPY . .
RUN dotnet publish "MLNetOnnxGPU.csproj" -c Debug -o /app/publish --runtime "linux-x64"

# Copy the Onnx runtimes to a specified directory and set the LD_LIBRARY_PATH so the application can find it
RUN mkdir -p /onnx
RUN cp /app/publish/libLdaNative.so /onnx/
RUN cp /app/publish/libonnxruntime.so /onnx/
RUN cp /app/publish/libonnxruntime_providers_cuda.so /onnx/
RUN cp /app/publish/libonnxruntime_providers_shared.so /onnx/

ENV PATH "$PATH:/onnx"
ENV LD_LIBRARY_PATH "$LD_LIBRARY_PATH:/onnx"

ENTRYPOINT ["dotnet", "/app/publish/MLNetOnnxGPU.dll"]