## Reference
https://www.jianshu.com/p/8c6b056f73ce </br>

## DPDK
传统数据包处理方式存在以下两个难题
- 每来一次报文，就触发一次中断。大量数据的到来会导致频繁的中断触发，系统承受不了如此频繁的中断。
- 数据包需要从内核态拷贝到用户态，当数据量巨大时，这个拷贝操作会大幅的降低数据包处理性能。

## Zero Copy
zero copy技术就是减少不必要的内核缓冲区跟用户缓冲区间的拷贝，从而减少CPU的开销和内核态切换开销，达到性能的提升。</br>
![](https://upload-images.jianshu.io/upload_images/207235-0c63dbf565386423.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/512/format/webp)
![](https://upload-images.jianshu.io/upload_images/207235-bc756e05a212b2ef.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/429/format/webp)
