
```
// initState out side
const appState = {
  title: {
    text: 'redux learing',
    color: 'red',
  },
  content: {
    text: 'redux learing content',
    color: 'blue',
  }
}

//  更新视图函数
const renderApp = (newAppState, oldAppState ={}) => {
  if (newAppState === oldAppState) return
  console.log('render app')
  renderTitle(newAppState.title, oldAppState.title)
  renderContent(newAppState.content, oldAppState.content)
}

function renderTitle (newTitle, oldTitle = {}) {
  if (newTitle === oldTitle) return // 数据没有变化就不渲染了
  console.log('render title...')
  const titleDOM = document.getElementById('title')
  titleDOM.innerHTML = newTitle.text
  titleDOM.style.color = newTitle.color
}

function renderContent (newContent, oldContent = {}) {
  if (newContent === oldContent) return // 数据没有变化就不渲染了
  console.log('render content...')
  const contentDOM = document.getElementById('content')
  contentDOM.innerHTML = newContent.text
  contentDOM.style.color = newContent.color
}


// redux 

// 创建总store，初始化state，增加listeners，把所有从外部通过subscriber传入的事件函数推入队列当中。
/*
*   暴露三个方法对象
*   1: getState用于获取所有的state
*   2: dispatch用于唤起更改state的reducer
*   3: subscribe用于store的事件监听函数
*/
const createStore = (reducer) => {
  let state = null;
  const listeners = [];
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state;
  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach((listener) => listener())
  }
  dispatch({});
  return {
    getState,
    dispatch,
    subscribe
  }
}

const themeReducer = (state, action) => {
  if (!state) return {
    themeName: 'Red Theme',
    themeColor: 'red',
  }
  switch (action.type) {
    case 'UPDATE_THEME_NAME':
      return {
        ...state,
        themeName: action.themeName,
      }
    case 'UPDATE_THEME_COLOR':
      return {
        ...state,
        themeColor: action.themeColor,
      }
    default:
      return state
  }
}
const reducer = (state, action) => {
  if (!state) {
    return {
      title: {
        text: 'redux learing',
        color: 'red',
      },
      content: {
        text: 'redux learing content',
        color: 'blue',
      }
    }
  }
  switch (action.type) {
    case 'UPDATE_TITLE_TEXT':
      return {
        ...state,
        title: {
          ...state.title,
          text: action.text
        }
      }
      break
    case 'UPDATE_TITLE_COLOR':
      return {
        ...state,
        titile: {
          ...state.title,
          color: action.color
        }
      }
      break
    default:
      return state
  }
}


const store = createStore(appState, reducer)
let oldState = store.getState()
store.subscribe(() => {
  const newState = store.getState();
  renderApp(newState, oldState)
  oldState = newState
}) 

renderApp(store.getState());
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: '《React.js 小书》' })
store.dispatch({ type: 'UPDATE_TITLE_TEXT', text: '《React.js 东北书' })
```
