# [Day 1] 브라우저 렌더링 파이프라인과 최적화 (Reflow & Repaint)

> **학습 날짜:** 2026.03.16
> **학습 목표:** 브라우저 렌더링 과정을 이해하고, 불필요한 렌더링 낭비를 직접 눈으로 확인한다.

## 1. 오늘의 핵심 개념 (파인만 기법으로 3줄 요약)
- 브라우저가 화면을 그리는 순서는 [ DOM -> CSSOM -> Render Tree -> Layout -> Paint -> Composite ] 이다.
- **Reflow(Layout)란?** [ 요소의 크기나 위치가 바뀌어 브라우저가 공간을 다시 계산하는 무거운 작업 ]
- **Repaint란?** [ 색상 등 시각적 요소만 바뀌어 레이아웃 계산 없이 화면만 다시 칠하는 작업 ]


## 2. 렌더링 비용 치트시트 (자주 쓰는 속성 위주)
- **🚨 Reflow 유발 (가장 비쌈, 레이아웃 변경):** `width`, `height`, `margin`, `padding`, `top`, `left`, `display`
- **⚠️ Repaint 유발 (중간 비용, 색상 변경):** `color`, `background-color`, `border-radius`, `box-shadow`
- **✅ Composite 유발 (가장 쌈, GPU 가속):** `transform`, `opacity`
> **결론:** 애니메이션이나 동적 UI를 만들 때는 가급적 `transform`과 `opacity`만 써서 GPU 가속을 태우자!

## 3. 실무 적용: 렌더링 성능 분석 매뉴얼 (SOP)
화면 버벅임이나 렌더링 지연 이슈가 발생했을 때, 다음 3단계로 원인을 분석한다.
1. **시각적 낭비 확인:** 개발자 도구 > `Rendering` > `Paint flashing` 체크 후 UI 조작해 보기 (초록색 깜빡임 확인)
2. **병목 수치화:** `Performance` 탭에서 녹화(Record) 후, 보라색(Layout)과 초록색(Paint) 블록 점유율 확인
3. **리팩토링:** 원인이 되는 CSS 속성을 파악하고, `transform`이나 `opacity`로 변경하여 GPU 가속 태우기

## 4. 크롬 개발자 도구 실습 (눈으로 확인하기)
- **테스트 타겟 사이트:** [ 예: 네이버 웹툰 메인 / 회사 관리자 페이지 ]
- **현상 관찰:** - [ 예: 스크롤을 내릴 때마다 상단 헤더 전체가 초록색으로 불필요하게 번쩍(Repaint)인다. ]
  - [ 예: 특정 버튼에 마우스를 올렸더니, 버튼뿐만 아니라 주변 영역까지 다시 그려진다. ]
- **스크린샷:** ![DevTools Paint Flashing 예시](이미지 링크)


## 4. 내일 실무 적용 포인트 (Action Item)
- [ ] 회사 프로젝트의 `[ 특정 모달창 / GNB 메뉴 ]` 컴포넌트 렌더링 성능 측정해 보기
- [ ] 애니메이션 효과가 들어간 CSS 코드에서 `width`나 `height`를 건드리고 있는지 찾아내기

## 5. 참고 자료 (Reference)
- [MDN: Critical rendering path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
- [web.dev: Rendering Performance](https://web.dev/rendering-performance/)
- [CSS Triggers - CSS 속성별 렌더링 비용](https://csstriggers.com/)