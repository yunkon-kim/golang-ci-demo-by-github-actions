*Read this in other languages: [한국어](https://github.com/hermitkim1/golang-ci-demo-by-github-actions/blob/master/README.md), [English](https://github.com/hermitkim1/golang-ci-demo-by-github-actions/blob/master/README.EN.md)*

# Golang CI Demo by GitHub Actions

> (번역) CI (Continuous Integration)는 여러 기여자의 코드 변경 사항을 단일 소프트웨어 프로젝트로 통합하는 것을 자동화하는 방법입니다. - *[Atlassian](https://www.atlassian.com/)*

코드 변경 사항 통합을 자동화 하기 위하여 CI는 오픈소스 프로젝트에 꼭 적용해야합니다. 몇 명의 메인테이너(Maintainer)들이 모든 컨트리뷰션(Contribution)을 일일히 대응하기 어렵기 때문입니다. 

**"GitHub Actions"은** 여러분의 (공공) 저장소에 CI 적용하기 위해 큰 도움이 될 것이라고 생각합니다. 무료라서 좋네요 :smile:

## CI 관련 개념 설명
GitHub Actions을 활용한 Golang CI를 시작하기에 앞서 몇가지 개념을 설명하고자 합니다. 모두 아는 내용이라면 Jump to [GitHub Actions을 통한 Golang CI 시작하기](https://github.com/hermitkim1/golang-ci-demo-by-github-actions#github-actions%EC%9D%84-%ED%86%B5%ED%95%9C-golang-ci-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0) :wink:

### Lint 또는 Linter
Coming soon

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
name: Go

# This workflow is triggered on push to default branch and on pull reqeusts to default branch.
on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]

jobs:
  # Set the job key. The key is displayed as the job name
  # when a job name is not provided
  # The job key is "ci-demo"
  ci-demo:
  
    # Job name is "Build"
    name: Build
    
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
추가 예정입니다 :)
- Linting
- Matrix
- On comment and contain specific words
- Test coverage
- [Default environment variables](https://docs.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables)
- [Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
- [Testing on all test files except for vendor packages](https://stackoverflow.com/questions/43507740/how-to-run-go-test-on-all-test-files-in-my-project-except-for-vendor-packages)
- [Commit comment](https://stackoverflow.com/questions/58468495/how-create-a-comment-on-commit-with-github-actions)
