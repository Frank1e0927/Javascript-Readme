# rc-form

> antd是我最喜欢的UI组件类库，这篇文章粗略分析rc-form的部分源码，用作个人对业务工程组件设计抽象的学习，不建议查看，但是喜欢的可以随意点赞。

#### 对于ant-design使用form时的疑问
- 包裹了{ getFieldDecorator } 的组件是如何进行数据收集的
- 包裹了{ getFieldDecorator } 的组件是如何进行数据校验的
- 包裹了{ getFieldDecorator } 的组件是对传入组件校验时是如何UI反馈的（红框，文字提示）


##### 1.对rc-form的example例子打开，找到getFieldDecorator.js文件夹看到以下代码


```
import { createForm } from 'rc-form';

class Form extends React.Component {
  static propTypes = {
    form: PropTypes.object,
  };

  componentWillMount() {
    this.nameDecorator = this.props.form.getFieldDecorator('name', {
      initialValue: '',
      rules: [{
        required: true,
        message: 'What\'s your name?',
      }],
    });
  }

  onSubmit = (e) => {
    e.preventDefault();
    this.props.form.validateFields((error, values) => {
      if (!error) {
        console.log('ok', values);
      } else {
        console.log('error', error, values);
      }
    });
  };

  onChange = (e) => {
    console.log(e.target.value);
  }

  render() {
    const { getFieldError } = this.props.form;

    return (
      <form onSubmit={this.onSubmit}>
        {this.nameDecorator(
          <input
            onChange={this.onChange}
          />
        )}
        <div style={{ color: 'red' }}>
          {(getFieldError('name') || []).join(', ')}
        </div>
        <button>Submit</button>
      </form>
    );
  }
}

const WrappedForm = createForm()(Form);
ReactDOM.render(<WrappedForm />, document.getElementById('__react-content'));
```
从最底部开始看，明显看到了这个FormComponent被creactForm()这个函数包裹起来了，可以判断出来createForm()是一个HOC，所以，我们可以看到在willmount阶段，props中有一个form字段属性，并且提供了getFieldDecorator这个函数。而这个函数又给了this.nameDecorator的组件内部属性最后在render函数的JSX里面，用到了这个this.nameDecorator，并且这是一个函数，参数是普通组件类型。从而可以推测出this.props.form.getFieldDecorator()这个函数是一个curry function。又返回出一个新的函数供外部调用。

以上的所有api都应该来自createForm返回的form对象里面的属性，那么我们继续往下走，看看creatForm来自的文件，是在src下的createForm.js文件，打开之前我以为是应该很大一坨代码的，果然，跟我想的不一样哈哈来看代码


```
import createBaseForm from './createBaseForm';

export const mixin = {
  getForm() {
    return {
      getFieldsValue: this.fieldsStore.getFieldsValue,
      getFieldValue: this.fieldsStore.getFieldValue,
      getFieldInstance: this.getFieldInstance,
      setFieldsValue: this.setFieldsValue,
      setFields: this.setFields,
      setFieldsInitialValue: this.fieldsStore.setFieldsInitialValue,
      getFieldDecorator: this.getFieldDecorator,
      getFieldProps: this.getFieldProps,
      getFieldsError: this.fieldsStore.getFieldsError,
      getFieldError: this.fieldsStore.getFieldError,
      isFieldValidating: this.fieldsStore.isFieldValidating,
      isFieldsValidating: this.fieldsStore.isFieldsValidating,
      isFieldsTouched: this.fieldsStore.isFieldsTouched,
      isFieldTouched: this.fieldsStore.isFieldTouched,
      isSubmitting: this.isSubmitting,
      submit: this.submit,
      validateFields: this.validateFields,
      resetFields: this.resetFields,
    };
  },
};

function createForm(options) {
  return createBaseForm(options, [mixin]);
}

export default createForm;

```

感觉有点像多重继承的概念，但是其实并不是多继承，其实看到对象名称mixin也是顾名思义，就是对原方法或者原函数注入一些方法别名。

看到下面的creactForm的确是返回了一个新的函数出来，这里的options应该就是先前的Form组件了，第二个参数就是mixin注入的别名方法。然后跟踪createBaseForm继续往下走

找到createBaseForm一打开，代码有点长，500左右，先缩小展示一下


```
function createBaseForm(options = {}, mixins = []) {
    const {
        validateMessages,
        onFieldsChange,
        onValuesChange,
        mapProps = identity,
        mapPropsToFields,
        fieldNameProp,
        fieldMetaProp,
        fieldDataProp,
        formPropName = 'form',
        // @deprecated
        withRef,
    } = option;
    
    return function decorate(WrappedComponent) {
        const Form = createReactClass({...})
        
        return argumentContainer(Form, WrappedComponent)
    }
}

export default createBaseForm
```

上面代码可以看到，createBaseForm其实也是返回一个新的函数出来。
以上代码归根结底可以变成

![image](https://note.youdao.com/yws/public/resource/5ec653a1c440bd78178c6940b4bff4e5/xmlnote/D9C40D3593334BB5B905697C84620353/6460)


```
return argumentContainer(Form, WrappedComponent)基本不用管
```
此处直接返回了的是一个剔除了静态方法之后的Component,用Form包裹住WrappedComponent.

马上来到几个问题的核心函数getFieldDecorator了。

我们先看一下在antd3.0里面，Form的getFieldDecorator是怎么用的


```
<FormItem
    {...formItemLayout}
        label="Select"
        hasFeedback
>
    {getFieldDecorator('select', {
    rules: [
        { required: true, message: 'Please select your country!' },
    ],
    })(
        <Select placeholder="Please select a country">
            <Option value="china">China</Option>
            <Option value="use">U.S.A</Option>
        </Select>
    )}
</FormItem>
```

我们可以看到此处getFieldDecorator接受两个参数，一个是fieldName,一个是对象，对应到createBaseForm里面的函数代码


```
    getFieldDecorator(name, fieldOption) {
        const props = this.getFieldProps(name, fieldOption);
        return (fieldElem) => {
          const fieldMeta = this.fieldsStore.getFieldMeta(name);
          const originalProps = fieldElem.props;
          if (process.env.NODE_ENV !== 'production') {
            //  ..省略部分代码
          }
          fieldMeta.originalProps = originalProps;
          fieldMeta.ref = fieldElem.ref;
          return React.cloneElement(fieldElem, {
            ...props,
            ...this.fieldsStore.getFieldValuePropValue(fieldMeta),
          });
        };
    },
```
很明显，这个函数返回了一个匿名函数，并且又返回出了一个React.cloneElement()关于cloneElement，很多同学在做一些普通的页面的时候都会比较少用到，但是做一些通用组件时，也是经常会用到的，甩一张代码，以供理解

```
React.cloneElement(
  element,
  [props],
  [...children]
)

```
Clone and return a new React element using element as the starting point. The resulting element will have the original element’s props with the new props merged in shallowly. New children will replace existing children. key and ref from the original element will be preserved.

简单来说，cloneElement会将原来传入的element重新替换成新的reactElement，并且传入新的props。