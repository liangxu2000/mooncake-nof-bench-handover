# 交接文件（单向：开发指挥 Agent → 测试执行 Agent）

- 最后更新：2026-10-30
- 仓库：`https://github.com/liangxu2000/mooncake-nof-bench-handover`（public，可匿名拉取，无需凭据）
- 首次：`git clone https://github.com/liangxu2000/mooncake-nof-bench-handover.git`
- 之后每轮：`cd mooncake-nof-bench-handover && git pull`
- 权限：**只 pull，不要 push**；本仓不接收反向提交
- 回执：按《接管说明》§1.3 格式整理后交给用户转交

## 协作约定

1. **方向单向**：本文件只由开发指挥 Agent 更新并 push；测试执行 Agent 只 pull，不 push。
2. **回执走人工通道**：测试 Agent 按《接管说明》§1.3 格式产出回报（结论 / 证据 / 未做 / 风险与偏差 / 产物），由用户转交给开发指挥 Agent。
3. 任务编号 `E3-A`、`E3-B`…；状态：`待执行` / `执行中` / `待回执` / `已关闭`。
4. 任务正文统一用《接管说明》§1.2 模板。
5. **资料与回执格式**：只给结论、≤20 行原始输出片段、`文件:行号`、diff；脚本/env 摘录 ≤15 行，CSV 只给相关行（≤5 行）。
6. 若出现代码补丁，放在 `patches/`，一律 `git diff`（必要时 `-U1` 压缩上下文）+ `base64 -w0` 单行 + patch/b64 两份 md5；落地前先 `git apply --check`。
7. **代码保管（长期）**：只有**要进 Mooncake git 仓的源码改动**（C++/头文件/构建配置等）必须把最小 diff + `base64 -w0` + sha256 回传给开发指挥 Agent；由外部本地归档、核对并在最终阶段统一提交 git。**仓外测试脚本/harness（如 `mc_bench_point.sh`）不需要回传 diff**；只需在 R*/meta.md 记录 env、命令、脚本 md5，供复现即可，因为它们不会进 git。测试 Agent 继续只 pull、不 push；回执不传整文件、构建产物或日志。

## 结果与交付约定（从 E3-B 起生效）

1. **交付表只增不改**：权威表 `测试相关资料/我们测试场景的测试结果benchmark.xlsx` 与参考表
   `测试相关资料/benchmark 主要看表格Mooncake Benchmark.xlsx` 都保持原样；每次填写前复制到
   `测试相关资料/交付表/benchmark_Mooncake_<批次号>_<YYYYMMDD>.xlsx`，只编辑副本。
   `交付表/README.md` 记录最新版本、填写批次与改动摘要。
2. **results 按批次组织**（替代当前扁平目录）：

   ```
   results/
     INDEX.md
     第二次测试结果&分析/        ← 接管后所有批次（R04 起）
       R<nn>_<YYYYMMDD>_<主题>/
         <内容>.csv | <内容>.txt
         meta.md
     R01–R03（第一次测试，保持原位）
   ```

   - 批次号全局递增、不复用；文件名不含日期（日期在目录名）。
   - **R04–R06 移入「第二次测试结果&分析」，INDEX.md 一并移入；此后新批次（R07 起）都放这里。**
   - `meta.md` 必填：任务号、起止时间、git commit、补丁状态、GDR 状态、池大小、关键 env、节点 load、异常。
   - `INDEX.md` 一行一批次：编号 | 日期 | 主题 | 关键结论 | 文件 | 任务号。
   - 存量 7 个 CSV 归档到 R01/R02/R03 并改语义文件名；旧名→新路径映射写进 `INDEX.md`。
3. **时延与吞吐一起报**：每个点至少给吞吐 GiB/s + 时延 min/avg/p50/p90/p99/p99.9/p99.99/max（µs 或 ms）；
   分析与交付表都要覆盖时延。16/16 的时延标注为排队时长（过载点），避免误读。
   权威表列映射（已核对）：K=keys/s、L=queries/s、M=min、N=avg、O=P50、P=P90、Q=P99、
   R=P99.9、S=P99.99、T=Pmax、U=吞吐 GiB/s。
4. E3-A 产物先按原路径落；E3-B 统一归档并建立 `INDEX.md`。

## 资料请求（文本摘录）

> 只贴关键行。

1. ~~昨日 d=878 的 CSV 行与 client env~~：已收到，无需再补。
2. `mc_bench_point.sh`：bench 实际命令行 + 脚本里所有 `export`/env 行（合计 ≤15 行）。
3. E2 程序：用**文字**说明三种源的分配方式（API 名：cudaMallocHost / malloc / mmap?）与计时循环位置即可，不贴代码。
4. ~~重启/变更时间线~~：已收到。
5. 判据 3 的 `/proc/<pid>/maps`、`numa_maps` 原始行（跑点时抓，≤20 行）。
6. 如确需 CSV，只贴相关行（每份 ≤5 行），不传文件。

## 任务历史

| 编号 | 日期 | 内容 | 状态 | 回执摘要 |
| --- | --- | --- | --- | --- |
| E3-A | 2026-10-27 | 暂存池 NUMA/pin 定位 + 拷贝源标定 | 已关闭 | H1 成立：池落点决定 d 延迟；2G@N2 1060 / 512M@N0 921 µs；池大小非主因；4 个补充点因 NoF 掉链作废 |
| E3-B | 2026-10-27 | 绑核控制池落点 + 结果归档/xlsx 副本 | 已关闭 | 绑 N0 后 d 1/1 1050→776 µs、极差 217→3.2 µs；c 462；b 940；段泄漏 32→32；xlsx 未找到 a/b 行 |
| E3-C | 2026-10-27 | 高并发绑核验证 + xlsx 核对 | 已关闭 | 绑 N0 过载点不反噬（d 16/16 +24%）；协议定稿；xlsx 误判已更正（解析器 bug，权威表与 CSV 一致）；b 按定稿协议重测（1/1 644–662 µs） |
| E3-D | 2026-10-27 | 最终 16 点全表（修订 v2） | 已关闭 | 40/40 有效、40/40 落 N0、canary 漂移 ≤0.65%、NoF 掉链 1/41 |
| E3-E | 2026-10-27 | 4MB 结果填表（用户清空副本） | 已关闭 | 4MB 表 16×11 填完，回读 OK=16 / MISMATCH=0 |
| E3-F | 2026-10-27 | 8MB 全量复测 + 填表 | 已关闭 | 8MB 表填完；b 16/16 91% TRANSFER_FAIL，待 E3-G 修复 |
| E3-G | 2026-10-27 | 8MB b 16/16 单格修复（UB staging pool 4G） | 已关闭 | 4G 修复成功：3 轮 failed=0，9.43 GiB/s / 211648 µs；R10 已更新，回读 OK=16 |
| E3-H | 2026-10-27 | 下载第二次测试分析报告到本地归档 | 待执行（第 4 次重拷） | 第三次重拷是 E3-L2 前版本；E3-L2 后报告再更新，需重新复制 |
| E3-I | 2026-10-28 | 4MB 1/1 时延打点拆分（query/read/copy/free） | 已关闭 | control 41–54；copy 416（pageable 384+32）；b−a 259；UB 178/337；12.69 = PCIe HBM write |
| E3-J | 2026-10-28 | PCIe 链路核对（V100 / UB NIC，只读） | 已关闭 | GPU 3.0 x16；Mellanox 4.0 x8；UB 不在 PCIe；12.69 是 PCIe HBM write、b UB 22.5 |
| E3-K | 2026-10-28 | UB 带宽 / urma_check.sh 单位核对 | 已关闭 | 46.4–46.9 GiB/s；脚本 MiB/s 无换算；E3-L 更正：-Q 是 perftest 攒批；b 21.9 单请求；node2 不再独占 |
| E3-L | 2026-10-28 | 4 MiB UB CQ moderation 核对 | 已关闭（源码） | CQ moderation 假设推翻；Mooncake 无设备 moderation；perftest -Q 是 post-list 攒批；b 21.9 单请求 |
| E3-L2 | 2026-10-28 | b 1/1 在飞 1→2→4 + b 16/16 UB 段 | 已关闭 | UB 恒 180–181（预期 178→120→90 被否）；copy 414→6589，聚合 ~9.5；b 下限 594、pin 569；500+ 不可达；16/16 8.67 可复现 |
| E3-M | 2026-10-28 | 读取用户提供的对照 xlsx（只读、只回文本） | 已关闭 | 1 个 xlsx / 42 有效行；CPU-only 4MB Host→Host/NoF；独立验证 b/d 的 UB/NoF 段与 GPU 瓶颈；原始数据不落公开仓 |
| E3-O | 2026-10-29 | RAID0 条带核对 + 512KB NoF chunk A/B | 已关闭 | 条带 64 KiB（SPDK Raid0，5× ES3000 V6）；128KB=2× 条带、512KB=8×；A/B 无收益（Δ≤0.63%），保持 128KB；R16 |
| E3-P | 2026-10-29 | 补 R16 分段列定义 + c/d 关键行 | 已关闭 | 228/337 同埋点；337 是 E3-I 重尾批次，R16 225；copy/query 稳定；R16 CSV 无 residual/total；batch>1 仅 transfer_data 无埋点，scatter_copy 仍有 |
| E3-Q | 2026-10-29 | 读取同事 stripe size 对照数据（xlsx） | 已关闭 | 两份文件不同层；256KB vs 512KB 线程不一致（E=32 vs 6），不受控；5SSD b1T1 128KB 拐点、64KB 差；客户端 chunk 保持 128KB；阵列 strip 改动需重建，未做 |
| E3-R | 2026-10-29 | node2 Raid0 strip 64→256KB 重建可行性核查（只读） | 已关闭 | 可行；`-z 256` 合法；重建清空阵列；仅 1.2GB 自写数据、无他人数据；停机 5–10s；env 干净；做法 A（改脚本+重启）推荐；执行前再查占用 |
| E3-S | 2026-10-29 | node2 Raid0 strip 64→256KB 重建 + c/d 测试 | 已关闭（回滚 64KB） | 256KB 变差：c 1/1 −6.5%、d −2.5%、16/16 持平；p50 改善但 p99 变差；已回滚 64KB；R17；交付表未动 |
| E3-T | 2026-10-29 | 汇总 R16/R17 c/d 时延对照表 | 已关闭 | c 1/1 avg +6.7%/P99 +15%；d P50 −4.7%/P99 +7.9%；16/16 持平；latency_64K_vs_256K.md/csv；P90/P99.9/P99.99/Pmax 未在 summary.csv |
| E3-U | 2026-10-29 | 4 MiB 单请求 vs 多请求：UB/PCIe 线速验证 | 已关闭 | UB 单请求 86.4 µs（45.1 GiB/s）；PCIe 266 不可达，pinned 334.8/pageable 383；Mooncake UB 180=栈 +94；copy 414=pageable+31；R18 |
| E3-N | 2026-10-30 | pinned 客户端 buffer 池 A/B（b 主目标；d 对照） | 已关闭 | copy 411/415→336 µs；b 568 / d 681；16/16 9.99 / 11.38；报告 §3.9 |
| E3-N-R | 2026-10-30 | E3-N 最小 diff 回传与本地归档 | 已关闭 | 3 文件/34 行；sha256 a19d…；d371e227 apply 成功；client 池析构 unregister 未接 |
| E3-V | 2026-10-30 | UB 180 µs 拆解：client submit/wait + worker queue/post/CQ | 已关闭 | submit 27.0 / wait 153.0；post→CQ 56.4/片；残差 66.2 是 32 片流水尾巴；batch=16 两池都在用 |
| E3-W | 2026-10-30 | UB 切片大小 A/B：64K/128K/256K/512K/1M/4M（4MB b 1/1） | 已关闭 | 63.9→1 片 total 180.3→157.0（−12.9%）；wait 恒 153；切片只占 ~23，固定 ~67 |
| E3-X | 2026-10-30 | UB 固定 wait ~67 µs：poll spin/sleep A/B（4MB b 1/1） | 已关闭 | sleep=0：wait 153→90.3、total 178.3→94.7；4M+0 b 1/1 483.8；固定 67 来自检测周期 |
| E3-Y | 2026-10-30 | busy poll 的 16/16 代价：sleep 0/10/50 × slice 4M（4MB b） | 已关闭 | 16/16 11.41 GiB/s 三档不变；CPU 6.5→16.8 核；4M vs 64K +15.7%；保留默认 sleep |
| E3-Z | 2026-10-30 | 4M slice 外推：8MB b 16/16 + d 16/16（default sleep） | 已关闭 | 8MB b 4M 11.43 vs 64K 10.20（+12.1%）；d 11.39/11.39 无差异（NoF 不消费 slice） |
| E3-AA | 2026-10-30 | 中间 slice 选择：16/16 的 256K/512K/1M vs 4M（4MB/8MB b） | 已关闭 | 4MB 稳健选 1M（96.1%，3/3 过阈）；8MB 512K 99.0%；1M 可作共用配置 |
| E3-AB | 2026-10-30 | 1M slice 的 1/1 验证与单配置折中（4MB/8MB b） | 已关闭 | 1M=4M（483.29 vs 483.77）；sleep 0/10/20/50 avg 483/511/522/551；sleep 斜率≈1 |
| E3-AC | 2026-10-30 | 1M slice 的 16/16 CPU/吞吐：sleep 0/10/50（4MB/8MB b） | 已关闭 | 16/16 三档吞吐不变；sleep0 +9.71 核、sleep10 +1.82 核；保留默认 50，走方案 A |
| E3-AD | 2026-10-31 | 方案 A 全场景复测：4MB a/b/c/d × 1/1/16/16（1M、1/1 sleep0、16/16 sleep50） | 已关闭 | 24 点零失败、零回退；a 1/1 +8.5%、b +35.5%、c +0.4%、d +11.5%；16/16 a/c 持平、b 10.97、d 11.39 |
| E3-AE | 2026-10-31 | 方案 A 补齐并发：4MB a/b/c/d × 1/16 + 16/1（sleep 0/50） | 已关闭 | 1/16、16/1 全部饱和；sleep0 零收益、16/1 a +10.12 核；最终只有 1/1 用 sleep0 |
| E3-AF | 2026-10-31 | 第三次测试：4MB 全表（干净）+ 1/1 打点拆解（独立诊断批） | 待执行 | — |

---

# E3-A（已关闭 · 修订 v2）

> 回执摘要（2026-10-27）：
> - H1 成立：池 NUMA 落点决定 d 延迟；池大小不是主因。
> - d(2G)@N2 中位 1060 µs；d(512M)@N0 中位 921 µs（r1 895）。
> - 标定：pinned 334 / 4K@N0 411.5 / 4K@N2 501.2 µs；跨 NUMA +90，4K/TLB +77。
> - 4 个补充点因 NoF 控制器掉链作废；r3 双慢窗口存疑；本轮未跑 b。
> - 代码事实与段泄漏见 E3-B 背景。

> 修订 v2（2026-10-27）：测试回执发现今日 E1 未传 `--local_buffer_size`（默认 512 MB），
> 而昨日 d=878 的 CSV 明写 `local_buffer_size=2G`。两者不是同一口径，原判据 2/3 的跑法
> 由下方「修订 v2 跑法」替代；其余判据不变。

## 背景（为什么做这一轮）

1. **E3 假设 1 已从代码层排除**：客户端 `scatter_host_to_maybe_device` 对每个 key 只做一次 4 MB `cudaMemcpy`；128 KB 分片发生在 NoF 提交层（`mooncake-store/src/transfer_task.cpp`，默认 `MC_NOF_SUBMIT_CHUNK_BYTES=128K`），c 路径同样承担这笔成本。**不要**再做"32×128KB 小拷贝"对比实验。
2. **新假设 H1**：client 暂存池由 `SpdkWrapper::Alloc(size, 4K, socket_id=-1)` 分配（SPDK DMA 大页），全仓只在 store segment（且仅 tcp/rdma 等协议）做 `cudaHostRegister`，该池未 pin；scatter 用同步 `cudaMemcpy`。E2 只标定了 pinned（335 µs）/ pageable（384 µs），**从未测过"从实际池子拷"**。
3. **P0 口径修正**：c 本身稳定（484 µs），d−c 从昨天的 ~385 µs 涨到 ~567 µs（+~180 µs）。差值增长落在"池 → HBM"这一段；`socket_id=-1` 使池的 NUMA 落点不可控（V100 在 NUMA0），可解释"只有 d 退、c 不退"。

## 指令（§1.2 模板）

```
【目标】定位 d−c 多出的 ~180 µs，验证"暂存池 NUMA 落点/未 pinned"假说。本轮零重编译。
【范围】client 节点：stress_cluster_bench + 现有 E2 的 nvcc 拷贝程序；只读 /proc；不碰 target 节点、不改 Mooncake 源码。
【允许】只读核查；c/d 1/1 各 3 轮交替（60 s/轮）；写/跑独立拷贝标定程序；读 /proc/<pid>/numa_maps、nvidia-smi。
【需报批】任何重启（master/client/target）、任何 rmmod/驱动变更 —— 一律先回报，等批准。
【判据】(每条附原始输出片段，不接受只报退出码)
1) 快照（口径同《接管说明》§6）：GDR 模块版本判定、master 段列表、client 段地址、NoF 注册 endpoint
   指向当前 target、a/b/c/d 各抽 1 key 可读、client 节点 load。
2) c 1/1 ×3、d 1/1 ×3 交替跑：GiB/s、avg/p99、三计数器（cuda_io_submitted/host_io/staged_bytes）、client 节点 load。
3) 池属性：bench 进程 2 GB 映射在 /proc/<pid>/maps 的落地（/dev/hugepages? 匿名?）、
   /proc/<pid>/numa_maps 的 N0/N1/N2/N3 分布；贴原始行。
4) 拷贝标定（4 MB H2D ×300 次）：
   a) cudaHostRegister 过的 pinned（应复现 ~335 µs）
   b) 普通 pageable（应复现 ~384 µs）
   c) 2 GB 大块内存（大页或普通均可），用 numa_alloc_onnode 分别落在 NUMA0 与非 NUMA0，
      在其中轮转 4 MB 偏移拷贝
   各报 avg/min/p99。
【产物】results/e3a_copy_source_20261028.csv（新增）+ 台账 §29 追加；回报按 §1.3 格式。
【失败处理】c 也明显变慢 → 停手回报；判据 1 不满足 → 停手，不自动重启。
```

## 修订 v2 跑法（以此为准）

1. **先读文件、后动手**：`git pull` 后确认任务状态；如与本仓描述和本地实测冲突，以本地原始证据为准，并回报冲突点。
2. **补资料**：按上方"资料请求"补齐；只贴关键行（脚本/env ≤15 行）。
3. **判据 1 快照核查**：用现有只读探测脚本 + 现查，不重启任何服务；key 探针批准（≤10 s、只读、不写数据）。
4. **配对 A/B（替代原判据 2）**：3 轮 × 4 点 = 12 个 60 s 点，顺序固定：
   - 轮1：`c(2G)` `d(2G)` `c(512M)` `d(512M)`
   - 轮2、轮3 同上
   每点记 GiB/s、avg/p99、三计数器、client 节点 load；点间留 10–30 s。
   脚本只允许增加 `--local_buffer_size=<2147483648|536870912>`，改完贴出改动的 ≤10 行。
5. **判据 3 池属性（修订）**：在第 1 轮的 `d(2G)` 与 `d(512M)` 运行中各抓一次
   `/proc/<pid>/maps` 落地类型与 `/proc/<pid>/numa_maps` 的 NUMA 分布；另抓 bench 进程的
   `MC_*` / `SPDK_*` env（只贴相关几行）。
6. **判据 4 拷贝标定**：配对 A/B 完成后再做（若 `d(2G)` 已回到 ~878 µs，仍做一份留基线）。
7. **回报**：按 §1.3 五段式；每个判据附原始输出片段。
   - 配对 A/B 结果落 `results/e3a_pool_ab_20261028.csv`（建议列：`case,pool_bytes,round,GiBps,avg_us,p99_us,cuda_io,host_io,staged_bytes,load`）
   - 拷贝标定落 `results/e3a_copy_source_20261028.csv`（建议列：`mode,size_bytes,iterations,avg_us,min_us,p99_us,bandwidth_GiBps,source_numa,dst`）
   - 台账 §29 追加
8. **停手条件**：判据 1 不满足；c 明显变慢；任何步骤需要重启或改驱动 —— 先回报等批准。
9. **其它（已确认）**：bench 跑在 client 节点，target 节点只做 target；preflight 只记 load，不用于 PASS/FAIL；
   client 节点大页 2048→2096 记台账并补一行 `HugePages_Free/Used`；GPU PCI 地址与文档不一致（27:00.0 vs 23:00.0），
   贴一行 `lspci`/`nvidia-smi` 原文确认并记台账。

## 回报要求

- 固定格式（§1.3）：【结论】【证据】【未做/未验证】【风险与偏差】【产物】
- CSV 建议列：`mode,size_bytes,iterations,avg_us,min_us,p99_us,bandwidth_GiBps,source_numa,dst`
- 判据 3 的抓取建议在 d 点位运行中做（进程存活 60 s，足够）
- 若环境无 libnuma，可用 `numactl --membind=0` / `--membind=<非0>` 跑同一个二进制
- 本轮不需要回答"怎么修"，只要把 4c 与 4a 的差距、以及池子实际落点报清楚

## 下一步

见下方 E3-B。

---

# E3-B（已关闭）

> 回执摘要（2026-10-27）：
> - `numactl --cpunodebind=0` → 池 100% N0；d 1/1 1050.4→775.7 µs（−26%），极差 217→3.2 µs；
>   b 1099.3→940.3（−14.5%）；c 492.9→462.0（不退化）。
> - 段泄漏修复：`--global_segment_size=0`，15 次运行 32→32。
> - 落点分解：N3 1050/1097；N0 不绑线程 879.9；N0+绑线程 775.7。
> - NoF 掉链 2/13（15%）；最终协议必须"失败即重跑"。
> - xlsx：未找到 a/b 行，仅 c/d 行 60–67；表头与数据列错位；已复制 R01 副本，原表未动。

> 承接 E3-A：H1 成立——暂存池的 NUMA 落点决定 d 的延迟，池大小不是主因。
> 本轮把落点变成可控变量，并验证运行时绑核不伤害其他 case，据此确定最终测量协议。

## 背景（E3-A 已确认）

1. 配对 A/B：d(2G) 中位 1060 µs（池落 N2）；d(512M) 中位 921 µs（落 N0）；r3 双慢与 NoF 控制器掉链同一窗口，作废。
2. 拷贝标定：pinned 334.0 / 4K@N0 411.5 / 4K@N2 501.2 µs；跨 NUMA +90 µs，4K/TLB +77 µs；观测差 165–167 µs 可被解释。
3. **池落点可控、不用改代码**：`--local_buffer_size` 池在 `protocol=ub` 下走
   `ub_allocate_memory` → `ub_allocate_memory_onnode(..., -1)` → `numa_alloc_local`，
   落点由**分配线程当时所在 CPU 的 NUMA** 决定。运行时 `numactl --cpunodebind=0`
   可把分配线程固定在 N0，从而控制池落点。
4. **段泄漏有解**：bench 默认 `--global_segment_size=4GB`，staging 模式没有清零
   （只有 gdr-peermem 模式强制 0）；每个 bench 进程会 mount 一个 4GB 段，TERM 后留
   0 容量残留段（18→31）。read-only 场景应显式 `--global_segment_size=0`。
5. 修正：GPU 在 `0000:27:00.0`（NUMA0）；`0000:23:00.0` 是 iBMC VGA。bench 跑在
   initiator 节点（有 V100 的那台）。

## 指令（§1.2 模板）

```
【目标】验证 numactl --cpunodebind=0 能否把暂存池稳定落到 N0，且不伤害 c/b；据此确定最终测量协议。
【范围】initiator 节点（GPU 侧）；target 节点只做 target；不改 Mooncake 源码。
【允许】改本地脚本（--global_segment_size=0、--local_buffer_size=2G、可选 numactl 前缀）；
        跑点；读 /proc；清理我们自己的残留段（先回报路径）。
【需报批】重启 master/client/target、改驱动、动共享面。
【判据】(每条附原始输出)
1) 健康：NoF 控制器/段状态、dmesg nvme 错误、master 段数与容量、GDR 模块。
2) 10 个点（1/1，4 MB，60 s，--local_buffer_size=2G）：
   - d：默认 ×3 / numactl --cpunodebind=0 ×3，交替；
   - c：默认 ×1 / 绑 N0 ×1；
   - b：默认 ×1 / 绑 N0 ×1。
   每点：GiB/s、avg/p50/p99/max、三计数器、initiator load、池落点（numa_maps 相关行）。
3) 段泄漏：加 --global_segment_size=0 后跑 2 个点，master 段数不增长；若仍增长，报残留段 ID 与容量。
4) 落点必须逐点抓，不能只抓一轮。
【产物】results/R05_20261028_E3B_绑核控制/（CSV + maps + meta.md）
【失败处理】NoF 掉链/环境退化 → 停手，作废该窗口；绑 N0 后 c 变慢 >5% → 停手，
            转"只绑池不绑线程"的最小补丁方案。
```

## 执行顺序

1. 健康检查 + 段数基线。
2. 脚本改动（≤15 行）：`--global_segment_size=0`、`--local_buffer_size=2G`、可选 `numactl` 前缀；备份；`bash -n`。
3. 跑 2 个点验证段不增长。
4. 跑 10 点矩阵，逐点抓落点。
5. Part 2（归档，可在跑点间隙做）：
   - 按「结果与交付约定」整理 R01–R05、`INDEX.md`、`meta.md`；
   - 复制原 xlsx 到 `测试相关资料/交付表/benchmark_Mooncake_R01_20261026.xlsx`，原表不动；
   - 以文本贴出 xlsx 表头（列名+单位）与 a/b/c/d 四行现有值（每行 ≤1 行）。
6. 回执（§1.3）。

## 决策规则

- 绑 N0 后池稳定落 N0、d 方差收敛、c 不退化 → 采用 `numactl --cpunodebind=0` 作为最终全表测量的固定协议。
- 绑 N0 伤 c（>5%）→ 不用进程级绑核；下一步只把池绑到 N0（`ub_allocate_memory_onnode` + env，最小补丁），另立任务。
- 段泄漏未解决 → 先不跑 a/b 的全表；c/d 不受影响。

---

# E3-C（已关闭）

> 回执摘要（2026-10-27）：
> - 绑 N0 在全部并发档不反噬：d 16/16 吞吐 7.59→9.41 GiB/s（+24%）、延迟 −19%；d 16/1 +37%；d 1/16 +41%；
>   b 16/16 +5.4%；c 16/16 打平。绑核落点 8/8 = 100% N0。
> - 本轮 15 次 bench 零 NoF 掉链、段数 32→32。
> - b 1/1 绑 N0 ×3 = 652 µs（极差 1.5%）；与 E3-B 的 940 µs 差 30%。
>   假设：E3-B 的 b 跑在 seq10 NoF 掉链之后的劣化窗口内；E3-C 窗口干净。最终全表用 canary 把关。
> - xlsx 更正（17:40）：上一轮"槽位/列错位"判断作废，根因是测试 Agent 的 XML 解析器 bug（自闭合单元格）；
>   权威表 `我们测试场景的测试结果benchmark.xlsx` 与原始 CSV 逐格一致：a R3–6、b R7–10（c/d 行号待确认）。
>   用户已确认暂不填表。
> - b 重测（R07，定稿协议）：1/1 5.94 GiB/s / 657 µs（6 次 644–662）；1/16 8.36 / 7476；
>   16/1 9.17 / 6818；16/16 8.62 / 115914。E3-B 的 940 µs 为单次异常值，撤回"跨批次不可复现"的说法。

> 承接 E3-B：绑 N0 在 1/1 的收益成立，但 16/16 等过载点未验证；xlsx 的 a/b 行位置与列映射未澄清。
> 本轮只做验证与澄清，不改代码；通过后即可定稿最终 16 点协议。

## 背景（E3-B 已确认）

- 绑 N0：d 1/1 775.7 µs / 5.03 GiB/s，极差 3.2 µs；c 462.0（不退化）；b 940.3。
- 落点 + 线程绑定两个因素都在起作用：N3 1050/1097；N0 不绑 879.9；N0+绑 775.7。
- `--global_segment_size=0` 修复段泄漏（32→32）。
- NoF 掉链 2/13（15%）；最终协议必须"失败即重跑"。
- b 的 `cuda_io/host_io` 均为 0，路径证明只能靠 `--gpu_mode=staging` + 延迟量级（既有埋点缺口）。

## 指令（§1.2 模板）

```
【目标】验证绑 N0 在 16/16、16/1、1/16 不反噬；确认 b；澄清 xlsx 行/列；定稿最终协议。
【范围】initiator 节点；target 节点只做 target；不改 Mooncake 源码。
【允许】跑点；读 /proc；a/b 10 s 冒烟；沿用 E3-B 的 MC_POOL_BYTES / MC_GLOBAL_SEG / MC_NUMA_PREFIX。
【需报批】重启、改驱动、动共享面。
【判据】(每条附原始输出)
1) 健康 + 段数；a/b 各 1 个 10 s 冒烟（验证残留段不影响发现）。
2) 高并发矩阵（1/1 除外）：
   - 16/16：c / b / d × {默认, 绑 N0} = 6 点；
   - 16/1：d × {默认, 绑 N0} = 2 点；
   - 1/16：d × {默认, 绑 N0} = 2 点。
   每点：GiB/s、min/avg/p50/p90/p99/p99.9/p99.99/max、三计数器、load、池落点。
3) b 1/1 绑 N0 ×3（确认 −159 µs）。
4) xlsx 澄清（只读，不改表）：
   - 列出所有 sheet 名；
   - 贴 Mooncake benchmark 页 A/B 两列第 1–80 行（或 40–70 行）；
   - 确认 a/b 是否存在于其他页/其他行；
   - 说明 c/d 数据行的实际列映射（min/avg/P50/P90/P99/P99.9/P99.99/Pmax/吞吐/keys/s/queries/s）。
5) NoF 掉链：记录；每点失败重跑 1 次；同一 case 连续 2 次失败 → 停手。
【产物】results/R06_20261027_E3C/（CSV + maps + meta.md）；xlsx 澄清文本。
【失败处理】绑 N0 在任一过载点比默认慢 >5% → 停手回报；不自动切换协议。
```

## 决策规则

- 绑 N0 在 16/16、16/1、1/16 均不慢于默认（±5%）→ 定稿协议：
  `--local_buffer_size=2G`、`--global_segment_size=0`、`numactl --cpunodebind=0`。
- 若绑 N0 在过载点明显反噬 → 改为"仅低并发绑 N0"或"只绑池不绑线程"（最小补丁），另议。
- xlsx 若确实没有 a/b 行 → 最终交付只填 c/d；a/b 数据仍保留在 CSV，等用户确认是否需要另加行。

---

# E3-D（修订 v2 · 待执行）· 最终 16 点全表

> 承接 E3-C 补充：b 已用定稿协议重测 6 点（R07，1/1 644–662 µs），作为复现性对照。
> 为避免再次跨批次拼接，本轮在同一窗口内重跑全部 16 点（a/b/c/d）；R07 的 b 只做对照，不直接混入最终表。
> xlsx 仍不动（用户已确认暂不填表）。

## 定稿协议

```
--local_buffer_size=2G  --global_segment_size=0  numactl --cpunodebind=0
失败即重跑 1 次
```

## 指令（§1.2 模板）

```
【目标】同一窗口、定稿协议下跑完 16 点（40 个样本），产出最终交付数据。
【范围】同 E3-C；本轮不动 xlsx。
【允许】跑点；沿用脚本；读 /proc；只读查看 load 来源。
【需报批】重启、改驱动、动共享面、清理他人进程。
【判据】(每条附原始输出)
1) 开跑 canary：c 1/1 与 b 1/1 各 10 s（绑 N0）。参考带：c 460–500 µs；b 644–680 µs。
   任一超出 → 停手回报（说明窗口不一致）。
2) 16 点：a/b/c/d × (1/1, 1/16, 16/1, 16/16)。
   1/1、16/1 各 3 轮；1/16、16/16 各 2 轮；共 40 点。
   每点：GiB/s、min/avg/p50/p90/p99/p99.9/p99.99/max、三计数器、load、池落点。
3) 失败即重跑 1 次；同一 case 连续 2 次失败 → 停手。
4) 收尾 canary：c/b 1/1 各 10 s；首尾漂移 >5% 标注。
5) 权威表核对（只读）：贴 `我们测试场景的测试结果benchmark.xlsx` 第 1–20 行 A/B 列，
   确认 a/b/c/d 四组行号与 K–U 列映射（K=keys/s、L=queries/s、M=min、N=avg、O=P50、P=P90、
   Q=P99、R=P99.9、S=P99.99、T=Pmax、U=吞吐 GiB/s）。
【产物】results/第二次测试结果&分析/R08_20261027_最终16点/（CSV + meta.md）；INDEX.md 更新。
【失败处理】canary 异常或环境退化 → 停手；不自动降级协议。
```

## 备注

- R05/R06/R07 一并移入「第二次测试结果&分析」。
- 用户已确认：4MB/8MB 两轮结果分别填入 `测试相关资料/第二次测试结果&分析/4MB-...xlsx` 与 `8MB-...xlsx`（见 E3-E/F）；原表仍不动。
- 开跑前若 load 明显偏高（>15），先只读排查来源并回报；不清理他人进程。

---

# E3-E（待执行 · E3-D 完成后）

> 目标：把 E3-D 的 4MB 全量结果填进用户清空后的交付表副本；不动原表、不改表头。

## 输入

- 数据：`results/第二次测试结果&分析/R08_20261027_最终16点/` 的 E3-D CSV。
- 表格：`测试相关资料/第二次测试结果&分析/4MB-我们测试场景的测试结果benchmark.xlsx`
  （已清空数据，保留表头/行标签）。
- 列映射：K=keys/s、L=queries/s、M=min、N=avg、O=P50、P=P90、Q=P99、
  R=P99.9、S=P99.99、T=Pmax、U=吞吐 GiB/s。
- 行映射：按 B 列场景名 + G 列 batch 确认 a/b/c/d（预计 a R3–6、b R7–10、c R11–14、d R15–18；
  以实际表为准）。

## 规则

1. 写入前备份同目录副本 `*.bak_<ts>`。
2. 用真正的 xlsx 库（openpyxl 等）读写；禁止正则解析 XML。
3. 每格取“Avg 中位”的那一轮全部指标；2 轮的格取最接近两轮均值的一轮；meta 记录各轮 Avg 与极差。
4. 只写数据格，不改表头/格式，不动其他 sheet。
5. 写完回读 16 行 × 11 列，与 CSV 对照，贴校验结果。
6. 若 4MB 表不存在，或 B/G 列无法映射到 a/b/c/d → 停手回报，不自行新建/改标签。

## 产物/回执

- 填好的 4MB xlsx；回执按 §1.3，附行/列映射与轮次选择规则。

---

# E3-F（待执行 · E3-E 完成后）

> 目标：对象大小 4MB → 8MB，同一套四场景跑完 16 格，填入 8MB 交付表副本。

## 前置与协议

- 协议：`--local_buffer_size=4G`（8MB × 16×16 = 2 GiB 在飞，2G 不够）、
  `--global_segment_size=0`、`numactl --cpunodebind=0`、失败重跑 1 次。
- 预填：memory 段与 NoF 段的 100 keys 重新预填为 8MB 对象（旧 4MB 数据会被替换）；
  预填后抽 key 验证可读、failed=0。
- 不重启 master/client/target；若需要重启 → 停手。

## 指令（§1.2 模板）

```
【目标】8MB 对象、同一协议下跑完 16 格，填入 8MB 表。
【范围】同 E3-D；不动原表。
【允许】预填 8MB 数据；跑点；读 /proc；写 8MB 表副本。
【需报批】重启、改驱动、动共享面。
【判据】
1) 预填校验：a/b/c/d 各抽 1 key（8MB）读取成功。
2) canary：c/b 1/1 各 10 s，failed=0，NoF 无掉链；池落点 N0。
3) 16 点：a/b/c/d × (1/1,1/16,16/1,16/16)；1/1、16/1 各 3 轮，1/16、16/16 各 2 轮；共 40 点。
   每点：GiB/s、min/avg/p50/p90/p99/p99.9/p99.99/max、三计数器、load、池落点。
4) 填表：`测试相关资料/第二次测试结果&分析/8MB-我们测试场景的测试结果benchmark.xlsx`，
   规则同 E3-E。填表前确认数据行 C 列（对象大小）为 8MB；若仍为 4MB，仅改数据行 C 列为 8MB
   并注明，表头不动。若 8MB 表不存在 → 停手回报，不自行新建。
5) 记录：NoF target max_io=4MB 下 8MB 对象的分片行为；池 4G 与 4MB 轮 2G 的差异写进 meta。
【产物】`results/第二次测试结果&分析/R09_20261027_8MB/`（CSV + maps + meta）；
        填好的 8MB xlsx；台账 §34。
【停手】预填失败、canary 失败、NoF 掉链、需要重启 → 停手留记录。
```

## 无人值守约定

- E3-D 正常结束后，自动执行 E3-E，再执行 E3-F；无需等待批准。
- 任一步失败或需要重启：停手，写清失败点、已完成产物与下一步建议，不自行重启。

---

# E3-G（已关闭）· 8MB b 16/16 单格修复

> 回执摘要：1G 默认 → 91% TRANSFER_FAIL；4G → 3 轮 failed=0、9.43/9.47/9.43 GiB/s；
> 8G 同样成功，4G 为最小可用值。8MB 表 R10 已更新（9.43 GiB/s / 211648 µs），
> 其余 15 行未动，回读 OK=16 / MISMATCH=0。211.6 ms 是 2 GiB 在飞的排队时长
> （Little's law 2 GiB ÷ 9.43 GiB/s ≈ 212 ms），不是退化。

> E3-F 唯一问题：8MB b 16/16 有 91% TRANSFER_FAIL，4 次重试一致，4G/8G 客户端池均无效。
> 根因已定位（代码）：UB transport 的 staging pool 默认 1 GiB
> （`ub_transport.cpp:38` `kDefaultUbStagingPoolSize = 1<<30`；env `MC_UB_STAGING_POOL_SIZE`），
> 8MB × 16 × 16 = 2 GiB 在飞，超过 1 GiB；4MB × 16 × 16 = 1 GiB 刚好卡边界，
> 所以 4MB 成功、8MB 失败。客户端 buffer pool（4G/8G）不是同一个池，调它无效。

## 指令（§1.2 模板）

```
【目标】用 4G UB staging pool 重跑 8MB b 16/16，修复 8MB 表第 10 行。
【范围】initiator 节点；只跑 b 16/16，不动其他 15 格。
【允许】设 MC_UB_STAGING_POOL_SIZE=4294967296；跑点；更新 8MB 表 R10；更新 R09 meta/分析文档。
【需报批】重启、改驱动、动共享面。
【判据】
1) 启动日志确认 `UbTransport: registered CPU staging pool ... size=4294967296`。
2) 8MB b 16/16 ×3 轮：failed=0；GiB/s、keys/s、queries/s、min/avg/P50/P90/P99/P99.9/P99.99/Pmax。
3) 成功后按 E3-E 规则更新 8MB 表 R10（备份、回读校验）；核对 keys/s × 8 MiB ≈ 吞吐。
4) 若 4G 仍失败：依次试 2G/8G，记录失败率；仍失败则判定该格在该平台不可用，标记无效并说明。
【产物】R09 下 `retry_b1616_ubpool/` + 更新后的 8MB xlsx；台账 §36。
【停手】NoF 掉链/环境退化/需要重启 → 停手留记录。
```

## 备注

- 这是环境变量，不改代码、不重编；UB staging pool 在进程启动时分配，重启 bench 进程即可生效。
- 4G 池在 N0 上分配（numa_alloc_local + cpunodebind=0），node4 内存充足。
- 该修复只影响 b 16/16；4MB 轮与 8MB 其他 15 格不动。

---

# 最终交付状态（2026-10-27）

| 轮次 | 批次 | 格数 | 有效 | 表 | 回读校验 |
|---|---|---|---|---|---|
| 4MB | R08 | 16 | 16/16 | `4MB-...xlsx` | OK=16 / MISMATCH=0 |
| 8MB | R09 | 16 | 16/16（E3-G 修复 b 16/16） | `8MB-...xlsx` | OK=16 / MISMATCH=0 |

## 最终口径

- 4MB：`--value_size=4194304 --num_keys=100 --local_buffer_size=2G --global_segment_size=0`
  + `numactl --cpunodebind=0`（key 不带 `_vs` 后缀）。
- 8MB：`--value_size=8388608 --num_keys=100 --local_buffer_size=4G --global_segment_size=0
  --key_include_value_size=true` + `MC_UB_STAGING_POOL_SIZE=4294967296` + `numactl --cpunodebind=0`。

## 交付说明必写

1. 8MB b 16/16 的 211.6 ms 是 2 GiB 在飞的排队时长（Little's law 吻合），不是退化；
   不要与失败态的 71 ms 对比。
2. 4MB R08 与首轮 R01、日报里的第一轮数据不是同一批，不可混用。
3. `--key_include_value_size=true` 与 `--verify=true` 互斥；8MB 完整性用
   “写 100/100 + 四路径抽 key + 全量回读”三证。
4. 8MB 的 staging 路径依赖 `MC_UB_STAGING_POOL_SIZE=4294967296`；默认 1G 会在 16×16×8MB 下溢出。
   写 `4G` 会被 `std::stoull` 解析成 4（4 MiB 池），导致 94% `Memory` 失败且静默降级。
5. b 的路径计数器（cuda_io/host_io）均为 0，路径证明靠 `--gpu_mode=staging` + 延迟量级；
   既有埋点缺口。
6. **12.69/12.68 GiB/s 是 PCIe 3.0 x16 HBM write 的 86.5%**（E3-I）；UB 是传输，
   b 的 UB 读 178 µs ≈ 22.5 GiB/s；E3-J 的“UB 12.69”是过度归因。
7. 4MB 1/1 的 b 650 = UB 178 + PCIe 拷贝 416（pageable 384+32）+ query 42 + residual 12；
   组长的 500+ 假设 copy ~266，实际 416，1/1 不可达。
8. UB 单向峰值 **46.4–46.9 GiB/s**（E3-K）；b 1/1 的 UB 178 µs = 21.9 GiB/s 是单请求带宽
   （峰值 47%），不是上限被压制；比带宽用 16/16。E3-L 更正：`-Q` 是 perftest 的 post-list 攒批，
   Mooncake 不设设备 moderation。E3-K 曾发现 node2/node4 有他人 root perftest；独占状态需用户确认。
9. E3-L2 已关闭：UB 段 180–181 µs 不随并发下降；copy 414→6589 µs，聚合 ~9.5 GiB/s；
   b 1/1 下限 594 µs（pin 515+控制≈569），500+ 不可达；b 16/16 8.67 可复现。
   原“UB 178→120→90”的预期错误，不要再引用。

---

# E3-H（待执行）· 下载分析报告到本地

> 报告已在 git 仓：`reports/第二次测试结果分析报告.md`（与本文件同一仓，已 push）。
> 报告已更新（E3-L2：UB 180 恒定 + copy 墙 + 500+ 不可达）；第三次重拷是 E3-L2 前版本，
> 需第 4 次复制。

## 指令

1) pull 到最新；
2) 复制 `reports/第二次测试结果分析报告.md` 到
   `测试相关资料/第二次测试结果&分析/第二次测试结果分析报告.md`；
3) 校验文件大小与 md5，回执。

## 备注

- 只复制，不改内容；本地文件夹已有 xlsx，不要混。
- 报告含完整性能数字，按内部资料管理。

---

# E3-I（已关闭）· 4MB 1/1 时延打点拆分

> 回执摘要：四段实测；a 391 / b 650 / c 419 / d 808（R08 400/656/470/773）。
> control 41–54（不是 92）；copy b 416 / d 415（pageable 384+32，不是 266/335）；
> UB 读 b 178 / d 337；a H2D 355；c NoF 379；b−a 259。
> **12.69 = PCIe 3.0 x16 HBM write，不是 UB**；12.69 vs 14.67 = 86.5%；
> b 的 UB 178 = 22.5 GiB/s；E3-J 的“UB 12.69”是过度归因。

> 组长方向：加打点，把 avg 拆成 query/read/copy/control，避免 b−a 差值的归因错误。
> 仅测量补丁、env 门控、可回退；不改数据路径。

## 指令（§1.2 模板）

```
【目标】把 a/b/c/d 4MB 1/1 的 avg 拆成 query/select、read、scatter/copy、free 四段。
【范围】initiator 节点；本地测量补丁；1/1 ×3，四场景。
【允许】加 PerfPoint 或 steady_clock 时间戳；重编；读日志。
【需报批】重启、改驱动、动共享面。
【判据】
1) 四场景各段中位：query/select、read into host/HBM、scatter/copy、free。
2) copy 段应 266–335 µs（pinned 335 / pageable 384）；若 <266，说明口径重叠/计量错误。
3) read_host vs read_hbm 的差值；control 段是否 ~90 µs。
4) 与 E2 拷贝标定对比；a/c/d 同口径。
【产物】results/第二次测试结果&分析/R10_打点拆分/（CSV + 日志）+ 台账 §37 + 补丁 diff（可回退）。
【停手】需重启/环境退化 → 停手。
```

## 备注

- b 路径当前没有埋点（NoF staging 块也没有）；这次是第一次能直接看到 copy 段。
- 打点会引入微小开销；在 1/1 下可忽略，但需在 meta 里注明。
- 若 copy 实测 ≈335 且 read_host ≈ read_hbm − 79，则 b 656 的拆分闭环；
  若 copy 实测 <266，先查是否把 read 的一部分算进了 copy。

---

# E3-J（已关闭）· PCIe 链路核对（只读）

> 回执摘要：GPU `0000:27:00.0` PCIe 3.0 x16（8 GT/s x16，15.75 GB/s）；
> Mellanox CX-6 Dx `0000:a6:00.0` PCIe 4.0 x8（16 GT/s x8，主板仅布 8 lane，15.75 GB/s）；
> **`udmac0d1e2` 不是 PCIe 设备**，位于 `ub_bus_controller0`，驱动 `ubase`，
> port0 active_speed=12 / width=4；数据通路 `--protocol=ub` 走 UB。
> E3-J 当时判断“12.69 = UB 上限”是过度归因；E3-I 打点修正为：
> **12.69/12.68 = PCIe 3.0 x16 HBM write 的 86.5%**；UB 是传输，
> b 的 UB 读 178 µs ≈ 22.5 GiB/s；PCIe 3.0 x16 只在 a/c 直写和 host→HBM 拷贝。
> Mellanox 不是 Mooncake 数据通路。

> 目的：确认 V100 与 UB NIC 的 PCIe 代际/宽度/当前速率，决定 b/d 第一次 crossing 的下限。
> 只读，不改环境。

## 指令

1) `lspci -Dnn` 找 V100（`0000:27:00.0`）与 UB NIC（`udmac0d1e2` 对应 BDF）；
2) 对每个 BDF `lspci -vv -s <BDF>`，贴 `LnkCap` 与 `LnkSta` 原始行（Speed / Width）；
3) 若中间有 PCIe switch / 上游桥，贴其 LnkCap/LnkSta；
4) 只读，不改环境。

## 判据/产物

- GPU：LnkCap/LnkSta（预期 8 GT/s x16 / 8 GT/s x16）。
- NIC：BDF + LnkCap/LnkSta（代际 / 宽度 / 当前速率）。
- 结论一行：host→HBM 与 NIC→host 是否同为 PCIe 3.0 x16；若不同，给各自线速。
- 台账 §38。

---

# E3-K（已关闭）· UB 带宽 / urma_check.sh 单位核对

> 回执摘要：UB 单向峰值 **46.4–46.9 GiB/s**（4 MiB -Q1：write 46.37 / read 46.91；
> 64 KiB 46.92；双向 77.97；J=4 不高于 J=1）。工具列头 MB/sec 实为 MiB/s（1024²）；
> `urma_check.sh` 不换算，47 = 48048.54 四舍五入，GiB/s 引用正确。
> **E3-L 更正**：`-Q` 是 perftest 的**用户态 post-list 攒批**，不是设备 moderation；
> Mooncake 不设设备 moderation；3.96 vs 46.37 的“11.7×”只对 perftest 成立；
> E3-I 的 b UB 178 = **21.9 GiB/s**（不是 22.5），是单请求带宽（峰值 47%），不是被压制。
> **node2/node4 发现他人 root perftest，node2 不再独占**；高精度测试前需用户确认。

> 用户提供：GPU 节点 ↔ store 节点 UB 峰值约 47 GiB/s；
> 脚本 `node1 bash /home/z00926396/tools/urma_check.sh`。只读/跑工具，不改环境。

## 目标

1) 跑/读脚本，确认 47 的单位：GiB/s / GB/s / Gb/s；per-port / aggregate；
   公式是否 ×8/÷8、1000/1024。
2) 独立复核 UB 带宽（urma_perf / 多引擎），给 GiB/s。
3) 与 E3-I 的 b UB 178 µs（22.5 GiB/s 单请求）对照：47 峰值 / 22.5 单请求是否一致。

## 判据/产物

- 脚本相关行（≤20 行）+ 原始输出（≤10 行）；不贴整脚本。
- 单位推导；至少 2 种方式/2 次；结论：UB 峰值 __ GiB/s；单请求 __；是否瓶颈。
- 若无法访问 node1 / z00926396 / 脚本，停手回报，不 sudo/不改权限，由用户执行。
- 台账 §40。

---

# E3-L（已关闭）· 4 MiB UB CQ moderation 核对

> 回执摘要：E3-L 第一步（源码只读）推翻 E3-K 的 CQ moderation 机理——
> Mooncake 全仓不设设备 moderation（无 `urma_modify_jfc`；`urma_jfc_cfg_t` 无 moderation 字段；
> JFC depth=4096、jfce=NULL、纯轮询）；`urma_perftest` 的 `-Q` 是**用户态 post-list 攒批**，
> 不是设备 moderation；3.96 vs 46.37 是攒批/发数节奏，E3-K 的“11.7× 低估”只对 perftest 成立。
> **b 的 21.9 GiB/s 是单请求带宽**：perftest 饱和 46.37 → 84 µs/op；b 1/1 178 µs，
> 178/84≈2.1；1/1 无流水线，Little's law 4 MiB/178=21.9，不是上限被压制。
> 第二步原 A/B 无意义，改为 E3-L2。

> E3-K 发现：`urma_perftest` 默认 CQ moderation=100 在 4 MiB 下低估 11.7×
> （4051.73 vs 47487.20 MiB/s）；64 KiB 不受影响。
> Mooncake UB 路径的 4 MiB（E3-I b UB 178 µs = 21.9 GiB/s，约为 4 MiB -Q1 46.37 的 47%）
> 是否也受 CQ moderation 影响？

## 第一步：源码只读（不需要独占，现在可做）

1) 读 Mooncake UB transport 的 CQ 创建 / moderation 配置（源码/日志/env），
   确认是否 100/默认；
2) 只在本地仓库 grep，不改代码、不跑测试；相关行 ≤20 行；台账 §42。

## 第二步：A/B（等用户申请到独占后）

- 若可配置，跑 b 1/1（或独立 4 MiB UB read）在 moderation 关闭/降低下的对比；
- 评估 b UB 178 是否可从 21.9 提升到 46.4；b 650 是否可到 ~561；
  500+ 是否可达（copy 416 仍是主成本）。

---

# E3-L2（已关闭）· b 1/1 在飞 1→2→4 + b 16/16 UB 段

> 回执摘要（2026-10-28 17:32）：
> - 1→2→4→16 在飞：UB 段 180/180/181/181 µs，**不降**；原预期 178→120→90 被否。
> - copy 段 414/603/1460/6589 µs；copy 聚合 9.44/12.96/10.70/9.49 GiB/s，高并发停在 ~9.5。
> - b 1/1 648 µs = UB 180 + copy 414 + query 42 + residual 12；R08 656。
> - b 16/16（16×16）115145 µs / 8.67 GiB/s（8.66/8.69/8.67），R08 8.62（−0.6%）。
> - 下限：UB 180 + copy 414 = 594 µs；pin 335 → 515 + 控制 ≈ 569；500+ 不可达。
> - 两条坑：batch>1 埋点失效；`MC_UB_STAGING_POOL_SIZE` 只认字节（`4G`→4）。
> - 产物 R14_20261028_E3L2；台账 §43；报告第三次重拷已归档（E3-L2 前版本）。

## 判据回答

| 问题 | 结论 |
| --- | --- |
| UB 是否随并发下降 | 否，恒 180–181 µs |
| b 21.9 是否被压制 | 否，1 个 4 MiB 在飞的 Little's law 值；峰值 46.4–46.9 |
| copy 是否瓶颈 | 是，聚合 ~9.5 GiB/s，吞吐 2 并发即饱和 |
| b 650→561 | 否；UB 180 + copy 414 已 594 |
| 500+ 是否可达 | 否；pin 后 ≈569；需改 copy 通路或绕过 staging |
| 16/16 基线 | 8.67 vs R08 8.62，可复现 |

## 复现坑

1) batch>1 走 `batch_get_into`，E3-I 的 `get_into/transfer_data/scatter_copy` 计数为 0；
   分段只能用 batch=1。摘录：`b=1 7065 7065 140518；b=2 1 1 5；b=16 0 0 0`。
2) `MC_UB_STAGING_POOL_SIZE` 只认纯数字字节；`4G` → `stoull("4G")` = 4 → 4 MiB 池 →
   94% `Memory` 失败（failed: 3322314），日志 `registered CPU staging pool ... size=4194304`；
   必须写 `4294967296`。不抛异常、静默降级。

## 可选下一步：E3-N（pinned staging A/B，待用户确认）

- 目的：验证 copy 的 ~9.5 GiB/s 墙是否来自 pageable（每次 pin/unpin）。
- 改动：UB staging 池分配后加 `cudaHostRegister`（env 门控，只本地验证、不提交上游）；
  析构时 `cudaHostUnregister`。
- 判据：copy 414→接近 335；b 1/1 650→~569；b 16/16 8.67→是否向 11.7–13 GiB/s 靠。
- 若仍 ~9.5：墙在 copy 引擎/并发提交方式，需 dev 侧改（多流 / DMA / 分块重叠）。
- 不做也不影响交付；只有它能把 b 的吞吐往上抬。

---

# E3-M（已关闭）· 读取用户提供的对照 xlsx（只读、只回文本）

> 回执摘要（2026-10-29）：
> - 1 个 xlsx（22,638 bytes，单 sheet，58×23，42 个有效行）；CPU-only 4MB Host→Host / Host→NoF。
> - Host→Host 1/1 186.7 µs、16/16 46.95 GiB/s；Host→NoF 5SSD 16/16 44.28 GiB/s。
> - 独立验证：我们 b 的 UB 段 180 µs 与同事 Host→Host 同量级；b/d 与同事的差 ≈ copy + 控制；
>   GPU 目的地的 12.69 是 PCIe HBM write，不是 UB/NoF/SSD。
> - 原始数据用于本地对照，**不落公开仓**。

> 用户提供了一批对照测试数据，放在
> `测试相关资料/同事B测试数据/`（以实际目录名为准）下的 `.xlsx`。
> 目的：与第二次测试的 4MB/8MB、a/b/c/d 结果对照。
> **只读、不改文件、不复制/移动、不传文件、不跑测试；回执只发文本。**

## 指令

用户说明：表不大，只有几十行。**不要只给表头/前几行，所有 sheet 的全部非空单元格都要回传。**

1) 列出目录下所有 `.xlsx`：文件名、大小、mtime、md5。
2) 用 openpyxl（`read_only=True, data_only=True`）读取每个 workbook：
   - 所有 sheet 名（含隐藏 sheet）与维度（max_row × max_col）；
   - 每个 sheet **全量**非空单元格，按行导出；空行/空列可跳过，但保留行号和列号；
   - 合并单元格取左上角值；公式取缓存值，无缓存值标 `<no cached value>`。
3) 文本格式（每行一个 Excel 行）：
   `sheet_name<TAB>row_number<TAB>A=值<TAB>B=值...`
   或等价的可读 TSV；只要值，不要公式。
4) 配置/说明页、备注页也全文回传；单位/列名原样保留，不要猜、不要做汇总计算。
5) 如果某个 sheet 意外超过 500 行，先回该 sheet 的表头 + 前 50 行 + 总行数，并说明原因；
   其余 sheet 仍全量回传。

## 回执

- 文件清单（名/大小/md5）+ 每个 sheet 的维度；
- 每个 sheet 的全量文本（含表头、备注、单位）；
- 标注哪些 sheet 对应 a/b/c/d、4MB/8MB；哪些是原始/汇总。

> 解析用 openpyxl，不要用正则/字符串切割 xlsx（之前出过自闭合单元格错位）。

---

# E3-O（已关闭）· RAID0 条带核对 + 512KB NoF chunk A/B

> 回执摘要（2026-10-29）：
> - 条带：SPDK Raid0 bdev `strip_size_kb=64`；5× 华为 ES3000 V6，vfio-pci；无 Linux md/硬件 RAID。
> - 128 KiB=2× 条带、512 KiB=8× 条带，均天然对齐；131072 vs 524288 的 c/d A/B 无收益（Δ≤0.63%），保持默认 128KB。
> - 16 点 failed=0；代码未动；R16（R15 已被 E3-M 占用）。
> - 过程：首轮 abort 误命中预期的 `cuMemGetHandleForAddressRange failed rc=801`，修正后重跑；`mc_bench_point.sh` 增加 1 行 env 透传（md5 58044b62→ef6914cd）。

> 背景：同事确认其 512KB 是 RAID0 条带，initiator 按条带切 child：4MB→8 child。
> 我们默认 `MC_NOF_SUBMIT_CHUNK_BYTES=131072`（128KB）→ 32 child。
> 目标：确认我们 5× SSD 的 RAID0 条带，并做 128KB vs 512KB 的受控 A/B。
> **只改 env，不改代码；不改交付表。**

## 第一步：只读核对（node2）

1) 找到承载 NoF 的 RAID0 设备（`lsblk`、`df`、`mount`）；
2) 读条带：
   - `mdadm --detail /dev/mdX`
   - `cat /sys/block/mdX/md/chunk_size`
   - 若是硬件 RAID，记录控制器型号/条带（只读命令）。
3) 回报设备名、条带字节数、5 块盘型号；不要改阵列。

## 第二步：A/B（node4，客户端 env）

- 基准：`MC_NOF_SUBMIT_CHUNK_BYTES=131072`（默认 128KB，32 child）
- 对照：`MC_NOF_SUBMIT_CHUNK_BYTES=524288`（512KB，8 child）
- **只认十进制字节；写 `512K` 会被 `strtol` 解析成 512，校验失败后回退 128KB。**
- 其余协议不变：4MB、`--local_buffer_size=2G`、`--global_segment_size=0`、`numactl --cpunodebind=0`。
- 跑 c、d 的 1/1 与 16/16；每点 60 s；同窗口、同 NoF target。
- 1/1 用 batch=1，拿打点分段；16/16 用 batch=16，拿吞吐/avg（batch>1 无逐请求埋点）。
- 每点记录：failed、avg、吞吐、NoF/transfer 段（若有打点）、copy 段（d）。
- 若 512KB 出现 NoF 错误/失败率升高，立即停，回报。

## 判据

- 先看条带：若是 512KB，128KB 就是 4× 细切；若是 128KB，128KB 已对齐。
- 512KB vs 128KB：
  - d 1/1 NoF 段 337 → 是否接近 288–310；total 773 → 是否 ~730；
  - c 1/1 NoF 段 379 → 是否接近 340；total 470 → 是否 ~430；
  - 16/16：copy/PCIe 仍是主成本，吞吐不应有大变化；若变化 >5%，记录。
- 不改交付表；只写 R15 meta/CSV/台账。

## 产物

- R16_20261029_NoF_chunkAB：meta.md、summary.csv、progress logs（R15 已被 E3-M 占用）；
- 台账；条带核对原始行（≤10 行）。

> 备注：同事的 288.78 是 5SSD/512KB/6 线程；我们 A/B 是 1 线程，只隔离 chunk，不隔离线程。

---

# E3-P（已关闭）· 补 R16 分段列定义 + c/d 关键行

> 回执摘要（2026-10-29）：
> - 228/337 是同一埋点（`transfer_data_breakdown`，client_service.cpp:4586），代码/字段未变；
>   337 是 E3-I 重尾补跑批次，R16 干净窗口 **225**；copy 413–415、query 42 稳定。
> - R16 `breakdown_median.csv` 没有 residual/total/select；`summary.csv` 的 avg 与 breakdown 中位不是同一口径。
> - 更正：batch>1 时 `transfer_data` 无埋点，但 `scatter_copy` 仍有（d 16/16 s_sc≈14 万）。
> - d 的无 copy 段 225+42+15=**282 µs**；d 的 808 是 E3-I 批次值，交付用 R08 773。
> - 结论：337 不能当稳定基线；228 与 337 的差是时间窗/批次，不是口径。

> 目的：对齐 E3-O 的 225–228 µs 与 E3-I 的 337 µs（同一埋点，不同批次），
> 判断同事 288.78 µs 与我们的差是口径还是真实差距。
> **只读 R16 产物；不重跑、不改代码、不传文件；回执只发文本。**

## 指令

1) 读 `测试相关资料/results/第二次测试结果&分析/R16_20261029_NoF_chunkAB/`：
   - `breakdown_median.csv`：给表头 + c/d 的 1/1 两档行（128KB/512KB）；
   - `summary.csv`：给表头 + c/d 的 1/1、16/16 行；
   - `meta.md` 中与分段/计时口径有关的段落（≤30 行）。
2) 说明 `breakdown_median.csv` 每列含义，特别是：
   - `transfer` 的起止点：只含 SPDK NoF I/O，还是含 URMA/暂存？
   - `copy` 的起止点；`residual` 是什么；`total` 是否等于 query+transfer+copy+residual；
   - 是否有 `transport`/`staging` 列，若有，给值。
3) 对比 E3-I 的 `transfer` 337（d）与 E3-O 的 `transfer` 228：是同一埋点吗？
   如果 E3-O 改了埋点/列名，说明改了哪里（≤10 行）。
4) 只发文本；不贴整份 CSV；不传文件；不重跑。

## 回执

- `breakdown_median.csv` 表头 + c/d 1/1 行；
- `summary.csv` 表头 + c/d 1/1、16/16 行；
- 列定义 + E3-I/E3-O 差异说明；
- 若某列在 R16 中不存在，明确说“没有”，不要猜。

---

# E3-Q（已关闭）· 读取同事 stripe size 对照数据（xlsx）

> 回执摘要（2026-10-29）：
> - 两份文件不同层：`stripesize影响.xlsx` 是 SPDK 裸条带扫描（node1 6×7T + node3，重建 RAID0）；
>   `test2.xlsx` 是 Mooncake 256KB stripe 补测。
> - 256KB vs 512KB 线程不一致（E=32 vs 6），**不是受控对照**；16/16 无增益；不能引用“256KB 更好”。
> - SPDK 扫描：5SSD b1T1 128KB 为拐点、64KB 差；b32T4 贴 UB 峰值（strip 无影响）；1SSD 无影响。
> - 结论：客户端 chunk 保持 128KB；node2 阵列 strip 64KB，改动需重建 RAID0，未做；不追 256KB。
> - 原始数据不落公开仓。

> 用户提供：`测试相关资料/同事B测试数据/` 下的 `stripesize影响*` 和 `test2.xlsx`
> （test2 是 256KB stripe 的补测）。目的：看 stripe size 对 Host→NoF 的影响，
> 和 R16 的 128KB/512KB 对照。
> **只读、不改、不传文件、不跑测试；回执只发文本。**

## 指令

1) 列出目录下所有 `.xlsx`（文件名/大小/mtime/md5）；标出新增文件（`test.xlsx` 已读过）。
2) 用 openpyxl（`read_only=True, data_only=True`）读 `stripesize影响*` 和 `test2.xlsx`：
   - 所有 sheet 名（含隐藏）与维度；
   - 所有非空单元格，按行导出；表小，**不要截断**；
   - 合并单元格取左上角；公式取缓存值，无缓存值标 `<no cached value>`。
3) 文本格式：`sheet_name<TAB>row_number<TAB>A=值<TAB>B=值...`；只要值，不要公式。
4) 重点标注：stripe size（64/128/256/512/1024KB？）、SSD 数、Client 线程、4MB/8MB、
   1/1、16/16、Avg/P99/Pmax、吞吐、单位；配置/说明/结论页全文。
5) 如果 `test2.xlsx` 是 `test.xlsx` 的增量，说明新增了哪些行/列；如果是完整副本，说明差异。

## 回执

- 文件清单 + 每个 sheet 的维度；
- `stripesize影响*` 全量文本；
- `test2.xlsx` 全量文本（或新增部分 + 说明）；
- 配置/说明页；哪些列是 stripe size、哪些是结果。

---

# 决策记录（2026-10-29）

1. **4MB / 8MB 场景的 NoF 客户端 chunk 统一用默认 128KB**（`MC_NOF_SUBMIT_CHUNK_BYTES=131072`，不设即默认；如脚本必须写，写 `131072`，不能写 `128K`）；R16 已证明 128K vs 512K 无收益。E3-O 的 512K 只作历史诊断，**后续跑点前确认 env 未设或=131072**。
2. **2026-10-29 用户定 256KB；E3-S 实测“256KB 条带 + 128KB chunk”变差**（c 1/1 −6.5%、d −2.5%、16/16 持平），已按预案回滚 **64KB**。建议保持 **64KB**；若仍要 256KB，需同时把客户端 chunk 改成 256KB（否则 128KB 是半条带），另做决定。
3. 重建测试已完成 R17；结果变差，已回滚；当前 **64KB stripe + 128KB chunk**。
4. 256KB vs 512KB 的 test2 数据线程不一致（E=32 vs 6），不作为条带结论；需同线程补测。
5. **128KB chunk 与 256KB 条带的交互**：128KB 是 256KB 条带的半条带，连续 128KB child 会落在同一块盘，5 盘并行度下降；这可能是 256KB 变差的原因之一。64KB 条带 + 128KB chunk 是对齐的（2× 条带）。
6. 详细结论见 `reports/stripe_chunk_conclusion_20261029.md`（chunk/stripe 两个旋钮、R16/R17、暂定；后续重评估条件）。

---

# E3-R（已关闭）· node2 Raid0 strip 64→256KB 重建可行性核查（只读）

> 回执摘要（2026-10-29）：
> - 可以做：`bdev_raid_create -z 256` 语法合法（256 是 2 的幂、≥ block size）；首次重建要在 RPC 返回处确认 `strip_size_kb=256`，不能只看退出码。
> - 必须清空阵列：`superblock=false`，strip 不落盘；raid0 映射重排，旧数据不可读。
> - 数据可丢：target 启动以来累计只写 1.2 GB（300 × 4 MiB），全是自己 prefill 的 4MB 对象，无他人数据，无需备份。
> - 停机：target 5–10 s；到可跑点（prefill + 冒烟）约 2–3 min。
> - env 干净：node4 没有导出 `MC_NOF_SUBMIT_CHUNK_BYTES`；`mc_bench_point.sh` 只在显式设置时透传；不要复用 `/tmp/e3o_ab.sh`（它写死 131072/524288）。
> - 当前无 initiator、4421 无连接、无进程占用 5 块盘；node2 仍有 16 个登录会话，执行前必须再查一次占用。

> 用户决定：后续 stripe size 用 **256KB**；客户端 chunk **4MB/8MB 都用默认 128KB**；不做 128KB 对照。
> 当前 node2：5× 华为 ES3000 V6，vfio-pci 交给 SPDK，SPDK Raid0 bdev，`strip_size_kb=64`，38.4TB。
> 改成 256KB 必须重建 Raid0 bdev，**会清空阵列**；本轮只做只读核查，不重建、不停服务、不改配置。

## 指令（只读）

1) 找 NoF target 的启动方式：进程、配置文件、RPC 脚本、raid bdev 名、5 个 base bdev、namespace/NQN。
2) 确认当前 `bdev_get_bdevs -b Raid0`：strip_size_kb=64、num_base_bdevs=5、state=online。
3) 查 `rpc.py bdev_raid_create --help`，确认 strip size 参数（`-z` 单位 KB）与重建命令；
   给出“改配置重启”和“RPC 删除/重建”两种做法的具体命令。
4) 确认数据：阵列上有什么数据、是否可由 bench 重新写入（100 keys）、是否有其他用户/服务；
   是否需要备份。
5) 确认重建后要做的事：重启 target、重建 namespace/注册、initiator 是否要重新发现；给出步骤。
6) 给出回滚到 64KB 的步骤。
7) 确认测试脚本不会带入 E3-O 的 512K env；跑点前 dump `/proc/<pid>/environ`，确认
   `MC_NOF_SUBMIT_CHUNK_BYTES` 未设或=131072（4MB/8MB 都一样）。
8) 不改配置、不删 bdev、不停服务、不重建；只回文本。

## 回执

- target 进程/配置/RPC/raid bdev/namespace 清单；
- 重建 256KB 的两种命令 + 风险；
- 数据/其他用户/备份结论；
- 回滚步骤；
- 预计停机时间。

> 实际重建等用户确认维护窗口/数据可丢/其他用户后，另发 E3-S；本轮不执行。

---

# E3-S（已关闭 · 回滚 64KB）· node2 Raid0 strip 64→256KB 重建 + c/d 测试

> 回执摘要（2026-10-29 16:30）：
> - 重建成功：strip=256、online、5/5 base、nsid=1、4421 LISTEN；停机 9.636 s；回滚 9.665 s。
> - 8 点 failed=0；chunk 默认 131072。
> - c 1/1 8.16/8.09 vs 64KB 8.67/8.65（−6.5%）；d 1/1 5.01/4.95 vs 5.08/5.16（−2.5%）；16/16 持平。
> - d NoF p50 228→185（−19%），但 P99 1032→1114（+8%）；c P99 598→688（+15%）。
> - 结论：256KB + 128KB chunk 组合变差，已回滚 64KB；R16 基线恢复（8.63 vs 8.65–8.67）。
> - **关键混杂**：客户端 chunk 128KB 对 256KB 条带是半个条带，连续 128KB child 落在同一块盘，5 盘并行度下降；这不是“纯 256KB 条带”的结论。
> - 运行注意：必须显式 `MC_NOF_PORT=4421`（脚本默认 4426）；key 用 `_vs`，控制点差 <1%；node4 load 25.84 对 1/1 下降无影响。

> 用户决定：stripe 用 **256KB**；4MB/8MB 客户端 chunk 用默认 **128KB**；不做 128KB 对照。
> 前置：E3-R 已确认可行、数据可丢、当前无 initiator；**用户已于 2026-10-29 明确授权执行**。
> 重建会清空阵列；R17；**不改交付表**。

## 执行前（只读复核）

1) 确认 node2 无 initiator、4421 无连接、无进程占用 5 块盘/SPDK 实例；
2) 确认无其他用户正在使用；记录 `date +%T`；
3) 备份脚本：`cp ~/nds/scripts/node2_target_up.sh ~/nds/scripts/node2_target_up.sh.bak_strip64`。

## 重建（做法 A：改脚本 + 整体重启）

1) 把 `bdev_raid_create -n Raid0 -z 64 -r raid0` 改成 `-z 256`；
2) `grep -n bdev_raid_create` 确认已是 256；
3) `bash ~/nds/scripts/node2_target_up.sh 4194304`；
4) 复核：`bdev_raid_get_bdevs all` → `strip_size_kb=256`、`online`、5/5 base；
   `nvmf_subsystem_get_subsystems` → nsid=1→Raid0；listener 4421 LISTEN。
   **必须看 RPC 返回的 `strip_size_kb`，不能只看脚本退出码。**

## prefill + 测试

1) prefill 100 keys（R09 同款：`--scenario=segment_write --role=writer
   --nof_segments="" --replica_num=0 --nof_replica_num=1 --num_keys=100
   --batch_size=32 --key_include_value_size=true`，换新 key 空间，避免旧 key 指向空位置）；
2) 10s 冒烟 c 1/1：`failed=0`、`scatter_copy` 有样本；
3) c/d 的 1/1、16/16，2 轮 × 60s；客户端 chunk 默认 128KB：
   - 不设 `MC_NOF_SUBMIT_CHUNK_BYTES`；**不要用 `/tmp/e3o_ab.sh`**；
   - 每点 dump `/proc/<pid>/environ`，确认该变量未设或=131072；
4) 结果写 R17，与现有 64KB 基线（R08/R16）比方向；不做 128KB 对照；不改交付表；
5) 记录重建前后时间戳、strip、failed、NoF/copy 段。

## 回滚

`cp node2_target_up.sh.bak_strip64 node2_target_up.sh && bash node2_target_up.sh 4194304`；
回滚同样清空阵列，需重新 prefill。

---

# E3-T（已关闭）· 汇总 R16/R17 c/d 时延对照表

> 回执摘要（2026-10-29）：
> - 产物：R17 `latency_64K_vs_256K.md`（8,613 B）、`.csv`（2,218 B；16 行时延 / 12 行 Δ / 10 行分段）。
> - c 1/1：avg +6.68%、P50 +1.52%、P99 +15.07%；d 1/1：avg +2.71%、P50 −4.73%、P99 +7.94%。
> - 16/16：Δ 全在 1.3% 内；其数值是排队时长。
> - d NoF 段 225→187（−17%），copy 413–415 不变；c transfer 367→371，原因未定。
> - `summary.csv` 没有 P90/P99.9/P99.99/Pmax；`raw_*.log` 有，未展开。

> 用户要看 strip 64KB vs 256KB 下 c/d 的时延（avg/P50/P90/P99/P99.9/P99.99/Pmax），判断是否真的变差。
> **只读 R16/R17 产物；不重跑、不改交付表；回执给一张文本表 + 保存 md/csv。**

## 指令

1) 读 `R16_20261029_NoF_chunkAB/summary.csv` 和 `R17_20261029_strip256/summary.csv`；
2) 按 case(c/d) × 并发(1/1、16/16) × round(R1/R2) 对齐；
3) 输出一张表：
   - 行：c 1/1、d 1/1、c 16/16、d 16/16；
   - 列：64KB 的 avg/P50/P90/P99/P99.9/P99.99/Pmax（R1/R2），256KB 同列，Δ（256 vs 64，按 avg/P50/P99 给百分比）；
   - CSV 里没有的分位写“无”，不要猜。
4) 另给一张 1/1 分段表（如果两边的 `breakdown_median.csv` 都有）：
   `query / transfer(NoF) / copy / residual`；注明 R16 d=225（干净窗口）、R17 d=185。
5) 注明条件：4MB、chunk 128KB、2 轮 × 60s；R17 用 `_vs` key（控制点差 <1%）；node4 load 25.84；16/16 是排队时长。
6) 保存 `latency_64K_vs_256K.md` + `.csv` 到 R17 目录；回执贴表。

## 回执

- 一张 c/d 时延对照表（R1/R2）；
- 1/1 分段表；
- 文件名/路径；
- 哪些分位缺失。

---

# E3-U（已关闭）· 4 MiB 单请求 vs 多请求：UB/PCIe 线速验证

> 回执摘要（2026-10-29）：
> - UB 4 MiB Q1 J=1：read 86.39/86.34 µs（45.1 GiB/s），write 86.07/86.50；J=1→8 吞吐 46.7–46.9，单 jetty 打满。
> - PCIe 4 MiB：pinned K=1 p50 334.8 µs，aggregate 11.67 GiB/s；K=2/4/8 per-copy 660/1320/2646，aggregate 11.84/11.76/11.73；pageable K=1 383.0/10.20；**266 µs 不可达**。
> - Mooncake：b UB 180 = 裸 86.4 + **93.6 µs 栈**；d copy 414/415 = pinned +79 / pageable +31（8%）。
> - 模型：b 当前 648；硬件地板 86.4+334.8+54≈**475**；只 pin 569、只 UB 554、双杠杆 475–506。
> - 产物 R18；未改交付表；无 pkill/明文密码。

> 目的：验证 **83 µs（UB 46.4 GiB/s）** 和 **266 µs（PCIe 14.67 GiB/s）** 是单请求可达的时延，
> 还是只有多请求/饱和时才达到；解决 1/1 模型争议。
> **只读/标准工具；不改代码、不改交付表；结果 R18。**

## A. UB 纯传输（urma_perftest）

- 4 MiB read/write，1 QP/线程，Q1、Q2、Q4、Q8（或 J=1/2/4），30 s；
- 每个配置报 **latency avg/P50/P99** 和 throughput（GiB/s）；
- 节点 node4↔node2，同一个 `udmac0d1e2`；记录完整命令；
- **不要用旧脚本里的 `pkill` / 明文 root 密码**；直接跑 `urma_perftest`；若必须用脚本，先读脚本、只执行 perftest 部分；
- 判据：**Q1 latency ≈83 µs** → 单请求可达；**Q1 latency ≈180 µs 且吞吐 46.4** → 46.4 是多请求/流水线值。

## B. PCIe 纯 copy（4 MiB）

- 复用 E3-A 的 copy 标定程序（pinned / pageable）；1/2/4/8 个并发 copy，300 次；
- 报 per-copy latency avg/P50/P99 和 aggregate GiB/s；
- 判据：单 copy pinned≈335、pageable≈414；若 4/8 并发 aggregate 接近 12.5–14.67 GiB/s → 266 是饱和值；若单 copy 能到 266 → 单请求满线速。
- 如果现有程序不支持并发，报告，不要新写大程序。

## C. Mooncake 对照

- 现有 b 1/1 UB 180、d 1/1 NoF 225、copy 414/415；与 A/B 对照，判断 180/414 是链路还是软件栈；
- 复用 R16/R17 数据，不重跑。

## 执行前

- 确认 node2/node4 没有 bench、没有其他 UB/PCIe 大流量；共享机器，遇到占用停手回报；
- 记录时间戳、设备、QP/线程、message size、轮次。

## 产物

- R18_20261029_line_rate_check：summary.csv、latency.csv、命令、日志；台账；
- 不改交付表；结果只用于解释模型。

---

# E3-N（已关闭 · 2026-10-30）· pinned 客户端 buffer 池 A/B（b 主目标；d 对照）

> 结果已收，报告 §3.9 已更新。**计划假设被实测纠正**：pin 位置不是 UB staging 池，
> 是 store 层 `client_buffer_allocator_` 的客户端 buffer 池（b/d 共用）。
> 原计划正文保留在下方作记录；“回执”一节为准。

## 前置（代码事实）

1) 确认 b（UB transport）和 d（NoF）的 staging 池是不是同一个：
   - b：`UbTransport::acquireStaging`（`ub_transport.cpp`）分配 UB staging；
   - d：NoF 路径的 staging 可能在 `transfer_task`/SPDK；若不同池，先只 pin UB，记录 d 不变。
2) 在现有本地构建（HEAD d371e227 + E3-I 埋点）上打补丁；不要提交上游。

## 代码（本地、env 门控）

1) `UbTransport::acquireStaging` 分配池后，若 `MC_UB_STAGING_POOL_PIN=1`：
   `cudaHostRegister(base, size, cudaHostRegisterPortable)`；
   - 记录返回值、耗时、`/proc/<pid>/status` 的 `VmLck`；
   - 失败则 log，继续 pageable，不改变行为。
2) 析构：先 `unregisterLocalMemory`，再 `cudaHostUnregister`，最后 `ub_free_memory`。
3) 若 build 未链接 cudart，用本地 `dlopen` 包装或本地 CMake 加 cudart；不改上游。
4) `MC_UB_STAGING_POOL_PIN=0` 即回退；保留 diff（≤15 行）和 build 命令。

## 测试

- 4MB：`--local_buffer_size=2G`、`--global_segment_size=0`、
  `numactl --cpunodebind=0`、chunk 默认；
- `MC_UB_STAGING_POOL_SIZE=2147483648`（2G，两臂相同）；
  A 臂 `MC_UB_STAGING_POOL_PIN=0`；B 臂 `=1`；
- b/d 的 1/1（3 轮）+ 16/16（2 轮），每点 60 s，同窗口；
- 1/1 用 batch=1 拿 copy 段/total；16/16 用 batch=16 拿吞吐；
- 先 smoke b 1/1 10 s：`failed=0`、pin 成功、`VmLck` 增加；再矩阵；
- pin 失败 / `cudaHostRegister` 超时 / 内存不足：停手报告；不要用 1G 池硬跑 16/16。

## 判据

- copy 段 414→?（目标 335–366）；
- b 1/1 648→?（目标 569–600）；
- d 1/1 697→?（若共用池；否则不变，记录）；
- b/d 16/16 8.67 / 9.40→?（若 copy 墙是 pageable，应升；若不变，墙在 copy 引擎/并发）。
- 不改交付表；R19。

## 产物

- R19_20261030_pin_staging：meta.md、summary.csv、breakdown_median.csv、
  env_*.txt、raw_*.log、pin_*.log；
- 台账；diff（≤15 行）+ build 命令。

## 回执（2026-10-30，已收）

**实际 pin 位置（与计划不同）**

- 4MB b/d 都走 store 层 `client_buffer_allocator_`（`--local_buffer_size=2G`），
  **pin 打在这个客户端 buffer 池，b/d 共用**；b 场景 `registered CPU staging pool` 计数=0，
  staging 模式下 `UbTransport::acquireStaging` 池未分配。
- 因此 E3-N 实际是“pin 客户端 buffer 池”A/B，不是“pin UB staging 池”。
  原任务的运行方式（b/d、1/1+16/16、同窗口）仍成立。

**结果（4MB，A 未 pin / B pin 2 GiB）**

| 点 | A | B | 变化 |
| --- | --- | --- | --- |
| b copy 段 | 411 µs | 336 µs | −18.3% |
| d copy 段 | 415 µs | 336 µs | −19.0% |
| b 1/1 | 660.2 µs | 567.9 µs | −14.0%（吞吐等效 +16.2%） |
| b 16/16 | 8.355 GiB/s | 9.990 GiB/s | +19.6% |
| d 1/1 | 773.2 µs | 681.3 µs | −11.9%（吞吐等效 +13.5%） |
| d 16/16 | 9.095 GiB/s | 11.380 GiB/s | +25.1% |

- 20 点 `failed=0`；copy 336 与 E3-U pinned 单 copy 334.8 吻合。
- d 16/16 11.38 = E3-U PCIe pinned 聚合上限 11.73–11.84 的 96%–97%，高并发已撞 copy 墙。
- b 568 = UB 180 + copy 336 + 控制 52；离地板 475 剩 93 µs，全在 UB 栈；
  b 16/16 9.99 的上限由 UB 栈决定（压到 86.4 估可到 ~11–12）。

**pin 证据**

- `cudaHostRegister rc=0`，2 GiB，96.63 ms；拷贝微测 384.3→335.8 µs；
  `cudaPointerGetAttributes` Other→Host(pinned/registered)。
- `VmLck=0` 属预期：`pin_user_pages` 记入 `mm->pinned_vm`，不是 `locked_vm`；
  `ulimit -l` 64 MiB 不构成限制。
- 改动 3 文件 34 行，默认关闭；`cudaHostUnregister` 析构路径未测。

**口径与未决**

- A 臂 b 1/1 660.2 vs R16 648：node2 load 36–40（308 核）；只做同窗口 A/B，
  绝对值不与历史批混比。
- **8MB b 16/16 的池身份待复核**：8MB 用 `--local_buffer_size=4G`；若也走客户端 buffer 池，
  E3-G 用 `MC_UB_STAGING_POOL_SIZE=4294967296` 修复 91% 失败的根因需重查。
  复核前只保留“设 4 GiB 后 9.43 GiB/s / 211.6 ms 有效”这一事实。
- 2 GiB 常驻 pin 的生产影响未评估。
- 结论：copy 杠杆已做掉；下一步 E3-V 给 UB transport 打点，拆 180 的构成。

---

# E3-V（已关闭 · 2026-10-30）· UB 180 µs 拆解（4MB b 1/1；只测量、不改代码）

> 已收，回执见下方。**执行偏差**：batch=1 不走 `batch_get_data_breakdown`（0 行），
> 改用 `transfer_data_breakdown` 的 `submit_us / wait_us / total_us`；
> `batch_get_data_breakdown` 只在 batch>1 生效。

> 目标：E3-N 后 b 568 = UB 180 + copy 336 + 控制 52；copy 已经到底，
> 剩余 93 µs 全在 UB。先把 180 拆成 **client submit / worker 排队 / post / 完成/轮询**，
> 找到第一大头，再决定下一轮改哪里。**本轮不改代码、不改交付表；R20。**
> 仓内 d371e227 已有诊断开关（`ub_diagnostics.h`），不需要新增埋点。

## 前置（代码事实，先确认再跑）

1) `MC_UB_WORKER_DIAG=1` 打开 worker 区间统计，日志前缀 `ub_worker_breakdown`；
   字段含 `queue_wait_avg_us / queue_wait_max_us / post_call_us / post_to_cq_avg_us /
   post_to_cq_max_us / poll_call_us / active_loop_gap_max_us / endpoint_prepare_us /
   dequeue_us / posted_slices / completed_slices`。
2) `MC_HIFREQ_LOG_SAMPLE_RATE=1` 让每个请求都打 `batch_get_data_breakdown`；
   字段含 `setup_us / submit_us / wait_blocked_us / result_us / total_us`。
3) 两个 env 都是上游已有，只影响日志；若 build 里没有，停手回报，不要自己加代码。
4) 不改 node2/阵列/交付表；只在 node4 客户端加 env。
5) **顺带只读确认（不改配置）**：8MB b 16/16 走的是哪个 buffer 池。用 10 s smoke 日志
   grep `registered CPU staging pool`、`local_buffer_size`、`client_buffer_allocator`，
   确认 UB staging 池是否分配；若 8MB 也走客户端 buffer 池，E3-G 的 env 修复根因需重查。
   只回文本，不跑 8MB 全表。

## 测试（4MB b，pin B 臂）

- 协议：`--local_buffer_size=2G`、`--global_segment_size=0`、
  `numactl --cpunodebind=0`、chunk 默认、batch=1、60 s。
- 沿用 E3-N 的 pin 开启（copy 336），**只在 b 上跑**；d 不跑。
- **先做 10 s diag 开销对照**：同一窗口 diag=0 与 diag=1 各 1 次，
  若 total 偏移 >3%，`MC_HIFREQ_LOG_SAMPLE_RATE` 降到 0.1 或 0.5，并在回执注明。
- 正式：diag=1、sample=1，跑 3 轮 60 s；每轮前清日志。
- 记录 node2/node4 load、pin 返回值、`failed`。

## 要出的表

| 层 | 指标 | 期望/用途 |
| --- | --- | --- |
| client | `submit_us` | submitTransfer 墙钟（slice 分配/准备/入队） |
| client | `result_us` | 等待完成（含 worker 排队 + post + URMA + 轮询检测） |
| worker | `queue_wait_avg_us` | submit 入队 → worker 取出（唤醒/排队） |
| worker | `post_call_us` | post send 调用耗时 |
| worker | `post_to_cq_avg_us` | post → CQ（含软件轮询延迟，不是纯 DMA） |
| worker | `poll_call_us` / `active_loop_gap_max_us` | 轮询检测开销/空转间隙 |
| 合计 | `submit + result` | 应 ≈ 180；给出各层占比与残差 |

- 客户端 `batch_get_data_breakdown` 取中位；worker `ub_worker_breakdown` 取稳态区间
  （去掉第一段和最后一段），按 `completed_slices` 加权。
- 同时给 `setup_us`、`wait_blocked_us`、`total_us` 作上下文。
- 若 `post_to_cq_avg_us ≈ 86`、`queue_wait ≈ 80`，说明 180 主要是 worker 唤醒 + DMA；
  若 `post_call_us` 或 `poll_call_us` 很大，则是软件路径。**不要在本轮下改法结论，先出数。**

## 判据

- 第一大头 >60 µs 即认为定位成功，给出它属于 client/worker-post/URMA/轮询 哪一层；
- 三层相加与 180 的残差 >20 µs 时，检查 slice 数、batch、pin、日志采样开销；
- diag 开关对 total 的影响必须 <3%，否则只把 diag 当定性参考。

## 产物

- R20_20261030_ub_breakdown：meta.md、summary.csv、breakdown.csv、
  `ub_worker_*.log`（过滤后的 `ub_worker_breakdown`）、`batch_get_*.log`
  （过滤后的 `batch_get_data_breakdown`）、命令、env、load；
- 台账；不改交付表。

## 回执（2026-10-30，已收）

**结果（4MB b 1/1，pin=1，diag=1，3×60 s，failed=0）**

- diag 开销：6.77 → 6.80 GiB/s（−0.40% / +0.38%），合格；3 轮 6.85 / 6.84 / 6.83 GiB/s，
  avg 569.75 / 571.14 / 571.88 µs。
- client：`submit 27.0` / `wait 153.0` / `total 180.7` µs；
  get_into：query 38.3 / read 525.3 / total 565.3（read ≈ UB 180.7 + copy 336）。
- worker（去掉首末区间、按 `completed_slices` 加权，2 worker、118 区间/轮）：
  `queue_wait 19.9`（max 101）、`post_call 11.2/call`、`post_to_cq 56.4`（max 158）、
  `poll_call 0.10 µs/call`（≈43 万次/s，≈4% 单核，不是瓶颈）、`dequeue 0.17/slice`、
  `active_loop_gap_max 36`、`31.9 片/请求`。

| 层 | µs | 占 180.7 |
| --- | --- | --- |
| client submit | 27.0 | 14.9% |
| worker queue_wait | 19.9 | 11.0% |
| worker post_call | 11.2 | 6.2% |
| post→CQ | 56.4 | 31.2% |
| 未归因残差 | 66.2 | 36.7% |

- **第一可归因大头是 post→CQ 56.4/片（URMA 完成/链路）**，不是软件 post 路径，
  也不是 worker 唤醒；`poll_call` 仅 0.1 µs/call，排除轮询空转。
- **残差 66.2 不是漏埋点**：4 MiB 被切成 31.9×~128 KiB 并发 post，worker 指标是单切片口径，
  不能对请求级总量直接求和；请求级 `wait 153 ≈ 首片延迟 + 32 片流水尾巴`。
  `post_to_cq 56.4` 是 32 片流水下的单切片均值，不能和 E3-U 的裸 86.4（整请求）直接比。
- 本轮按任务要求**不下改法结论**；下一步 E3-W 用切片大小 A/B 验证 32 片流水是否是 +94 的来源。

**池身份更正（R19）**

- 4MB b **1/1（batch=1）**：`registered CPU staging pool = 0`；只有 client buffer 池。
- 4MB b **16/16** 与 **8MB b 16/16（batch=16）**：两个池都在用
  （client 池 + UB staging 池 2 GiB）。因此 E3-N 的 16/16 收益（+19.6% / +25.1%）
  含两个池；1/1 的 +16.2% / +13.5% 只含 client 池，无歧义。
- **8MB b 16/16 的池身份已确认**；E3-G 的 `MC_UB_STAGING_POOL_SIZE=4294967296`
  修复根因是 UB staging 池，方向正确。E3-N 报告 §3.9 需按此更正。
- 代码零改动；只给 `mc_bench_point.sh` 加 `MC_UB_WORKER_DIAG` /
  `MC_UB_WORKER_DIAG_INTERVAL_MS` 透传，不设时行为不变。

**产物**：R20 `R20_20261030_ub_breakdown`（meta、summary、breakdown、worker/transfer 日志、
pool_identity、脚本副本）；R19 下归档 `e3n_min.patch`。

---

# E3-N-R（已关闭 · 2026-10-30）· E3-N 补丁回传与本地归档

> 目的：测试 Agent 不能 push，E3-N 的 3 文件 34 行改动目前只在 node4 本地 build；
> 外部必须掌握同一份代码，后续 E3-V/优化才能对照。**只回最小 diff，不传整文件、
> 不传构建产物、不传日志。**

## 回传内容（纯文本，分条）

1) `git rev-parse HEAD`、`git status --short`、`git diff --stat <E3-N 前基线>`；
2) E3-N 的最小 diff（只含 pin 相关 hunk，含文件路径和上下文行），
   用 `base64 -w0` 编成一行；同时给 diff 的 sha256；
3) 编译命令、pin 的 env 名称/默认值/取值；
4) pin 调用点（哪个文件哪个函数）、`cudaHostUnregister` 析构是否已接；
5) 若基线里已有 E3-I/其他本地补丁，说明如何从累计 diff 中剥离 E3-N 部分。

## 判据

- diff 能 `git apply --check` 到 d371e227（或说明所需前置补丁）；
- 变更行数与回执的 3 文件/34 行一致（上下文行不计）；
- 不包含 E3-I 埋点、日志、构建脚本等无关改动；
- 回传后外部本地 `git apply` 成功，`git diff --stat` 与回执一致。

## 产物

- 外部本地 `D:\gds性能\patches\E3-N\`：patch、sha256、README；
- **不 push 到公开仓，不写进报告**；只在本地保管。

## 回执（2026-10-30，已收）

- 累计 diff 265 行（E3-I 220 + E3-N 34 + 其它 11）；E3-N 单独 diff = **3 文件 / 34 行**，
  不含 E3-I，不含脚本/日志。
- patch：3768 B / 61 行；sha256 `a19d3290d2198fb14d478306036805284fad60ecd5c394c5d335bc0e43c81983`；
  md5 `9c469b6fa1ecf00f63853393cd5ef211`。
- 在纯 d371e227 上 `git apply --check`：3 个 hunk 全部成功
  （real_client.cpp hunk#2 offset −72；ub_transport.cpp hunk#1/#2 offset −11）；
  apply 后 `git diff --stat` = 3 files / 34 insertions。外部已复核并归档。
- env：`MC_UB_STAGING_POOL_PIN`；未设 = 关闭，只有 `"1"` 才 pin。
- 调用点：
  - client buffer 池：`real_client.cpp : RealClient::setup_internal`，池创建后；
  - UB staging 池：`ub_transport.cpp : UbTransport::acquireStaging`，分配+注册后。
- 析构：`UbTransport::~UbTransport` 已接 `cudaHostUnregister`（顺序 unregisterLocalMemory →
  cudaHostUnregister → ub_free_memory，仅 pin 成功时）；**client buffer 池没有配对的
  `cudaHostUnregister`**，实验进程退出可接受，上生产需在 allocator 析构补。
- 8MB b 16/16 池身份已由 E3-V 确认：两个池都在用；E3-G 根因是 UB staging 池，方向正确。
- 本地归档：`D:\gds性能\patches\E3-N\`（patch、README、sha256），不 push 公开仓。

---

# E3-W（已关闭 · 2026-10-30）· UB 切片大小 A/B（4MB b 1/1；验证 32 片流水）

> 已收，回执见下方。**两个前提被纠正**：`MC_SLICE_SIZE` 默认是 65536（不是 131072）；
> R20 的 31.9 片算错，真实 63.9 片。切片只解释 ~23 µs，wait 恒 153 µs。

## 前置

1) 先记录当前 `MC_SLICE_SIZE`（E3-V 31.9 片/请求 ≈ 128 KiB；确认脚本是否设了 131072）。
2) `MC_SLICE_SIZE` 只认十进制字节：`131072 / 262144 / 524288 / 1048576 / 4194304`；
   写 `1M` 会被 `atoi` 解析成 1，回退默认，视为无效。
3) 每档用 worker 日志的 `posted_slices / completed_slices / 请求数` 确认实际片数变少；
   片数没变就停，先查 env 是否生效。
4) 4 MiB 单 WR 可能被 NIC/驱动拒绝；post 失败/超时立即停手，退回 1 MiB/2 MiB，不硬跑。
5) 不改 node2/阵列/交付表；只在 node4 客户端加 env。

## 测试

- 场景：4MB b 1/1；`--local_buffer_size=2G`、`--global_segment_size=0`、
  `numactl --cpunodebind=0`、batch=1、60 s；沿用 E3-N 的 pin（`MC_UB_STAGING_POOL_PIN=1`）。
- 档位：`131072`（当前基线）、`262144`、`524288`、`1048576`、`4194304`；同窗口。
- 每档 2 轮 60 s；基线/最佳档补到 3 轮；`MC_UB_WORKER_DIAG=1`、
  `MC_HIFREQ_LOG_SAMPLE_RATE=1` 只用于确认片数/分段，记录开销。
- 每档记录：transfer_data total、submit/wait、worker `posted_slices`、`slices/请求`、
  `post_to_cq`、`failed`、node2/node4 load。

## 判据

- 片数下降与 total 下降同向，且 1 片 total ≈ 86–110 µs → **+94 主要来自切片流水**；
  下一步可把 UB slice 调大（需评估目标端/注册/兼容性）。
- 片数下降但 total 基本不变 → 限制在单 WR/URMA 服务或目标端，转逐片时间线，不再调 slice。
- 任何档 `failed>0` 或 post 超时：停，不把失败点写成交付；记录错误和档位。
- 不改交付表；R21。

## 产物

- R21_20261030_slice_ab：meta.md、summary.csv、worker 过滤日志、env、命令、load；
- 台账；不改交付表。

## 回执（2026-10-30，已收）

**前提纠正**

- `MC_SLICE_SIZE` 代码默认是 **65536**（`config.cpp:264` atoi → `config.h:52` 65536 →
  `ub_transport.cpp:693` kBlockSize）；`mc_ports.env`、`~/.bashrc`、`sudo -n env` 都没有该变量。
- R20 的“31.9 片/请求”算错，真实 **63.9 片**：错用了 `completed_slices ÷ 两个 worker 区间秒数和`；
  正确是 `completed_slices ÷ 总请求数 = 6,561,984 / 102,765 = 63.85`。R20 meta 已加更正块。
- 以 65536 为真基线，共 6 档；片数完全受控，4 MiB 单 WR 被 NIC/驱动接受。

**结果（60 s ×2，基线/最佳补第 3 轮，14 点 failed=0）**

| slice | 片数/请求 | GiB/s | submit µs | wait µs | total µs | post→CQ µs | queue µs |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 65536（默认） | 63.9 | 6.80 | 26.3 | 153.3 | 180.3 | 61.7 | 19.6 |
| 131072 | 31.9 | 6.86 | 16.0 | 153.5 | 170.0 | 65.5 | 13.0 |
| 262144 | 16.0 | 7.04 | 9.5 | 153.0 | 163.0 | 66.2 | 10.2 |
| 524288 | 8.0 | 7.08 | 7.0 | 153.0 | 161.0 | 54.8 | 9.4 |
| 1048576 | 4.0 | 7.09 | 5.0 | 153.0 | 158.5 | 59.1 | 7.2 |
| 4194304 | 1.0 | 7.17 | 3.3 | 153.0 | 157.0 | 86.2 | 6.6 |

- **wait 在 6 档恒为 153.0–153.5 µs（极差 0.5）**；只有 submit 随片数单调下降 26.3→3.3。
- 1 片 `post→CQ 86.2` ≈ E3-U 裸层 86.4；1 片 `wait−post→CQ = 66.8`；
  64 片时对应值 91.6（含 queue），同量级 → **~67 µs 与片数无关**。
- `posted_slices ≈ completed_slices`（差 <0.02%），`post_failed_slices` 全档 0。
- 结论：+94 = 切片流水 ~23（全在 submit）+ 固定 ~71（86 的 URMA 往返 + ~67 固定 wait）；
  切片调大只能拿回 ~23，1 片 total 仍 157，不是 86–110。判据 A 不成立，判据 B 部分成立。
- 偏差/风险：基线从 131072 改成 65536（默认就是 64K）；post_to_cq 有偶发尾噪
  （524288_r1 345、1048576_r1 351）；65536_r1 有冷启动噪声；node4 load 40.0–40.8；
  node2 零改动；C++ 源码零改动。
- 产物：R21 `R21_20261030_slice_ab`（14 点，53 文件）。
- 脚本改动：`mc_bench_point.sh` 仅加 2 行 `MC_SLICE_SIZE` 透传；md5
  `cf04e767…` → `8a0c5c78…`；不设时行为不变。外部已按 2 行 diff 归档到
  `D:\gds性能\patches\E3-W\`（无 C++ 变更）。

---

# E3-X（已关闭 · 2026-10-30）· UB 固定 wait ~67 µs：poll spin/sleep A/B（4MB b 1/1）

> 已收，回执见下方。**结论：固定 ~67 µs 就是“睡 50 µs 再查”的完成发现延迟。**
> sleep=0 后 wait 153→90.3、total 178.3→94.7；4M+0 的 b 1/1 到 483.8 µs。

## 前置（代码事实）

1) `transfer_task.cpp` 的 `TransferEngineOperationState::wait_for_completion()`：
   UB 时 `spin_time = MC_STORE_UB_POLL_SPIN_US`（默认 50），
   `sleep_time = MC_STORE_UB_POLL_SLEEP_US`（默认 50）；自旋到期后每轮 sleep 再查。
2) 两个 env 都走 `ReadUbPollDelayUs`：只认十进制整数，0–1,000,000；非法回退默认。
   `MC_STORE_UB_POLL_SLEEP_US=0` 会恢复 busy polling（日志会打
   `UB completion polling: spin_us=... sleep_us=...`）。
3) 先确认当前日志确实是 `spin_us=50 sleep_us=50`；不是就停手回报。
4) 不改 C++、不改 node2/阵列/交付表；只在 node4 加 env。

## 测试（4MB b 1/1）

- 协议：pin=1（`MC_UB_STAGING_POOL_PIN=1`）、2G 池、gseg=0、绑 N0、batch=1、60 s。
- 2×2 矩阵（隔离 polling 与切片）：
  1. `slice=65536` + 默认 poll（基线，应约 180.3）；
  2. `slice=65536` + `MC_STORE_UB_POLL_SLEEP_US=0`（busy poll）；
  3. `slice=4194304` + 默认 poll（E3-W 最佳，应约 157.0）；
  4. `slice=4194304` + `MC_STORE_UB_POLL_SLEEP_US=0`（叠加）。
- 每格 2 轮 60 s；基线/最佳补到 3 轮；14 点目标 failed=0。
- 记录：transfer_data `submit/wait/total`、worker `post_to_cq`、node4 load/CPU、
  `failed`；确认日志里的 `spin_us/sleep_us`。
- 不改 C++；如 busy poll 导致 CPU 争抢/worker 饿死，记录并停，不硬跑 16/16。

## 判据

- `sleep=0` 让 wait 从 153 降到 ~90（±15）→ **固定 ~67 来自完成发现延迟**；
  4M+0 的 total 应 ≈90–110；b 1/1 有望到 ~480–500。
- wait 基本不变 → 固定开销不在 store 轮询，转查 completion callback / worker→client 通知 / 目标端。
- 任何格 `failed>0` 或 CPU 明显异常：停，记录；不把失败点写成交付。
- 不改交付表；R22。

## 产物

- R22_20261030_ub_poll_ab：meta、summary.csv、worker/transfer 过滤日志、env、命令、load；
- 顺带回传 `mc_bench_point.sh` 2 行 diff 的 `base64 -w0` + sha256（E3-W 只给了明文 diff/md5）；
- 不改交付表。

## 回执（2026-10-30，已收）

**前置核实**：现场日志逐点确认 `UB completion polling: spin_us=50 sleep_us=50`；
sleep=0 时 `sleep_us=0`。12 点严格交替。

**正式矩阵（4 格 × 3 轮 × 60 s，12 点 failed=0）**

| 格 | slice | sleep | 片数 | GiB/s | avg µs | submit | wait | total | post→CQ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| g1 | 65536 | 默认 50/50 | 63.85 | 6.84 | 570.67 | 25.0 | 153.0 | 178.33 | 53.05 |
| g2 | 65536 | 0 | 63.85 | 7.51 | 520.26 | 28.33 | 101.0 | 129.33 | 53.99 |
| g3 | 4194304 | 默认 50/50 | 1.0 | 7.13 | 547.65 | 3.67 | 153.0 | 157.67 | 86.26 |
| g4 | 4194304 | 0 | 1.0 | 8.07 | **483.77** | 3.67 | **90.33** | **94.67** | 86.32 |

- **决定性证据**：g4 `wait−post→CQ = 90.33−86.32 = 4.0`；g3 是 66.7。
  那 67 µs 几乎全部来自“睡 50 µs 再查”的检测周期；去掉后只剩 4 µs。
- 三条判据全中：sleep=0 wait 75–105（g4 90.3 / g2 101）；4M+0 total 90–110（94.7）；
  b 1/1 480–500（483.77）。
- 边界：g4 的 wait 90.33 里 post→CQ 86.32 已是 URMA 单请求往返（≈E3-U 86.4）；
  UB 段剩余可优化空间只有 **~4 µs**，再往下只能动链路/目标端。
- 风险/偏差：busy poll 有旁证（64K 档 queue_wait 18.22→20.77，+2.5 µs；submit +3.3）；
  post_to_cq 偶发尖峰（g2_r2 519、g4_r3 470）；g2 wait=101 因 64K 逐片排队；
  **16/16 未测，busy poll 每线程一个自旋核，CPU 代价未知**；未测中间 sleep/ spin/ d/ 长稳态。
- 脚本 md5 更正：E3-W 前基线是 `46adf724…`（不是 `cf04e767…`，那是 R19）；E3-W 后
  `8a0c5c78…`，E3-X 后 `b2a43fb4…`。**按 2026-10-30 新规则，脚本 diff 不回传、不归档；
  只在 meta 记 env/命令/md5。**
- C++ 零改动；node2 零改动；R22。

---

# E3-Y（已关闭 · 2026-10-30）· busy poll 的 16/16 代价（4MB b）

> 已收，回执见下方。**结论：16/16 下 busy poll 零收益、CPU 2.6×；4M slice 才是关键
> （11.41 vs 64K 9.86，+15.7%）。16/16 保留默认 sleep=50；1/1 才用 sleep=0。**

> 目标：E3-X 在 1/1 上把 UB 180→94.7，但 busy poll 每线程占一个自旋核；
> 16/16 有 16 个客户端线程，CPU 争抢/worker 饿死完全未知。本轮决定
> **能不能上生产设置**：`sleep=0`、`sleep=10`、默认 `50` 三档，固定最佳 slice=4M。
> 只调 env，不改代码、不改交付表；R23。

## 测试（4MB b 16/16）

- 协议：pin=1、2G 池、gseg=0、绑 N0、batch=16、16 线程、60 s；
  `MC_SLICE_SIZE=4194304`（E3-W/E3-X 最佳），`MC_STORE_UB_POLL_SLEEP_US` 三档：
  `0`、`10`、默认 `50`（`spin` 保持默认 50）。
- 加一个对照：`slice=65536` + `sleep=0`，确认 4M 在 16/16 是否仍优。
- 4 格 × 3 轮 = 12 点，failed=0；同窗口。
- 记录：吞吐、avg/P99、`failed`、worker `queue_wait/post_to_cq`、**进程/线程 CPU**
  （`pidstat -t -p <pid> 1` 或 `top -H -b -n1`，至少每格一条）、node4 load。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- `sleep=0` 的 16/16 吞吐 ≥ 默认（或下降 <3%），且 CPU 有余量 → busy poll 可作候选；
- 若 16/16 明显下降/worker 饿死 → 用 `sleep=10` 找折中，或保留默认；
- `4M` 在 16/16 不劣于 `64K` 才保留；否则 1/1 的收益不能外推到 16/16；
- 任何格 `failed>0`/CPU 打满：停，记录，不把失败点写交付。
- 不改交付表；R23。

## 产物

- R23_20261030_busy_poll_16：meta、summary.csv、worker 过滤日志、CPU 采样、env、命令、load；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-30，已收）

**结果（b 16/16，4 MiB，pin=1，diag=1，3×60 s，12 点 failed=0）**

| 格 | slice | sleep | GiB/s | avg µs | P99 µs | 进程 CPU 核 | >0.5 核线程 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| g1 | 4M | 0 | 11.41 | 87547 | 88080 | 16.49–16.98 | 17/19 |
| g2 | 4M | 10 | 11.40 | 87589 | 88080 | 7.92–8.55 | 4–7/19 |
| g3 | 4M | 50（默认） | 11.41 | 87554 | 88080 | 6.16–6.73 | 1/19 |
| g4 | 64K | 0 | 9.86 | 101378 | 103809 | 18.16–18.26 | 18/19 |

- **sleep 0/10/50 在 4M 下吞吐都是 11.41/11.40/11.41，wait_blocked 87143/87211/87164
  （极差 68 µs）→ 16/16 旋钮失效**；busy poll 只把 CPU 从 6.5 核抬到 16.8 核（2.6×）。
- **4M vs 64K（都 sleep=0）：11.41 vs 9.86 = +15.7%**；64K 的 submit 61→753 µs、
  wait +13.4 ms（1024 片/批的开销）。
- 11.41 GiB/s = E3-U/R18 PCIe pinned 上限 11.84 的 **96.4%**，系统已是**吞吐饱和**；
  Little's law：1 GiB ÷ 11.7 GiB/s = 87.3 ms ≈ 实测 avg 87.5 / wait_blocked 87.1。
  单批 87 ms 全是排队，sleep 的 ~50 µs 只占 0.06%。
- **结论**：16/16 保留默认 sleep=50；busy poll 只在 1/1 低并发有价值；
  4M slice 在 1/1 和 16/16 都有效，是主收益。若必须一套配置通吃，sleep=10 是折中
  （16/16 多 1.7 核、零吞吐收益；1/1 略有收益）。
- 偏差/风险：CPU 是 `top -H` 8 s/20 s 两帧，不是 pidstat 平均；slices/post_to_cq 在
  16/16 的分母是“一批 16 keys”，g1 的 73440 µs 不可与 1/1 的 86 µs 比；
  load 43.7→55.5，部分来自共用机他人；9/9、32/32、c/d 未测。
- C++/脚本零改动；node2 零改动；R23。

---

# E3-Z（已关闭 · 2026-10-30）· 4M slice 外推：8MB b 16/16 + d 16/16（default sleep）

> 已收，回执见下方。**8MB b：4M 11.43 vs 64K 10.20（+12.1%），可外推；
> d 4M/64K 都是 11.39，但这不是收益——NoF 路径不消费 `MC_SLICE_SIZE`，两臂是同一配置。**

> 目标：E3-Y 已证明 4M slice 在 4MB b 16/16 把 9.99→11.41（96.4% PCIe）；
> 本轮验证能否外推到 **8MB b 16/16** 和 **d 16/16**，决定 4M 是否作为生产候选。
> 默认 `MC_STORE_UB_POLL_SLEEP_US=50`（16/16 不用 busy poll）；只调 env，不改代码；
> 不改交付表；R24。

## 测试

1) **8MB b 16/16**：`--local_buffer_size=4G`、`--global_segment_size=0`、
   `MC_UB_STAGING_POOL_SIZE=4294967296`、pin=1、batch=16、16 线程、60 s；
   `MC_SLICE_SIZE` 4M vs 64K，2 臂 ×3 轮。
2) **d 16/16（4MB）**：pin=1、batch=16、16 线程、60 s；`MC_SLICE_SIZE` 4M vs 64K，
   2 臂 ×3 轮；其余同 4MB 协议。
- 共 4 臂 ×3 轮 = 12 点，failed=0；同窗口。
- 记录：吞吐、avg/P99、failed、worker `queue_wait/post_to_cq`、CPU、load；
  8MB 额外记录 `registered CPU staging pool` 与 `MC_UB_STAGING_POOL_SIZE` 生效值。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- 8MB b：4M 比 64K ≥ +5% → 4M 可作 8MB 生产候选；若持平/变差，8MB 保留 64K；
- d：4M 不劣于 64K（<3% 回退）才保留；若 d 变差，按场景分别配置；
- 任何格 `failed>0`：停，记录，不写交付；
- 不改交付表；R24。

## 产物

- R24_20261030_slice_8mb_d：meta、summary.csv、worker 过滤日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-30，已收）

**结果（16 线程 × batch 16，pin=1，默认 poll 50/50，3×60 s，12 点 failed=0）**

| 臂 | 场景 | slice | GiB/s | avg µs | P99 µs | CPU 核 |
| --- | --- | --- | --- | --- | --- | --- |
| z1 | b 8MB | 4M | **11.43** | 174808 | 176161 | 8.66–9.37 |
| z2 | b 8MB | 64K | 10.20 | 195745 | 199229 | 7.16–7.91 |
| z3 | d 4MB | 4M | 11.39 | 87793 | 111149 | 2.33–2.41 |
| z4 | d 4MB | 64K | 11.39 | 87787 | 113246 | 2.27–2.37 |

- **8MB b：4M 11.43 vs 64K 10.20 = +12.1%**；11.43 = PCIe pinned 11.84 的 **96.5%**；
  64K 的 submit 73→1452 µs（2048 片/批）。8MB staging pool 两臂均 4 GiB 生效。
- **d 的 +0.0% 不是收益**：z3/z4 的 `ub_worker_breakdown` 全程 `iterations=2,
  dequeued_slices=0, post_calls=0`，且没有 `registered CPU staging pool` 行；
  `MC_SLICE_SIZE` 的消费点 `ub_transport.cpp:693` 不在 d 的 NoF 读路径上。
  两臂实际是同一配置，不能当作“4M 对 d 无害/有益”的 A/B。
- 8MB avg 174.8 ms ≈ 4MB 87.5 ms 的 2 倍，在飞 2 GiB ÷ 11.7 = 179 ms，Little 定律吻合。
- 偏差/风险：z1 的 `ub_worker_breakdown` 只有 68 行（应 ~120），
  `slices_per_req/queue_wait/post_to_cq` 不可用于精确拆解；z1 只有吞吐/时延/CPU 可靠。
  d 的 `batch_get_data_breakdown total` 1.8 ms 与实测 avg 87.8 ms 差 49 倍，不能当端到端拆解。
  CPU 仍是 top -H 两帧；load 41.5→48.95。
- `MC_SLICE_SIZE=4194304` 正好等于 `SPDK_URMA_MAX_IO_SIZE=4194304`，8MB 切 2 片，
  `post_failed_slices=0` 说明未越界；但单 WR 已到上限，重试粒度/兼容性风险最大。
- C++/脚本零改动；node2 零改动；R24。

---

# E3-AA（已关闭 · 2026-10-30）· 中间 slice 选择：16/16 的 256K/512K/1M vs 4M（4MB/8MB b）

> 已收，回执见下方。**4MB 稳健选 1M（96.1%，3/3 轮过阈）；8MB 512K 已 99.0%；
> 1M 可作 4MB/8MB 共用配置；256K/512K 在 4MB 是边缘通过，不建议生产默认。**

> 目标：1/1 六档显示 **256K→4M 只差 6 µs**，4M 在 1/1 不是必需；
> 16/16 只测了 64K 与 4M 两端，中间档未测。本轮找 **能拿到 4M 大部分收益、
> 但单 WR 更小/重试粒度更安全** 的 slice，避免直接上 4M（正好等于 URMA 上限）。
> 默认 sleep=50、pin=1；只调 env，不改代码；不改交付表；R25。

## 测试

1) **4MB b 16/16**：slice `65536 / 262144 / 524288 / 1048576 / 4194304`
   5 臂 ×3×60 s；默认 sleep=50；pin=1；batch=16、16 线程。
2) **8MB b 16/16**：slice `524288 / 1048576 / 4194304` 3 臂 ×3×60 s；
   64K 基线用 E3-Z 的 10.20，不重复跑；`--local_buffer_size=4G`、
   `MC_UB_STAGING_POOL_SIZE=4294967296`。
- 共 24 点，failed=0；同窗口；四/八臂按轮交错执行。
- 记录：吞吐、avg/P99、failed、worker `posted_slices/请求`、CPU、load；
  确认每档片数真的变（4MB：64/16/8/4/1 per key；8MB：16/8/4/2 per key）。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- 以 4M 为上限：选 **吞吐 ≥ 4M 的 95%** 的最小 slice
  （4MB ≥10.85 GiB/s；8MB ≥10.86 GiB/s），且 failed=0；
- 若 1M ≥97% 且 512K 也 ≥95% → 优先 512K/1M（重试粒度更小）；
- 若只有 4M 能过 95% → 保留 4M，并在报告注明“单 WR 到 URMA 上限”的风险；
- 任何档 failed>0 / post 超时：停，记录，不写交付。
- 不改交付表；R25。

## 产物

- R25_20261030_slice_mid：meta、summary.csv、worker 过滤日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-30，已收）

**结果（8 档 ×3×60 s，24 点 failed=0，默认 sleep=50）**

| 格 | value | slice | GiB/s 中位 | 占 4M | 逐轮 | CPU 核 |
| --- | --- | --- | --- | --- | --- | --- |
| a1 | 4MB | 64K | 9.96 | 87.3% | 9.96/9.99/9.92 | 7.91 |
| a2 | 4MB | 256K | 10.86 | 95.2% | 10.86/10.87/10.75 | 8.73 |
| a3 | 4MB | 512K | 10.90 | 95.5% | 10.84/10.90/10.92 | 8.91 |
| a4 | 4MB | **1M** | **10.97** | **96.1%** | **11.08/10.97/10.95** | 9.18 |
| a5 | 4MB | 4M | 11.41 | 100% | 11.41/11.39/11.42 | 6.37 |
| a6 | 8MB | **512K** | **11.32** | **99.0%** | **11.32/11.31/11.40** | 8.66 |
| a7 | 8MB | 1M | 11.35 | 99.3% | 11.35/11.35/11.36 | 9.18 |
| a8 | 8MB | 4M | 11.43 | 100% | 11.42/11.44/11.43 | 9.00 |

- **4MB 稳健答案 = 1M**：256K r3=10.75、512K r1=10.84 都低于阈值 10.85（3 轮里只 2 轮过），
  1M 三轮 11.08/10.97/10.95 全部过阈；8MB 512K/1M 都 3/3 过阈。
- **8MB 曲线平坦**：512K 已 99.0%；4MB 曲线陡（64K 87.3%→256K 95.2%→4M 100%）。
  原因：同档 slice 下 8MB 每批片数翻倍，worker 并行度天然充足；最小可用 slice
  随 value_size/每批片数变化，不是固定常数。
- **共用配置建议 1M**：4MB 96.1% + 8MB 99.3%，3/3 轮稳定；单 WR 从 4M（URMA 上限）
  降到 1/4，代价 4MB −3.9%、8MB −0.7%。**256K/512K 在 4MB 是边缘通过，不建议生产默认。**
- CPU 反向发现：4MB 64K 7.91 核 → 1M 9.18 核，但 4M 反而最低 6.37 核
  （大 slice 摊薄 post，worker 轮转少）；1M 比 4M 多 ~2.8 核。
- 偏差：`ub_worker_breakdown` 区间快照丢行（4MB/1M 81/122 行、8MB/4M 29/122 行），
  `per_key` 读数只能定性；吞吐/时延/CPU 不受影响（queries×16×value÷60s 交叉吻合）。
  queue_wait 是批量口径，跨档不可比；CPU 是 top -H 两帧；load 41–49。
- 4M 单 WR 正好等于 `SPDK_URMA_MAX_IO_SIZE=4194304`，无余量；1M 降到 1/4。
- C++/脚本零改动；node2 零改动；R25。

---

# E3-AB（已关闭 · 2026-10-30）· 1M slice 的 1/1 验证与单配置折中（4MB/8MB b）

> 已收，回执见下方。**1M 在 1/1 完全等价 4M（483.29 vs 483.77）；sleep 0/10/20/50
> 的 1/1 avg 是 483/511/522/551，斜率≈1；没有一档同时满足“1/1≤490 + sleep10/20≤20”
> （sleep10 差 1 µs）。16/16 的 1M CPU 仍缺，下一步 E3-AC。**

> 目标：E3-AA 已选 1M 作 16/16 共用配置；本轮补 1/1，决定最终生产配置：
> **1M + sleep=0/10/20/50** 哪个能同时兼顾 1/1 延迟和 16/16 CPU。
> 只调 env，不改代码、不改交付表；R26。

## 测试

1) **4MB b 1/1**：slice=1M，`MC_STORE_UB_POLL_SLEEP_US` = 0 / 10 / 20 / 50（默认）；
   4 臂 ×3×60 s；pin=1、2G 池、gseg=0、绑 N0、batch=1。
2) **8MB b 1/1**：slice=1M vs 4M，sleep=0（两臂）；2 臂 ×3×60 s；
   `--local_buffer_size=4G`、`MC_UB_STAGING_POOL_SIZE=4294967296`。
- 共 18 点，failed=0；同窗口。
- 记录：transfer_data submit/wait/total、b 1/1 avg/P99、CPU、failed、node4 load。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- 4MB 1/1：1M+sleep0 的 total 应 ≤~490 µs（4M+sleep0 是 483.8）；
  若 sleep10/20 与 sleep0 差 ≤20 µs，且 16/16 CPU 可接受，则单配置候选；
- 8MB 1/1：1M 与 4M 差 ≤3% 则选 1M；否则按 value 分别配置；
- 任何格 failed>0：停，记录，不写交付；
- 不改交付表；R26。

## 产物

- R26_20261030_1m_1to1：meta、summary.csv、worker/transfer 日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-31，已收；R26 目录按执行日 20261031）

**结果（1 线程 × batch 1，pin=1，6 臂×3×60 s，18 点 failed=0）**

| 格 | value | slice | sleep | GiB/s | avg µs | submit | wait | total | CPU 核 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| b1 | 4MB | 1M | 0 | 8.08 | **483.29** | 6.0 | 91 | 98.0 | 1.22 |
| b2 | 4MB | 1M | 10 | 7.64 | 510.95 | 6 | 113 | 119 | 1.08 |
| b3 | 4MB | 1M | 20 | 7.49 | 521.61 | 6 | 123 | 129 | 1.05 |
| b4 | 4MB | 1M | 50（默认） | 7.09 | 550.81 | 6.0 | 153 | 159.0 | 0.99 |
| b5 | 8MB | 1M | 0 | 8.68 | 900.35 | 7.0 | 176 | 184 | 1.18 |
| b6 | 8MB | 4M | 0 | 8.70 | 897.47 | 5.0 | 174 | 180.0 | 1.17 |

- **1M = 4M**：4MB avg 483.29 vs 483.77（−0.1%）；8MB −0.2%。多出的 3.3 µs total
  全在 submit（1M 在 transport 内循环 4 次，4M 1 次），wait 基本不动。
- **wait 对 sleep 是斜率≈1 的直线**：91/113/123/153（sleep=0/10/20/50），wait≈103+sleep；
  sleep=0 的 91 里 86.3 是 URMA 往返，残差 ~5 µs。**wait 只由 sleep 决定，与 slice 无关。**
- 1/1 avg：sleep0 483.29、sleep10 510.95（+27.7，+5.7%）、sleep20 521.61、默认 550.81（+67.5，+14.0%）；
  CPU 1.22/1.08/1.05/0.99。
- 稳定性：b1 三轮 avg 极差 2.3 µs、total 极差 0；b2–b4 ≤1 µs。
- **判据口径修正**：任务书把“1M+sleep0 total ≤490”写成了 total，实际 483.8 是 avg；
  本轮 avg 483.29、transport total 98.0，两者都 ≤490，判据 PASS；后续引用区分 avg（bench 端到端）
  与 total（transport 三段）。
- **最大缺口**：16/16 的 CPU 数字（6.5/8.2/16.8）来自 E3-Y 的 **4M slice**，不是 1M；
  E3-AA 有 1M 默认 sleep 的 16/16 CPU 9.18，但 **1M 下 sleep0/10 的 16/16 CPU 未测**。
- `transfer_data_breakdown.slices` 是 store 层 slice 描述符（每 key 1 条），不能判断片数；
  片数看 `ub_worker_breakdown.posted_slices`（`ub_transport.cpp:693`）。已更正。
- C++/脚本零改动；node2 零改动；R26。

---

# E3-AC（已关闭 · 2026-10-31）· 1M slice 的 16/16 CPU/吞吐：sleep 0/10/50（4MB/8MB b）

> 已收，回执见下方。**16/16 三档吞吐不变；sleep0 +9.71 核、sleep10 +1.82 核 →
> 16/16 保留默认 50。最终方案 A：1/1 sleep=0（483.3），16/16 sleep=50（1M 10.97/11.37）。**

> 目标：E3-AB 证明 1M 在 1/1 等价 4M，但 16/16 的 1M CPU 只测了默认 sleep；
> 本轮补 1M 下 sleep 0/10/50 的 16/16 吞吐/CPU，定最终配置：
> **按并发分别配置（1/1 sleep0，16/16 默认 50）**，还是**单配置 sleep10**。
> 只调 env，不改代码、不改交付表；R27。

## 测试

1) **4MB b 16/16**：slice=1M，sleep 0/10/50；3 臂×3×60 s；pin=1、batch=16、16 线程。
2) **8MB b 16/16**：slice=1M，sleep 10/50；2 臂×3×60 s；
   `--local_buffer_size=4G`、`MC_UB_STAGING_POOL_SIZE=4294967296`。
- 共 5 臂×3 = 15 点，failed=0；同窗口。
- 记录：吞吐、avg/P99、进程/线程 CPU、worker queue/post_to_cq、load；确认 1M 片数。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- 1M 16/16 下 sleep0/10 若吞吐不升、CPU 明显高于默认 50 → **16/16 保留默认 50**；
- 若 16/16 sleep0/10 吞吐不降且 CPU 可接受，可作为单配置候选；
- 8MB 1M 与 4MB 趋势一致；任何格 failed>0 停；
- 最终建议二选一：
  A. 按并发：1/1 sleep=0（483.3），16/16 sleep=50（1M 10.97/11.35，CPU 9.18）；
  B. 单配置：1M + sleep=10（1/1 510.95，+5.7%；16/16 待本轮 CPU）。
- 不改交付表；R27。

## 产物

- R27_20261031_1m_16：meta、summary.csv、worker 过滤日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-31，已收）

**结果（5 臂×3×60 s，15 点 failed=0）**

| 格 | value | sleep | GiB/s | avg µs | P99 µs | CPU 核 | >0.5 核线程 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| c1 | 4MB | 0 | 10.97 | 91061 | 92275 | 18.89 | 19/19 |
| c2 | 4MB | 10 | 10.94 | 91351 | 92275 | 11.00 | 7/19 |
| c3 | 4MB | 50（默认） | 10.97 | 91058 | 92275 | 9.18 | 4/19 |
| c4 | 8MB | 10 | 11.36 | 175846 | 178258 | 11.03 | 6/19 |
| c5 | 8MB | 50（默认） | 11.37 | 175605 | 178258 | 9.10 | 4/19 |

- **16/16 三档吞吐都在噪声带内（10.94–10.97 / 11.36–11.37），sleep 换不来吞吐**；
  sleep0 +9.71 核（2.06×）、sleep10 +1.82 核；8MB sleep10 +1.93 核。**16/16 保留默认 50。**
- 与 E3-AA 逐位重现：4MB 1M 默认 10.97/9.18 核完全一致；8MB 11.37 vs 11.35（+0.2%）、9.10 vs 9.18（−0.9%）。
- **最终方案 A**：1/1 sleep=0（483.3 µs），16/16 sleep=50（1M：4MB 10.97、8MB 11.37，CPU 9.18）。
  **方案 B（单配置 1M+sleep10）不成立**：16/16 零吞吐收益却多 1.82 核，1/1 又慢 5.7%。
- CPU 口径：4MB 的 avg_cores 因 16/16 线程过多（291–388）采样失败，用 8 s/20 s `top -H` 峰值；
  8MB 用 proc 均值（峰值互校差 <2%）。
- posted_slices 仍丢行（31%/49%/21%/26%/40%），per_key 只能定性；结论不依赖片数。
- E3-Y 的 16/16 CPU 是 4M slice；**本轮 1M 数据才是 1M 的权威 CPU 账**。
- C++/脚本零改动；node2 零改动；R27。

---

# E3-AD（已关闭 · 2026-10-31）· 方案 A 全场景复测（4MB a/b/c/d × 1/1/16/16）

> 已收，回执见下方。**方案 A 四场景全部通过、零回退：a 1/1 +8.5%、b +35.5%、
> c +0.4%、d +11.5%；16/16 a/c 持平、b 10.97、d 11.39。**
> 但 1/16 与 16/1 两个交付点未测，下一步 E3-AE 补齐。

> 目标：方案 A 目前只验证了 b；本轮把 **1M slice + 1/1 sleep=0 + 16/16 sleep=50 + pin**
> 套到 a/b/c/d 四个场景，确认没有把直写（a/c）和 NoF（d）带偏，再决定生产配置。
> 只调 env，不改代码、不改交付表；R28。

## 测试

- 4MB，四个场景 a/b/c/d × 1/1、16/16；每点 3×60 s，共 24 点。
- 统一：`MC_SLICE_SIZE=1048576`、`MC_UB_STAGING_POOL_PIN=1`；
  1/1 加 `MC_STORE_UB_POLL_SLEEP_US=0`；16/16 默认 50。
- 1/1 batch=1；16/16 16 线程 × batch16；同窗口。
- 记录：avg/P99、吞吐、failed、CPU、load；与 R08 交付逐点对比。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- b：1/1 ≤490 µs、16/16 ≥10.8 GiB/s（预期 483 / 10.97）；
- a/c：1/1 与 16/16 相对 R08 不回退 >3%；若回退，说明 slice/poll 对直写路径有副作用，
  按场景分别配置（a/c 保留原参数）；
- d：slice 对 NoF 无效，但 1/1 的 UB 段可能受 sleep0 影响；相对 R08 不回退 >3%；
- 任何格 failed>0：停，记录，不写交付；
- 不改交付表；R28。

## 产物

- R28_20261031_planA_all：meta、summary.csv、worker/transfer 日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-31，已收）

**结果（8 格×3×60 s，24 点 failed=0；方案 A：1M、pin、1/1 sleep0、16/16 sleep50）**

| 格 | case | T/B | sleep | GiB/s | avg µs | P99 µs | CPU 核 | R08 | Δ吞吐 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| p1 | a | 1/1 | 0 | 10.59 | 368.77 | 380.93 | 1.90 | 9.76 | +8.5% |
| p2 | b | 1/1 | 0 | 8.06 | 484.26 | 503.81 | 1.25 | 5.95 | +35.5% |
| p3 | c | 1/1 | 0 | 8.35 | 467.93 | 647.17 | 1.02 | 8.32 | +0.4% |
| p4 | d | 1/1 | 0 | 5.63 | 693.97 | 983.04 | 1.03 | 5.05 | +11.5% |
| p5 | a | 16/16 | 50 | 12.69 | 78757 | 79692 | 8.96 | 12.69 | 0.0% |
| p6 | b | 16/16 | 50 | 10.97 | 91093 | 92275 | 8.97 | 8.65 | +26.8% |
| p7 | c | 16/16 | 50 | 12.68 | 78838 | 81789 | 3.98 | 12.68 | 0.0% |
| p8 | d | 16/16 | 50 | 11.39 | 87793 | 114295 | 2.19 | 9.40 | +21.2% |

- **方案 A 四场景零回退**：a 1/1 400→369（sleep0）；b 656→484（sleep0）；c 468 基本不动；
  d 773→694（sleep0）；16/16 a/c 保持 12.69/12.68 上限，b 10.97，d 11.39。
- **c 1/1 不动是关键旁证**：c 瓶颈在 NoF 读盘，UB 等待不占主导，sleep0 对它无效；
  反证 a/b/d 的收益确实来自省掉 UB 完成等待，不是噪声。
- 路径计数器：c 全 `cuda_io`、d 全 `host_io`（16/16 staged_bytes 734.7 GB）；a/b 两计数器为 0
  （UB 路径无埋点，与 R08 同一缺口）→ 四场景没有被 env 串味。
- 交叉一致性：b 1/1 484.26 vs R26 483.29（+0.97 µs）；b 16/16 10.97/8.97 核 vs R27 10.97/9.18；
  d 16/16 11.39 与 R24 逐位一致；a 16/16 12.69 与 R08 逐位一致。
- **过程事故**：11:34 与 11:44 两个 `e3ad_matrix.sh` 并发，同压同一 target，11:34–11:50
  数据作废；按 PID 精确 kill 后 `setsid nohup` 单实例重启，本报告 24 点全部来自 11:51:56
  之后的单实例窗口；a/c 16/16 三轮回读逐位相同，并与 R24/R26/R27 交叉吻合。
- `ub_worker_breakdown` 丢行只在 16/16（78/65/122/126 行），1/1 四臂 122/122/122/122 零丢行。
- p8 的 +21.2% 相对 R08，但 R24 已是 11.39，本轮只是复现，不计作方案 A 的功劳。
- C++/脚本零改动；node2 零改动；R28。

---

# E3-AE（已关闭 · 2026-10-31）· 方案 A 补齐并发：4MB a/b/c/d × 1/16 + 16/1（sleep 0/50）

> 已收，回执见下方。**1/16、16/1 都已吞吐饱和，sleep0 零收益、只烧 CPU；
> 最终只有 1/1 用 sleep=0，其余三个并发点都用默认 50。**

> 目标：E3-AD 只测了 1/1 和 16/16；交付表还有 **1/16（1 线程×batch16）** 和
> **16/1（16 线程×batch1）** 两个并发点，sleep 该用 0 还是 50 未知。
> 本轮补齐，并确认 a/c/d 不回退；只调 env，不改代码、不改交付表；R29。

## 测试

- 4MB，a/b/c/d × 1/16、16/1；每场景每并发测 `sleep=0` 与 `sleep=50` 两档，共 4×2×2=16 臂。
- 统一 `MC_SLICE_SIZE=1048576`、`MC_UB_STAGING_POOL_PIN=1`；每臂 2×60 s，基线/最佳 3 轮。
- 记录：avg/P99、吞吐、failed、CPU、load；与 R08 同点对比。
- 不改 C++；node2/阵列/交付表不动。

## 判据

- 对每个并发点选 sleep：吞吐/延迟更优且 CPU 可接受者；
- 若 1/16、16/1 也像 16/16 一样吞吐饱和 → 保留 sleep=50；若像 1/1 延迟受限 → sleep=0；
- a/c/d 任一相对 R08 回退 >3%：记录并按场景调整；
- 任何格 failed>0：停，不写交付；
- 不改交付表；R29。

## 产物

- R29_20261031_planA_mid：meta、summary.csv、worker/transfer 日志、CPU/load、env、命令；
- 不改交付表；脚本 diff 不用回传（只记 env/md5）。

## 回执（2026-10-31，已收）

**结果（4MB，1/16 与 16/1，sleep 0/50；41 点 failed=0）**

| 并发 | case | sleep0 GiB/s / avg | sleep50 GiB/s / avg | Δ吞吐 | CPU 0→50 |
| --- | --- | --- | --- | --- | --- |
| 1/16 | a | 12.37 / 5050 | 12.22 / 5115 | +1.2% | 1.96→1.33 |
| 1/16 | b | 10.91 / 5724 | 10.91 / 5727 | 0.0% | 1.96→1.41 |
| 1/16 | c | 11.63 / 5373 | 11.65 / 5362 | −0.2% | 3.83→3.83 |
| 1/16 | d | 8.61 / 7262 | 8.61 / 7257 | 0.0% | 1.66→1.68 |
| 16/1 | a | 12.69 / 4925 | 12.69 / 4925 | 0.0% | 16.62→6.50 |
| 16/1 | b | 11.41 / 5473 | 11.40 / 5480 | +0.1% | 1.83→1.75 |
| 16/1 | c | 12.66 / 4935 | 12.66 / 4936 | 0.0% | 4.21→4.21 |
| 16/1 | d | 11.39 / 5487 | 11.39 / 5486 | 0.0% | 2.41→2.41 |

- **1/16、16/1 都已吞吐饱和**：sleep0 零收益，16/1 的 a 多烧 +10.12 核（2.56×）；
  1/16 的 a +1.2% 是噪声（R08 基线本身 +0.7%）。**都选 sleep=50。**
- **最终方案 A**：1/1 1M+sleep0；1/16、16/1、16/16 都是 1M+sleep50；pin=1。
- 机理统一：sleep0 的收益 ∝ UB 等待占比 × 是否独占关键路径。
  1/1 每请求一次、独占 → −67 µs；1/16 每批一次摊到 16 key ≈3 µs/key；
  16/1 每请求一次但 16 线程填空档；16/16 双重摊薄 → 后三者 0.0%。
- 丢行只在 16/16；1/1、1/16、16/1 的 ub_worker 都是 122 行零丢行。
- 稳定性：16 臂两轮极差 ≤0.02 GiB/s、avg ≤10 µs；16/1 四场景两轮逐位相同。
- b 的 +29.7%（1/16）/ +25.1%（16/1）记在 1M slice；a 已贴 PCIe 直写上限；
  c 由 NoF 盘限；d 的 +15.4%/+25.9% 是 R08→R24 的历史提升，本轮只是复现。
- C++/脚本零改动；node2 零改动；R29。

---

# E3-AF（待执行）· 第三次测试：4MB 全表（干净）+ 1/1 打点拆解（独立诊断批）

> 目标：把方案 A 的 4MB 结果正式跑成“第三次测试”的 16 格表；同时单独跑一批 1/1 打点，
> 做与第二次测试同样的时延拆解。**干净表和打点批分开，不混数。**
> 不改交付表；第三次测试表用新副本；R30。

## 一、干净正式表（16 格）

- 4MB，a/b/c/d × 1/1、1/16、16/1、16/16；共 16 格。
- 方案 A：`MC_SLICE_SIZE=1048576`、`MC_UB_STAGING_POOL_PIN=1`；
  **只有 1/1 加 `MC_STORE_UB_POLL_SLEEP_US=0`，其余三个并发点默认 50。**
- 采样同第二次：1/1 与 16/1 各 3 轮，1/16 与 16/16 各 2 轮（共 40 个样本，60 s/点）。
- **干净跑：不开 `MC_HIFREQ_LOG_SAMPLE_RATE`、不开 `MC_UB_WORKER_DIAG`。**
- **xlsx 输出**：复制权威表为
  `测试相关资料/第三次测试结果/benchmark_Mooncake_R30_20261031_第三次测试.xlsx`
  （用户已新建该目录；如无则创建）；
  **只填 4MB 的 a/b/c/d × 4 并发共 16 格**，8MB sheet 保持原样/不参与第三次测试；
  **只写干净正式表的 avg/P50/P90/P99/P99.9/P99.99/Pmax/吞吐，不写打点批的数**；
  原 `我们测试场景的测试结果benchmark.xlsx` 不动。
- 产物：R30 干净表批次 + 上述 xlsx 副本 + meta/CSV/日志。

## 二、独立打点批（只做 1/1）

- 4MB，a/b/c/d × 1/1，batch=1；配置同正式表（1M、pin、sleep=0）。
- 开 `MC_HIFREQ_LOG_SAMPLE_RATE=1`（每请求 breakdown）和 `MC_UB_WORKER_DIAG=1`；
  3×60 s，每场景 1 点，共 4 个诊断点。
- **不开打点的干净 1/1 仍用正式表的数**；打点批只用于拆解，不写进正式表。
- 先做 10 s A/B：干净 vs 打点，若 total 偏移 >3%，把 sample 降到 0.1/0.5 并注明；
  1/1 的 diag 开销之前是 −0.40%/+0.38%，预计合格。
- 拆解输出：query / read / transfer / copy / residual / total，按 a/b/c/d 分场景；
  与第二次测试同样的“理论 / 可达 / 实测”表。
- 注意：batch>1 不走 per-request breakdown；1/16、16/1、16/16 不做逐请求打点，
  只做干净表 + Little's law。
- 产物：R31 诊断批（与 R30 分开）；拆解 CSV/日志/图。

## 三、判据

- 16 格干净表 failed=0；第三次测试表不覆盖原表。
- 打点批与干净 1/1 的 total 偏移 <3%；超过则降采样并注明。
- 拆解表和第二次测试同口径：理论 = 线速/下限，可达 = 微基准拼，实测 = 第三次 avg。
- 不改 8MB（按用户要求，先不做 8MB）。
