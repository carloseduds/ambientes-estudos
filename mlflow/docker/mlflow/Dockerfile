FROM python:3.10

EXPOSE 5000

ARG MLFLOW_VERSION=2.11.1
ARG SERVER_DIR=/server

RUN pip install --upgrade pip && \
    pip install "protobuf<4.0.0" "numpy<1.24" "sqlalchemy<1.4" && \
    pip install mlflow==$MLFLOW_VERSION psycopg2-binary && \
    mkdir -p ${SERVER_DIR}

COPY ./scripts/wait-for-it.sh ${SERVER_DIR}/
RUN chmod +x ${SERVER_DIR}/wait-for-it.sh

WORKDIR ${SERVER_DIR}

CMD ./wait-for-it.sh ${MLFLOW_BACKEND_STORE}:${POSTGRES_PORT} -t ${WAIT_FOR_IT_TIMEOUT} -- \
    mlflow server \
    --backend-store-uri postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${MLFLOW_BACKEND_STORE}:${POSTGRES_PORT}/${POSTGRES_DATABASE} \
    --default-artifact-root /${MLFLOW_ARTIFACT_STORE} \
    --host ${MLFLOW_TRACKING_SERVER_HOST} \
    --port ${MLFLOW_TRACKING_SERVER_PORT}
