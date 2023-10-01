# builder step used to download and configure spark environment
FROM openjdk:11-jdk AS jdk

FROM python:3.11

USER root

# --------------------------------------------------------
# JAVA
# --------------------------------------------------------
RUN apt update
RUN apt-get install -y --no-install-recommends \
    python3-launchpadlib \
    software-properties-common
# RUN add-apt-repository ppa:openjdk-r/ppa
# RUN apt update
# RUN apt install -y --no-install-recommends \
#     openjdk-8-jdk
# For AMD based architecture use
# ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
COPY --from=jdk /usr/local/openjdk-11 /usr/lib/jvm/java-11-openjdk-arm64/
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-arm64/


# --------------------------------------------------------
# SPARK
# --------------------------------------------------------

ENV SPARK_VERSION spark-3.3.3
ENV SPARK_URL https://downloads.apache.org/spark/${SPARK_VERSION}/${SPARK_VERSION}-bin-hadoop3.tgz
ENV SPARK_HOME=/opt/$SPARK_VERSION
ENV PATH $SPARK_HOME/bin:$PATH
# ENV HADOOP_CONF_DIR=$SPARK_HOME/conf
ENV PYSPARK_PYTHON=python3
ENV PYTHONHASHSEED=1

RUN set -x \
    && curl -fSL "${SPARK_URL}" -o /tmp/spark.tar.gz \
    && tar -xvzf /tmp/spark.tar.gz -C /opt/ \
    && rm /tmp/spark.tar.gz*


ADD conf/core-site.xml $SPARK_HOME/conf
ADD conf/yarn-site.xml $SPARK_HOME/conf


WORKDIR $SPARK_HOME

ENV SPARK_MASTER_PORT=7077 \
SPARK_MASTER_WEBUI_PORT=8081 \
SPARK_LOG_DIR=/opt/spark/logs \
SPARK_MASTER_LOG=/opt/spark/logs/spark-master.out \
SPARK_WORKER_LOG=/opt/spark/logs/spark-worker.out \
SPARK_WORKER_WEBUI_PORT=8081 \
SPARK_WORKER_PORT=7000 \
SPARK_MASTER="spark://spark-master:7077" \  
SPARK_WORKLOAD="master"

EXPOSE 8081 7077 6066

RUN mkdir -p $SPARK_LOG_DIR && \
touch $SPARK_MASTER_LOG && \
touch $SPARK_WORKER_LOG && \
ln -sf /dev/stdout $SPARK_MASTER_LOG && \
ln -sf /dev/stdout $SPARK_WORKER_LOG

ADD spark/start-spark.sh start-spark.sh

RUN chmod a+x start-spark.sh

CMD ["start-spark.sh"]