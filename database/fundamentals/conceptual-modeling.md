[개념적 데이터 모델링](http://www.kocw.or.kr/home/cview.do?mty=p&kemId=1163794) 강의를 보고 정리합니다.

## 목차

1. [데이터베이스 설계 단계](#데이터베이스-설계-단계)
2. [요구 사항 분석](#요구-사항-분석)
3. [개념적 설계](#개념적-설계)
   - [개체와 속성 추출](#개념적-설계---개체와-속성-추출)
   - [관계 추출](#개념적-설계---관계-추출)
   - [E-R 다이어그램 작성](#개념적-설계---e-r-다이어그램-작성)

## 데이터베이스 설계 단계

<img width="200" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/2384368f-2e78-4364-a75d-58fcbdd4eabc">

$\rightarrow$ E-R 모델과 릴레이션 변환 규칙을 이용한 설계의 과정

설계 과정 중 오류 발견 시 이전 단계로 되돌아가 설계 내용을 변경할 수 있다.

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/24f1c409-8b12-433a-bf19-6ed28e250894">

$\rightarrow$ 데이터베이스 설계 과정의 각 단계별 주요 작업과 결과물

## 요구 사항 분석

### 목적

- 사용자의 요구 사항을 수집하고 분석하여 개발할 데이터베이스의 용도를 파악
- 업무에 필요한 데이터가 무엇인지, 그 데이터에 어떤 처리가 필요한지 등을 고려

### 결과물

- 요구 사항 명세서

### 주요 작업

- 데이터베이스를 실제로 사용할 주요 사용자의 범위를 결정
- 사용자가 조직에서 수행하는 업무를 분석
- 면담, 설문 조사, 업무 관련 문서 분석 등의 방법을 이용해 요구 사항 수집
- 수집된 요구 사항에 대한 분석 결과를 요구 사항 명세서로 작성

### 요구 사항 분석 예

- 인터넷으로 회원들에게 상품을 판매하는 한빛 마트의 데이터베이스 개발

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/5e8cd2d5-7641-4c8d-86c0-4ca56a86b0de">

$\rightarrow$ 한빛 마트의 데이터베이스를 위한 요구 사항 명세서

## 개념적 설계

### 목적

- DBMS에 독립적인 개념적 스키마 설계
- 요구 사항 분석 결과물을 개념적 데이터 모델을 이용해 개념적 구조로 표현 $\rightarrow$ 개념적 모델링

일반적으로 E-R 모델을 많이 이용한다.

### 결과물

- 개념적 스키마 : E-R 다이어그램

### 주요 작업

- 요구 사항 분석 결과를 기반으로 중요한 개체를 추출하고 개체 간의 관계를 결정하여 E-R 다이어그램으로 표현

### 작업 과정

1. 개체 추출, 각 개체의 주요 속성과 키 속성 선별
2. 개체 간의 관계 결정
3. E-R 다이어그램으로 표현

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/32e5e097-a6e1-4d9d-967a-b6aa9377329a">

1, 2번 과정에서 도출한 개체, 속성, 관계(주요 핵심 요소들)를 E-R 다이어그램으로 표현한다.

## 개념적 설계 - 개체와 속성 추출

### 개체

$\rightarrow$ 저장할만한 가치가 있는 중요 데이터를 가진 사람이나 사물 등

- 예) 병원 데이터베이스 개발에 필요한 개체
  - 병원 운영에 필요한 사람 : 환자, 의사, 간호사 등
  - 병원 운영에 필요한 사물 : 병실, 수술실, 의료 장비 등

### 개체 추출 방법

$\rightarrow$ 요구 사항 문장에서 업무와 관련이 깊은 의미 있는 명사를 찾는다. 찾아낸 명사를 개체와 속성으로 분류한다.

- 업무와 관련이 적은 일반적이고 광범위한 의미의 명사는 제외 (예: 한빛 마트)
- 의미가 같은 명사가 여러개일 경우 대표 명사 하나만 선택 (예: 회원, 고객)

모든 명사가 모두 개체가 되는 것이 아니므로 무엇이 개체인지, 무엇이 속성인지 분류해야 한다.

### 개체와 속성 추출 예시 1

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/3220adb9-0d18-4abd-8f55-74bd0ab1506c">

$\rightarrow$ 요구 사항 문장에서 명사를 선별한 예

- "한빛 마트"는 일반적이고 광범위한 의미의 명사이므로 제외
- "회원 아이디", "비밀번호", "이름", "나이", "직업", "등급", "적립금"은 회원의 속성으로 분류("회원"을 표현하는 속성들)
- "회원 아이디"는 키 속성으로 분류, "회원 아이디"를 가지고 "회원"을 식별

따라서 추출 결과는 아래와 같다.

- 개체 : 회원
- "회원" 개체의 속성 : 회원 아이디, 비밀번호, 이름, 나이, 직업, 등급, 적립금
- "회원" 개체의 키 속성 : 회원 아이디

### 개체와 속성 추출 예시 2

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/86c956c9-06b8-427f-9bad-e1ea4c561559">

- 개체 : 회원, 상품
- 속성 : 주문번호, 주문수량, 배송지, 주문일자

  $\rightarrow$ 회원이 상품을 주문해야 생기는 중요한 정보이기 때문에 회원이나 상품 개체의 속성으로 보기는 어렵다.  
  이후 추출할 특정 관계(주문)의 속성일 가능성이 높다.

### 개체와 속성 추출 결과

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d416dc7a-bbff-448b-b481-c2a758878b62">

$\rightarrow$ 한빛 마트 요구 사항 명세서에서의 명사를 선별한 결과

### 개체와 속성을 추출한 최종 결과

<img width="400" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/68ac160c-ac01-4e9d-8330-e4c63b34725c">

### 개체들의 E-R 다이어그램

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/d2d0adab-db59-4815-8d7c-58416f8f8782">

$\rightarrow$ 회원, 상품 개체의 E-R 다이어그램

<img width="300" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/749e811f-d4bf-42f6-849e-8c459b508da0">

$\rightarrow$ 제조업체, 게시글 개체의 E-R 다이어그램

## 개념적 설계 - 관계 추출

### 관계

$\rightarrow$ 개체 간의 의미 있는 연관성

### 관계 추출 방법

$\rightarrow$ 요구 사항 문장에서 개체 간의 연관성을 의미 있게 표현한 동사를 찾는다. 찾아낸 관계에 대해 매핑 카디널리티와 참여 특성을 결정한다.

- 의미가 같은 동사가 여러개일 경우 대표 명사 하나만 선택
- 매핑 카디널리티 : 일대일(1:1), 일대다(1:n), 다대다(n:m)
- 참여 특성 : 필수적 참여 / 선택적 참여

### 관계 추출 예시 1

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/c1e9bc1f-c851-4be4-b78e-b44aff46a98c">

$\rightarrow$ 요구 사항 문장에서 동사를 선별한 예

- "입력해야 한다"는 개체와 개체의 관계를 표현하는 동사로 볼 수 없으므로 제외
- "부여된다"는 개체와 개체의 관계를 표현하는 동사로 볼 수 없으므로 제외
- "식별한다"는 개체와 개체의 관계를 표현하는 동사로 볼 수 없으므로 제외

### 관계 추출 예시 2

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8069d497-a2ad-4f42-a9b2-549cb374a64b">

$\rightarrow$ 요구 사항 문장에서 동사를 선별한 예

- 관계 : 주문
  - "회원" 개체와 "상품" 개체가 맺는 관계, 다대다(n:m) 관계
  - "회원" 개체는 관계에 선택적으로 참여
  - "상품" 개체는 관계에 선택적으로 참여
- "주문" 관계의 속성 : 주문번호, 주문수량, 배송지, 주문일자

회원은 상품을 여러개 주문할 수 있다.  
상품은 여러명의 회원에게 주문될 수 있다.  
$\rightarrow$ 다대다(n:m) 관계

상품을 한번도 주문하지 않아도 회원 개체가 인스턴스로서 존재할 수 있다. $\rightarrow$ 선택적 참여  
상품이 하나도 주문되지 않아도 상품 개체가 인스턴스로서 존재할 수 있다. $\rightarrow$ 선택적 참여

### 관계 추출 예시 3

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/efccb6ce-f83b-46aa-a0bc-ebce8138c11d">

$\rightarrow$ 요구 사항 문장에서 동사를 선별한 예

- 관계 : 공급
  - "제조업체" 개체와 "상품" 개체가 맺는 관계, 일대다(1:n) 관계
  - "상품" 개체는 관계에 필수적으로 참여
  - "제조업체" 개체는 관계에 선택적으로 참여
- "공급" 관계의 속성 : 공급일자, 공급량

제조업체는 상품을 여러개 공급할 수 있다.  
상품은 하나의 제조업체에 의해 공급된다.  
$\rightarrow$ 일대다(1:n) 관계

제조업체 없이는 상품 개체가 존재할 수 없다. $\rightarrow$ 필수적 참여  
상품을 하나도 공급하지 않아도 제조업체 개체가 인스턴스로서 존재할 수 있다. $\rightarrow$ 선택적 참여

### 관계 추출 예시 4

<img width="500" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/df10ee2d-7916-4e25-9dcf-c4c4c817521d">

$\rightarrow$ 요구 사항 문장에서 동사를 선별한 예

- 관계 : 작성
  - "회원" 개체와 "게시글" 개체가 맺는 관계, 일대다(1:n) 관계
  - "회원" 개체는 관계에 선택적으로 참여
  - "게시글" 개체는 관계에 필수적으로 참여

회원은 게시글을 여러개 작성할 수 있다.  
게시글은 한명의 회원에 의해서만 작성된다.  
$\rightarrow$ 일대다(1:n) 관계

게시글을 작성하지 않아도 회원 개체가 인스턴스로서 존재할 수 있다. $\rightarrow$ 선택적 참여  
게시글은 회원이 작성해야만 존재할 수 있다. $\rightarrow$ 필수적 참여

### 관계를 추출한 최종 결과

<img width="560" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/0bb2a02a-a434-479a-9d9c-359f90b6c2ca">

### 개체, 관계 E-R 다이어그램

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/af8e6e87-efbf-4d64-bba1-68ea89ac18c7">

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/8ed6e3a3-550b-4ab0-b5bc-82f663df7be7">

<img width="600" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/78c542a4-a2c0-4b88-9058-b1442b6566a5">

## 개념적 설계 - E-R 다이어그램 작성

<img width="700" height="auto" src="https://github.com/usuyn/TIL/assets/68963707/f40f74c7-f6c0-4da7-9632-6ab48622a753">

$\rightarrow$ 요구 사항 명세서를 개념적 스키마로 작성한 결과
