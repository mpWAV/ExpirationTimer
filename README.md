# ExpirationTimer


## CMake
```CMake
include(lib/ExpirationTimer/chrono/exp_chrono.cmake)
```

## Usage  
+ NOTE : only works on ```Release``` mode

```cpp
if (is_expired(20230328_issued)) {
      printf("Program Expired!! Please contact contact the distributor for further information.");
      exit(-1);
  }
```

# 포크 메모 (mpWAV)

이 레포는 [kooBH/ExpirationTimer](https://github.com/kooBH/ExpirationTimer) 의
포크다. upstream 에 푸시 권한이 없어서 2026-10-27 에 갈라놨다.

+ 갈라진 지점 : `1b24b58`
+ 작업 브랜치 : `ko`
+ 전체 차이 보기 : `git diff 1b24b58..ko`
upstream 과 다른 점은 `chrono/exp_chrono.cpp` 의 한 줄뿐이다 (`59ecf45`).

## `is_expired()` 의 off-by-one 수정

```diff
-  return duration_cast< days >( system_clock::now() - issued_time ) > days { 1 };
+  return duration_cast< days >( system_clock::now() - issued_time ) >= days { 1 };
```

`duration_cast< days >` 는 내림이다. 그래서 `> days{1}` 은 경과일이 2일
이상이어야 참이 되고, 만료가 발급일 **+2일**이었다. `>= days{1}` 로 바꾸면
의도대로 발급일 **+1일** 00:00 에 만료된다.

`20261025_issued` 로 빌드한 경우 (`operator""_issued` 는 `%Y%m%d` 만 읽으므로
기준 시각은 그 날 **로컬 00:00** 이다) :

| 시점 | 경과 | `duration_cast<days>` | `> 1` (이전) | `>= 1` (현재) |
|---|---|---|---|---|
| 10-25 12:00 | 0.5일 | 0 | 유효 | 유효 |
| 10-26 00:00 | 1일 | 1 | 유효 | **만료** |
| 10-27 00:00 | 2일 | 2 | **만료** | 만료 |

## upstream 과 다시 합칠 때

`origin/main` 이 `1b24b58` 보다 두 커밋 앞서 있다 (2026-10-27 기준) :

```
ea4fe47 APPLE UUID
746a986 Merge branch 'main' of https://github.com/koobh/ExpirationTimer into HEAD
```

아직 가져오지 않았다. `chrono/exp_chrono.cpp` 를 건드리지 않는지 확인하고
가져올 것.

이 한 줄은 upstream 도 받아야 할 버그 수정이므로, 권한이 생기면 PR 로 올려서
포크를 없애는 쪽이 낫다.
