FROM python:3.9-bullseye

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      sudo \
      curl \
      vim \
      unzip \
      openjdk-11-jdk \
      build-essential \
      software-properties-common \
      ssh && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Jupyter and Python dependencies
COPY requirements.txt .    
RUN pip3 install -r requirements.txt

# Download and install IJava jupyter kernel
RUN curl https://github.com/SpencerPark/IJava/releases/download/v1.3.0/ijava-1.3.0.zip -Lo ijava-1.3.0.zip \
  && unzip ijava-1.3.0.zip \
  && python3 install.py --sys-prefix \
  && rm ijava-1.3.0.zip

## Download flink and hadoop dependencies and install

# Optional env variables
ENV HADOOP_VERSION 3.2.3
ENV HADOOP_URL https://www.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz
ENV HADOOP_HOME=${HADOOP_HOME:-"/opt/hadoop"}
ENV FLINK_VERSION 1.16.1
ENV FLINK_HOME=${FLINK_HOME:-"/opt/flink"}
ENV FLINK_URL=https://www.apache.org/dyn/closer.cgi?action=download&filename=flink/flink-${FLINK_VERSION}/flink-${FLINK_VERSION}-bin-scala_2.12.tgz


RUN mkdir -p ${HADOOP_HOME} && mkdir -p ${FLINK_HOME}
WORKDIR $FLINK_HOME

#Download Hadoop
RUN set -x \
    && curl -fSL $HADOOP_URL -o /tmp/hadoop.tar.gz \
    && tar -zxvf /tmp/hadoop.tar.gz --directory ${FLINK_HOME} --strip-components 1 \
    && rm /tmp/hadoop.tar.gz* \
    && ln -s ${HADOOP_HOME}/etc/hadoop /etc/hadoop
ENV PATH $HADOOP_HOME/bin/:$PATH

# Download flink
RUN set -x \
    && curl -fSL $FLINK_URL -o /tmp/flink.tar.gz \
    && tar -zxvf /tmp/flink.tar.gz --directory ${FLINK_HOME} --strip-components 1 \
    && rm /tmp/flink.tar.gz*

# Download postgres connector jar 
RUN curl https://jdbc.postgresql.org/download/postgresql-42.5.0.jar -o postgresql-42.5.0.jar \
 && mv postgresql-42.5.0.jar ${FLINK_HOME}/lib/ 

# Download iceberg flink runtime
RUN curl https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-flink-runtime-1.16/1.1.0/iceberg-flink-runtime-1.16-1.1.0.jar -Lo iceberg-flink-runtime-1.16-1.1.0.jar \
 && mv iceberg-flink-runtime-1.16-1.1.0.jar  ${FLINK_HOME}/lib/


# Download Java AWS SDK
RUN curl https://repo1.maven.org/maven2/software/amazon/awssdk/bundle/2.17.257/bundle-2.17.257.jar -Lo bundle-2.17.257.jar \
 && mv bundle-2.17.257.jar  ${FLINK_HOME}/lib/

# Download URL connection client required for S3FileIO
RUN curl https://repo1.maven.org/maven2/software/amazon/awssdk/url-connection-client/2.17.257/url-connection-client-2.17.257.jar -Lo url-connection-client-2.17.257.jar \
 && mv url-connection-client-2.17.257.jar  ${FLINK_HOME}/lib/

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
 && unzip awscliv2.zip \
 && sudo ./aws/install \
 && rm awscliv2.zip \
 && rm -rf aws/

# Add flink runtime jar to IJava classpath
ENV IJAVA_CLASSPATH=/opt/flink/lib/*


RUN mkdir -p /home/iceberg/data  /home/iceberg/notebooks /home/iceberg/warehouse

# Add a notebook command
RUN echo '#! /bin/sh' >> /bin/notebook \
 && echo "${FLINK_HOME}/bin/pyflink-shell.sh" >> /bin/notebook \
 && chmod u+x /bin/notebook

# Add Hadoop classpath to Flink config.sh
RUN echo 'export HADOOP_CLASSPATH=`hadoop classpath`' >> ${FLINK_HOME}/bin/config.sh

ENV PATH=$FLINK_HOME/bin:$PATH

COPY flink-conf.yaml ${FLINK_HOME}/conf/
COPY docker-entrypoint.sh .
ENTRYPOINT ["./docker-entrypoint.sh"]
EXPOSE 6123 8081 8888
CMD ["notebook"]