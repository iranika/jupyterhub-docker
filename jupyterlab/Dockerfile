FROM jupyter/scipy-notebook:8b98bb97bdae

LABEL maintainer="Luca De Feo <https://defeo.lu/>"

USER root

# APT packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    vim \
    iputils-ping \
    net-tools \
    curl \
    ssh
# install pwsh
RUN wget -q https://github.com/PowerShell/PowerShell/releases/download/v6.2.3/powershell_6.2.3-1.ubuntu.18.04_amd64.deb ; \
    dpkg -i powershell_6.2.3-1.ubuntu.18.04_amd64.deb ; \
    apt-get -y install -f

# install iruby
RUN sudo apt-get -y install libtool libffi-dev ruby ruby-dev make && \
    sudo apt-get -y install libzmq3-dev libczmq-dev && \
    gem install ffi-rzmq && \
    gem install iruby --pre && \
    iruby register --force

# crean apt
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# Python packages
ADD requirements.txt .
RUN pip install -r requirements.txt
RUN python -m bash_kernel.install
RUN python -m sshkernel install --sys-prefix
RUN python -m powershell_kernel.install --powershell-command pwsh
