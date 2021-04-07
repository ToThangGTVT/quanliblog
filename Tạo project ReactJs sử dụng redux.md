Từng là một sinh viên mới tốt nghiệp ra trường, tôi đã bị ném thẳng vào 1 dự án frontend đầy dãy những ma thuật thần kì. Tôi đã tìm đủ mọi cách để có thể hiểu được Redux... nhưng mọi thứ có vẻ cần khá mơ hồ.

Tôi nhận ra để có thể bắt đầu với Redux bạn cần phải hiểu những điều sau:
*  `State` là gì mà mỗi dev front-end cần biết về nó
* Làm thế nào mà redux có thể `quản lý state`
* `Actions`, `reducers`... bọn này là clgt ???
* Làm thế nào để `tích hợp` redux vào project react của bạn
* Nó có `ưu điểm` và `nhược điểm` gì

Nếu bạn đã từng gặp khó khăn khi bắt đầu tiếp cận Redux, Redux Thunk thì đây là một ví dụ giúp bạn hiểu được những thư viện này làm gì, nó giúp ích những gì và khi nào ta nên sử dụng chúng.

## 1. State là gì

Vì redux là một thư viện quản lý state do vậy ta cần phải hiểu state là gì.
> Theo cá nhân tôi, các đối tượng trên một trang web có rất nhiều dạng hiển thị. Như một button có thể có màu sắc khác nhau, nội dung khác nhau. Và để biểu thị cho từng trạng thái này ta cần 1 thứ gọi là `state`

Cụ thể hơn, trong javascript state là một object. 

```js
const state = {}
```
Bên cạnh đó ta có thể thêm các thuộc tính cho nó.
```js
const initialState = {
  posts: [],
  buttonDelete: {
    show: false
  }
}
```
Khi sử dụng một số framework như React ta có thể chỉ định Button Xóa này có được hiển thị hay không
```jsx
<div className={this.state.buttonDelete.show ? '' : 'hidden'}>
  Delete
</div>
```
Khi `this.state.buttonDelete.show` nhận giá trị `false` React sẽ thêm class `hidden` vào Button và từ đó button và nó sẽ nhận giá trị `display: none` và ngược lại khi`this.state.buttonDelete.show` nhận giá trị `true` thì React sẽ xóa class đó đi và Button Delete được hiển thị

Khi ta sử dụng javascript thuần để thực hiện điều này thì sao, nó sẽ trông như thế này:

```js
const buttonDelete = {
  show: false
}

if (buttonDelete.show === false) {
  document.querySelector('div').classList.add('hidden')
} else {
  document.querySelector('div').classList.remove('hidden')
}
```
Về mặt kĩ thuật, ở ví dụ trên ta vẫn sử dụng state là hằng số `buttonDelete` nhưng bạn có thể nhận ra rằng, mọi thứ đã gọn gàng hơn khi ta sử dụng React vì nó được tự động triển khai các class dựa trên `state` của chính nó.

Việc sử dụng các framework statefull như React, Angular hay Vue làm code gọn gàng, và cho phép ta tập trung vào một việc duy nhất đó là dữ liệu ảnh hưởng ra sao đến ứng dụng phía người dùng.
 
Nói chung state tham chiếu đến điều kiện của một thứ gì đó ở thời điểm cụ thể và chi phối nó, ví dụ như điều kiện hiển thị của một nút xóa.

> Tôi thích việc hiểu state giống như một `object` javascript, nó được một DOM tham chiếu đến và thông qua nó để biểu thị các trạng thái khác nhau

Việc thay đổi `object` này sẽ gây ra việc thay đổi DOM tự động trên giao diện phía người dùng

Đúng với cái tên React (phản ứng) ở đây DOM thực sự phản ứng lại với state khi state có sự thay đổi (theo mình hiểu là thế 😂)

Từ từ khoan, trên tài liệu của React còn có 1 thứ nữa đó là `props`
## 2. Props là gì

Tương tự với `state`, `props` trong react cũng là một đối tượng của javascript, nó còn là cách viết ngắn gọn properties. Hiểu một cách đơn giản, nó lưu trữ các giá trị `attribute` của HTML tag

Ví dụ:
```jsx
function Wall() {
  return (
    <div>
      <Greeting name="Nathan" age={27} occupation="Software Developer" />
    </div>
  );
}
```
Trong ví dụ component `Greeting` có 3 props đó là `name`, `age`, `occupation`. Bạn có thắc mắc gì về chúng ko? Tại sao lại sử dụng những props này mà ko phải là `state`. Khi tôi viết như này có thể bạn sẽ hiểu tại sao lại sử dụng `props`

```jsx
function Wall({profile}) {
  return (
    <div>
      <Greeting name={profile.name} age={profile.age} occupation={profile.carrier} />
    </div>
  );
}
```
Và ở một nơi khác
```jsx
function App() {
  return (
    <Wall profile={{name:"Tô", age: 23, carrier:"DEV"}}></Wall>
  );
}
```
Như bạn thấy, chúng ta đã truyền data từ component cha xuống component con. Và đây cũng chính là ứng dụng chủ yếu của props.

Một câu hỏi đặt ra là ta truyền ngược data từ component con lên component cha như nào?


Ở component cha:
```jsx
class Parent extends React.Component {
   state = { message: "parent message" }
   callbackFunction = (childData) => {
       this.setState({message: childData})
   },
   render() {
        return (
            <div>
                 <Child parentCallback = {this.callbackFunction}/>
                 <p> {this.state.message} </p>
            </div>
        );
   }
}
```
Ở component con:
```jsx
class Child extends React.Component{
    sendBackData = () => {
         this.props.parentCallback("child message");
    },
    render() { 
       <button onClick={sendBackData}>click me to send back</button>
    }
};
```
Ở đây ta đã truyền 1 function cho props ở phía component cha,
và ở phía component con ta gọi props đó và truyền value. Lúc này value ở component con sẽ thông qua function truyền đến component cha.

Đây là một cách có thể giải quyết được vấn đề truyền data từ component con đến component cha. Nhưng thử tưởng tượng, trong project của bạn có rất nhiều component, các component phân chia tầng tầng lớp lớp với nhau. Khi đó việc truyền data qua lại giữa các component là 1 điều khủng khiếp :((

![image](https://user-images.githubusercontent.com/33534455/113906382-62ccac00-97fe-11eb-869b-4e8ea6898cbe.png)
> Nó sẽ trông như này

Và từ đó redux ra đời...

**Bắt đầu:** Chia cấu trúc project thành các thư mục nhỏ: ‘**store, reducer, action**’ và trong mỗi thư mục nên có một fie index.js
**Cài đặt:** *npm i react-redux –save | npm i redux --save*

trong trường hợp này ta đã sử dụng context để đặt store là globle data cho toàn ứng dụng

```jsx
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

Và import Provider và store:

```jsx
import { Provider } from 'react-redux'
import store from './store/index'
```

**Trong thư mục store:** 

> store là một nơi để chứa toàn bộ state của ứng dụng. Nó nhận đầu vào  
> là một reducer. Reducer đưa state vào trong store để store quản lý

```jsx
import { createStore } from "redux";
import reducer from "../reduces";

export const store = createStore(reducer);
export default store;
```

Trong thư mục reducer: phần này có tác dụng nhận action và trả về state tương ứng với action đã khai báo (chỉ tạo ra bản sao của state chứ ko được sửa trực tiếp giá trị của state)

```jsx
export default function reducer(state = [], action) {
  return [
    ...state,
    {
      name: action.tech
    }
  ]
}
```

**Khai báo 1 action cơ bản:** action nên có 1 trường type để reducer nhận ra và xử lý state tương ứng với action

```jsx
export const addClick = tech => ({
  type: 'ADD_TODO',
  tech
})
```

**Bắt đầu sử dụng redux:**
Đây là button có tác dụng đưa value từ text box vào store để redux quản lý. 

> Connect là hàm có chức năng kết nối component với redux để value của
> text có thể truyền vào store và nhận state mới từ redux nếu có sự thay
> đổi từ store để render giá trị mới vào component

```jsx
import React, { Component } from 'react';
import { store } from '../store/index'
import { addClick } from '../action'
import { connect } from 'react-redux';

class Button extends Component {

  constructor(props) {
    super(props)
    this.myRef = React.createRef()
  }

  handleClick = () => {
    this.props.dispathButton(this.myRef.value)
  }

  render() {
    return (
      <div>
        <input ref={(input) => { this.myRef = input }}></input>
        <button onClick={this.handleClick}>Add</button>
      </div>
    )
  }
}

const mapDispatchToProps = () => {
  return {
    dispathButton: (val) => { store.dispatch(addClick(val)) }
  }
}

export default connect(mapDispatchToProps)(Button);
```


Và ở Component khác ta sẽ nhận state mới khi mà store có sự thay đổi:

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';

class List extends Component {

  render() {
    const { item } = this.props
    if (typeof item === 'undefined') {
      return (<div></div>)
    } else {
      return (
        <div>
          {
            item.map(el => <li>{el.name}</li>)
          }
        </div>
      )
    }
  }
}

const mapStateToProps = (state) => {
  return {
    item: state
  }
}

export default connect(mapStateToProps)(List)
```

**Lưu ý:**

```jsx
export default connect(mapStateToProps, mapDispatchToProps)(Paging)
```

**<u>mapStateToProps</u>** và **mapDispatchToProps** phải đúng thứ tự
