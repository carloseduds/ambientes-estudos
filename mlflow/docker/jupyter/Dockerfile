############## custom jupyter image ###############
ARG JUPYTER_BASE_IMAGE
ARG JUPYTER_BASE_VERSION
FROM jupyter/${JUPYTER_BASE_IMAGE}:${JUPYTER_BASE_VERSION} AS jupyter

ARG MLFLOW_ARTIFACT_STORE
ARG MLFLOW_VERSION
ARG JUPYTER_USERNAME
ENV HOME_DIR=/home/$JUPYTER_USERNAME
ENV WORK_DIR=$HOME_DIR/work

# Atualiza pip e instala dependências principais
USER root
RUN pip install --upgrade pip && \
    pip install \
        mlflow==${MLFLOW_VERSION} \
        psycopg2-binary \
        "protobuf<4.0.0" \
        "jupyterlab>=4.0.0" \
        "notebook>=7.0.0" \
        "jupyter_server>=2.0.0" \
        jupyter_contrib_nbextensions \
        jupyter_nbextensions_configurator \
        tqdm && \
    fix-permissions $CONDA_DIR

# Habilita extensões úteis no Jupyter
RUN ldconfig && \
    mkdir -p $MLFLOW_ARTIFACT_STORE

# Instala dependências do projeto
WORKDIR $WORK_DIR
COPY ./requirements.txt ./setup.py ./
COPY ./src/ ./src/
RUN chown -R $NB_UID:$NB_GID $HOME_DIR && \
    fix-permissions $HOME_DIR

USER $NB_UID
RUN pip install -r requirements.txt && \
    pip install -e .

################## test image ###################
FROM jupyter AS test

ARG TEST_DIR=/tests

COPY ./docker/jupyter/requirements.test.txt ${TEST_DIR}/
RUN pip install -r ${TEST_DIR}/requirements.test.txt

COPY ./tests/docker ${TEST_DIR}/docker
COPY ./tests/run_docker_tests.sh ${TEST_DIR}/
COPY ./docker/jupyter/scripts/wait-for-it.sh ${TEST_DIR}/

USER root
RUN chmod +x ${TEST_DIR}/wait-for-it.sh
USER $NB_UID

WORKDIR ${TEST_DIR}
