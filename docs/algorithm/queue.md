<!--
 * @Author: your name
 * @Date: 2021-11-17 10:02:23
 * @LastEditTime: 2021-12-22 16:21:33
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: /stars-doc/docs/algorithm/queue.md
-->
### 队列

先入先出（FIFO）的数据结构，即先进入的数据先被处理（删除的话就是先删除第一个进入的数据）。

<img src="https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/08/14/screen-shot-2018-05-03-at-151021.png" alt="img" style="zoom:50%;" />

### 队列实现

队列支持两种操作：入队和出队。入队会向队列追加一个新元素，而出队会删除第一个元素。所以我们需要一个索引来指出起点。

#### 缺点

<img src="https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/21/screen-shot-2018-07-21-at-153713.png" alt="img" style="zoom:50%;" />

如果有一个固定长度的队列，队列满了的时候再进行入队操作，将一个元素出队之后无法将新元素入队。这时候需要用到循环队列。

### 循环队列

