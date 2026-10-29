# 发行镜像说明（MIRROR）

本仓库是 **TuraIDC 内置 AI 助手插件的发行镜像**，内容逐字来自主仓：

```text
25Cloud/TuraIDC → backend/plugins/addons/ai_assistant/
```

- 插件 slug：`ai_assistant`，域：`addons`
- 当前版本：见 `config.php` 的 `info.version`（与本仓 tag `v{version}` 一致）
- 许可：**AGPL-3.0-or-later**，与主仓同源同许可（见 `LICENSE`）。商业授权见主仓 `LICENSE-COMMERCIAL.md`

## 为什么是镜像，不是开发仓库

AI 助手是 **内置插件**，它和主仓的核心代码（管理端页面、`App\Support\OptionalFeature` 开关、
`AdminPermissions` 权限码、`ai_*` 四张核心表）是配套关系，拆成两个开发源一定会漂。所以：

- **唯一真源在主仓**，不要在这个仓库提 PR、也不要直接改这里；
- 本仓只做分发：让人能用 `plugin:market:install` 或本地 zip 装到自己的站点，只读源码。

## 从主仓同步

```bash
# 在主仓根目录执行
rsync -a --delete \
  --exclude .git --exclude MIRROR.md --exclude LICENSE \
  backend/plugins/addons/ai_assistant/ ../turaidc-plugin-ai-assistant/

cd ../turaidc-plugin-ai-assistant
git add -A && git commit -m "Sync: v<version> ← 25Cloud/TuraIDC"
git tag -a "v<version>" -m "v<version>"
git push origin main --tags
```

`LICENSE` 由主仓根目录的 `LICENSE` 复制而来，不要在本仓单独修改。

## 安装到站点

```bash
cd backend
php artisan plugin:market:install ai_assistant                  # 需索引条目（25Cloud/turaidc-plugin-index）
php artisan plugin:market:install ai_assistant --zip=<本仓源码 zip>  # 本地包安装
```

安装器会把内容放到 `backend/plugins/addons/ai_assistant/`。**如果站点是直接从主仓部署的，
这里已经是内置插件，不需要再装一遍**——重复安装只会用同一份内容覆盖同一目录。

## 装完之后还会缺什么

本插件的能力有一部分由**核心**承载，单独装插件不会自动带上：

| 依赖 | 位置 | 缺失后果 |
| --- | --- | --- |
| `ai_conversations` / `ai_messages` / `ai_tool_invocations` / `ai_usage_daily` 四张表 | 核心迁移 | 接口 500（插件不建表） |
| `/admin/ai/*` 页面、侧栏分组、`ai.view`/`ai.manage` 权限码 | `frontend-admin-v3` + `App\Support\OptionalFeature` | 后台没有入口 |
| 登录/取用户信息接口下发的 `features` | 核心 | 侧栏分组不会按插件状态收敛 |

也就是说：**在主仓部署的实例里它本来就在，本仓面向的是"想单独拿到这份插件代码"的场景**，
完整功能仍以主仓为准。
