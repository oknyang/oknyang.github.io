---
title: spring-data-couchbase java config
date: 2017-02-21
categories: spring-data-couchbase
---

multi bucket , base repository class  변경. 
```java
package com.tmoncorp.batch.order.couchbase.config;

import com.couchbase.client.java.Bucket;
import com.tmoncorp.api.order.biz.cash.model.ReceiptHistory;
import com.tmoncorp.api.order.biz.order.model.OrderHistory;
import com.tmoncorp.batch.order.couchbase.repository.CustomRepositoryImpl;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.data.couchbase.config.AbstractCouchbaseConfiguration;
import org.springframework.data.couchbase.core.CouchbaseTemplate;
import org.springframework.data.couchbase.repository.config.EnableCouchbaseRepositories;
import org.springframework.data.couchbase.repository.config.RepositoryOperationsMapping;

import java.util.Arrays;
import java.util.List;

@Configuration
@EnableCouchbaseRepositories(basePackages = "com.tmoncorp.batch.order.couchbase.repository", repositoryBaseClass = CustomRepositoryImpl.class)
@PropertySource("classpath:applicationProperty.properties")
public class CouchbaseConfig extends AbstractCouchbaseConfiguration {
   private static final Logger LOGGER = LoggerFactory.getLogger(CouchbaseConfig.class);

   @Value("${couchbase.nodes}")
   private String nodes;

   @Value("${couchbase.bucketname}")
   private String orderBucketName;

   @Value("${couchbase.password}")
   private String orderBucketPassword;

   @Value("${couchbase.history.bucketname}")
   private String orderHisBucketName;

   @Value("${couchbase.history.password}")
   private String orderHisBucketPassword;

   @Override
   protected List<String> getBootstrapHosts() {
      String[] nodeArray = StringUtils.split(nodes, ",");

      return Arrays.asList(nodeArray);
   }

   @Override
   protected String getBucketName() {
      return orderBucketName;
   }

   @Override
   protected String getBucketPassword() {
      return orderBucketPassword;
   }

   @Bean
   public Bucket orderHisBucket() throws Exception {
      return couchbaseCluster().openBucket(orderHisBucketName, orderHisBucketPassword);
   }

   @Bean
   public CouchbaseTemplate orderHisCouchbaseTemplate() throws Exception {
      CouchbaseTemplate template = new CouchbaseTemplate(
            couchbaseClusterInfo(),
            orderHisBucket(),
            mappingCouchbaseConverter(),
            translationService()
      );
      template.setDefaultConsistency(getDefaultConsistency());
      return template;
   }

   @Override
   public void configureRepositoryOperationsMapping(RepositoryOperationsMapping baseMapping) {
      try {
         baseMapping.mapEntity(OrderHistory.class, orderHisCouchbaseTemplate());
         baseMapping.mapEntity(ReceiptHistory.class, orderHisCouchbaseTemplate());
      } catch (Exception e) {
         LOGGER.error("Couchbase Repository operation mapping error.", e);
      }
   }
}
```
