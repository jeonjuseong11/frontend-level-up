# [Day 2] 상태 관리 아키텍처와 리렌더링 최적화 (React 기준)

## 요약 (TL;DR)

- 리렌더링은 `state` 변경, `props` 변경, 부모 렌더로 발생한다. 불필요한 리렌더링은 메모이제이션과 상태 분리로 줄인다.
- React 19의 `React Compiler`는 빌드 단계 최적화를 통해 일부 identity 기반 리렌더를 줄일 수 있지만, 근본적 규칙은 런타임에 의해 유지된다. 프로파일러로 전후 비교가 필수다.

---

## 1. 학습 메타

> **학습 날짜:** 2026.03.20  
> **학습 목표:** React의 리렌더링 조건 이해 및 React Profiler로 병목 파악

## 2. 핵심 개념 (파인만 기법으로 3줄 요약)

- **리렌더링의 3가지 조건:**

1. `state` 변경
2. `props` 변경
3. 부모 컴포넌트의 렌더

- **불필요한 렌더링(Wasted Render):** UI가 동일한데 컴포넌트 함수가 반복 실행되는 현상
- **최적화 무기:** `React.memo`, `useMemo`, `useCallback` (단, 메모리 비용이 발생하므로 남발 금지)

## 3. 상태 관리 아키텍처 치트시트

- **지역 상태 (Local State):** 컴포넌트 내부/자식만 사용하는 상태 → `useState`
- **전역 상태 (Global State):** 앱 전역 공유(테마, 유저) → `Zustand` / `Recoil` / `Redux`
- **서버 상태 (Server State):** API 데이터 → `React Query` / `SWR`로 캐싱 및 동기화

## 4. 실무 SOP: 리렌더링 분석 절차

1. **렌더링 깜빡임 확인:** React DevTools Profiler에서 `Highlight updates` 활성화
2. **병목 수치화:** Profiler 녹화 → Flamegraph/Ranked view에서 비용 높은 컴포넌트 찾기
3. **원인 분석 및 조치:** "Why did this render?" 확인 → 상태 위치 이동(Lifting State Down), `React.memo`/`useCallback` 적용

## 5. React 19 `React Compiler` 심화 분석

### 5.1. 개요 및 런타임과의 관계

- `React Compiler`는 런타임 이전(빌드/트랜스파일 단계)에 소스 코드를 분석하고 변환해 런타임 성능을 개선하려는 도구다.
- **바뀌지 않는 것:** React의 기본 리렌더링 규칙(`state`/`props`/부모 렌더 기준)은 런타임의 행위이므로 컴파일러 도입만으로 규칙 자체가 바뀌지는 않는다.
- **바뀌는 것:** 컴파일러가 함수/객체의 identity를 안정화하면, identity 기반으로 발생하던 불필요한 자식 리렌더링이 줄어들어 전체 렌더 비용이 개선된다.

### 5.2. 주요 변환 예시 (컴파일러가 하는 일)

- **콜백/함수 호이스팅(hoisting):** 인라인으로 선언되는 함수 또는 단순한 콜백을 상위 스코프로 끌어올려 identity를 안정화
- **상수화(const-folding) / 리터럴 정적화:** 변경되지 않는 객체/배열을 빌드 시점에 고정하여 재생성 비용 제거
- **불필요한 표현식 제거:** dead code elimination, tree-shaking으로 런타임 코드 양을 감소
- **선택적 자동 메모이제이션:** 패턴에 따라 `useMemo` 수준의 캐싱 적용 가능

### 5.3. 예제: 인라인 함수 리렌더 유발 방지

```jsx
// Before (일반 패턴)
function Parent() {
  const [q, setQ] = useState("");
  return <Child onChange={(v) => setQ(v)} />;
}

// After (컴파일러가 호이스팅/안정화 적용할 수 있는 변환 예시)
const _hoisted_onChange = (v, setQ) => setQ(v);
function Parent() {
  const [q, setQ] = useState("");
  return <Child onChange={(v) => _hoisted_onChange(v, setQ)} />;
}
```

### 5.4. 실무 도입 시 한계 및 검증 절차

- **주의점:** 자동 변환은 클로저 캡처나 레퍼런스 의존 코드에서 의도치 않은 동작을 유발할 수 있다. 컴파일러가 자동으로 `useMemo`/`useCallback`을 완전히 대체하는 것은 아니며, 모든 코드가 항상 최적화되는 건 아니다.
- **실무 검증 절차:**
  1. 로컬에서 컴파일러 활성화 브랜치 생성
  2. 동일한 시나리오로 `⚛️ Profiler` 녹화 후 활성화 전/후(Before/After) 비교
  3. 클로저/레퍼런스 의존 코드에 대한 회귀 테스트 및 기능 검증
  4. CI 통과 후 점진적 롤아웃(Canary 배포)
  5. 여전히 발생하는 identity 기반 리렌더링은 `React.memo`/`useCallback` 등으로 수동 보완

---

## 6. 내일 실무 적용 포인트 (Action Items)

- [ ] 입력폼 타이핑 시 전체 리렌더링 체크 (Profiler `Highlight updates` 활용)
- [ ] API 응답을 전역 상태(Redux 등)에 보관한 레거시 코드 탐색 및 React Query 도입 검토

---

## 7. 참고 자료

- [React 공식 문서 - React.memo (한글)](https://ko.react.dev/reference/react/memo)
- [React 공식 문서 - React Developer Tools 사용법](https://ko.react.dev/learn/react-developer-tools)
- [Mark Erikson - A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/)
- [Dan Abramov - Before You memo()](https://overreacted.io/ko/before-you-memo/)
- [TkDodo - Practical React Query - React Query as a State Manager](https://tkdodo.eu/blog/react-query-as-a-state-manager)
