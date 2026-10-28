# 交接文件（单向：开发指挥 Agent → 测试执行 Agent）

- 最后更新：2026-10-27 15:45
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
| E3-H | 2026-10-27 | 下载第二次测试分析报告到本地归档 | 待执行（报告已更新，需重拷） | — |
| E3-I | 2026-10-28 | 4MB 1/1 时延打点拆分（query/read/copy/free） | 已关闭 | control 41–54；copy 416（pageable 384+32）；b−a 259；UB 178/337；12.69 = PCIe HBM write |
| E3-J | 2026-10-28 | PCIe 链路核对（V100 / UB NIC，只读） | 已关闭 | GPU 3.0 x16；Mellanox 4.0 x8；UB 不在 PCIe；12.69 是 UB 上限 |

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
  --key_include_value_size=true` + `MC_UB_STAGING_POOL_SIZE=4G` + `numactl --cpunodebind=0`。

## 交付说明必写

1. 8MB b 16/16 的 211.6 ms 是 2 GiB 在飞的排队时长（Little's law 吻合），不是退化；
   不要与失败态的 71 ms 对比。
2. 4MB R08 与首轮 R01、日报里的第一轮数据不是同一批，不可混用。
3. `--key_include_value_size=true` 与 `--verify=true` 互斥；8MB 完整性用
   “写 100/100 + 四路径抽 key + 全量回读”三证。
4. 8MB 的 staging 路径依赖 `MC_UB_STAGING_POOL_SIZE=4G`；默认 1G 会在 16×16×8MB 下溢出。
5. b 的路径计数器（cuda_io/host_io）均为 0，路径证明靠 `--gpu_mode=staging` + 延迟量级；
   既有埋点缺口。
6. **12.69/12.68 GiB/s 是 PCIe 3.0 x16 HBM write 的 86.5%**（E3-I）；UB 是传输，
   b 的 UB 读 178 µs ≈ 22.5 GiB/s；E3-J 的“UB 12.69”是过度归因。
7. 4MB 1/1 的 b 650 = UB 178 + PCIe 拷贝 416（pageable 384+32）+ query 42 + residual 12；
   组长的 500+ 假设 copy ~266，实际 416，1/1 不可达。

---

# E3-H（待执行）· 下载分析报告到本地

> 报告已在 git 仓：`reports/第二次测试结果分析报告.md`（与本文件同一仓，已 push）。
> 报告已更新（E3-I 打点后 §3.6 为最新，§3.5 为旧模型；§2.4 的 12.69 归因也已修正）；
> 若已归档旧版，请重新复制覆盖。

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
