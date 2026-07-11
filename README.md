# Chrome Hearts 官网新品监控器

每隔 5 分钟检查 Chrome Hearts 官方网站。发现官网出现此前没有的商品编号时，通过 Bark 向 iPhone 推送商品名、分类、价格、库存状态、商品图片和购买链接。

## 已实现

- 监控 Chrome Hearts 官网首页及所有当前公开销售分类
- 自动识别官网以后增加的新分类入口
- 以官方商品编号去重，不因页面排序变化而误报
- 首次运行只记录现有商品，不把整页商品当成新品
- 新品推送失败时保留为“未通知”，下一轮自动重试
- 商品暂时消失后再次出现，不会被误判为全新商品
- 可选择同时提醒补货，默认关闭
- 只使用 Python 标准库，无需安装依赖
- Bark 密钥只放在 GitHub Secret 中，不写入代码或日志

默认监控页面：

- `https://www.chromehearts.com/baccarat`
- `https://www.chromehearts.com/scents`
- `https://www.chromehearts.com/boxers-leggings`
- `https://www.chromehearts.com/intimates`
- `https://www.chromehearts.com/socks`

## 推荐部署：GitHub Actions

### 1. 准备 Bark

1. 在 iPhone 安装 **Bark**，第一次打开时允许通知。
2. 在 Bark 首页复制你的专属地址，格式类似：

   ```text
   https://api.day.app/xxxxxxxxxxxxxxxx/
   ```

这个地址相当于推送密码，不要公开，也不要写进任何代码文件。

### 2. 上传项目

新建一个 GitHub 仓库，把压缩包内的全部内容上传到仓库根目录。若希望每 5 分钟检查一次，建议使用公开仓库；代码和商品状态可以公开，但 Bark 地址仍应只保存在 Secret 中。

### 3. 添加 Bark Secret

进入 GitHub 仓库：

`Settings` → `Secrets and variables` → `Actions` → `New repository secret`

- Name：`BARK_URL`
- Secret：粘贴 Bark App 中复制的完整专属地址

### 4. 测试 iPhone 推送

进入仓库的 `Actions` 页面，选择 **Chrome Hearts Monitor**：

1. 点击 `Run workflow`
2. 模式选择 `test_bark`
3. 再点击绿色的 `Run workflow`

几秒后应收到“Chrome Hearts 监控测试成功”。

### 5. 建立初始基线

再次运行工作流，模式选择 `rebuild_baseline`。它会记录官网当前全部商品，并发送“监控已启动”，但不会把现有商品逐个推送。

完成后无需保持电脑开机。工作流会自动检查，发现新品时点击 iPhone 通知即可打开商品详情页。

## 可选：补货提醒

工作流默认只提醒新商品。若也需要监控补货，将 `.github/workflows/monitor.yml` 中：

```yaml
NOTIFY_RESTOCKS: "false"
```

改为：

```yaml
NOTIFY_RESTOCKS: "true"
```

## 在电脑上测试

macOS / Linux：

```bash
export BARK_URL='https://api.day.app/你的密钥/'
python3 monitor.py --test-bark
python3 monitor.py --dry-run
python3 monitor.py
```

Windows PowerShell：

```powershell
$env:BARK_URL='https://api.day.app/你的密钥/'
python monitor.py --test-bark
python monitor.py --dry-run
python monitor.py
```

## 其他设置

| 环境变量 | 默认值 | 用途 |
| --- | --- | --- |
| `BARK_URL` | 无 | Bark App 复制出的完整地址 |
| `BARK_DEVICE_KEY` | 无 | 也可只提供 Bark 设备密钥 |
| `BARK_SERVER` | `https://api.day.app` | 自建 Bark 服务地址 |
| `NOTIFY_RESTOCKS` | `false` | 是否提醒从售罄变为可购买 |
| `MONITOR_URLS` | 当前五个分类 | 用英文逗号分隔自定义页面 |
| `CRAWL_DELAY_SECONDS` | `0.35` | 请求各页面之间的间隔 |

常用命令：

```bash
# 只检查并显示结果，不推送也不修改状态
python3 monitor.py --dry-run

# 发送一条 Bark 测试通知
python3 monitor.py --test-bark

# 用当前官网商品重新建立基线
python3 monitor.py --reset-baseline
```

## 注意事项

- GitHub 的定时任务最低间隔为 5 分钟，但繁忙时可能延迟，不保证精确到秒。
- `data/state.json` 是去重记录，部署后不要随意删除；删除后下一次只会重新建立基线。
- 工具只读取无需登录的官网公开页面，不自动下单，也不绕过验证码或访问控制。
- 官网改版后若连续解析不到商品，工作流会失败而不会清空现有状态，避免误报。
- 请将工具用于个人提醒，并遵守网站条款及合理访问频率。

## 运行测试

```bash
python3 -m unittest discover -s tests -v
```
