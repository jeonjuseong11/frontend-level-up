# [Day 8] 쿠키와 토큰의 숨겨진 전쟁: 현대 웹 보안에서 놓치지 말아야 할 5가지 반전 인사이트

웹 로그인의 편의 뒤에 숨겨진 보안 설계와, 쿠키·토큰 전략에서 주목해야 할 다섯 가지 인사이트를 시니어 관점으로 정리합니다.

---

## 1. 핵심 요약 (Executive Summary)

현대 웹 애플리케이션의 인증·저장소 전략은 트레이드오프의 연속입니다. 주요 요점은 다음과 같습니다.

- 액세스 토큰은 인메모리에 보관하고(새로고침 시 소멸), 리프레시 토큰은 Secure·HttpOnly 쿠키에 두는 방식이 권장됩니다.
- LocalStorage는 XSS에 취약하고, HttpOnly 쿠키는 CSRF에 노출될 수 있습니다. 저장소 선택에 따라 방어 우선순위를 달리 설정해야 합니다.
- 브라우저 정책(예: SameSite=Lax)과 종속성 스캐닝(CI/CD 통합)은 운영 단계에서의 핵심 방어 수단입니다.

---

## 2. JWT 저장소의 딜레마: LocalStorage vs HttpOnly 쿠키

개발자들 사이의 오래된 질문 "JWT를 어디에 저장할 것인가?"는 공격 표면 관점에서 접근해야 합니다.

- **LocalStorage (편의성 높음, 접근 쉬움)**: 자바스크립트로 접근 가능하므로 XSS 발생 시 토큰 탈취 위험이 큽니다.
- **HttpOnly 쿠키 (자바스크립트 접근 차단)**: XSS로부터 안전하지만, 브라우저가 자동으로 쿠키를 포함하기 때문에 CSRF 방어가 필요합니다.

핵심 인사이트: 저장소 자체가 '절대적 해법'이 아니다 — 선택에 따라 방어해야 할 위협이 바뀝니다.

---

## 3. 토큰 저장 권장 아키텍처: 인메모리 + HttpOnly 리프레시

권장 패턴:

1. Access Token: 단기 유효(예: 10분), 브라우저 메모리에 보관.
2. Refresh Token: 장기 유효, Secure·HttpOnly 쿠키에 보관.
3. Silent Refresh: 클라이언트가 직접 `/refresh_token`을 호출하여 액세스 토큰을 갱신.

주의:

- 서버 미들웨어가 자동으로 모든 요청에서 토큰을 갱신하도록 하는 방식은 CSRF 위험을 높일 수 있습니다.
- 클라이언트 주도 갱신은 CSRF에 대한 추가 제어(예: SameSite, CSRF 토큰 등)와 결합되어야 안전합니다.

---

## 4. 전이적 의존성(Transitive Dependencies)과 공급망 보안

현대 프로젝트는 수백~수천 개의 전이적 의존성을 가집니다. 단 한 패키지의 취약점이 전체 시스템을 위태롭게 할 수 있습니다.

- 권장 실무: `npm audit`, `pip-audit`, `Snyk` 같은 도구를 CI/CD에 통합하고, Critical/High 수준 취약점은 차단 정책을 적용하세요.
- 의사결정 흐름: 직접 의존성 → 상위 패키지 업데이트 → 악용 가능성(Exploitable) 평가 → 대안/대체 패키지 검토 → 필요시 기능 자체 대체.

---

## 5. SameSite 정책과 실무 적용

- **Strict**: 가장 보수적, 외부 컨텍스트에서 쿠키 전송 차단.
- **Lax (브라우저 기본)**: 안전한 네비게이션에서는 쿠키를 허용하나 상태 변경 요청에서는 제한적 보호를 제공.
- **None**: 모든 컨텍스트에서 전송되므로 `Secure`(HTTPS)와 함께 사용해야 함.

실무 팁: 민감한 상태 변경엔 Anti-CSRF 토큰을 병행하는 것이 권장됩니다.

---

## 6. 프론트엔드에서의 XSS 방어: dangerouslySetInnerHTML 경고

React의 `dangerouslySetInnerHTML`은 이름 자체가 경고입니다. 하드코딩된 문자열이라도 동적 데이터와 결합하면 XSS 경로가 됩니다.

예시:

```tsx
const message = `Welcome, ${username}!`;
// 절대 정제 없이 innerHTML에 넣지 말 것
// 필요 시 DOMPurify로 정제
```

권장: 사용자 입력을 렌더링할 때는 컨텍스트에 맞는 이스케이프/정제(예: DOMPurify)와 CSP를 병행하세요.

---

## 7. 결론 및 실무 액션 아이템

웹 보안은 일회성 체크리스트가 아니라 지속적인 프로세스입니다. 권장 액션:

- `Access Token`은 인메모리, `Refresh Token`은 Secure·HttpOnly 쿠키로 구성.
- CI/CD에 종속성 스캐닝을 통합하고, 취약점 정책(차단 기준)을 마련.
- `dangerouslySetInnerHTML` 사용 지침을 문서화하고, DOMPurify/CSP를 표준으로 채택.

---

## 실무 예시 및 참고 자료

- `npm audit`, `Snyk`, `Dependabot`
- OWASP: XSS 및 CSRF 가이드
- Hasura authentication recommendations

필요하시면 이 문서를 `component-design` 또는 `performance-optimization` 등 원하는 폴더로 이동하고, 커밋까지 만들어드리겠습니다.
