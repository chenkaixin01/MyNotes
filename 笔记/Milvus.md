# 1 索引
## 1.1 内存索引

| 索引名称     | 索引类型    | 索引特性                                                                                        | 索引占用内存字节数预估                                              | 原理                                                                                                                                               |
| -------- | ------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| FLAT     | 精确检索    | - 数据集相对较小<br>- 需要 100% 的召回率<br>                                                             | 向量占用内存                                                   | 暴力搜索                                                                                                                                             |
| IVF_FLAT | 基于量化的索引 | - 高速查询<br>- 要求尽可能高的召回率<br>                                                                  | 包括两部分，倒排索引和向量占用内存，可以预估为向量占用内存                            | - 倒排索引平面模式，适用于中等规模数据集<br>- 是**一种基于倒排索引的向量索引结构**。 它将高维向量空间划分为多个子空间（称为聚类），并为每个聚类建立一个倒排索引。 在检索过程中，首先根据查询向量的位置确定其所属的聚类，然后在该聚类中进行相似度计算，返回相似度较高的结果。    |
| IVF_SQ8  | 基于量化的索引 | - 高速查询<br>- 内存资源有限<br>- 可接受召回率略有下降                                                          | 包括两部分，倒排索引和SQ量化                                          | - 倒排索引量化模式，适用于大规模数据集，牺牲精度提升速度。<br>- 通过执行标量量化（SQ）将每个 FLOAT（4 字节）转换为 UINT8（1 字节）                                                                   |
| IVF_PQ   | 基于量化的索引 | 查询速度高，召回效果好。  <br>适合集群数据亿级、单机数据量超过1000万的场景。  <br>如果对查询速度要求较高，并且对召回效果要求也较高，可以结合HNSW和OPQ一起使用。 | 包括两部分，倒排索引和PQ压缩编码，可以预估为向量数 * nsubvector                  | - `PQ` (乘积量化）将原始高维向量空间均匀分解为 低维向量空间的笛卡尔乘积，然后对分解后的低维向量空间进行量化<br>- IVF_PQ 先进行 IVF 索引聚类，然后再对向量的乘积进行量化。其索引文件比 IVF_SQ8 更小，但在搜索向量时也会造成精度损失。             |
| SCANN    | 基于量化的索引 | - 极高速查询<br>- 要求尽可能高的召回率<br>- 内存资源大<br>                                                      | 同IVFPQ一致，但是索引同时保存在内存和显存中各一份，占用内存和显存可以预估为向量数 * nsubvector | ScaNN（可扩展近邻）在向量聚类和乘积量化方面与 IVF_PQ 相似。它们的不同之处在于乘积量化的实现细节和使用 SIMD（单指令/多数据）进行高效计算。                                                                   |
| HNSW     | 基于图的索引  | 查询速度很快，召回效果很好。  <br>适合单机1000万数据左右的场景。  <br>缺点是比较耗费内存。                                       | 包括两部分，图索引和向量占用内存，可以预估为向量占用内存                             | HNSW（分层导航小世界图）是一种基于图的索引算法。它根据一定的规则为图像建立多层导航结构。在这个结构中，上层较为稀疏，节点之间的距离较远；下层较为密集，节点之间的距离较近。搜索从最上层开始，在这一层找到离目标最近的节点，然后进入下一层开始新的搜索。经过多次迭代后，就能快速接近目标位置。 |


## 1.2 磁盘索引
