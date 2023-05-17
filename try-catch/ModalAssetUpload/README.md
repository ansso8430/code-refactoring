1. try- catch 문

```jsx
// 파일 업로드
  const onSubmit = async () => {
    setState({ type: 'loading', value: true });

    const params = { uploadFile };
    const formData = new FormData();
    const source = axios.CancelToken.source();

    // eslint-disable-next-line guard-for-in
    for (const key in params) {
      const nkey = key as keyof typeof params;
      formData.append(key, params[nkey] as string | Blob);
    }

    cancelRef.current = { cancel: source.cancel, source };

   try {
    const response = await axios({
      method: 'post',
      headers: { 'Content-Type': 'multipart/form-data' },
      data: formData,
      url: '/api/...',
      onUploadProgress: ({ loaded, total }: { loaded: number; total: number }) => {
        setState({ type: 'progress', value: Math.round((loaded * 100) / total) });
      },
      cancelToken: source.token,
    });

      if (response && response.data) {
        setState({ type: 'status', value: response.data.statusCode });

        if (status === 200) {
          if (response.data.data.isOk >= 1) {
            dispatch(utilsActions.alertSet({ success: '성공적으로 등록 되었습니다.' }));
            setState({ type: 'isOkay', value: 1 });
            loadHandler('init');
            closeHandler();
          } else {
            dispatch(utilsActions.alertSet({ fail: '파일 업로드에 실패하였습니다.' }));
            const ErrorMessage = response.data.message;
            setState({ type: 'fileError', value: ErrorMessage });
            setState({ type: 'isOkay', value: 0 });
          }
          setState({ type: 'loading', value: false });
        }
      }
    } catch (error: any) {

      if (error && error.response.data.statusCode) {
        const errorStatusCode = error.response.data.statusCode;

        if (errorStatusCode >= 400) {
          setState({ type: 'status', value: errorStatusCode });
          setState({ type: 'isOkay', value: 0 });
          dispatch(utilsActions.alertSet({ fail: '파일 업로드에 실패했습니다.' }));
          setState({ type: 'loading', value: false });
        }
      }
    }
  };
```

> 생각생각생각생각

- try -catch 문이 가드라 생각
  - axios 밖에 있는 것보다 한 단계 내려와서 가드 치는게 맞지 않을까?
- 에러처리는 무조건 catch안에서 해야하는 걸까?
  - resonse.data.statusCode인데 try안에서 조건문 처리해도 되지않을끼?

<aside>
💡 try안에서 실패한 error를 catch에서 잡을 수 있음 > try 밖에 axios가 있다면 에러가 발생해도 catch로 가지 않음 > 그냥 axios 에러인 줄 알기 때문
error파악을 위해 try 안에서 axios 실행

</aside>

### catch에서 error처리

```jsx
const response = await axios({
  method: "post",
  headers: { "Content-Type": "multipart/form-data" },
  data: formData,
  url: "/api/v1/setting/asset/excel",
  onUploadProgress: ({ loaded, total }: { loaded: number, total: number }) => {
    setState({ type: "progress", value: Math.round((loaded * 100) / total) });
  },
  cancelToken: source.token,
  validateStatus(status) {
    return false;
  },
});
```

> validateStatus

[axios](https://sangcho.tistory.com/entry/axios)

- 무조건 400 error를 catch에서 처리하는 것은 아니다.
- axios 안에 있는 기능 중 validateStatus가 이를 지정한다.
- HTTP응답 상태 코드에 대해 promise의 반환 값이 resolve 또는 reject 할지 지정

```jsx
// validateStatus의 기본값
validateStatus: function (status) {
	return status >= 200 && status < 300; //default
}
```

- 기본 값이 200번대로 처리 되어있기 때문에 catch에 300~500 번대의 error가 들어간다.
- 응답 상태 코드의 범위가 다르게 들어갈 수 있기 때문에 try, catch문안에서 statusCode로 가드를 쳐줌
