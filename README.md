**Bazel 을 처음 접하는 기초 개념을 설명하기 위한 가이드**

공식문서에 [Bazel 개념관련](https://bazel.build/concepts/build-ref) 문서가 있지만 한국 자료뿐만 아니라 영어자료 조차도 부족한 상황이라 

본인처럼 Bazel을 처음 접한사람이 이해하기 어려운 면이 별도로 문서로 해서 정리한다. 

# Concepts

## Workspace

우리가 보통 서비스 리포지토리 단위로 생각하는 개념을 Bazel 에서 WORKSPACE 라고 한다 Bazel 은 monorepo 를 위해 만들

Bazel 로 구성된 프로젝트의 루트에는 WORKSPACE 라는 파일이 존재하는데 여기서 프로젝트 공용으로 빌드에 필요한 의존성들을 추가한다. (e.g go, python, java 빌드에 필요한 의존성들)

## Package

Bazel 에서 소스코드를 소프트웨어 모듈단위를 Package 라고 부른다. 모듈단위의 크기는 사용자가 정하기 나름인데 크게는 한 프로젝트를 모듈로 잡을수도있고 작게는 라이브러리 단위 혹은 더 작은 비슷한 기능을 모아놓는 소스코드들의 단위로 설정할수 있다.  Package 단위로 소스코드의 컴파일 단위가 정해지고 캐쉬단위도 정해진다

Package 들은 BUILD 파일이라고 이름지어진 파일들을 가지고 있는데 위에 말한 모듈단위의 동작들을 어떻게  실행하지 정의하게 되고 실행하게 된다. 그리고 Package 는 다른 Package 에서 참조 가능하다. 

아래 파일구조로 들면 예를 들면 각각의 service 과 handler 에서는 라이브러리 형태로 Pacakge 를 정의하고 main.go 를 포함한 a-service 폴더에는 service,  repository, handler 를 참조해서 하나의 실행가능한 바이너리 파일로 빌드하고 실행이 가능한다

```
repo/
├─ WORKSPACE
├─ a-service/
│  ├─ BUILD
│  ├─ main.go
│  ├─ service/
│  │  ├─ BUILD
│  ├─ repository/
│  │  ├─ BUILD
│  ├─ handler/
│  │  ├─ BUILD
```

## Target

Target 는 우리가 다른 빌드툴에서 보는  빌드툴의 명령어와 명령어 대상을 나타내는 논리적인 단위라고 볼수 있다. 각각의 Package 는 적개는 1개에서 다수의 Target 을 가지게 되는데 이를 통해 빌드 툴로서의 동작을 수행하게 된다. 예를들면 아래과 같은.  Target 은 다른 Target 을 참조할수도 있는데 그에 따라 Target 간의 의존성 형성인 다른 Target 에서 나온 아웃풋을 활용해서 Target 을 실핼할수도 있다

아래 예제에서 우리는 cmd 라는 **target** 을 선언하고 있으며 go_binary 라는 **rule** 을 사용했다. 

선언된 **target 은 package** 에서 go binary 파일을 생성한다 

```
go_binary(
    name = "cmd",
    embed = [":cmd_lib"],
    visibility = ["//visibility:public"],
)
```

Java 개발자들은 많이 사용하는 gradle 과 비교할때 특정 모듈의 플러그인을 사용하여 특정 task 수행할떄 아래와 같은 명령어를 실행

```
gradle :app:clean
```

## Rule

Target 이 what 을 담당하고 있다면 rule 은 how 를 담당한다고 볼수 있다. Target 섹션에서 go_binary 는 이미 다른 라이브러리에서 정의된 rule 이며 이 rule 을 사용함으로써 우리는 하나의 Target 을 생성하였다. 

Rule 내부에서 하는 행동들을 Bazel 에서는 action 이라고 칭한다. 

Bazel 확장성의 핵심을 담당하고 있고 오픈소스 혹은 bazel 자체에 현재 지원하려는 언어의 rule 이 없더라도 사용자가 임의로 프로그래밍에서 함수를 선언하듯 rule 을 구현해서 프로젝트내에서 공유해서 사용가능하다. 

## **Label**

라벨은 우리가 명령어를 실행하기 위한 경로라고 보면 된다. 위에서 작성한 Target 을 실행하기 위해서는 

bazel 프로젝트의 package (BUILD 파일이 포함된 폴더) 까지의 경로를 주고 package 안의 

`bazel <bazel 동작 명령어> //<package 경로>:<target>`

e.g)  Package 에서 보여준 폴더구조에 a-service 

```
bazel build //services/a-service/cmd:cmd 
           <-- package 경로 --> <- target ->
```

## Macro

특정 빌드 프로세스를 위헤 **기존에 있는 Rule 들을  묵어서  재사용하게 만들고 싶은 경우**는 어떻게 해야될까? 그런 경우를 위해 Macro 가 있다.

Docker 이미지 만들고 푸시하는 과정을 예로 들어보면. Bazel 의 관점에서 실제 이미지를 빌드하는 과정과 이미지를 푸시하는 별도의 동작들이고 각각의 rule 을 실행하여 동작한다. 하지만 이 **두 동작을 묶어서 하나의 동작을 만들고 싶다면** 두가지 방법이 있을수 있다. 한가지는 위에서 이야기한 rule 을 만들어서 두가지 동작을 바닥부터 만드는 방법과 다른 한가지는 이미 기존에 있는 이미지 빌드하는 rule 들을 묶는 방법이 있다. macro 는 그러한 

# FAQ

1. Bazel 은 무슨언어를 사용하는가?
    1. Python 을 기반으로 하는 https://github.com/bazelbuild/starlark/ 를 사용한다
2. Gazelle 은 무엇인가?
    1. 위의 Package 개념에서 보면 알겠지만 프로젝트를 빌드하기 위해서는 각각의 프로젝트들마다 BUILD 파일들을 선언해서 프로젝트들로 Package 구성해서 Bazel 에게 알려줘야한다. 사용자가 손수 작성할수도 있지만 프로젝트 크기가 커질수록 양이 상당해진다. 이 BUILD 파일들을 자동 생성해주는 도구가 Gazelle.

# Useful resources

1. [Custom rule 만드는 방법](https://www.youtube.com/watch?v=toPWLiUq5Ps)

[Bazel runbook](https://www.notion.so/Bazel-runbook-ba44ad0985ad4e37a6af261b328b731c?pvs=21)