# 레시피 생성 서비스

> 엘리스 AI 트랙 4기 · AI 프로젝트 11팀 · 2022.05.31 ~ 07.02 · 6명

냉장고에 남은 재료와 만들고 싶은 음식 이름을 입력하면, 레시피 데이터로 학습한 KoGPT2 모델이 새 레시피를 만들어 주는 서비스입니다. 만든 레시피는 커뮤니티에 올려 좋아요·댓글을 주고받을 수 있습니다.

## 내 역할: 백엔드 리드

백엔드 대부분을 설계하고 구현했습니다. 백엔드 코드 가운데 지금 남아 있는 줄의 **88%**(1,760 / 2,008줄)가 제 커밋이고, 백엔드 파일 중 제가 작성하지 않은 파일은 `models/index.js` 하나입니다.

- **DB 설계와 구성**: MongoDB(Mongoose)와 MySQL(Sequelize)을 함께 쓰는 구조를 잡고, 두 DB의 모델을 모두 작성했습니다.
  - MySQL 테이블: [`user`](back/src/db/models/tables/user.js), [`post`](back/src/db/models/tables/post.js), [`recipe`](back/src/db/models/tables/recipe.js), [`comment`](back/src/db/models/tables/comment.js), [`ingredient`](back/src/db/models/tables/ingredient.js)
  - MongoDB 스키마와 모델: [`db/schemas`](back/src/db/schemas), [`db/models`](back/src/db/models), [`db/collections`](back/src/db/collections)
- **커뮤니티 API 전체**
  - 게시글: 작성, 조회, 수정, 삭제, 좋아요, 랭킹 ([`postRouter`](back/src/routers/postRouter.js), [`postController`](back/src/controllers/postController.js), [`postService`](back/src/services/postService.js))
  - 댓글: 작성, 조회, 수정, 삭제 ([`commentRouter`](back/src/routers/commentRouter.js), [`commentController`](back/src/controllers/commentController.js), [`commentService`](back/src/services/commentService.js))
- **레시피 API와 AI 모델 서버 연동**: 레시피 생성 요청을 Flask 모델 서버로 보내고 결과를 저장합니다 ([`recipeRouter`](back/src/routers/recipeRouter.js), [`recipeAiModel`](back/src/utils/recipeAiModel.js)).
- **회원 API**: 회원가입, 로그인, 회원정보 수정·삭제 ([`userRouter`](back/src/routers/userRouter.js) 중 172줄, Sue와 함께 작업)
- **서버 기본 구성**: [`app.js`](back/src/app.js), 에러 처리 미들웨어, DB 연결 설정
- **API 문서**: Swagger로 User·Post·Comment API 문서를 작성했습니다. 마감 직전 자동 생성 폴더와 함께 정리되어 마지막 버전에는 남아 있지 않습니다.
- **프론트엔드 일부**: 메인 [`Home`](front/src/pages/Home.js), 소개 [`Prolog`](front/src/pages/Prolog.js), API 호출 모듈 [`Api.js`](front/src/Api.js)

## 주요 기능

- 재료와 음식 이름으로 레시피 생성 (KoGPT2)
- 생성한 레시피를 게시글로 공유
- 게시글 좋아요, 조회수, 랭킹
- 댓글
- 회원가입, 로그인, 마이페이지

## 기술 스택

| 구분 | 사용 기술 |
|---|---|
| Backend | Node.js, Express, MongoDB(Mongoose), MySQL(Sequelize), JWT, bcrypt |
| Frontend | React, React Router, styled-components, axios |
| AI | KoGPT2(PyTorch), Flask 모델 서버 |

## 팀 구성

커밋 기록 기준으로 각자 주로 작업한 부분입니다.

| 팀원 | 주로 작업한 부분 |
|---|---|
| **김현서** | 백엔드 전반 (DB 설계, 커뮤니티·레시피 API, 서버 구성), 프론트 메인·소개 페이지 |
| Sue | 백엔드: 회원 API, 레시피 저장 |
| Jeongsiwook | 프론트엔드: 커뮤니티, 메인, 결과, 소개 페이지 |
| 김별희 | AI 모델, 기획 문서 |
| KwanHoo | AI |
| Songwon Park | 프론트엔드: 커뮤니티 페이지 일부 |

## 폴더 구조

```
back/    Express API 서버
front/   React 앱
ai/      AI 모델 관련 안내
```

## 현재 상태

AI 모델 서버와 학습된 모델 파일은 이 저장소에 없어서, 레시피 생성 기능은 지금 그대로는 동작하지 않습니다. 나머지 API는 `back/`에서 `npm install` 후 실행할 수 있습니다 (MongoDB·MySQL 접속 정보가 담긴 `.env` 필요).

## 저장소 안내

| 구분 | 위치 |
|---|---|
| 팀 원본 | 엘리스 교육용 GitLab (외부에서 볼 수 없음, GitHub에는 원본이 없습니다) |
| 이 저장소 | 원본의 커밋 기록과 브랜치를 그대로 옮긴 개인 사본 |

접속 정보가 담긴 `.env` 파일은 이 사본의 커밋 기록에서 지웠습니다.

<details>
<summary>프로젝트 당시 기획서</summary>

### 임시 기획서

#### 프로젝트 목적:

냉장고 속에 남은 식재료들을 이용하여 만들 수 있는 요리 레시피를 제공하는 서비스<br/>
조리하고 싶은 식재료들과 음식명을 입력(input)하면 입력된 재료들을 활용한 레시피를 출력(output)으로 생성하여 사용자에게 제공

#### 기대효과와 활용방안

- 요리에 대한 동기부여를 주어 배달음식을 줄일 수 있음
- 기존에 남아있던 식재료를 활용하여, 음식물 쓰레기를 줄일 수 있음 -> 쓰레기 처리비용 감소로 경제적, 환경적 이익 기대

#### 사용할 데이터셋과 모델

- 모델: KoGPT2
- 데이터셋: - 데이터 파일 형식: .txt
- 데이터 출처: [해먹남녀](https://haemukja.com/) , [만개의레시피](https://www.10000recipe.com/), [메뉴판닷컴](https://www.menupan.com/), [공공데이터포털](https://www.data.go.kr/) 으로부터 레시피 데이터를 크롤링하여 구성
- train data: 18320개 / test data: 2181개 / valid data: 2181개
- 데이터 형태: 모델이 요리 이름/재료/레시피의 경계를 명확히 알 수 있도록 토큰 삽입 <br/> <unused0> : 요리 이름 시작, <unused1> : 요리 이름 끝, <unused2>: 재료 시작, <unused3>: 재료 끝, <unused4>: 레시피 본문 시작, <unused5>: 레시피 본문 끝 <br/>
  '$’ 문자로 재료를 구분 <br/>                        
(추가적으로 데이터를 크롤링을 한다면 아래와 같이 전처리 과정이 필요) <br/>
```<unused0>칼륨 듬뿍 고구마죽<unused1><unused2>고구마$설탕$찹쌀$가루$물$잣$<unused3><unused4>고구마는 깨끗이 씻어서 껍질을 벗기고 4 정도로 잘라준다. 찜기에 고구마를 넣고 20-30분 정도 삶아 주고 블렌더나 체를 이용하여 잘 으 깨어 곱게 만든다. 고구마와 물을 섞어 끓이면서 찹쌀가루로 농도를 맞추고 설탕을 넣어 맛을 낸다. 잣을 팬에 노릇하게 볶아 다져서 고구마 죽에 섞는다. 기호에 따라 고구마를 튀겨 얹어 먹어도 좋다.<unused5>```

#### 사용할 방법, 라이브러리, 알고리즘:

(라이브러리,패키지) torch, pytorch-lightning, ratsnlp, konlpy, Korpora

#### 프로젝트와 유사한 인공지능 기반 서비스의 활용 사례와 한계점

### 유사한 서비스

1. 해먹남녀: 레시피 제공 서비스(인공지능 서비스는 아님)  
   갖고 있는 재료를 입력하면 레시피를 제공  
   => 한계점: 갖고있는 재료에 맞는 레시피가 없다면 ‘조건에 맞는 레시피가 0개 있습니다.’ 라는 결과물이 출력
2. 만개의레시피- 음식명을 입력하거나 국/탕, 면, 만두 등과 같은 큰 분류로 레시피 검색이 가능한 서비스
   => 세부적인 재료를 통해서 레시피를 추천해주지는 못함
3. 메뉴판
   레시피 서비스, 식단추천, 아카데미, 실시간 급상승 맛집,랭킹(인기요리,새로운요리)

### 본 프로젝트의 한계점:

- 정확한 g 양을 제공하기 어렵다
- 검증되지 않은 레시피기 때문에 맛이 없을 수가 있음

#### 기존 서비스와 해당 프로젝트와의 차별점

기존 레시피 웹 서비스는 갖고 있는 재료에 대한 레시피 정보가 없는 경우 서비스를 제공해주지 못함<br/>
=> 기존에 있던 음식뿐만 아니라 새로운 음식 레시피를 제공할 수 있음

</details>
