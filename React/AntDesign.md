> <h1 id=""></h1>
- [**AntDesign 组件总览**](https://ant.design/components/tag-cn)
- [**阿里开源ReactUI组件库AntDesign**](#阿里开源ReactUI组件库AntDesign)
	- [Table组件使用](#Table组件使用)
		- [表单属性](#表单属性)
		- [pagination、expandable属性使用](#pagination、expandable属性使用)
			- [expandable中常用属性：columnWidth、expandIcon](#expandable中常用属性：columnWidth、expandIcon)
			- [表中行展开细节](#表中行展开细节)
- [**ProTable列表拖拽排序 ‌**](#ProTable列表拖拽排序)
- [下拉组件Select](#下拉组件Select)
	- [onChange函数](#onChange函数) 
	- [options数据源，加入数据](#options数据源，加入数据) 
	- [下拉提示文案placeholder](#下拉提示文案placeholder)
- [加载指示器组件Spin](#加载指示器组件Spin)
- [模态框组件-Modal](#模态框组件-Modal)
- [弹框提示组件message](#弹框提示组件message)
- [**Form表单**](#Form表单)
	- [Form.Item属性详解](#Form.Item属性详解)
	- [默认值initialValue](#默认值initialValue)   
	- [rules校验规则详解](#rules校验规则详解)
	- [Form表单基本属性讲解](#Form表单基本属性讲解)
- [Button组件](#Button组件)
- [**Upload图片上传组件**](#Upload图片上传组件)
- [Menu组件做侧边栏菜单](#Menu组件做侧边栏菜单)
- [加载组件Spin](#加载组件Spin)
- [骨架屏Skeleton组件](#骨架屏Skeleton组件)
- [标签组件Tag](#标签组件Tag)
- [Radio.Group单选按钮](#Radio.Group单选按钮)



<br/><br/><br/>

***
<br/>

> <h1 id= "阿里开源ReactUI组件库AntDesign">阿里开源ReactUI组件库AntDesign</h1>

Ant Design（简称 AntD）是阿里开源的一个优秀的 React UI 组件库。

***
<br/><br/><br/>
> <h2 id="Table组件使用">Table组件使用</h2>

 `<Table />` 是其中的**表格组件**，用于显示结构化的数据，比如列表、数据管理页、控制台等。

它支持：

* 分页（pagination）
* 排序（sorter）
* 过滤（filters）
* 自定义渲染（render）
* 可展开行、可选择行等复杂功能

---
**使用方式示例**

- **1.导入 AntD：**

```tsx
import React from 'react';
import { Table } from 'antd';
```

<br/>

- **2.准备数据 `dataSource` 和列配置 `columns`：**

```tsx
const dataSource = [
  {
    key: '1',
    name: '产品A',
    capabilities: ['蓝牙', 'Wi-Fi'],
  },
  {
    key: '2',
    name: '产品B',
    capabilities: ['以太网'],
  },
];
```

```tsx
const columns = [
  {
    title: '产品名称',
    dataIndex: 'name',
    key: 'name',
  },
  {
    title: '功能列表',
    dataIndex: 'capabilities',
    key: 'capabilities',
    render: (_, record) => {
      return <span>{record.capabilities.join(', ')}</span>;
    },
  },
];
```

<br/>

- **3.渲染 Table：**

```tsx
const MyTable = () => {
  return <Table dataSource={dataSource} columns={columns} />;
};
```

<br/>

- ✅ **效果截图（描述）**

最终效果是一个两行两列的表格：

| 产品名称 | 功能列表      |
| ---- | --------- |
| 产品A  | 蓝牙, Wi-Fi |
| 产品B  | 以太网       |

---
<br/>

- **进阶样式与交互（点击事件 + 类名）**

```tsx
const columns = [
  {
    title: '产品名称',
    dataIndex: 'name',
    key: 'name',
  },
  {
    title: '功能列表',
    dataIndex: 'capabilities',
    key: 'capabilities',
    className: 'can-click',
    render: (_, record) => {
      return (
        <a onClick={() => alert(`点击了：${record.name}`)}>
          {record.capabilities.join(', ')}
        </a>
      );
    },
  },
];
```

然后定义 CSS：

```css
.can-click {
  color: blue;
  cursor: pointer;
}
```



***
<br/><br/><br/>
> <h2 id="表单属性">表单属性</h2>

```tsx
<Form
  {...layout}
  form={form}
  onFinish={handleFinish}
  onFinishFailed={handleFinishFailed}
>
```

| 属性               | 类型     | 说明                          |
| ---------------- | ------ | --------------------------- |
| `{...layout}`    | Object | 表单布局控制，设置标签列宽与内容列宽          |
| `form`           | 实例     | 表单控制实例（来自 `Form.useForm()`） |
| `onFinish`       | 函数     | 表单提交成功时触发（校验通过）             |
| `onFinishFailed` | 函数     | 表单提交失败（校验不通过）时触发            |




***
<br/><br/>
> <h3 id="pagination、expandable属性使用">pagination、expandable属性使用</h3>

`pagination` 和 `expandable` 是两个非常实用的功能配置项，分别用于控制 **分页** 和 **展开行**。

| 属性名          | 作用     | 是否常用      | 示例用途              |
| ------------ | ------ | --------- | ----------------- |
| `pagination` | 控制分页   | ✅ 非常常用    | 数据量多时分页加载         |
| `expandable` | 控制可展开行 | ✅ 常用于详情展示 | 展示附加字段、嵌套表格、图片预览等 |


---
<br/>

**`pagination` —— 表格分页功能**

- **🔹 作用：**
	- 用于控制表格的分页行为，比如：
		* 每页显示多少条数据
		* 当前页码
		* 是否显示分页器
		* 翻页回调等

 **🧩 用法示例：**

```tsx
<Table
  dataSource={data}
  columns={columns}
  pagination={{
    pageSize: 10,         // 每页10条
    current: 1,           // 当前页是第1页
    onChange: (page, pageSize) => {
      console.log(`跳转到第${page}页, 每页${pageSize}条`);
    }
  }}
/>
```

<br/>

**✅ 效果：** 在表格底部会出现分页器，比如：

```
<< < 1 2 3 4 5 > >>
```

---
<br/>

- **`expandable` —— 可展开行**
	- 🔹 作用：
		- 让每一行可以“展开”来显示更多内容（比如子表格、详情说明等），通常用于嵌套数据或附加信息展示。

**🧩 用法示例：**

```tsx
<Table
  dataSource={data}
  columns={columns}
  expandable={{
    expandedRowRender: (record) => (
      <p style={{ margin: 0 }}>
        详细信息：{record.description}
      </p>
    ),
    rowExpandable: (record) => !!record.description,  // 只有有 description 的行才能展开
  }}
/>
```

<br/>

**效果：** 表格左边每行出现一个 `▸` 图标（点击后变 `▾`），点击会展开一个新行显示更多信息：

```
▸ 产品A    | Wi-Fi
▸ 产品B    | 蓝牙

点击后：

▾ 产品B    | 蓝牙
  详细信息：这是产品B的蓝牙说明
```

---
<br/>

**二者可组合使用**

```tsx
<Table
  dataSource={data}
  columns={columns}
  pagination={{ pageSize: 5 }}
  expandable={{
    expandedRowRender: (record) => <div>{record.details}</div>,
  }}
/>
```

这样每页只显示 5 条记录，并且每条记录都可以展开查看详情。


<br/><br/>
> <h3 id="expandable中常用属性：columnWidth、expandIcon">expandable中常用属性：columnWidth、expandIcon</h3>

**`columnWidth,expandIcon`**是 Ant Design 表格（`<Table />`）中的 `expandable` 常用两个属性，主要控制 **可展开行**的行为和样式。

---
<br/>

**`columnWidth: 0`**，作用： 设置“展开图标列”的宽度为 0（**隐藏该列**）。


**📍使用场景：**

当你想自己用别的方式（比如自定义图标、按钮）来控制展开/收起，而不想使用默认的那一列带 `▸ / ▾` 图标时，可以设置：

```ts
expandable={{
  columnWidth: 0, // 不显示默认图标列
  expandedRowRender: (record) => <div>{record.details}</div>,
}}
```

> ⚠️ 如果你把 `columnWidth` 设置为 0，但又没有自定义 `expandIcon`，那用户将 **无法展开任何行**，因为默认图标也被隐藏了。

---
<br/>

**`expandIcon`**，作用：自定义每一行左侧的“展开图标”。

<br/>

- **📍使用场景：**

你不喜欢默认的 `▸ / ▾`，想换成自己的按钮、图标、链接等，并添加交互逻辑。

**🧩 示例：**

```tsx
import { Table, Button } from 'antd';

<Table
  columns={columns}
  dataSource={data}
  expandable={{
    expandedRowRender: (record) => <div>{record.details}</div>,
    columnWidth: 0,
    expandIcon: ({ expanded, onExpand, record }) => (
      <Button
        type="link"
        onClick={(e) => onExpand(record, e)}
      >
        {expanded ? '收起' : '展开'}
      </Button>
    ),
  }}
/>
```

<br/>

**✅ 效果：**

表格中每一行 **不再显示默认的展开图标列**，而是显示你自定义的“展开 / 收起”按钮。

---

**✅ 总结**

| 属性名              | 作用        | 是否常用                   | 示例用途         |
| ---------------- | --------- | ---------------------- | ------------ |
| `columnWidth: 0` | 隐藏默认展开图标列 | ⚠️ 需配合 `expandIcon` 使用 | 自定义图标样式      |
| `expandIcon`     | 自定义展开图标   | ✅ 常用                   | 替换成按钮、图标、文字等 |




***
<br/><br/><br/>
> <h2 id="表中行展开细节">表中行展开细节</h2>

`<Table>` 中的 `pagination` 和 `expandable` 是两个常用的功能属性，分别用于 **分页** 和 **可展开行**。

<br/>

 **✅ 一、`pagination`（分页）属性详解：** 控制表格分页行为 —— 显示分页器、切换页码、页面大小等。

<br/>

**Table的基本属性：**

```tsx
<Table
  dataSource={data}
  columns={columns}
  pagination={{
    current: 1,
    pageSize: 10,
    total: 100,
    showSizeChanger: true,
    showQuickJumper: true,
    onChange: (page, pageSize) => {
      // 切页回调
    },
  }}
/>
```

<br/> 

**🔍 属性说明：**

| 属性名               | 类型                         | 说明         |
| ----------------- | -------------------------- | ---------- |
| `current`         | `number`                   | 当前页码       |
| `pageSize`        | `number`                   | 每页显示条数     |
| `total`           | `number`                   | 总数据条数      |
| `showSizeChanger` | `boolean`                  | 是否允许切换每页条数 |
| `showQuickJumper` | `boolean`                  | 是否允许快速跳页   |
| `onChange`        | `(page, pageSize) => void` | 分页变化时触发    |

---
<br/>


**🛠 示例（分页 + 请求数据）：**

```tsx
const [data, setData] = useState([]);
const [pagination, setPagination] = useState({ current: 1, pageSize: 10, total: 0 });

const fetchData = async (page = 1, pageSize = 10) => {
  const res = await request(`/api/list?page=${page}&size=${pageSize}`);
  setData(res.list);
  setPagination({ current: page, pageSize, total: res.total });
};

useEffect(() => {
  fetchData(pagination.current, pagination.pageSize);
}, []);

<Table
  dataSource={data}
  columns={columns}
  pagination={{
    ...pagination,
    showSizeChanger: true,
    showQuickJumper: true,
    onChange: (page, pageSize) => {
      fetchData(page, pageSize);
    },
  }}
/>
```

<br/>

 **✅ 二、`expandable`（可展开行）属性：** 用于显示“可展开的子行内容”，例如：点击某一行可展开更多详情信息（嵌套视图、子表格、额外描述等）。


```tsx
<Table
  dataSource={data}
  columns={columns}
  expandable={{
    expandedRowRender: (record) => <p>{record.description}</p>,
    rowExpandable: (record) => record.expandable !== false,
  }}
/>
```

<br/>

 **🔍 属性说明：**

| 属性名                 | 类型                           | 说明             |
| ------------------- | ---------------------------- | -------------- |
| `expandedRowRender` | `(record) => ReactNode`      | 渲染展开行内容        |
| `rowExpandable`     | `(record) => boolean`        | 判断某一行是否允许展开    |
| `onExpand`          | `(expanded, record) => void` | 展开/收起时触发       |
| `expandedRowKeys`   | `string[] \| number[]`       | 控制当前展开的行（受控模式） |

---
<br/>

**🛠 示例（点击展开详情）：**

```tsx
const columns = [
  { title: '姓名', dataIndex: 'name' },
  { title: '年龄', dataIndex: 'age' },
];

const data = [
  { key: 1, name: '张三', age: 32, description: '张三的详细信息...' },
  { key: 2, name: '李四', age: 28, description: '李四的详细信息...' },
];

<Table
  dataSource={data}
  columns={columns}
  expandable={{
    expandedRowRender: (record) => (
      <div style={{ margin: 0, padding: 8 }}>{record.description}</div>
    ),
  }}
/>
```

---
<br/>

**🧩 高级示例：展开行中嵌套子表格**

```tsx
expandable={{
  expandedRowRender: (record) => (
    <Table
      columns={[{ title: '订单号', dataIndex: 'orderId' }]}
      dataSource={record.orders}
      pagination={false}
    />
  ),
  rowExpandable: (record) => record.orders?.length > 0,
}}
```

<br/>

 **✅ 三、分页 + 展开行组合使用**

你可以同时使用两个属性，它们互不冲突：

```tsx
<Table
  dataSource={data}
  columns={columns}
  pagination={paginationConfig}
  expandable={{
    expandedRowRender: (record) => <p>{record.description}</p>,
  }}
/>
```




<br/><br/><br/>

***
<br/>
> <h1 id="ProTable列表拖拽排序">ProTable列表拖拽排序</h1>

**列表的json数据结构如下：**

```js
{
  categoryCode: "ipc",
  categoryId: "1823258198548017154",
  categoryName: "摄像机",
  children: [
    {
      categoryCode: "ipc",
      categoryId: "1937718753170132993",
      categoryName: "云台摄像机"
    }
  ]
}
```


<br/><br/>

**完整代码如下：**

```jsx
import React, { useState, useMemo } from 'react';
import ProTable from '@ant-design/pro-table';
import { DndContext, closestCenter } from '@dnd-kit/core';
import {
  SortableContext,
  useSortable,
  verticalListSortingStrategy,
  arrayMove,
} from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';

/// 这个是自定义的拖拽行所在按钮，是必须的，否则无法拖拽
const DraggableRow = (props) => {
  const { record, className, style, children, ...restProps } = props;

  const { attributes, listeners, setNodeRef, transform, transition } = useSortable({
    id: record.categoryId, // 这个相当于没放的标记ID，是json数据的一个字段
  });

  const rowStyle = {
    ...style,
    transform: CSS.Transform.toString(transform),
    transition,
  };

  const enhancedChildren = React.Children.map(children, (child, index) => {
    if (index === 0) {
      return React.cloneElement(child, {
        children: (
          <span
            style={{ cursor: 'grab', userSelect: 'none' }}
            {...attributes}
            {...listeners}
          >
            ☰
          </span>
        ),
      });
    }
    return child;
  });

  return (
    <tr ref={setNodeRef} style={rowStyle} className={className} {...restProps}>
      {enhancedChildren}
    </tr>
  );
};


/// 显示的列表
export default function CategoryTreeTable() {
  const [isSorting, setIsSorting] = useState(false);
  const [expandedRowKeys, setExpandedRowKeys] = useState([]);

  const [dataSource, setDataSource] = useState([
    {
      categoryId: "1823258198548017154",
      categoryCode: "ipc",
      categoryName: "摄像机",
      children: [
        {
          categoryId: "1937718753170132993",
          categoryCode: "ipc",
          categoryName: "云台摄像机",
        },
        {
          categoryId: "1937718753170132994",
          categoryCode: "ipc",
          categoryName: "球形摄像机",
        },
      ],
    },
    {
      categoryId: "1823258198548017155",
      categoryCode: "ipc",
      categoryName: "门铃",
    },
  ]);

  // 👉 拍平成 parentId-aware 的数组
  const flatten = (list, parentId = null) => {
    return list.reduce((acc, item) => {
      acc.push({ ...item, parentId });
      if (item.children) {
        acc.push(...flatten(item.children, item.categoryId));
      }
      return acc;
    }, []);
  };

  const flatList = flatten(dataSource);

	// 拖拽事件
	const onDragEnd = ({ active, over }) => {
    if (!over || active.id === over.id) return;

    const oldItem = flatList.find(item => item.categoryId === active.id);
    const overItem = flatList.find(item => item.categoryId === over.id);

    if (!oldItem || !overItem || oldItem.parentId !== overItem.parentId) return;

    const parentId = oldItem.parentId;

    const updateList = (list) => {
      if (parentId === null) {
        const oldIndex = list.findIndex(i => i.categoryId === active.id);
        const newIndex = list.findIndex(i => i.categoryId === over.id);
        return arrayMove(list, oldIndex, newIndex);
      }

      return list.map(item => {
        if (item.categoryId === parentId && item.children) {
          const oldIndex = item.children.findIndex(i => i.categoryId === active.id);
          const newIndex = item.children.findIndex(i => i.categoryId === over.id);
          item.children = arrayMove(item.children, oldIndex, newIndex);
        }
        return item;
      });
    };

    setDataSource(updateList(dataSource));
  };

  const columns = [
    ...(isSorting
      ? [{
          title: '',
          dataIndex: 'drag',
          width: 40,
          render: () => <span>☰</span>,
        }]
      : []),
    {
      title: '分类名称',
      dataIndex: 'categoryName',
    },
  ];

  const tableDom = (
    <ProTable
      columns={columns}
      dataSource={dataSource}
      rowKey="categoryId"
      pagination={false}
      search={false}
      expandable={{
        columnIndex: isSorting ? 1 : 0, // 👈 控制展开图标列
        expandedRowKeys,
        onExpandedRowsChange: setExpandedRowKeys,
        childrenColumnName: 'children',
      }}
      onRow={(record) => ({ record })}
      components={isSorting ? { body: { row: DraggableRow } } : undefined}
      toolBarRender={() => [
        <button
          key="sort"
          onClick={() => {
            if (isSorting) {
              console.log('🚀 排序后的结构:', dataSource);
            }
            setIsSorting(!isSorting);
          }}
        >
          {isSorting ? '完成排序' : '排序'}
        </button>
      ]}
    />
  );

  return isSorting ? (
    <DndContext collisionDetection={closestCenter} onDragEnd={onDragEnd}>
      <SortableContext
        items={flatList.map(i => i.categoryId)}
        strategy={verticalListSortingStrategy}
      >
        {tableDom}
      </SortableContext>
    </DndContext>
  ) : tableDom;
}
```

**调用：**

```jsx
import DragSortCategoryTable from './components/DragSortCategoryTable';

const yourData = [/* category 数据 */];

<DragSortCategoryTable
  dataSource={yourData}
  onSave={(sortedData) => {
    console.log('最终排序数据', sortedData);
    // 可调用 API 保存到后端
  }}
/>
```

***
<br/><br/>

**上述代码进行解读：**


**✅ 1. `flatList = useMemo(() => flatten(data), [data])` 是干什么的？**

这个是 React 的性能优化写法。

**🔍 `useMemo` 作用**

它的作用是“记住计算结果”，只有当依赖项 `[data]` 发生变化时，才重新计算 `flatten(data)`。

这样做的好处是避免在每次渲染组件时都重复执行耗时操作，比如递归遍历 `data`，提高性能。

<br/>

 **🔍 假设你的 `data` 是如下结构：**

```js
[
  {
    categoryId: "1",
    categoryName: "摄像机",
    children: [
      {
        categoryId: "1-1",
        categoryName: "云台摄像机"
      },
      {
        categoryId: "1-2",
        categoryName: "智能摄像机"
      }
    ]
  },
  {
    categoryId: "2",
    categoryName: "门铃"
  }
]
```

<br/>

执行完 `flatList = flatten(data)` 后结果是：

```js
[
  { categoryId: "1", categoryName: "摄像机", parentId: null },
  { categoryId: "1-1", categoryName: "云台摄像机", parentId: "1" },
  { categoryId: "1-2", categoryName: "智能摄像机", parentId: "1" },
  { categoryId: "2", categoryName: "门铃", parentId: null }
]
```

<br/>

**✅ 2. `flatten` 函数是干嘛的？怎么理解？**

这是一个递归函数，用来把树形结构「拍平」为一维数组，并标记每一项的 `parentId`。

```js
const flatten = (list, parentId = null) => {
  return list.reduce((acc, item) => {
    acc.push({ ...item, parentId }); // 当前节点加入结果数组
    if (item.children) {
      acc.push(...flatten(item.children, item.categoryId)); // 子节点继续递归处理
    }
    return acc;
  }, []);
};
```

你可以理解为：把 `tree -> flat`，并记录每项属于哪个父节点。

<br/>

**✅ 3. `onDragEnd` 函数的参数哪里来的？做了什么？**

这个是由 `DndContext` 触发事件自动提供的：

```jsx
<DndContext onDragEnd={onDragEnd} />
```

当你拖拽完成时，`dnd-kit` 会自动传一个对象给 `onDragEnd`，这个对象中包含了：

* `active.id`: 当前拖动的元素 id
* `over.id`: 放下时目标元素的 id

<br/>

 **👇 举例说明业务逻辑：**

假设拖动的是「云台摄像机」，即 `categoryId: 1-1`，拖动到「智能摄像机」上面（`categoryId: 1-2`）

那么：

```js
active.id === '1-1'
over.id === '1-2'
```

你希望只在**同一父级**下移动才生效，所以：

```js
const oldItem = flatList.find(i => i.categoryId === active.id); // 1-1
const overItem = flatList.find(i => i.categoryId === over.id);  // 1-2

if (oldItem.parentId !== overItem.parentId) return; // 不同父级不处理
```

然后通过 `arrayMove` 调整顺序：

* 若是顶层父级：就直接 `arrayMove(data, ...)`
* 若是某个父级下的子项：就只 `arrayMove(parent.children, ...)`

<br/><br/>

**`4.const onDragEnd = ({active, over}) => { }`函数细致理解阅读：**

<br/>

**4.1下面的这段代码什么意思？**

```js
if (!over || active.id === over.id) {
  console.log('未拖拽到目标或目标与自身相同，忽略操作');
  return;
}
```

- **分解说明：**

	* `!over`：

		* 表示用户拖拽时没有“落到”另一个合法的行上。
		* 可能是松手时鼠标在空白区域，找不到目标元素。

	* `active.id === over.id`：

		* 表示你把某一行拖到了它自己身上。
		* 这种情况不需要变动顺序。

<br/>

**✅ 作用：**

这是一个“提前退出”的判断，用于避免不必要的处理逻辑，提高健壮性。

<br/>

**🧪 举例：**

假设你拖动的是分类项 `"云台摄像机"`，它的 `id = "1-2"`：

* 如果你松手时没有对准其他行 → `!over` 为 `true`，不会排序。
* 如果你松手时又放回自己位置 → `active.id === over.id` 为 `true`，也不会排序。

<br/>

**4.2扁平数组中找到原始数据**

如下两行代码：

```js
const oldItem = flatList.find(item => item.categoryId === active.id);
const overItem = flatList.find(item => item.categoryId === over.id);
```

<br/>

**🔍 它们的作用是：**

在拖拽结束时，`dnd-kit` 提供了：

* `active.id`：被拖动的行的 ID（拖拽源）
* `over.id`：放下时目标行的 ID（拖拽目标）

<br/>

这两行代码用来在 `flatList` 这个「扁平化数组」中找到：

* 被拖动的原始数据项（`oldItem`）
* 被放下的目标数据项（`overItem`）

<br/>

**📘 举个例子说明：**

假设你拖动的是「云台摄像机」（`categoryId: "1-2"`），目标是「智能摄像机」（`categoryId: "1-1"`）

那么：

```js
active.id === "1-2"
over.id === "1-1"
```

<br/>

你就会从 `flatList` 中找到：

```js
oldItem = { categoryId: "1-2", categoryName: "云台摄像机", parentId: "1" }
overItem = { categoryId: "1-1", categoryName: "智能摄像机", parentId: "1" }
```

这些对象后续会用来判断：

* 是否是同一个父级（`parentId` 相等）
* 是哪一层级的排序
* 要不要执行 `arrayMove()` 排序


<br/><br/>

**4.3更新原始数据结构函数**

```js
const updateList = (list) => {
	....
	..
	.
}
```

这是一个更新数据的函数，接收当前的 `list`（即表格的数据）作为参数，返回排序后的新列表。

<br/>

**第一种情况：顶层排序**

```js
if (parentId === null) {
  const oldIndex = list.findIndex(i => i.categoryId === active.id);
  const newIndex = list.findIndex(i => i.categoryId === over.id);
  console.log('顶层拖拽排序:', oldIndex, '=>', newIndex);
  return arrayMove(list, oldIndex, newIndex);
}
```

* `parentId === null` 说明拖动的是“顶层分类”。
* `list.findIndex(...)` 找到被拖动项和目标项在数组中的下标。
* `arrayMove(list, oldIndex, newIndex)` 表示：将 `oldIndex` 的项移动到 `newIndex` 位置。
* ✅ 直接返回新的顶层顺序数组。

<br/>

**第二种情况：子项排序**

```js
return list.map(item => {
  if (item.categoryId === parentId && item.children) {
    const oldIndex = item.children.findIndex(i => i.categoryId === active.id);
    const newIndex = item.children.findIndex(i => i.categoryId === over.id);
    console.log(`子项拖拽排序 [parentId=${parentId}]`, oldIndex, '=>', newIndex);
    item.children = arrayMove(item.children, oldIndex, newIndex);
  }
  return item;
});
```

* 如果拖动的是“某个父分类下的子项”，则 `parentId !== null`。
* 遍历 `list`，找出哪个父分类的 `categoryId === parentId`，也就是当前子项的所属父类。
* 使用 `arrayMove` 排序这个父分类下的 `children` 数组。

<br/><br/>

**🧪 举个完整例子：**

假设你的数据结构如下：

```js
[
  {
    categoryId: "1",
    categoryName: "摄像机",
    children: [
      { categoryId: "1-1", categoryName: "云台摄像机" },
      { categoryId: "1-2", categoryName: "智能摄像机" }
    ]
  },
  {
    categoryId: "2",
    categoryName: "门铃"
  }
]
```

<br/>

**情况 1：拖动“摄像机”到“门铃”下面（顶层）**

```js
parentId === null
list = 原始数组
active.id = "1"
over.id = "2"
```

<br/>

返回：

```js
[
  { categoryId: "2", ... },
  { categoryId: "1", ... }
]
```


<br/>

**情况 2：拖动“云台摄像机”到“智能摄像机”上（子项）**

```js
parentId === "1"
item.categoryId === "1"
item.children = [ {1-1}, {1-2} ]
```

<br/>

结果：

```js
item.children = [ {1-2}, {1-1} ]
```




<br/><br/><br/>

***
<br/><br/><br/>
> <h1 id="下拉组件Select">下拉组件Select</h1>

**Select组件原代码，如下：**

```ts
declare const Select: (<ValueType = any, OptionType extends BaseOptionType | DefaultOptionType = DefaultOptionType>(
  props: React.PropsWithChildren<SelectProps<ValueType, OptionType>> & React.RefAttributes<BaseSelectRef>
) => React.ReactElement) & {
  displayName?: string;
  SECRET_COMBOBOX_MODE_DO_NOT_USE: string;
  Option: typeof Option;
  OptGroup: typeof OptGroup;
  _InternalPanelDoNotUseOrYouWillBeFired: typeof PurePanel;
};
```

 `Select` 是一个功能强大的下拉选择组件， `declare const Select...` 是它的 TypeScript 类型定义，声明了 `Select` 的泛型能力和拓展成员（如 `Option`, `OptGroup` 等），**说明 `Select`：**

* 是一个 **泛型函数组件**，支持传入自定义的 `ValueType` 和 `OptionType`
* 返回的是 `ReactElement`
* 拥有几个静态属性，如：
	* `Select.Option`（用于定义选项）
	* `Select.OptGroup`（用于选项分组）
	* `SECRET_COMBOBOX_MODE_DO_NOT_USE`（私有 API，不建议用）
	* `_InternalPanelDoNotUseOrYouWillBeFired`（内部 API）

***
<br/><br/><br/>
> <h2 id="onChange函数">onChange函数</h2>

**`onChange={handleSelectChange}`**

- **作用：**当用户选择某一项时触发这个函数。
- **函数参数：**

```
js
const handleSelectChange = (value, option) => {
  console.log("选中的值:", value);
  console.log("选项对象:", option);
};
```
- value 是选中项的值（value 字段）
- option 是选中项的完整对象，包含 label、value、等额外数据

***
<br/><br/><br/>
> <h2 id="options数据源，加入数据">options数据源，加入数据</h2>

**`options={dataTypeList}`**

* **作用**：快速通过数组渲染出下拉选项

* **数组格式要求**：

```js
const dataTypeList = [
{ label: '文本类型', value: 'text' },
{ label: '数字类型', value: 'number' },
{ label: '日期类型', value: 'date' },
];
```

* 每一项都必须包含：

* `label`：用户看到的文字
* `value`：用户选择后拿到的值

> ⚠️ 注意：`Select` 会根据 `value` 唯一标识项。

<br/>

```tsx
import React from 'react';
import { Select } from 'antd';

const { Option } = Select;


const MySelect = () => {
  const handleChange = (value: string) => {
    console.log(`Selected: ${value}`);
  };

  return (
    <Select defaultValue="apple" style={{ width: 200 }} onChange={handleChange}>
      <Option value="apple">🍎 Apple</Option>
      <Option value="banana">🍌 Banana</Option>
      <Option value="cherry">🍒 Cherry</Option>
    </Select>
  );
};
```

<br/>

**效果说明（想象一个下拉框）：**

```
▼ 🍎 Apple
  🍌 Banana
  🍒 Cherry
```

选择后会调用 `handleChange` 输出选中值。


<br/><br/>

**使用 `options` 属性（推荐做法）**

```tsx
const options = [
  { label: 'Apple 🍎', value: 'apple' },
  { label: 'Banana 🍌', value: 'banana' },
  { label: 'Cherry 🍒', value: 'cherry' },
];

const MySelect = () => {
  return (
    <Select
      style={{ width: 200 }}
      defaultValue="banana"
      options={options}
      onChange={(value) => console.log('Selected:', value)}
    />
  );
};
```

<br/>

**使用 `Select.Option` 和 `Select.OptGroup`**

```tsx
const MyGroupedSelect = () => (
  <Select style={{ width: 200 }} placeholder="Select a fruit">
    <Select.OptGroup label="Citrus">
      <Select.Option value="orange">Orange</Select.Option>
      <Select.Option value="lemon">Lemon</Select.Option>
    </Select.OptGroup>
    <Select.OptGroup label="Berries">
      <Select.Option value="strawberry">Strawberry</Select.Option>
      <Select.Option value="blueberry">Blueberry</Select.Option>
    </Select.OptGroup>
  </Select>
);
```

<br/>

**效果：**

```
▼ Select a fruit
  Citrus
    Orange
    Lemon
  Berries
    Strawberry
    Blueberry
```

<br/><br/>

**带搜索功能的 Select（带 `<input>` 框）**

```tsx
<Select
  showSearch
  placeholder="Select a person"
  optionFilterProp="children"
  onChange={(value) => console.log('Selected:', value)}
  filterOption={(input, option) =>
    (option?.children as string).toLowerCase().includes(input.toLowerCase())
  }
>
  <Option value="jack">Jack</Option>
  <Option value="lucy">Lucy</Option>
  <Option value="tom">Tom</Option>
</Select>
```

<br/>

**效果：**

* 有搜索输入框
* 可以通过输入过滤选项

<br/>

**🖼️ 效果图（示意）**

```
| 🍌 Banana ▼ |
----------------
| 🍎 Apple    |
| 🍌 Banana   |
| 🍒 Cherry   |
----------------
```

<br/>

或带搜索：

```
| 🔍 [luc] ▼  |
----------------
| Lucy        |
----------------
```

---
<br/>

**✅ 总结**

| 功能                | 方法                            |
| ----------------- | ----------------------------- |
| 基本下拉选择            | `<Select><Option /></Select>` |
| 使用 `options` 数组配置 | `<Select options={...} />`    |
| 分组显示              | `<Select.OptGroup />`         |
| 带搜索框              | `showSearch + filterOption`   |
| 受控/非受控            | `value` / `defaultValue` 控制   |


***
<br/><br/><br/>
> <h2 id="下拉提示文案placeholder">下拉提示文案placeholder</h2>

**`placeholder={intl.formatMessage(...)}`**

* **作用**：下拉框未选择时显示的提示文本。

* `intl.formatMessage(...)` 是国际化方法，用于根据语言包翻译内容。

* 如果你不使用国际化，可以直接写：

```jsx
placeholder="请选择数据类型"
```

* 使用 `react-intl` 国际化框架的写法如下：

```js
placeholder={intl.formatMessage({ id: 'pages.message.select.placeholder' })}
```

配套语言包中必须有：

```json
{
"pages.message.select.placeholder": "请选择数据类型"
}
```





<br/><br/><br/>

***
<br/><br/><br/>
> <h1 id="加载指示器组件Spin"> 加载指示器组件Spin</h1>

 **`<Spin />`** 是一个**加载指示器组件**，用于在页面或组件正在加载时，给用户一个\*\*“加载中”反馈\*\*，通常是一个旋转的图标（类似 Loading 圈）。
 
 <br/>
 
 **常用属性总结**

| 属性名         | 类型                                | 说明           |
| ----------- | --------------------------------- | ------------ |
| `spinning`  | `boolean`                         | 是否显示加载状态     |
| `tip`       | `ReactNode`                       | 自定义加载文案      |
| `size`      | `'small' \| 'default' \| 'large'` | 控制圈的大小       |
| `indicator` | `ReactElement`                    | 自定义加载图标      |
| `delay`     | `number(ms)`                      | 延迟显示时间（避免闪烁） |

---
<br/>

- **1.简单 loading 圈**

```tsx
import { Spin } from 'antd';

const MyComponent = () => (
  <Spin />
);
```

🔁 显示默认的旋转加载圈。

<br/>

- **2.控制显示隐藏（通过 `spinning` 属性）**

```tsx
<Spin spinning={isLoading}>
  <div>
    加载完成后显示这个内容
  </div>
</Spin>
```

* `spinning: true` → 显示 loading 圈，内容被遮挡
* `spinning: false` → 显示原内容，不再加载

<br/>

- **3.自定义描述文字**

```tsx
<Spin spinning={true} tip="加载中，请稍后...">
  <div style={{ height: 100, background: '#f0f2f5' }} />
</Spin>
```

效果是一个旋转圈，下面显示“加载中，请稍后...”。

<br/>

- **4.自定义指示符（替换默认 loading 圈）**

```tsx
import { LoadingOutlined } from '@ant-design/icons';
import { Spin } from 'antd';

const antIcon = <LoadingOutlined style={{ fontSize: 24 }} spin />;

<Spin indicator={antIcon} />
```

用自己的图标替代默认的旋转指示器。

<br/>

- **5.全屏 loading（配合样式使用）**

```tsx
<Spin tip="页面加载中..." size="large" style={{
  position: 'fixed',
  top: '50%',
  left: '50%',
  transform: 'translate(-50%, -50%)',
  zIndex: 1000
}} />
```

<br/>

**完整示例**

```tsx
import React, { useState, useEffect } from 'react';
import { Spin } from 'antd';

const Demo = () => {
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setTimeout(() => setLoading(false), 2000);
  }, []);

  return (
    <Spin spinning={loading} tip="加载数据中...">
      <div style={{ padding: 24, background: '#fff' }}>
        {loading ? null : '数据已加载完成 🎉'}
      </div>
    </Spin>
  );
};
```

***
<br/><br/><br/>
> <h2 id="模态框组件-Modal">模态框组件-Modal</h2>

**代码引入：**

```tsx
<Modal
  centered={props.centered || true}
  className={`${styles.customConfirm} ${props.contentLeft ? styles.customConfirmLeft : ''}`}
  width={props.width || 400}
  {...props}
  title={null}
  closable={false}
  footer={null}
/>
```

<br/>

| 属性                         | 含义             | 解释                                                                               |      |                                           |
| -------------------------- | -------------- | -------------------------------------------------------------------------------- | ---- | ----------------------------------------- |
| \`centered={props.centered |                | true}\`                                                                          | 居中显示 | **始终居中**（⚠️ 注意：这个写法可能有 bug，详见后面）          |
| `className=...`            | 自定义样式类名        | 使用了两个样式：`customConfirm` 和可选的 `customConfirmLeft`（当 `props.contentLeft` 为 true 时） |      |                                           |
| \`width={props.width       |                | 400}\`                                                                           | 宽度   | 设置模态框宽度，优先使用 props 传入的 `width`，否则默认 400px |
| `{...props}`               | 扩展传参           | 接收父组件传入的其他属性，比如 `visible`、`onOk`、`onCancel` 等                                    |      |                                           |
| `title={null}`             | 去除标题           | 不显示模态框标题栏                                                                        |      |                                           |
| `closable={false}`         | 不显示右上角关闭按钮 `×` | 必须靠逻辑或按钮关闭                                                                       |      |                                           |
| `footer={null}`            | 去除底部操作区        | 不显示“确认/取消”按钮区域                                                                   |      |                                           |


- **⚠️ 注意点** ❗️`centered={props.centered || true}` 可能有问题
	
	* JS 表达式中，`props.centered || true` 会在 `props.centered === false` 时仍然返回 `true`
	* 即使你显式传了 `centered={false}`，也会被忽略
	  ✅ **正确写法** 应该是：

```tsx
centered={props.centered !== undefined ? props.centered : true}
```

<br/>

**举例：**

```tsx
<Modal
  title="新增用户"
  open={isVisible}
  maskClosable={false}
  onCancel={handleCancel}
  confirmLoading={loading}
  onOk={handleOk}
>
  弹窗内容
</Modal>
```

| 属性名                 | 类型          | 说明                | 示例                                |
| ------------------- | ----------- | ----------------- | --------------------------------- |
| `title`             | `ReactNode` | 弹窗标题              | `title="新增用户"`                    |
| `open`（以前叫 visible） | `boolean`   | 是否显示弹窗            | `open={true}`                     |
| `maskClosable`      | `boolean`   | 点击遮罩是否关闭弹窗        | `maskClosable={false}`            |
| `onCancel`          | `function`  | 点击取消按钮或遮罩时触发      | `onCancel={() => setOpen(false)}` |
| `onOk`              | `function`  | 点击确定按钮时触发         | `onOk={submitForm}`               |
| `confirmLoading`    | `boolean`   | 确定按钮是否 loading 状态 | `confirmLoading={loading}`        |



---
<br/>

**完整示例**

**组件封装 `MyCustomModal.tsx`**

```tsx
import React from 'react';
import { Modal } from 'antd';
import styles from './MyModal.module.css'; // 样式文件

const MyCustomModal = (props) => {
  return (
    <Modal
      centered={props.centered !== undefined ? props.centered : true}
      className={`${styles.customConfirm} ${props.contentLeft ? styles.customConfirmLeft : ''}`}
      width={props.width || 400}
      {...props}
      title={null}
      closable={false}
      footer={null}
    >
      {props.children}
    </Modal>
  );
};

export default MyCustomModal;
```

<br/>

**使用该组件：**

```tsx
import React, { useState } from 'react';
import MyCustomModal from './MyCustomModal';

const DemoPage = () => {
  const [visible, setVisible] = useState(false);

  return (
    <>
      <button onClick={() => setVisible(true)}>打开自定义模态框</button>
      <MyCustomModal
        visible={visible}
        onCancel={() => setVisible(false)}
        contentLeft={true}
      >
        <div>
          <h3>确认删除？</h3>
          <p>这个操作将无法撤销！</p>
          <button onClick={() => setVisible(false)}>关闭</button>
        </div>
      </MyCustomModal>
    </>
  );
};
```

<br/>

**样式参考（MyModal.module.css）**

```css
.customConfirm {
  text-align: center;
  padding: 24px;
}

.customConfirmLeft {
  text-align: left;
}
```

<br/>

- **效果描述**

	* 页面上有一个按钮【打开自定义模态框】
	* 点击后弹出一个**无标题、无底部按钮、居中展示**的模态框
	* 内容自定义，可左对齐或居中
	* 模态框只能通过点击“关闭”按钮或外部关闭逻辑来退出

---
<br/>

- **✅ 总结**

| 功能点      | 实现                                 |
| -------- | ---------------------------------- |
| 自定义内容模态框 | `title={null}` + `footer={null}`   |
| 可控显隐     | `visible={...}` + `onCancel={...}` |
| 自定义样式    | 通过 `className` 组合控制                |
| 默认居中显示   | `centered={true}`                  |
| 更灵活的封装   | 接收 `children` 和其余 props            |


***
<br/><br/><br/>
> <h2 id="弹框提示组件message">弹框提示组件message</h2>

```ts
// 组件
declare const staticMethods: MessageMethods & BaseMethods;


// 导入消息提示组件
import { message } from 'antd';
```

这是 Ant Design 中的消息提示组件：`message`，它提供一种**全局展示操作反馈信息**的方式，类似于“Toast 弹出提示”。

---
<br/>

`message` 是 Ant Design 提供的一个静态对象（不是 React 组件），调用它的方法可以直接在页面右上角弹出提示信息。

- **📌 它用来显示：**
	* 操作成功 / 失败提示
	* 加载中提示
	* 一般信息提醒

<br/>

**常用用法（基础）**


```tsx
import { message } from 'antd';

// 成功提示
message.success('操作成功');

// 错误提示
message.error('操作失败');

// 警告提示
message.warning('警告：请检查输入');

// 信息提示
message.info('正在加载页面...');

// 加载中
const hide = message.loading('加载中...', 0); // 0 表示不会自动关闭
setTimeout(hide, 2000); // 2 秒后手动关闭
```

> 所有的提示默认显示 3 秒，除非你设置了 `duration`。

---
<br/>

**参数详解**

```ts
message.success(content, duration, onClose)
```

| 参数名        | 类型                    | 说明                 |
| ---------- | --------------------- | ------------------ |
| `content`  | `string \| ReactNode` | 显示的内容              |
| `duration` | `number`（秒）或 `0`      | 自动关闭的时间，0 表示不会自动关闭 |
| `onClose`  | `function`            | 关闭时的回调函数           |

<br/>

- **示例：**

```tsx
message.error('上传失败', 2, () => {
  console.log('用户看到后消失了');
});
```

---
<br/>

**类型和方法列表（你看到的 `MessageMethods` & `BaseMethods`）**

**Ant Design 的 `message` 暴露了这些方法：**

| 方法名               | 功能        |
| ----------------- | --------- |
| `message.info`    | 信息提示（蓝色）  |
| `message.success` | 成功提示（绿色）  |
| `message.warning` | 警告提示（橙色）  |
| `message.error`   | 错误提示（红色）  |
| `message.loading` | 加载中（转圈）   |
| `message.open`    | 自定义配置打开消息 |
| `message.destroy` | 立即销毁所有消息  |

<br/>

**✨ 自定义配置打开**

```tsx
message.open({
  type: 'success',
  content: '操作成功！',
  duration: 3,
  onClose: () => console.log('关闭了'),
});
```

<br/>

**样式效果（视觉描述）**

打开 `message.success('保存成功')` 会出现这样的效果：

```
✅ 保存成功        [3s 后自动消失]
```

在页面右上角浮出，多个 message 会自动堆叠，最多显示 5 条，之后会覆盖前面的。

<br/>

**如何修改默认配置（全局）**

```tsx
message.config({
  top: 100,         // 距离顶部距离 px
  duration: 2,      // 默认持续时间
  maxCount: 3,      // 同时最多显示几个
});
```





<BR/><BR/><BR/><BR/>

***
<BR/>
># <H1 ID="Form表单">[Form表单](https://ant.design/components/form-cn#form-demo-basic)</H1>

[Ant Design](https://ant.design) 的表单组件 `<Form />` 和 `<Form.Item />`，再配合 `<Input />` 输入框实现一个带有**表单验证**和**国际化提示**的输入区域。

<br/>

**📦 使用的组件：**

| 组件                                   | 作用              |
| ------------------------------------ | --------------- |
| `<Form />`                           | 整个表单容器          |
| `<Form.Item />`                      | 表单项（label + 控件） |
| `<Input />`                          | 输入框控件           |
| `this.props.intl.formatMessage(...)` | 国际化，显示不同语言的文字   |

---
<br/> 

**1.Form 表单组件**

```tsx
<Form
  name="copyform"
  ref={this.copyFormRef}
  labelCol={{ span: 4 }}       // label 宽度 4 栅格（共 24）
  wrapperCol={{ span: 20 }}    // 输入控件占 20 栅格
/>
```

- **✅ 解释：**
	* `name`: 表单名称，字段绑定时会用到。
	* `ref`: React 的引用对象，可以通过 `this.copyFormRef.current` 获取表单实例（例如做验证、提交等）。
	* `labelCol`: 控制 `<Form.Item>` 中 label 的宽度（栅格系统：一行共 24）
	* `wrapperCol`: 控制输入控件部分的宽度。

<br/>

 **🔹 2.Form.Item 表单项**

```tsx
<Form.Item
  label={this.props.intl.formatMessage({ id: 'pages.product.list.add.name' })}
  name="productName"
  rules={[{
    required: true,
    message: this.props.intl.formatMessage({ id: 'pages.product.list.copy.name.tip' })
  }]}
/>
```

- **✅ 解释：**

	* `label`: 表单项的标签（使用国际化方式获取，比如“产品名称”）
	* `name`: 这个字段的 key，会绑定到 `form.getFieldValue('productName')`
	* `rules`: 校验规则数组，常见规则如：`required`, `minLength`, `pattern` 等
	
		* `required: true`: 必填
		* `message`: 验证不通过时显示的错误提示

---
<br/>

- **🔹3.Input 输入框**

```tsx
<Input
  maxLength={60}
  placeholder={...}
/>
```

- **✅ 解释：**
	* `maxLength={60}`：最多输入 60 个字符
	* `placeholder={}`：占位文本，建议配合国际化文本

---
<br/>

**完整组合示例（React 类组件）**

对应国际化配置（假设你使用的是 `react-intl`）,在语言文件中定义：

```json
{
  "pages.product.list.add.name": "产品名称",
  "pages.product.list.copy.name.tip": "请输入产品名称",
  "pages.product.list.name.placeholder": "请输入名称（最多 60 字）"
}
```

<br/>

```tsx
<Form
  name="copyform"
  ref={this.copyFormRef}
  labelCol={{ span: 4 }}
  wrapperCol={{ span: 20 }}
>
  <Form.Item
    label={this.props.intl.formatMessage({ id: 'pages.product.list.add.name' })}
    name="productName"
    rules={[{
      required: true,
      message: this.props.intl.formatMessage({ id: 'pages.product.list.copy.name.tip' })
    }]}
  >
    <Input
      maxLength={60}
      placeholder={this.props.intl.formatMessage({ id: 'pages.product.list.name.placeholder' })}
    />
  </Form.Item>
</Form>
```

<br/>

**运行效果（视觉描述）**

```
产品名称: [              输入框                ]
          ↑ 占位符：请输入名称（最多 60 字）

【提交按钮】
```

🔺 如果点击提交而不输入，会提示：

```
请输入产品名称
```


<br/><br/>
> <h3 id="Form.Item属性详解">Form.Item属性详解</h3>

`<Form.Item>` 是表单的单个字段容器，用于显示标签和控件。

```tsx
<Form.Item
  name="username"
  label="用户名"
  rules={[{ required: true, message: '请输入用户名' }]}
>
  <Input />
</Form.Item>
```

| 属性      | 说明                  | 示例                                          |
| ------- | ------------------- | ------------------------------------------- |
| `name`  | 字段名，用于数据提交绑定        | `name="username"` 提交时 `{ username: 'xxx' }` |
| `label` | 左侧显示的字段名称           | `label="用户名"`                               |
| `rules` | 校验规则数组（支持必填、长度、格式等） | `rules={[{ required: true }]}`              |

<br/>

**🛠 常见校验规则 rules：**

```tsx
rules={[
  { required: true, message: '不能为空' },
  { min: 3, message: '最少3个字符' },
  { type: 'email', message: '邮箱格式不正确' },
]}
```


 <br/><br/>
> <h3 id="默认值initialValue">默认值initialValue</h3>

* **作用**：为这个字段设置一个默认值。
* **示例**：

  ```jsx
  <Form.Item name="age" initialValue={18} label="年龄">
    <InputNumber />
  </Form.Item>
  ```

  页面加载时，表单中 `age` 字段就自动显示 `18`。

> ⚠️ 注意：如果 `Form` 本身使用了 `form.setFieldsValue()` 设置默认值，则 `initialValue` 会被覆盖。

<br/><br/>
> <h3 id="rules校验规则详解">rules校验规则详解</h3>

下面这段代码是一个**自定义校验函数**，用于表单联动校验 —— 比如“最小值不能大于最大值”。

```jsx
<Form form={form} layout="vertical">
  <Form.Item name="maxValue" label="最大值" rules={[{ required: true, message: '请输入最大值' }]}>
    <Input />
  </Form.Item>

  <Form.Item
    name="minValue"
    label="最小值"
    rules={[
      ({ getFieldValue }) => ({
        validator(_, value) {
          if (
            value &&
            getFieldValue('maxValue') &&
            Number(value) >= Number(getFieldValue('maxValue'))
          ) {
            return Promise.reject(new Error('最小值不能大于或等于最大值'));
          }
          return Promise.resolve();
        },
      }),
    ]}
  >
    <Input />
  </Form.Item>
</Form>
```

**💡 效果：**

* 当用户输入的最小值 `>=` 最大值时，会报错“不合法”；
* 否则就通过校验。


<br/>

**🧩 `({ getFieldValue }) => ({ validator(...) })`**

* `getFieldValue` 是 antd 内部提供的方法，可以获取其他字段的当前值。
* 它在 **写联动验证逻辑时非常有用**。

<br/> 

**🧩 `validator(_, value) { ... }`**

* `validator` 是一个异步函数，你可以写任意逻辑进行校验。
* `_` 表示当前字段的 `rule` 配置（我们不使用它所以用 `_` 占位）
* `value` 是当前字段输入的值（即最小值的输入框里的内容）

<br/>

**🚫 核心逻辑是：**

```js
if (
  value && // 当前字段有输入
  getFieldValue('maxValue') && // maxValue 也有值
  Number(value) >= Number(getFieldValue('maxValue')) // 最小值 ≥ 最大值
)
```

就会报错：

```js
return Promise.reject(new Error('最小值不能大于或等于最大值'));
```

否则验证通过：

```js
return Promise.resolve();
```







***
<br/><br/><br/>
> <h2 id="Form表单基本属性讲解">Form表单基本属性讲解</h2>

**基本使用如下：**

```jsx
<Form
  form={form1}
  labelCol={{ span: 6 }}
  wrapperCol={{ span: 18 }}
  onFinish={onFinish}
  onValuesChange={({ isUniversal }) => {
    setIsUniversal(isUniversal);
  }}
>
```

上述`<Form>` 表单组件，它封装了很多表单逻辑，比如输入校验、布局、表单状态管理等。

<br/>

**1.`form={form1}`**

* **含义**：使用外部通过 `Form.useForm()` 创建的 form 实例 `form1`。
* **作用**：可以操作表单，比如重置、获取值、设置值等。
* **示例**：

  ```js
  const [form1] = Form.useForm();

  form1.setFieldsValue({ name: "张三" }); // 设置某个字段的值
  form1.resetFields(); // 重置表单
  form1.getFieldsValue(); // 获取所有表单字段的值
  ```

<br/>

**2.`labelCol={{ span: 6 }}`**

* **含义**：label（标签）的列宽占 6 份（共 24 格栅格系统）。
* **作用**：控制表单中每一行左侧“字段名称”部分的宽度。
* **示例效果**：

  * “用户名”四个字会占 6/24 的宽度
  * 如果设置为 `span: 4` 就更窄，`span: 8` 就更宽

<br/>

**3.`wrapperCol={{ span: 18 }}`**

* **含义**：输入框部分的宽度占 18 格。
* **作用**：与 `labelCol` 配合实现布局（6 + 18 = 24，刚好一整行）
* **注意**：这两个一般要配合调整成总数不超过 24，否则会换行错乱。

<br/>

**4.`onFinish={onFinish}`**

* **含义**：表单校验通过后，点击“提交”会调用 `onFinish` 函数。
* **作用**：收集提交数据的地方。
* **示例**：

  ```js
  const onFinish = (values) => {
    console.log("提交的表单数据:", values);
  };
  ```

<br/>

**5.`onValuesChange={({ isUniversal }) => { setIsUniversal(isUniversal); }}`**

* **含义**：当表单中任意字段的值发生变化时触发。
* **作用**：可以监听变化做联动，比如某个字段勾选后，其他字段显示/隐藏。
* **举例说明**：

  假设你有一个表单字段：

  ```jsx
  <Form.Item name="isUniversal" label="通用设置" valuePropName="checked">
    <Switch />
  </Form.Item>
  ```

  然后你写了：

  ```js
  const [isUniversal, setIsUniversal] = useState(false);
  ```

  每次 `Switch` 的开关状态改变时，`setIsUniversal` 就会更新，UI 就能联动其他部分（比如显示额外字段）。


---
<br/>

**✅ 组合使用场景示例：**

```jsx
<Form
  form={form1}
  labelCol={{ span: 6 }}
  wrapperCol={{ span: 18 }}
  onFinish={(values) => {
    console.log("提交成功:", values);
  }}
  onValuesChange={({ isUniversal }) => {
    setIsUniversal(isUniversal);
  }}
>
  <Form.Item
    name="username"
    label="用户名"
    rules={[{ required: true, message: "请输入用户名" }]}
  >
    <Input />
  </Form.Item>

  <Form.Item
    name="isUniversal"
    label="是否通用"
    valuePropName="checked"
  >
    <Switch />
  </Form.Item>

  <Form.Item wrapperCol={{ offset: 6, span: 18 }}>
    <Button type="primary" htmlType="submit">提交</Button>
  </Form.Item>
</Form>
```






<br/><br/><br/>

***
<br/>

> <h1 id="Button组件">Button组件</h1>

```jsx
disabled={true}     // 明确写法
disabled={!isActive} // 动态控制
disabled             // 等同于 true（简写）
```

<br/>

**1.`disabled={1}` ❌（⚠️写法错误）**

* `disabled` 表示按钮是否为“禁用”状态，**应该是一个布尔值**（`true` / `false`）。
* 你写的 `disabled={1}` 虽然在 JavaScript 中等同于 `true`，但**不是推荐写法**。

<br/>

**2.`type="primary"`**

* 按钮的类型，用于**设置颜色和样式风格**
* 可选值有：

| 类型          | 说明        |
| ----------- | --------- |
| `"primary"` | 主要按钮（蓝色）  |
| `"default"` | 默认按钮      |
| `"dashed"`  | 虚线按钮      |
| `"text"`    | 文字按钮（无背景） |
| `"link"`    | 超链接样式按钮   |

<br/>

**3. `size="small"`**

* 设置按钮的大小
* 可选值有：

| 值          | 大小说明 |
| ---------- | ---- |
| `"large"`  | 较大按钮 |
| `"middle"` | 默认大小 |
| `"small"`  | 较小按钮 |

---
<br/>

**✅ 示例代码（推荐写法）**

```jsx
import { Button } from 'antd';

function Demo() {
  const isDisabled = true;

  return (
    <Button
      disabled={isDisabled}
      type="primary"
      size="small"
    >
      确认
    </Button>
  );
}
```

<br/><br/><br/>

***
<br/>

> <h1 id="鼠标悬停或聚焦显示组件Tooltip">鼠标悬停或聚焦显示组件Tooltip</h1>


`Tooltip` 是一个**悬浮提示组件**，通常包裹在按钮、图标、文字等外部，当用户悬停时显示提示信息。

<br/>

**✅ 基本用法**

```jsx
import { Tooltip, Button } from 'antd';

function Demo() {
  return (
    <Tooltip title="这是提示内容">
      <Button>悬停我</Button>
    </Tooltip>
  );
}
```

**🔍 效果：**

* 鼠标悬停在按钮上时，会显示一个小的浮层文字：“这是提示内容”。

---
<br/>

**🧩 常用属性说明**

| 属性名         | 类型          | 说明            |
| ----------- | ----------- | ------------- |
| `title`     | `ReactNode` | 要显示的提示内容（必填）  |
| `placement` | `string`    | 提示框出现的位置（见下方） |
| `color`     | `string`    | 提示框的背景颜色（可选）  |
| `arrow`     | `boolean`   | 是否显示箭头，默认显示   |

<br/>

**✅ `placement` 可选值**

| 值                                | 含义    |
| -------------------------------- | ----- |
| `top`, `bottom`, `left`, `right` | 四个方向  |
| `topLeft`, `topRight` 等          | 更具体位置 |

<br/>

**🔁 示例：多个 Tooltip 用法**

```jsx
import { Tooltip, Button, Space } from 'antd';

function TooltipExample() {
  return (
    <Space direction="vertical" size="middle">
      {/* 默认 top */}
      <Tooltip title="默认提示">
        <Button>默认</Button>
      </Tooltip>

      {/* 指定位置 */}
      <Tooltip title="提示在右侧" placement="right">
        <Button>右侧提示</Button>
      </Tooltip>

      {/* 自定义颜色 */}
      <Tooltip title="红色提示" color="red">
        <Button>自定义颜色</Button>
      </Tooltip>

      {/* 包裹文本 */}
      <Tooltip title="这是一个很长的说明">
        <span style={{ textDecoration: 'underline', cursor: 'pointer' }}>
          悬停查看说明
        </span>
      </Tooltip>
    </Space>
  );
}
```

---
<br/>

 **小技巧**

**1.`Tooltip` 不要包裹 disabled 的按钮，建议用 `<span>` 外层包一下：**

   ```jsx
   <Tooltip title="按钮已禁用">
     <span>
       <Button disabled>禁用按钮</Button>
     </span>
   </Tooltip>
   ```

2. **`Tooltip` 也支持 `trigger="click"`、`focus`，但默认是 `hover`**



<br/><br/><br/>

***
<br/>

> <h1 id="Upload图片上传组件">Upload图片上传组件</h1>


**`<Upload>` 组件结构**

```jsx
<Upload
  name="avatar"
  disabled={!isEditing}
  style={{ width: 104, height: 104 }}
  accept=".png"
  listType="picture-card"
  className="avatar-uploader"
  showUploadList={false}
  beforeUpload={(file) => {
    return this.beforeUploadImg(file).catch(() => Upload.LIST_IGNORE);
  }}
  onChange={this.handleChange}
>
  <img src={url} />
</Upload>
```

<br/>

**✅ 属性详解：**

| 属性                                    | 类型                                          | 说明 |
| ------------------------------------- | ------------------------------------------- | -- |
| `name="avatar"`                       | 上传的字段名（后端接收的字段）                             |    |
| `disabled={!isEditing}`               | 控制是否禁用（不可点击），编辑模式下才可上传                      |    |
| `style={{ width: 104, height: 104 }}` | 自定义上传区域的宽高                                  |    |
| `accept=".png"`                       | 仅允许上传 `.png` 格式的文件                          |    |
| `listType="picture-card"`             | 上传列表样式为“图片卡片”形式                             |    |
| `className="avatar-uploader"`         | 自定义样式类名                                     |    |
| `showUploadList={false}`              | 不显示上传后的文件列表（只展示你手动插入的 img）                  |    |
| `beforeUpload={(file) => ...}`        | 上传前钩子，返回 `false` 或 `Promise.reject()` 会阻止上传 |    |
| `onChange={this.handleChange}`        | 上传状态改变后的回调，比如获取 `file.url`、状态更新             |    |


<br/>

**🧩 第二部分：上传前校验方法 `beforeUploadImg`**

```js
beforeUploadImg = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    // 把 file 读取为 base64 编码（DataURL）
    reader.readAsDataURL(file);

    reader.onload = (e) => {
      const img = new Image();
      img.src = e?.target?.result; // base64 结果

      img.onload = () => {
        // 图片加载完成，可拿到宽高等信息
        // 你可以在这里校验尺寸、大小、格式等

        resolve(); // 放行上传
      };

      img.onerror = () => {
        reject(new Error('图片加载失败'));
      };
    };

    reader.onerror = () => {
      reject(new Error('文件读取失败'));
    };
  });
};
```

<br/> 

**✅ 这个方法做了什么？**

| 步骤                               | 说明                          |
| -------------------------------- | --------------------------- |
| `FileReader.readAsDataURL(file)` | 将用户选择的文件读取为 base64          |
| `reader.onload`                  | 文件读取成功，拿到 base64 字符串        |
| `new Image().src = base64`       | 创建 `<img>` 元素尝试加载 base64 图片 |
| `img.onload`                     | 图片加载成功，你可以：                 |

* 拿到图片宽高
* 检查是否符合规范
* 决定是否允许上传
  \| `resolve()` | 放行上传 |
  \| `reject()` | 阻止上传（配合 `Upload.LIST_IGNORE`）|

<br/> 

**🔁 结合整体逻辑**

```js
beforeUpload={(file) => {
  return this.beforeUploadImg(file).catch(() => Upload.LIST_IGNORE);
}}
```

* 如果 `beforeUploadImg()` **校验成功** → 上传继续
* 如果 `beforeUploadImg()` **校验失败** → 触发 `catch`，返回特殊值 `Upload.LIST_IGNORE`，**阻止上传**

<br/> 

**✅ 你可能会在 `img.onload` 中做什么？**

```js
if (img.width > 1024 || img.height > 1024) {
  reject(new Error('图片太大'));
} else {
  resolve();
}
```

也可以在这里压缩、画到 canvas 里等等。

---
<br/>


**📌 总结流程图**

```
用户选择图片
     ↓
beforeUploadImg(file)
     ↓
FileReader 读为 base64
     ↓
new Image() 加载该 base64 图片
     ↓
img.onload → 判断宽高、大小、格式
     ↓
满足条件 → resolve() → 上传
不满足 → reject() → Upload.LIST_IGNORE → 阻止上传
```


<br/><br/><br/>

***
<br/>
> <h1 id="Menu组件做侧边栏菜单">Menu组件做侧边栏菜单</h1>

使用 **Ant Design 的 `<Menu />`** 组件做侧边栏菜单， 点击菜单展开和选中。

---
<br/>

- **需求：**

	* 使用 `react-router-dom` 控制页面跳转
	* URL 路径可能是二级或三级（如 `/product/argusProductNetwork`）
	* 菜单结构中，部分子路径没有直接出现在菜单中
	* 你希望菜单能自动识别当前路径，**展开正确的父级**，并且**高亮选中项**

<br/> 


**✅ 菜单结构示意（假设）**

```tsx
const menuItems = [
  {
    key: '/dashboard',
    label: '首页',
  },
  {
    key: '/product',
    label: '产品管理',
    children: [
      { key: '/product/list', label: '产品列表' },
      { key: '/product/create', label: '创建产品' },
    ],
  },
  {
    key: '/user',
    label: '用户管理',
    children: [
      { key: '/user/list', label: '用户列表' },
      { key: '/user/roles', label: '角色权限' },
    ],
  },
];
```

<br/>

**✅ `specialMenu` 特殊路由映射**

这是为了解决**非菜单路由也能正确展开父级菜单**的关键：

```ts
const specialMenu: { [key: string]: string } = {
  '/product/argusProductNetwork': '/product',
  '/user/settings': '/user',
};
```

<br/>

**✅ 完整组件代码**

```tsx
import { Menu } from 'antd';
import { useLocation, useNavigate } from 'react-router-dom';
import { useEffect, useState } from 'react';

// 特殊子页面映射到其父菜单
// { [key: string]: string }： 是 TypeScript 中的 索引签名类型（Index Signature），它的意思是：
// 这个对象的键（key）必须是字符串类型，对应的值（value）也是字符串类型。
const specialMenu: { [key: string]: string } = {
  '/product/argusProductNetwork': '/product',
  '/user/settings': '/user',
};

// 菜单项定义
const menuItems = [
  {
    key: '/dashboard',
    label: '首页',
  },
  {
    key: '/product',
    label: '产品管理',
    children: [
      { key: '/product/list', label: '产品列表' },
      { key: '/product/create', label: '创建产品' },
    ],
  },
  {
    key: '/user',
    label: '用户管理',
    children: [
      { key: '/user/list', label: '用户列表' },
      { key: '/user/roles', label: '角色权限' },
    ],
  },
];

const SidebarMenu = () => {
  const location = useLocation();
  const navigate = useNavigate();

  const [openKeys, setOpenKeys] = useState<string[]>([]);
  const [selectedKeys, setSelectedKeys] = useState<string[]>([]);

  // 当路径变化时更新菜单状态
  useEffect(() => {
    const path = location.pathname;

    // 选中项：直接使用 pathname
    setSelectedKeys([path]);

    // 展开项：优先从 specialMenu 映射，找不到则取前两级路径
    const matchedOpenKey =
      specialMenu[path] || '/' + path.split('/').slice(1, 2).join('');
    setOpenKeys([matchedOpenKey]);
  }, [location.pathname]);

  // 点击菜单跳转页面
  const handleMenuClick = (e: { key: string }) => {
    navigate(e.key);
  };

  return (
    <Menu
      mode="inline"
      style={{ width: 240 }}
      items={menuItems}
      openKeys={openKeys}
      selectedKeys={selectedKeys}
      onOpenChange={(keys) => setOpenKeys(keys)}
      onClick={handleMenuClick}
    />
  );
};

export default SidebarMenu;
```

<br/>

**✅ 路由示例（React Router 配置）**

```tsx
<BrowserRouter>
  <SidebarMenu />
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/product/list" element={<ProductList />} />
    <Route path="/product/create" element={<ProductCreate />} />
    <Route path="/product/argusProductNetwork" element={<SpecialProduct />} />
    <Route path="/user/list" element={<UserList />} />
    <Route path="/user/settings" element={<UserSettings />} />
  </Routes>
</BrowserRouter>
```

<br/>

**✅ 效果说明**

| 当前路径                           | 展开菜单                         | 选中菜单项                          |
| ------------------------------ | ---------------------------- | ------------------------------ |
| `/product/list`                | `/product`                   | `/product/list`                |
| `/product/argusProductNetwork` | `/product`（通过 `specialMenu`） | `/product/argusProductNetwork` |
| `/user/settings`               | `/user`（通过 `specialMenu`）    | `/user/settings`               |

<br/> 

✅ 补充建议

* 如果你后期有**动态菜单数据（后端返回）**，也可以结合路由和菜单 key 做匹配
* 对于**路径参数**如 `/user/detail/123`，可使用正则或 startsWith 来模糊匹配菜单项
* 如果使用的是 `antd@5`，可以考虑新的 `items` API + `useMemo` 优化


<br/><br/><br/>

***
<br/>
> <h1 id="加载组件Spin">加载组件Spin</h1>


```tsx
<Spin spinning={loading} style={{ height: 'calc(100vh - 88px)' }} />
```

是使用了 Ant Design 中的 **加载中动画组件 `<Spin>`**，结合 React 状态 `loading` 来显示一个 **全屏的加载指示器**。我们来逐个详细解释：

<br/>

✅ **1.`<Spin />` 是 Ant Design 的 加载中状态组件**，通常用于网络请求、异步操作时显示“旋转的小圆圈”，表示“数据加载中”。

官方文档地址（含示例）：
📎 [https://ant.design/components/spin-cn/](https://ant.design/components/spin-cn/)

<br/>

- **✅ 2.`spinning={loading}` 用法：**
	
	* `loading === true` 时：显示转圈动画；
	* `loading === false` 时：不显示。

`loading` 通常是 React 里的状态变量，比如：

```tsx
const [loading, setLoading] = useState(true);
```

当你完成数据加载后：

```tsx
setLoading(false);
```

<br/>

**✅ 3.`style={{ height: 'calc(100vh - 88px)' }}` 是什么意思？**

这是给 `<Spin />` 设置了一个内联样式：

```css
height: calc(100vh - 88px);
```

- **📐 解释：**

	* `100vh` 表示 **浏览器可视窗口的高度**；
	* `88px` 是某个顶部元素的高度（比如导航栏、高度为 88px）；
	* 所以这个计算是：让 `<Spin>` 的高度等于 **除去顶部栏之后的剩余高度**，以便让 loading 居中展示在“有效页面内容区域”中。

<br/>

**✅ 4.实际效果示意：**

如果你页面上方有个固定头部（比如 `<Header>` 高度 88px），那么这个 `<Spin>` 会填满剩下的内容区域并居中显示转圈：

```
┌────────────── 88px Header ──────────────┐
│                                          │
├───────────── <Spin /> 居中 ─────────────┤
│                                          │
│        ⏳ Loading... 数据加载中...        │
│                                          │
└──────────────────────────────────────────┘
```

<br/>

**✅ 5.使用示例（完整）**

```tsx
import { Spin } from 'antd';
import { useEffect, useState } from 'react';

export default function Page() {
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 模拟接口请求
    setTimeout(() => {
      setLoading(false);
    }, 2000);
  }, []);

  return (
    <Spin
      spinning={loading}
      style={{ height: 'calc(100vh - 88px)', display: 'flex', alignItems: 'center', justifyContent: 'center' }}
    >
      {!loading && <div>页面内容加载完成</div>}
    </Spin>
  );
}
```

<br/>

**✅ 如果你想更美观些，还可以使用嵌套方式：**

```tsx
<Spin spinning={loading}>
  <YourContentComponent />
</Spin>
```

这表示：`loading === true` 时，`YourContentComponent` 会被模糊遮罩并显示加载中动画。


<br/><br/><br/>

***
<br/>
> <h1 id="骨架屏Skeleton组件">骨架屏Skeleton组件</h1>

`<Skeleton />` 是 Ant Design 提供的一个 **骨架屏（Skeleton Screen）组件**，用于在数据加载过程中，展示“灰色占位图”，提升用户的视觉体验。

<br/>

✅ 一句话定义：

> **Skeleton 是一种“占位图”组件，用于在数据未加载完成时，先显示结构轮廓，等数据加载完成后再替换为真实内容。**

<br/>

- **✅ Skeleton 的好处：**

	* 让用户感觉页面结构已经准备好；
	* 减少“白屏”感；
	* 提前预览页面骨架，提高感知性能；
	* 常见于：**社交流、文章列表、卡片、图文内容加载**

<br/>

 **✅ 最基础：**

```tsx
import { Skeleton } from 'antd';

<Skeleton active />
```

* `active`: 是否展示动画
* 默认显示三行段落

<br/>

**页面加载数据时使用**

```tsx
const [loading, setLoading] = useState(true);

useEffect(() => {
  setTimeout(() => setLoading(false), 2000); // 模拟请求
}, []);

return (
  <div>
    {loading ? (
      <Skeleton active />
    ) : (
      <div>
        <h3>真实标题</h3>
        <p>真实内容段落</p>
      </div>
    )}
  </div>
);
```

***
<br/>

 ✅ 进阶用法：自定义头像、标题、段落

```tsx
<Skeleton
  avatar={{ size: 'large' }}
  title={{ width: '50%' }}
  paragraph={{ rows: 3 }}
  active
/>
```

| 属性          | 类型        | 说明       |               |
| ----------- | --------- | -------- | ------------- |
| `avatar`    | \`boolean | object\` | 是否显示头像        |
| `title`     | \`boolean | object\` | 是否显示标题（可设宽度）  |
| `paragraph` | \`boolean | object\` | 是否显示段落（行数、宽度） |

<br/>

 **✅ 配合数据使用：Skeleton.Button / Skeleton.Input / Skeleton.Image 等**

```tsx
<Skeleton.Button active size="large" shape="round" />
<Skeleton.Input active size="default" style={{ width: 200 }} />
<Skeleton.Image style={{ width: 200 }} />
```

<br/>

 **✅ 搭配 `loading` 的语法糖：`<Skeleton loading={loading} >...</Skeleton>`**

```tsx
<Skeleton loading={loading} active>
  <Card title="真实数据" />
</Skeleton>
```

* loading 为 true：显示骨架；
* loading 为 false：显示 children 内容。

<br/>

 **✅ 配合列表使用（Skeleton.List）**

```tsx
{loading
  ? Array.from({ length: 3 }).map((_, i) => <Skeleton key={i} active avatar paragraph={{ rows: 2 }} />)
  : realList.map(item => <ListItem key={item.id} data={item} />)}
```


<br/>

**✅ 总结：**

> `Skeleton` 是一个用于“等待数据加载”期间的占位组件，让用户**更快感知页面结构**，比单纯的 `<Spin>` 动画更优雅，广泛用于内容型产品和后台系统页面。



<br/><br/><br/>

***
<br/>
> <h1 id="标签组件Tag">标签组件Tag</h1>

 `<Tag>` 标签组件用于展示简洁的 **状态、分类、标签内容、标识性信息** 等内容，广泛用于后台系统的状态标记、文章分类、标签管理等场景。

<br/>

**✅ 一、Tag 是什么？**

> `<Tag>` 是 Ant Design 的轻量级标签组件，支持不同颜色、关闭操作、图标、可选择等功能。

📎 官方文档地址：[https://ant.design/components/tag-cn/](https://ant.design/components/tag-cn/)

<br/>

**✅ 二、Tag 的常见用法**

**✅ 基础使用：**

```tsx
import { Tag } from 'antd';

<Tag>默认标签</Tag>
<Tag color="magenta">粉色</Tag>
<Tag color="green">绿色</Tag>
<Tag color="#f50">自定义颜色</Tag>
```

<br/>

**✅ 可关闭标签（带 x 关闭按钮）：**

```tsx
<Tag closable onClose={() => console.log('标签被关闭')}>
  可关闭标签
</Tag>
```

> 提示：被关闭的标签不会自动消失，你需要在父组件中删除对应数据项。

<br/>

**✅ 循环渲染多个标签：**

```tsx
const tags = ['React', 'Vue', 'Angular'];

{tags.map(tag => (
  <Tag key={tag}>{tag}</Tag>
))}
```

<br/>

**✅ 标签颜色状态用途：**

Ant Design 提供了一些语义化颜色（默认内置），如：

* `success` ✅ 成功
* `processing` 🔄 处理中
* `error` ❌ 错误
* `warning` ⚠️ 警告
* `default` 默认灰色

```tsx
<Tag color="success">已完成</Tag>
<Tag color="error">失败</Tag>
<Tag color="warning">待处理</Tag>
```

<br/>

**✅ 动态选择标签（CheckableTag）**

```tsx
import { Tag } from 'antd';
const { CheckableTag } = Tag;

const options = ['Apple', 'Banana', 'Orange'];
const [selectedTags, setSelectedTags] = useState<string[]>([]);

return (
  <>
    {options.map(tag => (
      <CheckableTag
        key={tag}
        checked={selectedTags.includes(tag)}
        onChange={checked => {
          const next = checked
            ? [...selectedTags, tag]
            : selectedTags.filter(t => t !== tag);
          setSelectedTags(next);
        }}
      >
        {tag}
      </CheckableTag>
    ))}
  </>
);
```

✅ **场景**：多选标签、文章标签筛选、兴趣偏好等。

<br/>

**✅ 标签中添加图标：**

```tsx
import { Tag } from 'antd';
import { SmileOutlined } from '@ant-design/icons';

<Tag icon={<SmileOutlined />} color="success">
  带图标的标签
</Tag>
```

<br/>

**✅ 标签组 + 展示控制：**

```tsx
<Tag.Group>
  <Tag color="blue">开发</Tag>
  <Tag color="gold">测试</Tag>
  <Tag color="lime">上线</Tag>
</Tag.Group>
```

<br/>

**✅ 三、在表格中使用 Tag（状态展示经典场景）**

```tsx
const columns = [
  {
    title: '状态',
    dataIndex: 'status',
    render: (text) => {
      let color = text === '完成' ? 'green' : text === '异常' ? 'red' : 'blue';
      return <Tag color={color}>{text}</Tag>;
    },
  },
];
```

<br/>

 **✅ 四、自定义颜色支持**

你可以使用任意颜色（16进制或颜色名）：

```tsx
<Tag color="#87d068">自定义颜色</Tag>
```

<br/>

 **✅ 五、常见场景总结：**

| 场景       | 示例                                |
| -------- | --------------------------------- |
| 状态标记     | <Tag color="success">成功</Tag>     |
| 类别标识     | <Tag color="blue">前端</Tag>        |
| 标签选择     | <CheckableTag>可选标签</CheckableTag> |
| 可关闭的动态标签 | <Tag closable>移除我</Tag>           |
| 数据渲染配合   | 表格、卡片、表单内标注                       |


<br/><br/><br/>

***
<br/>
> <h1 id="Radio.Group单选按钮">Radio.Group单选按钮</h1>

```jsx
import { Form, Radio } from 'antd';

const typeOptions = [
  { label: '个人用户', value: 'person' },
  { label: '企业用户', value: 'company' },
];

const handleFormChange = (e) => {
  console.log('你选中了:', e.target.value);
};

export default function Demo() {
  return (
    <Form>
      <Form.Item name="type" label="用户类型">
        <Radio.Group
          name="type"
          options={typeOptions}
          onChange={handleFormChange}
          optionType="button"
          buttonStyle="solid"
        />
      </Form.Item>
    </Form>
  );
}
```

- **✅ 效果：**
	* 会渲染两个按钮【个人用户】【企业用户】；
	* 点按钮后会在控制台输出选中的值；
	* 表单提交时，字段 `type` 对应当前选中值（如 `'company'`）；

<br/>

**1.`name="type"`**

* **作用**：字段的名称，用于标识这个输入字段的 key。
* **使用场景**：当这个 `Radio.Group` 被放进 `<Form.Item name="type">` 里时，它就会自动对应这个字段。

🔸 示例：

```jsx
<Form.Item name="type" label="类型">
  <Radio.Group name="type" options={...} />
</Form.Item>
```

<br/>

 **2.`options={typeOption}`**

* **作用**：快速生成一组单选按钮，而不用手动写 `<Radio>`。
* **格式要求**：`typeOption` 必须是一个数组，数组中每项是一个对象，常见结构如下：

```js
const typeOption = [
  { label: '苹果', value: 'apple' },
  { label: '香蕉', value: 'banana' },
  { label: '橘子', value: 'orange', disabled: true }, // 可禁用某项
];
```

* **效果**：会自动生成对应的按钮。

<br/>

**3.`onChange={handleFormChange}`**

* **作用**：当选择的单选项发生变化时，触发这个回调。
* **事件对象结构**：

```js
const handleFormChange = (e) => {
  console.log('当前选中的值:', e.target.value);
};
```

* **常用场景**：当选中某项后，动态控制其他表单字段的显示/隐藏、联动等。

<br/>

 **4.`optionType="button"`**

* **作用**：改变单选按钮的展示样式，渲染为按钮风格，而不是传统的圆圈选项。

* **取值**：

  * `"default"`：传统样式（圆圈）
  * `"button"`：按钮样式 ✅（你现在用的）

* **对比图**：

| optionType | 效果                                                                |
| ---------- | ----------------------------------------------------------------- |
| default    | ![默认圆圈样式](https://ant.design/components/radio/img/radio.svg)      |
| button     | ![按钮样式](https://ant.design/components/radio/img/radio-button.svg) |

<br/> 

5. **`buttonStyle="solid"`**

* **作用**：控制按钮选中时的样式。
* **取值**：

  * `"solid"`：选中后填充背景（强调）
  * `"outline"`：选中后只是边框加粗（不填充）

🔸 示例区别：

```jsx
<Radio.Group buttonStyle="solid" />
// vs
<Radio.Group buttonStyle="outline" />
```



