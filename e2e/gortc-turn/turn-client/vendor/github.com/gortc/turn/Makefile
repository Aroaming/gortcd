GO_VERSION=$(shell gobuild -v)
GO := $(or $(GOROOT),/usr/lib/go)/bin/go
PROCS := $(shell nproc)
cores:
	@echo "cores: $(PROCS)"
test:
	go test -v
bench:
	go test -bench .
bench-record:
	$(GO) test -bench . > "benchmarks/turn-go-$(GO_VERSION).txt"
fuzz-prepare-setters:
	go-fuzz-build -func FuzzSetters -o turn-setters-fuzz.zip github.com/gortc/turn
fuzz-setters:
	go-fuzz -bin=./turn-setters-fuzz.zip -workdir=fuzz/turn-setters
fuzz-test:
	go test -tags gofuzz -run TestFuzz -v .
fuzz-prepare-chandata:
	go-fuzz-build -func FuzzChannelData -o turn-chandata-fuzz.zip github.com/gortc/turn
fuzz-chandata:
	go-fuzz -bin=./turn-chandata-fuzz.zip -workdir=fuzz/turn-chandata
lint:
	@echo "linting on $(PROCS) cores"
	@gometalinter \
		--enable-all \
		--vendor \
		-e "_test.go.+(gocyclo|errcheck|dupl)" \
		-e "attributes\.go.+credentials,.+,LOW.+\(gas\)" \
		-e "parameter v always receives 4" \
		-e "function \(\*Client\).allocate\(\) is high .+\(gocyclo\)" \
		--enable="lll" --line-length=100 \
		--enable="gofmt" \
		--enable="goimports" \
		--enable="misspell" \
		--enable="unused" \
		--deadline=300s \
		--disable="gochecknoglobals" \
		--disable="interfacer" \
		-j $(PROCS)
	@echo "ok"
escape:
	@echo "Not escapes, except autogenerated:"
	@go build -gcflags '-m -l' 2>&1 \
	| grep -v "<autogenerated>" \
	| grep escapes
format:
	goimports -w .
docker-build:
	docker build -t gortc/turn .
test-integration:
	@cd e2e && ./test.sh
check-api:
	api -c api/turn1.txt github.com/gortc/turn
write-api:
	api github.com/gortc/turn > api/turn1.txt

