# Dockerfile changes

## Changes in this version with .Net 10.0 SDK

> This version combines multiple RUN commands into a single layer to reduce image size as well as performs final cleanup to minimize the image footprint.

```yml
FROM mcr.microsoft.com/dotnet/sdk:10.0

ENV USER="user" \
    HOME="/home/user" \
    PATH="/home/user/anaconda/bin:/home/user/.dotnet/tools:/home/user/anaconda/envs/py3.12/bin:${PATH}" \
    DOTNET_CLI_TELEMETRY_OPTOUT=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    wget && \
    rm -rf /var/lib/apt/lists/* && \
    useradd -ms /bin/bash $USER && \
    mkdir -p $HOME && \
    chown -R $USER:$USER $HOME

USER $USER
WORKDIR $HOME

RUN wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh && \
    chmod +x anaconda.sh && \
    ./anaconda.sh -b -p $HOME/anaconda && \
    rm ./anaconda.sh && \
    dotnet tool install -g --add-source "https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet-tools/nuget/v3/index.json" Microsoft.dotnet-interactive && \
    dotnet interactive jupyter install  && \
    conda create -n py3.12 python=3.12 -y && \
    conda install -n py3.12 notebook -y && \
    echo "source activate py3.12" > ~/.bashrc

# Final cleanup command (add this after all your installations)
RUN conda clean --all --yes && \
    rm -rf $HOME/anaconda/pkgs/cache/* && \
    rm -rf $HOME/.cache/pip/* 2>/dev/null || true && \
    dotnet nuget locals all --clear && \
    find $HOME/anaconda -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true && \
    find $HOME/anaconda -name "*.pyc" -delete && \
    find $HOME/anaconda -name "*.pyo" -delete && \
    find $HOME/anaconda -name '*.tar.bz2' -delete && \
    find $HOME/anaconda -name '*.conda' -delete && \
    rm -rf $HOME/anaconda/share/man/* 2>/dev/null || true && \
    rm -rf $HOME/anaconda/share/info/* 2>/dev/null || true && \
    rm -rf $HOME/.wget-hsts 2>/dev/null || true

EXPOSE 8888
ENTRYPOINT ["jupyter", "notebook", "--no-browser", "--ip=0.0.0.0"]
```

## Changes in this version with .Net 9.0.203 SDK

> This version combines multiple RUN commands into a single layer to reduce image size.

```yml
FROM mcr.microsoft.com/dotnet/sdk:9.0.203

ENV USER="user" \
    HOME="/home/user" \
    PATH="/home/user/anaconda/bin:/home/user/.dotnet/tools:/home/user/anaconda/envs/py3.12/bin:${PATH}" \
    DOTNET_CLI_TELEMETRY_OPTOUT=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    wget && \
    rm -rf /var/lib/apt/lists/* && \
    useradd -ms /bin/bash $USER && \
    mkdir -p $HOME && \
    chown -R $USER:$USER $HOME

USER $USER
WORKDIR $HOME

RUN wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh && \
    chmod +x anaconda.sh && \
    ./anaconda.sh -b -p $HOME/anaconda && \
    rm ./anaconda.sh && \
    dotnet tool install -g --add-source "https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet-tools/nuget/v3/index.json" Microsoft.dotnet-interactive && \
    dotnet interactive jupyter install  && \
    conda create -n py3.12 python=3.12 -y && \
    conda install -n py3.12 notebook -y && \
    echo "source activate py3.12" > ~/.bashrc

EXPOSE 8888
ENTRYPOINT ["jupyter", "notebook", "--no-browser", "--ip=0.0.0.0"]
```

## Original dockerfile for Jupyter .NET notebooks

```yml
FROM mcr.microsoft.com/dotnet/sdk:9.0.200

RUN apt install -y --no-install-recommends wget && \
    apt autoremove -y && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Set non-root user
ENV USER="user"
RUN useradd -ms /bin/bash $USER
USER $USER 
ENV HOME="/home/$USER"
WORKDIR $HOME

# Install Anaconda
RUN wget https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh -O anaconda.sh
RUN chmod +x anaconda.sh
RUN ./anaconda.sh -b -p $HOME/anaconda
RUN rm ./anaconda.sh
ENV PATH="/${HOME}/anaconda/bin:${PATH}"

# Install .NET kernel
RUN dotnet tool install -g --add-source "https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet-tools/nuget/v3/index.json" Microsoft.dotnet-interactive
ENV PATH="/${HOME}/.dotnet/tools:${PATH}"
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
RUN dotnet interactive jupyter install

# Create conda environment with Python 3.12
RUN conda create -n py3.12 python=3.12 -y
RUN echo "source activate py3.12" > ~/.bashrc
ENV PATH="/${HOME}/anaconda/envs/py3.12/bin:${PATH}"

# Install Jupyter Notebook
RUN conda install -n py3.12 notebook -y

# Run Jupyter Notebook
EXPOSE 8888
ENTRYPOINT ["jupyter", "notebook", "--no-browser", "--ip=0.0.0.0"]
```

