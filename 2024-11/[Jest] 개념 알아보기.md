# [Jest] 개념 알아보기

## 기본 문법
```js
describe('테스트 그룹 설명', () => {
  test('개별 테스트 설명', () => {
    expect(실제값).matcher(기대값);
  });
});
```

## 구성 요소
### `describe`
> 테스트 그룹을 정의
```js
describe('계산기 테스트', () => {
  // 여러 관련 테스트들을 그룹화
});
```
### `test` / `it`
> 개별 테스트를 정의
```js
test('2더하기 2는 4이다', () => {
  expect(2 + 2).toBe(4);
});

it('2더하기 2는 4이다', () => {
  expect(2 + 2).toBe(4);
});
```

## 자주 사용되는 matcher
```js
test('다양한 matcher 예시', () => {
  // 정확한 값 비교
  expect(2 + 2).toBe(4);
  
  // 객체나 배열의 값 비교
  expect({name: 'jest'}).toEqual({name: 'jest'});
  
  // 참/거짓 확인
  expect(true).toBeTruthy();
  expect(false).toBeFalsy();
  
  // null, undefined 체크
  expect(null).toBeNull();
  expect(undefined).toBeUndefined();
  
  // 배열/문자열 포함 여부
  expect([1, 2, 3]).toContain(2);
  expect('Hello Jest').toContain('Jest');
});
```

## 테스트 라이프 사이클 훅
```js
describe('테스트 라이프사이클 예시', () => {
  beforeAll(() => {
    // 모든 테스트 전에 1번 실행
  });

  afterAll(() => {
    // 모든 테스트 후에 1번 실행
  });

  beforeEach(() => {
    // 각 테스트 전에 실행
  });

  afterEach(() => {
    // 각 테스트 후에 실행
  });

  test('테스트 1', () => {
    // 테스트 코드
  });
});
```