## middleWare?

action과 reducer 중간 사이

![images_cyongchoi_post_2ea52982-cab7-4b75-80da-d4a848b56dd9_1_2YaEymkor_mUY1Fcaljw3Q.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eaff0af7-e1a1-4ac7-80d5-887c7a6f98a3/images_cyongchoi_post_2ea52982-cab7-4b75-80da-d4a848b56dd9_1_2YaEymkor_mUY1Fcaljw3Q.png)

- 전달 받은 액션을 콘솔에 기록
- 전달 받은 액션을 취소, 다른 종류의 액션을 추가로 dispatch(디스패치) 할 수도 있음
  ![210927_fin_05-1.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b589a875-2728-4d3b-a541-094eb346426f/210927_fin_05-1.jpg)
  ![images_cyongchoi_post_a9ba8061-23a1-4f42-870e-4e52e4880fd5_reduxMiddleware.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e76fd170-9d63-4e65-bb2f-f2cac8d7dfbe/images_cyongchoi_post_a9ba8061-23a1-4f42-870e-4e52e4880fd5_reduxMiddleware.png)
- 함수를 반환

```jsx
// 각각의 미들웨어는 next 함수에 액션을 담아서
// 리듀서까지 전파해 나갑니다.

// 아래는 커스텀 미들웨어를 작성한 코드입니다.

const customMiddleware = (storeApi) => {
  return (next) => {
    return (action) => {
      // 개발자는 이곳에 자신의 목적에 알맞은 코드를 추가할 수 있습니다.
      // ...

      return next(action);
    };
  };
};

const store = configureStore({
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(customMiddleware),
});
```

```jsx
const store = configureStore({
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ serializableCheck: false }),
});
```

- 직렬화 : 데이터를 다른 데이터 구조로 맞추어 가공하는 행위

      가 불가능한 값(non-seralizable value)을 사용하는 실수를 방지할 수 있도록 경고함

- 직렬화 불가능 >> 어떤 데이터를 직렬화 하는 과정에서 데이터를 유실할 수 있다.
- 직렬화가 불가능한 값 : Promise, Symbol, Map/Set, function, class instance

```jsx
// { it: 'has data' } 객체를 JSON 데이터 포맷으로 직렬화합니다.

const stringifiedObject = JSON.stringify({ it: "has data" }); // {"it":"has data"}

// 이것을 다시 역직렬화를 하면
// 처음 내용과 동일한 데이터를 담고 있음을 확인할 수 있습니다.

JSON.parse(stringifiedObject); // { it: 'has data' }

// 그러나 아래처럼 Set 자료형을 직렬화 한다면
// 그 과정에서 데이터가 유실됩니다.

const setObject = new Set([1, 2, 3]); // Set(3) {1, 2, 3}
const stringifiedSetObject = JSON.stringify(setObject); // {}

// 역직렬화해도 데이터가 이미 유실된 상태이기 때문에
// 본래 값과 차이가 발생하여 앱이 기대하지 않은 방식으로 동작할 수 있습니다.

JSON.parse(stringifiedSetObject); // {}
```
