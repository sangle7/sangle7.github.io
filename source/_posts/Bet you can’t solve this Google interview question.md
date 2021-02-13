---
title: 打赌你无法解决这个Google面试题
date: 2020-08-21 10:14:44
tags: 
  - 翻译
---

> 原文链接： https://medium.com/free-code-camp/bet-you-cant-solve-this-google-interview-question-4a6e5a4dc8ee

> 将棘手的问题分解为小问题

## TechLead 的问题

在这个问题中，TechLead 要求我们观察下面的网格，并获取所有颜色相同的最大连续块的数目。

![](https://miro.medium.com/max/1400/1*uake3YsUeGzRiPGEt3RuSA.png)

![](https://miro.medium.com/max/1400/1*WmODExHDGW4i23Cwg6-27Q.png)


当我听到他的问题并看到图片时，我在想：“天哪，我必须做一些2D图像建模才能弄清楚这一点”，而这在面试中是几乎不可能的。

但是在他进一步解释之后，我发现事实并非如此。我们需要处理的是图像的数据，而不需要解析图像并转化为数据。


## 数据模型

在编写代码之前，我们需要定义数据模型。

在我们的案例中，TechLead 给了我们这些限制：
- 我们将其称为彩色正方形或“节点”的概念
- 我们的数据集中有1万个节点
- 节点分为列和行（2D）
- 列和行的数量可以不均匀
- 节点具有颜色和某种表示邻接的方式

<!-- more -->

我们还可以从数据中获取更多信息：
- 没有两个节点会重叠
- 节点永远不会与它们自己相邻
- 节点永远不会有重复的邻接关系
- 位于侧面和角落的节点将分别缺少一两个相邻关系

我们不知道的是：
- 行与列的比例
- 可能的颜色数
- 所有节点只有一种颜色的机会
- 颜色的粗略分布

职级越高，您将要问的问题越多。这也不是经验问题。虽然这样做有帮助，但是如果您不能挑出未知数，也不会使您变得更好。

我并没有指望大多数人都能发现这些未知项。直到我开始研究算法时，我也不全都知道。未知项需要时间才能弄清楚。需要与人进行大量讨论并来回查阅题目。

看他的图像，似乎分布是随机的。他只使用了3种颜色，并且从不说其他任何东西，所以我们也一样。我们还将假设所有颜色都可能相同。

由于它可能会破坏我们的算法，因此我假设我们正在使用 100x100 的网格。这样，我们不必处理1行和 10K 列的极端情况。

在典型的情况下，我会在问题提出的最初几个小时内发现所有这些问题。这就是 TechLead 真正关心的问题。你讲直接开始编码？还是会先分析题目找出问题所在？

## 创建数据模型

我们数据的基本模块：
 - 颜色
 - ID
 - X
 - Y

ID是干什么的？它是每个网格的唯一标志。

由于是在网格中整理数据，因此我假设我们将使用X和Y值描述数据。 仅使用这些属性，我就能生成一些HTML，以确保我们生成的内容就像 TechLead 给我们的一样。

我们使用绝对定位来完成：

![](https://miro.medium.com/max/800/1*waDXrVIZHjYIHAEVWin54w.png)

也适用于更大的数据集：

![](https://miro.medium.com/max/1400/1*9vrgCN2D53GLsxAHB_Xx4Q.png)

这里是代码：

```javascript
const generateNodes = ({
  numberOfColumns,
  numberOfRows,
}) => (
  Array(
    numberOfColumns
    * numberOfRows
  )
  .fill()
  .map((
    item,
    index,
  ) => ({
    colorId: (
      Math
      .floor(
        Math.random() * 3
      )
    ),
    id: index,
    x: index % numberOfColumns,
    y: Math.floor(index / numberOfColumns),
  }))
)
```

我们取出列和行，从项目数量中创建一维数组，然后根据该数据生成节点。

我使用的是 `colorId`，而不是 `color`。 首先，因为随机化比较容易。 其次，我们通常必须自己检查颜色值。

尽管他从未明确声明，但他只使用了3种颜色值。 我也将数据集限制为3种颜色。 只需知道它可能是数百种颜色，并且最终的算法就无需更改。

举一个简单的例子，这里是2x2节点列表：

```javascript
[
  { colorId: 2, id: 0, x: 0, y: 0 },
  { colorId: 1, id: 1, x: 1, y: 0 },
  { colorId: 0, id: 2, x: 0, y: 1 },
  { colorId: 1, id: 3, x: 1, y: 1 },
]
```

## 数据处理

无论我们要使用哪种方法，我们都想知道每个节点的邻接关系。 X和Y值不会减少。

因此，给定X和Y，我们需要弄清楚如何找到相邻的X和Y值。 这很简单。 我们只需在X和Y上找到正负1的节点即可。

我为此逻辑编写了一个辅助函数：

```javascript
const getNodeAtLocation = ({
  nodes,
  x: requiredX,
  y: requiredY,
}) => (
  (
    nodes
    .find(({
      x,
      y,
    }) => (
      x === requiredX
      && y === requiredY
    ))
    || {}
  )
  .id
)
```
我们生成节点的方式实际上是一种数学方法，用来找出相邻节点的ID。 相反，我假设节点将以随机顺序进入我们的系统。

我通过第二遍运行所有节点以添加邻接关系：

```javascript
const addAdjacencies = (
  nodes,
) => (
  nodes
  .map(({
    colorId,
    id,
    x,
    y,
  }) => ({
    color: colors[colorId],
    eastId: (
      getNodeAtLocation({
        nodes,
        x: x + 1,
        y,
      })
    ),
    id,
    northId: (
      getNodeAtLocation({
        nodes,
        x,
        y: y - 1,
      })
    ),
    southId: (
      getNodeAtLocation({
        nodes,
        x,
        y: y + 1,
      })
    ),
    westId: (
      getNodeAtLocation({
        nodes,
        x: x - 1,
        y,
      })
    ),
  }))
  .map(({
    color,
    id,
    eastId,
    northId,
    southId,
    westId,
  }) => ({
    adjacentIds: (
      [
        eastId,
        northId,
        southId,
        westId,
      ]
      .filter((
        adjacentId,
      ) => (
        adjacentId !== undefined
      ))
    ),
    color,
    id,
  }))
)
```

我避免在此预处理程序代码中进行任何不必要的优化。 它不会影响我们的最终性能数据，只会帮助简化算法。

我继续将 `colorId` 更改为另一种颜色。 对于我们的算法而言，这完全没有必要，但我想使其更易于可视化。

我们为每组相邻的X和Y值调用 `getNodeAtLocation`并找到我们的`northId`，`eastId`，`southId`和`westId`。 不再传递X和Y值，因为它们不再需要。

在获得基本ID后，我们将其转换为一个仅包含值的ID数组。 这样，如果我们有边角，就不必担心检查这些ID是否为空。 它还允许我们循环一个数组，而不是手动注释算法中的每个基本ID。

这是另一个2x2的示例，该示例使用通过 `addAdjacencies` 运行的一组新节点：

```javascript
[
  { adjacentIds: [ 1, 2 ], color: 'red',  id: 0 },
  { adjacentIds: [ 3, 0 ], color: 'grey', id: 1 },
  { adjacentIds: [ 3, 0 ], color: 'blue', id: 2 },
  { adjacentIds: [ 1, 2 ], color: 'blue', id: 3 },
]
```

## 处理优化

我想大大简化本文的算法，因此添加了另一个优化过程。 这将删除与当前节点颜色不匹配的相邻ID。
重写`addAdjacencies`函数之后，现在是这样的：

```javascript
const addAdjacencies = (
  nodes,
) => (
  nodes
  .map(({
    colorId,
    id,
    x,
    y,
  }) => ({
    adjacentIds: (
      nodes
      .filter(({
        x: adjacentX,
        y: adjacentY,
      }) => (
        adjacentX === x + 1
        && adjacentY === y
        || (
          adjacentX === x - 1
          && adjacentY === y
        )
        || (
          adjacentX === x
          && adjacentY === y + 1
        )
        || (
          adjacentX === x
          && adjacentY === y - 1
        )
      ))
      .filter(({
        colorId: adjacentColorId,
      }) => (
        adjacentColorId
        === colorId
      ))
      .map(({
        id,
      }) => (
        id
      ))
    ),
    color: colors[colorId],
    id,
  }))
  .filter(({
    adjacentIds,
  }) => (
    adjacentIds
    .length > 0
  ))
)
```

我减少了 `addAdjacencies` 的代码量，同时添加了更多功能。

通过删除颜色不匹配的节点，我们的算法可以100％确保 `adjacentIds` 中的ID都是连续的节点。

最后，我删除了没有相同颜色邻接关系的所有节点。 这进一步简化了我们的算法，并且将总节点缩减为仅关心的节点。

## 错误的方式—递归

TechLead 表示我们无法递归执行此算法，因为我们遇到了堆栈溢出的情况。

尽管他部分正确，但有几种方法可以缓解此问题。 迭代或使用尾递归。 我们将以迭代示例为例，但是JavaScript不再具有尾部递归作为本机语言功能。

虽然我们仍然可以在JavaScript中模拟尾部递归，但我们将保持这种简单性，而是创建一个典型的递归函数。

在编写代码之前，我们需要弄清楚我们的算法。 对于递归，使用深度优先搜索是有意义的。 不必担心知道计算机科学术语。 当我向他展示我想出的不同解决方案时，一位同事说过。

### 算法

我们将从节点开始，并尽我们所能直到到达终点。 然后，我们将返回并采用下一个分支路径，直到我们扫描了整个连续块。

这就是其中的一部分。 我们还必须跟踪我们去过的地方以及最大的连续街区的长度。

我所做的就是将我们的功能分为两部分。 一个将拥有最大的列表和先前扫描的ID，同时至少每个节点循环一次。 另一个将从未扫描的根节点开始，然后进行深度优先遍历。

函数长这样：

```javascript
const getContiguousIds = ({
  contiguousIds = [],
  node,
  nodes,
}) => (
  node
  .adjacentIds
  .reduce(
    (
      contiguousIds,
      adjacentId,
    ) => (
      contiguousIds
      .includes(adjacentId)
      ? contiguousIds
      : (
        getContiguousIds({
          contiguousIds,
          node: (
            nodes
            .find(({
              id,
            }) => (
              id
              === adjacentId
            ))
          ),
          nodes,
        })
      )
    ),
    (
      contiguousIds
      .concat(
        node
        .id
      )
    ),
  )
)

const getLargestContiguousNodes = (
  nodes,
) => (
  nodes
  .reduce(
    (
      prevState,
      node,
    ) => {
      if (
        prevState
        .scannedIds
        .includes(node.id)
      ) {
        return prevState
      }

      const contiguousIds = (
        getContiguousIds({
          node,
          nodes,
        })
      )

      const {
        largestContiguousIds,
        scannedIds,
      } = prevState

      return {
        largestContiguousIds: (
          contiguousIds.length
          > largestContiguousIds.length
          ? contiguousIds
          : largestContiguousIds
        ),
        scannedIds: (
          scannedIds
          .concat(contiguousIds)
        ),
      }
    },
    {
      largestContiguousIds: [],
      scannedIds: [],
    },
  )
  .largestContiguousIds
)
```
疯了吧？这个代码简直太长了。

接下来我们一步步来缩减代码

### 递归函数

`getContiguousIds` 是我们的递归函数，每个节点调用一次。 每次都会返回更新的连续节点列表。

此功能只有一个条件：列表中是否已存在我们的节点？ 如果不是，请再次调用

`getContiguousIds`。 返回时，我们将获得一个更新的连续节点列表，该列表将返回到化简器，并用作下一个相邻ID的状态。

您可能想知道我们在哪里向contiguousIds添加值。 当我们将当前节点连接到contiguousIds上时，就会发生这种情况。 每次进一步递归时，我们都确保在循环其相邻ID之前将当前节点添加到contiguousId列表中。

始终添加当前节点可确保我们不会无限递归。

### 循环

此功能的后半部分还将遍历每个节点一次。

我们在递归函数周围使用了reducer。 此人检查我们的代码是否已被扫描。 如果是这样，请继续循环，直到找到一个尚未存在的节点，或者直到退出循环为止。

如果尚未扫描我们的节点，请调用getContiguousIds并等待完成。 这是同步的，但可能需要一些时间。

返回带有contiguousIds的列表后，请对照largestantContiguousIds列表进行检查。 如果更大，则存储该值。

同时，我们将这些contiguousId添加到我们的scandIds列表中，以标记我们去过的地方。
当您看到所有布局时，这非常简单。

### 执行

即使有1万个节点+3种随机颜色，也不会遇到堆栈溢出问题。 但如果只有一种颜色，那么将会将遇到堆栈溢出的情况。 那是因为我们的递归函数要进行1万次递归。

## 顺序迭代


由于内存大于函数调用堆栈，因此我的下一个想法是在单个循环中完成整个操作。

我们将跟踪节点列表列表。 我们将继续添加它们并将它们链接在一起，直到脱离循环为止。

此方法要求我们将所有可能的节点列表保留在内存中，直到完成循环为止。 在递归示例中，我们仅在内存中保留了最大的列表。

```javascript
const getLargestContiguousNodes = (
  nodes,
) => (
  nodes
  .reduce(
    (
      contiguousIdsList,
      {
        adjacentIds,
        id,
      },
    ) => {
      const linkedContiguousIds = (
        contiguousIdsList
        .reduce(
          (
            linkedContiguousIds,
            contiguousIds,
          ) => (
            contiguousIds
            .has(id)
            ? (
              linkedContiguousIds
              .add(contiguousIds)
            )
            : linkedContiguousIds
          ),
          new Set(),
        )
      )

      return (
        linkedContiguousIds
        .size > 0
        ? (
          contiguousIdsList
          .filter((
            contiguousIds,
          ) => (
            !(
              linkedContiguousIds
              .has(contiguousIds)
            )
          ))
          .concat(
            Array
            .from(linkedContiguousIds)
            .reduce(
              (
                linkedContiguousIds,
                contiguousIds,
              ) => (
                new Set([
                  ...linkedContiguousIds,
                  ...contiguousIds,
                ])
              ),
              new Set(adjacentIds),
            )
          )
        )
        : (
          contiguousIdsList
          .concat(
            new Set([
              ...adjacentIds,
              id,
            ])
          )
        )
      )
    },
    [new Set()],
  )
  .reduce((
    largestContiguousIds = [],
    contiguousIds,
  ) => (
    contiguousIds.size
    > largestContiguousIds.size
    ? contiguousIds
    : largestContiguousIds
  ))
)
```

另一个疯狂的。 让我们从顶部开始进行分解。 我们将每个节点循环一次。 但是现在我们必须检查我们的id是否在节点列表的列表中：contiguousIdsList。

如果它不在任何contiguousId列表中，我们将添加它及其相邻ID。 这样，在循环时，其他东西将链接到它。

如果我们的节点在其中一个列表中，则可能其中有很多。 我们希望将所有这些链接在一起，并从contiguousIdsList中删除未链接的链接。

而已。

提出节点列表列表后，我们检查最大的节点列表，然后完成。

### 执行

与递归版本不同，当所有10K项都具有相同的颜色时，该函数确实会完成。

除此之外，它非常慢； 比我最初预期的要慢得多。 我忘了在效果评估中考虑循环列表列表，这显然会对性能产生影响。

### 随机迭代

我想将方法学放在递归方法的后面，并迭代地应用它。
我花了一整夜的时间试图记住如何动态更改循环中的索引，然后想起了while（true）。 自从我写了传统的循环已经很久了，我完全忘记了它。
现在我有了武器，我就开始进攻了。 由于我花了很多时间试图加快可观察版本的速度（稍后会详细介绍），因此我决定将数据懒散地进行老式修改。
该算法的目标是精确击中每个节点一次，并且仅存储最大的连续块：


```javascript
const getLargestContiguousNodes = (
  nodes,
) => {
  let contiguousIds = []
  let largestContiguousIds = []
  let queuedIds = []
  let remainingNodesIndex = 0

  let remainingNodes = (
    nodes
    .slice()
  )

  while (remainingNodesIndex < remainingNodes.length) {
    const [node] = (
      remainingNodes
      .splice(
        remainingNodesIndex,
        1,
      )
    )

    const {
      adjacentIds,
      id,
    } = node

    contiguousIds
    .push(id)

    if (
      adjacentIds
      .length > 0
    ) {
      queuedIds
      .push(...adjacentIds)
    }

    if (
      queuedIds
      .length > 0
    ) {
      do {
        const queuedId = (
          queuedIds
          .shift()
        )

        remainingNodesIndex = (
          remainingNodes
          .findIndex(({
            id,
          }) => (
            id === queuedId
          ))
        )
      }
      while (
        queuedIds.length > 0
        && remainingNodesIndex === -1
      )
    }

    if (
      queuedIds.length === 0
      && remainingNodesIndex === -1
    ) {
      if (
        contiguousIds.length
        > largestContiguousIds.length
      ) {
        largestContiguousIds = contiguousIds
      }

      contiguousIds = []
      remainingNodesIndex = 0

      if (
        remainingNodes
        .length === 0
      ) {
        break
      }
    }
  }

  return largestContiguousIds
}

module.exports = getLargestContiguousNodesIterative2
```
即使我像大多数人一样写了这篇文章，但到目前为止，它的可读性最差。 我什至无法告诉您这是怎么回事，除非自己先自上而下地进行。

我们没有添加到以前扫描的ID列表中，而是从剩余的Nodes数组中拼接出了值。

懒！ 我永远不会建议自己这样做，但是我在创建这些样本的最后阶段想尝试一些不同的尝试。

## 性能对比

### 随机颜色

| 耗时      | 方法                      |
| --------- | --------------------------- |
| 229.481ms | Recursive                   |
| 272.303ms | Iterative Random            |
| 323.011ms | Iterative Sequential        |
| 391.582ms | Redux-Observable Concurrent |
| 686.198ms | Redux-Observable Random     |
| 807.839ms | Redux-Observable Sequential |

### 单一颜色

| 耗时       | 方法                      |
| ---------- | --------------------------- |
| 1061.028ms | Iterative Random            |
| 1462.143ms | Redux-Observable Random     |
| 1840.668ms | Redux-Observable Sequential |
| 2541.227ms | Iterative Sequential        |