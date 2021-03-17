# using Miniconda 3 as the base image...
FROM continuumio/miniconda3:4.7.12

LABEL maintainer="ashalper@usgs.gov"

# install additional packages
RUN apt-get update && \
    apt-get -y install dialog file gcc gfortran make procps unzip && \
    apt-get autoclean && apt-get purge

# It should only be necessary to set this to "-k" (the
# "conda update/install ..." option) at certain installations in
# .usgs.gov where the DOI root SSL certificate has not been installed
# (e.g., on a VM).
ARG insecure
RUN if [ "$insecure" = -k ]; then \
       wget http://sslhelp.doi.net/docs/DOIRootCA2.cer ; \
       # see
       # https://stackoverflow.com/questions/9072376/configure-git-to-accept-a-particular-self-signed-server-certificate-for-a-partic
       git config --global http.sslCAInfo DOIRootCA2.cer ; \
    fi

# Update/upgrade Conda
RUN conda update -n base conda -y $insecure
RUN conda install -n base -c defaults -c conda-forge $insecure \
      python=3.8 \
      bottleneck \
      cartopy \
      dask \
      geopandas \
      jupyter \
      matplotlib \
      netcdf4 \
      notebook \
      numpy \
      pandas \
      rasterio \
      rasterstats \
      scipy \
      xarray \
      xmltodict \
      geojason \
      metpy \
      rioxarray \
      shapely
RUN conda clean -a

ENV SOURCE_DIR=/usr/local/src

ARG VERSION_GRIDMETETL_S2S=0.1

RUN wget -P $NHM_SOURCE_DIR https://github.com/rmcd-mscb/gridmet_cfsv2/archive/$VERSION_GRIDMETETL_S2S.tar.gz
RUN cd $NHM_SOURCE_DIR && tar -xzf $VERSION_GRIDMETETL_S2S.tar.gz
RUN mv $NHM_SOURCE_DIR/gridmet_cfsv2-$VERSION_GRIDMETETL_S2S $NHM_SOURCE_DIR/gridmet_cfsv2
RUN rm $NHM_SOURCE_DIR/$VERSION_GRIDMETETL_S2S.tar.gz

RUN mkdir -p /var/lib/nhm/ofp/Output

ENV USER=nhm

RUN useradd -ms /bin/bash $USER

RUN chown -R $USER /var/lib/nhm
RUN chgrp -R $USER /var/lib/nhm

# shell functions used by more than one entry-point script
RUN mkdir -p /usr/local/share/nhm
COPY nhm.sh /usr/local/share/nhm
RUN chown -R $USER /usr/local/share/nhm
RUN chmod 744 /usr/local/share/nhm/nhm.sh

ENV HOMEDIR=/home/$USER

VOLUME ["/var/lib/nhm"]