# dsh-plugin-avatar

在 DeepSeek Harness 的对话界面里给双方加头像：

- 助手回复：左侧显示头像
- 用户消息：右侧显示账号头像（从侧边栏 `[data-avatar-status]` 里取，图源是账号头像）
- **双击助手头像**会弹出设置面板：换图片、调大小（16–96 px）、恢复默认
- 设置保存在 `localStorage`（键名 `dshAvatar.v1`），重启后仍然有效

## 安装

1. 把整个目录复制到 profile 的 `node_modules` 下：

   ```
   ~/.dsh/profiles/desktop/node_modules/dsh-plugin-avatar/
   ```

2. 在 `~/.dsh/profiles/desktop/cordis.patch.yml` 末尾追加：

   ```yaml
   - insert:
       - id: avatar-plugin
         name: "dsh-plugin-avatar"
   ```

3. 重启 DeepSeek Harness
4. 如果界面还是旧样子，在窗口里按 `Ctrl + Shift + R` 强制刷新（客户端 bundle 有缓存）

## 卸载

```powershell
Remove-Item "$env:USERPROFILE\.dsh\profiles\desktop\node_modules\dsh-plugin-avatar" -Recurse -Force
```

然后把 `cordis.patch.yml` 里那段 `insert` 删掉，重启。

## 实现说明

DSH 的消息节点插槽（`conversation.chat.node`）是按 key 注册的，占进去会**替换**整个节点渲染，所以这里没有走插槽，改用 DOM 注入：

- 用 `MutationObserver` 监听 `document.body`，变化时（节流到 `requestAnimationFrame`）扫描
- 助手消息容器选择器：`[data-chat-group-part="response"]`
- 用户消息：`[data-slot*="user.text"]`，向上找到宽度 > 300px 的祖先作为定位容器
- 头像用 `position:absolute` 贴在容器左侧或右侧，容器若为 `static` 则改成 `relative`

这套做法依赖 DOM 结构，DSH 前端改版后可能需要跟着调整选择器。

## 已知限制

- 只在桌面端（Electron / web 表面）验证过
- 每帧扫描是一次全文档查询，消息很多时有轻微开销
- 头像图片以 data URI 内嵌，默认 192 px，替换图片后大小取决于所选文件
