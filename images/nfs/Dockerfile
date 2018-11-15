# Copyright 2018 The Rook Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#Portions of this file came from https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/deploy/docker/Dockerfile, which uses the same license.

FROM NFS_BASEIMAGE

# Build ganesha from source, installing deps and removing them in one line.
# Why?
# 1. Root_Id_Squash, only present in >= 2.4.0.3 which is not yet packaged
# 2. Set NFS_V4_RECOV_ROOT to /export
# 3. Use device major/minor as fsid major/minor to work on OverlayFS

#RUN dnf install -y tar gcc cmake autoconf libtool bison flex make gcc-c++ krb5-devel dbus-devel jemalloc-devel libnfsidmap-devel patch libnsl2-devel && dnf clean all 
RUN dnf install -y gcc


RUN mkdir -p /var/run/dbus
RUN mkdir -p /export

COPY start.sh /

EXPOSE 2049 38465-38467 662 111/udp 111

COPY rook /usr/local/bin/

ENTRYPOINT ["/usr/local/bin/rook"]
CMD [""]
