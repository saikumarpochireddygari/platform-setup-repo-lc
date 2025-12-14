FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      python3 python3-yaml python3-requests git && \
    rm -rf /var/lib/apt/lists/*

USER jenkins