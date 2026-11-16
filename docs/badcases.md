# Boundary badcases

These are the three claims that v1.1 still gets wrong on the 20-claim set. They are kept intentionally: they show where the **annotation rubric**, not retrieval, is the bottleneck.

| # | Claim | Human label | v1.1 label | Boundary |
| --- | --- | --- | --- | --- |
| 4 | 鸡尾酒排序对序列两端局部有序的数据更有效 | `Supported` | `Weak` | Citation marker vs cited content |
| 7 | 小规模/几乎有序数据上，插入排序往往比快速排序更高效 | `Supported` | `Weak` | Citation marker vs cited content |
| 12 | Timsort 会识别并合并自然升序/降序游程，在部分有序数据上接近线性 | `Weak` | `Unsupported` | General algorithm fact vs claim about this paper's algorithm |

## 1. Citation marker vs cited content (#4, #7)

The paper contains the claim and a citation marker (`[6]` / `[9]`), but the retrieved snippet does not include the cited article's full text. The v1.1 prompt asks for "checkable evidence"; the model interprets that as needing the cited content and returns `Weak`.

The human annotation treats **a citation marker plus a matching paper statement** as sufficient for `Supported`. This is a real product decision: should the auditor verify the cited source itself, or only verify that the paper cites it?

**Possible v1.2 rule:** if the citation number is in range and the paper statement matches, treat it as citation support; do not require the full cited text to appear in the retrieved snippet.

## 2. General algorithm fact vs this paper's algorithm (#12)

Timsort is a general algorithm. The paper makes a qualitative statement about it without a direct citation. The v1.1 prompt has a rule that quantitative complexity claims without measurement/reference should be `Unsupported`; the model applies that rule to "接近线性" and returns `Unsupported`.

The human label is `Weak`: the paper has an author statement, but no direct citation or experiment, so it is a weak claim rather than a contradiction.

**Possible v1.2 rule:** apply the strict "quantitative claim needs measurement" rule only to claims about **this paper's own algorithm**; treat general algorithm background statements without citations as `Weak`.

## Why these are kept

The project could have been tuned to 20/20 by adding rules for these exact claims. That would make the evaluation less useful. The 17/20 result plus these three documented boundaries is a more honest signal of where the system is reliable and where the rubric is still ambiguous.
