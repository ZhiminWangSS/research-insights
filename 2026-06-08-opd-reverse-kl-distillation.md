# OPD：基于 On-Policy 轨迹的蒸馏

## OPD 的定义

OPD, On-Policy Distillation, 将蒸馏位置从 teacher/data trajectory 转到 student 自己生成的 trajectory 上。

给定 prompt \(x\)，student 先采样生成：

$$
y \sim \pi_\theta(\cdot \mid x)
$$

对每个 student-generated prefix：

$$
s_t = (x, y_{<t})
$$

再让 teacher 在同一 prefix 上给出 next-token distribution：

$$
\pi_T(\cdot \mid s_t)
$$

一个常见的 OPD 目标可以写成：

$$
\mathcal{L}_{\mathrm{OPD}}
=
\mathbb{E}_{y \sim \pi_\theta}
\sum_t
D_{\mathrm{KL}}
\left(
\pi_\theta(\cdot \mid s_t)
\|
\pi_T(\cdot \mid s_t)
\right)
$$

这里的关键是 **on-policy state distribution**。训练信号来自 student 实际会到达的 prefix，而不是 teacher 或离线数据中的 prefix。

如果使用 full-vocab KL，那么在某个固定 prefix 上，teacher 对整个 vocabulary 的概率都会参与计算。OPD 的限制不在于 token 空间是否可见，而在于 state/prefix 空间是否被 student 访问到。

## 逆向 KL 的设计动机

OPD 通常使用 student 到 teacher 的 KL：

$$
D_{\mathrm{KL}}(\pi_\theta \| \pi_T)
=
\sum_y
\pi_\theta(y \mid s)
\left[
\log \pi_\theta(y \mid s)
-
\log \pi_T(y \mid s)
\right]
$$

这个目标由 student distribution 加权。也就是说，loss 主要作用在 student 当前愿意生成的 token 上。

这带来一个直接后果：reverse KL 更关注 student 自身高概率区域是否被 teacher 支持，而不是强制 student 覆盖 teacher 的所有高概率模式。

相比之下，forward KL 为：

$$
D_{\mathrm{KL}}(\pi_T \| \pi_\theta)
=
\sum_y
\pi_T(y \mid s)
\left[
\log \pi_T(y \mid s)
-
\log \pi_\theta(y \mid s)
\right]
$$

它由 teacher distribution 加权，因此 teacher 认为重要的 token 会被更强地监督。

因此，两者的差异不是简单的符号顺序，而是优化时由谁决定权重：

| KL 方向 | 加权分布 | 行为倾向 |
|---|---|---|
| \(D_{\mathrm{KL}}(\pi_T \| \pi_\theta)\) | teacher | mode-covering |
| \(D_{\mathrm{KL}}(\pi_\theta \| \pi_T)\) | student | mode-seeking |

在 OPD 中，reverse KL 的作用更像是在 student 已经进入的区域内做约束：保留 student 的 on-policy 分布，同时减少它相对 teacher 的偏移。

## 与普通 Distillation 的区别

普通 distillation 通常在 teacher 或离线数据提供的 prefix 上训练 student。目标常写作：

$$
\mathcal{L}_{\mathrm{KD}}
=
\mathbb{E}_{s \sim \mathcal{D}}
D_{\mathrm{KL}}
\left(
\pi_T(\cdot \mid s)
\|
\pi_\theta(\cdot \mid s)
\right)
$$

其中 \(\mathcal{D}\) 可以来自人工数据、teacher 采样，或固定训练集。

这种设置强调让 student 覆盖 teacher 在这些 prefix 上的输出分布。它更接近 supervised learning：teacher 给出软标签，student 拟合这些软标签。

OPD 改变了两个部分：

1. prefix 来自 student 自己，而不是固定数据或 teacher trajectory；
2. KL 通常采用 reverse direction，由 student distribution 加权。

因此 OPD 的目标不是让 student 完整复刻 teacher 的行为分布，而是在 student 自己会到达的状态上，用 teacher 修正 student 的局部分布。

简化对比：

| 方法 | Prefix 来源 | KL 方向 | 主要作用 |
|---|---|---|---|
| 普通蒸馏 | data / teacher | \(D_{\mathrm{KL}}(\pi_T \| \pi_\theta)\) | 覆盖 teacher 分布 |
| OPD | student on-policy | \(D_{\mathrm{KL}}(\pi_\theta \| \pi_T)\) | 修正 student 自身轨迹上的偏移 |

核心区别可以概括为：

普通 distillation 让 student 学 teacher 在给定数据分布下的行为。  
OPD 让 student 在自己的生成分布下，向 teacher 保持局部对齐。

## 参考

- Kevin Lu et al., Thinking Machines Lab, [On-Policy Distillation](https://thinkingmachines.ai/blog/on-policy-distillation/), 2025-10-27.
