# Summary
## Generating intermediate steps improves LLM performance

  - prompting with intermediate steps
  - Zero-shot, 
  - analogical reasoning, 
  - special decoding：作者惊讶的发现，在不使用逐步推理的prompt情况下，模型往往会给包含COT的回答更高的置信度
    >   具体来说，研究者不再使用传统的贪婪解码，而是探索解码过程中的前k个备选标记（top-k alternative tokens）。
       他们发现，这些备选序列中经常包含CoT推理路径，这表明预训练的LLMs本身就具有一定的推理能力，无需额外的提示。

- Self-consistency greatly improves step-by-step reasoning

    > 1.生成多条推理路径:
    对于一个给定的问题，语言模型不会只生成一个答案，而是被要求生成多条不同的推理过程。
    
    > 2.在每条推理路径完成后，从中提取出最终的答案:答案的形式取决于问题的类型，可能是数值、是/否判断，或者其他具体结论。
    
    > 3.聚合答案以找到一致性:
    将所有推理路径的答案汇总起来，通过某种聚合方法（如多数投票）找出出现最频繁或最一致的答案。例如，如果某个问题生成了5条推理路径，其中3条得出答案“A”，2条得出答案“B”，那么“A”将被选为最终答案。
    
    > Self-Consistency的核心洞察在于：单一推理路径可能出错，但正确答案往往在多条路径中表现出一致性


#### **[Q1] 当LLM直接输出答案而没有中间步骤时，是否仍然需要多次采样并选择最常见的答案？**

**答案**：不需要。

**解释**：  
根据 \(\arg \max P(\text{final answer} \mid \text{problem})\) 的原则，我们的目标是找到给定问题下最可能的最终答案。当LLM直接输出答案时，模型已经在内部完成了推理过程，并给出了它认为最优的答案。这种情况下，模型的输出可以看作是对 \(\arg \max P(\text{final answer} \mid \text{problem})\) 的直接估计。多次采样并选择最常见的答案是一种通过统计方式逼近最优解的方法，但如果LLM已经直接给出了答案，额外采样并不会改变这一结果，因为直接输出已经反映了模型的最大概率判断。因此，在这种情况下不需要多次采样。

#### **[Q2] 改变self-consistency的方法，让LLM一次性生成多个回答，而不是多次采样，然后选择最常见的答案。这种方法是否合理？**

**答案**：不合理。

**解释**：  
self-consistency方法的核心是通过多次独立采样生成不同的推理路径，然后选择最常见的答案，以逼近 \(\arg \max P(\text{final answer} \mid \text{problem})\)。这种方法的有效性依赖于采样的独立性和多样性。如果改为让LLM一次性生成多个回答，这些回答并不是独立采样的结果，而是由模型在单次生成中产生的相关输出。这种方式可能导致回答之间的多样性不足，且无法充分探索所有可能的推理路径，因此不能有效逼近 \(\arg \max P(\text{final answer} \mid \text{problem})\)。相比之下，传统的多次采样方法更能保证结果的鲁棒性，所以这种改变是不合理的。

---
## Limitation
  - irrelevant context
  - self-correction
    - 如果严格地，每一次生成回答都要求自我检查，那么往往会把正确的改成错的；
    - 但是如果只检查错误的答案，那么综合正确率会提高（但模型不知道谁对谁错）
  - premise order