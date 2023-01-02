# GraphQL

1. end-system간의 데이터 전송
2. API
3. JSON
4. CRUD
5. GraphQL 사용 예시
---------------

# 1. 컴퓨터 간 데이터 전송

- 데이터 전송 프로토콜
    * FTP : 파일전송
    * SMTP : 메일전송
    * HTTP : 텍스트/html

- HTTP 데이터 전송
    1. FE --data(request)--> BE
    2. FE <--data(response)-- BE

    * response에는 상태코드가 같이 전송된다.
    * 상태코드 : 성공(2xx), Front-end 에러(4xx), Back-end 에러(5xx)

</br></br>

# 2. API

- API : 특정 기능을 가진 BE의 함수로 다음 2가지 종류가 있다
    * REST
        * 주소 형식
        * BE에서 전송한 모든 데이터를 받아야 하므로 원치않는 데이터까지 모두 받아야 한다.
    * GraphQL
        * 함수 형식
        * FE에서 원하는 데이터만 요청할때 BE에서 해당 데이터만 전송 --> 데이터양이 적은만큼 비용 절감
        * 사용자가 많은 경우에 유리

</br></br>

# 3. JSON

- JSON : javascript를 객체처럼 표기

- response와 request에는 모두 header와 body가 포함되어있다.
    * 헤더 : 데이터에 대한 요약정보
    * body : json타입

</br></br>

# 4. CRUD

- API를 분류하는 기준 중 하나로 다음 4가지타입이 존재
- create, read, update, delete
- axios
    * post : create
    * put : update
    * delete : delete
    * get : read
- apollo-client
    * mutation : create, update, delete
    * query : read

</br></br>

# 5. GraphQL 사용 예시

```js

const typeDefs = gql`
    input CreateBoardInput {
        writer : String
        title : String
        contents: String
    }

    type BoardReturn {
        number: Int
        writer : String
        title: String
        contents: String
    }

    type Query {
        fetchBoards: [BoardReturn]
    }

    type Mutation {
        createBoard(writer: String, title: String, contents: String): String
    }
`;

const resolvers = {
    Query: {
        fetchBoards: () => {
            const result = [
                ........
            ];
            return result;
        }
    },

    Mutation: {
        //args : fe에서 호출, parent : 다른 api에서 호출
        createBoard: (parent, args) => {
            console.log(args)
        }

    }



}

```

- input vs type : fe에서 받아오는 경우 type을 정의할때 input을 사용하고 be에서 내보낼때 type은 type으로 사용

</br>

요청쿼리예시
```
query{
      user(user_no:1){
            user_name  
      }
}

```