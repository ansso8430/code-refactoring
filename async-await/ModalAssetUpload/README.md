1. 필요 없는 async()

```jsx
// 양식 다운로드
const assetDownloader = async () => {
  fileDownloader({
    url: "...",
    method: "get",
    data: {
      pathType: 2,
    },
  });
  loadHandler("init");
  // closeHandler();
};
```

> 생각생각생각생각

- 왜 `await`없이 `async`만 썼을까?
  - promise 코드를 직관적으로 사용하기 위해?
  - await 할 필요가 없어서 >> 완료될 때까지 기다릴 필요가 없어서 >> 다른 곳에서 동시 작업을 할 수 있어서
- fileDownloader() 함수 안에 axios .then .catch로 비동기처리 되어있었음
  - .then .catch는 await만 대체하는거 아닐까?
  - async도 대체 가능한걸까? 그래서 밖에서 async만 쓴거지 않을까?

<aside>
💡 fileDownloader() 함수 안에 then-catch는 api 통신을 axios로 쓰기 위함

</aside>

```jsx
// fileDownloader()
const downloader = () => {
    axios({
      url: `/api/v1/${url}`,
      headers: { 'Content-Type': 'application/vnd.openxmlformats' },
      responseType: 'arraybuffer',
      method: method ?? 'post',
      data: data || null,
      params: method === 'get' ? params : null,
    })
        .then((response: Any) => {
            const blob = new Blob([response.data], { type: response.headers['content-type'] });
            const filename = getFileName(response.headers['content-disposition']);
            if ((window.navigator as Any).msSaveOrOpenBlob) {
            // ie 10+
            (window.navigator as Any).msSaveOrOpenBlob(blob, filename);
            } else {
            // not ie
            let link: HTMLAnchorElement | null = document.createElement('a');
            link.href = window.URL.createObjectURL(blob);
            link.target = '_self';
            if (filename) link.download = filename;
            link.click();
            link = null;
            }

            if (setLoading) setLoading(false);
      })
      .catch((err) => {
        if (setLoading) setLoading(false);
        console.error(`파일 다운로드 => ${err}`);
      });
  downloader();
}


```

- [x] async/await로 바꿔보자!!!!!!!!

```jsx
// fileDownloader()
const downloader = async () => {

    try {
        await axios({
        url: `/api/v1/${url}`,
        headers: { 'Content-Type': 'application/vnd.openxmlformats' },
        responseType: 'arraybuffer',
        method: method ?? 'post',
        data: data || null,
        params: method === 'get' ? params : null,
        })
        if(response) {
            const blob = new Blob([response.data], { type: response.headers['content-type'] });
            const filename = getFileName(response.headers['content-disposition']);
            if ((window.navigator as Any).msSaveOrOpenBlob) {
            // ie 10+
            (window.navigator as Any).msSaveOrOpenBlob(blob, filename);
            } else {
            // not ie
            let link: HTMLAnchorElement | null = document.createElement('a');
            link.href = window.URL.createObjectURL(blob);
            link.target = '_self';
            if (filename) link.download = filename;
            link.click();
            link = null;
            }

        if (setLoading) setLoading(false);
      }
	}
    catch(err) {
        if (setLoading) setLoading(false);
        console.error(`파일 다운로드 => ${err}`);
    };

  downloader();
};
```

### 결론 : 필요 없는 async() 였다.
