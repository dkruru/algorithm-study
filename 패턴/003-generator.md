# 제너레이터 (Generator)

**출처**: 프로그래머스 - 돗자리 깔기 (학습 메모)
**날짜**: 2026-04-16
**태그**: #python #generator #lazy-evaluation #memory #idiom

## 한 줄 정의

> **값을 미리 다 만들어두지 않고, 필요할 때마다 하나씩 꺼내서 주는 것**

리스트는 "도시락 100개 미리 싸두는 것", 제너레이터는 "주문 들어올 때마다 하나씩 만들어주는 것".

---

## 리스트 vs 제너레이터

```python
# 리스트 — 100만 개 다 만들어서 메모리에 보관
nums_list = [x * 2 for x in range(1_000_000)]

# 제너레이터 — 만들 "방법"만 가지고 있음, 메모리 거의 0
nums_gen = (x * 2 for x in range(1_000_000))
```

**대괄호 `[]` → 리스트, 소괄호 `()` → 제너레이터**.
이 차이만으로 동작이 완전히 달라짐.

### 메모리 차이
```python
import sys
sys.getsizeof([x for x in range(1_000_000)])    # 약 8MB
sys.getsizeof((x for x in range(1_000_000)))    # 약 200바이트
```

약 4만 배 차이. 제너레이터는 "내용물"이 아니라 "만드는 레시피"만 들고 있음.

---

## 동작 원리: Lazy Evaluation

요청받을 때만 값을 계산 (게으른 계산).

```python
gen = (x * 2 for x in range(5))

next(gen)   # 0   ← 이때 처음으로 0*2 계산
next(gen)   # 2   ← 이때 처음으로 1*2 계산
next(gen)   # 4
next(gen)   # 6
next(gen)   # 8
next(gen)   # StopIteration 예외 (끝)
```

`for` 루프나 `all()`/`any()`는 내부적으로 `next()`를 자동 호출.

---

## `all()` + 제너레이터의 위력

```python
all(park[x][y] == '-1'
    for x in range(i, i + size)
    for y in range(j, j + size))
```

**리스트였다면**:
1. `size × size`개 칸을 **전부 검사해서** True/False 리스트를 만든 뒤
2. `all()`이 그 리스트를 또 검사

**제너레이터라서**:
1. `all()`이 한 칸씩 요청
2. 첫 `False`가 나오는 순간 **즉시 멈춤** (short-circuit)
3. 나머지 칸은 검사조차 안 함

5×5 영역에서 첫 칸이 사람 있으면, 25번 검사할 걸 1번만 하고 끝.

---

## 제너레이터를 만드는 두 가지 방법

### 1. 제너레이터 표현식 (간단, 자주 씀)

```python
gen = (x * 2 for x in range(10))
```

리스트 컴프리헨션과 똑같은데 **대괄호만 소괄호로**.

### 2. `yield` 키워드를 쓴 함수

```python
def my_gen():
    yield 1
    yield 2
    yield 3

for x in my_gen():
    print(x)   # 1, 2, 3
```

`return` 대신 `yield`를 쓰면 제너레이터 함수가 됨.
`yield`마다 잠깐 멈췄다가 다음 호출 때 그 자리에서 재개.

복잡한 로직 → 함수 형태, 단순 → 표현식 형태.

---

## ⚠️ 주의사항: 한 번 쓰면 끝

제너레이터는 **1회용**:

```python
gen = (x * 2 for x in range(3))

list(gen)   # [0, 2, 4]
list(gen)   # []  ← 이미 다 소진됨!
```

리스트는 여러 번 OK:
```python
lst = [x * 2 for x in range(3)]
list(lst)   # [0, 2, 4]
list(lst)   # [0, 2, 4]  ← 몇 번이고 OK
```

여러 번 써야 하면 리스트, 한 번만 훑으면 제너레이터.

---

## 언제 제너레이터를 쓸까?

### ✅ 쓰면 좋을 때
- **데이터가 큼** (메모리 절약)
- **`all`/`any`/`sum`/`max`/`min` 같은 단일 패스 함수와 결합** (short-circuit)
- **무한 시퀀스** (예: 피보나치를 영원히)
- **파이프라인 처리** (한 번만 흘려보냄)

### ❌ 쓰면 안 좋을 때
- **여러 번 순회** 필요
- **인덱스 접근** 필요 (`gen[3]` 안 됨)
- **길이** 알아야 할 때 (`len(gen)` 안 됨)
- **데이터가 작음** — 차이 미미, 리스트가 더 직관적

---

## 비교표

| 항목 | 리스트 | 제너레이터 |
|---|---|---|
| 비유 | 도시락 100개 미리 만듦 | 주문 시 하나씩 만듦 |
| 메모리 | 전체 차지 | 거의 0 |
| 재사용 | 가능 | 1회용 |
| 인덱싱 | `lst[5]` OK | 불가 |
| `len()` | OK | 불가 |
| 무한 시퀀스 | 불가 | 가능 |
| 만드는 법 | `[ ... ]` | `( ... )` 또는 `yield` |

---

## 실전 예시

### 1. 큰 파일 한 줄씩 처리
```python
# 파일 객체 자체가 제너레이터처럼 동작
with open('big_file.txt') as f:
    for line in f:           # 한 줄씩 lazy하게
        process(line)
```

### 2. 무한 수열
```python
def naturals():
    n = 1
    while True:
        yield n
        n += 1

from itertools import islice
list(islice(naturals(), 10))   # [1,2,3,...,10]
```

### 3. all/any와 결합 (이번 패턴)
```python
# 큰 리스트에 음수 하나만 있어도 즉시 종료
any(x < 0 for x in huge_list)
```

### 4. 합계 계산
```python
# 메모리 효율적으로 큰 데이터 합계
total = sum(x ** 2 for x in range(10_000_000))
```

---

## C와의 비교

C에는 직접 대응 개념 없음. 굳이 비유하면:
- **리스트** = 배열 전체를 `malloc`해서 채워둠
- **제너레이터** = 콜백 호출할 때마다 다음 값을 계산해 리턴

코루틴(coroutine), 이터레이터 패턴과 유사. 파이썬은 이걸 언어 차원에서 자연스럽게 지원.

---

## 한줄 정리

> **제너레이터 = 값의 시퀀스를 lazy하게 만드는 객체**
> **소괄호 `( ... for ... )`** 한 줄로 만들 수 있고
> **`all`/`any`와 결합하면 short-circuit으로 효율 극대화**

## 관련 노트

- [001 all/any + 제너레이터](001-all-any-generator.md)
- [002 컴프리헨션 다중 for](002-comprehension-multi-for.md)
