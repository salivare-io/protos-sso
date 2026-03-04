---

# Proto SSO Contract

### Overview
Этот репозиторий содержит контракт SSO/Auth сервиса в формате Protocol Buffers, а также конфигурацию для генерации Go, TypeScript и OpenAPI артефактов. Генерация выполняется через **Buf v2** и **remote‑plugins**, поэтому локальная установка `protoc`, `go install` или `npm install` больше не требуется. Вся генерация выполняется в Docker.

---

## Getting started

### Requirements
Локально требуется только:

- **Buf CLI**  
  Установка:
  ```bash
  brew install buf
  ```
- **Docker**
- Плагин Buf для вашей IDE (GoLand / VS Code) — для корректного резолвинга импортов.

### Project structure

```
proto/
  sso/
    auth/
      v1/
        auth.proto

gen/
  go/
  ts/
  openapi/

buf.yaml
buf.gen.yaml
buf.work.yaml
DockerfileGenProto
Taskfile.yml
```

---

## Code generation

### Buf v2 (managed mode)

Файл `buf.gen.yaml`:

```yaml
version: v2

managed:
  enabled: true

plugins:
  - remote: buf.build/connectrpc/es
    out: gen/ts

  - remote: buf.build/protocolbuffers/go
    out: gen/go

  - remote: buf.build/connectrpc/go
    out: gen/go

  - remote: buf.build/grpc-ecosystem/openapiv2
    out: gen/openapi
    opt:
      - allow_merge=true
      - merge_file_name=sso_auth
```

Buf автоматически скачивает плагины из Buf Registry.  
Никаких локальных бинарников не требуется.

---

## Docker‑based generation

### DockerfileGenProto

```dockerfile
FROM alpine:3.19

RUN apk add --no-cache curl unzip

RUN curl -sSL \
    "https://github.com/bufbuild/buf/releases/download/v1.34.0/buf-Linux-x86_64" \
    -o /usr/local/bin/buf \
    && chmod +x /usr/local/bin/buf

WORKDIR /workspace

CMD ["buf", "generate"]
```

### Run generation

```bash
docker build -t protos-sso-gen -f DockerfileGenProto .
docker run --rm -v $(pwd):/workspace protos-sso-gen
```

Сгенерированные файлы появятся в:

```
gen/go
gen/ts
gen/openapi
```

---

## Taskfile (optional but convenient)

```yaml
version: "3"

tasks:
  gen:
    desc: Generate all code via Docker (Buf v2 remote plugins)
    cmds:
      - docker build -t protos-sso-gen -f DockerfileGenProto .
      - docker run --rm -v {{.PWD}}:/workspace protos-sso-gen

  lint:
    desc: Run buf lint locally
    cmds:
      - buf lint

  deps:
    desc: Update buf dependencies
    cmds:
      - buf dep update
```

---

## Notes and recommendations

- Buf v2 remote‑plugins обеспечивают одинаковую генерацию локально, в Docker и в CI.
- IDE использует локальный Buf CLI для резолвинга импортов — Docker на это не влияет.
- Не требуется устанавливать:
  - `protoc`
  - `protoc-gen-go`
  - `protoc-gen-es`
  - `protoc-gen-connect-go`
  - `npm install`
  - `go install`
- Добавьте генерацию в CI, чтобы артефакты всегда были актуальны.
- Поддерживайте `buf.lock` в репозитории — это фиксирует версии remote‑plugins.

---
