# 🐾 Zoo 프로그램 구조 피드백

## 📌 클래스별 역할 요약

| 클래스 | 역할 및 책임 | 상호작용 대상 |
|--------|--------------|----------------|
| `Animal` | ID, 이름, 종에 대한 정보 보관 및 동등성 비교 로직 제공 | `AnimalCollection`에서 비교 및 저장 |
| `Species` | 동등성 및 암묵적 형변환, 유효성 검사 로직 포함 | `Animal`에서 내부 종 표현으로 사용됨 |
| `AnimalCollection` | 동물 객체의 컨테이너 역할<br>추가, 검색, 삭제, 중복 방지 로직 포함 | `Application`이 이 컬렉션을 이용해 상호작용 |
| `Application` | 실행 흐름 제어 및 사용자 입출력 처리 | 사용자 입력을 받아 `AnimalCollection`과 상호작용 |
| `ApplicationInputResult` | 사용자 입력을 래핑하고 종료 요청 여부 판단 | `Application`에서 종료 조건 판단 시 사용 |

---

## 🔐 캡슐화

- 모든 필드는 `private`이며, `public` 속성으로 **읽기 전용(read-only)** 제공
- `Animal`, `Species`는 **불변(Immutable)** 구조로 설계되어 있음
- `AnimalCollection` 내부 리스트 `_animals`는 외부 접근 불가 → `Add`, `Remove`, `Find` 메서드만 허용

### ✅ 의도

- **책임 분리**  
  `Application`은 입출력 담당, `AnimalCollection`은 상태 저장 담당으로 역할이 명확함

- **데이터 보호**  
  외부에서 내부 필드를 직접 수정하지 못하게 하여 이상 동작 및 버그 예방

---

## 🚫 확장을 염두하지 않은 부분

- **UI와 로직이 강결합**  
  `Console.WriteLine`, `Console.ReadLine`을 직접 사용 → 테스트 코드 작성이 어려움

- **데이터 영속성 없음**  
  모든 데이터는 메모리에만 존재하며, 프로그램 종료 시 모두 소멸

- **중복 판별이 단순함**  
  `Equals`는 ID만 비교하므로, 이름과 종이 같아도 ID 다르면 다른 개체로 인식됨

---

## 🛠️ 설계 관련 개선 의견

1. ### `AnimalCollection`의 얕은 복사 문제
   - `List<Animal>`이 외부에 노출될 경우 상태가 변경될 수 있음  
   → **방어적 복사**를 통해 외부로 안전하게 제공해야 함

2. ### `Species`에서 연산자 오버로딩 시 null 체크 필요
   - `==`, `!=` 오버로딩 시 `x`, `y`가 null일 때 예외 발생 가능  
   → null 안전성 추가 필요

3. ### `ApplicationInputResult`의 암시적 변환 문제
   - `implicit operator string`은 **명확성과 가독성 저하**  
   → 암시적 변환 대신 명시적 메서드 또는 속성 접근으로 변경 제안

4. ### 사용자 입력 검증 부족
   - `ReadValue()`에서 사용자 입력을 필터링 없이 사용  
   → **정규식**, **길이 제한** 등 유효성 검사 추가 필요

5. ### `Animal`의 `Equals`와 `GetHashCode` 불일치
   - ID가 고유한데 `Equals`는 Name, Species까지 비교함  
   → 중복 판단 기준을 **ID만으로 일관성 있게 유지**하거나,  
     ID는 식별자고 비교는 Name + Species 기준으로 정리 필요

6. ### `Species` 표현의 일관성 부족
   - "개", "dog", "Dog" 등의 표현이 동일한 종인데 다르게 인식될 수 있음  
   → `ToLowerInvariant()` + normalization 또는 `enum` 사용 고려

7. ### `Animal`의 불변성 구조 재고
   - ID는 불변이 좋지만, Name과 Species는 변경 가능한 구조도 검토할만 함  
   → `UpdateInfo()` 같은 메서드를 제공해 제한된 변경 허용 가능

---

## ✅ 요약

이 코드는 **작고 명확한 설계**를 기반으로 동물 정보를 관리하는 시스템을 구현하고 있으며, **불변성과 책임 분리**에 대한 의식이 잘 반영되어 있음.  
다만 **확장성, 테스트성, 표현 일관성** 등에서 몇 가지 개선점이 발견되며, 향후 프로젝트 확장을 고려한다면 구조의 리팩토링이 필요함.
