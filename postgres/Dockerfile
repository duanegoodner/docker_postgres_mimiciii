FROM postgres:latest

ARG PRIMARY_USER
ARG PRIMARY_USER_HOME=/home/${PRIMARY_USER}
ARG COMPONENTS=./postgres/components


# specifically for mimiciii
ARG LOCAL_MIMICIII_BUILD_TOOLS
ARG POSTGRES_ENTRYPOINT


SHELL [ "/bin/bash", "--login", "-c" ]

COPY ${COMPONENTS}/create_sudo_user /tmp/create_sudo_user
RUN /tmp/create_sudo_user/main.sh ${PRIMARY_USER} \
    && sudo rm -rf /tmp/create_sudo_user

RUN echo postgres ALL=\(root\) NOPASSWD:ALL \
  > /etc/sudoers.d/postgres
RUN chmod 0440 /etc/sudoers.d/postgres

USER ${PRIMARY_USER}

ARG BUILD_DIR=${PRIMARY_USER_HOME}/docker_build
WORKDIR ${BUILD_DIR}

COPY ${COMPONENTS}/install_apt_pkgs ${BUILD_DIR}/install_apt_pkgs
COPY ./postgres/pkglist ${BUILD_DIR}/pkglist
RUN ${BUILD_DIR}/install_apt_pkgs/main.sh ${BUILD_DIR}/pkglist \
    && sudo rm -rf ${BUILD_DIR}/*

COPY ${COMPONENTS}/zsh_setup ${BUILD_DIR}/zsh_setup
RUN ${BUILD_DIR}/zsh_setup/main.sh ${PRIMARY_USER} \
    && sudo rm -rf ${BUILD_DIR}/*

WORKDIR ${PRIMARY_USER_HOME}
RUN sudo rm -rf ${BUILD_DIR}


USER root
COPY ${LOCAL_MIMICIII_BUILD_TOOLS}/docker/setup.sh \
  ${POSTGRES_ENTRYPOINT}/setup.sh
COPY ${LOCAL_MIMICIII_BUILD_TOOLS}/docker \
  ${POSTGRES_ENTRYPOINT}/buildmimic/docker
COPY ${LOCAL_MIMICIII_BUILD_TOOLS}/postgres \
  ${POSTGRES_ENTRYPOINT}/buildmimic/postgres

COPY ./postgres/entrypoints/postgres_data_permissions \
  ${POSTGRES_ENTRYPOINT}/postgres_data_permissions/

# ENV LOCAL_DB_OWNER=${PRIMARY_USER}