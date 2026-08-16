# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

journal 的某一行先包含一个哈希链完全有效的事件，后面又紧跟第二个 JSON 值时，校验仍报告成功。请修复为严格的一行一个事件：第二个 JSON 值或其他畸形尾随内容必须被拒绝，合法尾随空白仍应接受，并保证全量测试通过。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/gogo-42
- 仓库地址：https://github.com/zhanglei10281852-gif/gogo-42.git
- parent SHA：97ca32e84277935ca860af7b04a6666ec453dcf4

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/gogo-42.git bug-repro
cd bug-repro
git checkout --detach 97ca32e84277935ca860af7b04a6666ec453dcf4
go test ./internal/store -run "^TestVerifyRejectsExtraJSONValueOnJournalLine$" -count=1 -v
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/store -run "^TestVerifyRejectsExtraJSONValueOnJournalLine$" -count=1 -v
=== RUN   TestVerifyRejectsExtraJSONValueOnJournalLine
    store_test.go:79: expected verifier to reject a second JSON value on one line
--- FAIL: TestVerifyRejectsExtraJSONValueOnJournalLine (0.01s)
FAIL
FAIL	QueueForge/internal/store	0.014s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/store -run "^TestVerifyRejectsExtraJSONValueOnJournalLine$" -count=1 -v
=== RUN   TestVerifyRejectsExtraJSONValueOnJournalLine
    store_test.go:79: expected verifier to reject a second JSON value on one line
--- FAIL: TestVerifyRejectsExtraJSONValueOnJournalLine (0.06s)
FAIL
FAIL	QueueForge/internal/store	0.198s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

首个事件后的第二个 JSON 值和畸形尾随字节均被拒绝；仅有尾随空白仍合法；现有哈希链与恢复行为不回归；双架构定向、全量、build/vet 通过。
