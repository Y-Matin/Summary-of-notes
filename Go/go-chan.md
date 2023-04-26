### golang channel  常见场景 的表现

#### 写入：
- 未初始化chan： 阻塞
- 已初始化未满： 立即写入
- 已初始化已满： 阻塞写入
- 已经close的chan： 触发panic （panic: send on closed channel）

 #### 读取：
- 未初始化chan： 阻塞
- 已初始化未满： 立即读取
- 已初始化已满： 阻塞读取
- 已经close的chan： 读取零值

 #### 关闭：
- 未初始化chan：触发panic （panic: close of nil channel）
- 已初始化的，未close的： 关闭成功，读完数据后，返回零值
- 已经close的chan：触发panic （panic: close of closed channel）

### 总结： 
- 读取 非常 稳定，不会panic，只会阻塞
- close 非常严格，非正常情况close，会触发panic
- 写入比较特殊,写入未初始化的，会阻塞；写入已经close的，会panic


- 注意点是 判断chan是否close
- i,ok:= <- chan，
- ok 只是表示 从chan中取的数据是否是有效的，《非零值》
只有当ok是false时，表示 chan被close了，
ok是true，chan可能没有被close，也可能是close了，但chan里面还有数据


- 写入 未初始化的chan 会阻塞， 当chan 被初始化后，可以被之前的逻辑正确写入