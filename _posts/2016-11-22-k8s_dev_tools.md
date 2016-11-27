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

## Update vendors

```
// use new/clear $GOPATH 
export KPATH=$HOME/code/kubernetes
mkdir -p $KPATH/src/k8s.io
cd $KPATH/src/k8s.io

// restore the dependencies into new $GOPATH
godep restore

// Update dependencies, e.g. godep get

rm -rf Godeps
rm -rf vendor
./hack/godep-save.sh
```

## Verify before commit

```
hack/update-munge-docs.sh
hack/update-swagger-spec.sh
hack/update-openapi-spec.sh
make verify
```

## Reference

* [End-to-End Testing in Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/e2e-tests.md)
* [Using godep to manage dependencies](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/godep.md)
