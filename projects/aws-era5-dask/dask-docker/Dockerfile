FROM public.ecr.aws/docker/library/python:3.10-slim-bullseye

ENV PYTHONUNBUFFERED=TRUE
ENV DEBIAN_FRONTEND noninteractive
ENV CONDA_DIR="/opt/conda"
ENV PATH="${CONDA_DIR}/bin:${PATH}"

RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y bzip2 gcc wget curl \
  && rm -rf /var/lib/apt/lists/*
  
RUN arch=$(uname -m) && \
    if [ "${arch}" = "x86_64" ]; then \
        arch="64"; \
    fi && \
    curl -L https://micro.mamba.pm/api/micromamba/linux-${arch}/latest | tar -xvj bin/micromamba --strip-components=1

RUN mkdir -p "${CONDA_DIR}" && \
    mv micromamba /bin/micromamba  

RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.5/dumb-init_1.2.5_x86_64
RUN chmod +x /usr/local/bin/dumb-init

RUN micromamba install --root-prefix="${CONDA_DIR}" --prefix="${CONDA_DIR}" \
    -y -c conda-forge \
    python==3.10.8 \
    nomkl==1.0 \
    bokeh==2.4.3 \
    cloudpickle==2.2.0 \
    dask==2022.12.1 \
    distributed==2022.12.1 \
    fsspec==2022.11.0 \
    h5netcdf==1.1.0 \
    h5py==3.7.0 \
    intake==0.6.6 \
    intake-esm==2022.9.18 \
    kerchunk==0.0.9 \
    lz4==4.2.0 \
    ujson==5.5.0 \
    msgpack-python==1.0.4 \
    netcdf4==1.6.2 \
    numpy==1.24.1 \
    pandas==1.5.2 \
    python-blosc==1.10.6 \
    rechunker==0.5.0 \
    s3fs==2022.11.0 \
    tornado==6.2 \ 
    xarray==2022.12.0 \
    zarr==2.13.3 \
    zict==2.2.0 \
    && find /opt/conda/ -type f,l -name '*.a' -delete \
    && find /opt/conda/ -type f,l -name '*.pyc' -delete \
    && find /opt/conda/ -type f,l -name '*.js.map' -delete \
    && find /opt/conda/lib/python*/site-packages/bokeh/server/static -type f,l -name '*.js' -not -name '*.min.js' -delete \
    && rm -rf /opt/conda/pkgs

COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh

RUN mkdir /opt/app /etc/dask
COPY dask.yaml /etc/dask/

ENTRYPOINT ["/usr/local/bin/dumb-init", "/usr/bin/prepare.sh"]
