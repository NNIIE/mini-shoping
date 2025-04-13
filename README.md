# 요구 사항
다음 8개의 카테고리에서 상품을 하나씩 구매하여, 코디를 완성하는 서비스입니다.

- 상의, 아우터, 바지, 스니커즈, 가방, 모자, 양말, 액세서리

단, 구매 가격 외의 추가적인 비용(예, 배송비 등)은 고려하지 않고, 브랜드의 카테고리에는 1개의 상품은 존재하고, 구매를 고려하는 모든 쇼핑몰에 상품 품절은 없다고 가정하고 다음 4가지 기능을 구현해야 합니다.
1. 고객은 카테고리 별로 최저가격인 브랜드와 가격을 조회하고 총액이 얼마인지 확인할 수 있어야 합니다.
2. 고객은 단일 브랜드로 전체 카테고리 상품을 구매할 경우 최저가격인 브랜드와 총액이 얼마인지 확인할 수 있어야 합니다.
3. 고객은 특정 카테고리에서 최저가격 브랜드와 최고가격 브랜드를 확인하고 각 브랜드 상품의 가격을 확인할 수 있어야 합니다.
4. 운영자는 새로운 브랜드를 등록하고, 모든 브랜드의 상품을 추가, 변경, 삭제할 수 있어야 합니다.

<br>

# 실행 방법
- 추후 작성

<br>

# 테스트
- Unit Test
- Integration Test

<br>

# 개발 환경
#### Language: Java 17 
- LTS 버전 중 JDK 8 / 17 / 21 사이에서 고민했습니다. JDK 21은 가상 스레드를 포함한 최신 동시성 모델과 향상된 패턴 매칭을 제공하고 JDK 8은 출시된지 오래된 버전으로 새로운 언어 기능과 성능 개선이 부족합니다. JDK 17도 안정성이 검증되었지만 Record Class 등의 JDK 17의 모든 기능을 가지면서도 추가적인 성능 개선과 더 발전된 언어 기능을 제공하는 JDK 21을 선택했습니다.
#### GC: G1 GC
- 이 어플리케이션은 전형적인 웹 어플리케이션의 특성을 가지고 있습니다. 따라서 처리량 중심의 Parallel GC보다는 처리량과 일시 정지 시간 사이의 균형을 잘 제공하며 Full GC 빈도가 낮아 안정적인 웹 어플리케이션의 운영에 적합한 G1 GC를 선택했습니다.
#### Framework: Spring Boot 3.4.4
- JDK 21의 기능들을 지원하는 최신 정식 릴리즈 버전인 3.4.4를 선택했습니다.
#### Database: MySQL 8.0
- 읽기 작업에 최적화된 MySQL을 선택했습니다. MySQL의 스토리지 엔진인 InnoDB에서는 클러스터드 인덱스로 인하여 순차적 읽기, 범위 검색같은 쿼리 패턴에 좋은 성능을 보이며 모든 보조인덱스가 PK를 포함하는 구조이기 때문에 커버링 인덱스 효과를 잘 얻을 수 있습니다. 또한 옵티마이저, 버퍼 풀 등에서 큰 성능향상이 있는 8.0 버전을 선택했습니다.
#### Build Tool: Gradle
- Maven 과 비교하여 Groovy/Kotlin DSL을 통한 선언적 방식으로 더 간결한 표현력 있는 빌드 스크립트 작성이 가능하고 증분 빌드 기능으로 변경된 부분만 재컴파일하여 빌드 시간을 크게 단축시킬 수 있는 Gradle을 선택했습니다

<br>

# 프로젝트 구조
요구사항이 복잡하지 않아 최대한 단순하고 파악하기 쉬운 모놀리식 아키텍처 (Controller-Service-Repository) 로 설계했습니다.
```
product-service/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── shopping/
│   │   │           ├── ShoppingApplication.java
│   │   │           ├── controller/
│   │   │           │   ├── SearchController.java
│   │   │           │   ├── ProductController.java
│   │   │           ├── service/
│   │   │           │   ├── SearchService.java
│   │   │           │   └── ProductService.java
│   │   │           ├── repository/
│   │   │           │   ├── SearchRepository.java
│   │   │           │   ├── SearchRepositoryImpl.java
│   │   │           │   ├── ProductRepository.java
│   │   │           ├── model/
│   │   │           │   └── entity/
│   │   │           │       ├── BaseEntity.java
│   │   │           │       ├── Brand.java
│   │   │           │       └── Product.java
│   │   │           │   └── domain/
│   │   │           │       ├── BestBrandProduct.java
│   │   │           │       ├── BestPriceProduct.java
│   │   │           │       └── CategoryPriceProduct.java
│   │   │           │   └── dto/
│   │   │           │       ├── request/
│   │   │           │       │   ├── ProductCreateRequest.java
│   │   │           │       └── └── ProductUpdateRequest.java
│   │   │           │       ├── response/
│   │   │           │       │   ├── BestBrandResponse.java
│   │   │           │       │   ├── LowestProductsResponse.java
│   │   │           │       └── └── BestPriceResponse.java
│   │   │           ├── exception/
│   │   │           │   ├── GlobalExceptionHandler.java
│   │   │           │   └── GlobalExceptionResponse.java
│   │   │           ├── config/
│   │   │           │   ├── DatabaseConfig.java
│   │   │           │   ├── SwaggerConfig.java
│   │   │           │   └── WebConfig.java
│   │   │           └── util/
│   │   │               └── RegexpConstants.java 
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── data.sql
│   │       └── schema.sql
│   └── test/
│       └── java/
│           └── com/
│               └── shopping/
│                   ├── controller/
│                   │   ├── SearchControllerTest.java
│                   │   └── ProductControllerTest.java
│                   ├── service/
│                   │   ├── SearchServiceTest.java
│                   │   └── ProductServiceTest.java
│                   └── repository/
│                       ├── SearchRepositoryTest.java
│                       └── ProductRepositoryTest.java
└── build.gradle
```

<br>

# API
### 상품
|    METHOD   | URL |  기능                 |
|----------|--------|----------------------|
| GET      | /product | 상품 목록 조회        |
| GET      | /product/{id} | 특정 상품 조회   |
| POST     | /product | 상품 등록            |
| PATCH    | /product/{id} | 상품 수정       |
| DELETE   | /product/{id} | 상품 삭제       |

### 검색
| METHOD       | URL | 기능                  |
|----------|--------|----------------------|
| GET      | /search/categories/lowest-prices | 카테고리 별 최저가 브랜드의 상품가격 및 총액 조회 |
| GET      | /search/brands/best-value | 단일 브랜드로 모든 카테고리 구매시 최저가격 브랜드와 가격, 총액 조회 |
| GET      | /search/category/{category}/price-range | 카테고리 이름으로 최저, 최고 가격 브랜드와 상품 가격 조회 |

