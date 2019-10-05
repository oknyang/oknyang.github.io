---
title: 카우치베이스 인덱스 정리
date: 2018-04-04
---

### 1. 인덱스 생성은 특정 노드에만 저장해야 하는가? 
- 동일한 인덱스를 여러노드에 생성가능함. 
- 이름이 서로 달라야함. 
- 힌트를 줘서 명확히 해주는것이 좋을 듯함. 

### 2. 인덱스 키가 리스트에 있는 값이라면 리스트갯수만큼 들어가나? 
- distinct로 중복제거하여 index 가능함 

### 3. _type 속성도 primary index처럼 기본생성해야함? 
- 인덱스 생성시에 where 절에 type이 들어가는 경우 생성해야하고 
- 인덱스 where절에 type 들어가면 쿼리시에도 항상 들어가야함, type 도 같이 index로 잡히는거 같음 

### 4. 인덱스 데이터가 어떻게 들어가는지 데이터를 볼수 있나? 
- 모르겠음 

### 5. 주의사항 및 기타 
- 카우치베이스 5.0부터는 join 성능도 좋아짐 
- 서비스 중인 index는 마음대로 지우거나 삭제하 장애 유발됨 
- db 처럼 explain 가능함.

### 6. array index 구문 풀이
- 아래에서 `mappedViewCategories` 1depth 속성이고 해당 속성은 리스트여서 index를 잡을때 array로접근하여 해당 속성을 distinct한다는의미임
```
ON `tpin`((distinct (array (`vc`.`categoryNo`) for `vc` in `mappedViewCategories` end)),`mappedViewCategories`) 
```

### 7. 인덱싱 배열 사이즈
- array index의 경우 하나의 문서에서 인덱싱되는 배열의 사이즈가 10k를 넘으면 안됨. 그래서 array index를 생성할 때 distinct로 중복을 걸러 주는듯.  
- 10k를 초과할 경우 "Encoded array key is too long” 라는 오류가 indexer.log 파일에 기록됨.

### 8. 인덱스 서비스 노드
- 인덱스 서비스 노드는 하나만 설정가능. 해당 노드가 failout 되면 쿼리 자체가 불가능한 문제가 발생함.

### 9. failout
- failout으로 인해 노드가 빠졌다가 다시 추가되면 해당 노드에서 서비스되던 인덱스들은 전부 초기화됨. (재생성, 빌드해야함)
- 애플리케이션 서버는 failout된 노드를 알지 못해 주기적으로 failout된 노드에 접근을 시도하고, 이로 인해 애플리케이션 서버 재시작을 하지 않는한 계속 에러가 발생함.
