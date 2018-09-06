# Dealloc 的实现机制
1. Dealloc 调用流程
    
    * 首先调用_objc_rootDealloc()
    * 接下来调用rootDealloc()这时候会判断是否可以被释放，判断的依据主要有5个，判断是否有以下5种情况

        1. NONPointer_ISA
        2. weakly_reference
        3. has_assoc
        4. has_cxx_dtor
        5. has_sidetable_rc
    
    * 如果有以上五中任意一种，将会调用 object_dispose()方法，做下一步的处理。
    * 如果没有之前五种情况的任意一种，则可以执行释放操作，C函数的 free()。
    * 执行完毕。

2. object_dispose() 调用流程。

    1. 直接调用 objc_destructInstance()。
    2. 之后调用 C函数的 free()。

3. objc_destructInstance() 调用流程

    * 先判断 hasCxxDtor，如果有 C++ 的相关内容，要调用 object_cxxDestruct() ，销毁 C++ 相关的内容。
    * 再判断 hasAssocitatedObjects，如果有的话，要调用 object_remove_associations()，销毁关联对象的一系列操作。
    * 然后调用 clearDeallocating()。
    * 执行完毕。

4. clearDeallocating() 调用流程。

    * 先执行 sideTable_clearDellocating()。
    * 再执行 weak_clear_no_lock,在这一步骤中，会将指向该对象的弱引用指针置为 nil。
    * 接下来执行 table.refcnts.eraser()，从引用计数表中擦除该对象的引用计数。
    * 至此为止，Dealloc 的执行流程结束。


