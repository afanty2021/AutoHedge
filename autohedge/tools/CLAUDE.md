[根目录](../../CLAUDE.md) > [autohedge](../) > **tools**

# 交易工具模块

## 模块职责

Tools模块提供各种券商和交易平台的API集成接口，为AutoHedge核心系统提供标准化的交易执行能力。

## 入口与启动

### 主要文件
- **`__init__.py`**: 模块初始化 (空文件)
- **`td_ameritrade.py`**: TD Ameritrade API客户端
- **`trade_station.py`**: TradeStation API接口
- **`e_trade_wrapper.py`**: E*TRADE API封装

### 使用示例
```python
# TD Ameritrade
from autohedge.tools.td_ameritrade import TDAmeritradeClient
client = TDAmeritradeClient()

# TradeStation
from autohedge.tools.trade_station import confirm_order
response = confirm_order(symbol="AAPL", quantity="10", trade_action="BUY")

# E*TRADE
from autohedge.tools.e_trade_wrapper import ETradeClient
client = ETradeClient(account_id="12345", production_url="https://api.etrade.com/v1")
```

## 对外接口

### TDAmeritradeClient
```python
class TDAmeritradeClient:
    def __init__(self, api_key: Optional[str], access_token: Optional[str], account_id: str)
    # 具体接口方法需要在完整代码中确认
```

### TradeStation接口 (函数式)
```python
def confirm_order(
    account_id: str,
    symbol: str,
    quantity: str,
    order_type: str = "Market",
    trade_action: str = "BUY",
    time_in_force: Optional[Dict[str, str]] = None,
    limit_price: Optional[str] = None,
    stop_price: Optional[str] = None,
    token: Optional[str] = None
) -> Dict[str, Any]
```

### ETradeClient
```python
class ETradeClient:
    def __init__(self, account_id: str, production_url: str)
    def place_order(self, account_id: str, symbol: str, quantity: int, action: str, price: Optional[float]) -> Dict[str, Any]
    def get_account_info(self) -> Dict[str, Any]
    def logout(self) -> None
```

## 关键依赖与配置

### 外部依赖
- **requests**: HTTP客户端
- **requests-oauthlib**: OAuth认证 (E*TRADE)
- **tenacity**: 重试机制 (TD Ameritrade)
- **dotenv**: 环境变量管理
- **loguru**: 日志记录

### 环境变量配置

#### TD Ameritrade
```bash
TD_API_KEY=""               # TD Ameritrade API密钥
TD_ACCESS_TOKEN=""          # OAuth访问令牌
```

#### TradeStation
```bash
TRADE_STATION_ACCOUNT_ID=""  # TradeStation账户ID
TRADE_STATION_TOKEN=""       # OAuth访问令牌
```

#### E*TRADE
```bash
ETRADE_CONSUMER_KEY=""        # E*TRADE消费者密钥
ETRADE_CONSUMER_SECRET=""     # E*TRADE消费者密钥
ETRADE_OAUTH_TOKEN=""         # OAuth访问令牌
ETRADE_OAUTH_TOKEN_SECRET=""  # OAuth令牌密钥
ETRADE_ACCOUNT_ID=""          # E*TRADE账户ID
```

## 核心功能

### TD Ameritrade集成 (td_ameritrade.py)
- **OAuth认证**: 基于Bearer Token的安全认证
- **会话管理**: 持久化HTTP会话
- **错误处理**: 完整的异常处理和重试机制
- **配置验证**: 启动时检查必需的认证信息

#### 特性
- 自动环境变量加载
- 请求头自动设置
- 详细错误日志
- 超时和重试保护

### TradeStation集成 (trade_station.py)
- **订单确认**: 实时订单成本和佣金估算
- **多种订单类型**: 支持市价单、限价单、止损单等
- **灵活参数**: 支持高级订单选项和OSOs
- **安全认证**: Bearer Token认证

#### 支持的订单类型
- **Market**: 市价单
- **Limit**: 限价单
- **StopMarket**: 止损市价单
- **StopLimit**: 止损限价单

#### 交易行为
- **BUY**: 买入
- **SELL**: 卖出
- **BUYTOCOVER**: 买回补仓
- **SELLSHORT**: 卖空
- **BUYTOOPEN**: 买入开仓
- **BUYTOCLOSE**: 买入平仓
- **SELLTOOPEN**: 卖出开仓
- **SELLTOCLOSE**: 卖出平仓

### E*TRADE集成 (e_trade_wrapper.py)
- **OAuth 1.0认证**: 完整的三阶段OAuth流程
- **账户管理**: 余额和持仓查询
- **订单执行**: 限价单和市价单支持
- **会话管理**: 自动会话生命周期管理

#### API端点
- **账户信息**: `/accounts/{account_id}/balance`
- **下单交易**: `/accounts/{account_id}/orders/place`

## 数据模型

### TradeStation订单确认响应
```python
{
    "EstimatedCostDisplay": "$3,450.00",
    "EstimatedCommission": "6.95",
    "EstimatedTotalCost": "3456.95",
    # ... 其他字段
}
```

### E*TRADE订单载荷
```python
order_payload = {
    "orderType": "LIMIT" | "MARKET",
    "clientOrderId": "unique_id",
    "orderAction": "BUY" | "SELL",
    "instrument": [{
        "symbol": "AAPL",
        "quantity": 10,
        "orderAction": "BUY"
    }],
    "priceType": "LIMIT" | "MARKET",
    "limitPrice": 150.00,
    "marketSession": "REGULAR",
    "orderTerm": "GOOD_FOR_DAY"
}
```

## 错误处理

### TD Ameritrade
- **环境错误**: 缺失认证信息时抛出EnvironmentError
- **网络错误**: requests异常完整处理
- **认证错误**: 401/403响应检测
- **重试机制**: tenacity库自动重试

### TradeStation
- **参数验证**: 必需参数检查
- **网络异常**: requests.RequestException处理
- **API错误**: 4xx/5xx状态码处理
- **详细日志**: 请求和响应日志记录

### E*TRADE
- **认证检查**: OAuth凭据完整性验证
- **请求异常**: HTTP错误处理
- **业务错误**: 交易失败错误信息
- **清理机制**: finally块确保资源清理

## 使用示例

### TradeStation订单确认
```python
# 确认订单（不实际执行）
response = confirm_order(
    account_id="123456782",
    symbol="MSFT",
    quantity="10",
    order_type="Market",
    trade_action="BUY",
    token="your_token_here"
)

print(f"预估成本: {response['EstimatedCostDisplay']}")
print(f"佣金: {response['EstimatedCommission']}")
```

### E*TRADE交易执行
```python
# 创建客户端
client = ETradeClient(account_id="12345678")

# 下限价单
buy_response = client.place_order(
    account_id="12345678",
    symbol="AAPL",
    quantity=10,
    action="BUY",
    price=150.00
)

# 查询账户信息
account_info = client.get_account_info()
print(f"账户余额: {account_info}")
```

## 测试与质量

### 测试覆盖
- ❌ 单元测试缺失
- ❌ 集成测试缺失
- ❌ 模拟交易测试缺失
- ✅ 文档示例完整

### 代码质量
- **类型提示**: 完整的类型注解
- **文档字符串**: 详细的docstring
- **错误处理**: 全面的异常处理
- **日志记录**: loguru集成

## 安全考虑

### 凭证管理
- **环境变量**: 敏感信息不硬编码
- **Token安全**: 访问令牌安全存储
- **OAuth标准**: 标准认证流程

### 交易安全
- **订单确认**: TradeStation提供预确认机制
- **参数验证**: 严格的输入验证
- **错误日志**: 安全信息不泄露

## 常见问题 (FAQ)

### Q: 如何获取TD Ameritrade的API密钥？
A: 在TD Ameritrade开发者平台注册应用，获取API Key和完成OAuth流程。

### Q: TradeStation的订单确认是否实际执行交易？
A: 不，confirm_order只是预估成本和佣金，不实际执行交易。

### Q: E*TRADE的OAuth流程复杂吗？
A: 需要完成三步OAuth认证流程，获取所有必需的令牌。

### Q: 如何处理网络连接问题？
A: 所有客户端都内置了重试机制和错误处理，网络问题会自动重试。

## 相关文件清单

- `__init__.py` - 模块初始化
- `td_ameritrade.py` - TD Ameritrade客户端 (部分代码，50行示例)
- `trade_station.py` - TradeStation订单确认 (187行)
- `e_trade_wrapper.py` - E*TRADE客户端 (174行)

## 变更记录 (Changelog)

### 2025-01-19
- 完成工具模块详细分析
- 创建模块级文档
- 识别三大券商集成接口
- 标记测试覆盖缺口
- 提供完整使用示例