---
chapter: 07-AI基础-Prompt与LLM
week: W06
title: 从零实现 GPT
duration: 10-14 天
prereq: W01-W05、Python、基础线性代数
goal: 在 M1/M2 Mac 上从零写出能模仿莎士比亚的小 GPT，建立 LLM 终极心智模型
tags: [transformer, attention, pytorch, nanogpt, from-scratch]
updated: 2026-06-19
---

# W06 · 从零实现 GPT

> 这一周是**整章的终极挑战**：你将不再把 LLM 当黑盒，而是亲手把 Transformer 的每一个齿轮拼起来。完成它，你看 Claude / GPT 的眼神都不一样了。

---

## 0. 为什么必须从零写一遍

读 100 篇 RAG 综述不如自己训出一个 GPT 的那个下午。原因：

1. **建立心智模型**：调 prompt、做 RAG、写 Agent，本质都基于"模型怎么看 token"的直觉
2. **去神秘化**：Transformer 加起来 200 行代码，没那么玄
3. **求职资本**：项目+博客比 leetcode 抢眼

不是要你训 GPT-5，而是把"小莎士比亚 GPT"训出来——它会写"To be or not to be"风的废话，但**每一行代码你都懂**。

---

## 1. 神经网络 10 分钟速览

> 已学过可跳。

### 1.1 神经元

输入 x，权重 w，偏置 b，激活 σ：

```
y = σ(w·x + b)
```

### 1.2 层

把 N 个神经元堆一起 = 一层；堆 L 层 = 深度网络。

### 1.3 训练 = 找 w

定义 **损失** L（预测与真值差距），对 w 求梯度（链式法则 = 反向传播），用 **优化器**（SGD / Adam）沿负梯度更新 w。

```
w ← w - lr × ∂L/∂w
```

数据反复喂进来 → loss 一步步降 → 模型学会任务。

---

## 2. PyTorch 最小集

```python
import torch, torch.nn as nn

# 1. tensor
x = torch.tensor([1.0, 2, 3], requires_grad=True)

# 2. autograd
y = (x ** 2).sum()
y.backward()
print(x.grad)   # tensor([2., 4., 6.])

# 3. nn.Module
class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = nn.Linear(3, 1)
    def forward(self, x): return self.fc(x)

# 4. optimizer
net = Net()
opt = torch.optim.AdamW(net.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()

for _ in range(100):
    pred = net(torch.randn(8, 3))
    loss = loss_fn(pred, torch.zeros(8, 1))
    opt.zero_grad(); loss.backward(); opt.step()
```

知道这五样（tensor / autograd / nn.Module / optimizer / loss）就够入门。

---

## 3. 数据：tinyshakespeare

```bash
curl -O https://raw.githubusercontent.com/karpathy/char-rnn/master/data/tinyshakespeare/input.txt
```

约 1MB，全是莎翁戏剧。我们让模型学会接龙。

```python
text = open("input.txt").read()
print(text[:200])
```

---

## 4. Tokenizer：先字符级，再 BPE

### 4.1 字符级（最简单）

```python
chars = sorted(set(text))
vocab = {c:i for i,c in enumerate(chars)}
ivocab = {i:c for c,i in vocab.items()}

encode = lambda s: [vocab[c] for c in s]
decode = lambda l: "".join(ivocab[i] for i in l)
```

vocab 大约 65 个字符。

### 4.2 BPE（GPT 用的）

字符级太长（"hello" = 5 token），效率低。BPE 把高频字符对**合并**：

```
初始：h e l l o
统计："l l" 频率最高 → 合并为 "ll"
新序列：h e ll o
继续统计：合并最高频对……
```

可以用 `tiktoken` 直接看 GPT 的 tokenizer：

```python
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")
print(enc.encode("hello world"))  # [15339, 1917]
```

第六周教程**先用字符级**通关，最后选做章节再写 BPE。

---

## 5. Embedding 层

把整数 token id 映射成 d 维稠密向量：

```python
emb = nn.Embedding(vocab_size, n_embd)
ids = torch.tensor([5, 12, 7])
print(emb(ids).shape)   # (3, n_embd)
```

理解：每个 token 学到一个"语义向量"。

---

## 6. Self-Attention 核心

> 这是 Transformer 的灵魂，必懂。

### 6.1 直觉

读到 "The bank raised rates"，你想搞清 bank 是哪个 bank，会**回头看上下文**。Attention 让每个 token **看其他 token，并加权融合信息**。

### 6.2 公式

对每个 token 学 3 个投影：

- Q (query)：我要找什么
- K (key)：我能被什么找到
- V (value)：我携带什么信息

```
Attention(Q,K,V) = softmax(Q K^T / √d_k) V
```

`Q K^T` 衡量每对 token 相似度；除 √d_k 防数值过大；softmax 转概率；乘 V 加权聚合。

### 6.3 PyTorch 实现（单头）

```python
class SelfAttention(nn.Module):
    def __init__(self, n_embd, head_size, block_size):
        super().__init__()
        self.q = nn.Linear(n_embd, head_size, bias=False)
        self.k = nn.Linear(n_embd, head_size, bias=False)
        self.v = nn.Linear(n_embd, head_size, bias=False)
        self.register_buffer("mask",
            torch.tril(torch.ones(block_size, block_size)))
        self.head_size = head_size

    def forward(self, x):
        B, T, C = x.shape
        Q, K, V = self.q(x), self.k(x), self.v(x)
        att = Q @ K.transpose(-2, -1) / self.head_size**0.5
        att = att.masked_fill(self.mask[:T,:T] == 0, float("-inf"))
        att = att.softmax(-1)
        return att @ V
```

注意 `tril` 下三角 mask = **causal mask**：第 t 个 token 只能看 0..t，不能偷看未来。

---

## 7. Multi-Head Attention

把 d 维拆成 h 个 head 各看一部分，再拼起来：

```python
class MHA(nn.Module):
    def __init__(self, n_embd, n_head, block_size):
        super().__init__()
        head_size = n_embd // n_head
        self.heads = nn.ModuleList([
            SelfAttention(n_embd, head_size, block_size) for _ in range(n_head)])
        self.proj = nn.Linear(n_embd, n_embd)
    def forward(self, x):
        out = torch.cat([h(x) for h in self.heads], dim=-1)
        return self.proj(out)
```

每个 head 学不同关系（句法 / 共指 / 语义……），效果远好于单头。

---

## 8. Position Embedding

Attention 本身**对顺序无感**（"猫咬狗" = "狗咬猫"），必须加位置信息。

### 8.1 绝对位置 (GPT-2)

```python
pos_emb = nn.Embedding(block_size, n_embd)
x = tok_emb + pos_emb(torch.arange(T))
```

### 8.2 相对 / RoPE (LLaMA / Claude)

把 Q、K 旋转一个与位置相关的角度，自然带相对位置信息，且能外推到更长序列。简化版：

```python
def rope(x, pos):
    d = x.size(-1); half = d // 2
    freq = 1 / (10000 ** (torch.arange(0, half) / half))
    angle = pos[:, None] * freq[None, :]
    cos, sin = angle.cos(), angle.sin()
    x1, x2 = x[..., :half], x[..., half:]
    return torch.cat([x1*cos - x2*sin, x1*sin + x2*cos], -1)
```

入门先用绝对位置，跑通后再换 RoPE。

---

## 9. Feed Forward + LayerNorm + 残差

```python
class FFN(nn.Module):
    def __init__(self, n_embd):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_embd, 4*n_embd),
            nn.GELU(),
            nn.Linear(4*n_embd, n_embd),
        )
    def forward(self, x): return self.net(x)
```

Block 把它们组合：

```python
class Block(nn.Module):
    def __init__(self, n_embd, n_head, block_size):
        super().__init__()
        self.ln1 = nn.LayerNorm(n_embd)
        self.attn = MHA(n_embd, n_head, block_size)
        self.ln2 = nn.LayerNorm(n_embd)
        self.ffn = FFN(n_embd)
    def forward(self, x):
        x = x + self.attn(self.ln1(x))   # 残差
        x = x + self.ffn(self.ln2(x))
        return x
```

**Pre-LN** 写法（先 norm 再 sublayer），训练稳定性好。

---

## 10. 完整 GPT 模型

```python
class GPT(nn.Module):
    def __init__(self, vocab_size, n_embd=192, n_head=6, n_layer=6, block_size=128):
        super().__init__()
        self.block_size = block_size
        self.tok_emb = nn.Embedding(vocab_size, n_embd)
        self.pos_emb = nn.Embedding(block_size, n_embd)
        self.blocks = nn.Sequential(*[
            Block(n_embd, n_head, block_size) for _ in range(n_layer)])
        self.ln_f = nn.LayerNorm(n_embd)
        self.head = nn.Linear(n_embd, vocab_size, bias=False)

    def forward(self, idx, targets=None):
        B, T = idx.shape
        x = self.tok_emb(idx) + self.pos_emb(torch.arange(T, device=idx.device))
        x = self.blocks(x); x = self.ln_f(x)
        logits = self.head(x)
        loss = None
        if targets is not None:
            loss = nn.functional.cross_entropy(
                logits.view(-1, logits.size(-1)), targets.view(-1))
        return logits, loss

    @torch.no_grad()
    def generate(self, idx, max_new=200, temperature=1.0, top_k=None):
        for _ in range(max_new):
            idx_cond = idx[:, -self.block_size:]
            logits, _ = self(idx_cond)
            logits = logits[:, -1, :] / temperature
            if top_k:
                v, _ = torch.topk(logits, top_k)
                logits[logits < v[:, [-1]]] = -float("inf")
            probs = logits.softmax(-1)
            nxt = torch.multinomial(probs, 1)
            idx = torch.cat((idx, nxt), 1)
        return idx
```

---

## 11. 训练循环

```python
import torch
data = torch.tensor(encode(text), dtype=torch.long)
n = int(0.9 * len(data))
train_data, val_data = data[:n], data[n:]

block_size, batch_size = 128, 64
device = "mps" if torch.backends.mps.is_available() else "cpu"

def get_batch(split):
    src = train_data if split == "train" else val_data
    ix = torch.randint(len(src) - block_size - 1, (batch_size,))
    x = torch.stack([src[i:i+block_size] for i in ix])
    y = torch.stack([src[i+1:i+1+block_size] for i in ix])
    return x.to(device), y.to(device)

model = GPT(vocab_size=len(chars), block_size=block_size).to(device)
opt = torch.optim.AdamW(model.parameters(), lr=3e-4)

for step in range(5000):
    xb, yb = get_batch("train")
    _, loss = model(xb, yb)
    opt.zero_grad(set_to_none=True); loss.backward(); opt.step()
    if step % 500 == 0:
        with torch.no_grad():
            xv, yv = get_batch("val")
            _, vloss = model(xv, yv)
        print(f"step {step}: train {loss.item():.3f}  val {vloss.item():.3f}")

# 生成
ctx = torch.zeros((1,1), dtype=torch.long, device=device)
print(decode(model.generate(ctx, 500)[0].tolist()))
```

M1/M2 Mac MPS 上约 **10-20 分钟**能训到 loss ~1.5，输出已有莎翁味儿：

```
ROMEO:
That stay'd to break the field and the fortune
Of Lancaster the doom which my brother...
```

---

## 12. 生成策略

| 方法 | 说明 |
|---|---|
| **贪心** | 每步取最大概率，重复严重 |
| **temperature** | <1 更确定，>1 更随机 |
| **top-k** | 只在概率前 k 中采样 |
| **top-p (nucleus)** | 累计概率到 p 内采样，更自适应 |

实战：`temperature=0.8, top_k=200` 是常用组合。

---

## 13. 跟着两本"圣经"

### 13.1 nanoGPT (Karpathy)

仓库：`karpathy/nanoGPT`。一个 ~300 行 Python 实现 GPT-2 复现。**所有概念**都看得见摸得着。建议把 `model.py` 抄一遍。

### 13.2 LLMs-from-scratch (Sebastian Raschka)

仓库：`rasbt/LLMs-from-scratch`。**章节式教学**（含 BPE、SFT、LoRA、DPO），中文社区翻译多。和本周联动学最佳。

---

## 14. 推理优化简介（了解即可）

| 技术 | 一句话 |
|---|---|
| **KV Cache** | 推理时缓存历史 K/V，避免重复计算 |
| **Flash Attention** | IO 感知的 attention 实现，2-4× 提速 |
| **量化 (GGUF / AWQ / GPTQ)** | 把 fp16 压成 int4，内存降 4× |
| **PagedAttention (vLLM)** | 像 OS 分页一样管理 KV，吞吐 ↑↑ |
| **Speculative Decoding** | 小模型先猜，大模型批量校验 |

llama.cpp + GGUF 让 70B 模型在你笔记本上跑成为可能。

---

## 15. 实战交付

完成清单：

1. **跑通**字符级小 GPT，loss < 1.5
2. **可视化** loss 曲线（matplotlib / wandb）
3. **生成 500 字莎翁腔**
4. **加 dropout** 看过拟合改善
5. **换 RoPE**，对比效果
6. **写 5000 字博客**：《我从零实现了一个 GPT》
   - 结构：动机 → Transformer 拆解 → 关键代码截图 → loss 曲线 → 生成样例 → 踩过的坑 → 我学到了什么
   - 发布到 GitHub Pages / 掘金 / 知乎

---

## 16. 练习

1. **数据替换**：用《红楼梦》训中文小 GPT
2. **加 SFT**：手工编 100 对"指令-回答"，让模型听话
3. **加 LoRA**：只训低秩矩阵，省 10× 显存
4. **加 KV Cache**：让 generate 速度翻倍
5. **跑 Karpathy 的 GPT-2 复现**：扩到 124M 参数

---

## 17. Checklist

- [ ] PyTorch 五件套熟练
- [ ] 能口述 Q/K/V/softmax/causal mask
- [ ] 多头、残差、LayerNorm、FFN 全写过
- [ ] 绝对位置 + RoPE 都试过
- [ ] tinyshakespeare loss < 1.5
- [ ] 生成莎翁腔输出
- [ ] 至少 4 种解码策略实验过
- [ ] nanoGPT 阅读 + 注释
- [ ] LLMs-from-scratch 至少跑完前 5 章
- [ ] 推理优化 5 个名词能解释
- [ ] 5000 字博客发布

---

## 18. 第 07 章总结

到这里，你已经：

- **W01** 知道 LLM 是什么、市场上有什么、怎么调 API
- **W02** 能用 prompt 精确控制模型
- **W03** 让模型动手做事（Tool Use）
- **W04** 给模型外接知识库（RAG）
- **W05** 能科学地评测 LLM 应用
- **W06** 从零实现了 Transformer，建立了终极心智模型

下一章 **08 Agent 系统** 将把这一切组合起来：让 LLM **自己规划、自己用工具、自己反思** —— 而你已经站在了最坚实的基础上。
