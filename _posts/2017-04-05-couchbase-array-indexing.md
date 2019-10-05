---
title: n1ql array indexing
date: 2017-04-05
---

document 구조가 document 안에 여러개의 list 가 있는 형태라서,  
여태까지 써오던 view 기반에서는 array 안의 값들을 key로 사용함..  
아래와 같이.. (하나하나 따로 뷰를 만들어도 되나.. 그렇게 안씀.. -_-)  

```javascript
function (doc, meta) {
  if(doc._class && doc._class == "com.pkg.api.order.biz.order.model.Order") {
    for(var i=0; i<doc.orderDetailList.length; i++) {
      emit(["TICKET", doc.orderDetailList[i].statusType, doc.orderDetailList[i].saveDt], null);
    }

    for(var i=0; i<doc.orderDeliveryFeeDetailList.length; i++) {
      emit(["DELIVERY", doc.orderDeliveryFeeDetailList[i].status, doc.orderDeliveryFeeDetailList[i].saveDt], null);
    }

    if(doc.customDetailList != null){
      for(var i=0; i<doc.customDetailList.length; i++) {
        emit(["CUSTOMS", doc.customDetailList[i].statusType, doc.customDetailList[i].saveDt], null);
      }
    }

    if(doc.internationalDeliveryFeeDetailList != null){
      for(var i=0; i<doc.internationalDeliveryFeeDetailList.length; i++) {
        emit(["INTERNATIONAL_DELIVERY", doc.internationalDeliveryFeeDetailList[i].status, doc.internationalDeliveryFeeDetailList[i].saveDt], null);
      }
    }
  }
}
```



이걸 유류할증료부터 n1ql을 사용하도록 변경하려면 인덱스를 만들어 줘야 하는데, 이때 array index를 사용해야 한다고 함.  
https://developer.couchbase.com/documentation/server/4.6/n1ql/n1ql-language-reference/indexing-arrays.html  

그래서 아래와 같이 인덱스를 생성해봄  
```
create index ix_fuelSurcharge_statusType on `order` 
(distinct array fs.statusType for fs in fuelSurchargeList end)
where _type = 'ORDER'
```
: fuelSurchargeList 의 statusType 을 인덱스 키로 쓴다는..  
잘 생성됨.. 이제 saveDt도 같이 인덱스 키로 사용하고자 아래와 같이 인덱스를 생성..  

```
create index ix_fuelSurcharge_statusType_saveDt on `order` 
(distinct array fs.statusType for fs in fuelSurchargeList end, distinct array fs.saveDt for fs in fuelSurchargeList end)
where _type = 'ORDER'
```
```
[
  {
    "code": 5000,
    "msg": "GSI CreateIndex() - cause: Multiple expressions with ALL are found. Only one array expression is supported per index.",
    "query_from_user": "create index ix_fuelSurcharge_statusType_saveDt on `order` \n(distinct array fs.statusType for fs in fuelSurchargeList end, distinct array fs.saveDt for fs in fuelSurchargeList end)\nwhere _type = 'ORDER'"
  }
]
```


에러남.. 대충 보면 array 표현식이 하나만 가능한데 여러개를 썼다는 말..  
https://forums.couchbase.com/t/index-multiple-arrays/12059 요 글을 보면 하나의 인덱스에 2개의 array 표현식은 쓸 수 없고,  
대신 index 2개를 각각 생성해서 사용하면 교차스캔 된다는.. (IntersectScan)  

그래서 아래와 같이 fuelSurcharge의 saveDt로 인덱스를 하나 더 만들어줌..  

```
create index ix_fuelSurcharge_saveDt on `order` 
(distinct array fs.saveDt for fs in fuelSurchargeList end)
where _type = 'ORDER'
```
drop index : 
```
drop index `order`.`ix_fuelSurcharge_statusType_saveDt`
```
index 조회 : 
```
SELECT * FROM system:indexes
```
defer_build:true 옵션 - 지연 build.. 인덱스를 생성할때 이 옵션을 주면 인덱스가 바로 빌드되지 않고, build index 문을 실행해야 빌드된다.  
이 옵션의 장점은.. 생성을 일단 한번에 해놓고 build index 문을 실행하면 생성된 인덱스들을 한번에 다 빌드 할 수 있다는..  


위에 생성한 2개의 인덱스를 이용해 조회  
```sql
SELECT * FROM `order` 
WHERE _type = 'ORDER' 
AND ANY fs IN fuelSurchargeList SATISFIES fs.statusType = 'AV' END
AND ANY fs IN fuelSurchargeList SATISFIES fs.saveDt BETWEEN STR_TO_MILLIS('2017-04-03 00:25:56') AND STR_TO_MILLIS('2017-04-03 19:47:34') END
;
```


STR_TO_MILLIS : 스트링을 밀리세컨으로 바꾸는거??  
https://developer.couchbase.com/documentation/server/4.6/n1ql/n1ql-language-reference/datefun.html 참고  


explain  
```
EXPLAIN
SELECT * FROM `order` 
WHERE _type = 'ORDER' 
AND ANY fs IN fuelSurchargeList SATISFIES fs.statusType = 'AV' END
AND ANY fs IN fuelSurchargeList SATISFIES fs.saveDt BETWEEN STR_TO_MILLIS('2017-04-03 00:25:56') AND STR_TO_MILLIS('2017-04-03 19:47:34') END
;


플랜 결과 : 
[
  {
    "plan": {
      "#operator": "Sequence",
      "~children": [
        {
          "#operator": "IntersectScan",
          "scans": [
            {
              "#operator": "DistinctScan",
              "scan": {
                "#operator": "IndexScan",
                "index": "ix_fuelSurcharge_statusType",
                "index_id": "506859031ff25520",
                "keyspace": "order",
                "namespace": "default",
                "spans": [
                  {
                    "Range": {
                      "High": [
                        "\"AV\""
                      ],
                      "Inclusion": 3,
                      "Low": [
                        "\"AV\""
                      ]
                    }
                  }
                ],
                "using": "gsi"
              }
            },
            {
              "#operator": "DistinctScan",
              "scan": {
                "#operator": "IndexScan",
                "index": "ix_fuelSurcharge_saveDt",
                "index_id": "c94234752f25ca0c",
                "keyspace": "order",
                "namespace": "default",
                "spans": [
                  {
                    "Range": {
                      "High": [
                        "str_to_millis(\"2017-04-03 19:47:34\")"
                      ],
                      "Inclusion": 3,
                      "Low": [
                        "str_to_millis(\"2017-04-03 00:25:56\")"
                      ]
                    }
                  }
                ],
                "using": "gsi"
              }
            }
          ]
        },
        {
          "#operator": "Fetch",
          "keyspace": "order",
          "namespace": "default"
        },
        {
          "#operator": "Parallel",
          "~child": {
            "#operator": "Sequence",
            "~children": [
              {
                "#operator": "Filter",
                "condition": "((((`order`.`_type`) = \"ORDER\") and any `fs` in (`order`.`fuelSurchargeList`) satisfies ((`fs`.`statusType`) = \"AV\") end) and any `fs` in (`order`.`fuelSurchargeList`) satisfies ((`fs`.`saveDt`) between str_to_millis(\"2017-04-03 00:25:56\") and str_to_millis(\"2017-04-03 19:47:34\")) end)"
              },
              {
                "#operator": "InitialProject",
                "result_terms": [
                  {
                    "expr": "self",
                    "star": true
                  }
                ]
              },
              {
                "#operator": "FinalProject"
              }
            ]
          }
        }
      ]
    },
    "text": "\nSELECT * FROM `order` \nWHERE _type = 'ORDER' \nAND ANY fs IN fuelSurchargeList SATISFIES fs.statusType = 'AV' END\nAND ANY fs IN fuelSurchargeList SATISFIES fs.saveDt BETWEEN STR_TO_MILLIS('2017-04-03 00:25:56') AND STR_TO_MILLIS('2017-04-03 19:47:34') END\n;"
  }
]

```




