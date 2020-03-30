-   [出版者的忠告](FromthePublisher/README.md)
-   [致谢](Acknowledgements/README.md)
-   [简介](Introduction/README.md)
-   [第一章 类型推导](DeducingTypes/README.md)

*   [条款 1：理解模板类型推导](DeducingTypes/1-Understand-template-type-deduction.md)
*   [条款 2：理解 auto 类型推导](DeducingTypes/2-Understand-auto-type-deduction.md)
*   [条款 3：理解 decltype](DeducingTypes/3-Understand-decltype.md)
*   [条款 4：知道如何查看类型推导](DeducingTypes/4-Know-how-to-view-deduced-types.md)

-   [第二章 auto 关键字](auto/README.md)

*   [条款 5：优先使用 auto 而非显式类型声明](auto/5-Prefer-auto-to-explicit-type-declarations.md)
*   [条款 6：当 auto 推导出非预期类型时应当使用显式的类型初始化](auto/6-Use-the-explicitly-typed-initializer-idiom-when-auto-deduces-undesired-types.md)

-   [第三章 使用现代 C++](MovingtoModernC++/README.md)

*   [条款 7：创建对象时区分()和{}](MovingtoModernC++/7-Distinguish-between-xxx-when-creating-objects.md)
*   [条款 8：优先使用 nullptr 而不是 0 或者 NULL](MovingtoModernC++/8-Prefer-nullptr-to-0-and-NULL.md)
*   [条款 9：优先使用声明别名而不是 typedef](MovingtoModernC++/9-Prefer-alias-declarations-to-typedefs.md)
*   [条款 10：优先使用作用域限制的 enmu 而不是无作用域的 enum](MovingtoModernC++/10-Prefer-scoped-enum-to-unscoped-enums.md)
*   [条款 11：优先使用 delete 关键字删除函数而不是 private 却又不实现的函数](MovingtoModernC++/11-Prefer-deleted-functions-to-private-undefined-ones.md)
*   [条款 12：使用 override 关键字声明覆盖的函数](MovingtoModernC++/12-Declare-overriding-functions-override.md)
*   [条款 13：优先使用 const_iterator 而不是 iterator](MovingtoModernC++/13-Prefer-const_iterators-to-iterators.md)
*   [条款 14：使用 noexcept 修饰不想抛出异常的函数](MovingtoModernC++/14-Declare-functions-noexcept-if-they-won’t-emit-exceptions.md)
*   [条款 15：尽可能的使用 constexpr](MovingtoModernC++/15-Use-constexpr-whenever-possible.md)
*   [条款 16：保证 const 成员函数线程安全](MovingtoModernC++/16-Make-const-member-functions-thread-safe.md)
*   [条款 17：理解特殊成员函数的生成](MovingtoModernC++/17-Understand-special-member-function-generation.md)

-   [第四章 智能指针](SmartPointers/README.md)

*   [条款 18：使用 std::unique_ptr 管理独占资源](SmartPointers/18-Use-std-unique_ptr-for-exclusive-ownership-resource-management.md)
*   [条款 19：使用 std::shared_ptr 管理共享资源](SmartPointers/19-Use-std-shared_ptr-for-shared-ownership-resource-management.md)
*   [条款 20：在 std::shared_ptr 类似指针可以悬挂时使用 std::weak_ptr](SmartPointers/20-Use-std-weak_ptr-for-std-shared_ptr-like-pointers-that-can-dangle.md)
*   [条款 21：优先使用 std::make_unique 和 std::make_shared 而不是直接使用 new](SmartPointers/21-Prefer-std-make_unique-and-std-make_shared-to-direct-use-of-new.md)
*   [条款 22：当使用 Pimpl 的时候在实现文件中定义特殊的成员函数](SmartPointers/22-When-using-the-Pimpl-Idiom-define-special-member-functions-in-the-implementation-file.md)

-   [第五章 右值引用、移动语义和完美转发](RvalueReferencesMoveSemanticsandPerfectForwarding/README.md)

*   [条款 23：理解 std::move 和 std::forward](RvalueReferencesMoveSemanticsandPerfectForwarding/23-Understand-std-move-and-std-forward.md)
*   [条款 24：区分通用引用和右值引用](RvalueReferencesMoveSemanticsandPerfectForwarding/24-Distinguish-universal-references-from-rvalue-references.md)
*   [条款 25：在右值引用上使用 std::move 在通用引用上使用 std::forward](RvalueReferencesMoveSemanticsandPerfectForwarding/25-Use-std-move-on-rvalue-references-std-forward-on-universal-references.md)
*   [条款 26：避免在通用引用上重定义函数](RvalueReferencesMoveSemanticsandPerfectForwarding/26-Avoid-overloading-on-universal-references.md)
*   [条款 27：熟悉通用引用上重定义函数的其他选择](RvalueReferencesMoveSemanticsandPerfectForwarding/27-Familiarize-yourself-with-alternatives-to-overloading-on-universal-references.md)
*   [条款 28：理解引用折叠](RvalueReferencesMoveSemanticsandPerfectForwarding/28-Understand-reference-collapsing.md)
*   [条款 29：假定移动操作不存在，不廉价，不使用](RvalueReferencesMoveSemanticsandPerfectForwarding/29-Assume-that-move-operations-are-not-present-not-cheap-and-not-used.md)
*   [条款 30：熟悉完美转发和失败的情况](RvalueReferencesMoveSemanticsandPerfectForwarding/30-Familiarize-yourself-with-perfect-forwarding-failure-cases.md)

-   [第六章 Lambda 表达式](LambdaExpressions/README.md)

*   [条款 31：避免默认的参数捕捉](LambdaExpressions/31-Avoid-default-capture-modes.md)
*   [条款 32：使用 init 捕捉来移动对象到闭包](LambdaExpressions/32-Use-init-capture-to-move-objects-into-closures.md)
*   [条款 33：在 auto&&参数上使用 decltype 当 std::forward auto&&参数](LambdaExpressions/33-Use-decltype-on-auto&&-parameters-to-std-forward-them.md)
*   [条款 34：优先使用 lambda 而不是 std::bind](LambdaExpressions/34-Prefer-lambdas-to-std-bind.md)

-   [第七章 并发 API](TheConcurrencyAPI/README.md)

*   [条款 35：优先使用 task-based 而不是 thread-based](TheConcurrencyAPI/35-Prefer-task-based-programming-to-thread-based.md)
*   [条款 36：当异步是必要的时声明 std::launch::async](TheConcurrencyAPI/36-Specify-std-launch-async-if-asynchronicity-is-essential.md)
*   [条款 37：使得 std::thread 在所有的路径下无法 join](TheConcurrencyAPI/37-Make-std-threads-unjoinable-on-all-paths.md)
*   [条款 38：注意线程句柄析构的行为](TheConcurrencyAPI/38-Be-aware-of-varying-thread-handle-destructor-behavior.md)
*   [条款 39：考虑在一次性事件通信上 void 的特性](TheConcurrencyAPI/39-Consider-void-futures-for-one-shot-event-communication.md)
*   [条款 40：在并发时使用 std::atomic 在特殊内存上使用 volatile](TheConcurrencyAPI/40-Use-std-atomic-for-concurrency-volatile-for-special-memory.md)

-   [第八章 改进](Tweaks/README.md)

*   [条款 41：考虑对拷贝参数按值传递移动廉价，那就尽量拷贝](Tweaks/41-Consider-pass-by-value-for-copyable-parameters-that-are-cheap-to-move-and-always-copied.md)
*   [条款 42：考虑使用 emplace 代替 insert](Tweaks/42-Consider-emplacement-instead-of-insertion.md)
