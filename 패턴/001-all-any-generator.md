# all() + 제너레이터로 영역 검사

**출처**: 프로그래머스 - 돗자리 깔기
**날짜**: 2026-04-16
**태그**: #python #generator #idiom #2d-grid

## 인상 깊었던 코드

```python
if all(park[x][y] == '-1'
       for x in range(i, i + size)
       for y in range(j, j + size)):
    return True
```

## 내 코드는 이랬다

```python
ok = True
for di in range(k):
    for dj in range(k):
        if park[i + di][j + dj] != "-1":
            ok = False
            break
    if not ok:
        break
if ok:
    return k
```

## 왜 인상적인가

- 4중 for + 플래그 변수 + 다중 break **9줄을 1줄로 압축**
- 제너레이터라 short-circuit → 첫 실패 시 즉시 종료 (효율 동일)
- C 스타일 사고로는 떠올리기 힘든 발상 — 파이썬다운 관용구

---

## 핵심: `all()`과 `any()`

### `all(iterable)`
모든 원소가 참이면 `True`. C의 `&&`를 쭉 연결한 것과 같음.

```python
all([True, True, True])    # True
all([True, False, True])   # False
all([])                    # True (vacuous truth, 빈 것은 참)
all([1, 2, 3])             # True (0이 아닌 숫자는 참)
all([1, 0, 3])             # False (0은 거짓)
```

### `any(iterable)`
하나라도 참이면 `True`. C의 `||`를 쭉 연결한 것.

```python
any([False, False, True])  # True
any([False, False, False]) # False
any([])                    # False
```

### 비교표

| 함수 | 의미 | C 비유 | 빈 iterable 결과 |
|---|---|---|---|
| `all(xs)` | 모두 참 | `x1 && x2 && ...` | `True` |
| `any(xs)` | 하나라도 참 | `x1 \|\| x2 \|\| ...` | `False` |

---

## 단축 평가 (Short-circuit)

`all()`/`any()`는 결과가 확정되면 **즉시 멈춤**. 효율적.

```python
all(x > 0 for x in [1, 2, -1, 3, 4])
# -1을 보는 순간 False 확정 → 뒤의 3, 4는 검사 안 함

any(x < 0 for x in [1, 2, -1, 3, 4])
# -1을 보는 순간 True 확정 → 뒤는 검사 안 함
```

---

## ⚠️ 대괄호 vs 소괄호

```python
# 권장 — 제너레이터, 만들면서 검사 (빠름)
all(x > 0 for x in nums)

# 비권장 — 리스트, 전부 만들고 검사 (느림, 메모리 낭비)
all([x > 0 for x in nums])
```

작은 데이터에선 차이 미미하지만, 큰 데이터에선 short-circuit 효과 사라짐.

---

## 활용 패턴

### 1. 모두 양수인가?
```python
all(x > 0 for x in nums)
```

### 2. 문자열이 전부 숫자인가?
```python
all(c.isdigit() for c in s)
```

### 3. 2D 영역이 전부 특정 값인가? (이번 문제 패턴)
```python
all(grid[x][y] == 0
    for x in range(i, i + size)
    for y in range(j, j + size))
```

### 4. 하나라도 음수가 있는가?
```python
any(x < 0 for x in nums)
```

### 5. 리스트에 짝수가 하나라도 있는가?
```python
any(x % 2 == 0 for x in nums)
```

### 6. 모든 원소가 None이 아닌가?
```python
all(x is not None for x in items)
```

---

## 응용: 드모르간 법칙

`all`과 `any`는 부정으로 서로 변환 가능:

```python
# "모두 양수다" == "음수가 하나도 없다"
all(x > 0 for x in nums) == (not any(x <= 0 for x in nums))

# "하나라도 양수가 있다" == "모두 음수는 아니다"
any(x > 0 for x in nums) == (not all(x <= 0 for x in nums))
```

읽기 자연스러운 쪽을 고르면 됨.

---

## 언제 쓸 수 있나

- **조건 검사 루프 + 플래그 변수** 패턴이 보이면 거의 항상 `all`/`any`로 바꿀 수 있음
- 2D/다차원 격자 영역 검사
- 입력 유효성 검증 (모든 필드가 유효한가, 금지어가 하나라도 있는가)
- 자료구조 일관성 검사 (모든 노드가 조건 만족하는가)

## 한줄 정리

> **"모두 ~인가"** → `all`, **"하나라도 ~인가"** → `any`
> 둘 다 제너레이터(`( ... )`)와 함께 쓰는 게 정석.

