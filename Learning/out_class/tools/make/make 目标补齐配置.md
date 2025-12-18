
- 下面介绍一种利用 Bash 自动补全功能实现 make 命令补全当前目录 Makefile 目标的方法
### 1. 安装 bash-completion

- 确保系统已安装 bash-completion（Ubuntu 默认通常已安装，但如没有可执行以下命令）：

``` shell
sudo apt-get update 
sudo apt-get install bash-completion
```

### 2. 添加补全脚本

- 在所使用环境空间中，编辑 `~/.bashrc` 文件，在文件末尾添加以下代码

```shell
sudo vim ~/.bashrc
```

- 补全脚本：

```bash
_make_target_completion() {
    local cur="${COMP_WORDS[COMP_CWORD]}"
    local targets_all targets_builtin user_targets
    local tmp_all tmp_builtin

    # 从当前目录的 Makefile 中提取所有目标
    targets_all=$(make -rqp 2>/dev/null | awk -F':' '/^[a-zA-Z0-9][^$#\/\t=]*:([^=]|$)/ {print $1}' | sort -u)

    # 使用空 Makefile 提取内置目标（即不含用户自定义内容）
    targets_builtin=$(make -rqp -f /dev/null 2>/dev/null | awk -F':' '/^[a-zA-Z0-9][^$#\/\t=]*:([^=]|$)/ {print $1}' | sort -u)

    # 将两个列表写入临时文件，再利用 comm 命令取差集（即去除内置目标）
    tmp_all=$(mktemp)
    tmp_builtin=$(mktemp)
    echo "$targets_all" > "$tmp_all"
    echo "$targets_builtin" > "$tmp_builtin"
    user_targets=$(comm -23 "$tmp_all" "$tmp_builtin" | grep -v '^Makefile$')
    rm -f "$tmp_all" "$tmp_builtin"

    COMPREPLY=( $(compgen -W "$user_targets" -- "$cur") )
}
complete -F _make_target_completion make

```

- 这是一种基于 `make` 内部数据库的方案，通过对比实际 `Makefile` 与空 `Makefile`（即没有用户目标）得到的内置目标，自动排除内置目标，只保留用户定义的目标

### 3. 使配置生效

- 保存 `~/.bashrc` 后，运行下面命令使配置立即生效：

```bash
source ~/.bashrc
```