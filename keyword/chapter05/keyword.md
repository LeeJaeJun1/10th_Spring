### 빌더패턴이란?

  **Builder 패턴 :** 여러 생성자 인수가 필요한 복잡한 개체를 만드는 문제를 해결하는 데 사용되는 생성 디자인 패턴입니다.

  - Builder 패턴은 객체의 구성과 표현을 분리합니다. 이를 통해 동일한 구성 프로세스에서 다른 표현을 만들 수 있습니다.
  - 이러한 Builder 패턴은 Spring Boot에서 주로 Lombok 라이브러리를 활용해 사용된다.

  **[장점]**

  - Builder 패턴은 각 메소드 호출이 객체의 어떤 부분을 설정하는지 명확하게 알 수 있어서 가독성이 향상됩니다.
  - Builder 패턴을 사용하면 불변 객체를 쉽게 구현할 수 있습니다. 한 번 객체가 생성되면 불변하게 사용할 수 있어 예측 가능하고 안정적인 코드를 작성할 수 있습니다.
  - Builder 메소드를 통해 필요한 매개변수만 선택적으로 설정할 수 있습니다.

    ```
    public class MemberResDTO {
    
        @Builder
        public record GetInfo(
            String name,
            String profileUrl,
            String email,
            String phoneNumber,
            Integer point
        ) {}
    }
    
    public class MemberConverter {
    
        public static MemberResDTO.GetInfo toGetInfo(Member member) {
            return MemberResDTO.GetInfo.builder()
                .email(member.getEmail())
                .name(member.getName())
                .point(member.getPoint())
                .phoneNumber(member.getPhoneNumber())
                .profileUrl(member.getProfileUrl())
                .build();
        }
    }
    ```

<hr>

### record vs static class

  - 자바에서 DTO를 만들 때 대표적으로 쓰이는 두 가지 방식

  데이터를 전달하기 위한 용도지만 설계 철학과 사용 목적이 다르다

  1. public static class DTO 방식
     - 전통적인 자바 방식의 DTO
     - 클래스 내부에 public static class 형태로 선언해 관련 DTO들을 한 곳에서 관리할 때 사용
     - getter, setter, builder 등 자유롭게 추가 가능

    
    public class UserDto {
        
        // 요청(Request) DTO
        public static class Request {
            private String name;
            private int age;
        
            public Request() {} // 기본 생성자
            public Request(String name, int age) {
                this.name = name;
                this.age = age;
            }
        
            public String getName() { return name; }
            public void setName(String name) { this.name = name; }
            public int getAge() { return age; }
            public void setAge(int age) { this.age = age; }
        }
        
        // 응답(Response) DTO
        public static class Response {
            private Long id;
            private String name;
        
            public Response(Long id, String name) {
                this.id = id;
                this.name = name;
            }
        
            public Long getId() { return id; }
            public String getName() { return name; }
        }
    }
    

  [특징]
    
  - 한 파일 안에 여러 DTO를 정리 가능
  - 가변 객체 - setter로 수정 가능
  - Lombok 사용 시 코드 간결화
  - 필요시 메서드 추가 및 필드 가공 로직 삽입 가능하기 때문에 유연함
  - 한 클래스 내부에서 여러 DTO 관리 가능
    
  [단점]
    
  - 코드가 길어짐
  - setter로 인해 불변성이 깨질 수 있다
    
  2. record DTO 방식
     - Java 16 이상에서 지원하는 불변 데이터 클래스
     - 데이터 전달만 담당하기 때문에 DTO 목적에 완벽히 부합한다
     - 필드, 생성자, getter, equals, hashCode, toString 자동 생성
        

    
    public record UserResponse(Long id, String name, int age) {}
        
    ---------------------------------------------------------
        
    public final class UserResponse {
        private final Long id;
        private final String name;
        private final int age;
        
        public UserResponse(Long id, String name, int age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }
        
        public Long id() { return id; }
        public String name() { return name; }
        public int age() { return age; }
        
        public boolean equals(Object o) { ... }
        public int hashCode() { ... }
        public String toString() { ... }
    }
    ```
        
    
  [특징]

   - 불변객체
   - 간결함
   - 데이터 전달에 특화됨
   - Lombok 불필요
   - JSON 직렬화 완벽 호환
    
   [단점]
    
   - setter, builder 사용불가
   - 복잡한 로직이나 가공 메서드를 담기 어려움

<hr>

### 제네릭이란?

  **제네릭** : 데이터의 타입(Type)을 내부에서 확정하지 않고, 외부에서 사용할 때 결정하는 방식

    
    List list = new ArrayList();
    list.add("주니"); // 문자열 저장
    
    String name = (String) list.get(0); // 꺼낼 때 반드시 (String)이라고 수동으로 알려줘야 함
    
    -----
    
    List<String> list = new ArrayList<>(); // "이 리스트는 String 전용이야"
    list.add("주니");
    
    String name = list.get(0); // 형변환 없이 바로 꺼내 쓸 수 있다
    
    -----
    
    public class ApiResponse<T> {
        private Boolean isSuccess;
        private String code;
        private T result; // 어떤 데이터가 올지 모르니 일단 T로 비워둔다
    }
    

  ⇒ 하나의 `ApiResponse` 클래스만 만들어두고, **안에 담길 `result`만 그때그때 바꿔서 재사용**할 수 있다

  [장점]

  - 타입 안정성 : 엉뚱한 타입의 데이터가 들어오는 걸 컴파일 단계에서 바로 잡아냅니다.
  - 가독성: 이 객체가 어떤 데이터를 다루는지 코드를 보는 즉시 알 수 있습니다.
  - 코드 재사용성: 클래스나 메서드를 하나만 만들어두고 다양한 타입에 돌려쓸 수 있습니다.

<hr>

### @RestControllerAdvice이란?

  **@RestControllerAdvice** : Spring에서 전역 예외 처리나 공통 응답 처리를 할 때 사용하는 기능

  → @ControllerAdvice + @ResponseBody

  → @RestController에 대해 전역적으로 적용되는 설정이나 예외 처리를 담당하는 클래스

  [장점]

  1. 예외 처리 코드의 중복 제거
        - 각 컨트롤러마다 try-catch 필요 없이 모든 예외를 한 곳에서 관리 가능
        - 코드가 깔끔해지고 유지보수 쉬워짐
  2. AOP 기반의 전역 처리 가능
        - 스프링의 AOP 기능으로 동작하기 때문에 컨트롤러 내부 코드에 영향 주지 않음
  3. 일관된 에러 응답 형식 유지
        - 모든 에러 응답을 통일할 수 있다.
        - 에러 형식 일정해짐

  [없을 때 불편한점]

  1. 예외 처리할 때 각 컨트롤러마다 try-catch 문을 사용해야한다
  2. 에러 응답 형식이 컨트롤러마다 다를 수 있다
  3. 예외를 하나 추가할 때마다 여러 컨트롤러의 수정이 필요하다
  4. 코드가 복잡하고 중복이 많기 때문에 가독성이 떨어진다

  [같이 사용되는 어노테이션]

  | 어노테이션 | 설명 |
      | --- | --- |
  | `@ExceptionHandler(Exception.class)` | 특정 예외 타입 처리 |
  | `@ResponseStatus(HttpStatus.BAD_REQUEST)` | 응답 상태코드 지정 |
  | `@RestControllerAdvice(basePackages = "com.example.api")` | 특정 패키지에만 적용 |
  | `@Slf4j` | 로그 찍기 (예외 로그 출력용) |

  [@RestControllerAdvice 사용 과정]

  1. 에러 코드 정의

    
    package com.example.umc9th.global.apiPayload.code;
    
    import org.springframework.http.HttpStatus;
    
    public interface BaseErrorCode {
        HttpStatus getStatus();
        String getCode();
        String getMessage();
    }
    
    package com.example.umc9th.global.apiPayload.code;
    
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import org.springframework.http.HttpStatus;
    
    @Getter
    @AllArgsConstructor
    public enum GeneralErrorCode implements BaseErrorCode{
    
        BAD_REQUEST(HttpStatus.BAD_REQUEST,
                "COMMON400_1",
                "잘못된 요청입니다."),
        UNAUTHORIZED(HttpStatus.UNAUTHORIZED,
                "AUTH401_1",
                "인증이 필요합니다."),
        FORBIDDEN(HttpStatus.FORBIDDEN,
                "AUTH403_1",
                "요청이 거부되었습니다."),
        NOT_FOUND(HttpStatus.NOT_FOUND,
                "COMMON404_1",
                "요청한 리소스를 찾을 수 없습니다."),
        INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR,
                "COMMON500_1",
                        "예기치 않은 서버 에러가 발생했습니다."),
                ;
    
        private final HttpStatus status;
        private final String code;
        private final String message;
    }
    
    

  2. 커스텀 예외를 만든다

    ```
    package com.example.umc9th.global.apiPayload.exception;
    
    import com.example.umc9th.global.apiPayload.code.BaseErrorCode;
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    
    @Getter
    @AllArgsConstructor
    public class GeneralException extends RuntimeException {
    
        private final BaseErrorCode code;
    }
    
    ```

  3. @RestControllerAdvice를 선언한 클래스 만든다

    
    package com.example.umc9th.global.apiPayload.handler;
    
    import com.example.umc9th.global.apiPayload.ApiResponse;
    import com.example.umc9th.global.apiPayload.code.BaseErrorCode;
    import com.example.umc9th.global.apiPayload.code.GeneralErrorCode;
    import com.example.umc9th.global.apiPayload.exception.GeneralException;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestControllerAdvice;
    
    @RestControllerAdvice
    public class GeneralExceptionAdvice {
    
        // 애플리케이션에서 발생하는 커스텀 예외를 처리
        @ExceptionHandler(GeneralException.class)
        public ResponseEntity<ApiResponse<Void>> handleException(
                GeneralException ex
        ) {
    
            return ResponseEntity.status(ex.getCode().getStatus())
                    .body(ApiResponse.onFailure(
                                    ex.getCode(),
                                    null
                            )
                    );
        }
    
        // 그 외의 정의되지 않은 모든 예외 처리
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ApiResponse<String>> handleException(
                Exception ex
        ) {
    
            BaseErrorCode code = GeneralErrorCode.INTERNAL_SERVER_ERROR;
            return ResponseEntity.status(code.getStatus())
                    .body(ApiResponse.onFailure(
                                    code,
                                    ex.getMessage()
                            )
                    );
        }
    }


  4. Controller나 Service에서 커스텀 예외를 발생시킨다.

<hr>

### Optional이란?

  **Optional** : `Optional`은 이 null을 직접 다루지 않도록 감싸주는 '캡슐' 같은 존재

  `Optional<T>`는 어떤 객체를 담고 있는 컨테이너입니다. 이 상자 안에는 내용물이 있을 수도 있고, 비어 있을 수도 있습니다.

  - 상자에 내용물이 있음: `Optional` 객체가 그 값을 가지고 있음.
  - 상자가 비어 있음: `Optional.empty()` 상태 (기존의 `null` 대신 사용).

    ```
    기존방식
    
    Member member = memberRepository.findById(id);
    if (member != null) {
        System.out.println(member.getName());
    } else {
        throw new MemberHandler(MemberErrorCode.MEMBER_NOT_FOUND);
    }
    
    -> if (member != null) 처리를 깜빡하는 순간 프로그램은 NPE(Null Pointer Exception)
    와 함께 멈춰버립니다.
    
    memberRepository.findById(id)
        .ifPresentOrElse(
            member -> System.out.println(member.getName()),
            () -> { throw new MemberHandler(MemberErrorCode.MEMBER_NOT_FOUND); }
        );
        
    -> 이 값이 없을 수도 있다고 명시적으로 알려주기 때문에 개발자가 null 처리 강제하게 한다
    ```

  | **메서드** | **설명** |
      | --- | --- |
  | **`Optional.ofNullable(val)`** | 값이 null일 수도, 아닐 수도 있을 때 상자에 담습니다. |
  | **`orElse(default)`** | 상자가 비어있다면 `default` 값을 대신 줍니다. |
  | **`orElseThrow()`** | 상자가 비어있다면 내가 지정한 예외를 던집니다. |
  | **`map()` / `filter()`** | 상자 안의 값을 변환하거나 조건에 맞는지 검사합니다. |

  [주의할 점]

  - .get()을 사용하게 되면 비어있는 경우 에러가 나기 때문에 가급적 사용하지 않는다
  - Optional은 반환타입으로만 사용해야 한다
  - 컬렉션 (set, list)를 감싸면 안된다. 그냥 빈 리스트를 반환하는게 표준이다

    ```
    
    Member member =  memberRepository.findById(memberId)
                .orElseThrow(() -> new MemberException(MemberErrorCode.MEMBER_NOT_FOUND));
        
        
    멤버 레포지터리에서 해당하는 MemberID를 찾지 못한다면 해당 에러코드를 발생시켜라
    ```