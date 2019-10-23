# Link to all the dockerfiles which this image is based upon. This provides info about various packages
# already present in the image. When updating the upstream image tag, remember to update the links as well.

# Minimal: https://github.com/jupyter/docker-stacks/tree/83ed2c63671f/minimal-notebook/Dockerfile
# Base: https://github.com/jupyter/docker-stacks/tree/83ed2c63671f/base-notebook/Dockerfile

FROM jupyter/datascience-notebook:2ce7c06a61a1 as miniconda
RUN conda update conda python && pip install -U 'pip==18.*'
RUN conda install --quiet --yes -c conda-forge \
    'jupyter_contrib_nbextensions=0.5*' \
    'jupyterlab=1.1.1' \
    'tqdm=4.35*' \
    'yapf=0.26*' \
    'nbdime=1.0*' \
    'jupyter-server-proxy=1.1.0' \
    'nbconvert=5.5*' && \
    conda install --quiet --yes -c damianavila82 'rise=5.4.*' && \
    conda clean -tipsy && \
    jupyter labextension install \
    '@jupyterlab/github@1.0.1' \
    'nbdime-jupyterlab@1.0.0' \
    '@jupyterlab/toc@1.0.1' \
    '@jupyterlab/hub-extension@1.0.5' && \
    pip install ipyparallel==6.2.* jupyterlab-github==1.0.0 escapism && \
    git clone https://github.com/paalka/nbresuse /tmp/nbresuse && pip install /tmp/nbresuse/ && \
    jupyter serverextension enable --py nbresuse --sys-prefix && \
    jupyter serverextension enable jupyter_server_proxy --sys-prefix && \
    jupyter nbextension install --py nbresuse --sys-prefix && \
    jupyter nbextension enable --py nbresuse --sys-prefix


FROM jupyter/datascience-notebook:2ce7c06a61a1
LABEL maintainer Uninett As <system@uninett.no>
USER root

# Setup ENV for Appstore to be picked up
ENV APP_UID=999 \
	APP_GID=999 \
	PKG_JUPYTER_NOTEBOOK_VERSION=5.7.x
RUN groupadd -g "$APP_GID" notebook && \
    useradd -m -s /bin/bash -N -u "$APP_UID" -g notebook notebook && \
    usermod -G users notebook && chmod go+rwx -R "$CONDA_DIR/bin"
COPY --chown=notebook:notebook --from=miniconda $CONDA_DIR $CONDA_DIR
# hadolint ignore=DL3002
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update && apt-get install -y --no-install-recommends \
	openssh-client=1:7.6p1-4 \
	less=487-0.1 \
	net-tools=1.60+git20161116.90da8a0-1ubuntu1 \
	man-db=2.8.3-2 \
	iputils-ping=3:20161105-1ubuntu2 \
	screen=4.6.2-1 \
	tmux=2.6-3 \
	graphviz=2.40.1-2 \
	cmake \
	rsync=3.1.2-2.1ubuntu1 \
	p7zip-full=16.02+dfsg-6 \
	tzdata \
	vim \
	unrar=1:5.5.8-1 \
        "openmpi-bin=2.1.1-8" && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/* && \
    ln -sf /usr/share/zoneinfo/Europe/Oslo /etc/localtime


ENV HOME=/home/notebook \
    XDG_CACHE_HOME=/home/notebook/.cache/
COPY normalize-username.py start-notebook.sh /usr/local/bin/
COPY --chown=notebook:notebook .jupyter/ /opt/.jupyter/
RUN mkdir -p /home/notebook/.ipython/profile_default/security/ && chmod go+rwx -R "$CONDA_DIR/bin" && chown notebook:notebook -R "$CONDA_DIR/bin" "$HOME" && \
    mkdir -p "$CONDA_DIR/.condatmp" && chmod go+rwx "$CONDA_DIR/.condatmp" && chown notebook:notebook "$CONDA_DIR"

# hadolint ignore=DL3002
RUN chmod go+w -R "$HOME" && chmod o+w /home

USER notebook
WORKDIR $HOME
CMD ["/usr/local/bin/start-notebook.sh"]
