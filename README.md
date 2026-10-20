# AutoTags 自动标签生成插件 v1.5.0

AutoTags 是 Typecho 插件。发布文章时，它会提取标题和部分正文，调用所选模型生成标签，并添加到文章中。支持 DeepSeek Flash 和 MiniMax-M3.1-Flash。

## 安装与升级

1. 将插件文件夹命名为 `AutoTags`，上传到 Typecho 的 `/usr/plugins/` 目录。
2. 在 Typecho 后台激活插件，打开插件设置。
3. 选择生成标签使用的模型，填写对应的 API Key，并保存设置。

从旧版本升级时，默认继续使用 DeepSeek，原有 `deepseekApiKey` 配置项保持不变。若要切换到 MiniMax，请选择 MiniMax 并填写 MiniMax API Key。两个提供方的 Key 分开保存，只有所选模型的 Key 会用于请求。

## 设置

- **模型**：`DeepSeek Flash (V4.1)` 或 `MiniMax-M3.1-Flash`，默认 DeepSeek。
- **DeepSeek API Key**：选择 DeepSeek 时使用。
- **MiniMax API Key**：选择 MiniMax 时使用。
- **文章正文提取内容长度**：默认 333 字符。输入须为整数；长度越大，输入 token 用量通常越高。
- **编辑文章时是否重新生成标签**：默认否。设为是后，编辑已有标签的文章时会重新生成标签。

若需调整标签数量或规则，可修改 `Plugin.php` 的 `generateTags()` 中的 `$prompt`。

## 工作方式

- 发布文章且未手动添加标签时，插件自动生成标签。已有标签时，默认保留原标签且不调用 API。
- 开启“编辑文章时是否重新生成标签”后，插件先等待模型返回有效标签，再替换旧标签。API 请求失败或没有生成标签时，旧标签保留。
- 替换标签时只处理标签关系，不删除文章分类。
- 钩子为 Typecho 的 `Widget_Contents_Post_Edit::finishPublish`。

## 模型与接口

| 提供方 | 模型 ID | 接口 | 请求设置 |
| --- | --- | --- | --- |
| DeepSeek | `deepseek-flash` | `https://api.deepseek.com/chat/completions` | 非流式；关闭 thinking；`max_tokens=256` |
| MiniMax | `MiniMax-M3.1-Flash` | `https://api.minimax.cn/anthropic/v1/messages` | 非流式；`output_config.effort=low`；`max_tokens=2048` |

MiniMax-M3.1-Flash 强制开启 thinking，思考 token 计入 `max_tokens`；因此插件使用低思考档位，并只读取响应中的 `text` 内容块。DeepSeek 官方当前推荐使用 `deepseek-flash`，该 ID 对应 DeepSeek-V4.1-Flash-1017。旧 `deepseek-v4-flash` 名称仍可请求，但对应的旧模型已退役。

接口依据：[MiniMax Anthropic API 文档](https://platform.minimax.cn/docs/api-reference/text-anthropic-api)、[DeepSeek 入门文档](https://api-docs.deepseek.com/)、[DeepSeek Chat Completions 文档](https://api-docs.deepseek.com/api/create-chat-completion)。

## 注意事项

- 需要自行在所选提供方获取有效 API Key；调用可能产生费用。
- 服务器需能访问所选提供方的 HTTPS 接口，并安装 PHP cURL 和 mbstring 扩展。
- API 失败信息记录在 PHP 错误日志中；日志不会记录 API Key 或文章内容。

## 更新日志

### v1.5.0 (2026-10-20)

- 新增后台模型选择与独立的 MiniMax API Key，保留原有 DeepSeek API Key 和默认选择。
- 接入 `MiniMax-M3.1-Flash` 的 Anthropic 兼容接口，适配 thinking 与文本块响应。
- 将 DeepSeek 模型 ID 更新为官方推荐的 `deepseek-flash`，限制标签任务的生成 token。
- API 失败时保留原标签；重新生成只清除标签关系，不影响分类。
- 恢复 HTTPS 证书校验，增加请求失败和无效响应的错误日志。

### v1.4.8 (2026-07-17)

- 修复设置页 `contentLength` 校验规则导致的保存失败。

### v1.4.7 (2026-07-09)

- 增加编辑文章时是否重新生成标签的设置。

### v1.4.6 (2026-06-17)

- 调整 DeepSeek 接口、标签生成提示词及文章编辑行为。
