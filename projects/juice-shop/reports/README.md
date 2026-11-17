# Final Security Assessment Report

## 1. Executive Summary
 - 본 프로젝트는 OWASP Juice Shop을 대상으로 웹 애플리케이션의 보안 구조 분석 및 주요 취약점 검증을 위해 수행한 개인 실습 프로젝트
 - 정찰 및 HTTP 요청·응답 분석을 시작으로 인증 구조, 접근 제어, 입력값 처리 영역 중심으로 보안 테스트 진행
 - 분석 과정에서 사용자 간 장바구니 데이터에 대한 비인가 접근(Broken Object Level Authorization)과 로그인 API의 SQL Injection 취약점 확인
 - SQL Injection은 SQL 쿼리의 파라미터 바인딩을 적용하여 개선, 수정 후 재검증 수행, BOLA는 취약점 재현과 영향 확인 완료, 개선 방안 도출
 - 본 프로젝트는 취약점 발견에 그치지 않고 원인 분석, 영향 검증, 개선 및 재검증 과정을 경험하는 것을 목적으로 진행

## 2. Assessment Scope

### 2.1 Target Environment
 - Target Application: OWASP Juice Shop
 - Target System: Ubuntu VM
 - Attacker System: Kali Linux VM
 - Virtualization: Oracle VirtualBox
 - Web Browser: Firefox
 - Proxy Tool: Burp Suite Community Edition
 - Target URL: http://10.10.10.4:3000/

### 2.2 Assessment Areas
 - Authentication: 사용자 인증 및 세션 처리
 - Access Control: 사용자별 객체 접근 권한
 - Input Validation: 사용자 입력값의 SQL 쿼리 처리

### 2.3 Testing Scope
 - 테스트는 개인 가상 환경에 구축한 OWASP Juice Shop을 대상으로 수행
 - 외부 운영 시스템이나 제3자의 서비스는 테스트 범위에 미포함

## 3. Assessment Methodology
 - 본 프로젝트 순서 진행

### Phase 1. Environment Setup
 - Nmap을 이용한 대상 시스템 포트 스캔 및 서비스 확인
 - VirtualBox 기반으로 Kali Linux와 Ubuntu 가상 환경 구성, Ubuntu에서 OWASP Juice Shop 실행
 - Firefox 및 Burp Suite를 활용한 웹 애플리케이션 접근 및 HTTP 요청,응답 분석

### Phase 2. Reconnaissance
 - 대상 IP와 서비스 접근 가능 여부 확인, Firefox 및 Burp Suite를 활용하여 웹 애플리케이션의 HTTP 요청과 응답 분석

### Phase 3. Vulnerability Analysis
 - 인증, 접근 제어, 입력값 처리 영역 중심으로 취약점 가능성 분석 및 테스트 수행

### Phase 4. Exploitation & Verification
 - 허가된 실습 환경에서 취약점 재현 여부와 실제 데이터 접근 영향 확인

### Phase 5. Remediation & Retesting
 - SQL Injection에 대한 개선 적용, 동일한 조건 재검증, BOLA는 개선 방안 정리

## 4. Security Findings

### 4.1 Authentication Analysis
 - Analysis Target : GET /rest/user/whoami?fields=email
 - Cookie 및 Authorization 헤더에 따른 응답 차이 확인
 - 인증 정보에 따른 사용자 정보 반환 동작 분석

**Assessment Result**
 - 인증 처리 구조에 대한 분석 수행, 현재 확보한 테스트 결과만으로 인증 우회 취약점 미확정

### 4.2 Broken Object Level Authorization (BOLA)
**Affected Endpoint**
 - GET /rest/basket/{basketId}

**Test Result**
 - User A ID: 25
 - User A Basket ID: 7
 - User B ID: 26
 - User B Basket ID: 6

 - User B의 인증 상태에서 User A의 Basket ID인 7 요청
 - 응답에서 HTTP 200과 함께 User A의 장바구니 데이터 및 소유자 정보 반환 확인

**Security Impact**
 - 사용자 간 데이터 접근 권한이 적절하게 검증되지 않을 경우 다른 사용자의 장바구니 정보 노출의 가능성

**Assessment Result**
 - 사용자 간 객체 접근 제어 문제를 재현 및 데이터 접근 영향 확인

### 4.3 SQL Injection
**Affected Endpoint**
 - POST /rest/user/login

**Test Result**
 - 로그인 요청의 이메일 입력값에 SQL Injection 테스트 입력값 적용
 - 기존 코드에서는 문자열 결합 방식으로 SQL 쿼리를 구성, 잘못된 비밀번호를 사용한 테스트에서도 인증 토큰 발급되는 현상 확인

**Security Impact**
 - 인증 검증을 우회하여 비인가 접근이 발생할 수 있는 위험 확인

**Assessment Result**
 - SQL Injection 재현에 성공, 이후 SQL 쿼리의 파라미터 바인딩 적용 및 개선

## 5. Remediation & Retesting

### 5.1 SQL Injection
**Applied Remediation**
 - Sequelize replacements를 사용하여 SQL 쿼리와 사용자 입력값 분리

**Retesting Results**
 - 정상 로그인 : HTTP 200
 - 잘못된 비밀번호 : HTTP 401
 - SQL Injection 테스트 입력값 : HTTP 401
 - 기존에 재현되었던 SQL Injection 테스트 입력값으로 인증이 진행되지 않는 것 확인

**Status: Remediated and Retested**

### 5.2 BOLA
**Recommended Remediation**
 - 서버 측 사용자 소유권 검증
 - 객체 조회 시 사용자 식별 정보 적용
 - 권한이 없는 객체에 대한 접근 차단
 - 실제 소스 코드 수정과 재검증 미수행

**Status: Remediation Recommended**

## 6. Assessment Limitations
 - 본 프로젝트는 개인 가상 환경에서 OWASP Juice Shop을 대상으로 수행한 학습 및 보안 검증 프로젝트

 - 전체 애플리케이션의 모든 기능과 API를 분석한 것은 아님
 - 인증 영역에서 인증 우회 취약점을 확정하지 않음
 - BOLA는 취약점 재현까지 수행, 실제 수정 및 재검증 미진행
 - SQL Injection은 확인한 테스트 조건에서 개선 결과 검증, 모든 입력값과 공격 유형에 대한 완전한 안전성 미입증

## 7. Lessons Learned
 - 이번 프로젝트를 통해 웹 애플리케이션 보안 분석은 단순히 공격을 시도하는 과정이 아니라, 대상 시스템의 구조를 이해, 요청과 응답을 분석, 취약점의 원인과 영향을 검증하는 과정이라는 점을 경험

 - 아래 보안 분석 원칙 및 기술적 경험
 - HTTP 상태 코드만으로 인증 성공 여부나 보안성을 판단할 수 없음
 - 인증과 객체 접근 권한 검증은 서로 다른 보안 요소임
 - 사용자 입력값을 SQL 쿼리와 분리하는 것이 중요함
 - 소스 코드 수정 이후 실제 실행 환경에서 재검증하는 과정이 필요함
 - 분석 결과와 증거를 명확하게 기록하는 것이 중요함

## 8. Conclusion
 - 본 프로젝트에서는 OWASP Juice Shop을 대상으로 정찰, 웹 요청 분석, 인증 구조 분석, 접근 제어 검증 및 SQL Injection 테스트 수행
 - BOLA와 SQL Injection 취약점 재현, SQL Injection에 대해서는 개선 및 재검증 진행
 - 이번 프로젝트는 Red Team Specialist를 목표로 취약점 분석과 침투 테스트 역량을 함께 발전시키기 위한 첫 번째 주요 실습 프로젝트
 - 앞으로도 허가된 환경에서 취약점 분석, 영향 검증 및 보안 개선 과정 반복, 취약점 분석과 침투 테스트 역량을 지속적으로 발전시킬 계획