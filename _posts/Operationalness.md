**Operationalness**

- *How good is this code from an operational standpoint?* 
- *Specific things to keep in mind: Correctness, Efficiency, Validation of input and good feedback about errors, Robustness, Build script.* 
  - 合理的使用数据结构，如 PriorityQueue。

**Readability**

- *Does the code read well to a programmer familiar with the language?* 
- *Specific things to keep in mind: Idiomatic code, Use of constants; no magic numbers, Size and length, Understandability.* 
  - 对常量做了抽取
  - 逻辑清晰，代码可读性比较高

**Focus on code maintenance**

- *Can the code be extended easily for “part 2” of the problem?* 
- *Specific things to keep in mind: Simplicity and complexity, Extraneous matter, Duplication.* 
  - 

**Code design**

- *How is the design and architecture of the code?* 
- *Specific things to keep in mind: Naming, Organization, Separation of Concerns, Loose coupling, Exception or error handling strategy, SOLID Principles, Delegation, Use of Patterns., Use of Abstraction, Encapsulation, Separation of I/O and non-I/O code, Domain Modeling, Law of Demeter, Extensibility, Use of Data Structures.*

  

  - 每个方法的指责都比较清晰
  - 某些类可以再提供一些方法，避免对外暴露过多细节。比如point 中应该封装一个 addEdge  方法 供 graph 的 addEdge 调用 
  - 抽象类接口，但使用时并为使用接口（public static GraphComputeImpl gci = new GraphComputeImpl(); ）

**Testing**

- *How well did the candidate test their solution?*  
- *Specific things to keep in mind: Tests of any kind, even coarse acceptance tests, Unit tests, Evidence of TDD.*
  - 对每个罗列的问题都做了单元测试
  - 测试方法名过于随意了

**Artistic Impression**

- *Overall, how did you feel about this code?* 
- *Specific things to keep in mind: Textual elegance, Excessive comments, Gold plating or over engineering, User experience when running the program, README included.* 
  * 

**Code Review Grading Scale**

 

We use a numbers-­based scale to indicate how we feel about the candidate’s code submission.

 

-1   No. If you want to hire them, talk to me and convince me.

0    Meh.

1    Good.

2    Great! If we don’t hire this person, talk to me and explain their fatal flaws, because

this code was rocking.





思路清晰，面向对象的思维比较好，各个类的指责划分也很合理
看得出来有工程实践相关的知识，从各种数据结构与算法的运用看出基础应该比较扎实。



对常量做了抽取
合理的使用数据结构，如 PriorityQueue。
每个方法的职责都比较清晰
使用了开源的日志组件
对每个罗列的问题都做了单元测试，且测试方法简洁正确



某些类可以再提供一些方法，避免对外暴露过多细节。比如point 中应该封装一个 addEdge  方法 供 graph 的 addEdge 调用 
抽象类接口，但使用时并为使用接口（public static GraphComputeImpl gci = new GraphComputeImpl(); ）
测试方法名过于随意了



数据结构和算法基础都比较扎实，代码整体设计很不错