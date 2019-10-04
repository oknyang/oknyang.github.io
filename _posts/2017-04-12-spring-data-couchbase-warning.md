---
title: Couchbase.. More than 1 Couchbase Environments found 경고
date: 2017-04-12
categories: spring-data-couchbase warning
---

아래와 같이 하나의 클러스터에 environments를 설정하지 않고 여러개의 버켓을 사용하면 서버가 뜨긴 뜨나 경고가 발생한다.

```xml

<couchbase:cluster>
    <couchbase:node>${couchbase.nodes}</couchbase:node>
</couchbase:cluster>

<couchbase:clusterInfo login="${couchbase.bucketname}" password="${couchbase.password}"/>
<couchbase:bucket id="orderCouchbaseClient" bucketName="${couchbase.bucketname}" bucketPassword="${couchbase.password}"/>
<couchbase:bucket id="orderHisCouchbaseClient" bucketName="${couchbase.history.bucketname}" bucketPassword="${couchbase.history.password}"/>

<couchbase:template id="orderCouchbaseTemplate" bucket-ref="orderCouchbaseClient"/>

<couchbase:template id="orderHisCouchbaseTemplate" bucket-ref="orderHisCouchbaseClient"/>

<couchbase:env/>

<couchbase:bucket id="orderSnapshotCouchbaseClient" bucketName="${couchbase.snapshot.bucketname}" bucketPassword="${couchbase.snapshot.password}" />
<couchbase:template id="orderSnapshotCouchbaseTemplate" bucket-ref="orderSnapshotCouchbaseClient"/>


<bean class="com.tmoncorp.api.order.config.CouchbaseTemplateBeanPostProcessor" />

```

경고 : 
```
[2017-04-12 17:54:57] [main] [WARN ] c.c.c.c.l.Slf4JLogger.warn[151] More than 1 Couchbase Environments found (3), this can have severe impact on performance and stability. Reuse environments!
```

Couchbase Environment가 1개 이상 발견되었고, 이는 서버에 큰 영향을 줄 수 있다는..  
이를 해결하기 위해서 아래와 같이 default environment 빈을 생성해서 공유함..

```xml
<bean id="couchbaseEnvironment" class="com.couchbase.client.java.env.DefaultCouchbaseEnvironment" factory-method="create"/>

<couchbase:cluster id="couchbaseCluster" env-ref="couchbaseEnvironment">
    <couchbase:node>${couchbase.nodes}</couchbase:node>
</couchbase:cluster>

<couchbase:clusterInfo cluster-ref="couchbaseCluster" login="${couchbase.bucketname}" password="${couchbase.password}"/>

<couchbase:bucket id="orderCouchbaseClient" cluster-ref="couchbaseCluster" bucketName="${couchbase.bucketname}" bucketPassword="${couchbase.password}"/>
<couchbase:bucket id="orderHisCouchbaseClient" cluster-ref="couchbaseCluster" bucketName="${couchbase.history.bucketname}" bucketPassword="${couchbase.history.password}"/>
<couchbase:bucket id="orderSnapshotCouchbaseClient" cluster-ref="couchbaseCluster" bucketName="${couchbase.snapshot.bucketname}" bucketPassword="${couchbase.snapshot.password}" />

<couchbase:template id="orderCouchbaseTemplate" bucket-ref="orderCouchbaseClient"/>

<couchbase:template id="orderHisCouchbaseTemplate" bucket-ref="orderHisCouchbaseClient"/>

<couchbase:template id="orderSnapshotCouchbaseTemplate" bucket-ref="orderSnapshotCouchbaseClient"/>


<bean class="com.tmoncorp.api.order.config.CouchbaseTemplateBeanPostProcessor" />
```
