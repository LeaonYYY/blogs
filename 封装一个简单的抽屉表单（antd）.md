# 小组件：封装一个简单的抽屉表单（antd）

2023-3-23

在做毕设的时候大部分功能都需要进行新增和修改操作，例如添加宿舍、调整宿舍等

初步的想法是两种功能共用同一个表单，添加和修改差别只在于是否有表单初始值，然后通过状态提升，把方法和数据都放在这个表单组件的父组件上。后面发现有其他功能也需要这个表单，所以把表单的内容也作为组件的props传入封装的组件。

## 使用须知

如果项目没有引入antd的话，需要先引入antd

[antd官网](https://ant.design/index-cn)

## 组件代码

```TSX
import { Drawer, Form, Button } from 'antd';
import '@wangeditor/editor/dist/css/style.css'; // 引入 css

// 表单item的配置
export interface FormFieldType {
  id: number;
  name: string; // 字段名
  label: string; // 标签
  component?: any; // 表单组件
  hidden?: boolean; // 是否隐藏
}

interface DrawerFormProps {
  type?: 'edit' | 'add'; // 表明该表格的用途
  title?: string; // 抽屉标题
  isOpen: boolean; // 控制抽屉是否可见
  formFields: FormFieldType[]; // 表单内容
  loading?: boolean;
  initialData?: any; // 表单初始化数据
  handleDrawerClose: () => void; // 关闭抽屉
  handleSubmit: (formData: any) => void; // 提交表单
}

const DrawerForm: React.FC<DrawerFormProps> = ({
  isOpen,
  title,
  formFields,
  loading,
  initialData,
  handleDrawerClose,
  handleSubmit,
}) => {
  return (
    <Drawer
      open={isOpen}
      onClose={handleDrawerClose}
      title={title}
      width="100%"
      destroyOnClose
    >
      <Form onFinish={handleSubmit} initialValues={initialData || {}}>
        {formFields.map((item: FormFieldType) => {
          return (
            <Form.Item
              key={item.id}
              name={item.name}
              label={item.label}
              hidden={item.hidden ?? false}
            >
              {item.component}
            </Form.Item>
          );
        })}
        <Form.Item
          style={{
            textAlign: 'center',
          }}
        >
          <Button htmlType="submit" type="primary" loading={loading}>
            保存
          </Button>
        </Form.Item>
      </Form>
    </Drawer>
  );
};
export default DrawerForm;

```