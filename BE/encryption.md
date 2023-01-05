# Encryption

1. 양방향 암호화
2. 단방향 암호화
3. bcrypt

-----------------------

# 1. 양방향 암호화

- 복호화가 가능하다.

</br></br>

# 2. 단방향 암호화(hashing)

- 복호화 불가능
- 역방향으로 원본을 추론할 때 1 : 1 대응관계가 아닌 n : 1 대응관계이므로 암호화된 데이터에 대한 많은 경우가 존재한다.
- 현재 쓰이는 암호화 방식은 암호화된 데이터에 임의의 숫자(salt)를 추가하여 저장한다.

</br></br>

# 3. bcrypt

- 암호화를 처리해주는 라이브러리

1. 설치 & 설정

```
> npm install bcrypt
```
```js
bcrypt = require('bcrypt');
```

2. 사용

- 암호화
```js
const pwEncrypted = bcrypt.hash(password,10);

```
- 첫번째 파라미터 : 원본 password, 두번째 파라미터 : salt로 사용할 숫자

</br></br>

- 비교
```js
bcrypt.compare(password, pwEncrypted, function(err, result) {
    // result == true
});

```
- 첫번째 파라미터 : 입력받은 password, 두번째 파라미터 : 암호화시킨 password
- 일치여부에따라 result의 참거짓 결정