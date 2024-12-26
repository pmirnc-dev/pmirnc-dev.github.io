---
layout: post
title:  "Zod, React Hook Form으로 심플하게 form 작성하기"
date:   2024-12-26 13:47:00 +0900
author: ymshim
categories: js ts nextJS nestJS
---

<hr/>

# Zod, React Hook Form으로 심플하게 form 작성하기
Zod, React Hook Form 라이브러리를 사용하여 반복적이고 지루하지만<br> 
절대로 대충해서 안되는 스키마 선언과 유효성 검사를 조금 더 간결하게 해보도록 하겠습니다. 


## 1. Frontend

간단히 리액트 프로젝트를 생성하고 로그인 폼을 만들어 보겠습니다.

### 기본 SignIn Component
```tsx
export default function SignIn() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    
    const handleEmailOnChange = (e: ChangeEvent<HTMLInputElement>) => {
        setEmail(e.target.value)
    }

    const handlePasswordOnChange = (e: ChangeEvent<HTMLInputElement>) => {
        setPassword(e.target.value)
    }
    
    const handleOnSubmit = (e: FormEvent<HTMLFormElement>) => {
        e.preventDefault();
        if (!email.length) {
            
        }
        // .....유효성 검사 및 API 통신
    }
    
    return (
        <div>
            <main>
                <form onSubmit={handleOnSubmit}>
                    <div>
                        <div>
                            <div>
                                <div>Email</div>
                                <input 
                                    type="text"
                                    value={email}
                                    onChange={handleEmailOnChange}
                                />
                            </div>
                            <div>
                                <div>Password</div>
                                <input 
                                    type="password"
                                    value={password}
                                    onChange={handlePasswordOnChange}
                                />
                            </div>
                            <div>
                                <button type="submit">submit</button>
                            </div>
                        </div>
                    </div>
                </form>
            </main>
        </div>
    );
}
```
익숙한 form 입니다.<br>
`handleOnSubmit` 함수를 실행할 때 로그인을 위한 스키마와 그에 맞는 유효성을 검사하기 위해 작업하게 됩니다.<br>
이 과정들은 반복적이고 자칫 실수를 불러 이르킬 수 있습니다.

### Zod와 React Hook Form 사용하기
### SignInSchema
```ts
import { z } from 'zod';

export const SignInSchema = z.object({
  email: z
    .string()
    .min(1, { message: '아이디를 입력하세요.' })
    .email({ message: '이메일 형식에 맞추세요.' }),
  password: z
    .string()
    .min(1, {
      message: '패스워드를 입력하세요.',
    })
    .max(6, {
      message: '패스워드는 6자 이하로 입력하세요.',
    }),
});

export type SingInDto = z.infer<typeof SignInSchema>;
```
Zod의 `object` 메서드로 schema를 생성합니다.<br>
그리고 `infer`로 해당 스키마로 DTO type도 같이 생성해줍니다.

### React Hook Form/Zod SignIn Component
```tsx
import { SubmitHandler, useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import axios from "axios";
import { useState } from "react";
import {SignInSchema, SingInDto} from "@/app/libs/zod/schemas/user.zod.schema";

export default function SignIn() {
    const {
        register,
        handleSubmit,
        formState: { errors },
    } = useForm({
        resolver: zodResolver(SignInSchema),
        defaultValues: {
            email: "",
            password: "",
        },
    })
    const [successMsg, setSuccessMsg] = useState('');

    const onSubmit: SubmitHandler<SingInDto> = async (body) => {
        try {
            const url = ''
            const { data } = await axios.post(url, body);
            if (data) setSuccessMsg(data);
        } catch (e) {
            console.error(e);
        }
    }
    return (
        <div>
            <main>
                <form onSubmit={handleSubmit(onSubmit)}>
                    <div>
                        <div>
                            <div>
                                <div>Email</div>
                                <input 
                                    {...register('email')} 
                                    type="text" 
                                />
                                <div>{errors.email?.message}</div>
                            </div>
                            <div>
                                <div>Password</div>
                                <input
                                    {...register('password')} 
                                    type="password"
                                />
                                <div>{errors.password?.message}</div>
                            </div>
                            <div>
                                <button type="submit">submit</button>
                            </div>
                            <div>
                                {successMsg}
                            </div>
                        </div>
                    </div>
                </form>
            </main>
        </div>
    );
}
```
React Hook Form의 `useForm` 메서드로 `resolver`와 `defaultValues`를 할당한 뒤<br>
`register`로 input 태그에 해당 하는 value를 등록해주면 모든 작업은 끝입니다.<br>
`formState`의 `errors`에서 zod schema에 설정한 `message`를 사용자에게 보여주면 끝입니다.<br>

### Backend
nest 공식 문서에서 찾은 Zod 활용법을 소개합니다. 
### SignInSchema
```ts
import { z } from 'zod';

export const SignInSchema = z.object({
    email: z
        .string()
        .min(1, { message: '아이디를 입력하세요.' })
        .email({ message: '이메일 형식에 맞추세요.' }),
    password: z
        .string()
        .min(1, {
            message: '패스워드를 입력하세요.',
        })
        .max(6, {
            message: '패스워드는 6자 이하로 입력하세요.',
        }),
});

export type SingInDto = z.infer<typeof SignInSchema>;
```

Zod schema를 작성해봅시다.<br>
어디서 많이 본 형태입니다.<br>
Frontend에서 작성한 schema와 완벽히 동일합니다.<br>
### ZodValidationPipe
```ts
import {
    PipeTransform,
    ArgumentMetadata,
    Injectable,
    BadRequestException,
} from '@nestjs/common';
import { ZodSchema } from 'zod';

@Injectable()
export class ZodValidationPipe implements PipeTransform {
    constructor(private readonly schema: ZodSchema) {}

    transform(value: unknown, metadata: ArgumentMetadata) {
        try {
            this.schema.parse(value);
            return value;
        } catch (error) {
            throw new BadRequestException('Validation failed');
        }
    }
}

```
그 후 Zod schema로 유효성 검사를 해줄 Pipe를 생성합니다.<br>
만약 Frontend에서 올바르지 못한 데이터 타입의 발견된다면 해당 Pipe에서 error가 발생하고 Frontend는 BadRequestException을 받게 됩니다.<br>
바로 컨트롤러에서 사용해봅시다.

### UserController
```ts
@Controller()
export class UserController {
    constructor(private readonly userService: UserService) {}

    @Post('/signIn')
    signIn(@Body(new ZodValidationPipe(SignInSchema)) body: SingInDto) {
        // ....
        return '회원가입이 완료되었습니다.';
    }
}
```
ZodValidationPipe와 Zod Schema 그리고 Schema를 토대로 만든 Dto 이 세가지로 Frontend에서 넘어오는 데이터의 유효성 검사를 해줍니다.


### 마치며
monorepo와 같이 백엔드에서 수정 또는 프론트에서 수정시 연동되지는 않지만<br>
스키마를 통일시키면서<br><br>
`Frontend는 인터페이스 생성/유효성 검증`<br>
`Backend는 DTO 생성(class-validator를 대체)/유효성 검증`<br><br>
을 전보다 명확하고 심플하게 할 수 있게 되었습니다.<br>
혼자 작업하면 더욱 편하게 작업 할 수 있으며<br>
팀 작업을 한다면 백/프론트의 한쪽이 맡아서 스키마를 작성하고 공유한다면<br>
비즈니스 로직에 더욱 집중할 수 있어 보이지만 이 또한 팀원간의 소통이 중요하다고 생각합니다.<br>
피넷 리뉴얼 프로젝트에 monorepo를 구성하기 위한 여러 시도 끝에 결국 실패했지만 좋은 경험이었고<br>
저에게 Zod라는 멋진 녀석을 남겨주었습니다.<br>
이미 많은 개발자들은 Zod를 사용하고 있습니다.<br>
2024.12.26 기준<br>
`npm Weekly Downloads 12,662,457`<br>
`github Star 34,709`<br>
늦게라도 알고 적용해 볼 수 있어 다행입니다.<br>
이번 포스팅에서는 Zod와 React Hook Form 둘의 활용법을 소개해드렸습니다.<br>
각각의 라이브러리가 궁금하시다면 아래 링크로...

### React Hook Form
[https://react-hook-form.com/get-started](https://github.com/YoungminShimPMI/data_binding_test){:target="_blank"}

### Nest의 Zod 활용
[https://docs.nestjs.com/pipes](https://ko.vuejs.org/api/sfc-script-setup){:target="_blank"}

### Zod
[https://zod.dev/](https://ko.vuejs.org/api/sfc-script-setup){:target="_blank"}
