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

RUN useradd -m -u 1000 db2adm
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
# Final image with installed DB2
#####################################
FROM base AS final

EXPOSE 50000
USER db2adm
WORKDIR /home/db2adm

COPY --from=install /home/db2adm /home/db2adm

# Set db conncetion port to 50000
# If done in entrypoint.sh, it throws an error. It doesn't throw an error here though
RUN source $DB2_INSTALL_LOC/db2profile \
    && echo "0 ${HOSTNAME} 0" > $DB2_INSTALL_LOC/db2nodes.cfg \
    && db2 update dbm cfg using SVCENAME 50000

COPY --chown=1000:1000 --chmod=700 ./entrypoint/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh", "start"]