# 学习React(3) 管理State

## 对输入作出反应

React使用声明式的方法操作UI,我们不是直接操作UI的单个部分,而是通过描述UI可能存在的不同的状态,并根据用户的输入在这些状态之间切换

在React中，我们不会直接操作UI，这意味着您不会直接启用、禁用、显示或隐藏组件。相反，我们只需要**声明我们想要显示的内容，**React会弄清楚如何更新用户界面。

## 选择State结构

良好的State结构可以在我们维护或修改一个组件的时候提供很大的便利.当我们在编写具有一些状态的组件时,虽然使用次优状态的结构并不会导致错误产生,但还是有几点原则可以帮助我们对State的结构作出更好的选择:

* 组合相关的状态
* 避免状态上的矛盾
* 避免冗余状态
* 避免状态出现重复
* 避免深层嵌套状态

### 组合相关的状态

如果我们总是同时更新两个或多个状态变量时,可以优先考虑将它们组合成单个状态变量.

例如下面这个组件,我们在移动光标的时候,x与y总是要同时移动,所以我们可以将它们整合成一个状态变量,这样我们就不会忘记始终保持它们的同步

```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}

```

需要注意的是,我们更新的状态变量是一个对象,如果没有显示地复制其它字段,我们就不能只更新其中的某个字段.例如我们无法在上例中执行 setPosition({ x: 10 }) 因为它根本就没有y属性.

### 避免状态上的矛盾

当我们状态上有某些变量存在矛盾:即它们的某些状态不会共存.当我们只更新其中的某个变量而忘记更新其它的变量时,这时候程序就可能会出现我们意想不到的结果,并且这个错误是比较难以发现的.

### 避免冗余状态

如果我们能够在已有的信息计算得到我们想要的值,我们就不要将这个值再使用状态变量管理了

### 避免状态中的重复

当相同的数据在多个状态变量之间或嵌套对象中复制时，很难保持它们的同步。我们需要尽可能减少重复。

```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>What's your travel snack?</h2> 
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}

```

这个例子中把items的某一项作为另外一个状态变量.如果我们首先单击项目上的“选择”，*然后*对其进行编辑，**输入会更新，但底部的标签不会反映编辑。**这是因为有重复的状态，并且忘记更新`selectedItem`。

当然我们可以更新selectedItem,但是更简单的解决方式是删除重复项:可以用id来标志选中的item,在选中的时候更新选中的id,并在最后一段中用 items[selectedId] 来展示, 这样就能达到我们预期的效果了.

### 避免使用深度嵌套的状态

假设现在有一个地图项目,上面的地点与附属地的关系可能会引导你使用嵌套的状态.但是之前我们有说明,如果要对某个数据进行操作,就必须显式地声明其它状态.这样一来更新这个地图数据就会显得十分繁琐了(你需要在更新时声明其它嵌套上层的数据).

我们可以考虑将其扁平化,例如之前如果我们使用对象内嵌套对象数组的方式来存储这个地图数据,我们可以改为让每个地方持有其子位置*ID*的数组，而不是树状结构.这样的数据更新起来会更加便捷.
