# 코테 01 · Hash

> 분류: 학습 / 코테 / 백엔드
> 언어: Java · 플랫폼: 프로그래머스
> 진행: 완주하지 못한 선수 ✅ / 위장 · 전화번호 목록 · 베스트앨범 예정

---

## 1. 언제 해시인가 — 문제에서의 신호

| 문장 신호 | 자료구조 |
|---|---|
| "중복이 있는지" / "짝이 안 맞는 것" | `HashSet` |
| "~별 개수" / "몇 번 등장" | `HashMap<K,Integer>` |
| "~별로 묶어서" | `HashMap<K,List<V>>` |
| "포함되어 있는지 빠르게" | `HashSet.contains` |

**판별 기준 하나면 충분하다.** 이중 반복문으로 비교하는 그림이 먼저 떠오르고, N이 10만 이상이면 거의 확정적으로 해시 문제다. O(N²) → O(N).

반대로 **순서가 답에 영향을 주면 `HashMap`은 오답**이다.
- 입력 순서 보존 → `LinkedHashMap`
- 키 정렬 필요 → `TreeMap` (조회 O(log N))

---

## 2. 실제로 쓰는 API만

### 개수 세기 — 셋 중 하나

```java
Map<String, Integer> cnt = new HashMap<>();

cnt.put(k, cnt.getOrDefault(k, 0) + 1);   // 가장 무난
cnt.merge(k, 1, Integer::sum);            // 가장 짧음
```

### 그룹핑

```java
Map<String, List<String>> group = new HashMap<>();
group.computeIfAbsent(key, x -> new ArrayList<>()).add(value);
```

`computeIfAbsent`를 모르면 매번 `containsKey` 분기를 쓰게 된다. 이거 하나로 코드 3줄이 1줄이 된다.

### put vs merge vs compute 계열 — 정리

**put은 무조건 덮어쓴다. merge는 있으면 합치고 없으면 넣는다.**

```java
map.put(k, map.getOrDefault(k, 0) + 1);  // 해시 탐색 2번 (get + put)
map.merge(k, 1, Integer::sum);           // 탐색 1번, 제자리 갱신
```

`merge(key, value, f)`의 3분기:

| 상황 | 동작 |
|---|---|
| 키 없음 (또는 값 null) | `value`를 그대로 넣음. **함수 호출 안 됨** |
| 키 있음 | `f(기존값, value)` 결과를 넣음 |
| 함수 결과가 null | **엔트리 삭제** (put엔 없는 동작) |

**반환값이 반대다.** `put`은 *이전 값*(없으면 null), `merge`는 *새로 저장된 값*을 돌려준다.

null 반환으로 삭제까지 되므로 "차감하다 0이면 키 제거"가 한 줄:

```java
// 완주하지 못한 선수를 "맵 빌 때까지 차감"으로 풀 때
map.merge(name, -1, (old, d) -> old + d == 0 ? null : old + d);
```

**merge vs compute 계열 — 언제 뭘 쓰나:**

| 메서드 | 쓰는 상황 | 예 |
|---|---|---|
| `merge` | 기본값이 명확한 누적 (카운트) | `merge(k, 1, Integer::sum)` |
| `computeIfAbsent` | 없을 때만 초기화, 주로 컬렉션 그룹핑 | `computeIfAbsent(k, x -> new ArrayList<>()).add(v)` |
| `compute` | 기존값·신규 모두 함수로 계산 | 조건부 갱신 |
| `getOrDefault` | 읽기만, 맵 수정 안 함 | `getOrDefault(k, 0)` |

**주의 — merge는 value나 함수가 null이면 즉시 NPE.** `put`은 둘 다 허용(HashMap 기준).

### 중복 판정 한 줄

```java
Set<String> seen = new HashSet<>();
if (!seen.add(x)) { /* 이미 있었음 */ }
```

`add`는 **새로 들어갔을 때만 true**를 반환한다. `contains` + `add` 두 번 호출할 필요 없다.

### 값 기준 정렬 (베스트앨범 유형)

```java
List<String> sorted = cnt.entrySet().stream()
    .sorted((a, b) -> b.getValue() - a.getValue())
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```

### 순회

```java
for (Map.Entry<String, Integer> e : cnt.entrySet()) {
    e.getKey(); e.getValue();
}
```

`keySet()` 돌면서 `get()` 하는 건 조회가 두 번이다. `entrySet()`을 쓴다.

---

## 3. 대표 문제 5개 (프로그래머스)

| # | 문제 | Lv | 핵심 |
|---|---|---|---|
| 1 | 폰켓몬 | 1 | `Set.size()` vs `N/2` 중 작은 값 |
| 2 | 완주하지 못한 선수 | 1 | 카운트 맵 만들고 완주자에서 차감, 남는 것 |
| 3 | 전화번호 목록 | 2 | 접두어 판정 |
| 4 | 위장 | 2 | 종류별 개수 → `∏(n+1) - 1` |
| 5 | 베스트앨범 | 3 | 이중 정렬 |

### 함정 정리

**2. 완주하지 못한 선수** — 동명이인이 있다. `Set`으로 풀면 틀린다. 반드시 카운트 맵.

**3. 전화번호 목록** — 정직하게 모든 쌍을 `startsWith`로 비교하면 O(N²)에 시간 초과. 두 가지 접근:
- **정렬 후 인접 비교**: 문자열 정렬하면 접두어 관계는 반드시 붙어 있다. O(N log N)
- **Set + substring**: 각 번호의 모든 접두어를 만들어 Set에 있는지 확인. 번호 길이가 짧으므로 가능

**4. 위장** — 각 종류마다 "안 입는다"는 선택지를 하나 추가해 `(개수+1)`을 전부 곱하고, 전부 안 입는 경우 1을 뺀다. 조합을 직접 세려고 하면 안 풀린다.

**5. 베스트앨범** — 정렬 기준이 3단계다. ① 장르 총 재생수 내림차순 ② 장르 내 재생수 내림차순 ③ 재생수 같으면 고유번호 오름차순. 장르별 총합 맵과 `(인덱스, 재생수)` 리스트 맵 **두 개**를 따로 유지하는 게 깔끔하다.

---

## 4. 면접 연결 — 여기가 백엔드 가산점 구간

코테는 통과가 목적이지만, 해시는 면접에서 반드시 다시 나온다. 아래는 6년차 기준으로 답할 수 있어야 하는 것들.

### HashMap 내부 구조
- 배열(버킷) + 연결리스트. 해시값으로 버킷 인덱스를 정하고, 충돌하면 같은 버킷에 이어 붙인다.
- **Java 8부터** 한 버킷의 노드가 8개를 넘고 전체 버킷이 64개 이상이면 **Red-Black Tree로 전환**. 최악의 경우 O(N) → O(log N).

### 로드팩터와 resize
- 기본 로드팩터 0.75. `size > capacity * 0.75`면 버킷 배열을 2배로 늘리고 **전체 rehash**.
- 담을 크기를 알면 `new HashMap<>(expectedSize)`로 초기 용량을 주는 게 실무 최적화 포인트.

### equals / hashCode 계약
- `equals`가 true면 `hashCode`도 반드시 같아야 한다. 역은 성립하지 않는다(충돌 허용).
- **커스텀 객체를 키로 쓰면서 둘 다 재정의하지 않으면 `get()`이 null을 반환**한다. 실무에서 실제로 터지는 버그.
- **가변 객체를 키로 쓰면 안 되는 이유**: 넣은 뒤 필드가 바뀌면 해시값이 달라져 원래 버킷을 못 찾는다. 값이 맵 안에 있는데 영원히 조회되지 않는 상태가 된다.

### HashMap vs ConcurrentHashMap
- `HashMap`은 스레드 안전하지 않다. 동시 resize 시 Java 7에선 연결리스트가 순환해 **무한 루프**, Java 8 이후에도 데이터 유실이 발생한다.
- `ConcurrentHashMap`은 버킷 단위 CAS + `synchronized`로 락 범위를 좁힌다. `Collections.synchronizedMap`(전체 락)보다 처리량이 높다.

### put vs merge의 원자성 — 동시성 카운터
```java
chm.put(k, chm.getOrDefault(k, 0) + 1);  // 깨진다: get~put 사이 끼어들면 카운트 유실
chm.merge(k, 1, Integer::sum);           // 안전: 버킷 락 안에서 3단계가 통째로 원자적
```
- `put` 자체는 원자적이지만 **읽고-계산하고-쓰는 3단계는 원자적이지 않다.** 그래서 `getOrDefault + put` 조합은 동시성에서 카운트가 샌다.
- `merge`/`compute` 계열은 해당 버킷을 잠근 채 함수를 실행하므로 그 구간이 원자적이다. 동시성 카운터엔 `merge`가 정답.
- 단, 이 락 구간에서 실행되는 함수는 **짧고 부작용이 없어야** 한다. 안에서 다른 락을 잡거나 같은 맵을 재귀적으로 건드리면 데드락.

### 8월 Redis로 이어지는 지점
Redis의 Hash 타입과 키스페이스 자체가 해시 테이블 기반이고, **점진적 rehashing**(요청마다 조금씩 옮기는 방식)으로 resize 지연을 분산시킨다. 위의 "전체 rehash 비용" 문제를 Redis가 어떻게 다르게 푸는지가 그대로 비교 소재가 된다.

---

## 5. 풀이 기록

### 완주하지 못한 선수 (Lv.1) ✅

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public String solution(String[] participant, String[] completion) {
        Map<String, Integer> map = new HashMap<>();
        for (String p : participant) {
            map.merge(p, 1, Integer::sum);
        }
        for (String c : completion) {
            map.merge(c, -1, Integer::sum);
        }
        for (Map.Entry<String, Integer> e : map.entrySet()) {
            if (e.getValue() > 0) return e.getKey();
        }
        return "";
    }
}
```

- **접근**: 참가자를 카운트 맵에 세고 → 완주자에서 차감 → 양수로 남는 키가 답.
- **함정**: 동명이인 존재. `Set`으로 풀면 틀린다. 카운트 맵이 필수.
- **복잡도**: 시간 O(N), 공간 O(N).
- **처음 짠 버전에서 고친 것**:
  - `put(k, get(k) - 1)` → `merge(k, -1, Integer::sum)` — 조회 2번 → 1번
  - `keySet()` + `get()` → `entrySet()` — 조회 중복 제거
  - 안 쓰이는 `String answer = ""` 제거
- **대안 풀이**: 두 배열 정렬 후 앞에서부터 비교, 처음 어긋나는 원소가 답. O(N log N). 면접에서 "정렬로 풀면?" 물어오면 이걸 답하고 O(N) vs O(N log N)으로 연결.

---

## 6. 다음 액션

- [x] 완주하지 못한 선수 (Lv.1) — merge·entrySet 관용구 적용
- [ ] 폰켓몬 (Lv.1 — Set.size vs N/2)
- [ ] 위장 (Lv.2 — 조합을 곱셈으로, 오늘의 1문제)
- [ ] 전화번호 목록 (Lv.2 — 시간복잡도 함정)
- [ ] 베스트앨범 (Lv.3 — 이중 정렬, 주말 집중 블록)

문제별 풀이 코드는 `backend-notes` repo에 누적 커밋.
