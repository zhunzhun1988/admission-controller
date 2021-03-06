all: build

TAG?=v0.2.0
REGISTRY?=ihub.helium.io:29006
FLAGS=
ENVVAR=
GOOS?=linux
ROOTPATH=`cd ../../; pwd` 
BUILDGOPATH=/tmp/k8splugin-build
BUILDPATH=$(BUILDGOPATH)/src/k8s-plugins/admission-controller/pkg/hppvtocsipv
IMAGENAME=${REGISTRY}/library/hppvtocsipv-admission-controller:${TAG}
.IGNORE : deletehookconfig
.IGNORE : deletedeploy
.IGNORE : buildEnvClean

deps:
	@go get github.com/tools/godep
	
buildEnvClean:
	@rm $(BUILDGOPATH)/src/k8s-plugins/admission-controller 1>/dev/null 2>/dev/null || true

buildEnv: buildEnvClean
	@mkdir -p $(BUILDGOPATH)/src/k8s-plugins/ 1>/dev/null 2>/dev/null
	@ln -s $(ROOTPATH) $(BUILDGOPATH)/src/k8s-plugins/admission-controller
	
build: buildEnv clean deps 
	@cd $(BUILDPATH) && GOPATH=$(BUILDGOPATH) $(ENVVAR) GOOS=$(GOOS) CGO_ENABLED=0   godep go build ./...
	@cd $(BUILDPATH) && GOPATH=$(BUILDGOPATH) $(ENVVAR) GOOS=$(GOOS) CGO_ENABLED=0   godep go build -o hppvtocsipv

docker:
ifndef REGISTRY
	ERR = $(error REGISTRY is undefined)
	$(ERR)
endif
	docker build --pull -t ${IMAGENAME} .
	docker push ${IMAGENAME}

deletedeploy:
	@kubectl delete -f ../../deploy/hppvtocsipv-admission-controller-deployment.yaml 1>/dev/null 2>/dev/null || true

deletehookconfig:
	@kubectl delete MutatingWebhookConfiguration  hppvtocsipv 1>/dev/null 2>/dev/null || true

install: deletehookconfig deletedeploy
	./gencerts.sh
	echo ${IMAGENAME}
	@cat ../../deploy/hppvtocsipv-admission-controller-deployment.yaml | sed "s!{hppvtocsipv-image}!${IMAGENAME}!g" > ../../deploy/tmp.yaml
	kubectl create -f ../../deploy/tmp.yaml
	@rm ../../deploy/tmp.yaml

uninstall: deletedeploy deletehookconfig

release: build docker
	rm -f hppvtocsipv

clean: buildEnvClean
	@rm -f hppvtocsipv

format:
	test -z "$$(find . -path ./vendor -prune -type f -o -name '*.go' -exec gofmt -s -d {} + | tee /dev/stderr)" || \
	test -z "$$(find . -path ./vendor -prune -type f -o -name '*.go' -exec gofmt -s -w {} + | tee /dev/stderr)"

.PHONY: all deps build test-unit clean format release

