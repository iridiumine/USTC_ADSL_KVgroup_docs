# paper list

## Summary about LSM-tree

- Luo, Chen, and Michael J. Carey. "LSM-based storage techniques: a survey." *The VLDB Journal* 29.1 (2020): 393-418.

## LSM-tree read and write amplification optimization

- L. Lu, T. S. Pillai, A. C. Arpaci-Dusseau, and R. H. Arpaci-Dusseau. " WiscKey: Separating keys from  values in SSD-conscious storage. " USENIX FAST, 2016.
- P. Raju, R. Kadekodi, V. Chidambaram, and I. Abraham. " PebblesDB: Building key-value stores using  fragmented log-structured merge trees. " ACM SOSP, 2017.
- N. Dayan, M. Athanassoulis, and S. Idreos. " Monkey: Optimal navigable key-value store. " ACM  SIGMOD, 2017.
- O. Balmau, D. Didona, R. Guerraoui, W. Zwaenepoel, H. Yuan, A. Arora, K. Gupta, and P. Konka. "  TRIAD: Creating synergies between memory, disk and log in log structured key-value stores. " USENIX  ATC, 2017. 
- Y. Li, C. Tian, F. Guo, C. Li, and Y. Xu. " ElasticBF: Elastic bloom filter with hotness awareness for  boosting read performance in large key-value stores. " USENIX ATC, 2019. 
- T. Yao, Y. Zhang, J. Wan, Q. Cui, .ang, H. Jiang, C. Xie, and X. He. "MatrixKV: Reducing Write Stalls and  Write Amplification in LSM-tree Based KV Stores with Matrix Container in NVM." USENIX ATC, 2020.
- Zhang, Qiang, et al. "UniKV: Toward high-performance and scalable KV storage in mixed workloads via unified indexing." IEEE ICDE, 2020.
- Yongkun Li, Zhen Liu, Patrick P. C. Lee, Jiayu Wu, Yinlong Xu, Yi Wu, Liu Tang, Qi Liu, and Qiu Cui.  "Differentiated Key-Value Storage Management for Balanced I/O Performance.“ ATC 2021.

## New Media: PM KV Storage

- I. Oukid, J. Lasperas, A. Nica, T. Willhalm, and W. Lehner, "Fptree: A hybrid scm-dram 
  persistent and concurrent b-tree for storage class memory." ACM SIGMOD, 2016.
- D. Hwang, W. H. Kim, Y. Won, and B. Nam, "Endurable transient inconsistency in byte-addressable persistent b+-tree." USENIX FAST, 2018.
- O. Kaiyrakhmet, S. Lee, B. Nam, S. H. Noh, and Y. R. Choi, "SLM-DB: Single-Level Key-Value store with persistent memory." USENIX FAST, 2019.
- Y. Chen, Y. Lu, F. Yang, Q. Wang, Y. Wang, and J. Shu, "Flatstore: An efficient log-structured key-value 
- L. Benson, H. Makait, and T. Rabl, "Viper: an efficient hybrid pmem-dram key-value 
  store." VLDB 2021.
- W. H. Kim, R. M. Krishnan, X. Fu, S. Kashyap, and C. Min, “Pactree: A high performance 
  persistent range index using pac guidelines." ACM SOSP, 2021.

## Distributed KV

-  J. C. Corbett, J. Dean, M. Epstein, A. Fikes, C. Frost, J. J. Furman, S. Ghemawat, A. Gubarev, C. Heiser, 
  P. Hochschild, W. Hsieh, S. Kanthak, E. Kogan,H. Li, A. Lloyd, S. Melnik, D. Mwaura, D. Nagle, S. 
  Quinlan, R. Rao, L. Rolig, Y. Saito, M. Szymaniak, C. Taylor, R. Wang, and D. Woodford. " Spanner: 
  Google’s globally distributed database. " ACM TOCS, 2013.
- H. Zhang, M. Dong, and H. Chen. " Efficient and available in-memory KV-store with hybrid erasure coding and replication. " USENIX FAST, 2016.
- X. Jin, X. Li, H. Zhang, R. Soulé, J. Lee, N. Foster, C. Kim, and I. Stoica. "Netcache: Balancing key-value 
  stores with fast in-network caching." ACM SOSP, 2017.
- B. Chandramouli, G. Prasaad, D. Kossmann, J. Levandoski, J. Hunter, and M. Barnett. " FASTER: A 
  concurrent key-value store with in-place updates. " ACM SIGMOD, 2018.
- J. Chen, L. Chen, S. Wang, G. Zhu, Y. Sun, H. Liu, and F. Li. " HotRing: A hotspot-aware in-memory keyvalue store. " USENIX FAST, 2020.
- Q. Zhang, Y. Li, Patrick P. C. Lee, Yi. Xu, and S. Wu. "DEPART: Replica Decoupling for Distributed Key-Value Storage." USENIX FAST, 2022. 

## Index

- V. Leis, A. Kemper, T. Neumann. "The Adaptive Radix Tree: ARTful Indexing for Main-Memory  Databases.“ IEEE ICDE, 2013. 
- X. Wu, F. Ni, S. Jiang. "Wormhole: A fast ordered index for in-memory data management." EuroSys, 2019.
- R. Binna, E. Zangerle, M. Pichl, G. Specht, Vi. Leis. "HOT: A Height Optimized Trie Index for Main-Memory Database Systems." ACM SIGMOD, 2018. 
- T. Kraska, A. Beutel, Ed H. Chi, J. Dean, N. Polyzotis. "The Case for Learned Index Structures." ACM  SIGMOD, 2018. 
- J. Ding, U. F. Minhas, J. Yu, C. Wang, J. Do, Y. Li, H. Zhang, B. Chandramouli, J. Gehrke, D. Kossmann,  D. Lomet, T. Kraska. "ALEX: An Updatable Adaptive Learned Index." ACM SIGMOD, 2020. 
- C. Tang, Y. Wang, Z. Dong, G. Hu, Z. Wang, M. Wang, H. Chen. "XIndex: A Scalable Learned Index for  Multicore Data Storage." PPoPP, 2020. 
- Yifan Dai, Yien Xu, Aishwarya Ganesan, Ramnatthan Alagappan, Brian Kroth, Andrea C. Arpaci-Dusseau,  Remzi H. Arpaci-Dusseau. From WiscKey to Bourbon: A Learned Index for Log-Structured Merge Trees.  OSDI 2020.

## Application

- B. Iordanov. Hyper " GraphDB: A Generalized Graph Database. " WAIM, 2010. 
- R. Cheng, J. Hong, A. Kyrola, Y. Miao, X. Weng, M. Wu, F. Yang, L. Zhou, F. Zhao, and E. Chen. " Kineograph: taking the pulse of a fast-changing and connected world. " ACM EuroSys, 2012.
- Mondal and A. Deshpande. " Managing large dynamic graphs efficiently. " ACM SIGMOD,  2012. 
- B. Shao, H. Wang, and Y. Li. " Trinity: A Distributed Graph Engine on a Memory Cloud. " ACM  SIGMOD, 2013.
- Matsunobu, Yoshinori, S. Dong, and H. Lee. "MyRocks: LSM-tree database storage engine serving Facebook's Social Graph." VLDB, 2020.
- Dongxu Huang, Qi Liu, Qiu Cui, Zhuhe Fang, Xiaoyu Ma, Fei Xu, Li Shen, Liu Tang, Yuxing Zhou, Menglong Huang, Wan Wei, Cong Liu, Jian Zhang, Jianjun Li, Xuelian Wu, Lingyu Song,  Ruoxi Sun, Shuaipeng Yu, Lei Zhao, Nicholas Cameron, Liquan Pei, Xin Tang. TiDB: A Raft- based HTAP Database. VLDB 2020.
- Markus Pilman, Kevin Bocksrocker, Lucas Braun, Renato Marroquín, and Donald Kossmann. 2017. Fast scans on key-value stores. Proc. VLDB Endow. 10, 11 (August 2017), 1526–1537. 