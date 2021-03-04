控制流(没什么好说的，各种语言都差不多)  
涉及的关键字包括 if-else，while，do-while，for，return，break 和选择语句 switch，所有的条件语句都利用条件表达式的“真”或“假”来决定执行路径。  

return：  
1.指定一个方法返回值 (在方法返回类型非 void 的情况下)；  
2.退出当前方法，并返回作用 1 中值。  
break VS continue  
    break 表示跳出当前循环体；  
    continue 表示停止本次循环，开始下一次循环。  
臭名昭著的goto  
goto本身没有错，错的是过度使用；尽管goto是Java中的关键字，但是没有被正式启用，可以在break和continue身上看见goto的影子。  
reak continue goto使用了相同的机制：标签  
break 和 continue 关键字通常只中断当前循环，但若搭配标签一起使用，它们就会中断并跳转到标签所在的地方开始执行。  
```
label1:
outer-iteration { 
  inner-iteration {
  // ...
  break; // [1] 
  // ...
  continue; // [2] 
  // ...
  continue label1; // [3] 
  // ...
  break label1; // [4] 
  } 
}
```  
[1] break 中断内部循环，并在外部循环结束。  
[2] continue 移回内部循环的起始处。但在条件3中，continue label1 却同时中断内部循环以及外部循环，并移至 label1 处。  
[3] 随后，它实际是继续循环，但却从外部循环开始。  
[4] break label1 也会中断所有循环，并回到 label1 处，但并不重新进入循环。也就是说，它实际是完全中止了两个循环。  