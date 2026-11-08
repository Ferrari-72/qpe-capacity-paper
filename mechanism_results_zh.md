# Stage2 适用机制验收实际结果（2026-11-08）

工作稿，未经同行评审。

## 运行与验收

v3/v3a 机制包（输出改为逐行证据文件，数学与审计逻辑与 v2 逐字节相同）在固定镜像 sha256:286b4627… 中逐行运行：network none、只读根、1 CPU、uid 65534，每行最长 4 小时。5 个 ADCRS 行全部 HOST_ACCEPTED，docker exit 0、OOM false，容器已删除；每行均为 COMPLETED_SAME_BASE_FINITE_DOMAIN。independent integer audit 全部 all_match=true，固定节点证书在所有槽位都是 UNREJECTED_BY_FIXED_NODES。

| ordinal | 行 | 证据字节 | 用时 | 候选槽 |
|---|---|---:|---:|---|
| 2 | N48-s20-L_ADCRS | 35222401 | 60 s | native1, native2, signal1, seeded_other |
| 4 | N64-s10-L_ADCRS | 30777314 | 37 s | native1, native2 |
| 6 | N64-s20-L_ADCRS | 27906209 | 45 s | native1, native2 |
| 8 | N96-s10-L_ADCRS | 70203741 | 114 s | native1, native2 |
| 10 | N96-s20-L_ADCRS | 84646188 | 129 s | native1, native2 |

数值汇总见 `results/qpe_mechanism_v3_summary_20261108.txt`，原始证据在 `results/qpe_mechanism_v3_ord2`、`v3_ord4`、`v3a_ord6/8/10`。v3 的 SCIENCE_SOURCES 只记录了 v8 时期的 entry/screen 哈希，所以 R60 及之后的行最初被拒，拒绝原因是 frozen science source map 不一致。v3a 改为从各行自己的 receipt 取 source map；baselines、generator、trace、v5_common 四个科学文件均断言未变。被拒的尝试保留在 `results/qpe_mechanism_v3_failed_staging_20261108/`。

## 结果（相对实际 incumbent 的总工作量下降）

| ord / 槽 | S（只切片） | F（只换树） | J（联合） | FS/SF | slices I→J |
|---|---:|---:|---:|---:|---|
| 2 native1 | 0.02% | 3.73% | 3.76% | 3.76% | 2→2 |
| 2 native2 | 0.02% | 14.13% | 14.16% | 14.16% | **2→1** |
| 2 signal1 | 0.02% | 3.73% | 3.76% | 3.76% | 2→2 |
| 2 seeded | 0.02% | 5.32% | 5.34% | 5.34% | 2→2 |
| 4 native1/2 | 0.00% | 0.00% | 0.00% | 0.00% | 3→3 |
| 6 native1/2 | 0.02% | 0.00% | ≤0.03% | ≤0.03% | 1→1 |
| 8 native1/2 | 0.00% | 0.00% | 0.00% | 0.00% | 28→28 |
| 10 native1/2 | ≤0.01% | 0.00% | ≤0.02% | ≤0.02% | 19→19 |

## 解读

1. 机制检验在真实返回底座上完整执行，没有因实现问题中断，审计全部一致。
2. 只有最小的 ord2 出现有意义的局部改善，而且改善几乎全部来自换树（F），只切片（S）只有约 0.02%。联合 J 比 F 只多 0.03 个百分点左右；严格交替 FS/SF 与 J 相同。
3. 切片恢复只出现一次：ord2 的 native2 槽从 2 个切片变成 1 个，同时总工作量下降 14.16%。这一次是"联合能恢复切片"的直接证据；其余 13 个槽都没有出现。
4. 对 N64/N96 的 ADCRS 行，有限域内的 I/F/S/J 几乎完全相同：native 返回的底座在这 10 节点 frontier、6 可编辑的局部域里已接近局部最优。
5. 搜索前信号槽（signal1）与 native1 结果相同，没有表现出优于 native 排序的选择能力；seeded_other 反而略好。

结论：在这批自然 QPE 网络的 ADCRS 表示上，"容量受阻、换树后才能恢复切片"的机制确实存在，但只在 1/14 个槽位出现。大多数实际底座的局部域里，这一机制几乎不贡献收益。这是一个限定范围的负结果，不能写成 QPE 专用机制成立。

## 未覆盖

- no_ADCRS 行（1、3、5、7、9、11）：每个固定节点叶集的证书规模估计为每槽数百 MB 到数十 GB，计算需数小时到数天。记为 MECHANISM_IMPLEMENTATION_BLOCKED，不是"不适用"。
- ord0 为 NULL（没有切片），不适用。
- 生产来源（N48–N96 完整 provenance 与 hook 不变性）未实跑；scalable cone 证书源码在 `paper/qpe_production_scalable_cone_source_v1/`，NOT_RUN。

对 Stage3 的含义：搜索前容量信号在 Stage2 中没有表现出排序优势，所以 Stage3 的四臂比较按协议只作为通用根排序可行性诊断，预期很可能是 null 对比。
