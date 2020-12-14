*Read this in other languages: [한국어](https://github.com/hermitkim1/golang-ci-demo-by-github-actions/blob/master/README.md), [English](https://github.com/hermitkim1/golang-ci-demo-by-github-actions/blob/master/README.EN.md)*

# Golang CI Demo by GitHub Actions

> (번역) CI (Continuous Integration)는 여러 기여자의 코드 변경 사항을 단일 소프트웨어 프로젝트로 통합하는 것을 자동화하는 방법입니다. - *[Atlassian](https://www.atlassian.com/)*

오픈소스 프로젝트에서 여러 컨트리뷰터(Contributor)가 함께 소프트웨어를 만들어 갑니다. 여러 사람이 기능 개선을 위해 소스코드를 수정하는데 몇 명의 메인테이너(Maintainer)들이 모든 컨트리뷰션(Contribution)을 일일히 대응하는 것은 어렵습니다. 따라서, 소스코드 통합을 자동화하는 CI는 오픈소스 프로젝트에 없어서는 안될 매우 중요한 요소 입니다.

먼저, CI 관련 개념을 설명한 후, **"GitHub Actions"를** 활용하여 Golang CI 하는 방법을 설명하겠습니다. Golang에 CI 적용할 때 도움이 되기를 바랍니다. 

GitHub Actions가 무료라서 좋네요 :smile:

## CI 관련 개념 설명
GitHub Actions을 활용한 Golang CI를 시작하기에 앞서 몇가지 개념을 설명하고자 합니다.   
(모두 아는 내용이라면 **Jump to [GitHub Actions을 통한 Golang CI 시작하기](https://github.com/hermitkim1/golang-ci-demo-by-github-actions#github-actions%EC%9D%84-%ED%86%B5%ED%95%9C-golang-ci-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)**) :wink:

### Lint 또는 Linter란?
> 린트(lint) 또는 린터(linter)는 소스 코드를 분석하여 **<ins>프로그램 오류, 버그, 스타일 오류, 의심스러운 구조체에</ins>** 표시(flag)를 달아놓기 위한 도구들을 가리킨다. 이 용어는 C 언어 소스 코드를 검사하는 유닉스 유틸리티에서 기원한다. - *[린트 (소프트웨어) from Wikipedia](https://ko.wikipedia.org/wiki/%EB%A6%B0%ED%8A%B8_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4))*

**<ins>쉽게 말해, 린트는 코딩 컨벤션(Coding convention)과 에러를 체크해 주는 프로그램 입니다.</ins>** 아래와 같은 사항도 체크해주니 탄탄한 SW 개발을 위해서 꼭 적용해야 할 것 같네요 :wink:

[참고] Staticcheck's [Checks](https://staticcheck.io/docs/checks):
- Code simplifications
- Various misuses of the standard library
- Concurrency issues
- Testing issues
- Code that isn't really doing anything
- Correctness issues
- Performance issues
- Dubious code constructs that have a high probability of being wrong
- Stylistic issues

저는 제 저장소에 린트를 적용하고, 아래와 같은 사항을 수정할 수 있었습니다.

- Write() function 호출 후 error(예외처리)처리 누락
- 비어 있는 if 또는 else 구문
- 불필요한 the blank identifier의 사용(예, _ )

### `matrix`란?
**<ins>`matrix`는 동시에 여러 환경에서 테스트를 진행하고 싶을 때 유용하게 쓸 수 있는 GitHub Actions의 Workflow을 위한 YAML syntax(문법)입니다.</ins>**

예를 들어, 개발한 프로그램을 Golang 1.13.x, 1.15.x 두 버전으로 ubuntu-18.04, macos-latest, windows-latest 세가지 OS 상에서 테스트 하고 싶은 경우 아래 예제 1과 같이 `matrix`를 사용하면 편리합니다.

`strategy`아래에 `matrix`를 선언한 후, 사용시 `${{matrix.os}}`, `${{matrix.go-servion}}`과 같이 사용할 수 있습니다. 자세한 사용방법은 아래 예제 1, 2를 참고 바랍니다.

예제 1:
```yaml
    strategy:
      matrix:
        go-version: [1.13.x, 1.15.x]
        os: [ubuntu-18.04, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
```

예제 2:
```yaml
strategy:
  matrix:
    node: [6, 8, 10]
steps:
  # Configures the node version used on GitHub-hosted runners
  - uses: actions/setup-node@v1
    with:
      # The Node.js version to configure
      node-version: ${{ matrix.node }}
```


## GitHub Actions을 통한 Golang CI 시작하기
1. Test codes 준비
2. GitHub Actions에서 CI workflow 설정

### Test codes 준비
[여기](http://golang.site/go/article/115-Go-%EC%9C%A0%EB%8B%9B-%ED%85%8C%EC%8A%A4%ED%8A%B8)를 참고 바랍니다.

가정: 아래 Sum function을 이미 가지고 있고, 이 코드를 단위 테스트(Unit test)하기를 원함

**`calc.go`**
```go
package calc
 
// Sum -
func Sum(a ...int) int {
    sum := 0
    for _, i := range a {
        sum += i
    }
    return sum
}
```

단위 테스트 하기 위해서, `xxx_test.go` file이 필요합니다. 여기서는 `calc_test.go`을 생성했습니다.

**`calc_test.go`**
```
package calc_test
 
import (
    "calc"
    "testing"
)
 
func TestSum(t *testing.T) {
    s := calc.Sum(1, 2, 3)
 
    if s != 6 {
        t.Error("Wrong result")
    }
}
```

### GitHub Actions에서 CI workflow 설정
GitHub Actions에서 제공하는 스타터 워크플로우를 사용하였고, 이해가 쉽도록 주석을 추가하였습니다.

```yaml
# The name of your workflow. GitHub displays the names of your workflows on your repository's actions page. 
# If you omit name, GitHub sets it to the workflow file path relative to the root of the repository.
name: Golang CI

# This workflow is triggered on push to default branch and on pull reqeusts to default branch.
on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]

jobs:
  # Set the job key. The key is displayed as the job name
  # when a job name is not provided
  # The job key is "build-and-test"
  build-and-test:
  
    # Job name is "Build and test"
    name: Build and test
    
    # This job runs on Ubuntu-latest
    runs-on: ubuntu-latest
    
    steps:
      # This action sets up a go environment for use in actions by:
      #     optionally downloading and caching a version of Go by version and adding to PATH
      #     registering problem matchers for error output
      # This step uses GitHub's setup-go: https://github.com/actions/setup-go
      - name: Set up Go 1.x
        uses: actions/setup-go@v2
        with:
          go-version: ^1.13
        id: go
        
      # This action checks-out your repository under $GITHUB_WORKSPACE, so your workflow can access it.
      # This step uses GitHub's checkout: https://github.com/actions/checkout
      - name: Check out code into the Go module directory
        uses: actions/checkout@v2

      # This step installs dependencies
      - name: Get dependencies
        run: |
          go get -v -t -d ./...
          if [ -f Gopkg.toml ]; then
              curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
              dep ensure
          fi
          
      # This step builds source codes
      - name: Build
        run: go build -v .

      # This step performs unit tests specified.
      - name: Test
        run: go test -v .
```

## 데모 순서
1. 이 저장소를 fork함
2. 브랜치를 생성함
3. `calc.go`를 수정함
4. Pull Request (PR)을 생성함

## 향상된 워크플로우
Cloud-Barista의 CB-Larva 저장소에서 아래 lint 워크 플로우를 테스트함(2020년 12월 11일)

### lint
```yaml
# The name of your workflow. GitHub displays the names of your workflows on your repository's actions page. 
# If you omit name, GitHub sets it to the workflow file path relative to the root of the repository.
name: lint-on-push

# This workflow is triggered on push
on: 
  push

jobs:
  # Set the job key. The key is displayed as the job name
  # when a job name is not provided
  # The job key is “lint"
  lint:
    # Job name is “xxx”
    name: Lint

    strategy:
      matrix:
        go-version: [1.15.x]
        os: [ubuntu-18.04] #macos-latest, windows-latest
    runs-on: ${{ matrix.os }}

    steps:
    - uses: actions/checkout@v2
    - name: golangci-lint
      uses: golangci/golangci-lint-action@v2
      with:
        # Required: the version of golangci-lint is required and must be specified without patch version: we always use the latest patch version.
        version: v1.29
        
        # Add args to skip dirs
        args: --skip-dirs poc-cb-net/archive

        # Optional: working directory, useful for monorepos
        # working-directory: somedir

        # Optional: golangci-lint command line arguments.
        # args: --issues-exit-code=0

        # Optional: show only new issues if it's a pull request. The default value is `false`.
        # only-new-issues: true
```

추가 예정입니다 :)
- On comment and contain specific words
- Code coverage
- [Default environment variables](https://docs.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables)
- [Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
- [Testing on all test files except for vendor packages](https://stackoverflow.com/questions/43507740/how-to-run-go-test-on-all-test-files-in-my-project-except-for-vendor-packages)
- [Commit comment](https://stackoverflow.com/questions/58468495/how-create-a-comment-on-commit-with-github-actions)
