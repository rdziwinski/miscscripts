# syntax = docker/dockerfile:1.0-experimental
ARG SUBBUILD=1.13
FROM pnnlmiscscripts/rpms-node-base:latest
FROM pnnlmiscscripts/rpms-containerd:latest
FROM pnnlmiscscripts/rpms-openvswitch:latest
FROM pnnlmiscscripts/rpms-kubernetes:$SUBBUILD

FROM centos:centos7
COPY --from=0 /data /rpmdata
COPY --from=1 /data /data/containerd
COPY --from=2 /data /data/openvswitch
COPY --from=3 /data /data/kubernetes
ADD rpm.pub /root/rpm.pub
RUN --mount=type=secret,id=gpg \
  yum install -y createrepo yum-utils gnupg2 && \
  mkdir -p /data/repodata && \
  zcat /rpmdata/repodata/*comps.xml.gz > /data/repodata/comps.xml && \
  cp -a /rpmdata/*.rpm /data && \
  createrepo -g /data/repodata/comps.xml /data && \
  gpg --import /run/secrets/gpg && \
  gpg --import /root/rpm.pub && \
  gpg --detach-sign --armor /data/repodata/repomd.xml && \
  rm -rf ~/.gnupg && \
  echo Done building repo.

FROM pnnlmiscscripts/rpms-node-base:latest
COPY --from=2 /data /data/openvswitch
COPY --from=1 /data /data/containerd
COPY --from=3 /data /data/kubernetes
COPY --from=4 /data/repodata /data/repodata

