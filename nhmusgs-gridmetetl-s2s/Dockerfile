# using Miniconda 3 as the base image...
FROM continuumio/miniconda3:4.8.3

LABEL maintainer="rmcd@usgs.gov"

# install additional packages
RUN apt-get update && \
    apt-get -y install dialog file make unzip ca-certificates libspatialindex-dev && \
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
RUN conda install -n base -c conda-forge $insecure \
      python=3.8 \
      gdal \
      fiona 
      
RUN conda clean -a

ENV NHM_SOURCE_DIR=/usr/local/src

# download and install gridmetetl_s2s
ARG VERSION_GRIDMETETL_S2S=0.2.0
# RUN wget --no-check-certificate -P $NHM_SOURCE_DIR https://github.com/rmcd-mscb/gridmet_cfsv2/archive/refs/tags/$VERSION_GRIDMETETL_S2S.tar.gz
# RUN cd $NHM_SOURCE_DIR && tar -xzf $VERSION_GRIDMETETL_S2S.tar.gz
# RUN mv $NHM_SOURCE_DIR/gridmet_cfsv2-$VERSION_GRIDMETETL_S2S $NHM_SOURCE_DIR/gridmet_cfsv2
# RUN cd $NHM_SOURCE_DIR/gridmet_cfsv2 && pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install .
RUN pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install git+https://github.com/rmcd-mscb/gridmet_cfsv2.git@$VERSION_GRIDMETETL_S2S#egg=gridmet_cfsv2
# RUN rm $NHM_SOURCE_DIR/$VERSION_GRIDMETETL_S2S.tar.gz

#download and install grid2shp
ARG VERSION_GRD2SHP=0.2.0
# RUN wget --no-check-certificate -P $NHM_SOURCE_DIR https://github.com/nhm-usgs/grd2shp/archive/refs/tags/$VERSION_GRD2SHP.tar.gz
# RUN cd $NHM_SOURCE_DIR && tar -xzf $VERSION_GRD2SHP.tar.gz
# RUN mv $NHM_SOURCE_DIR/grd2shp-$VERSION_GRD2SHP $NHM_SOURCE_DIR/grid2shp
# RUN cd $NHM_SOURCE_DIR/grid2shp && pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install .
RUN pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install git+https://github.com/nhm-usgs/grd2shp.git@$VERSION_GRD2SHP#egg=grd2shp
# RUN rm $NHM_SOURCE_DIR/$VERSION_GRD2SHP.tar.gz

ENV USER=nhm

RUN useradd -ms /bin/bash $USER

COPY --chown=nhm gmetl_cfsv2 /usr/local/bin
RUN chmod 744 /usr/local/bin/gmetl_cfsv2

COPY --chown=nhm rungmetl_cfs.py /usr/local/bin
RUN chmod 744 /usr/local/bin/rungmetl_cfs.py

# RUN chown -R $USER /nhm
# RUN chgrp -R $USER /nhm

# # shell functions used by more than one entry-point script
# RUN mkdir -p /usr/local/share/nhm
# COPY nhm.sh /usr/local/share/nhm
# RUN chown -R $USER /usr/local/share/nhm
# RUN chmod 744 /usr/local/share/nhm/nhm.sh

ENV HOMEDIR=/home/$USER

VOLUME ["/nhm"]