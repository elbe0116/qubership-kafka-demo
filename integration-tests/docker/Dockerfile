FROM ghcr.io/elbe0116/qubership-docker-integration-tests-demo:PSUPATPOS-63

ENV ROBOT_OUTPUT=${ROBOT_HOME}/output \
    DISTR_DIR=/tmp/deps \
    SERVICE_CHECKER_SCRIPT=${ROBOT_HOME}/kafka_pods_checker.py \
    SERVICE_CHECKER_SCRIPT_TIMEOUT=500

RUN echo "https://dl-cdn.alpinelinux.org/alpine/v3.23/main" > /etc/apk/repositories && \
    echo "https://dl-cdn.alpinelinux.org/alpine/v3.23/community" >> /etc/apk/repositories

RUN mkdir -p ${ROBOT_HOME} \
    && mkdir -p ${ROBOT_OUTPUT}

COPY docker/requirements.txt ${ROBOT_HOME}/requirements.txt
COPY docker/kafka_pods_checker.py ${ROBOT_HOME}/kafka_pods_checker.py
COPY robot ${ROBOT_HOME}

# Upgrade all tools to avoid vulnerabilities
RUN set -x && apk upgrade --no-cache --available

RUN set -x \
    && pip3 install -r ${ROBOT_HOME}/requirements.txt \
    && rm -rf /var/cache/apk/*

# Switch to root temporarily to set correct permissions for AWS EKS security policies.
# The base image includes S3 adapter scripts that require write access to output directories.
# AWS EKS enforces non-root user execution, so we set ownership to user 1000 (non-root)
# and group 0 (root group) to allow proper file operations within the container.
USER root
RUN chown -R 1000:0 /opt/robot && \
    chmod -R g+rwX /opt/robot

# Run as non-root user for security compliance with AWS EKS policies
USER 1000:0

# Expose the port
EXPOSE 8080
VOLUME ["${ROBOT_OUTPUT}"]
