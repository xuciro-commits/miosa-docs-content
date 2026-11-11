---
slug: "mcp/tools/computers"
title: "Computer tools"
description: "Hosted MIOSA MCP computer tools: lifecycle, files, checkpoints, env, domains, and the desktop surface (screen, pointer, keyboard, windows)."
section: "CLI, SDKs & MCP"
order: 11026
---

Every tool in this family, exactly as the hosted server registers it. Arguments
are the tool's own parameter names; optional ones are still listed, because the
description says what they change.

| Tool | Arguments | Description |
|---|---|---|
| `computer_accessibility_tree` | `computer_id` | Get the AT-SPI accessibility tree for the current desktop state. |
| `computer_bash` | `computer_id`, `command`, `timeout` | Execute a bash command on the computer and return stdout + stderr. |
| `computer_checkpoint_create` | `computer_id`, `comment` | Save the current state of the computer as a checkpoint (snapshot). |
| `computer_checkpoint_delete` | `computer_id`, `checkpoint_id` | Delete a saved checkpoint. |
| `computer_checkpoint_list` | `computer_id` | List all saved checkpoints for the computer. |
| `computer_checkpoint_restore` | `computer_id`, `checkpoint_id` | Restore the computer to a previously saved checkpoint. |
| `computer_click` | `computer_id`, `x`, `y`, `button` | Click a mouse button at the given screen coordinates. |
| `computer_close_window` | `computer_id`, `window_id` | Close a window. |
| `computer_copy_file` | `computer_id`, `source`, `dest`, `recursive` | Copy a file or directory tree inside the computer. |
| `computer_create` | `name`, `template_type`, `size`, `workspace_id`, `external_workspace_id`, `external_project_id` | Create a new MIOSA computer and wait until it is active. |
| `computer_delete_file` | `computer_id`, `path` | Delete a file or directory inside the computer. |
| `computer_destroy` | `computer_id` | Permanently destroy a computer. |
| `computer_domain_add` | `computer_id`, `fqdn` | Add a custom domain to the computer. Returns CNAME verification |
| `computer_domain_delete` | `computer_id`, `domain_id` | Delete a custom domain mapping from the computer. |
| `computer_domain_list` | `computer_id` | List all custom domains registered for the computer. |
| `computer_double_click` | `computer_id`, `x`, `y` | Double-click at the given screen coordinates. |
| `computer_drag` | `computer_id`, `from_x`, `from_y`, `to_x`, `to_y` | Click-and-drag from one screen position to another. |
| `computer_env_delete` | `computer_id`, `name` | Delete an environment variable from the computer. |
| `computer_env_list` | `computer_id` | List all environment variables set on the computer. |
| `computer_env_set` | `computer_id`, `name`, `value` | Set (create or update) an environment variable on the computer. |
| `computer_expose_port` | `computer_id`, `port`, `protocol` | Expose a port on the computer and return the public URL. |
| `computer_focus_window` | `computer_id`, `window_id` | Bring a window to the foreground by its window ID. |
| `computer_get` | `computer_id` | Get details and current status of a computer. |
| `computer_get_clipboard` | `computer_id` | Read the current clipboard text content. |
| `computer_get_cursor_position` | `computer_id` | Get the current mouse cursor position (x, y). |
| `computer_get_desktop_env` | `computer_id` | Get desktop environment info (name, resolution, session type). |
| `computer_get_screen_size` | `computer_id` | Get the screen resolution (width and height in pixels). |
| `computer_hotkey` | `computer_id`, `keys`, `items` | Press a keyboard shortcut (multiple keys simultaneously). |
| `computer_key` | `computer_id`, `key` | Press a single key. Use standard key names: |
| `computer_key_down` | `computer_id`, `key` | Press and hold a key without releasing it. Pair with computer_key_up. |
| `computer_key_up` | `computer_id`, `key` | Release a previously held key. |
| `computer_launch` | `computer_id`, `app` | Launch an application by name (e.g. 'firefox', 'gedit', 'xterm'). |
| `computer_list` | - | List all computers in the tenant. |
| `computer_list_files` | `computer_id`, `path` | List directory contents on the computer. |
| `computer_list_ports` | `computer_id` | List all currently exposed ports on the computer. |
| `computer_list_regions` | - | List available compute regions with GPU availability details. |
| `computer_logs` | `computer_id`, `lines` | Get recent VM-level logs from the computer. |
| `computer_maximize_window` | `computer_id`, `window_id` | Maximize a window. |
| `computer_minimize_window` | `computer_id`, `window_id` | Minimize (iconify) a window. |
| `computer_mkdir` | `computer_id`, `path`, `recursive` | Create a directory (and any missing parents) inside the computer. |
| `computer_mouse_down` | `computer_id`, `x`, `y`, `button` | Press and hold a mouse button at (x, y). Pair with computer_mouse_up to release. |
| `computer_mouse_up` | `computer_id`, `x`, `y`, `button` | Release a held mouse button at (x, y). |
| `computer_move_cursor` | `computer_id`, `x`, `y` | Move the mouse cursor to the given coordinates without clicking. |
| `computer_network_policy_get` | `computer_id` | Get the current network policy (firewall rules) for the computer. |
| `computer_network_policy_reset` | `computer_id` | Reset the network policy for the computer to the platform default (allow all). |
| `computer_network_policy_set` | `computer_id`, `rules`, `items`, `default_effect` | Set the network policy (firewall rules) for the computer. |
| `computer_preview_url` | `computer_id`, `port` | Return the public preview URL for a given port on the computer. |
| `computer_read_file` | `computer_id`, `path` | Read a file from inside the computer and return its content as text. |
| `computer_rename_file` | `computer_id`, `source`, `dest` | Rename or move a file/directory inside the computer. |
| `computer_restart` | `computer_id` | Restart a computer. |
| `computer_right_click` | `computer_id`, `x`, `y` | Right-click at the given screen coordinates. |
| `computer_rotate_viewer_password` | `computer_id` | Rotate and return the raw external desktop viewer password once for sharing a raw *.computer.miosa.ai link. |
| `computer_screenshot` | `computer_id` | Capture a PNG screenshot of the computer desktop. |
| `computer_scroll` | `computer_id`, `direction`, `clicks`, `x`, `y` | Scroll in a direction on the desktop. |
| `computer_service_create` | `computer_id`, `name`, `command`, `working_dir`, `port`, `restart_policy` | Create and start a long-running background service on the computer |
| `computer_service_delete` | `computer_id`, `service_id` | Delete a background service (stops it first if running). |
| `computer_service_list` | `computer_id` | List all background services registered on the computer. |
| `computer_service_logs` | `computer_id`, `service_id` | Retrieve recent log output from a background service (last 100 lines). |
| `computer_service_restart` | `computer_id`, `service_id` | Restart a background service (stop then start). |
| `computer_service_start` | `computer_id`, `service_id` | Start a stopped background service. |
| `computer_service_stop` | `computer_id`, `service_id` | Stop a running background service (sends SIGTERM). |
| `computer_set_clipboard` | `computer_id`, `text` | Set the clipboard text content. |
| `computer_set_wallpaper` | `computer_id`, `path` | Set the desktop wallpaper. Accepts an absolute path inside |
| `computer_set_window_position` | `computer_id`, `window_id`, `x`, `y` | Move a window to the given screen coordinates. |
| `computer_set_window_size` | `computer_id`, `window_id`, `width`, `height` | Resize a window to the given width and height in pixels. |
| `computer_start` | `computer_id` | Start a stopped computer. |
| `computer_stat_file` | `computer_id`, `path` | Get file/directory metadata (size, type, permissions, mtime) inside the computer. |
| `computer_stop` | `computer_id` | Stop a running computer. |
| `computer_template_create` | `workspace_id`, `name`, `template_type`, `size`, `selected_apps`, `items` | Create a new computer template in a workspace. |
| `computer_template_list` | `workspace_id` | List computer templates available in a workspace. |
| `computer_type` | `computer_id`, `text` | Type text into the currently focused field. |
| `computer_update` | `computer_id`, `name`, `metadata` | Rename a computer or update its metadata. |
| `computer_upload_file` | `computer_id`, `local_path`, `remote_path` | Upload a local file from the host machine into the computer at a given path. |
| `computer_viewer_password` | `computer_id` | Show whether the raw external desktop viewer password is set. Authenticated platform desktop links do not need this password. |
| `computer_wait` | `computer_id`, `seconds` | Pause execution inside the computer for N seconds. |
| `computer_windows` | `computer_id` | List all open windows on the desktop with their IDs, titles, and positions. |
| `computer_write_file` | `computer_id`, `path`, `content` | Write text content to a file path inside the computer. |
