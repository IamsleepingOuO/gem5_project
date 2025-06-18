# Q1:建置環境 (40%)
照著簡報來就行了
# Q2:Enable L3 last level cache in GEM5 + NVMAIN (15%)
1. 在gem5/config/common/Caches.py 中新增L3 Cache
   
![加入l3_cache](https://github.com/user-attachments/assets/4193e6c0-27eb-44cb-aca2-b52cd5f208e5)

2. 在 gem5/config/common/CacheConfig.py
   - 第三個 if 中加上 L3 Cache

![在if中加入l3cache](https://github.com/user-attachments/assets/7033fbd3-8c6c-4f00-a506-6129dde27d8d)

   - 新增一個 if 判斷式，用來判斷 option 中啟用 L3 Cache 的狀況

![修改if l2cache](https://github.com/user-attachments/assets/07fdcc23-eebf-43e5-800f-f1ecea83c730)

3. 在 gem5/config/common/Options.py 中新增一條 option

![新增一條option](https://github.com/user-attachments/assets/0972af8a-f969-48f0-9492-b7fed090060c)


4. 在 gem5/src/mem/XBar.py 中新增class L3XBar (從L2XBar複製即可)

![新增l3xbar](https://github.com/user-attachments/assets/837d71e9-7290-419d-b37f-9483a3ec9466)

5. 在 gem5/src/cpu/BaseCPU.py 中
   - import L3XBar

 ![import l3xbar](https://github.com/user-attachments/assets/77134aeb-e9f8-400a-8b9b-8c3f45b298b3)

   - 定義L3

![在cpu定義l3](https://github.com/user-attachments/assets/55a496e0-62ae-4356-b9f2-30608b3d1f04)

6. 重新編譯
   ```
   scons build/X86/gem5.opt -j$(nproc)
7. 執行時加入--l3cache就可以了
    ```
    ./build/X86/gem5.opt \
     -d m5out \
     configs/example/se.py \
     -c tests/test-progs/hello/bin/x86/linux/hello \
     --cpu-type=TimingSimpleCPU --caches --l2cache --l3cache \
     --mem-type=NVMainMemory \
     --nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config \
     > m5out/run.log 2>&1
    ```
# Q3: Config last level cache to 2-way and full-way associative cache and test performance (15)
1. 2-way
   - 指令
   ```
   ./build/X86/gem5.opt -d m5out_2way \
     configs/example/se.py \
     -c tests/test-progs/hello/bin/x86/linux/benchmark/quicksort \
     --cpu-type=TimingSimpleCPU --caches --l2cache --l3cache \
     --l1i_size=32kB --l1d_size=32kB \
     --l2_size=128kB --l3_size=256kB --l3_assoc=2 \
     --mem-type=NVMainMemory \
     --nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config \
     > m5out_2way/run.log 2>&1
   ```
   ![2way misses](https://github.com/user-attachments/assets/6a6d49bc-af92-439d-9476-10ef799978d9)

2. full-way
   - 指令
   ```
   ./build/X86/gem5.opt -d m5out_fullway \
     configs/example/se.py \
     -c tests/test-progs/hello/bin/x86/linux/benchmark/quicksort \
     --cpu-type=TimingSimpleCPU --caches --l2cache --l3cache \
     --l1i_size=32kB --l1d_size=32kB \
     --l2_size=128kB --l3_size=256kB --l3_assoc=512 \
     --mem-type=NVMainMemory \
     --nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config \
     > m5out_fullway/run.log 2>&1
   ```
   ![fullway misses](https://github.com/user-attachments/assets/0ee463ee-f7f8-42bf-a1f1-27d437269eec)

- 調整L3 Cache到適當大小以觀測結果(我用256kb)
# Q4: Modify last level cache policy based on frequency based replacement policy (15%)
- 我用2-way associative
1. 在 gem5/config/common/Caches.py
   - 新增規則：replacement_policy = Param.BaseReplacementPolicy(LFURP(),"Replacement policy")

   ![新增replacement policy](https://github.com/user-attachments/assets/80875dd9-13c1-45ec-8ca7-a25a2fff2afa)

-原replacement policy: 

![原replacement policy(lru)](https://github.com/user-attachments/assets/1def02d9-1744-4899-9dd0-c09ee828e097)

-更改後replacement policy

![更改後replacement policy (lfu)](https://github.com/user-attachments/assets/acf4455b-eef2-4a96-ac3e-f750a75fed8f)

# Q5: Test the performance of write back and write through policy based on 4-way associative cache with isscc_pcm(15%)
- wrtie back
  - 執行指令
    ```
    ./build/X86/gem5.opt -d m5out \
     configs/example/se.py \
     -c ./tests/test-progs/hello/bin/x86/linux/benchmark/multiply \
     --cpu-type=TimingSimpleCPU \
     --caches --l1d_assoc=4 --l1i_assoc=4 --l2cache --l2_assoc=4 \
     --l3cache --l3_assoc=4 \
     --l1i_size=32kB --l1d_size=32kB \
     --l2_size=128kB --l3_size=512kB \
     --mem-type=NVMainMemory \
     --nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config \
     > m5out/run.log 2>&1
    ```
  - 執行結果

    ![write back 執行結果](https://github.com/user-attachments/assets/507c8d3c-6662-42b6-9873-7950bbf8d2f5)
- write through
  1. 在gem5/src/mem/cache/base.cc 的 BaseCache::access 中加入以下程式碼
     ```
     if (blk->isWritable()) {
       PacketPtr writeclean_pkt = writecleanBlk(blk, pkt->req->getDest(), pkt->id);
       writebacks.push_back(writeclean_pkt);
     }
     ```

     ![write through 更改base cc](https://github.com/user-attachments/assets/037b0b39-6d40-4d67-bade-5a31de0476ae)

  2. 重新編譯
     -指令
     ```
     scons build/X86/gem5.opt -j$(nproc)
     ```
     
  - 執行指令
    ```
    ./build/X86/gem5.opt \
      -d m5out \
      configs/example/se.py \
      -c ./tests/test-progs/hello/bin/x86/linux/benchmark/multiply \
      --cpu-type=TimingSimpleCPU \
      --caches --l2cache --l3cache \
      --l1d_assoc=4 --l1i_assoc=4 \
      --l2_assoc=4 --l3_assoc=4 \
      --l1i_size=32kB --l1d_size=32kB \
      --l2_size=128kB --l3_size=512kB \
      --mem-type=NVMainMemory \
      --nvmain-config=../NVmain/Config/PCM_ISSCC_2012_4GB.config \
      > m5out/run.log 2>&1
    ```
  - 執行結果
    
    ![write through 執行結果](https://github.com/user-attachments/assets/16e98c07-f0d1-47a8-9c42-a13d3b3eaa16)
