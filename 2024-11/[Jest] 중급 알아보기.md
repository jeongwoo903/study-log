# [Jest] 중급 알아보기

## mock 이란?
> 실제 객체를 대체하는 가짜 객체

### 주요 목적
1. 외부 의존성 제거
2. 테스트 속도 향상
3. 예측 가능한 테스트 환경 구성
4. 특정 상황 시뮬레이션

```js
// 예시: 실제 API 호출 vs Mock
// 실제 API 호출 (테스트하기 어려움)
const realApiCall = async () => {
  const response = await fetch('https://api.example.com/data');
  return response.json();
};

// Mock을 사용한 테스트 (제어 가능하고 예측 가능함)
test('API 호출 테스트', () => {
  const mockFetch = jest.fn().mockResolvedValue({
    json: () => Promise.resolve({ data: 'test' })
  });
  global.fetch = mockFetch;
  
  // 이제 테스트가 실제 API에 의존하지 않음
});
```

## 목(Mock) 함수 사용법
```js
describe('Mock 함수 기본', () => {
  test('jest.fn() 기본 사용법', () => {
    // 기본 mock 함수 생성
    const mockFn = jest.fn();
    
    // 반환값 설정
    mockFn.mockReturnValue('기본값');
    
    // 한 번만 특정 값 반환
    mockFn.mockReturnValueOnce('첫번째 호출');
    
    // 구현 추가
    const mockFnWithImpl = jest.fn(() => 'custom return');
    
    // 비동기 mock
    const asyncMock = jest.fn().mockResolvedValue('async value');
    
    // 결과 테스트
    expect(mockFn()).toBe('첫번째 호출');
    expect(mockFn()).toBe('기본값');
    expect(mockFnWithImpl()).toBe('custom return');
  });

  test('mock 함수 추적', () => {
    const mockFn = jest.fn();
    
    mockFn('arg1', 'arg2');
    mockFn('arg3', 'arg4');

    // 호출 횟수 확인
    expect(mockFn).toHaveBeenCalledTimes(2);
    
    // 특정 인자로 호출됐는지 확인
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
    
    // 마지막 호출 인자 확인
    expect(mockFn).toHaveBeenLastCalledWith('arg3', 'arg4');
    
    // 모든 호출 정보 확인
    console.log(mockFn.mock.calls); // 호출된 인자들의 배열
    console.log(mockFn.mock.results); // 반환값들의 배열
  });
});
```

## 스파이(spy) 이란?
> 실제 객체를 복제해 감시(추적)하는 테스트 도구

```js
// math.js
export const math = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

// math.test.js
import { math } from './math';

describe('Spy 테스트', () => {
  test('math.add 함수 스파이', () => {
    // 특정 메소드만 스파이
    const addSpy = jest.spyOn(math, 'add');
    
    math.add(1, 2);
    
    expect(addSpy).toHaveBeenCalledWith(1, 2);
    expect(addSpy).toHaveBeenCalledTimes(1);
    
    // 스파이 제거
    addSpy.mockRestore();
  });

  test('스파이로 구현 변경', () => {
    const addSpy = jest.spyOn(math, 'add')
      .mockImplementation(() => 'mocked');
    
    expect(math.add(1, 2)).toBe('mocked');
    
    // 원래 구현으로 복구
    addSpy.mockRestore();
    expect(math.add(1, 2)).toBe(3);
  });
});
```

### `spyOn().mockImplementation()`
> 아직 개발 되지 않은 부분의 함수를 mocking 하여 테스트 코드를 작성할 수 있다.

