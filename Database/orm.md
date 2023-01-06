# Entity

1. ERD생성
2. Entity 생성
3. mysql db CRUD
4. relation 등록
---------------

# 1. ERD생성

- ERDcloud : 테이블들과 테이블들의 관계를 가시적으로 볼 수 있게 함

</br>

- 식별관계로연결 : Column 여러개가 PK로 엮인 관계 (N : M관계)
- 비식별관계로 연결

</br></br>

# 2. Entity 구현

1. 1 : 1관계 예시

product.entity.ts

```ts
import {Column, Entity, PrimaryGeneratedColumn} from 'typeorm'

@Entity
export class Product {
    @PrimaryGeneratedColumn("uuid")
    id: string

    @Column(type: "varchar")    //string number외의 구체적 타입 명시
    name: string

    @Column()
    description: string

    @Column()
    price: number

    @Column()
    isSoldout: boolean

    @JoinColumn()   //연결명시
    @OneToOne(() => ProductSaleslocation)   // 1 : 1관계로 연결할 외부 entity
    productSaleslocation: ProductSaleslocation;//타입지정
}

```
</br>

productSaleslocation.ts
```ts
import {Column, Entity, PrimaryGeneratedColumn} from 'typeorm'

@Entity
export class ProductSaleslocation {
    @PrimaryGeneratedColumn("")
    id: string

    @Column()
    name: string

    @Column()
    description: string

    @Column()
    price: number

    @Column()
    isSoldout: boolean
}

```
</br></br>

2. 1 : N관계 예시

product.entity.ts

```ts
import {Column, Entity, PrimaryGeneratedColumn} from 'typeorm'

@Entity
export class Product {
    @PrimaryGeneratedColumn("uuid")
    id: string

    @Column(type: "varchar")    //string number외의 구체적 타입 명시
    name: string

    @Column()
    description: string

    @Column()
    price: number

    @Column()
    isSoldout: boolean

    @JoinColumn()   //연결명시
    @OneToOne(() => ProductSaleslocation)   // 1 : 1관계로 연결할 외부 entity
    productSaleslocation: ProductSaleslocation;//타입지정

    @ManyToOne(()=> Product)    // N:1 관계로 연결할 외부 entity. 기준은 작성한 곳의 entity가 첫번째임(many가 현재 entity임)
    productCategory: ProductCategory;

}

```
</br>

productCategory.entity.ts
```ts

@Entity()
export class ProductCategory{
    @PrimaryGeneratedColumn('uuid')
    id : string

    @Column({unique: true}) //해당 column에 중복된 값을 허용하지 않음
    name: string
}

```

</br></br>

3. N : M 관계 예시

product.entity.ts

```ts
import {Column, Entity, PrimaryGeneratedColumn} from 'typeorm'

@Entity
export class Product {
    @PrimaryGeneratedColumn("uuid")
    id: string

    @Column(type: "varchar")    //string number외의 구체적 타입 명시
    name: string

    @Column()
    description: string

    @Column()
    price: number

    @Column({default : false})  //default값 지정
    isSoldout: boolean

    @JoinColumn()   //연결명시
    @OneToOne(() => ProductSaleslocation)   // 1 : 1관계로 연결할 외부 entity
    productSaleslocation: ProductSaleslocation;//타입지정

    @ManyToOne(()=> ProductCategory)    // N:1 관계로 연결할 외부 entity. 기준은 작성한 곳의 entity가 첫번째임(many가 현재 entity임)
    productCategory: ProductCategory;

    @JoinTable()    //상대쪽이나 여기나 둘중아무곳에 작성
    @ManyToMany(()=> ProductTag, (productTags)  => productTags.products )   // N : M관계로 연결할 외부 entity, 외부에서 이 entity를 접근할때 어떻게 접근하는지
    productTags: ProductTag[]

}

```
</br>

productTag.entity.ts
```ts

@Entity()
export class ProductTag{
    @PrimaryGeneratedColumn('uuid')
    id : string

    @Column()
    field : string

    @ManyToMany(()=> Product, (products)  => products.productTags ) //상호적으로 작성해줘야함
    products: Product[]
}

```

</br></br>

4. 생성한 entity들을 mysql에 전달

app.modules.ts
```ts

@Modules({
    ....
    TypeOrmModule.forRoot({
        ....

        entites: [__dirname + "/apis/**/*.entity.*"],  //entity.*(ts는 나중에 javascript로 바뀌기 때문에..)로 끝나는 모든 파일을 등록. **은 하위폴더를 포함한 모든폴더를 탐색한다는 의미

        ....
    })
})

```

</br></br>

# 3. mysql db CRUD

- 파일구조
    * product
        * product.entity.ts
        * product.module.ts
        * product.resolver.ts
        * product.service.ts
    * productsCategory
        * entities
            * productCategory.entity.ts
        * productCategory.module.ts
        * productCategory.resolver.ts
        * productCategory.service.ts
    * productsSaleslocation
        * productsSaleslocation.entity.ts
    * productTags
        * productTags.entity.ts
    * users
        * users.entity.ts



1. insert

productCategory.module.ts
```ts
import {Module} from "@nestjs/common"
import { TypeOrmModule } from '@nestjs/typeorm';
import { ProductCategory } from './entities/productCategory.entity';
import { ProductCategoryResolver } from './productCategory.resolver';
import { ProductCategoryService } from './productCategory.service';


@Module({
    providers: [
        imports: [TypeOrmModule.forFeature(ProductCategory)],   //()안에 resolver와 service에서 사용할 테이블들을 모두 포함시킴

        ProductCategoryResolver,
        ProductCategoryService, //resolver와 service에서의 생성자로 주입시킬 객체들을 여기서 제공한다.
    ]
})
export class ProductCategoryModule {}

```
</br></br>

productCategory.resolver.ts
```ts
import {Mutation, Resolver} from "@nestjs/graphql";

@Resolver()
export class ProductCategoryResolver {
    constructor(private readonly productCategoryService: ProductCategoryService){
    }// private readonly를 사용하면 this.productCategoryservice = productCategoryService 코드를 생략가능


    @Mutation(()=> ProductCategory) // ProductCategory 타입을 return
    createProductCategory(@Args('name') name: string){//DB에 카테고리 등록
        return this.productCategoryService.create({name});
    }

}


```
- 일반적으로 mutation에 대한 return값은 변경된 결과에 대한 객체로 한다.

</br></br>

productCategory.service.ts
```ts

@Injectable()
export class ProductCategoryService{
    constructor(
        @InjectRepository(ProductCategory)
        private readonly productCategoryRepository: Repository<ProductCategory>){}//db 주입받기. repository : db타입이라고 생각하면됨

    async create({name}){//데이터가 저장될때까지 기다리게 하기 위해서 async와 await를 붙임
        const result = await this.productCategoryRepository.save({
            name: name, //name(1)이라는 키 값에 전달받은 name(2)을 저장. key와 value가 같으므로 축약해서 그냥 name만 써도 됨
        });
        return result;
    }

}

```
</br></br>

productCategory.entity.ts
```ts

@Entity()
@ObjectType()//input type : 입력할때 들어가는 부분. objecttype : return으로 받을때
export class ProductCategory{
    @PrimaryGeneratedColumn('uuid')
    @Field(() => String )   //graphQL을 위한 타입정의.만약 배열로 정의 하고 싶으면 [String]이런식으로 작성
    id : string

    @Column()
    @Field(()=> String)//graphQL을 위한 타입정의
    name: string
}

```

</br></br>

app.module.ts

```ts

@Module({

    imports: [
        ....

        ProductCategoryModule,

        ...
    ]

})


```
- 새로 만든 모듈이 있으면 app. module에서 이를 반영하는 코드를 추가하여 합쳐주는 거 잊지 말기

</br></br>

2. read

product.resolver.ts
```ts

@Resolver()
export class ProductResolver{
    constructor(private readonly productService: ProductService){}

    @Query(() => [Product])    //grapQL의 Query로 import할것!
    fetchProducts(){
        this.productService.findAll();
    }

    @Query(() => Product)
    fetchProduct(@Args("productId") productId: string){ //Args : 브라우저에서 넘긴 값
        this.productService.findOne({productId});
    }

}

```

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
    ){}

    async findAll(){
       return await this.productRepository.find();
    }

    async findOne({productId}){
        return await this.productRepository.findOne({where: {id:productId}});
    }


}


```

</br></br>

3. update

products > dto > updateProduct.input.ts

```ts

@InputType()
export class UpdateProductInput{

    @Field(() => String, {nullable: true})  //{nullable: true} : 반드시 수정해야 할 필요는 없는 column. (playground에서 !옵션을 추가해주는 효과
    name: string;

    @Field(() => String, {nullable: true})
    description: string

    @Field(()=>Int, {nullable: true})
    price: number;

}


```

product.resolver.ts
```ts

@Resolver()
export class ProductResolver{
    constructor(private readonly productService: ProductService){}

    @Mutation(()=> Product)
    async updateProduct(
        @Args("productId") productId: string,
        @Args("updateProductInput") updateProductInput: UpdateProductInput
    ){

        // 처리될때까지 기다려야 되므로
        await this.productService.checkSoldout({productId});

        return this.productService.update(productId, updateProductInput);
    }
}

```

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
    ){}

    async update(productId, updateProductInput){
        return await this.productRepository.save({
            id: ProductId,
            ...updateProductInput,  //하나하나 지정하기 귀찮으니까 이와 같이 spread시켜 한번에 대입할 수 있음
        });//변경된 attribute만 리턴됨에 주의. 해당 객체에 대한 모든 데이터를 받아오고 싶다면 일치하는 id를 한번 조회한후 save()파라미터 내에 모든 값을 spread(...fetechedproduct)시키는 코드를 추가. 중복된 키는 마지막에 대입한 값으로 덮어쓰기 된다!
    }

    async checkSoldout({productId}){

        const product = await this.productRepository.findOne({
            where: {id: productId},
        });

        if(product.isSoldout){
            throw new HttpException("이미 판매 완료되어 수정될 수 없음", HttpStatus.UNPROCESSABLE_ENTITY);  //이렇게 쓰면상태코드 안외워도 됨
        }

    }
}

```
- 별해 : exception부분은 HttpExceptionFilter를 사용하면 모든 지점에 try catch문을 번거롭게 사용하지 않아도 된다.

</br></br>

4. delete

product.resolver.ts
```ts

@Resolver()
export class ProductResolver{
    constructor(private readonly productService: ProductService){}

    @Mutation(()=>Boolean)
    deleteProduct(
        @Args("productId") productId: string
    ){
        return this.productService.delete();
    }
}

```

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
    ){}

    async update(productId, updateProductInput){
        return await this.productRepository.save({
            id: ProductId,
            ...updateProductInput,  //하나하나 지정하기 귀찮으니까 이와 같이 spread시켜 한번에 대입할 수 있음
        });//변경된 attribute만 리턴됨에 주의. 해당 객체에 대한 모든 데이터를 받아오고 싶다면 일치하는 id를 한번 조회한후 save()파라미터 내에 모든 값을 spread(...fetechedproduct)시키는 코드를 추가. 중복된 키는 마지막에 대입한 값으로 덮어쓰기 된다!
    }

    async checkSoldout({productId}){

        const product = await this.productRepository.findOne({
            where: {id: productId},
        });

        if(product.isSoldout){
            throw new HttpException("이미 판매 완료되어 수정될 수 없음", HttpStatus.UNPROCESSABLE_ENTITY);  //이렇게 쓰면상태코드 안외워도 됨
        }

    }

    async delete({productId}){
        const result = await this.productRepository.delete({id:productId});
    
        //soft delete isDeleted
        /*this.productRepository.update({id : productId}, {isDeleted: true});*/

        //soft delete deletedAt
        /*this.productRepository.update({id: productId},{deletedAt: new Date()});*/

        //soft delete softRemove(typeORM 제공.id로만 삭제가능. 위방법들과 달리 접근시 삭제여부를 일일히 확인하지 않아도 됨)
        /*
        this.productRepository.softRemove({id: productId})
        */
        //soft delete (typeORM 제공. id외에도 삭제가능)
        /*
        this.productRepository.softDelete({id : productId})
        */

        return result.affected ? true : false;
    }
}

```

- 표면적으로만 사용자에게 보이지 않도록 하고 실제로는 delete를 수행하지 않는 경우가 많음(soft delete)

</br></br>

# 4. relation 등록

1. 1:1 relation 등록

productSaleslocation.input.ts
```ts

@InputType()
export class CreateProductInput{

    @Field(() => ProductSaleslocationInput)
    productSaleslocation: ProductSaleslocationInput

}
```

```ts

@InputType()
export class ProductSaleslocationInput extends omitType(ProductSaleslocation, ["id"], InputType){//productSaleslocation class에서 id를 제외하고 이것을 InputType으로 받겠다

    // @Field(()=>String)
    // address: string;

    // @Field(()=>String)
    // addressDetail: string;

    // @Field(() => Float)
    // lat: number

    // @Field(()=>Float)
    // lng: number

    // @Field(()=>Date)
    // meatingTime : Date
}

```

- relation은 db가 아니므로 grapql type만 적으면 된다.
- productSaleslocation의 id는 빼고 작성

</br></br>

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
        @InjectRepository(ProductSaleslocation)
        private readonly productSaleslocationRepository: Repository<ProductSaleslocation>,
    ){}

    //상품과 상품거래위치를 같이 등록
    async create({createProductInput}){
        
        const {productSaleslocation, ...product} = createProductInput;

        /*const productSaleslocation = createProductInput.productSaleslocation
        const product = {
            name: createProductInput.name
            description: createProductInput.description
            price: createProductInput.price
        };*/

        //주소테이블
        const result = await this.productSaleslocationRepository.save({
            ...productSaleslocation,
        });

        //상품테이블에저장
        const result2 = awaitthis.productRepository.save({
            ...product,
            productSaleslocation: result,//location의 단독 id가 아닌 전체를 넣거나, id만 단독으로 넣어도 상관없음 브라우저에서 원하는 반환값에 따라 선택.
        });

        return result2;
    }
    
}

```

</br></br>

2. 1:N relation 등록

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
        @InjectRepository(ProductSaleslocation)
        private readonly productSaleslocationRepository: Repository<ProductSaleslocation>,
    ){}

    async findAll(){
       return await this.productRepository.find({relations: ['productSaleslocation']}); //relations: productSaleslocation : location 과 join시켜 가져올 수 있다.
    }

    async findOne({productId}){
        return await this.productRepository.findOne({where: {id:productId}});
    }



    //상품과 상품거래위치를 같이 등록
    async create({createProductInput}){
        
        const {productSaleslocation, productCategoryId, ...product} = createProductInput;

        /*const productSaleslocation = createProductInput.productSaleslocation
        const product = {
            name: createProductInput.name
            description: createProductInput.description
            price: createProductInput.price
        };*/

        //주소테이블
        const result = await this.productSaleslocationRepository.save({
            ...productSaleslocation,
        });

        //상품테이블에저장
        const result2 = awaitthis.productRepository.save({
            ...product,
            productSaleslocation: result,
            productCategory: {  //category와의 relation 추가
                id: productCategoryId
            }
        });



        return result2;
    }
    
}

```
createProduct.input.ts
```ts

@InputType()
export class CreateProductInput{

    .......

    @Field(()=>String)
    productCategoryId: string
}
```
</br></br>

3. N : M relation 등록

createProduct.input.ts
```ts

@InputType()
export class CreateProductInput{

    .......

    @Field(()=> [String] )
    productTags: string[]
}
```

product.service.ts
```ts

@Injectable()
export class ProductService{
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
        @InjectRepository(ProductSaleslocation)
        private readonly productSaleslocationRepository: Repository<ProductSaleslocation>,
        @InjectRepository(ProductTag)
        ProductTagRepository: Repository<ProductTag>
    ){}

    async findAll(){
       return await this.productRepository.find({relations: ['productSaleslocation','productCategory','productTags']}); //relations: productSaleslocation : location 과 join시켜 가져올 수 있다.
    }

    async findOne({productId}){
        return await this.productRepository.findOne({where: {id:productId}});
    }



    //상품과 상품거래위치를 같이 등록
    async create({createProductInput}){
        
        const {productSaleslocation, productCategoryId,productTags, ...product} = createProductInput;

        /*const productSaleslocation = createProductInput.productSaleslocation
        const product = {
            name: createProductInput.name
            description: createProductInput.description
            price: createProductInput.price
        };*/

        //주소테이블
        const result = await this.productSaleslocationRepository.save({
            ...productSaleslocation,
        });
        const result2 = []
        for(let i = 0; i<productTags.legnth; i++){
            const tagname = productTags[i].replace("#",""); 

            //이미 등록된 태그인지 확인
            const prevTag = this.productTagRepository.findOne({name: tagname});

            if(prevTag){
                result2.push(prevTag);
            }else{        
                const newTag = await this.productTagRepository.save({name: tagname});
                result2.push(newTag);
            }

        }

        //상품테이블에저장
        const result3 = await this.productRepository.save({
            ...product,
            productSaleslocation: result,
            productCategory: {  //category와의 relation 추가
                id: productCategoryId
            }
            productTags: result2;
        });



        return result2;
    }
    
}

```