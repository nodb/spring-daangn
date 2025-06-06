# 지난주 보완 및 서비스 작성

## 리펙토링

### DirectMessage와 DirectMessageRoom 기능 분리

- DirectMessageRoom, DirectMessage 엔티티 설계 및 JPA 매핑
- 게시글 기반 1:1 채팅방 자동 생성 로직 구현
- 기존 방 존재 시 재활용 로직 적용

# API 작성
## Swagger
![swagger-ui_index html](https://github.com/user-attachments/assets/48102802-bfb5-4e6a-9d1d-a1ca62093d13)

### BoardController

| 메서드 | URL                                | 설명                                      |
| ------ | ---------------------------------- | ----------------------------------------- |
| POST   | `/api/v1/board`                    | 게시글 생성                               |
| GET    | `/api/v1/board`                    | 전체 게시글 조회 (삭제되지 않은 게시글만) |
| GET    | `/api/v1/board/deleted`            | 삭제된 게시글 목록 조회                   |
| GET    | `/api/v1/board/member/{memberId}`  | 특정 회원의 게시글 목록 조회              |
| GET    | `/api/v1/board/search?keyword=xxx` | 게시글 제목 검색                          |
| DELETE | `/api/v1/board/{boardId}`          | 게시글 soft delete                        |
| PATCH  | `/api/v1/board/{boardId}`          | 게시글 제목/내용 수정                     |

### BoardPhotoController

| 메서드 | URL                                   | 설명                  |
| ------ | ------------------------------------- | --------------------- |
| POST   | `/api/v1/board-photo`                 | 게시글 사진 등록      |
| GET    | `/api/v1/board-photo/{photoId}`       | 특정 사진 단건 조회   |
| GET    | `/api/v1/board-photo/board/{boardId}` | 게시글 사진 목록 조회 |
| DELETE | `/api/v1/board-photo/{photoId}`       | 게시글 사진 삭제      |

### CommentController

| 메서드 | URL                                        | 설명                         |
| ------ | ------------------------------------------ | ---------------------------- |
| POST   | `/api/v1/comment`                          | 댓글 또는 대댓글 생성        |
| GET    | `/api/v1/comment/board/{boardId}`          | 게시글의 모든 댓글 조회      |
| GET    | `/api/v1/comment/board/{boardId}/root`     | 게시글의 루트 댓글 조회      |
| GET    | `/api/v1/comment/parent/{parentCommentId}` | 특정 부모 댓글의 대댓글 조회 |
| DELETE | `/api/v1/comment/{commentId}`              | 댓글 삭제 (soft delete)      |
| PATCH  | `/api/v1/comment/{commentId}`              | 댓글 수정                    |

### DirectMessageController

| 메서드 | URL                          | 설명                          |
| ------ | ---------------------------- | ----------------------------- |
| POST   | `/api/v1/dm/board/{boardId}` | 게시글 기반 DM 전송           |
| GET    | `/api/v1/dm/room/{roomId}`   | 특정 DM 방의 메시지 목록 조회 |

### DirectMessageRoomController

| 메서드 | URL                                                                       | 설명                                        |
| ------ | ------------------------------------------------------------------------- | ------------------------------------------- |
| GET    | `/api/v1/dm/room/board/{boardId}/member/{memberId}`                       | 특정 게시글에서 내가 참여한 DM 방 목록 조회 |
| GET    | `/api/v1/dm/room/board/{boardId}/sender/{senderId}/receiver/{receiverId}` | 1:1 DM 방 조회 또는 생성                    |
| GET    | `/api/v1/dm/room/member/{memberId}`                                       | 내가 참여한 전체 DM 방 목록 조회            |

### FavoriteController

| 메서드 | URL                                            | 설명                                   |
| ------ | ---------------------------------------------- | -------------------------------------- |
| POST   | `/api/v1/favorite`                             | 좋아요 토글 (추가/삭제)                |
| GET    | `/api/v1/favorite/status?memberId=1&boardId=2` | 특정 회원이 게시글에 좋아요했는지 확인 |
| GET    | `/api/v1/favorite/member/{memberId}`           | 회원이 좋아요한 게시글 목록 조회       |
| GET    | `/api/v1/favorite/board/{boardId}`             | 게시글 좋아요한 회원 목록 조회         |
| GET    | `/api/v1/favorite/board/{boardId}/count`       | 게시글 좋아요 수 조회                  |

### MemberController

| 메서드 | URL                         | 설명                                       |
| ------ | --------------------------- | ------------------------------------------ |
| POST   | `/api/v1/member`            | 회원 생성                                  |
| GET    | `/api/v1/member`            | 전체 회원 목록 조회 (삭제되지 않은 회원만) |
| GET    | `/api/v1/member/deleted`    | 삭제된 회원 목록 조회                      |
| GET    | `/api/v1/member/{memberId}` | 특정 회원 상세 조회                        |
| DELETE | `/api/v1/member/{memberId}` | 회원 삭제 (soft delete)                    |
| PATCH  | `/api/v1/member/{memberId}` | 회원 정보 수정                             |

> [!NOTE]
> - 기본적인 CRUD 및 전체 조회와 특정값 조회가 필요한 부분은 따로 작성하였다.

> [!IMPORTANT]
> API 주소에 `v1`을 작성한 이유 : 버전 관리, 안정성, 유지보수성↑
> 
> 1. 향후 변경에 대비한 유연성 확보
> - 기존 클라이언트를 깨뜨리지 않고 새로운 버전(`v2`, `v3`) API 도입 가능
> 예
> - `/api/v1/member` ← 현재 버전
> - `/api/v2/member` ← 향후 구조 변경된 새 API
> 
> 2. 하위 호환성 유지
> - `v1`을 사용하는 기존 앱이나 프론트엔드가 그대로 동작 가능
> - 새 버전(`v2`)에서만 새로운 필드나 구조 도입 가능
> 
> 3. API 문서와 테스트 분리 용이
> - Swagger 문서, 테스트 케이스 등을 API 버전에 따라 구분해 관리할 수 있어 구조적으로 깔끔함
> 
> 4. 운영 및 배포 관리에 유리
> - 점진적 마이그레이션이 가능하므로 운영 리스크를 줄일 수 있음
> - 새 버전에서 오류 발생 시 기존 버전으로 빠르게 롤백 가능

## 실행화면
### 회원 생성
![swagger-ui_index html](https://github.com/user-attachments/assets/e0d8d945-3e15-4207-a419-f20eafd02368)
![swagger-ui_index html](https://github.com/user-attachments/assets/9b2fac4f-9423-46cf-8498-820ee56033dc)

### 게시글 생성
![swagger-ui_index html](https://github.com/user-attachments/assets/946501f8-fd53-4aa9-b4a4-3ef262644481)
![swagger-ui_index html](https://github.com/user-attachments/assets/ac2213ed-f51f-4f65-87cb-c4d8f5dead74)

## 댓글 생성
![swagger-ui_index html](https://github.com/user-attachments/assets/bb275d0f-ec30-4cad-85ea-06895982bfca)
![swagger-ui_index html](https://github.com/user-attachments/assets/435edddc-90eb-4ca9-88fc-2d4d2f0f513e)

# 정적 팩토리 메서드
- 기존에는 외부에서 Member.builder().xxx().build() 형태로 직접 생성했기 때문에, createdAt, updatedAt, isDeleted 등의 기본 필드 설정이 누락 가능
- 생성 시점의 일관된 기본값을 강제하고, 의미 있는 이름(of)을 통해 코드 가독성 향상과 의도 표현력 증가

## 적용 예(Member.of())
Member.java
```java
public static Member of(String membername, String email, String password, String profileUrl) {
    LocalDateTime now = LocalDateTime.now();
    return Member.builder()
            .membername(membername)
            .email(email)
            .password(password)
            .profileUrl(profileUrl)
            .isDeleted(false)
            .createdAt(now)
            .updatedAt(now)
            .build();
}
```
MemberService.java
```java
Member member = Member.of(
    dto.getMembername(),
    dto.getEmail(),
    dto.getPassword(),
    dto.getProfileUrl()
);
```

# Global Exception Handling

- 전역 예외 처리(Global Exception Handling) 를 통해 예외 발생 시 클라이언트에 일관된 형태의 JSON 응답을 반환

## GlobalExceptionHandler

- @RestControllerAdvice를 활용하여 예외를 처리
  | 예외 타입 | 설명 | HTTP 상태코드 |
  | -------------------------- | --------------------------- | --------------------------- |
  | `IllegalArgumentException` | 잘못된 파라미터 등 클라이언트 잘못으로 인한 예외 | 400 (Bad Request) |
  | `RuntimeException` | 예상하지 못한 런타임 오류 | 500 (Internal Server Error) |
  | `Exception` | 기타 모든 예외 | 500 (Internal Server Error) |
> [!NOTE]
> 추후 개발하며 상황에 따른 Exception을 더 세분화!

```json
// 예시: IllegalArgumentException 발생 시 응답
{
  "success": false,
  "data": null,
  "message": "존재하지 않는 회원입니다."
}
```

## ApiResponseDto

- ApiResponseDto<T>는 API 응답을 표준 구조로 래핑
  | 필드 | 설명 |
  | --------- | ------------------------------ |
  | `success` | 요청 처리 성공 여부 (`true` / `false`) |
  | `data` | 처리 결과 데이터 (성공 시) |
  | `message` | 오류 메시지 (실패 시) |

```java
// 성공 응답 예시
return ResponseEntity.ok(ApiResponseDto.success(responseData));

// 실패 응답 예시
return ResponseEntity.badRequest().body(ApiResponseDto.fail("오류 메시지"));
```

> [!NOTE]
> 적용 효과
>
> - 프론트엔드에서는 응답 구조를 일관되게 처리 가능
> - 예상치 못한 오류를 서버에서 로깅하며, 사용자에게는 깔끔한 메시지 전달
> - Swagger 문서에도 통일된 응답 포맷 적용 가능

# 테스트

## Controller 통합 테스트

- @SpringBootTest와 @AutoConfigureMockMvc를 활용하여 실제 컨트롤러에 HTTP 요청을 보내는 방식으로 테스트를 진행

### 테스트 목적

- 요청 → 서비스 → 리포지토리 → DB 흐름 검증
- 잘못된 입력, 비정상 요청, 예외 처리도 함께 검증 가능
- Swagger 문서와 실제 API 응답이 일치하는지 검증

### 테스트 클래스

| 테스트 클래스                     | 설명                           |
| --------------------------------- | ------------------------------ |
| `BoardControllerTest`             | 게시글 API의 CRUD 테스트       |
| `BoardPhotoControllerTest`        | 게시글 사진 업로드/삭제 테스트 |
| `CommentControllerTest`           | 댓글/대댓글 관련 API 테스트    |
| `FavoriteControllerTest`          | 좋아요 토글, 조회 API 테스트   |
| `MemberControllerTest`            | 회원 생성/수정/삭제 등 테스트  |
| `DirectMessageControllerTest`     | DM 전송/조회 테스트            |
| `DirectMessageRoomControllerTest` | DM 방 생성/조회 테스트         |

> [!NOTE]
> 다음 주의 Spring Security와 관련하여 인증 설정을 통한 보안 테스트 작성을 생각해보아야 할 것 같다.
