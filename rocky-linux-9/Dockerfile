#    Copyright 2025, Chipovaneyyy @ Github
#
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
#
#        http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.
#
##########################################

#####################################
# Base image to install and run in
#####################################
FROM rockylinux:9 AS base

# Install dependencies
RUN dnf install -y pam \
    pam.i686 \
    ncurses-libs.i686 \
    file \
    libaio \
    libstdc++-devel.i686 \
    numactl-libs \
    libxcrypt-compat \
    && dnf clean all

RUN useradd -m -u 1000 db2adm \
    && echo "password" | passwd --stdin db2adm
ENV DB2_INSTALL_LOC="/home/db2adm/sqllib"

####################################
# Install DB2
####################################
FROM base AS install

# Edit the tar file name to match the one you have
ADD --chown=1000:1000 ./v11.5.9_linuxx64_server_dec.tar.gz /tmp/db2_installer
COPY --chown=1000:1000 ./response-file /tmp/db2_installer/response-file

USER db2adm
WORKDIR /home/db2adm

# exit 0 is a workaround needed in Rocky linux
# Installer does not create necessary links to correct libraries,
# they need to be created manually.
RUN /tmp/db2_installer/server_dec/db2setup -r /tmp/db2_installer/response-file; exit 0

# Fix missing links
RUN ln -s $DB2_INSTALL_LOC/lib64/awssdk/RHEL/9.2/libaws-cpp-sdk-core.so $DB2_INSTALL_LOC/lib64/libaws-cpp-sdk-core.so \
    && ln -s $DB2_INSTALL_LOC/lib64/awssdk/RHEL/9.2/libaws-cpp-sdk-transfer.so $DB2_INSTALL_LOC/lib64/libaws-cpp-sdk-transfer.so \
    && ln -s $DB2_INSTALL_LOC/lib64/awssdk/RHEL/9.2/libaws-cpp-sdk-s3.so $DB2_INSTALL_LOC/lib64/libaws-cpp-sdk-s3.so \
    && ln -s $DB2_INSTALL_LOC/lib64/awssdk/RHEL/9.2/libaws-cpp-sdk-kinesis.so $DB2_INSTALL_LOC/lib64/libaws-cpp-sdk-kinesis.so 

#####################################
# Final image with installed and configured DB2
#####################################
FROM base AS final

COPY --from=install --chown=1000:1000 /home/db2adm /home/db2adm

RUN sed -ri 's/(ENABLE_OS_AUTHENTICATION=).*/\1YES/g' $DB2_INSTALL_LOC/instance/db2rfe.cfg \
    && sed -ri 's/(RESERVE_REMOTE_CONNECTION=).*/\1YES/g' $DB2_INSTALL_LOC/instance/db2rfe.cfg \
    && sed -ri 's/^\*(SVCEPORT)=48000/\1=50000/g' $DB2_INSTALL_LOC/instance/db2rfe.cfg \
    && echo "0 localhost 0" > $DB2_INSTALL_LOC/db2nodes.cfg \
    && $DB2_INSTALL_LOC/instance/db2rfe -f $DB2_INSTALL_LOC/instance/db2rfe.cfg \
    && su - db2adm -c "db2 update dbm cfg using SVCENAME 50000" \
    && su - db2adm -c "db2set DB2COMM=TCPIP"

EXPOSE 50000
USER db2adm
WORKDIR /home/db2adm

COPY --chown=1000:1000 --chmod=700 ./entrypoint/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh", "start"]