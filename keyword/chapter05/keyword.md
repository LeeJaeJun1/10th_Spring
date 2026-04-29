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

<hr>

### 오케스트레이션 서비스와 쿠버네티스

  **컨테이너 지옥과 오케스트레이션의 등장**

  - 과거에는 거대한 서버 컴퓨터 하나에 모든 프로그램을 욱여넣었지만, 요즘은 도커(Docker)를 이용해 프로그램을 실행 환경까지 통째로 포장하는 **'컨테이너'** 방식을 주로 사용합니다.
  - 처음엔 내 컴퓨터와 서버의 환경을 똑같이 맞출 수 있어 평화로웠습니다. 하지만 서비스가 커지면서 이른바 '컨테이너 지옥(Container Hell)'이 시작되었습니다.
      - "결제용 컨테이너 50개, 로그인용 20개를 띄워야 해!"
      - "어? 3번 서버가 죽어서 컨테이너 10개가 날아갔어. 빨리 다시 띄워!"
      - "트래픽이 폭주한다! 당장 컨테이너 100개로 늘려!"
  - 수백, 수천 개의 컨테이너를 사람이 일일이 감시하고 관리하는 것은 불가능합니다. 이때 등장한 것이 바로 컨테이너들의 배치, 관리, 확장, 복구를 자동으로 해주는 컨테이너 오케스트레이션(Container Orchestration)입니다. 이름 그대로 수많은 악기(컨테이너)를 조화롭게 연주하는 '지휘자'의 역할이죠.

  **쿠버네티스(K8s)**

  - 이 오케스트레이션이라는 전략을 실제로 구현한 전 세계 1위 도구가 바로 쿠버네티스(Kubernetes)입니다. 구글이 자사 서비스를 운영하던 엄청난 노하우를 녹여 만든 오픈소스 프로젝트입니다.
  - 쿠버네티스의 구조는 크게 '명령을 내리는 두뇌(Control Plane)'와 '명령을 수행하는 근육(Worker Node)'으로 나뉩니다.

  **컨트롤 플레인 (마스터 노드) :** 클러스터(서버들의 무리)를 총괄하는 지휘 본부입니다.

  - **kube-apiserver:** 쿠버네티스의 입구. 모든 명령은 이곳을 거칩니다.
  - **etcd:** 클러스터의 모든 상태 정보가 저장되는 핵심 저장소(금고)입니다.
  - **kube-scheduler:** 빈 서버를 찾아 컨테이너를 적절히 배치하는 인력 배치 팀장입니다.
  - **kube-controller-manager:** 시스템이 목표 상태를 잘 유지하는지 끊임없이 감시하는 순찰대입니다.

  **워커 노드 :** 실제 우리의 앱(컨테이너)이 돌아가는 일꾼 서버들입니다.

  - **kubelet:** 마스터 노드의 명령을 받아 컨테이너를 관리하는 현장 소장입니다.
  - **kube-proxy:** 네트워크 트래픽을 알맞게 배분하는 교통 경찰입니다.
  - **Pod (파드):** 쿠버네티스에서 가장 작고 중요한 **배포 단위**입니다. K8s는 컨테이너를 직접 다루지 않고, 이 Pod라는 캡슐로 감싸서 관리합니다.

  **쿠버네티스의 3대 마법**

  쿠버네티스는 절차를 명령하는 것이 아니라, "내 앱(Pod)이 항상 3개 켜져 있게 해 줘"라고 목표 상태(Desired State)를 선언하는 방식을 사용합니다.

  1. **자가 치유 (Self-Healing):** 특정 서버가 다운되어 Pod가 죽으면, 즉시 다른 정상적인 서버에 새로운 Pod를 띄워 개수를 맞춥니다.
  2. **오토 스케일링 (Auto-Scaling):** 접속자가 폭증해 CPU 사용량이 늘어나면 자동으로 Pod 개수를 늘리고, 트래픽이 빠지면 다시 줄여 서버 비용을 최적화합니다.
  3. **무중단 배포 (Rolling Update):** 새로운 버전으로 업데이트할 때, 기존 서버를 한 번에 끄지 않고 새 버전을 하나씩 교체하여 사용자가 눈치채지 못하게 배포합니다.

<hr>

### EDA

  인프라를 쿠버네티스로 탄탄하게 다졌다면, 내부 소프트웨어 설계는 어떻게 결합도를 낮출 수 있을까요? 그 해답 중 하나가 EDA(Event-Driven Architecture)입니다.

  우리가 흔히 아는 기존의 통신 방식은 요청-응답(Request-Response)입니다.

  - "내가 이걸 줄 테니, 넌 바로 처리해서 나한테 완료됐다고 알려줘!" (강한 결합)

  반면 **EDA**는 "어떤 일이 발생했다!"라고 소문(Event)만 내고 자기 할 일을 하러 가는 방식입니다.

  - "음식이 완성됐어!" (진동벨을 울림) -> 손님이 알아서 음식을 가져감.

  **EDA의 핵심 구성 요소**

  1. **발행자(Producer):** "주문이 완료됐어!"라는 이벤트 메시지를 발행합니다.
  2. **채널/브로커(Broker):** 메시지를 중간에서 보관하고 전달합니다. (Kafka, RabbitMQ 등)
  3. **구독자(Consumer):** 메시지를 듣고 있다가 "어? 주문됐네? 난 배송 준비해야지!" 하고 각자의 로직을 비동기적으로 처리합니다.

  **EDA의 장점**

  시스템들이 서로를 굳이 알 필요가 없기 때문에(Decoupling), 새로운 기능이 추가되거나 트래픽이 몰려도 유연하게 대처하고 확장할 수 있습니다.

  잘 짜인 스프링 부트 애플리케이션의 결합도를 **EDA**로 낮추고, 이를 도커 이미지로 말아 **쿠버네티스** 위에 올린다면, 트래픽 폭주나 서버 장애 앞에서도 끄떡없는 견고한 백엔드 시스템을 구축할 수 있습니

<hr>

### 카프카

  **카프카**

  - EDA(이벤트 기반 아키텍처)에서 이벤트를 안전하고 빠르게 전달해 주는 '초고속 우체국' 또는 '거대한 게시판' 역할을 하는 것이 바로 카프카
  - 링크드인(LinkedIn)에서 매일 쏟아지는 엄청난 양의 로그와 데이터를 처리하기 위해 만들었다가 오픈소스로 푼 '분산 이벤트 스트리밍 플랫폼'입니다.
  - 기존의 메시지 큐(Message Queue) 시스템들이 "메시지 하나 전달하고 지우기"에 집중했다면, 카프카는 "엄청난 양의 데이터를 잃어버리지 않고, 병렬로 아주 빠르게 처리하기"에 특화되어 있습니다.

  [용어]

  - **발행자 (Producer):** "주문이 완료됐어!"라는 이벤트(메시지)를 생성해서 카프카로 보내는 애플리케이션입니다. (예: 주문 서버)
  - **토픽 (Topic):** 메시지가 저장되는 **'카테고리'** 또는 '게시판'입니다. "주문 내역 게시판(`order-topic`)", "회원 가입 게시판(`member-topic`)"처럼 목적에 따라 여러 개를 만듭니다.
  - **구독자 (Consumer):** 토픽(게시판)을 구독하고 있다가, 새로운 메시지가 올라오면 가져가서 처리하는 애플리케이션입니다. (예: 결제 서버, 알림 서버)
  - **브로커 (Broker):** 카프카 서버 그 자체입니다. 보통 데이터가 날아가는 것을 방지하기 위해 브로커 3대 이상을 묶어서(클러스터) 운영합니다.

  [장점]

  1. 디스크(Disk)에 영구 저장한다
     - 일반적인 메시지 큐는 컨슈머가 메시지를 읽어가면 그 메시지를 큐에서 바로 삭제합니다. 하지만 카프카는 **메시지를 하드디스크에 파일 형태로 지정된 기간(예: 7일) 동안 보관**합니다.
     - 컨슈머 서버가 죽었다가 며칠 뒤에 살아나도, 자기가 읽다 만 시점(Offset)부터 다시 데이터를 안전하게 읽어갈 수 있습니다.
  2. 둘째, 파티션(Partition)을 통한 극강의 병렬 처리
     - 카프카는 하나의 토픽을 여러 개의 '파티션(쪼개진 방)'으로 나눕니다.
     - 대기 줄이 1개일 때보다 3개일 때 손님을 훨씬 빨리 받을 수 있듯이, 파티션을 나누면 여러 명의 컨슈머가 동시에 달라붙어 데이터를 엄청나게 빠른 속도로 병렬 처리할 수 있습니다.
  3. 완벽한 디커플링 (Decoupling)
     - 프로듀서는 컨슈머가 죽었든 살았든, 몇 개가 있든 신경 쓰지 않고 오직 카프카의 '토픽'에만 메시지를 던집니다. 컨슈머 역시 프로듀서의 상태와 무관하게 카프카에서 메시지를 가져오기만 하면 됩니다. 서비스 간의 의존성이 완벽하게 분리됩니다.

  [사용 경우]

  - 초당 수만 건 이상의 압도적인 트래픽(로그, 클릭 스트림, 결제 이벤트)을 지연 없이 처리해야 할 때
  - 데이터를 절대 잃어버리면 안 될 때 (장애 복구가 중요할 때)
  - 하나의 이벤트를 여러 개의 다른 서비스(알림, 통계, 결제 등)가 동시에 가져가서 처리해야 할 때

<hr>

### 메시지큐와 이벤트 소싱

  **메시지큐 : "데이터를 임시로 보관하는 안전한 우체통"**

  - 시스템 간에 데이터를 주고받을 때, 직접 바로 전달하지 않고 중간에 큐(Queue, 대기열)라는 버퍼를 두어 메시지를 차례대로 쌓아두고 처리하는 방식입니다.

  [장점]

  - **비동기 처리 (Asynchronous):** 메시지를 던져놓고 내 할 일을 계속할 수 있어 시스템이 멈추지 않습니다.
  - **의존성 분리 (Decoupling):** A 서버는 B 서버가 지금 살아있는지, 바쁜지 신경 쓸 필요 없이 MQ에만 데이터를 보내면 됩니다.
  - **충격 흡수 (Buffer):** 갑자기 주문(트래픽)이 폭주해도 MQ에 쌓아두면 되므로, 서버가 한 번에 터지는 것을 막아줍니다.

  **이벤트 소싱** : "결과만 저장하지 않고, 결과가 만들어진 '모든 과정'을 저장하는 방식”

  - 이벤트 소싱은 데이터가 변경된 '모든 이벤트(Event)들의 역사'를 순서대로 전부 기록

  [장점]

  - **완벽한 추적과 감사 (Audit):** "이 데이터가 도대체 왜 이렇게 변했지?"라는 의문을 100% 추적할 수 있습니다.
  - **상태 복원 (Replay):** 과거 특정 시점까지의 이벤트만 다시 재생(Replay)하면, 언제든 과거의 상태로 시스템을 완벽하게 되돌릴 수 있습니다.
  - **데이터 유실 방지 (Immutability):** 한 번 발생한 이벤트는 절대 수정되거나 삭제되지 않고(Update, Delete 없음) 오직 추가(Insert)만 되므로 데이터가 안전합니다

<hr>

### 2PC 패턴과 SAGA 패턴

  **2PC** : Two-Phase Commit, 2단계 커밋 ("모두가 완벽하게 준비될 때까지 아무도 움직이지 마!”)

  - 전통적인 분산 트랜잭션 처리 방식입니다. 이름 그대로 준비(Prepare)와 **실행(Commit)** 두 단계로 나누어 처리합니다. 이 과정을 지휘하는 '코디네이터(Coordinator)'가 존재합니다.

  [장단점]

  - **장점:** 데이터의 일관성이 100% 보장됩니다. (강한 일관성)
  - **단점:** 누군가 대답을 늦게 하거나 에러가 나면, **모든 DB가 멈춘 채로(Lock) 대기**해야 합니다. 시스템 전체의 성능이 뚝 떨어지기 때문에 빠르고 유연해야 하는 MSA 환경에서는 거의 **금기시**되는 패턴입니다.

  **SAGA** : "일단 각자 할 일 하고, 문제 생기면 수습(환불/취소)하자!”

  - 1단계 - 2단계 - 3단계를 순차적으로 검사하면서 commit하고 다음으로 넘김
  - 만약에 3단계에서 실패한다면 2단계 - 1단계로 돌아가며 보상 트랜잭션을 진행함 (rollback)
  - MSA에서 2PC의 느린 속도와 Lock 문제를 해결하기 위해 등장한 현대적인 패턴입니다. 중앙 통제 대신, **각 서비스가 로컬 트랜잭션을 순차적으로 실행**하며 이벤트를 통해 다음 단계로 넘어갑니다.

  **[구현방식]**

  - **코레오그래피 (Choreography - 안무):** 중앙 지휘자 없이, 댄서들이 음악(이벤트)에 맞춰 알아서 춤을 추는 방식. (서비스들끼리 카프카 같은 MQ를 통해 직접 이벤트를 주고받음)
  - **오케스트레이션 (Orchestration - 지휘):** 중앙에 '주문 관리자(Orchestrator)'라는 지휘자가 있어서, "결제 서비스야, 돈 빼라!", "재고 서비스야, 취소해라!"라고 일일이 명령을 내리는 방식.

  [장단점]

  - **장점:** Lock을 걸지 않으므로 처리 속도가 매우 빠르고 확장성이 뛰어납니다. (MSA에 찰떡!)
  - **단점:** 코드가 엄청나게 복잡해집니다. 개발자가 모든 실패 케이스에 대한 '취소 로직(보상 트랜잭션)'을 일일이 다 짜야 합니다. 또한, 잠깐 동안은 "결제는 됐는데 재고는 아직 안 줄어든" 상태가 유지되는 '결과적 일관성(Eventual Consistency)'을 감수해야 합니다.

  | **구분** | **2PC (Two-Phase Commit)** | **SAGA 패턴** |
      | --- | --- | --- |
  | **핵심 전략** | Lock을 걸고 다 같이 동시에 Commit | 각자 Commit 하고, 실패 시 취소(보상) |
  | **일관성** | 강한 일관성 (항상 정확함) | 결과적 일관성 (최종적으로만 정확하면 됨) |
  | **처리 속도** | 느림 (병목 현상 발생) | 매우 빠름 (비동기 처리) |
  | **MSA 적합도** | ❌ 부적합 | 🟢 매우 적합 (표준) |
  | **개발 난이도** | 인프라 차원에서 지원 (비교적 쉬움) | 모든 롤백(보상) 로직을 직접 구현해야 함 (어려움) |

<hr>

### Spring Cloud Eureka

  **Spring Cloud Eureka** : 모든 마이크로서비스들의 이름과 현재 살아있는 IP 주소를 적어두는 **중앙 전화번호부**

  - Eureka는 크게 Server(서버)와 **Client(클라이언트)** 두 가지로 나뉨
  - Client : 회원 서비스, 주문 서비스 등 우리가 만든 스프링 부트 앱들입니다. 이들은 켜지자마자 가장 먼저 Eureka Server에 자신의 주소를 등록합니다. (**Service Registration**)
      - Eureka Client들은 **기본적으로 30초마다 한 번씩 "저 아직 살아있어요! 쿵쾅!" 하고 하트비트(Heartbeat) 신호를 보냅니다**