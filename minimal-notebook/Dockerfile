###############################################################################
# Base on centos 6.6
###############################################################################
FROM centos:6.6
###############################################################################


USER root

###############################################################################
# Yum install dependens
###############################################################################
RUN yum update -y
RUN yum install -y epel-release
RUN yum groupinstall -y 'Development tools'
RUN yum install -y python-devel openssl-devel python-pip wget tar libcurl.x86_64 libcurl-devel.x86_64
RUN yum install -y which
RUN yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-headless javacc.x86_64
RUN pip install --upgrade pip
###############################################################################

###############################################################################
# Install Tini
###############################################################################
RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.9.0/tini && \
    echo "faafbfb5b079303691a939a747d7f60591f2143164093727e870b289a44d9872 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini
###############################################################################

###############################################################################
# Configure environment
###############################################################################
ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH
ENV SHELL /bin/bash
ENV NB_USER jovyan
ENV NB_UID 1000
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV JAVA_HOME /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/
ENV SCALA_HOME=/tmp/scala-2.11.8
ENV PATH $PATH:$SCALA_HOME/bin
###############################################################################

###############################################################################
# Create jovyan user with UID=1000 and in the 'users' group
###############################################################################
RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER
RUN mkdir -p /opt/conda
RUN chown jovyan /opt/conda
###############################################################################

###############################################################################
# Install conda as jovyan
###############################################################################
USER jovyan
WORKDIR /tmp
RUN mkdir -p $CONDA_DIR
COPY ./Anaconda2-2.5.0-Linux-x86_64.sh /tmp/
RUN /bin/bash Anaconda2-2.5.0-Linux-x86_64.sh -f -b -p $CONDA_DIR
RUN conda install --quiet --yes 'notebook=4.1*' terminado
RUN conda clean -tipsy
###############################################################################

###############################################################################
# Install python 2.7 and 3.5
###############################################################################
USER root
COPY ./install.sh /tmp/
RUN /bin/bash /tmp/install.sh
RUN ls -la /usr/local/share/jupyter/kernels/
###############################################################################


###############################################################################
# Install scala
###############################################################################
COPY ./scala-2.11.6.rpm /tmp/
RUN yum localinstall -y /tmp/scala-2.11.6.rpm
COPY ./jupyter-scala_2.11.6-0.2.0-SNAPSHOT /tmp/jupyter-scala_2.11.6-0.2.0-SNAPSHOT
RUN ls -al /tmp/
RUN /bin/bash ./jupyter-scala_2.11.6-0.2.0-SNAPSHOT/bin/jupyter-scala
RUN mv /root/.ipython/kernels/scala211 /usr/local/share/jupyter/kernels/

###############################################################################

###############################################################################
# Clear all
###############################################################################
RUN rm -f /tmp/Anaconda3-2.5.0-Linux-x86_64.sh
RUN rm -f /tmp/install.sh
###############################################################################

###############################################################################
# Setup jovyan home directory
###############################################################################
RUN mkdir /home/$NB_USER/work && \
    mkdir /home/$NB_USER/.jupyter && \
    mkdir /home/$NB_USER/.local
RUN chmod 777 -R /home/$NB_USER/ && chown $NB_USER /home/$NB_USER/work
###############################################################################

###############################################################################
# Configure container startup as root
###############################################################################
EXPOSE 8888
WORKDIR /home/$NB_USER/work
ENTRYPOINT ["tini", "--"]
CMD ["start-notebook.sh"]
###############################################################################

###############################################################################
# Add local files as late as possible to avoid cache busting
###############################################################################
COPY start-notebook.sh /usr/local/bin/
COPY jupyter_notebook_config.py /home/$NB_USER/.jupyter/
RUN chown -R $NB_USER:users /home/$NB_USER/.jupyter
###############################################################################

###############################################################################
# Development
###############################################################################
RUN pip install ipyparallel
RUN ipcluster nbextension enable
###############################################################################

###############################################################################
# Switch back to jovyan to avoid accidental container runs as root
###############################################################################
USER jovyan
###############################################################################
