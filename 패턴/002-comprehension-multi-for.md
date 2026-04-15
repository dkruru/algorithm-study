# 컴프리헨션의 다중 for 문법

**출처**: 프로그래머스 - 돗자리 깔기 (학습 메모)
**날짜**: 2026-04-16
**태그**: #python #comprehension #generator #idiom

## 인상 깊었던 코드

```python
all(park[x][y] == '-1'
    for x in range(i, i + size)
    for y in range(j, j + size))
```

`for`를 두 개 연결해서 2D 영역을 한 번에 순회. C에는 없는 문법이라 처음 보면 어색하지만 익히면 강력함.

---

## 기본 규칙

컴프리헨션/제너레이터에서 `for`를 여러 개 연결하면 **중첩 루프와 동일**.
**왼쪽이 바깥, 오른쪽이 안쪽**.

```python
# 컴프리헨션
[(x, y) for x in range(3) for y in range(2)]

# 동일한 일반 루프
result = []
for x in range(3):       # 바깥 (왼쪽)
    for y in range(2):   # 안쪽 (오른쪽)
        result.append((x, y))
```

결과:
```python
[(0,0), (0,1), (1,0), (1,1), (2,0), (2,1)]
```

---

## 헷갈리기 쉬운 점: 일반 루프와 시각적 순서가 반대

| 형태 | 바깥 위치 | 안쪽 위치 |
|---|---|---|
| 일반 루프 (세로) | 위 | 아래 (들여쓰기) |
| 컴프리헨션 (가로) | 왼쪽 | 오른쪽 |

세로 ↔ 가로 변환이라고 생각하면 직관적.

---

## 안쪽 루프가 바깥 변수 참조 가능

```python
# 위쪽 삼각형 좌표만
[(x, y) for x in range(3) for y in range(x + 1)]
# [(0,0), (1,0), (1,1), (2,0), (2,1), (2,2)]
```

`y`의 범위가 `x`에 따라 변함. 일반 루프로:
```python
for x in range(3):
    for y in range(x + 1):    # x를 참조
        ...
```

---

## 조건 필터 (`if`)

```python
# 짝수 합 좌표만
[(x, y) for x in range(3) for y in range(3) if (x + y) % 2 == 0]
```

`if`는 가장 안쪽 for 다음에 위치. 여러 `if` 연결도 가능:
```python
[(x, y) for x in range(10) for y in range(10) if x > y if x + y < 8]
```

---

## 이번 문제에 적용

```python
all(park[x][y] == '-1'
    for x in range(i, i + size)      # 바깥 (행)
    for y in range(j, j + size))     # 안쪽 (열)
```

동등한 일반 루프:
```python
for x in range(i, i + size):
    for y in range(j, j + size):
        yield park[x][y] == '-1'
```

---

## 실용적 한계: 2~3중까지만

문법적으로는 무한히 연결 가능하지만, **가독성은 2~3중이 마지노선**.

```python
# 2중 — 깔끔
[(x, y) for x in A for y in B]

# 3중 — 아직 읽힘
[(x, y, z) for x in A for y in B for z in C]

# 4중 이상 — 일반 for로 풀어 쓰는 게 나음
[... for a in A for b in B for c in C for d in D]   # ❌
```

4중 이상은 본인도 일주일 뒤에 못 읽음. 일반 루프로.

---

## 줄바꿈 권장 (PEP 8)

```python
# 가독성 나쁨
all(park[x][y] == '-1' for x in range(i, i + size) for y in range(j, j + size))

# 가독성 좋음 — for마다 줄바꿈
all(park[x][y] == '-1'
    for x in range(i, i + size)
    for y in range(j, j + size))
```

---

## 적용 가능한 컴프리헨션 종류

같은 다중 for 문법이 모두 동일하게 동작:

```python
# 리스트
[expr for x in A for y in B]

# 제너레이터
(expr for x in A for y in B)

# 집합
{expr for x in A for y in B}

# 딕셔너리
{k: v for x in A for y in B}

# all/any와 결합
all(cond for x in A for y in B)
any(cond for x in A for y in B)
```

---

## 정리

| 문법 | 의미 |
|---|---|
| `expr for x in A for y in B` | A가 바깥, B가 안쪽 (이중 루프) |
| `expr for x in A for y in range(x)` | 안쪽이 바깥 변수 참조 |
| `expr for x in A for y in B if cond` | 조건 필터 |

> **왼쪽이 바깥** — 이것 하나만 기억하면 됨.

## 관련 노트
- [001 all/any + 제너레이터](001-all-any-generator.md)
