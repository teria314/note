# FileUpload

1. 개요
2. Promise 
--------------

# 1. 개요

- DB의 file테이블에 file자체를 저장하기보다는 storage를 제공하는 cloud업체를 이용하여 파일을 저장하고 이 파일은 url로 접근하여 다운 받을 수 있다.

# 2. Promise

```ts

func1 = async () => {
    new Promise((resolve, reject) = > {

        setTimeout(()=>{
            try{
                resolve("성공시 then의 파라미터로 넘어가는 데이터")
            }catch(error){
                reject("실패");
            }

        }).then((res) =>{
            console.log("위부분이 모두 실행되고 나서야 then이 실행")
        }, 2000);   //2초 count
    })

    console.log("promise처리가 끝나지 않음에도 실행될수있음");
}

func1();

```

promise.all
```ts

func2 = async () => {
    await Promise.all([
        new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve("success");
            },2000);
        }),
        new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve("success");
            },2000);
        }),
        new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve("success");
            },2000);
        }),

    ]);

    console.log("Promise가 모두 끝나고 실행");
}


```

</br></br>

# 3. 파일 1개 업로드

1. 파일구조
    * file
        * file.module.ts
        * file.resolver.ts
        * file.service.ts


file.module.ts
```ts
@Module({
    provider: [
        FileResolver,
        FileService,
    ]
})
export calss FileModule{}

```

file.resolver.ts
```ts
....
import {FileUpload, GraphQLUpload} from 'graphql-upload';


@Resolver()
export calss FileResolver{

    constructor(
        private readonly fileService: FileService,
    ){}

    @Mutation(()=> [String])
    uploadFile(
        // @Args() 파일 받기
        @Args("files", type () => [GraphQLUpload]) files: FileUpload[],
    ){

        return this.fileService.upload({files});
    }

}

```

- graphql전용 파일업로드 모듈 사용 : 'graphql-upload'
- graphql에서 받을때의 타입 : GraphQlUpload,
받고 난 이후의 타입 : FileUpload사용


file.service.ts
```ts

@Injectable()
export calss FileService{

    async upload({files}){
        const myfile = file[0];

    //storage에 파일 업로드
        const storage = new Storage({
            projectId: '프로젝트id', //google cloud storage의 프로젝트아이디
            keyFilename: '키파일이름' //카파일이름을 알고 있어야 폴더에 파일 저장가능
        }).bucket("폴더명").file(myfile.filename);

        myfile.createReadStream().pipe(storage);    //파일을 readstream으로 읽어내고 읽은 결과를 pipe()로 storage에 업로드

        await new Promise((resolve, reject)=>{
            myfile
                .createReadStream()
                .pipe(storage.createWriteStream())
                .on('finish', ()=>resolve())   //업로드 성공시 실행할 함수. resolve가 실행될때까지 await에 걸려서 그 뒤의 코드는 실행되지 않아 바로 리턴x
                .on('error', () => reject()); //업로드 실패시 실행할 함수
        
        });

        return ['url'];

    }


}

```
- google storage를 사용하기 위해 추가 library 생성. > npm install @google-cloud/storage
- myfile()에는 await를 붙이는 것이 불가능하고 Promise에는 가능하므로 위와 같이 작성

</br></br>

# 4. 복수의 파일 업로드

file.service.ts
```ts

@Injectable()
export calss FileService{

    async upload({files}){
        const waitedFiles = await promise.all(files);

    //storage에 파일 업로드
        const storage = new Storage({
            projectId: '프로젝트id', //google cloud storage의 프로젝트아이디
            keyFilename: '키파일이름' //카파일이름을 알고 있어야 폴더에 파일 저장가능
        }).bucket("폴더명");

        myfile.createReadStream().pipe(storage);    //파일을 readstream으로 읽어내고 읽은 결과를 pipe()로 storage에 업로드

        const result = await Promise.all(
            waitedFiles.map((el)=>{
            return new Promise((resolve,reject) => {
                el
                    .createReadStream()
                    .pipe(storage.file(el.filename).createWriteStream())    //filename이 각각의 파일마다 다르므로 여기서 파일이름지정
                    .on('finish', ()=>resolve(`폴더명/${el.filename}`))
                    .on('error', () => reject()); 
            
            });
        })

        );

        return result;  //url이 리턴되어 전달

    }


}

```