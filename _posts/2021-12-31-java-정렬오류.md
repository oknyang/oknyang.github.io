---
title: Comparison method violates its general contract
date: 2021-12-31
---

## 장애 인입
장바구니 페이지가 뜨지 않는다는 장애가 인입됨.

## 원인 파악
장바구니에 담긴 상품을 조회하는 api를 호출했을때 아래와 같은 오류 발생.  
```
[ERROR] c.t.c.c.ApiExceptionController.handleException[118] handleException error :: uri :: /api/cart/xx :: msg -> Comparison method violates its general contract! ,
class -> class java.lang.IllegalArgumentException ,
className -> java.util.TimSort ,
methodName -> mergeHi ,
line -> 899 ,
file -> TimSort.java
```

해당 오류는 java 정렬시 비교 값의 우선순위가 명확하게 정의되지 않아 객체간의 우선순위가 제대로 정해지지 않아 발생한다고 한다.  
참고 : https://118k.tistory.com/291  

## 장애 발생부분 확인
장바구니 조회 api에서 아이템들간 상품번호 및 상품의 상태를 비교하여 정렬하는 부분이 존재.  

```
@Override
public int compare(CartDeal d1, CartDeal d2) {
    boolean d1Status = isValidStatus(d1.getPurchaseInfo());
    boolean d2Status = isValidStatus(d2.getPurchaseInfo());
    if (d1.getMainSrl().equals(d2.getMainSrl()) == false) {
        return 0; // 메인 상품 번호가 다를 경우 정렬 안함
    } else if ((d1Status == true && d2Status == true) || (d1Status == false && d2Status == false)) {
        // d1, d2  상태 같을 경우 옵션 번호가 큰 딜은 뒤로
        if (d1.getSrl() > d2.getSrl()) {
            return 1;
        }
        return -1;
    } else if (d1Status == false && d2Status == true) {
        return 1; // 상태 이상 상품 뒤로
    }
    return -1; // 나머지 앞으로
}
```

## 처리
위 코드에서 어떤 조건이 빠졌는지 파악하기 위해 로그 추가 후 테스트.  

### AS-IS
```
        if (d1.getSrl() > d2.getSrl()) {
            return 1;
        }
```

### TO-BE
```
        if (d1.getSrl() >= d2.getSrl()) {
            return 1;
        }
```

