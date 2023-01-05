# nest.js

1. OOP
2. 파일분리
3. MVC 패턴
4. DI/ IoC
5. nest.js
6. typescript
7. graphQL 연결
8. mySQL 연결
----------------------

# 1. OOP

1. class 정의

```js
class SkyMonster{
    power = 10
    
    constructor(p){
        this.power = p
    }

    attack = ()=>{
        console.log(`가한 공격력 ${this.power}`);
    }

    run = ()=>{
        console.log('날면서ㅌㅌ');
    }
}
class GroundMonster{
    power = 10
    
    constructor(p){
        this.power = p
    }

    attack = ()=>{
        console.log(`가한 공격력 ${this.power}`);
    }

    run = ()=>{
        console.log('뛰면서ㅌㅌ');
    }
}

const monster1 = new SkyMonster(10);
const monster2 = new GroundMonster(20);

monster1.attack();
monster2.run();

```
- 문제 : 두 class내에 중복되는 부분이 너무 많음

</br></br>

2. 상속

```js

class Monster{
    
    power = 10
    
    constructor(p){
        this.power = p
    }

    attack = ()=>{
        console.log(`가한 공격력 ${this.power}`);
    }

}


class SkyMonster extends Monster{

    consturctor(p){
        super(p);
    }

    run = ()=>{
        console.log('날면서ㅌㅌ');
    }
}
class GroundMonster extends Monster{
    
    consturctor(p){
        super(p);
    }

   
    run = ()=>{
        console.log('뛰면서ㅌㅌ');
    }
}

const monster1 = new SkyMonster(10);
const monster2 = new GroundMonster(20);

```

- 공통된 기능, 속성을 부모class에 정의하고 자식class에서 상속
- 상속받을 수 있는 부모class는 하나만 허용. (java에서처럼 마름모형태의 상속관계 비허용)

</br></br>

# 2. 파일분리

- api는 최대한 main파일이 아닌 구조적으로 다른 파일에 작성하는 것이 유지보수와 같은 면에서 좋을 것

1. 사용예시
cash.js
```js
export class CashService {
    checkValue = () => {
        .....
    }
}


```
product.js
```js

export class ProductService{
    checkSoldout = ()=>{
        ...
    }
}

```
index.js
```js

import {CashService} from './cash.js';
import {ProductService} from './product.js';

app.post("/products/buy", (req,res)=>{
    const cashService = new CashService();
    cashService.checkValue();

    const productService = new ProductService();
    productService.checkSoldout();
    ......
});

.....
```
- export : 외부에서 사용하게 할 class 앞에 붙임
- import : 외부에서 가져와 사용할 class, 함수들을 명시

</br></br>

# 3. MVC 패턴

- mvc
    * controllers : 미들웨어 함수
    * models : db생성관리, schema 정의
    * views : html파일들 (FE와 BE로 구분되지 않은 좀 옛날 방식..)

    </br></br>

1. 사용예시

    * 파일 구조
        * controllers
            * services
                * cash.js
                * product.js
            * product.controller.js
        * models
            * product.model.js

</br></br>

product.controllr.js        
```js
import {CashService} from './services/cash.js';
import {ProductService} from './services/product.js';

export class ProductController{
    buyProduct = (req,res) =>{
        const cashService = new CashService();
        cashService.checkValue();

        const productService = new ProductService();
        const isSoludout = productService.checkSoldout();
        ....
    }

    refundProduct = (req,res) => {
        ......
    }

}


```

index.js
```js
import {ProductService} from './mvc/controllers/product.controller';

const productController = new ProductController();

app.post("/products/buy", productController.buyProduct
);
app.post("/products/refund", productController.refundProduct
);

...

```

</br></br>

# 4. DI / IOC

1. DI(Dependency Injection)

- Tight-coupling : 
    * 클래스가 내부에 다른 클래스를 직접 사용하여 객체에 대해 강하게 의존하고 있음
    * 클래스 내부에서 새로운 객체 생성을 비교적 많이 하게 되고 이것은 메모리 낭비로 이어짐 

- loose-coupling :
    * 다른 클래스를 직접적으로 사용하는 의존성을 줄임(결합이 되있긴 함)

- DI(Dependency Injection)
    * Tight-coupling처럼 클래스명을 직접 명시하는 것이 아니라 외부에서 명시할수 있도록 함
    * 즉 Tight Coupling -> Loose Coupling으로 전환시킴
    * 따라서 외부모듈을 import할 필요x

</br></br>

- DI 사용예시

product.controllr.js        
```js
export class ProductController{

    moneyService;

    constructor(moneyService, productService){
        this.moneyService = moneyService;
        this.productService = productService;
    }


    buyProduct = (req,res) =>{
        //const cashService = new CashService()
        const hasMoney = this.moneyService.checkValue();

        //const productService = new ProductService();
        const isSoldout = this.productService.checkSoldout();
        ....
    }

    refundProduct = (req,res) => {
        ......
    }

}


```

index.js
```js
import {ProductService} from './mvc/controllers/product.controller';
import {cashService} from './mvc/controllers/services/cash.js';
import {ProductService} from './mvc/controllers/services/product.js';

//의존대상의 객체를 선언
const ProductService = new ProductService();
const CashService = new CashService();


const productController = new ProductController();

app.post("/products/buy", productController.buyProduct
);
app.post("/products/refund", productController.refundProduct
);

...

```
</br></br>

2. singleton pattern

- 새로운 객체를 한 번만 생성하는 것만으로도 모든 곳에서 사용가능하게 한 디자인패턴
</br></br>

3. IOC(제어의 역전)

- 의존성을 제어하는 부분을 개발자가 아닌 프레임워크가 맡는다

</br></br>

# 5. nest.js

- node.js에 규칙을 추가해 기본 구조 틀을 제공하여 통일성을 부여하여 협업에 편의성을 제공한다

</br></br>

1. 설치

```
> npm i -g @nextjs/cli
> nest new projectname

```
또는
```
> npx nest new projectname
```

- 다운 받은 명령어를 자동으로 삭제해줌

</br></br>

2. 구조

- 보일러 플레이트 : 초기 폴더구조
- (.eslintrc.js)린터 : 규칙을 정의
- 포멧터 : 코딩정리규칙 정의
- package.json : 기본 메뉴얼
    * dependencies : 실행할때 필요함
    * devdependencies : 개발할때만 필요함(배포 시 필요 x)
    * `>` yarn install --production 명령은 dependencies에 있는 것들만 설치해줌 (devdependencies는 설치x)
- src : api가 존재하는 소스코드
    * app.controller.ts
    * app.modules.ts
    * app.service.ts
    * main.ts


</br></br>

# 6. TypeScript

- 자바스크립트의 타입을 강제

1. 사용예시
```s
let s:string = 'hhh';
let n:number = 123;

interface IProfile {
    name: string
    age: number
}
let profile:IProfile = {name:"이름", age : 23};

```
</br></br>

2. 규칙
- 타입추론 : 초기화 시 type을 지정해주지 않으면 대입한 값으로 자동으로 형을 정의
- 여러개의 타입을 사용할 수 있음

```s
let var: string|number = "str";
var = 10;

let var2: boolean = true;
var2 = "false"  #""이면 false 그 외에는모두 true

let arr1: number[] = [1,2,3,4];
let arr2: (string | number)[] = [1,2,"str",3];

let prifile:IProfile = {
    name: string
    age: number
    hobby?: string
}

```

- ? : 이 요소를 필수로 하지 않음을 나타냄

```s

const add = (n1,n2,unit) => {
    return n1 + n2 + unit
};
const result = add(100,200,"일");

```
- 함수에서 받는 parameter는 타입추론이 되지 않는다. 따라서 아래 코드와 같이 타입을 명시할 것을 권장

```s
const add = (n1 : number, n2: number,unit: String): String => {
    return n1 + n2 + unit;
};
```
</br></br>

3. decorator

```ts

function zzz(para){
    console.log(para);
}

@zzz
class AppController {}  
```
- 실행될때 인자로서 들어가는 함수로 @로 명시

</br></br>

4. public, private, readonly

- public : 외부에서도 접근 가능
- private : 외부에서 접근불가능
- readonly : 내부에서도 변경 불가능

```
class Aaa {
  constructor(public mypower) {
    // this.mypower = mypower; 
  }

}


```
- parameter로 받은 변수는 자동으로 클래스 내 변수로 저장된다.

</br></br>

# 7. grahQL 연결

- nest.js 사용시 초기세팅정보는 다음 링크를 참고하자 </br>
[nest.js setting] https://github.com/nestjs/nest/tree/master/sample

- graphQL의 스키마를 먼저 정의한후 그 정의에 맞게 코드를 작성하는 기존의 Schema first방식과는 반대로 nest.js에서는 resolver만 만드는 것만으로도 스키마, docs가 만들어질 수 있는 code-first방식을 지원한다.

1. 설치

```
> npm install @nestjs/graphql @nestjs/apollo graphql apollo-server-express
```


2. 사용예시

- 기존의 controller --> resolver 로 사용

module.ts
```ts

@Module({
    imports: [],
    controllers: [],
    providers: [BoardResolver, BoardService]
})

export class BoardModule{

}

```

- 최종적인 모듈들의 결합은 app모듈에서 이루어진다.

</br></br>

resolver.ts
```ts

@Resolver()

class BoardResolver{
    constructor(private readonly boardService:BoardService){}
    
    @Query(()=> String) //getHello의 return을 string으로 자동으로 만들어준다.
    getHello(){
        return this.boardService.aaa()
    }

}


```
- query()함수 import graphql로 해야되는 거 주의

</br></br>

service.ts
```ts

@Injectable()

class BoardService{
    aaa

}

```

app.module.ts
```ts

@Module({
    import: [
        BoardModule,
        //이부분은 외우지말고 걍 docs참고ㄱㄱ
        GraphQLModule.forRoot<ApolloDriverConfig>({
            driver: ApolloDriver,
            
            //스키마를 저장할 파일 경로
            autoSchemaFile: 'src/commons/graphql/schema.gql',
        }),

    ],

})

export class AppModule{}

```

</br></br>

# 8. mySQL 연결

1. 설치

```js
> npm install --save @nestjs/typeorm typeorm@0.2 mysql2
```
- 실제 mysql이 아닌 mysql에 연결하기 위한 프로그램을 설치하는 것임

</br></br>

2. 사용

app.module.ts
```ts

@Module({
    import: [
        BoardModule,
        GraphQLModule.forRoot<ApolloDriverConfig>({
            driver: ApolloDriver,
            autoSchemaFile: 'src/commons/graphql/schema.gql',
        }),

        TypeOrmModule.forRoot({
            type: "mysql",
            host: "localhost",
            port: 3006,
            username: 'root',
            password: '~~~',
            database: '~~', //사용할 db
            entities: [Board], //테이블들
            synchronize: true,  //동기화여부
            logging: true,  //sql쿼리문으로 어떻게 변경되었는지를 log로 확인가능하게 할지

        })

    ],

})

export class AppModule{}

```
</br></br>

- 테이블작성
entities> board.entity.ts
```ts

@Entity()
class Board{
    
    @PrimaryGeneratedColumn("uuid")
    number: number

    @Column()
    writer: string

    @Column()
    title: string

    @Column()
    contents: string
}

```
- @를 달아서 table 생성과 column생성 요청
- unique한 id를 자동으로 생성하게 하고 싶다면 @PrimaryGeneratedColumn()을 추가 
    * "uuid" : 랜덤문자열, 
    * "increment" : 게시글번호와같이 1씩 증가시킨 숫자를 primary key로 함

