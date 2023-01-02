# Docker

1. DB
2. Docker
3. dockerfile 작성과 이용
4. docker compose
5. mongodb 연결
6. mongodb 접속
7. volumes
--------------

# 1. DB

크게 다음 2가지로 나뉨

- SQL
    * 테이블에 데이터를 저장
    * table간의 relation을 정의할 수 있음
    * Oracle, MySQL...
- NoSQL : collection으로 데이터를 저장하고 컬렉션은 document의 집합.
    * mongoDB, Firebase...

- ORM : 객체와 관계형데이터베이스(sql)을 연결
- ODM : 객체와 문서형데이터베이스(NoSQL)을 연결

</br></br>

# 2. Docker

- Docker의 경우 kernel과 같은 os핵심기능은 공유하므로 os전체를 새로 설치하는 가상머신을 사용하는 것보다 빠르고 가볍다
- dockerfile만 있으면 빌드시키는 것만으로도 바로 환경세팅이 가능하다

# 3. dockerfile 작성과 이용

1. dockerfile 작성방법
    - docker hub에서 원하는 환경 찾기

    - dockerfile 작성
Dockerfile
```
FROM ubuntu

RUN apt install nodejs

RUN mkdir projectFolderName

COPY ./index.js projectFolderNam 

RUN cd ./projectFolderNam

CMD node index.js
```

- copy : 작성한 파일을 가상컴퓨터에 옮기기
- RUN vs CMD : RUN은 여러번 시행 가능(build과정에 포함), CMD는 한번만 시행 가능(build과정에 포함x. 마지막위치에 사용. 계속 실행,작동이 가능한 명령을 사용)

```
> docker build .
```
```
> docker images
```
```
> docker run image_id
```

- images에서 image id를 확인하고
해당 image_id로 run시켜 실행한다.

- node modules를 생성하기 RUN yarn install을 실행하거나 외부의 nodemodules폴더를 docker내에 copy하는 방법이 있다. docker를 사용하는 모든 사용자의 환경을 통일해주기 위해 일반적으로는 전자의 방법을 사용한다.


</br></br>

2. caching 

dockerfile
```
From node:14
WORKDIR /folder/
COPY ./package.json /folder/
COPY ./yarn.lock /myfolder/
RUN yarn install

COPY ./folder/

CMD yarn dev

```
- 한 번 빌드시켜 설치된 것들은 이후에 다시 빌드시킬때 cache에서 가져온다
- 이후 수정된부분이 있으면(cache가 깨진곳) 그 지점부터 다시 시작되어야하므로 처리과정이 오래 걸릴수있다. --> cache가 깨질수있는 부분은 마지막 부분에 위치시켜야함
- yarn install을 실행하기전 package.json과 yarn.lock을 미리 copy시켜 정상적으로 install되게 함

</br></br>

3. dockerignore
- dockerignore

```
node_modules

```
- dockerignore파일내에 있는 목록은 무시하게 됨


</br></br>

4. docker내의 터미널 접근
```
> docker ps
```
```
> docker exec -it containerid /bin/bash
```

- ps로 container 정보 확인
- 해당 container에 접속해서 bash를 열어 변경 수정이 가능하게 하는 명령

</br></br>

5. 포트포워딩

```
> docker run -p docker밖에서 안으로들어오는 포트:나가는포트 imageid
```

- docker내의 express 포트번호와 연결하기 위해 포트포워딩이 필요
- 실행중인 container를 멈추고 다시 run

</br></br>

6. image 삭제

```
docker ps -a
```
```
> docker rm containerid
```
```
> docker rm `docker ps -a -q` //모든 container 삭제 
```
```
> docker rm `docker images -a -q`//모든 image삭제
```
```
> docker system prune -a    //초기화
```

- docker의 image파일은 많은 용량을 차지하므로 필요없는 image파일은 지워줄 필요가 있다
- 위는 모든 container와 삭제를 삭제하는 코드이다
- 초기화시킬 경우 stop된 모든 container는 삭제되고 image파일또한 삭제된다.

</br></br>

# 5. mongodb 연결

dockerfile
```
FROM mongo:5

```

- 백엔드서버와 db서버를 다른 docker로 만들면 포트포워딩이 필요하다.
- 하지만 docker compose를 수행하면 이 둘을 group으로 묶어 docker들 간의 포트포워딩이 필요없다!

</br></br>

docker-compose.yaml
```
version: "3.7"

services:
    mybackend:
        build:
            context: .
            dockerfile: Dockerfile
        ports:
            - 3000:3000
    mydatabase:
        build:
            context: .
            dockerfile: Dockefile.mongo
        ports:
            - 27017:27017
```

- 들여쓰기 주의
- services : 연결할 컴퓨터들
    * build : 각 컴퓨터들을 build시킨다.
        * context: dockerfile의 위치
        * dockerfile: 도커파일명
    * ports: 포트포워딩

```
> docker-compose build

```
- build
```
> docker-compose up
```
- up : 실행

```
> docker-compose stop
```
```
> docker-compose down
```
- down시키면 db데이터가 모두 날라감

</br></br>

다른 형식

docker-compose.yaml

```yaml
version: "3.7"

services:
    mybackend:
        build:
            context: .
            dockerfile: Dockerfile
        ports:
            - 3000:3000
    mydatabase:
        image: mongo:5
        ports:
            - 27017:27017
```
- 위의 db를 위한 dockerfile내용이 간단하므로 위처럼 별도의 db dockerfile을 만들지 않고 yaml파일만 위처럼 작성하여 사용할 수 있음

</br></br>

# 6. mongodb 접속

0. mongoose : 원래 명령어 대신 객체형식으로 편하게 명령을 사용할수 있도록 만든 ODM

1. mongoose 설치
```
> npm install mongoose
```

2. 설정

```js
import mongoose from 'mongoose';

```

3. 접속
```js
//db 접속 x
mongoose.connect("mongodb://localhost/db이름");
```
```js
//db 접속 o
mongoose.connect("mongodb://mydatabase:27017/db이름");
```

- be에서 localhost경로를 사용하게 되면 db조회 불가능하다.
- compose시켰다 하더라도 자신의 docker내에서 db를 찾기때문
- compose로 연결된 다른 docker의 db로 접속하기 위해서는 docker이름에 대해 접속해야함

</br></br>

4. 사용예시

- 스키마작성
```js
const BoardSchema = new mongoose.Schema({
    writer: String,
    title: String,
    contents: String
});

```

- collection 생성
```js
const Board = mongoose.model('Board', BoardSchema);
```

- 데이터 가져오기
```js

const result = await Board.find();

```
- await : 다음 로직으로 넘어가기전까지 기다림

- 데이터 insert
```js
const board = new Board({
    writer: 'input',
    title: 'input',
    contents: 'input'
});
await board.save();
```

</br></br>

# 7. volumes

- docker내부의 파일들은 로컬 컴퓨터에 있는 파일들과 기본적으로 동기화되지 않는다.
- volumes 옵션을 추가하면 local의 소스파일들과 동기화되어 수정사항을 즉각 반영하게 된다.

```yaml
version: "3.7"

services:
    mybackend:
        build:
            context: .
            dockerfile: Dockerfile
#volumes option 추가
        volumes:
            - ./index.js:/myfoler/
            index.js
            - ./email.js:/myfolder/email.js
        ports:
            - 3000:3000
    mydatabase:
        image: mongo:5
        ports:
            - 27017:27017
```
- :을 기준으로 왼쪽은 local의 파일, 오른쪽은 docker내부의 파일을 나타낸다.
- 위처럼 원하는 파일들만 골라서 연결 시키거나 폴더 자체를 선택해 동기화시키는 것도 가능하다.