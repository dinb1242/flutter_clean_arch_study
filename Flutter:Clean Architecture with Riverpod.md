**원문 링크**: [여기를 클릭하기](https://theutsavg1.medium.com/implementing-clean-architecture-with-riverpod-for-modular-flutter-apps-7d21acfa9db0)

# Flutter: Riverpod 을 활용한 Clean Architecture

- 클린 아키텍처는 인터페이스 와 추상화 등에 있어서 기능에 대한 관심을 분리하는 것.
- 클린 아키텍처를 적용함으로써 기능이 변경되더라도 전체 어플리케이션의 코어 로직에 대한 의존성에 있어서 영향을 끼치지 않는다는 특징이 존재한다.

![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*h4ahfMrkEhwmx5_Y6Y7zOA.png)

## 클린 아키텍처의 레이어(Layers)
### 데이터 (Data)
- 어플리케이션의 가장 바깥 쪽에 존재하는 레이어는 데이터 레이어이다.
- 어플리케이션 내에서 서버-사이드 또는 로컬 데이터베이스, 데이터 관리 로직과의 통신을 담당한다.
- Repository 의 구현체가 존재한다.

#### 데이터 소스 (Data Source)
- 데이터에 대한 Fetch 와 Update 에 대한 프로세스를 정의한다.
- 원격 데이터 소스(Remote Data Source) 또는 로컬 데이터 소스(Local Data Source) 로 구성되어있다.
- 원격 데이터 소스의 경우, HTTP 요청을 통해 외부 API 를 호출한다.
- 로컬 데이터 소스의 경우, 데이터를 캐싱하거나, 영구화시킨다.
  
#### 레포지토리 (Repository)
- 데이터 레이어와 도메인 레이어 사이의 다리 역할을 담당한다.
- 도메인 레이어 내에 위치하는 레포지토리의 실제 구현체들이 해당 레이어에 존재한다.
- 레포지토리의 역할은 각기 다른 데이터 소스로부터의 데이터를 취득하는 역할을 담당한다.

### 도메인 (Domain)
- 도메인 레이어는 전체 비즈니스 로직을 담당한다.
- 도메인은 전체 어플리케이션의 비즈니스 로직으로서의 역할을 담당해야하므로 플러터의 요소는 배제시켜야한다. 즉, 순수 Dart 언어를 통해서 작성이 되어야한다.
#### 프로바이더 (Providers)
- 어플리케이션에서 로직 처리를 담당하는 요소이다.
- 레포지토리와 직접적으로 통신한다.

#### 레포지토리 (Repository)
- 바깥쪽 레이어에서 추가적인 기능 구현을 위해 필요한 추상화 클래스를 의미한다.

### 프레젠테이션 (Presentation)
- 앱 내에서 프레임워크에 가장 의존하는 계층이다.
- UI 와 UI 내에서 이벤트에 대한 로직을 담당한다. 단, 비즈니스 로직은 포함되어서는 안 된다.
#### 위젯 (Screens/Views)
- 위젯은 `StateNotiferProvider` 에 의해 수신되는 상태(State) 를 구독하거나, 위젯 내에서 발생하는 이벤트를 Notify 하는 역할을 담당한다.

#### 프로바이더 (Providers)
- 프레젠테이션 계층에서 필요한 로직에 대한 처리를 담당한다.
- 도메인 레이어에 위치한 `프로바이더 (Provider)` 와 직접적으로 통신한다.

## 프로젝트 설명 (Project Description)
- `main.dart` 파일은 서비스 초기화 코드 및 루트 `MyApp` 을 `ProviderScope` 로 감싼 로직이 존재한다.
- `main/app.dart` 내에서는 루트 `MaterialApp` 이 존재하며, 전체 어플리 케이션에서 라우팅을 다루기 위한 `AppRouter` 를 초기화하는 로직이 존재한다. 또한, 테마를 정의하기 위해 `AppTheme` 을 구현한 로직이 존재한다.
- `services` 는 구현체를 통해 앱-레벨에서의 서비스를 추상화한다.
- `shared` 디렉토리는 전반적인 기능에 있어서 공유되는 코드들을 내포한다.
- `theme` 은 앱에서의 전반적인 스타일 (색상, 테마 & 텍스트 스타일) 등을 정의한다.
- `model` 은 어플리케이션 내에서 필요한 전반적인 데이터 모델들을 정의한다.
- `http` 내에는 `Dio` 를 통해 구현된 로직들이 존재한다.
- `storage` 는 `SharedPreferences` 를 통해 구현된 로직들이 존재한다.
- `Service Locator` 패턴과 `Riverpod` 은 다른 계층에서 사용되는 서비스를 추상화하는데에 사용된다.

## 함수형 프로그래밍
- 클린 아키텍처는 놀라울만큼 특별한 기능들로 구성되어있지는 않다. 따라서 함수형 프로그래밍을 통해 충분히 구현이 가능하다.

### 아키텍처에 대한 주요 아이디어
 - 추상화된 `DataSource` 는 레포지토리의 구현을 통해 접근이 가능하다.
 - 추상화된 `Repository` 는 `StateNotifer` 를 통해 접근이 가능하고, `StateNotifer` 의 구현체는 위젯에서 접근하며, 이를 통해 각각의 계층에 대한 내부 구현이 변경되어도 구조의 분리 및 확장성을 보장할 수 있다.
  
    #### 예제 코드
    ```dart
    final storageServiceProvider = Provider((ref) {
        return SharedPrefsService();
    });
    ```

    ```dart
    // Usage:
    // ref.watch(storageServiceProvider);
    ```


# 추가 정리 필요