# applications/

이 디렉토리는 의도적으로 비어있다.

마이크로서비스 5개의 Helm 차트는 **`msa-spring-boot` 레포의
`charts/services/{user-api-gateway,product-service,inventory-service,order-service,notification-service}/`**
에 거주한다. 코드와 차트가 같은 레포에 있어야 PR 단위로 함께 변경/리뷰되기 때문이다.

`bootstrap/apps-appset.yaml`의 ApplicationSet (Git Generator)이 그 디렉토리를 스캔해서
서비스가 추가될 때마다 Application을 자동 생성한다. 따라서 본 디렉토리에는
**아무 Application도 만들지 않는다**.
