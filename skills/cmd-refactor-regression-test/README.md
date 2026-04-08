# cmd-refactor-regression-test

用于验证本仓库 `cmd/` 层重构后的回归测试 skill。

适用场景：

- `cmd/<command>/index.go` 子目录化迁移
- `cmd/utils` 公共函数抽取
- `cmd/root.go` 命令注册调整
- `add/install/list/push/remove/search/status/update/repo*` 相关修改

核心动作：

1. `gofmt`
2. `go test ./...`
3. CLI 帮助和关键命令冒烟

推荐最小命令集：

```powershell
go test ./...
go run . list --repo
go run . install -h
go run . update -h
go run . push -h
go run . add -h
go run . search -h
go run . remove -h
go run . status -h
```
