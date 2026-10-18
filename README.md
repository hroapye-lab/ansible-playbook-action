# ansible-playbook-action

安装 Ansible collections 后，在指定工作目录执行 `ansible-playbook`。

## Inputs

| 输入 | 必填 | 默认 | 说明 |
|---|---|---|---|
| `working_directory` | 否 | `.` | 执行目录，相对仓库根目录 |
| `playbook` | 是 | — | Playbook 文件，相对执行目录 |
| `inventory` | 否 | 空 | Inventory 文件，相对执行目录；留空时使用 `ANSIBLE_INVENTORY` 或 `ansible.cfg` |
| `requirements` | 否 | `requirements.yml` | requirements 文件清单，每行一个，相对仓库根目录 |
| `check` | 否 | `true` | `true` 只检查；`false` 才实际变更 |
| `limit` | 否 | 空 | `--limit` |
| `tags` | 否 | 空 | `--tags` |

## 行为

- 先安装 collections，失败则不会执行 playbook。
- 始终带 `--diff`。
- `check` 默认 `true`，只有明确传 `false` 才实际变更。
- `requirements` 支持多行清单，空行和 `#` 开头的行忽略。
- `requirements` 相对仓库根目录；`playbook` / `inventory` 相对 `working_directory`。
- 不负责 checkout、安装 Python/Ansible、配置 SSH key 或管理 secret。

## 使用

```yaml
- uses: actions/checkout@v4

- uses: actions/setup-python@v5
  with:
    python-version: "3.12"

- name: Install Ansible
  run: pip install --requirement requirements.txt

- uses: shimataro/ssh-key-action@v2
  with:
    key: ${{ secrets.ANSIBLE_SSH_PRIVATE_KEY }}
    name: ansible_key
    known_hosts: unnecessary
    if_key_exists: replace

- uses: https://gitea.example.com/owner/ansible-playbook-action@v1
  with:
    working_directory: freeswitch
    playbook: freeswitch.yml
    inventory: hosts.yaml
    requirements: freeswitch/requirements.yml
```

如果 Gitea 配置了 `[actions].DEFAULT_ACTIONS_URL = self`，可以简写为：

```yaml
uses: owner/ansible-playbook-action@v1
```

生产环境建议锁到具体 tag 或 commit SHA。
