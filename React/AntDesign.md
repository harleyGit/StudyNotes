> <h1 id=""></h1>
- [**阿里开源ReactUI组件库AntDesign**](#阿里开源ReactUI组件库AntDesign)
	- [Table组件使用](#Table组件使用)
		- [pagination、expandable属性使用](#pagination、expandable属性使用)
			- [expandable中常用属性：columnWidth、expandIcon](#expandable中常用属性：columnWidth、expandIcon)
	- [下拉组件Select](#下拉组件Select)
	- [加载指示器组件Spin](#加载指示器组件Spin)
	- [模态框组件-Modal](#模态框组件-Modal)
	- [弹框提示组件message](#弹框提示组件message)
	- [Form表单](#Form表单)



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
> <h2 id="下拉组件Select">下拉组件Select</h2>

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

<br/>

**用法示例**

### 1. 引入组件

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
> <h2 id="加载指示器组件Spin"> 加载指示器组件Spin</h2>

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


***
<BR/><BR/><BR/>
> <H2 ID="Form表单">Form表单</H2>

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


