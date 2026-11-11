---
slug: "mcp/tools/computers"
title: "电脑工具"
description: "托管 MIOSA MCP 计算机工具：生命周期、文件、检查点、环境、域和桌面表面（屏幕、指针、键盘、窗口）。"
section: "CLI, SDKs & MCP"
order: 11026
---

该系列中的每个工具都与托管服务器注册的完全相同。论点
是工具自己的参数名称；仍然列出了可选的，因为
描述说明了它们改变了什么。

| 工具 | 论点 | 描述 |
|---|---|---|
| `computer_accessibility_tree` | `computer_id` |获取当前桌面状态的 AT-SPI 可访问性树。 |
| `computer_bash` | `computer_id`、`command`、`timeout` |在计算机上执行 bash 命令并返回 stdout + stderr。 |
| `computer_checkpoint_create` | `computer_id`、`comment` |将计算机的当前状态保存为检查点（快照）。 |
| `computer_checkpoint_delete` | `computer_id`、`checkpoint_id` |删除已保存的检查点。 |
| `computer_checkpoint_list` | `computer_id` |列出计算机所有已保存的检查点。 |
| `computer_checkpoint_restore` | `computer_id`、`checkpoint_id` |将计算机恢复到以前保存的检查点。 |
| `computer_click` | `computer_id`、`x`、`y`、`button` |在给定的屏幕坐标处单击鼠标按钮。 |
| `computer_close_window` | `computer_id`, `window_id` |关闭一个窗口。 |
| `computer_copy_file` | `computer_id`、`source`、`dest`、`recursive` |复制计算机内的文件或目录树。 |
| `computer_create` | `name`、`template_type`、`size`、`workspace_id`、`external_workspace_id`、`external_project_id` |创建一台新的 MIOSA 计算机并等待其激活。 |
| `computer_delete_file` | `computer_id`, `path` |删除计算机内的文件或目录。 |
| `computer_destroy` | `computer_id` |永久毁坏一台计算机。 |
| `computer_domain_add` | `computer_id`、`fqdn` |将自定义域添加到计算机。返回CNAME验证 |
| `computer_domain_delete` | `computer_id`、`domain_id` |从计算机中删除自定义域映射。 |
| `computer_domain_list` | `computer_id` |列出为计算机注册的所有自定义域。 |
| `computer_double_click` | `computer_id`、`x`、`y` |双击给定的屏幕坐标。 |
| `computer_drag` | `computer_id`、`from_x`、`from_y`、`to_x`、`to_y` |单击并从一个屏幕位置拖动到另一屏幕位置。 |
| `computer_env_delete` | `computer_id`、`name` |从计算机中删除环境变量。 |
| `computer_env_list` | `computer_id` |列出计算机上设置的所有环境变量。 |
| `computer_env_set` | `computer_id`、`name`、`value` |在计算机上设置（创建或更新）环境变量。 |
| `computer_expose_port` | `computer_id`、`port`、`protocol` |公开计算机上的端口并返回公共 URL。 |
| `computer_focus_window` | `computer_id`, `window_id` |通过窗口 ID 将窗口置于前台。 |
| `computer_get` | `computer_id` |获取计算机的详细信息和当前状态。 |
| `computer_get_clipboard` | `computer_id` |读取当前剪贴板文本内容。 |
| `computer_get_cursor_position` | `computer_id` |获取当前鼠标光标位置（x，y）。 |
| `computer_get_desktop_env` | `computer_id` |获取桌面环境信息（名称、分辨率、会话类型）。 |
| `computer_get_screen_size` | `computer_id` |获取屏幕分辨率（以像素为单位的宽度和高度）。 |
| `computer_hotkey` | `computer_id`、`keys`、`items` |按键盘快捷键（同时按下多个键）。 |
| `computer_key` | `computer_id`、`key` |按一个键。使用标准键名： |
| `computer_key_down` | `computer_id`、`key` |按住某个键而不松开。与computer_key_up 配对。 |
| `computer_key_up` | `computer_id`、`key` |释放之前持有的密钥。 |
| `computer_launch` | `computer_id`, `app` |按名称启动应用程序（例如“firefox”、“gedit”、“xterm”）。 |
| `computer_list` | - |列出租户中的所有计算机。 |
| `computer_list_files` | `computer_id`、`path` |列出计算机上的目录内容。 |
| `computer_list_ports` | `computer_id` |列出计算机上当前公开的所有端口。 |
| `computer_list_regions` | - |列出可用的计算区域以及 GPU 可用性详细信息。 |
| `computer_logs` | `computer_id`, `lines` |从计算机获取最新的虚拟机级别日志。 |
| `computer_maximize_window` | `computer_id`, `window_id` |最大化窗口。 |
| `computer_minimize_window` | `computer_id`、`window_id` |最小化（图标化）窗口。 |
| `computer_mkdir` | `computer_id`、`path`、`recursive` |在计算机内创建一个目录（以及任何缺失的父目录）。 |
| `computer_mouse_down` | `computer_id`、`x`、`y`、`button` |在 (x, y) 处按住鼠标按钮。与computer_mouse_up配对即可释放。 |
| `computer_mouse_up` | `computer_id`、`x`、`y`、`button` |在 (x, y) 处释放按住的鼠标按钮。 |
| `computer_move_cursor` | `computer_id`、`x`、`y` |将鼠标光标移动到给定坐标而不单击。 |
| `computer_network_policy_get` | `computer_id` |获取计算机的当前网络策略（防火墙规则）。 |
| `computer_network_policy_reset` | `computer_id` |将计算机的网络策略重置为平台默认值（全部允许）。 |
| `computer_network_policy_set` | `computer_id`、`rules`、`items`、`default_effect` |设置计算机的网络策略（防火墙规则）。 |
| `computer_preview_url` | `computer_id`, `port` |返回计算机上给定端口的公共预览 URL。 |
| `computer_read_file` | `computer_id`、`path` |从计算机内部读取文件并将其内容作为文本返回。 |
| `computer_rename_file` | `computer_id`、`source`、`dest` |重命名或移动计算机内的文件/目录。 |
| `computer_restart` | `computer_id` |重新启动计算机。 |
| `computer_right_click` | `computer_id`、`x`、`y` |右键单击给定的屏幕坐标。 |
| `computer_rotate_viewer_password` | `computer_id` |轮换并返回原始外部桌面查看器密码一次，以共享原始 *.computer.miosa.ai 链接。 |
| `computer_screenshot` | `computer_id` |捕获计算机桌面的 PNG 屏幕截图。 |
| `computer_scroll` | `computer_id`、`direction`、`clicks`、`x`、`y` |在桌面上沿某个方向滚动。 |
| `computer_service_create` | `computer_id`、`name`、`command`、`working_dir`、`port`、`restart_policy` |在计算机上创建并启动长时间运行的后台服务 |
| `computer_service_delete` | `computer_id`，`service_id` |删除后台服务（如果正在运行，请先停止它）。 |
| `computer_service_list` | `computer_id` |列出计算机上注册的所有后台服务。 |
| `computer_service_logs` | `computer_id`, `service_id` |从后台服务检索最近的日志输出（最后 100 行）。 |
| `computer_service_restart` | `computer_id`, `service_id` |重新启动后台服务（停止然后启动）。 |
| `computer_service_start` | `computer_id`，`service_id` |启动已停止的后台服务。 |
| `computer_service_stop` | `computer_id`, `service_id` |停止正在运行的后台服务（发送 SIGTERM）。 |
| `computer_set_clipboard` | `computer_id`、`text` |设置剪贴板文本内容。 |
| `computer_set_wallpaper` | `computer_id`、`path` | 设置桌面壁纸。接受计算机内的绝对路径 |
| `computer_set_window_position` | `computer_id`、`window_id`、`x`、`y` |将窗口移动到给定的屏幕坐标。 |
| `computer_set_window_size` | `computer_id`、`window_id`、`width`、`height` |将窗口大小调整为给定的宽度和高度（以像素为单位）。 |
| `computer_start` | `computer_id` |启动已停止的计算机。 |
| `computer_stat_file` | `computer_id`, `path` |获取计算机内的文件/目录元数据（大小、类型、权限、运行时间）。 |
| `computer_stop` | `computer_id` |停止正在运行的计算机。 |
| `computer_template_create` | `workspace_id`、`name`、`template_type`、`size`、`selected_apps`、`items` |在工作区中创建新的计算机模板。 |
| `computer_template_list` | `workspace_id` |列出工作区中可用的计算机模板。 |
| `computer_type` | `computer_id`, `text` |在当前聚焦的字段中输入文本。 |
| `computer_update` | `computer_id`、`name`、`metadata` |重命名计算机或更新其元数据。 |
| `computer_upload_file` | `computer_id`、`local_path`、`remote_path` |将本地文件从主机上传到计算机的给定路径。 |
| `computer_viewer_password` | `computer_id` |显示是否设置了原始外部桌面查看器密码。经过身份验证的平台桌面链接不需要此密码。 |
| `computer_wait` | `computer_id`, `seconds` |在计算机内部暂停执行 N 秒。 |
| `computer_windows` | `computer_id` |列出桌面上所有打开的窗口及其 ID、标题和位置。 |
| `computer_write_file` | `computer_id`、`path`、`content` |将文本内容写入计算机内的文件路径。 |
