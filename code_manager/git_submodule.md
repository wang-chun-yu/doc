# Git Submodule 概念、用法与实践

## 1. 概念

**Submodule** 让「父仓库」在指定路径下**固定引用**另一个 Git 仓库的某个 **commit**（不是「随便跟踪对方分支头」）。

- 父仓库里会多一条记录：路径 → 远端 URL + **已检出的 commit SHA**。
- 克隆父仓库时，默认**不会**自动拉齐子模块的完整历史；需要显式初始化/更新。
- 适合场景：依赖第三方库、复用独立维护的包、把大仓库拆成多个仓库但主工程要「钉死」版本。

与 **subtree**、**复制 vendor 目录** 的区别（简述）：

| 方式 | 特点 |
|------|------|
| submodule | 子目录是独立仓库；版本是 commit；协作者需熟悉 submodule 工作流 |
| subtree | 子树合并进父历史；无「嵌套 .git」；历史可能更臃肿 |
| 直接拷贝 | 简单但难同步上游、难审计版本 |

---

## 2. 常用命令

### 添加子模块

```bash
git submodule add <repository-url> <path>
```

会在 `.gitmodules` 里登记 URL 与路径，并把子模块当前 HEAD 的 commit 记入父仓库索引。

### 首次克隆带子模块的仓库

```bash
git clone --recurse-submodules <url>
# 或克隆后：
git submodule update --init --recursive
```

### 拉取父仓库更新后，同步子模块到父仓库记录的 commit

```bash
git pull
git submodule update --init --recursive
```

### 在子模块里切到新版本并提交到父仓库

```bash
cd path/to/submodule
git fetch
git checkout <branch-or-tag>   # 或 git pull 后停在某 commit
cd ../..
git add path/to/submodule
git commit -m "Bump submodule to ..."
```

父仓库的 diff 体现为「子模块指针从旧 SHA 变到新 SHA」。

### 一次性更新所有子模块到远端跟踪分支的最新（慎用，需明确团队约定）

```bash
git submodule update --remote --recursive
git add -A
git commit -m "Update submodules to remote tracking branches"
```

需在 `.gitmodules` 中为子模块配置 `branch = ...` 时，`--remote` 才有明确目标分支。

### 查看状态

```bash
git submodule status
```

---

## 3. 实践注意点

1. **CI / 新同事**：文档里写清 `clone --recurse-submodules` 或 `submodule update --init --recursive`。
2. **父仓库提交**：改子模块内容后，既要**在子模块里 commit/push**（若子模块有独立远端），也要在**父仓库**里 `git add <submodule-path>` 并 commit，否则别人拉到的仍是旧指针。
3. **Detached HEAD**：`submodule update` 后子模块常在 detached HEAD；若要在子模块长期开发，应 `checkout` 到分支再改。
4. **URL 变更**：改 `.gitmodules` 后执行 `git submodule sync`，必要时改本地 `git config submodule.*.url`。

---

## 4. 实际例子（ROS 2 / 多仓库工作区）

假设主仓库是 `ros2_test_env`，想固定使用上游 `openarm_ros2` 的某一版本：

```bash
cd /path/to/ros2_test_env
git submodule add https://github.com/example/openarm_ros2.git src/openarm_ros2_ws/vendor/openarm_ros2
git commit -m "Add openarm_ros2 as submodule"
```

他人克隆：

```bash
git clone --recurse-submodules <your-main-repo-url>
```

日后上游修复 bug，在子模块目录更新并提升父仓库指针：

```bash
cd src/openarm_ros2_ws/vendor/openarm_ros2
git fetch origin
git checkout v1.2.3   # 或某 commit
cd ../../../../
git add src/openarm_ros2_ws/vendor/openarm_ros2
git commit -m "Bump openarm_ros2 to v1.2.3"
```

构建脚本里可先确保子模块就绪：

```bash
git submodule update --init --recursive
colcon build ...
```

---

## 5. 何时不用 submodule

- 团队不熟悉 submodule，且频繁在子目录里双向改代码 → 考虑 monorepo、subtree 或包管理器。
- 需要「把对方代码完全打进父仓库历史、无独立远端」→ subtree 或复制 + 注明版本可能更简单。

---

## 参考

- [Git 官方文档：Git Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
