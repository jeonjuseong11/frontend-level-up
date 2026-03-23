# [Day 4] 컴포넌트 디자인 패턴과 선언적 아키텍처

성능 최적화가 애플리케이션의 "속도"를 결정한다면, 아키텍처와 컴포넌트 설계는 코드의 "유지보수와 지속 가능성"을 결정합니다. 오늘은 복잡한 비즈니스 로직과 UI를 분리하여 변화에 유연하게 대응할 수 있는 설계 방법을 다룹니다.

---

## 1. 🧠 오늘의 핵심 개념 (Core Concepts)

### ① Headless Component 패턴

- **개념:** UI(디자인)와 로직(상태 및 기능)을 완전히 분리하는 방식입니다.
- **특징:** 컴포넌트는 오직 '기능'과 '상태'만을 관리하며, 렌더링을 위한 UI는 사용하는 쪽에서 결정합니다. (`Radix UI`, `Headless UI`가 대표적)
- **효과:** 디자인 요구사항이 변하더라도 핵심 비즈니스 로직을 전혀 수정할 필요 없이 재사용할 수 있습니다.

**💡 코드 예시:**

```tsx
// 로직만 담고 있는 Custom Hook (Headless)
function useToggle(initialState = false) {
  const [isOpen, setIsOpen] = useState(initialState);
  const toggle = () => setIsOpen((prev) => !prev);
  return { isOpen, toggle };
}

// UI는 사용하는 곳에서 자유롭게 구성
function ToggleButton() {
  const { isOpen, toggle } = useToggle(); // 로직 가져오기
  return <button onClick={toggle}>{isOpen ? "켜짐" : "꺼짐"}</button>; // UI만 작성
}
```

### ② Compound Component (합성 컴포넌트) 패턴

- **개념:** 여러 개의 작은 컴포넌트들이 암묵적으로 상태를 공유하며 하나의 큰 기능을 완성하는 패턴입니다.
- **특징:** `<Select>`, `<Select.Option>` 처럼 부모-자식 관계의 컴포넌트가 협력합니다. Context API를 활용해 자식 컴포넌트 간에 상태를 공유합니다.
- **효과:** 직관적이고 가독성이 높으며, 사용자(개발자)가 컴포넌트의 배치를 유연하게 결정할 수 있습니다.

**💡 코드 예시:**

```tsx
const SelectContext = createContext();

// 1. 부모 컴포넌트: 상태 관리 및 Context Provider 역할
function Select({ children }) {
  const [value, setValue] = useState('');
  return (
    <SelectContext.Provider value={{ value, setValue }}>
      <div className="select-box">{children}</div>
    </SelectContext.Provider>
  );
}

// 2. 자식 컴포넌트: 실제 선택지 UI 요소
Select.Option = function Option({ children, val }) {
  const { value, setValue } = useContext(SelectContext);
  return (
    <div onClick={() => setValue(val)} className={value === val ? 'selected' : ''}>
      {children}
    </div>
  );
}

// 3. 실제 사용처: Props를 깊게 전달하지 않고 선언적인 UI 구조 구성
<Select>
  <Select.Option val="react">React</Select.Option>
  <Select.Option val="vue">Vue</Select.Option>
</Select>
```

### ③ 관심사 분리 (Separation of Concerns, SoC)

- **개념:** 데이터의 호출(Data), 가공(Logic), 출력(View) 영역을 명확히 나누는 원칙입니다.
- **적용:** 3년 차 프론트엔드 개발자라면 UI 컴포넌트 내부에 복잡한 상태 관리나 API 호출 로직을 직접 두기보다, 커스텀 훅(`use...`)을 통해 철저히 분리해야 합니다.

**💡 코드 예시:**

```tsx
// ❌ Bad: API 호출, 상태 관리, 렌더링이 한 곳에 섞인 컴포넌트
function UserList() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch("/api/users")
      .then((res) => res.json())
      .then(setUsers);
  }, []);
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}

// ✅ Good: 데이터 패칭 로직(Hook)과 View(Component) 분리
function useUsers() {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch("/api/users")
      .then((res) => res.json())
      .then(setUsers);
  }, []);
  return users;
}

function UserList() {
  const users = useUsers(); // 비즈니스 로직은 Custom Hook으로 위임
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  ); // UI는 렌더링에만 집중
}
```

---

## 2. 💡 컴포넌트 설계 철학 (Design Philosophy)

- **재사용성(Reusability) vs 복잡도(Complexity)**
  - "모든 것을 공통 컴포넌트로 만들려는 함정"을 경계하세요. 처음부터 완벽한 추상화를 하려다 보면 오히려 컴포넌트가 무거워집니다. **(Rule of Three: 3번 이상 중복될 때 추상화를 고민할 것)**
- **Prop Drilling과 Composition**
  - 상태나 함수를 3~5단계 아래로 계속 전달하고 있다면 아키텍처를 의심해야 합니다.
  - `Context API`를 남용하기 전에, `children` `renderProps`을 활용한 **컴포넌트 합성(Composition)**으로 해결할 수 있는지 먼저 고민하세요.

---

## 3. 🛠️ 실무 SOP: 좋은 컴포넌트인지 검증하는 3가지 질문

1. **단일 책임 원칙 (SRP):** _"이 컴포넌트가 하는 일을 한 문장으로 명확히 설명할 수 있는가?"_
   - (설명에 '그리고(~하고)'가 들어간다면 분리 신호입니다.)
2. **의존성 결합도 (Coupling):** _"이 컴포넌트가 특정 API 응답 구조나 전역 상태(Redux/Zustand 등)에 너무 강하게 결합되어 있지 않은가?"_
3. **제어권 위임 (Inversion of Control):** _"컴포넌트 내부에서 모든 것을 하드코딩으로 결정하고 있지 않은가? 부모 컴포넌트가 제어할 수 있는 여지(Props, Render Props)를 남겨두었는가?"_

---

## 4. 💻 실습 가이드

아래 템플릿을 참고하여 직접 리팩토링을 진행해 보세요.

```markdown
### [Day 4] 컴포넌트 디자인 패턴 실습

- **오늘의 도전:** 프로젝트 내 가장 복잡한 '모달(Modal)'이나 '필터(Filter)' 컴포넌트를 Compound 패턴 또는 Headless 패턴으로 리팩토링하기
- **Before:** 하나의 컴포넌트에 수많은 Props(isOpen, onConfirm, onCancel, title, description...)가 전달되고, 내부가 300줄이 넘어가는 상태.
- **After:** `<Modal> <Modal.Header /> <Modal.Body /> <Modal.Footer /> </Modal>` 구조로 책임을 분리.
- **배운 점/느낀 점:** 부모와 자식 간의 책임이 명확해졌고, 특정 페이지에서 `Modal.Footer`가 필요 없을 때 쉽게 제외할 수 있어 유연성이 크게 향상됨.
```

---

## 5. 🚀 내일 실무 적용 포인트 (Action Items)

- [ ] 현재 작성 중인 컴포넌트 중 **JSX와 비즈니스 로직(useEffect, 복잡한 데이터 가공 등)이 뒤섞인 곳**을 찾아 커스텀 훅으로 분리해 보기.
- [ ] 디자인 변경 요건이 잦은 '공통 버튼/입력창'에 **Headless 패턴**을 일부 도입해 유연성 확보하기.

---

## 6. 📚 참고 자료 (References)

- [Patterns.dev - Design Patterns for Modern Web Apps](https://www.patterns.dev/react)
- [Kent C. Dodds - Compound Components in React](https://kentcdodds.com/blog/compound-components-in-react)
- [Headless UI 공식 문서](https://headlessui.com/)

> **💡 Today's Review:**
> 코드를 무작정 작성하기보다, **"어떻게 하면 1개월 뒤의 내가(또는 동료가) 이 코드를 편하게 수정할 수 있을까?"**를 고민하며 구조를 스케치하는 시간을 가져보세요.
