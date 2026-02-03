# ctpbee_opt_api

基于 [ctpbee_api](https://github.com/ctpbee/ctpbee_api) 的优化版本，感谢原作者 [somewheve](https://github.com/somewheve) 的贡献。

## 优化内容

### 1. 合约查询批量模式 (Instrument Batch Mode)

解决全量查询合约时 GIL 争抢导致 UI 界面卡顿的问题。

**问题背景**: 查询全量合约（约 14000 条）时，CTP 回调会逐条触发 Python 回调，每次都需要获取 GIL，导致 Qt 主线程无法响应，界面完全卡死约 15 秒。

**解决方案**: 新增批量模式，在 C++ 层缓存所有合约数据，仅在最后一条数据到达时一次性回调 Python，GIL 只获取一次。

## 安装

```bash
pip install ctpbee_opt_api
```

## 使用方法

```python
from ctpbee_api import TdApi

class MyTdApi(TdApi):
    def onRspQryInstrumentBatch(self, instruments):
        # 批量模式下，一次性收到所有合约
        print(f"收到 {len(instruments)} 条合约")
        for inst in instruments:
            print(inst["InstrumentID"], inst["InstrumentName"])

td_api = MyTdApi()
td_api.setInstrumentBatchMode(True)  # 开启批量模式
td_api.reqQryInstrument({}, 0)
# 等待 onRspQryInstrumentBatch 回调
```

### 默认模式（向后兼容）

不调用 `setInstrumentBatchMode(True)` 时，行为与原版 ctpbee_api 完全一致：

```python
class MyTdApi(TdApi):
    def onRspQryInstrument(self, data, error, reqid, last):
        # 每条合约数据都会触发此回调
        if data:
            print(data["InstrumentID"])
        if last:
            print("查询完成")
```

## 新增 API

| 方法 | 说明 |
|------|------|
| `setInstrumentBatchMode(enable: bool)` | 设置合约查询批量模式 |
| `getInstrumentBatchMode() -> bool` | 获取当前批量模式状态 |
| `onRspQryInstrumentBatch(instruments: list)` | 批量模式下的合约查询回调 |

## 支持平台

- Windows: `ctp`, `rohon`, `ctp_mini`
- Linux: `ctp`, `rohon`, `ctp_mini`
- macOS: `ctp`

## Python 版本

- Python 3.8+

## 致谢

- 原项目: [ctpbee_api](https://github.com/ctpbee/ctpbee_api)
- 原作者: [somewheve](https://github.com/somewheve)
- 底层 API 基于 [vnpy](https://github.com/vnpy/vnpy)

## License

MIT
