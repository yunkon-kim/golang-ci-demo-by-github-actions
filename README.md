# Golang CI Demo by GitHub Actions

According to [Atlassian](https://www.atlassian.com/), *Continuous Integration (CI) is the practice of automating the integration of code changes from multiple contributors into a single software project.*

CI is specially mandatory for open source projects to automate the integration of all code changes because several maintainers couldn't handle all contributions one by one.

**"GitHub Actions"** is helpful to apply CI on your (public) repository. It's for free!!

## Getting started with Golang CI by GitHub Actions
1. Preparing test codes
2. Setting up a CI workflow on GitHub Actions

### Preparing test codes
Test code refers to [here](http://golang.site/go/article/115-Go-%EC%9C%A0%EB%8B%9B-%ED%85%8C%EC%8A%A4%ED%8A%B8)

Assume that I already have Sum function as follows and I want to unit test this source code.

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

To unit test, `xxx_test.go` file is necessary. I create `calc_test.go`

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

### Setting up a CI workflow on GitHub Actions
I used a starter workflow provided from GitHub Actions, and I added comments for better understanding.
```yaml
name: Go

# This workflow is triggered on push to default branch and on pull reqeusts to default branch.
on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]

jobs:
  build:
  
    # Job name is Build
    name: Build
    
    # this job runs on Linux (Ubuntu-latest)
    runs-on: ubuntu-latest
    
    steps:
      # This step uses GitHub's setup-go: https://github.com/actions/setup-go
      - name: Set up Go 1.x
        uses: actions/setup-go@v2
        with:
          go-version: ^1.13
        id: go
        
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

## Demonstaration steps
1. Fork this repository
2. Create a branch
3. Modify `calc.go`
4. Make a pull request

## Advanced workflows
To be added :)
- Linting
- Matrix
- On comment and contain specific words
- Test coverage
- [Default environment variables](https://docs.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables)
- [Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
- [Testing on all test files except for vendor packages](https://stackoverflow.com/questions/43507740/how-to-run-go-test-on-all-test-files-in-my-project-except-for-vendor-packages)
- [Commit comment](https://stackoverflow.com/questions/58468495/how-create-a-comment-on-commit-with-github-actions)
