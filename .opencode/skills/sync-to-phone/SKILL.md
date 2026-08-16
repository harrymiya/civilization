---
name: sync-to-phone
description: |
  把本仓库（/root/civilization）同步到手机 /sdcard/Download/civilization（= /mnt/sdcard/Download/civilization）的经验 skill。
  当用户说"复制到手机""同步手机""复制一下""弄到手机上""跟手机同步"以及任何把 repo 拷贝到 /sdcard/Download/civilization 的请求时使用。
  触发词：复制、复制到手机、同步、手机、sdcard、/mnt/sdcard、Download/civilization、传到手机、手机路径。
  Use ONLY when working on this repo (/root/civilization)。只用于本仓库→手机 sdcard 的同步，不用于其他项目。
---

# 同步本仓库到手机

> 目标路径是固定的：`/sdcard/Download/civilization`（在本机 Termux 里等价于 `/mnt/sdcard/Download/civilization`）。
> 别问路径，别探索，直接执行。整仓同步，不做选择。

## 1. 目标与源

- 源：`/root/civilization`
- 目标：`/mnt/sdcard/Download/civilization`（即 `/sdcard/Download/civilization`）

## 2. 直接执行命令（一次性跑完）

```bash
cp -a /root/civilization/. "/mnt/sdcard/Download/civilization/" >/tmp/cp_err.log 2>&1
```

注意：

- 用 `cp -a /root/civilization/.` （带尾点），不是 `cp -a /root/civilization`，这样直接覆盖目标内容，不产生嵌套目录。
- **绝对不要先 `rm -rf` 目标**。sdcard 是 Android FUSE 挂载，递归删除会报 `Directory not empty`（空目录也删不掉，是已知 FUSE bug），删不干净还浪费时间。
- 覆盖复制即可，效果等同全新同步。

## 3. 预期错误（都是正常的，不必惊慌）

以下错误必然出现且无害，忽略即可：

- `cp: preserving times ... Permission denied` —— sdcard FUSE 不允许改时间戳，正常。
- `cp: cannot create symbolic link ... Permission denied` —— sdcard 不支持符号链接，主要发生在 `.opencode/node_modules/.bin/*`，正常。
- `exit code 1` —— 因为上面的符号链接失败，属于预期。

`.git` 目录部分内容、`node_modules` 里的符号链接在 sdcard 上无法完整保留，不影响阅读。

## 4. 校验（跑完一定要校验）

### 4.1 顶层文件 md5 对比

```bash
for f in README.md PERSONAS.md ROADMAP.md WRITING_LOG.md ARCHITECTURE.md .gitignore; do
  a=$(md5sum "/root/civilization/$f" 2>/dev/null | cut -d' ' -f1)
  b=$(md5sum "/mnt/sdcard/Download/civilization/$f" 2>/dev/null | cut -d' ' -f1)
  [ "$a" = "$b" ] && echo "OK $f" || echo "DIFF $f"
done
```

### 4.2 卷目录 diff

```bash
for d in "卷0 序章" "卷1-起源" "卷2-发现" "卷3-存在" "卷4-文明" "卷5-终极" "卷6-回声" "卷7-跋"; do
  diff -rq "/root/civilization/$d" "/mnt/sdcard/Download/civilization/$d" 2>&1 | grep -i "differ\|Only in /root" && echo "!!! $d 有差异"
done
```

无输出 = 全部一致。

## 5. 已知坑（重要）

- **`ls` 可能不显示新文件**：sdcard FUSE 目录缓存 bug，`ls` 会漏显示刚复制/已存在的文件（如 README.md 在 `ls` 里看不到）。**判断是否同步成功永远用 md5 对比 / `diff`，不要用 `ls` 下结论。**
- 如果曾经用 `rm -rf` 删过目标并报 `Directory not empty`，不要纠缠，直接覆盖复制即可。
- 别把目标当成可编辑的 git 仓库去操作，它就是给手机 App（Obsidian 等）读的副本。