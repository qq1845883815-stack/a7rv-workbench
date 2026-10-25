# 构建与离线验证

1. 下载并核验 0.1.0 mac-arm64 发行 ZIP，解压应用；解压源码归档。
2. 在 Apple Silicon/macOS 14+ 上安装 Xcode 命令行工具。
3. 使用 Python 3.11+ 执行（会编译 Swift，输出目录必须不存在）：

```sh
python3 build_app.py --engine "/path/to/A7R V 工坊.app/Contents/Resources/engine" --output "dist/A7R V 工坊.app"
"dist/A7R V 工坊.app/Contents/MacOS/A7RVWorkbench" --selfcheck
codesign --verify --deep --strict "dist/A7R V 工坊.app"
```

`engine-source/` 会覆盖外部运行环境中的对应源码；运行环境的 Python、扩展、库仍来自已核验发行包。原生机内模板使用 `build-assets/` 中封存的 AArch64 ELF。对应 C 源码在 `native/`，信任写入模板含生成配置，采集模板的配置生成依赖固定上游 prepare.py；本发布不宣称所有原生依赖和模板均可一条命令从零重建。无需连接相机即可构建。

离线测试使用 pytest，不连接或操作相机：

```sh
A7RV_ENGINE="/path/to/A7R V 工坊.app/Contents/Resources/engine" python3 -m pytest src/tests -q
```

默认跳过三项需要私人原始镜像的验证。若自行持有完整且匹配的原始备份，可在本地额外设置 `A7RV_STOCK_IMAGE=/path/to/partition1.logical.bin`，运行全部 29 项。不要上传镜像或把它放入仓库。

`--selfcheck` 仅核验应用资源身份，不是设备兼容性测试。重新构建会改变封条与应用签名；应在审查修改后核对新的构建产物。

发布前已清空两个采集 ELF 的非加载调试节，并逐字节确认所有 PT_LOAD 段完全不变。处理脚本为 `sanitize_debug.py`，资源摘要已同步更新。
