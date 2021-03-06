# zagane

[![CircleCI](https://circleci.com/gh/gcpug/zagane.svg?style=svg)](https://circleci.com/gh/gcpug/zagane)
[![GoDoc](https://godoc.org/github.com/gcpug/zagane?status.svg)](https://godoc.org/github.com/gcpug/zagane)

`zagane` is a static analysis tool which can find bugs in spanner's code.
`zagane` consists of several analyzers.

* `unstopiter`: it finds iterators which did not stop.

## Install

You can get `zagane` by `go get` command.

```bash
$ go get -u github.com/gcpug/zagane
```

## How to use

`zagane` run with `go vet` as below when Go is 1.12 and higher.

```bash
$ go vet -vettool=$(which zagane) github.com/gcpug/spshovel/...
# github.com/gcpug/spshovel/spanner
spanner/spanner_service.go:29:29: iterator must be stop
```

When Go is lower than 1.12, just run `zagane` command with the package name (import path).
But it cannot accept some options such as `--tags`.

```bash
$ zagane github.com/gcpug/spshovel/...
~/go/src/github.com/gcpug/spshovel/spanner/spanner_service.go:29:29: iterator must be stop
```

## Analyzers

### unstopiter

`unstopiter` finds spanner.RowIterator which is not calling [Stop](https://godoc.org/cloud.google.com/go/spanner#RowIterator.Stop) method or [Do](https://godoc.org/cloud.google.com/go/spanner#RowIterator.Do) method such as below code.

```go
iter := client.Single().Query(ctx, stmt)
for {
	row, err := iter.Next()
	// ...
}
```

This code must be fixed as below.

```go
iter := client.Single().Query(ctx, stmt)
defer iter.Stop()
for {
	row, err := iter.Next()
	// ...
}
```

## Ignore Checks

Analyzers ignore nodes which are annotated by [staticcheck's style comments](https://staticcheck.io/docs/#ignoring-problems) as belows.
A ignore comment includes analyzer names and reason of ignoring checking.
If you specify `zagane` as analyzer name, all analyzers ignore corresponding code.

```go
//lint:ignore zagane reason
var n int

//lint:ignore unstopiter reason
_, _ = client.Single().Query(ctx, stmt).Next()
```

## Analyze with golang.org/x/tools/go/analysis

You can get analyzers of zagane from [zagane.Analyzers](https://godoc.org/github.com/gcpug/zagane/zagane/#Analyzers).
And you can use them with [unitchecker](https://golang.org/x/tools/go/analysis/unitchecker).

## Why name is "zagane"?

"zagane" (座金) means "washer" in Japanese.
A washer works between a spanner and other parts.
`zagane` also works between Cloud Spanner and your applications.
