---
title: "[Day 3] 네트워크 최적화와 비동기 데이터 관리 (Data Fetching)"
---

# [Day 3] 네트워크 최적화와 비동기 데이터 관리 (Data Fetching)

1~2일 차에서 화면을 '그리는' 비용을 줄였다면, 오늘은 데이터를 '가져오는' 과정에서 발생하는 병목을 제거하고 사용자 경험(UX)을 극대화하는 법을 다룹니다.

## 1. 오늘의 핵심 개념 (3줄 요약)

- **Stale-While-Revalidate (SWR):** 낡은 데이터를 즉시 보여주고 백그라운드에서 새 데이터를 가져와 교체하는 전략. 로딩 스피너 시간을 최소화합니다.
- **Waterfall 현상 방지:** 여러 API가 순차적으로 호출되어 렌더가 지연되는 것을 `Promise.all` 또는 상위 레벨 호이스팅으로 해결합니다.
- **Optimistic UI (낙관적 업데이트):** 서버 응답을 기다리지 않고 미리 성공을 가정해 UI를 업데이트하여 즉각적인 반응성을 제공합니다.

## 2. 네트워크 성능 최적화 치트시트

- **전략적 데이터 호출**
  - **Prefetching:** 링크 hover나 라우트 진입 직전에 `queryClient.prefetchQuery`로 미리 로드.
  - **Lazy Loading:** 당장 필요 없는 데이터/이미지는 `loading="lazy"` 또는 `IntersectionObserver`로 지연 로드.
- **데이터 페이로드 최적화**
  - **BFF(Backend For Frontend):** 프론트엔드에 꼭 필요한 필드만 내려주어 전송량을 줄임.
- **병렬화 & 데듀프**
  - 여러 독립적 쿼리는 `Promise.all`로 병렬 실행. 같은 `queryKey`는 TanStack Query가 데듀프 처리.

## 3. 실무 SOP: 네트워크 분석 절차

1. **중복 호출 확인:** DevTools > Network에서 동일한 엔드포인트/파라미터가 반복되는지 확인.
2. **Waterfall 분석:** 호출이 계단식으로 이어지면 병렬화·호이스팅 필요.
3. **Throttling 테스트:** 네트워크를 `Fast 3G`로 강제해 스켈레톤/Suspense 도입 필요 여부 판단.

## 4. 구현 팁 & 코드 스니펫

- TanStack Query 기본값 예시:

```js
const qc = new QueryClient({
  defaultOptions: {
    queries: { staleTime: 1000 * 60 * 2, cacheTime: 1000 * 60 * 5, retry: 2 },
  },
});
```

- 라우트 진입 전 prefetch 예시:

```js
await queryClient.prefetchQuery(["profile", id], () => fetchProfile(id));
```

- 병렬 데이터 페칭 예시:

```js
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

- 요청 취소 기본 패턴:

```js
const controller = new AbortController();
fetch(url, { signal: controller.signal });
// 컴포넌트 언마운트 시
controller.abort();
```

## 5. 옵티미스틱 업데이트(낙관적 업데이트)

- 옵티미스틱 업데이트는 UX를 개선하지만 롤백·동시성 문제를 고려해야 합니다. 상세 구현 및 패턴은 별도 문서에 정리했습니다: [옵티미스틱 업데이트 — TanStack Query 가이드](optimistic-updates-tanstack-query.md)

## 6. 심화(권장 검토 항목)

- **스트리밍 SSR / 점진 하이드레이션:** React 19의 스트리밍을 활용해 중요한 부분 우선 전송, `startTransition`/`Suspense`로 비핵심 하이드레이션 지연.
- **HTTP/2·HTTP/3 활용:** 멀티플렉싱으로 연결 재사용, 서버 푸시는 신중히 사용.
- **재시도 정책:** 지수 백오프 + 지터; 실패폭이 클 땐 서킷브레이커 도입 고려.
- **관측성:** TTFB, P50/P95/P99 레이턴시, 에러율, RUM + 분산 트레이스 조합 권장.

## 7. 실전 체크리스트

- 데이터는 가능한 한 라우트/페이지 수준에서 호이스팅.
- 동일한 데이터는 `queryKey`로 데듀프 처리.
- `staleTime`/`cacheTime`으로 재요청 최소화.
- LCP 리소스는 `fetchpriority="high"`, 기타는 `loading="lazy"`.
- Chrome Network Waterfall + 개별 요청 스로틀링으로 병목 재현.

## 8. 참고 자료

- Day1: [day1-pipeline-and-reflow.md](day1-pipeline-and-reflow.md)
- Day2: [day2-state-management-and-rendering.md](day2-state-management-and-rendering.md)
- 옵티미스틱 심화: [optimistic-updates-tanstack-query.md](optimistic-updates-tanstack-query.md)
