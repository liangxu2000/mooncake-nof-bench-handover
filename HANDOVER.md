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
5. 若出现代码补丁，放在 `patches/`，diff 一律 base64 编码（防空白被吃）。

## 资料请求（请优先补，按优先级）

1. 昨天 d=878 µs 那次的采集上下文：CSV 行 + 当时 client 的完整启动命令/env（重点 `MC_STORE_USE_HUGEPAGE`、`--local_buffer_size`、`MC_NOF_SUBMIT_CHUNK_BYTES`、绑核方式）。
2. 脚本原文：`run/mc_bench_point.sh`、`run/mc_ports.env`、`preflight_affinity_check.sh`、E2 的 nvcc 源码。
3. 原始 CSV：`e1_rerun_20261027.csv`、`e2_gpu_copy_20261027.csv`、`mooncake_get_d_20261026.csv`。
4. 昨晚到今早的重启/变更时间线（台账 §28 末尾即可）。

## 任务历史

| 编号 | 日期 | 内容 | 状态 | 回执摘要 |
| --- | --- | --- | --- | --- |
| E3-A | 2026-10-27 | 暂存池 NUMA/pin 定位 + 拷贝源标定 | 待执行 | — |

---

# E3-A（待执行）

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

## 执行顺序（建议）

1. **先读文件、后动手**：`git pull` 后确认任务状态为"待执行"；如与本仓描述和本地实测冲突，以本地原始证据为准，并回报冲突点。
2. **补资料**：按上方"资料请求"1→4 补齐；能直接传文件的传文件，不能传的内嵌关键原文（启动命令/env 必须逐行原文）。
3. **判据 1 快照核查**：用现有只读探测脚本 + 现查，不重启任何服务；任何一项不满足就停手回报。
4. **判据 2 c/d 交替**：顺序建议 `c,d,c,d,c,d`；每点 60 s，点间留 10–30 s；每轮记 GiB/s、avg/p99、三计数器、节点 load。
5. **判据 3 池属性**：在任一轮 d 运行中抓（进程存活 60 s 足够）；贴 `/proc/<pid>/maps` 与 `numa_maps` 原始行。
6. **判据 4 拷贝标定**：在 E2 的 nvcc 程序上加三种源（pinned / pageable / 2 GB 大块按 NUMA0 与非 NUMA0），各 300 次；无 libnuma 就用 `numactl --membind`。
7. **回报**：按 §1.3 五段式；每个判据附原始输出片段；CSV 落 `results/e3a_copy_source_20261028.csv` 并追加台账 §29。
8. **停手条件**：判据 1 不满足；c 明显变慢；任何步骤需要重启或改驱动 —— 先回报等批准。

## 回报要求

- 固定格式（§1.3）：【结论】【证据】【未做/未验证】【风险与偏差】【产物】
- CSV 建议列：`mode,size_bytes,iterations,avg_us,min_us,p99_us,bandwidth_GiBps,source_numa,dst`
- 判据 3 的抓取建议在 d 点位运行中做（进程存活 60 s，足够）
- 若环境无 libnuma，可用 `numactl --membind=0` / `--membind=<非0>` 跑同一个二进制
- 本轮不需要回答"怎么修"，只要把 4c 与 4a 的差距、以及池子实际落点报清楚

## 下一步（由开发指挥 Agent 负责）

结果回来后：若 4c 明显慢于 4a，出"池绑 NUMA0 / pin 池"的最小补丁（单实验单补丁、附回退命令），重编后先测 b/d 1/1 验证收益；无收益（<5%）即回退，转入全表同快照重跑。
