# Step 1: Use the updated base image (which now has Spark 3.5)
FROM liquidelake-base AS base

# Step 2: Main Image (TensorFlow + GPU + Jupyter)
FROM tensorflow/tensorflow:latest-gpu-jupyter

# Standard System Updates - Updated to OpenJDK 11
RUN apt-get -qq update  \
 && DEBIAN_FRONTEND=noninteractive apt-get -qq install --no-install-recommends \
      sudo \
      openjdk-11-jdk \
      curl \
      coreutils \
      libc6-dev \
 && rm -rf /var/lib/apt/lists/*

# Setup User (Keep your existing UID/GID logic)
ARG USERNAME=jupyter
ARG GROUPNAME=jupyter
ARG UID=1001
ARG GID=1001

RUN groupadd -g $GID $GROUPNAME || true \
 && useradd -m -s /bin/bash -u $UID -g $GID $USERNAME || true \
 && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
 && chmod 0440 /etc/sudoers.d/$USERNAME

# --- Updated Environment Variables ---
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV SPARK_HOME=/opt/spark
ENV HADOOP_HOME=/opt/hadoop
ENV HIVE_HOME=/opt/hive

# Copy tools from your updated base
COPY --from=base --chown=$USERNAME:$GROUPNAME /opt/hadoop /opt/hadoop
COPY --from=base --chown=$USERNAME:$GROUPNAME /opt/spark /opt/spark
COPY --from=base --chown=$USERNAME:$GROUPNAME /opt/hive /opt/hive

# --- Fix Python Path for Spark 3.5 ---
# Spark 3.5 uses py4j-0.10.9.7 (Verify this in your /opt/spark/python/lib folder)
ENV PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.9.7-src.zip
ENV PATH=$JAVA_HOME/bin:$SPARK_HOME/bin:$HADOOP_HOME/bin:$PATH

# --- Add Modern Connectors ---
USER root
RUN curl -s https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.4/hadoop-aws-3.3.4.jar -Lo /opt/spark/jars/hadoop-aws-3.3.4.jar \
 && curl -s https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.262/aws-java-sdk-bundle-1.12.262.jar -Lo /opt/spark/jars/aws-java-sdk-bundle-1.12.262.jar \
 && chown -R $USERNAME:$GROUPNAME /opt/spark/jars/

# Create Spark event log dir to prevent the HDFS URI error
RUN mkdir -p /tmp/spark-events && chown $USERNAME:$GROUPNAME /tmp/spark-events && chmod 1777 /tmp/spark-events

USER $USERNAME
RUN sudo pip install -U pip && \
    sudo pip install --no-cache-dir \
    pandas \
    numpy \
    pyspark==3.5.0 \
    findspark \
    tensorflow-serving-api

WORKDIR /home/$USERNAME
COPY run.sh /usr/local/sbin/run.sh
RUN sudo chmod a+x /usr/local/sbin/run.sh
CMD ["run.sh"]