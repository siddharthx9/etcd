REPOSITORY_ROOT := $(shell git rev-parse --show-toplevel)
USER_ID := $(shell id -u)
GROUP_ID := $(shell id -g)
ARCH ?= $(shell go env GOARCH)

.PHONY: antithesis-build-client-docker-image
antithesis-build-client-docker-image:
	docker build --build-arg GO_VERSION=$(shell cat $(REPOSITORY_ROOT)/.go-version) -f $(REPOSITORY_ROOT)/tests/antithesis/test-template/Dockerfile $(REPOSITORY_ROOT) -t etcd-client:latest

.PHONY: antithesis-build-etcd-image
antithesis-build-etcd-image:
	git fetch origin release-3.5
	git worktree remove -f /tmp/etcd3.5 || true
	git worktree add /tmp/etcd3.5 origin/release-3.5
	cd /tmp/etcd3.5 && make build
	cd /tmp/etcd3.5 && cp ./Dockerfile-release.$(ARCH) ./bin/Dockerfile
	cd /tmp/etcd3.5 && docker build \
		--tag etcd-server:latest \
		./bin
	git worktree remove -f /tmp/etcd3.5

.PHONY: antithesis-docker-compose-up
antithesis-docker-compose-up:
	export USER_ID=$(USER_ID) && export GROUP_ID=$(GROUP_ID) && docker-compose up

.PHONY: antithesis-run-container-tests
antithesis-run-container-tests:
	export USER_ID=$(USER_ID) && export GROUP_ID=$(GROUP_ID) && docker-compose exec client /opt/antithesis/test/v1/robustness/singleton_driver_main

.PHONY: antithesis-run-local-tests
antithesis-run-local-tests:
	go run --race ./test-template/robustness/main.go --local

.PHONY: antithesis-clean
antithesis-clean:
	export USER_ID=$(USER_ID) && export GROUP_ID=$(GROUP_ID) && docker-compose down
	rm -rf /tmp/etcddata0 /tmp/etcddata1 /tmp/etcddata2 /tmp/etcdreport
