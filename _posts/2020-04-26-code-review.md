---
layout: post
title: "Code Review实践"
date: 2020-04-26 00:00:00
categories: refactoring
comments: true
---

# Code Reivew

## 目的

- 提高代码质量
- 降低bug率
- 提升团队技术水平

## 立场

- 不是批评，不是批评，不是批评!
- 不是你的代码，也不是我的代码，是我们的代码。
- 写代码是team work。
- 不要害怕犯错。
- 不论职位高低，经验多少，每个人的代码都应该被Review。

## 方法

交流，学习，提高

### Author

- 在指定Reviewer前，做好Self Review。
- 在指定Reviewer前，处理好Conflicts.
- 在大多时候，在PR中只提供一个需求连接是不够的。
- 在PR中尽量描述：
  - 这个修改要解决什么问题
  - 这个修改如何解决上面这个问题
  - 你解决这个问题的思路，代码是如何组织的

### Reviewer

- 提出建议，而不是命令或要求。
- 从点赞开始，尽量在提出建议前给出赞扬。
- 如果你从某个PR中学到了知识，请不要吝啬的夸赞。
- 不要使用“你”，所有的评论都是针对的代码，而不是人。
- 遇到不理解的地方请果断提出。不好理解的代码通常是因为代码有可以优化的地方，而不是你的能力不足。

### Review什么？

- 命名准确清晰
- 代码是否易于理解
- SOLID: 最重要的是：单一职责
- 边界条件是否考虑完整
- 测试覆盖率是否足够

### 代码质量相关
- [Clean Code](https://www.amazon.cn/dp/B00CBBJWJQ/)
- [Refactoring](https://www.amazon.cn/dp/B07QKC6RN7)
- [Design Pattern](https://sourcemaking.com/design_patterns)
