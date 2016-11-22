---
layout: post
title: Kubernetes developer tools
categories: tech
---

## Build

```
make
```

## Unit test

```
make check WHAT=pkg/kubectl
make check WHAT=pkg/kubectl KUBE_TEST_ARGS="-run TestOfferStorage" GOFLAGS=-v
```

## E2E test

```
go run hack/e2e.go -v --test
```

## Reference

* [End-to-End Testing in Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/e2e-tests.md)

