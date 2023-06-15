# async-await

### 왜 쓰는가?

async/await는 비동기 코드를 동기식 코드처럼 깔끔하고 읽기 쉬움

- promise 화 된 async-await

```tsx
async function getWikipediaHeaders() {
  if ((await stat("./headers.txt")) != null) {
    console.log("headers already collected");
  }
  const res = await get({
    host: "www.wikipedia.org",
    port: 80,
  });

  await writeFile("./headers.txt", JSON.stringify(res.headers));
  console.log("Great success!");
}
```

### async에 then vs await 쓰기, promise에 then vs await 쓰기

- `await`은 자바스크립트로 하여금 내부적으로 `then()` 함수를 호출 함
- `await` 이벤트 루프의 다음 차례까지 실행을 일시 정지 시킴
  - await vs return ???
  | promise         | await            | return                           |
  | --------------- | ---------------- | -------------------------------- |
  | async 함수 실행 | 일시정지 후 재개 | 종료하고 return된 함수 재개 안함 |

```tsx
async function test() {
  try {
    return Promise.reject(new Error("Oops!"));
  } catch (error) {
    return "ok";
  }
}

// "Oops!"가 출력됩니다.
test().then(
  (v) => console.log(v),
  (err) => console.log(err.message)
);
```

- ok를 catch 못함
- 왜 `await` 만이 error를 catch 할 수 있을까?
  - `await` 실행을 재개할 때 error를 throw 하기 때문
  - `return` async 함수 본체의 실행을 멈추고 이 async 함수의 promise에 대해 `resolve()`수행

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2347e102-18b4-4463-ac1b-8750bb8006e2/Untitled.png)

### promise가 될 수 없는 값에 `await` 사용 하지 않기

async로 만드는 유일한 이유는 실행을 일시 정지 시켜서 다른 함수를 처리 해야할 때

(대량의 배열을 처리한다고 할 때…

- promise가 아니면 그냥 return 사용

```tsx
async function fn1() {
  // try/catch 사용시 오류를 처리하지 못하는 문제
  return asyncFunction();
}

async function fn2()
  //  asyncFunction()의 오류들을 처리하고자 한다면 이 방법이 좋다.
  const ret = await asyncFunction();
  return ret;
}
```

### 오류는 .catch()로 잡기

`.catch()`가 없는 async/await 코드는 어딘가 처리되지 못할 오류가 존재하고 있음

<aside>
💡 **결론**

- Async 함수는 항상 promise를 반환
- `await`는 대상 promise가 정착될 때까지 호출한 async 함수를 일시 정지 시킴
- promise가 아닌 값에 대한 `await`은 아무런 영향이 없음
- async 함수가 일시 정지 일 때 다른 자바스크립트가 작동할 수 있음
- 오류 처리는 일상적인 문제입니다. 오류는 가능한 `.catch()`로 처리
</aside>
