# docker build -t ibmresilient/fn_ansible:1.1.0 -t ibmresilient/fn_ansible:latest .

# Base image using Red Hat's universal base image (rhel 8) for python
FROM registry.access.redhat.com/ubi8/python-36:latest

ARG APPLICATION=fn_ansible
ARG RESILIENT_CIRCUITS_VERSION=37.0
ARG PATH_RESILIENT_CIRCUITS=rescircuits

# Update to latest packages, user 0 for root privilege
USER 0

# Update to latest pip
RUN pip install --upgrade pip

# install resilient-circuits
RUN pip install "resilient-circuits>=${RESILIENT_CIRCUITS_VERSION}"

## ---- section for changes ----
# uncomment to support sshpass in your container
#RUN wget http://mirror.pit.teraswitch.com/fedora/epel/epel-release-latest-8.noarch.rpm && \
#    rpm -ivh epel-release-latest-8.noarch.rpm && \
#    yum --enablerepo=epel -y install sshpass

# install the base package
COPY ./dist /tmp/packages
RUN pip install /tmp/packages/${APPLICATION}-*.tar.gz

# uncomment and replicate if additional pypi packages are needed
#RUN pip install <package>

# uncomment and replicate if additional local packages are needed
#COPY /path/to/extra_package /tmp/packages/.
#RUN pip install /tmp/packages/<extra_package>*.tar.gz

# uncomment to expose port only if a custom threat feed
#EXPOSE 9000
## ---- end section for changes ----

# set up configuration and log locations using /etc and /var/log, the conventional locations for config and logs
RUN mkdir /etc/${PATH_RESILIENT_CIRCUITS}
ENV APP_CONFIG_FILE /etc/${PATH_RESILIENT_CIRCUITS}/app.config

# create arbitrary group for user 1001
RUN groupadd -g 1001 default && usermod -g 1001 default

# create directory for logs and set to be root group to allow access by non root processes
# See https://docs.openshift.com/container-platform/4.2/openshift_images/create-images.html#images-create-guide-openshift_create-images
RUN mkdir /var/log/${PATH_RESILIENT_CIRCUITS} && \
    chgrp -R 1001 /var/log/${PATH_RESILIENT_CIRCUITS} && \
    chmod -R g=u /var/log/${PATH_RESILIENT_CIRCUITS}
ENV APP_LOG_DIR /var/log/${PATH_RESILIENT_CIRCUITS}

# setup entrypoint for read-only enterprise data used by integration, if needed
RUN mkdir /var/${PATH_RESILIENT_CIRCUITS}

# ansible directory structure
RUN mkdir -p /var/${PATH_RESILIENT_CIRCUITS}/ansible/env && \
    chgrp -R 1001 /var/${PATH_RESILIENT_CIRCUITS}/ansible && \
    chmod -R g=u /var/${PATH_RESILIENT_CIRCUITS}/ansible
# ansible artifacts
RUN mkdir /tmp/artifacts

# entrypoint for resilient-circuits.  Use /opt, the conventional location for optional software on Linux
RUN mkdir /opt/${PATH_RESILIENT_CIRCUITS}
COPY entrypoint.sh /opt/${PATH_RESILIENT_CIRCUITS}/entrypoint.sh

# arbitrary user, support running as non-root. Required on OpenShift. Generally a good practice.
USER 1001
ENTRYPOINT [ "sh", "/opt/rescircuits/entrypoint.sh" ]
