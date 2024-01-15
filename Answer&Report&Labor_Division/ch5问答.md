stride 算法深入

stride 算法原理非常简单，但是有一个比较大的问题。例如两个 pass = 10 的进程，使用 8bit 无符号整形储存 stride， p1.stride = 255, p2.stride = 250，在 p2 执行一个时间片后，理论上下一次应该 p1 执行。

- 实际情况是轮到 p1 执行吗？为什么？

  ​	不是。

  ​	因为在p2执行完一个时间片之后，它的stride理论上=260，这溢出了8bit无符号整形的储存范围，变成了4，而4<250，所以p2优先级比p1高，接下来还是p2执行。

我们之前要求进程优先级 >= 2 其实就是为了解决这个问题。可以证明，在不考虑溢出的情况下, 在进程优先级全部 >= 2 的情况下，如果严格按照算法执行，那么 STRIDE_MAX – STRIDE_MIN <= BigStride / 2。

- 为什么？尝试简单说明（不要求严格证明）。

    因为，假设STRIDE_MAX =BigStride/x,  STRIDE_MIN=BigStride/y, 其中x和y是大于等于2的整数且x<=y，代表两个stride对应的优先级。
那么STRIDE_MAX – STRIDE_MIN=BigStride/(1/x-1/y)＜BigStride/2。得证。

- 已知以上结论，考虑溢出的情况下，可以为 Stride 设计特别的比较器，让 BinaryHeap<Stride> 的 pop 方法能返回真正最小的 Stride。补全下列代码中的 `partial_cmp` 函数，假设两个 Stride 永远不会相等。

  ​	思路：如果差＞128，则认为已经溢出

        ```
        use core::cmp::Ordering;
      
        struct Stride(u64);
      
        impl PartialOrd for Stride {
            fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
                //补充的代码
                let max_difference = 128;
                let difference = if self.0 > other.0 {
                    self.0 - other.0
                } else {
                    other.0 - self.0
                };
      
                if difference > max_difference {
                    // 如果两个值相差超过128，我们认为较小的值实际上已经溢出
                    if self.0 < other.0 {
                        Some(Ordering::Greater)
                    } else {
                        Some(Ordering::Less)
                    }
                } else {
                    // 在未发生溢出的情况下进行正常比较
                    if self.0 < other.0 {
                        Some(Ordering::Less)
                    } else {
                        Some(Ordering::Greater)
                    }
                }
            }
        }
      
        impl PartialEq for Stride {
            fn eq(&self, other: &Self) -> bool {
                false
            }
        }
      
        ```
    TIPS: 使用 8 bits 存储 stride, BigStride = 255, 则: `(125 < 255) == false`, `(129 < 255) == true`.