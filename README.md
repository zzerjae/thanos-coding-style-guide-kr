## thanos-coding-style-guide-kr

- [Thanos](https://thanos.io/) 팀의 코드 스타일 가이드를 한국어로 번역했습니다.

- [8af5266](https://github.com/thanos-io/thanos/commit/8af526602ddc680c377d56f08c9d57a55553c191) 커밋까지의 문서를 번역하였습니다.

- 오타 및 매끄럽지 않은 번역에 대한 PR 환영합니다.

---

# 타노스 스타일 가이드(Thanos Coding Style Guide)

이 문서는 Thanos 프로젝트에서 사용하는 다양한 언어의 공식 스타일 가이드를 자세히 설명합니다. 이 내용을 숙지하고, 코드 리뷰 중에 이 문서를 참고하세요. 우리의 코드 베이스에서 스타일 가이드와 맞지 않는 것이 있다면, 가이드를 놓쳤거나, 이 문서가 쓰이기 이전에 작성된 코드입니다. 고치는데 도움을 주세요! (:

전체적으로, 아래를 주의합니다:

- 가독성, [인지 부하](https://www.dabapps.com/blog/cognitive-load-programming/)를 낮춥니다.
- 유지보수성. **놀라게**하는 코드를 피합니다.
- 성능은 오직 가독성을 헤치지 않는 선에서, 중요한 부분만 고려합니다.
- 테스트 용이성. 프로덕션 코드의 일부분이 변경되더라도 고려합니다. `timeNow func() time.Time`와 같은 경우도 `mock`합니다.
- 일관성: 어떤 패턴이 반복되면, 놀랄 경우가 줄어듭니다.

일부 스타일은 우리의 `linter`에 의해서 강제되고 별도의 작은 세션으로 구분됩니다. 규칙의 일부를 프로젝트에서 사용하고 싶다면, 각 세션을 살펴보세요! Thanos 개발자의 경우 개발 중에 수동으로 적용할 규칙에 대한 섹션을 읽는 것이 좋습니다. 일부 규칙은 현재 `linter`로는 감지되지 않습니다. 이상적으로는, 모든 규칙의 적용이 자동화될 것입니다. (:

## TOC

- [타노스 스타일 가이드(Thanos Coding Style Guide)](#------------thanos-coding-style-guide-)
  - [TOC](coding-style-guide.md#toc)
- [Go](#go)
  - [Development / Code Review](#development---code-review)
    - [Reliability](#reliability)
      - [Defers: Don't Forget to Check Returned Errors](#defers--don-t-forget-to-check-returned-errors)
      - [Exhaust Readers](#exhaust-readers)
      - [Avoid Globals](#avoid-globals)
      - [Never Use Panics](#never-use-panics)
      - [Avoid Using the `reflect` or `unsafe` Packages](#avoid-using-the--reflect--or--unsafe--packages)
      - [Avoid variable shadowing](#avoid-variable-shadowing)
    - [Performance](#performance)
      - [Pre-allocating Slices and Maps](#pre-allocating-slices-and-maps)
      - [Reuse arrays](#reuse-arrays)
    - [Readability](#readability)
      - [Keep the Interface Narrow; Avoid Shallow Functions](#keep-the-interface-narrow--avoid-shallow-functions)
      - [Use Named Return Parameters Carefully](#use-named-return-parameters-carefully)
      - [Clean Defer Only if Function Fails](#clean-defer-only-if-function-fails)
      - [Explicitly Handled Returned Errors](#explicitly-handled-returned-errors)
      - [Avoid Defining Variables Used Only Once.](#avoid-defining-variables-used-only-once)
      - [Only Two Ways of Formatting Functions/Methods](#only-two-ways-of-formatting-functions-methods)
      - [Control Structure: Prefer early returns and avoid `else`](#control-structure--prefer-early-returns-and-avoid--else-)
      - [Wrap Errors for More Context; Don't Repeat "failed ..." There.](#wrap-errors-for-more-context--don-t-repeat--failed---there)
      - [Use the Blank Identifier `_`](#use-the-blank-identifier----)
      - [Rules for Log Messages](#rules-for-log-messages)
      - [Comment Necessary Surprises](#comment-necessary-surprises)
    - [Testing](#testing)
      - [Table Tests](#table-tests)
      - [Tests for Packages / Structs That Involve `time` package.](#tests-for-packages---structs-that-involve--time--package)
  - [Enforced by Linters](#enforced-by-linters)
    - [Avoid Prints](#avoid-prints)
    - [Ensure Prometheus Metric Registration](#ensure-prometheus-metric-registration)
    - [go vet](#go-vet)
    - [golangci-lint](#golangci-lint)
    - [misspell](#misspell)
    - [Comments Should be Full Sentences](#comments-should-be-full-sentences)
- [Bash](#bash)

<small><i>Table of contents generated with <a href='http://ecotrust-canada.github.io/markdown-toc/'>markdown-toc</a></i></small>

# Go

[Go](https://golang.org/)로 작성된 코드에 대해서, 표준 Go 스타일 가이드([Effective Go](https://golang.org/doc/effective_go.html),
[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments))에 추가로 보다 엄격한 룰을 적용하여 사용합니다. 따라서 안정성, 성능 그리고 유지보수성 등이 매우 중요한 Thanos와 같은 분산 시스템 데이터베이스에서 일관성이 훨씬 향상됩니다.

## Development / Code Review

이 섹션에서는, 개발 및 코드 리뷰에 적용되는 표준 스타일 가이드에 추가로 다음의 규칙을 적용합니다.

NOTE: 아래의 규칙 중 하나라도 linter에 의해 자동으로 적용될 수 있는 것을 알고 있다면, 알려주세요! (:

### Reliability

코딩 스타일은 순전히 무엇이 더러워 보이거나 그렇지 않아 보이는 것에 대한 것은 아닙니다. 주로 사고 없이 24시간 프로덕션 환경에서 프로그램을 실행할 수 있도록 프로그램을 안정적으로 유지하는 것에 대해서 다룹니다. 다음의 규칙은 Go 커뮤니티에서 자주 잊혀지는 해로운 패턴에 대해서 설명합니다. 다음의 패턴들은 버그로 보여지거나, 버그를 유발할 상황을 상당히 증가시킬 수 있습니다.

#### Defers: Don't Forget to Check Returned Errors

- `defer` 처리된 `Close` 메서드에서 반환된 `error`를 확인하는 것을 잊어버릴 때가 있습니다.

```go
f, err := os.Open(...)
if err != nil {
    // handle..
}
defer f.Close() // 여기에서 에러가 발생한다면?

// Write something to file... etc.
```

이처럼 확인되지 않은 `error`는 중대한 버그를 유발할 수 있습니다. 위의 예시를 보세요. `*os.File.Close()` 메서드는 실제로 파일을 플러쉬(flush)하는 책임을 지고있으므로, 여기에 `error`가 발생한다면, **쓰기 작업 전체가 중단(abort)될 수 있습니다!** 😱

항상 `error`를 확인하세요! 일관성을 유지하면서 주의전환시키지 않으려면, [runutil](https://pkg.go.dev/github.com/thanos-io/thanos@v0.11.0/pkg/runutil?tab=doc) 같은 헬퍼 패키지를 사용하세요.

```go
// Use `CloseWithErrCapture` if you want to close and fail the function or
// method on a `f.Close` error (make sure thr `error` return argument is
// named as `err`). If the error is already present, `CloseWithErrCapture`
// will append (not wrap) the `f.Close` error if any.
defer runutil.CloseWithErrCapture(&err, f, "close file")

// Use `CloseWithLogOnErr` if you want to close and log error on `Warn`
// level on a `f.Close` error.
defer runutil.CloseWithLogOnErr(logger, f, "close file")
```

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
func writeToFile(...) error {
    f, err := os.Open(...)
    if err != nil {
        return err
    }
    defer f.Close() // What if an error occurs here?

    // Write something to file...
    return nil
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
func writeToFile(...) (err error) {
    f, err := os.Open(...)
    if err != nil {
        return err
    }
    // Now all is handled well.
    defer runutil.CloseWithErrCapture(&err, f, "close file")

    // Write something to file...
    return nil
}
```

</td></tr>
</tbody></table>

#### Exhaust Readers

가장 일반적인 버그 중 하나는 `error` 발생시에, HTTP 요청이나 응답의 `body`를 `close`하거나 완전히 `read`하는 것을 잊는 것입니다. 이러한 구조에서 `body`를 읽을 때 [runutil](https://pkg.go.dev/github.com/thanos-io/thanos@v0.11.0/pkg/runutil?tab=doc) 헬퍼 메서드를 사용할 수 있습니다.

```go
defer runutil.ExhaustCloseWithLogOnErr(logger, resp.Body, "close response")
```

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle...
}
defer runutil.CloseWithLogOnErr(logger, resp.Body, "close response")

scanner := bufio.NewScanner(resp.Body)
// If any error happens and we return in the middle of scanning
// body, we can end up with unread buffer, which
// will use memory and hold TCP connection!
for scanner.Scan() {
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle...
}
defer runutil.ExhaustCloseWithLogOnErr(logger, resp.Body, "close response")

scanner := bufio.NewScanner(resp.Body)
// If any error happens and we return in the middle of scanning body,
// defer will handle all well.
for scanner.Scan() {
```

</td></tr>
</tbody></table>

#### Avoid Globals

`const`를 제외한 전역 변수는 사용하지 않습니다. 이 것은 `init` 함수 또한 사용하지 않는 것을 의미합니다.

#### Never Use Panics

`panic`을 사용하지 마세요. 의존하는 패키지에서 `panic`을 사용한다면 `recover`를 사용하세요. 또한, 패닉을 일으키는 그 패키지를 사용하지 마세요. 🙈

#### Avoid Using the `reflect` or `unsafe` Packages

매우 구체적이고, 중요한 경우에만 사용하세요. 특히 `reflect`는 매우 느린 경향이 있습니다. 테스트 코드에 한해서, `reflect`를 사용하는 것이 좋습니다.

#### Avoid variable shadowing

변수 그림자 만들기는 더 작은 범위(scope)에서 같은 변수 명을 사용하는 것을 말합니다. 이 것은 매우 위험하고 많은 사람을 놀라게 할 수 있습니다. 관련 없는 코드에서 변수 그림자가 나타날 때, 이 것은 디버깅하기 매우 어렵습니다. 단지 `:`가 있거나 빠졌거나 할 때, 오류가 발생할 수 있습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
    var client ClientInterface
    if clientTypeASpecified {
        // Ups - typo, should be =`
        client, err := clienttypea.NewClient(...)
        if err != nil {
            // handle err
        }
        level.Info(logger).Log("msg", "created client", "type", client.Type)
    } else {
        // Ups - typo, should be =`
         client, err := clienttypea.NewClient(...)
         level.Info(logger).Log("msg", "noop client will be used", "type", client.Type)
    }

    // In some further deeper part of the code...
    resp, err := client.Call(....) // nil pointer panic!
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
    var client ClientInterface = NewNoop(...)
    if clientTypeASpecified {
        c, err := clienttypea.NewClient(...)
        if err != nil {
            // handle err
        }
        client = c
    }
    level.Info(logger).Log("msg", "created client", "type", c.Type)

    resp, err := client.Call(....)
```

</td></tr>
</tbody></table>

이런 문제 때문에, 할 수 있으면, 우리는 에러의 범위를 제한하는 것을 추천합니다.

```go
    if err := doSomething(); err != nil {
        // handle err
    }
```

아직 설정되지 않았지만, 나중에 우리는 [golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow](https://godoc.org/golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow)를 사용하여 변수 그림자 만들기를 허용하지 않을 생각입니다. 언어 자체에서 이 것을 방지하기 위한 [Go 2 proposal](https://github.com/golang/go/issues/21114)이 있었지만 받아들여지지 않았습니다.

패키지 이름에도 그림자가 생길 수 있습니다. 위험은 덜하지만, 비슷한 문제가 생길 수 있으므로, 가능하면 같은 패키지 이름을 사용하는 것을 피하세요.

### Performance

어쨌든, Thanos system은 사람들이 용인하는 응답 시간내에서, 수 테라바이트 크기의 데이터에 쿼리를 수행하는 데이터베이스 입니다. 이는 우리 코드에 추가적인 패턴을 요구합니다. 가독성을 해치지 않는 선에서, 중요한 부분에만 추가적인 패턴을 적용합니다.

**결과를 항상 측정해야 한다는 것을 명심하세요.** Go의 성능은 많은 숨겨진 것들과 조정(tweak)에 의존하기 때문에, 최적화가 적절한 것인지 판단하는 데에는, 실제 시스템에 로드 테스트를 해서 얻는 세밀한 벤치마크가 필요합니다.

#### Pre-allocating Slices and Maps

항상 미리 할당된 `slice`와 `map`을 사용하세요. 집어 넣을 요소의 갯수를 미리 알고 있다면 이를 사용하세요. 미리 할당하는 것은 이러한 경우에 지연을 많이 줄여줍니다. 세밀한 최적화의 측면에서, 복잡성을 거의 증가시키지 않으므로, 항상 이 것을 수행하는 것이 좋습니다. 성능 측면에서 보자면, 배열이 큰 결정적인 코드에서만 관련이 있습니다.

NOTE: 단순하게 말하자면, Go 런타임은 현재 크기의 2배를 할당합니다. 그래서 미리 할당하는 작업을 하면, 수 백만개의 요소를 사용할 것이 예상될때, Go는 요소가 추가될 때, 하나가 아닌 많은 수를 할당합니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
func copyIntoSliceAndMap(biggy []string) (a []string, b map[string]struct{})
    b = map[string]struct{}{}

    for _, item := range biggy {
        a = append(a, item)
        b[item] = struct{}
    }
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
func copyIntoSliceAndMap(biggy []string) (a []string, b map[string]struct{})
    b = make(map[string]struct{}, len(biggy))
    a = make([]string, len(biggy))

    // Copy will not even work without pre-allocation.
    copy(a, biggy)
    for _, item := range biggy {
        b[item] = struct{}
    }
}
```

</td></tr>
</tbody></table>

#### Reuse arrays

위에 더해서, 항상 메모리에 새로운 공간을 할당할 필요가 없는 경우가 있습니다. `slice`에 대하여 특정한 연산을 순차적으로 반복 수행하고, 매 반복마다 해당 배열의 메모리 공간을 놓아준다면, 이러한 배열의 메모리를 재사용하는 것이 합당합니다. 중요한 로직에서 이 것은 꽤 많은 이익을 줄 수 있습니다.
불행히도, 현재 `map`의 내부에서 사용하는 `array`를 재사용하는 방법은 없습니다.

NOTE: 왜 `slice`를 할당하고 놓아주고, 새로운 반복에서 이 것을 또 할 수 없을 까요? Go가 사용가능한 메모리 공간을 보고 이 것을 재사용할 수 있어야하는 것 아닐까? (: 음, 이건 그렇게 쉬운 이야기가 아닙니다. 짧게 이야기 해보자면, Go의 가비지 컬렉션은 주기적, 혹은 특정 경우(큰 heap이 해제 될때)에 실행 되지만, 매 `loop`가 반복되는 경우에 동작하는 것은 아닙니다.(이렇게 되면 매우 매우 느릴 것입니다.). 보다 자세한 내용은 [여기](https://about.sourcegraph.com/go/gophercon-2018-allocator-wrestling)를 읽어보세요.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
var messages []string{}
for _, msg := range recv {
    messages = append(messages, msg)

    if len(messages) > maxMessageLen {
        marshalAndSend(messages)
        // This creates new array. Previous array
        // will be garbage collected only after
        // some time (seconds), which
        // can create enormous memory pressure.
        messages = []string{}
    }
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
var messages []string{}
for _, msg := range recv {
    messages = append(messages, msg)

    if len(messages) > maxMessageLen {
        marshalAndSend(messages)
        // Instead of new array, reuse
        // the same, with the same capacity,
        // just length equals to zero.
        messages = messages[:0]
    }
}
```

</td></tr>
</tbody></table>

### Readability

모든 Gopher들이 사랑하는 부분입니다. ❤️ 어떻게 보다 읽기 좋은 코드를 만들까요?

Thanos 팀에 있어서, 가독성이란 것은 코드를 읽는 사람을 놀라게 하지 않는 방법으로 프로그래밍하는 것 입니다. 모든 디테일과 일관성이 없어 보이는 것은 독자를 혼란스럽게 하거나 독자를 잘못된 방향으로 이끌 수 있으므로, 모든 문자나 줄 바꿈이 중요할 수 있습니다. 그렇기에 우리는 모든 풀리퀘의 검토에 시간을 많이 투자합니다. 특히 초기에는 더욱 그러합니다. 하지만 이는 시스템을 더 빨리 이해하고 확장할 수 있도록 도와주며, 우리 시스템의 문제를 고칠 수 있도록 합니다.

#### Keep the Interface Narrow; Avoid Shallow Functions

이 것은 코딩보다는 API 설계에 더 치우친 이야기지만, 작은 코드 부분을 결정할 때도 중요합니다. 예를 들어, 함수 혹은 메서드를 정의하는 방법이 있습니다. 두가지 일반적인 규칙이 있습니다.

- 간단한 (보통 더 작은) 인터페이스가 좋습니다. 이 것은 인터페이스에서 더 작고 간단한 함수 선언부 뿐만 아니라 더 적은 수의 메서드를 의미할 수 있습니다. 기능에 따라 인터페이스를 그룹화하여 가능한 겨우 최대 1개에서 3개의 메서드만 노출하세요.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
// Compactor aka: The Big Boy. Such big interface is really useless ):
type Compactor interface {
    Compact(ctx context.Context) error
    FetchMeta(ctx context.Context) (metas map[ulid.ULID]*metadata.Meta, partial map[ulid.ULID]error, err error)
    UpdateOnMetaChange(func([]metadata.Meta, error))
    SyncMetas(ctx context.Context) error
    Groups() (res []*Group, err error)
    GarbageCollect(ctx context.Context) error
    ApplyRetentionPolicyByResolution(ctx context.Context, logger log.Logger, bkt objstore.Bucket) error
    BestEffortCleanAbortedPartialUploads(ctx context.Context, bkt objstore.Bucket)
    DeleteMarkedBlocks(ctx context.Context) error
    Downsample(ctx context.Context, logger log.Logger, metrics *DownsampleMetrics, bkt objstore.Bucket) error
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
// Smaller interfaces with a smaller number of arguments allow functional grouping, clean composition and clear testability.
type Compactor interface {
    Compact(ctx context.Context) error

}

type Downsampler interface {
    Downsample(ctx context.Context) error
}

type MetaFetcher interface {
    Fetch(ctx context.Context) (metas map[ulid.ULID]*metadata.Meta, partial map[ulid.ULID]error, err error)
    UpdateOnChange(func([]metadata.Meta, error))
}

type Syncer interface {
    SyncMetas(ctx context.Context) error
    Groups() (res []*Group, err error)
    GarbageCollect(ctx context.Context) error
}

type RetentionKeeper interface {
    Apply(ctx context.Context) error
}

type Cleaner interface {
    DeleteMarkedBlocks(ctx context.Context) error
    BestEffortCleanAbortedPartialUploads(ctx context.Context)
}
```

</td></tr>
</tbody></table>

- 사용자에게 불필요한 복잡성을 숨길 수 있다면, 더 좋습니다. 이 것은 얕은 함수를 사용하는 것이 함수를 이해하는데 사용자에게 더 많은 인지 부하를 일으키고, 이해하기 위해 구현부를 쫓아가야 하게 되는 것을 의미합니다. 대신에 그 함수의 내용을 호출부에 직접 추가하는 것이 더 읽기 쉽습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
    // Some code...
    s.doSomethingAndHandleError()

    // Some code...
}

func (s *myStruct) doSomethingAndHandleError() {
    if err := doSomething(); err != nil {
        level.Error(s.logger).Log("msg" "failed to do something; sorry", "err", err)
    }
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
    // Some code...
    if err := doSomething(); err != nil {
        level.Error(s.logger).Log("msg" "failed to do something; sorry", "err", err)
    }

    // Some code...
}
```

</td></tr>
</tbody></table>

이 것은 `명백하게 그 것을 수행하는 방법이 하나의(바람직하게는 오직 하나만) 있어야 한다.`와 [`DRY`](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 규칙에 좀 관련이 있습니다. 어떤 작업을 수행하는데, 하나 이상의 많은 방법을 가지고 있다면, 더 넓은 인터페이스를 가지고 있다는 뜻이고, 이 것은 모호함과 오류를 발생시키고, 유지보수 하는데 따르는 짐을 더 만들어냅니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
// We have here SIX potential how caller can get an ID. Can you find all of them?

type Block struct {
    // Things...
    ID ulid.ULID

    mtx sync.Mutex
}

func (b *Block) Lock() {  b.mtx.Lock() }

func (b *Block) Unlock() {  b.mtx.Unlock() }

func (b *Block) ID() ulid.ULID {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.ID
}

func (b *Block) IDNoLock() ulid.ULID {  return b.ID }
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
type Block struct {
    // Things...

    id ulid.ULID
    mtx sync.Mutex
}

func (b *Block) ID() ulid.ULID {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.id
}
```

</td></tr>
</tbody></table>

#### Use Named Return Parameters Carefully

어떤 메서드나 함수가 실제로 응답한 것에 대해서 타입이 충분한 정보를 주지 못할 때는, 이름이 있는 리턴 파라미터를 사용해도 괜찮습니다. 또다른 유즈 케이스는 `slice` 같은 변수를 정의하는 경우 입니다.

**IMPORTANT:** 이름 있는 리턴 파라미터와 함께 노출된 `return`문을 사용하지 마세요. 이 것은 컴파일은 되지만, 반환 값을 암시적으로 만들어서, 놀라게 할 수 있습니다.

#### Clean Defer Only if Function Fails

각각의 `error`가 발생했을 때 모두 올바르게 `close`하기 위해서 `defer`를 희생하는 방법이 있습니다. 무언가를 반복한다는 것은 에러를 만들기 쉽고, 코드를 변경할 때 그 것을 까먹기 쉽습니다. 따라서, `defer`를 사용해서 `error` 발생시 필요한 동작을 처리할 수 있습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
func OpenSomeFileAndDoSomeStuff() (*os.File, error) {
    f, err := os.OpenFile("file.txt", os.O_RDONLY, 0)
    if err != nil {
        return nil, err
    }

    if err := doStuff1(); err != nil {
        runutil.CloseWithErrCapture(&err, f, "close file")
        return nil, err
    }
    if err := doStuff2(); err != nil {
        runutil.CloseWithErrCapture(&err, f, "close file")
        return nil, err
    }
    if err := doStuff232241(); err != nil {
        // Ups.. forgot to close file here.
        return nil, err
    }
    return f, nil
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
func OpenSomeFileAndDoSomeStuff() (f *os.File, err error) {
    f, err = os.OpenFile("file.txt", os.O_RDONLY, 0)
    if err != nil {
        return nil, err
    }
    defer func() {
        if err != nil {
             runutil.CloseWithErrCapture(&err, f, "close file")
        }
    }

    if err := doStuff1(); err != nil {
        return nil, err
    }
    if err := doStuff2(); err != nil {
        return nil, err
    }
    if err := doStuff232241(); err != nil {
        return nil, err
    }
    return f, nil
}
```

</td></tr>
</tbody></table>

#### Explicitly Handled Returned Errors

항상 반환되는 에러를 처리하세요. 이 것은 에러를 무시할 수 없다는 것을 의미하지 않습니다. 예를 들어, 구현이 아무것도 리턴하지 않을 것이라는 것을 아는 것은 의미있습니다. 에러를 무시할 수 있습니다. 하지만 명백하게 그 것을 보여주세요.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
someMethodThatReturnsError(...)
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
_ = someMethodThatReturnsError(...)
```

</td></tr>
</tbody></table>

예외: 잘 알려진 `level.Debug|Warn` 나 `fmt.Fprint*` 종류의 것들.

#### Avoid Defining Variables Used Only Once.

뭔가 더 큰 것을 만들기 위해, 간간히 사용되는 단계에서 변수를 정의하고 싶은 생각이 듭니다. 오직 한 번만 사용될 때, 이러한 변수를 정의하는 것을 피하세요. 당신이 변수를 정의할 때, *독자*는 보통 그 변수에 대해서 한 번 이상의 어떤 사용처가 있을 것이라고 예상하게 되므로, 독자는 매번 다시 확인하면서 그 변수가 한 번만 사용된다는 것을 깨달아야 합니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
    someConfig := a.GetConfig()
    address124 := someConfig.Addresses[124]
    addressStr := fmt.Sprintf("%s:%d", address124.Host, address124.Port)

    c := &MyType{HostPort: addressStr, SomeOther: thing}
    return c
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
    // This variable is required for potentially consistent results. It is used twice.
    someConfig := a.FetchConfig()
    return &MyType{
        HostPort:  fmt.Sprintf("%s:%d", someConfig.Addresses[124].Host, someConfig.Addresses[124].Port),
        SomeOther: thing,
    }
```

</td></tr>
</tbody></table>

#### Only Two Ways of Formatting Functions/Methods

함수/메서드 정의부에서 한 줄로 인자를 쓰도록 하세요. 너무 길다면 각 인수를 한 줄마다 두세요.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
func function(argument1 int, argument2 string,
    argument3 time.Duration, argument4 someType,
    argument5 float64, argument6 time.Time,
) (ret int, err error) {
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
func function(
    argument1 int,
    argument2 string,
    argument3 time.Duration,
    argument4 someType,
    argument5 float64,
    argument6 time.Time,
) (ret int, err error)
```

</td></tr>
</tbody></table>

이 것을 함수/메서드를 호출하는데에도 적용됩니다.

NOTE: 가변의 (`...string`) 인자가 쌍으로 채워질 것이 예상되는 경우에는 예외입니다.

```go
level.Info(logger).Log(
    "msg", "found something epic during compaction; this looks amazing",
    "compNumber", compNumber,
    "block", id,
    "elapsed", timeElapsed,
)
```

#### Control Structure: Prefer early returns and avoid `else`

대부분의 경우에 `else`는 필요 없습니다. `if` 블록에서는 `continue`, `break`, `return`을 끝에서 사용할 수 있습니다.
이 것은 들여쓰기를 하나 줄여줘서, 일관성을 유지해주고 읽기 쉽게 해줍니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
for _, elem := range elems {
    if a == 1 {
        something[i] = "yes"
    } else
        something[i] = "no"
    }
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
for _, elem := range elems {
    if a == 1 {
        something[i] = "yes"
        continue
    }
    something[i] = "no"
}
```

</td></tr>
</tbody></table>

#### Wrap Errors for More Context; Don't Repeat "failed ..." There.

우리는 `errors`를 위해 [`pkg/errors`](https://github.com/pkg/errors) 패키지를 사용합니다. 우리는 표준의 에러 래핑인 `fmt.Errorf` + `%w`보다 `pkh/errors` 방식을 선호합니다. `errors.Wrap`이 보다 명시적이기 때문입니다. 실수로 `%w` 를 `%v` 로 바꾸거나 일관되지 않은 문자열 방식을 사용하는 것이 흔하기 때문입니다.

[`pkg/errors.Wrap`](https://github.com/pkg/errors)를 사용해서 오류가 발생할 때, 이후의 컨텍스트에 대해서 오류를 래핑하세요. 보다 정보가 있는 변수를 컨텍스트에 담기 위해서는 `errors.Wrapf`를 사용하세요. 예로, file names, IDs 또는 실패할 수 있는 다른 것들이 있습니다.

NOTE: `failed ...`나 `error occurred while...` 같은 문구를 에러 메시지에 접두사로 붙이지 마세요. 그 접두사는 단지 노이즈입니다. 우리는 에러를 래핑하는 중이므로, 에러가 발생한 것은 이미 분명합니다. 그렇죠? (: 가독성을 높이기 위해 이런 것은 피하세요.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
if err != nil {
    return fmt.Errorf("error while reading from file %s: %w", f.Name, err)
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
if err != nil {
    return errors.Wrapf(err, "read file %s", f.Name)
}
```

</td></tr>
</tbody></table>

#### Use the Blank Identifier `_`

공백 식별자는 사용하지 않는 변수를 표시하는데 유용합니다. 다음과 같은 경우를 확인하세요.

```go
// We don't need the second return parameter.
// Let's use the blank identifier instead.
a, _, err := function1(...)
if err != nil {
    // handle err
}
```

```go
// We don't need to use this variable, we
// just want to make sure TypeA implements InterfaceA.
var _ InterfaceA = TypeA
```

```go
// We don't use context argument; let's use the blank
// identifier to make it clear.
func (t *Type) SomeMethod(_ context.Context, abc int) error {
```

#### Rules for Log Messages

Thanos에서는 [go-kit logger](https://github.com/go-kit/kit/tree/master/log)를 사용합니다. 이는 log 코드가 특정한 구조체를 사용하는 것을 의미합니다. 이 것은 메시지에 변수를 추가하는 대신 별도의 필드를 전달하는 것을 의미합니다. Thanos의 모든 log는 `lowercase`를 사용하고 모든 구조체의 `key`는 `camelCase`를 사용합니다(가독성과 일관성). `key`의 이름은 짧고 일관되야 합니다. 예를 들어 항상 우리는 block ID 에는 `block`을 사용하고, 다른 단일 로그 메시지 `id`에는 사용하지 않습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
level.Info(logger).Log("msg", fmt.Sprintf("Found something epic during compaction number %v. This looks amazing.", compactionNumber),
 "block_id", id, "elapsed-time", timeElapsed)
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
level.Info(logger).Log("msg", "found something epic during compaction; this looks amazing", "compNumber", compNumber,
"block", id, "elapsed", timeElapsed)
```

</td></tr>
</tbody></table>

추가적으로, 여러 로그 레벨을 사용할 때, 제안하는 규칙이 있습니다.

- level.Info: `msg` 필드가 항상 있어야 합니다. 자주 발생하지 않을 중요한 이벤트에만 사용해야 합니다.
- level.Debug: `msg` 필드가 항상 있어야 합니다. 좀 더 스팸성일 수 있지만, 모든 곳에 존재해서는 안됩니다. 특정한 영역에서 실제로 문제에 뛰어들어야 할때만 사용하세요.
- level.Warn: `msg`, `err` 또는 둘 다 항상 있어야 합니다. 의심스러운 이벤트를 경고하고, 이를 조사해야 합니다. 하지만 프로세스는 정상적으로 이벤트를 처리할 수 있습니다. 항상 이벤트가 _어떻게_ 처리될지, 어떤 조치가 수행되는지 설명하세요. 예를 들어, `value will be skipped`
- level.Error: `msg`, `err` 또는 둘 다 항상 있어야 합니다. 크리티컬한 이벤트에만 사용하세요.

#### Comment Necessary Surprises

코멘트가 최선의 것은 아닙니다. 그것들은 빠르게 뒤쳐지고, 컴파일러는 업데이트하는 것을 잊어버려도 실패하지 않습니다. 따라서 필요한 경우에만 코멘트를 사용하세요. **그리고 사용자를 놀라게 할 수 있는 코드에는 주석을 달아야 합니다.** 때로는, 성능을 위해 복잡성이 필요할 수 있습니다. 이 경우 왜 그러한 최적화라 필요한지 코멘트를 작성하세요. 무언가 일시적으로 수행한 것이 있다면 다음처럼 코멘트를 작성하세요. `TODO(<github name>): <something, with GitHub issue link ideally>`.

### Testing

#### Table Tests

가독성을 위해 [t.Run](https://blog.golang.org/subtests)을 사용하는 테이블 스타일의 테스트를 사용하세요. 읽기 쉽고 각 테스트 사례에 대해 명확한 설명을 추가할 수 있습니다. 추가하거나 수정하는 것도 쉽습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
host, port, err := net.SplitHostPort("1.2.3.4:1234")
testutil.Ok(t, err)
testutil.Equals(t, "1.2.3.4", host)
testutil.Equals(t, "1234", port)

host, port, err = net.SplitHostPort("1.2.3.4:something")
testutil.Ok(t, err)
testutil.Equals(t, "1.2.3.4", host)
testutil.Equals(t, "http", port)

host, port, err = net.SplitHostPort(":1234")
testutil.Ok(t, err)
testutil.Equals(t, "", host)
testutil.Equals(t, "1234", port)

host, port, err = net.SplitHostPort("yolo")
testutil.NotOk(t, err)
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
for _, tcase := range []struct{
    name string

    input     string

    expectedHost string
    expectedPort string
    expectedErr error
}{
    {
        name: "host and port",

        input:     "1.2.3.4:1234",
        expectedHost: "1.2.3.4",
        expectedPort: "1234",
    },
    {
        name: "host and named port",

        input:     "1.2.3.4:something",
        expectedHost: "1.2.3.4",
        expectedPort: "something",
    },
    {
        name: "just port",

        input:     ":1234",
        expectedHost: "",
        expectedPort: "1234",
    },
    {
        name: "not valid hostport",

        input:     "yolo",
        expectedErr: errors.New("<exact error>")
    },
}{
    t.Run(tcase.name, func(t *testing.T) {
        host, port, err := net.SplitHostPort(tcase.input)
        if tcase.expectedErr != nil {
            testutil.NotOk(t, err)
            testutil.Equals(t, tcase.expectedErr, err)
            return
        }
        testutil.Ok(t, err)
        testutil.Equals(t, tcase.expectedHost, host)
        testutil.Equals(t, tcase.expectedPort, port)
    })
}
```

</td></tr>
</tbody></table>

#### Tests for Packages / Structs That Involve `time` package.

실제 시간에 의존하는 유닛 테스트를 하지 마세요. 항상 `timeNow func() time.Time` 필드를 사용해서 구조체 내에서 사용하는 시간을 `mock`하세요.
프로덕션 코드의 경우 `time.Now`로 필드를 초기화 할수 있습니다. 테스트 코드에는 구조체에서 사용할 시간을 설정할 수 있습니다.

<table>
<tbody>
<tr><th>Avoid 🔥</th></tr>
<tr><td>

```go
func (s *SomeType) IsExpired(created time.Time) bool {
    // Code is hardly testable.
    return time.Since(created) >= s.expiryDuration
}
```

</td></tr>
<tr><th>Better 🤓</th></tr>
<tr><td>

```go
func (s *SomeType) IsExpired(created time.Time) bool {
    // s.timeNow is time.Now on production, mocked in tests.
    return created.Add(s.expiryDuration).Before(s.timeNow())
}
```

</td></tr>
</tbody></table>

## Enforced by Linters

아래부터는 우리가 자동으로 검사하는 규칙 목록입니다. 이 섹션은 왜 그러한 `lint` 규칙이 추가되었는지 궁금하거나, 유사한 규칙을 자신들의 Go 프로젝트에 추가하고 싶은 사람을 위한 것입니다. 🤗

#### Avoid Prints

`print`를 사용하지 않습니다. 항상 전달된 `go-kit/log.Logger`를 사용하세요.

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L311)에서 검사합니다.

#### Ensure Prometheus Metric Registration

`registry.MustRegister` 함수에 프로메테우스 메트릭(예를 들어 `prometheus.Counter`)을 추가하는 것을 잊어버리기 쉽습니다.
이 것을 방지하기 위해, `promtest.With(r).New*`를 통해 모든 메트릭을 생성하고 구 버전의 등록을 허용하지 않습니다.
[여기](https://github.com/thanos-io/thanos/issues/2102)에서 이 문제에 대해 읽어볼 수 있습니다.

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L308)에서 검사합니다.

#### go vet

표준 `Go vet`은 많이 엄격하지만, 합당한 이유가 있습니다. 항상 Go code를 `vet` 하세요!

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L313)에서 검사합니다.

#### golangci-lint

[golangci-lint](https://github.com/golangci/golangci-lint)는 당신의 코드에 대해서, Go 커뮤니티의 다양한 `lint`를 수행할 수 있는 멋진 도구입니다. star를 주고 이를 사용하세요. (:

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L315)에서 [those linters](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/.golangci.yml#L31)를 사용합니다.

#### misspell

`misspell`은 훌륭합니다. 주석과 문서에서 오타를 찾아냅니다.

아직 문법 플러그인이 없습니다 ): (우리는 원합니다).

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L317)에서 검사합니다.

#### Comments Should be Full Sentences

모든 주석은 완전한 문장이어야 합니다. 대문자로 시작하고 마침표로 끝나야 합니다.

[여기](https://github.com/thanos-io/thanos/blob/40526f52f54d4501737e5246c0e71e56dd7e0b2d/Makefile#L194)에서 검사합니다.

# Bash

전반적으로 `Bash`를 사용하지 마세요. 30줄보다 긴 스크리브의 경우 [여기](https://github.com/thanos-io/thanos/blob/55cb8ca38b3539381dc6a781e637df15c694e50a/scripts/copyright/copyright.go)처럼 Go로 작성해보세요.

필요한 경우 Google Shell 스타일 가이드를 따릅니다: https://google.github.io/styleguide/shellguide.html

