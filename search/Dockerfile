ARG SEARCH_TAG
FROM quay.io/alfresco/search-services:${SEARCH_TAG}

# COMMON
ARG ALFRESCO_HOSTNAME
ARG SOLR_HOSTNAME
ENV ALFRESCO_HOSTNAME $ALFRESCO_HOSTNAME
ENV SOLR_HOSTNAME $SOLR_HOSTNAME

# Configure Alfresco Service Name
RUN sed -i '/^bash.*/i sed -i "'"s/alfresco.host=localhost/alfresco.host=${ALFRESCO_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh && \
    sed -i '/^bash.*/i sed -i "'"s/solr.host=localhost/solr.host=${SOLR_HOSTNAME}/g"'" ${DIST_DIR}/solrhome/conf/shared.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh

# COMMS
ARG ALFRESCO_COMMS
ENV ALFRESCO_COMMS $ALFRESCO_COMMS

# Configure SOLR cores to run in HTTPs mode from template
RUN if [ "$ALFRESCO_COMMS" == "https" ] ; then \
    sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=none/alfresco.secureComms=https/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
else \
    sed -i '/^bash.*/i sed -i "'"s/alfresco.secureComms=https/alfresco.secureComms=none/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi

# SSL
ARG TRUSTSTORE_TYPE
ENV TRUSTSTORE_TYPE $TRUSTSTORE_TYPE
ARG KEYSTORE_TYPE
ENV KEYSTORE_TYPE $KEYSTORE_TYPE

# Set SSL properties
RUN if [ "$ALFRESCO_COMMS" == "https" ] ; then \
    sed -i '/^bash.*/i \
    sed -i "'"s/alfresco.encryption.ssl.keystore.location=.*/alfresco.encryption.ssl.keystore.location=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl.repo.client.keystore/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
    sed -i "'"s/alfresco.encryption.ssl.keystore.passwordFileLocation=.*/alfresco.encryption.ssl.keystore.passwordFileLocation=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl-keystore-passwords.properties/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
    sed -i "'"s/alfresco.encryption.ssl.keystore.type=.*/alfresco.encryption.ssl.keystore.type=${KEYSTORE_TYPE}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
    sed -i "'"s/alfresco.encryption.ssl.truststore.location=.*/alfresco.encryption.ssl.truststore.location=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl.repo.client.truststore/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
    sed -i "'"s/alfresco.encryption.ssl.truststore.passwordFileLocation=.*/alfresco.encryption.ssl.truststore.passwordFileLocation=\\\/opt\\\/alfresco-search-services\\\/keystore\\\/ssl-truststore-passwords.properties/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties && \
    sed -i "'"s/alfresco.encryption.ssl.truststore.type=.*/alfresco.encryption.ssl.truststore.type=${TRUSTSTORE_TYPE}/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties' \
    ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi

# REPLICATION
ARG ENABLE_MASTER
ARG ENABLE_SLAVE
ARG MASTER_HOST
ENV ENABLE_MASTER $ENABLE_MASTER
ENV ENABLE_SLAVE $ENABLE_SLAVE
ENV MASTER_HOST $MASTER_HOST

# Set Master / Slave configuration for this Node
RUN if [ "$ENABLE_MASTER" == "true" ] ; then \
        sed -i '/^bash.*/i echo "\nenable.master=${ENABLE_MASTER}\nenable.slave=${ENABLE_SLAVE}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
        ${DIST_DIR}/solr/bin/search_config_setup.sh; \
        sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
            <lst name=\"master\">\
            <str name=\"replicateAfter\">commit</str>\
            <str name=\"replicateAfter\">startup</str>\
            <str name=\"confFiles\">schema.xml,stopwords.txt</str>\
            </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
    fi
RUN if [ "$ENABLE_SLAVE" == "true" ] ; then \
        sed -i '/^bash.*/i echo "\nenable.master=${ENABLE_MASTER}\nenable.slave=${ENABLE_SLAVE}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
        ${DIST_DIR}/solr/bin/search_config_setup.sh; \
        sed -i "/^bash.*/i sed -i '/^\\\\\s*<requestHandler name=\"\\\\/replication\".*/a \
            <lst name=\"slave\">\
            <str name=\"masterUrl\">http://${MASTER_HOST}:8983/solr/alfresco</str>\
            <str name=\"pollInterval\">00:00:60</str>\
            </lst>' ${DIST_DIR}/solrhome/templates/rerank/conf/solrconfig.xml\n" ${DIST_DIR}/solr/bin/search_config_setup.sh; \
    fi

# SHARDING
ARG ENABLE_SHARDING
ARG NUM_SHARDS
ARG SHARD_ID
ENV ENABLE_SHARDING $ENABLE_SHARDING
ENV NUM_SHARDS $NUM_SHARDS
ENV SHARD_ID $SHARD_ID

# Set Port Number and Sharding ID for this Shard Service
RUN if [ "$ENABLE_SHARDING" == "true" ] ; then \
  sed -i '/^bash.*/i echo "\nsolr.port.ssl=8983\nshard.count=${NUM_SHARDS}\nshard.instance=${SHARD_ID}" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
  ${DIST_DIR}/solr/bin/search_config_setup.sh && \
  sed -i '/^bash.*/i echo "\nalfresco.port=8080\nalfresco.port.ssl=8443\nalfresco.baseUrl=/alfresco" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
  ${DIST_DIR}/solr/bin/search_config_setup.sh; \
fi

# EXPLICIT SHARDING
RUN sed -i '/^bash.*/i sed -i "'"s/shard.method=DB_ID/shard.method=EXPLICIT_ID_FALLBACK_DBID/g"'" ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
${DIST_DIR}/solr/bin/search_config_setup.sh

RUN sed -i '/^bash.*/i echo "\nshard.key=shard:shardId\n" >> ${DIST_DIR}/solrhome/templates/rerank/conf/solrcore.properties\n' \
${DIST_DIR}/solr/bin/search_config_setup.sh


# Useless for 'none'/'http' communications with Alfresco
RUN mkdir ${DIST_DIR}/keystore \
    && chown -R solr:solr ${DIST_DIR}/keystore

VOLUME ["${DIST_DIR}/keystore"]
