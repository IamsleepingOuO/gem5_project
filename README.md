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

5.在 gem5/src/cpu/BaseCPU.py 中
 - import L3XBar

 ![import l3xbar](https://github.com/user-attachments/assets/77134aeb-e9f8-400a-8b9b-8c3f45b298b3)

 - 定義L3

![在cpu定義l3](https://github.com/user-attachments/assets/55a496e0-62ae-4356-b9f2-30608b3d1f04)
