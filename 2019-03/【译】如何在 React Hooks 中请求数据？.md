# 如何在 React Hooks 中请求数据？

> 原文：<https://www.robinwieruch.de/react-hooks-fetch-data/>
>
> 原作者：[Robin Wieruch](https://disqus.com/by/rwieruch/)

在本文中，我将会向你展示**在React中怎样用Hooks来获取数据**通过使用[state](https://reactjs.org/docs/hooks-state.html)和[effect](https://reactjs.org/docs/hooks-effect.html) hooks。我们将用众所周知的[Hacker News API](https://hn.algolia.com/api)来获取科技界的热门文章。你也可以实现获取数据的自定义hook，在应用的任何位置复用，也可以作为独立的依赖包在npm上发布。

如果你对这个React的新功能一无所知，请查看我的另一篇文章 [introduction to React Hooks](https://www.robinwieruch.de/react-hooks/)。如果你想查看直接查看文章的示例，请查看此[Github仓库](https://github.com/the-road-to-learn-react/react-hooks-introduction)。

**提示**：在将来的版本中，React Hooks不适用于在React中获取数据。取而代之的是一个叫做`Suspense`的功能。尽管如此，下面的练习依然是了解 state 和 effect 两种 Hooks 的好方法。

## 使用 React Hooks 获取数据

如果你不熟悉在React中获取数据，可以阅读我的文章：[How to fetch data in React](https://link.juejin.im/?target=https%3A%2F%2Fwww.robinwieruch.de%2Freact-fetching-data%2F)。文章将讲解如何使用class components获取数据，如何复用[Render Prop Components](https://www.robinwieruch.de/react-render-props-pattern/)和[Higher-Order Components](https://www.robinwieruch.de/gentle-introduction-higher-order-components/)，以及如何进行错误处理和 loading 状态。在本文中，我会在function components中使用React Hooks来重新实现这些功能。

```javascript
import React, { useState } from 'react';

function App() {
  const [data, setData] = useState({ hits: [] });

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

App 组件将展示一个列表，hits列表信息来自 Hacker News articles 。状态和状态更新函数将通过 `useState` 的hook来生成，它将负责管理hits列表数据的本地状态。初始状态是一个空数组，此时还没有为其设置任何的状态。

我们将使用[axios](https://github.com/axios/axios)来获取数据，当然你也可以使用其他的库或者浏览器的原生fetch API，如果你还没安装axios，你可以在命令行使用`npm install axios`来安装。然后来实现用于数据获取的effect hook：

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(async () => {
    const result = await axios(
      'http://hn.algolia.com/api/v1/search?query=redux',
    );

    setData(result.data);
  });

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

通过axios调用API在`useEffect`这个 effect hook中获取数据，然后通过setData将数据放到组件本地的state中，然后通过async/await来处理Promise。

但是，当你运行应用程序时，你会遇到一个讨厌的循环。因为effect hook不仅在组件挂载是执行，在组件更新过程中也会执行。因为我们在每一次的数据获取后都会重新设置状态，这时候组件update然后effect hook就会重新运行一遍，这就造成了数据一次又一次的获取。**我们只想在组件挂载阶段时获取数据**。这就是你要给effect hook的第二个参数传入一个空数组的原因，这样做可以避免组件更新阶段执行 effect hook ，但是依然会在挂载阶段执行它。

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(async () => {
    const result = await axios(
      'http://hn.algolia.com/api/v1/search?query=redux',
    );

    setData(result.data);
  }, []);

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

第二个参数用来定义hooks所依赖的全部变量（存放在数组中），如果一个变量改变了，effect hook就会被执行一次，如果是一个空数组的话，hooks将不会在组件更新的时候执行，因为它没有监听到任何的变量。

还有最后一个问题，在代码中，我们使用了 async/await 来获取第三方 API 提供的数据。根据文档，每一个 async 函数都将返回一个隐式的 promise：*“async 函数定义了一个异步函数，它返回的是一个`AsyncFunction`对象，异步函数是一个通过事件循环进行操作的函数，使用隐式的 Promise 返回最终的结果。”*。

但是，effect hook应该不返回任何内容或返回一个clean up函数，这就是为什么你会在控制台看到以下警告：`07:41:22.910 index.js:1452 Warning: useEffect function must return a cleanup function or nothing. Promises and useEffect(async () => …) are not supported, but you can call an async function inside an effect..`这就是为什么不允许在`useEffect`函数中直接使用async的原因。让我们通过在effect内部使用异步函数来实现它的解决方案。

```
import ...

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'http://hn.algolia.com/api/v1/search?query=redux',
      );

      setData(result.data);
    };

    fetchData();
  }, []);

  return ...;
}

export default App;
```

这就是用React Hooks获取数据的小🌰。但是，如果你对错误处理、loading状态、如何从表单中触发数据获取以及如何复用数据获取的hook感兴趣，请继续阅读。

## 如何手动或者自动触发一个 hook？

好的，我们在组件挂载后获取了一次数据。但是如何使用输入字段告诉API我们感兴趣的主题？“Redux”做为默认查询。但是如果想要查询关于“React”的呢？让我们实现一个input元素，可以获得“Redux”之外的话题。因此，就要为input元素引入一个新的state。

```
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    ...
  }, []);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <ul>
        ...
      </ul>
    </Fragment>
  );
}
export default App;
```

现在，请求数据和查询参数这两个 state 相互独立，但是我们希望将它们耦合起来，以获取输入框输入的参数指定的话题文章。通过以下修改，组件应该在挂载后按照查询参数获取相应文章。

```
...

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${query}`,
      );

      setData(result.data);
    };

    fetchData();
  }, []);

  return (
    ...
  );
}

export default App;
```

现在还差一部分：当你在input中输入一些内容时，在挂载后就不会再获取任何数据了，因为我们提供了`[]`作为第二个参数，effect没有依赖任何变量，因此只会在挂载阶段触发，但是现在的effect应该依赖`query`，每当`query`改变的时候，就应该重新获取数据。

```
...

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    ...
  }, [query]);

  return (
    ...
  );
}

export default App;
```

好了，现在input的值改变就会重新获取数据。但是又出现另外一个问题：每次输入一个新字符，就会触发 effect 进行一次新的请求。那么我们如何提供一个按钮来手动触发数据请求呢？

```
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [search, setSearch] = useState('');

  useEffect(() => {
    ...
  }, [query]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button type="button" onClick={() => setSearch(query)}>
        Search
      </button>

      <ul>
        ...
      </ul>
    </Fragment>
  );
}
```

现在，effect的触发依赖于search，而不是随输入字段中变化的query。一旦用户点击按钮，新的search才会被设置，并且会手动触发effect hook。

```
...

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [search, setSearch] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${search}`,
      );

      setData(result.data);
    };

    fetchData();
  }, [search]);

  return (
    ...
  );
}

export default App;
```

此外，search的初始状态也被设置成了与 query相同，因为组件在挂载阶段会请求一次数据，此时的结果也应该反映的是输入框中的搜索条件。然而， search和 query有点类似，这看起来比较困惑。为什么不将请求的实际URL设置到 search state 中呢？

```javascript
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(url);

      setData(result.data);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

      <ul>
        {data.hits.map(item => (
          <li key={item.objectID}>
            <a href={item.url}>{item.title}</a>
          </li>
        ))}
      </ul>
    </Fragment>
  );
}
```

这就是通过 effect hook 获取数据的案例，你可以决定effect依赖于哪个state。一旦在点击或其他effect中设置此state，此effect就会再次运行。在这种情况下，如果 URL 的 state 发生改变，则再次运行该effect通过 API 重新获取主题文章。

## React Hooks中的loading

让我们在数据的加载过程中引入一个 Loading 状态。它只是另一个由 state hook 管理的状态。Loading state 用于在 App 中渲染一个 Loading 状态。

```
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);

      const result = await axios(url);

      setData(result.data);
      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
```

一旦调用了effect获取了数据，（在组件挂载阶段或URL状态更改时发生），则加载状态设置为true。请求完成后，加载状态再次设置为false。

## React Hooks中的错误处理

如果在React Hooks中进行错误处理呢，错误只是用state hook初始化的另一个状态。一旦出现错误状态，应用程序组件就可以为用户显示反馈。使用async/await时，通常使用try/catch块进行错误处理。你可以在effect内做到：

```
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);

      try {
        const result = await axios(url);

        setData(result.data);
      } catch (error) {
        setIsError(true);
      }

      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
```

每次hook重新运行时，error state都会被重置。这会很有用，因为每次请求失败后，用户可能重新尝试，这样就能够重置错误。为了观察代码是否生效，你可以填写一个无用的 URL ，然后检查错误信息是否会出现。

## 使用表单进行数据获取

现在我们只有输入框和按钮进行组合，一旦引入更多的 input 元素，你可能想要使用表单来进行包装。此外表单还能够触发键盘的 “Enter” 事件。

```
function App() {
  ...

  return (
    <Fragment>
      <form
        onSubmit={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      ...
    </Fragment>
  );
}
```

但是现在浏览器在单击提交按钮时会重新加载，因为这是浏览器在提交表单时的默认行为。为了阻止默认行为，我们可以通过event.preventDefault()取消默认行为。这和你在React Class组件中的实现方式相同。

```
function App() {
  ...

  const doFetch = () => {
    setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`);
  };

  return (
    <Fragment>
      <form onSubmit={event => {
        doFetch();

        event.preventDefault();
      }}>
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      ...
    </Fragment>
  );
}
```

现在，当你单击提交按钮时，浏览器不会再重新加载。它和以前一样工作，但这次使用的是表单，而不是简单的input和按钮组合。你也可以按键盘上的“回车”键。

## 自定义 hook 获取数据

我们可以定义一个自定义的 hook，提取出所有与数据请求相关的东西，除了输入框的 query state，以及 Loading 状态、错误处理。还要确保返回组件中需要用到的变量。

```
const useHackerNewsApi = () => {
  const [data, setData] = useState({ hits: [] });
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);

      try {
        const result = await axios(url);

        setData(result.data);
      } catch (error) {
        setIsError(true);
      }

      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  const doFetch = () => {
    setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`);
  };

  return { data, isLoading, isError, doFetch };
}
```

现在，您的hook可以再次在App组件中使用：

```
function App() {
  const [query, setQuery] = useState('redux');
  const { data, isLoading, isError, doFetch } = useHackerNewsApi();

  return (
    <Fragment>
      ...
    </Fragment>
  );
}
```

接下来，将URL传递给`doFetch`函数：

```
const useHackerNewsApi = () => {
  ...

  useEffect(
    ...
  );

  const doFetch = url => {
    setUrl(url);
  };

  return { data, isLoading, isError, doFetch };
};

function App() {
  const [query, setQuery] = useState('redux');
  const { data, isLoading, isError, doFetch } = useHackerNewsApi();

  return (
    <Fragment>
      <form
        onSubmit={event => {
          doFetch(
            `http://hn.algolia.com/api/v1/search?query=${query}`,
          );

          event.preventDefault();
        }}
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      ...
    </Fragment>
  );
}
```

初始状态也可以是通用的。将它简单地传递给新的自定义hook：

```
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

const useDataApi = (initialUrl, initialData) => {
  const [data, setData] = useState(initialData);
  const [url, setUrl] = useState(initialUrl);
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);

      try {
        const result = await axios(url);

        setData(result.data);
      } catch (error) {
        setIsError(true);
      }

      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  const doFetch = url => {
    setUrl(url);
  };

  return { data, isLoading, isError, doFetch };
};

function App() {
  const [query, setQuery] = useState('redux');
  const { data, isLoading, isError, doFetch } = useDataApi(
    'http://hn.algolia.com/api/v1/search?query=redux',
    { hits: [] },
  );

  return (
    <Fragment>
      <form
        onSubmit={event => {
          doFetch(
            `http://hn.algolia.com/api/v1/search?query=${query}`,
          );

          event.preventDefault();
        }}
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
```

这就是使用自定义 hook 获取数据的方法，hook 本身对API无感知，它从外部获取参数，只管理必要的 state ，如数据、 Loading 和error state ，并且执行请求并将数据通过 hook 返回给组件。

## 关于数据获取的 Reducer Hook

目前为止，我们已经使用 state hooks 来管理了我们获取到的数据、Loading 状态、error state。然而，所有的状态都有属于自己的 state hook，但是他们又都连接在一起，关心的是同样的事情。如你所见，所有的它们都在数据获取函数中被使用。它们一个接一个的被调用（比如：`setIsError`、`setIsLoading`），这才是将它们连接在一起的正确用法。让我们用一个 Reducer Hook 将这三者连接在一起。

Reducer Hook 返回一个 state 对象和一个函数（用来改变 state 对象）。这个函数被称为分发函数（dispatch function），它分发一个 action，action 具有 type 和 payload 两个属性。所有的这些信息都在 reducer 函数中被接收，根据之前的状态提取一个新的状态。让我们看看在代码中是如何工作的：

```
import React, {
  Fragment,
  useState,
  useEffect,
  useReducer,
} from 'react';
import axios from 'axios';

const dataFetchReducer = (state, action) => {
  ...
};

const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  ...
};
```

Reducer Hook将reducer函数和初始state作为参数。在我们的例子中，数据，Loading和error state的初始状态的参数没有改变，但它们已经聚合到一个由一个reducer hook而不是单个的state hook。

```
const dataFetchReducer = (state, action) => {
  ...
};

const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  useEffect(() => {
    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });

      try {
        const result = await axios(url);

        dispatch({ type: 'FETCH_SUCCESS', payload: result.data });
      } catch (error) {
        dispatch({ type: 'FETCH_FAILURE' });
      }
    };

    fetchData();
  }, [url]);

  ...
};
```

现在，在获取数据时，可以使用 dispatch 函数向 reducer 函数发送信息。使用 dispatch 函数发送的对象具有一个必填的 `type` 属性和一个可选的 `payload` 属性。type 属性告诉 reducer 函数需要转换的 state 是哪个，还可以从 payload 中提取新的 state。在这里只有三个状态转换：初始化数据过程，通知数据请求成功的结果，以及通知数据请求失败的结果。

在自定义 hook 的末尾，state 像以前一样返回，但是因为我们所有的 state 都在一个对象中，而不再是独立的 state ，所以 state 对象进行解构返回。这样，调用 `useDataApi` 自定义 hook 的人仍然可以 `data`  、`isLoading`和`isError`:

```
const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  ...

  const doFetch = url => {
    setUrl(url);
  };

  return { ...state, doFetch };
};
```

最后，我们还缺少reducer函数的实现。它需要处理三个不同的状态转换，称为`FETCH_INIT`，`FETCH_SUCCESS`和`FETCH_FAILURE`。每个状态转换都需要返回一个新的状态对象。让我们看看如何使用switch case语句实现它：

```
const dataFetchReducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_INIT':
      return { ...state };
    case 'FETCH_SUCCESS':
      return { ...state };
    case 'FETCH_FAILURE':
      return { ...state };
    default:
      throw new Error();
  }
};
```

reducer函数可以通过其参数访问当前状态和传入操作。到目前为止，在out case case语句中，每个状态转换仅返回先前的状态。解构语句用于保持状态对象不可变 - 意味着状态永远不会直接变异 - 以强制执行最佳实践。现在让我们覆盖一些当前状态返回的属性来改变每个状态转换的状态：

```
const dataFetchReducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_INIT':
      return {
        ...state,
        isLoading: true,
        isError: false
      };
    case 'FETCH_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isError: false,
        data: action.payload,
      };
    case 'FETCH_FAILURE':
      return {
        ...state,
        isLoading: false,
        isError: true,
      };
    default:
      throw new Error();
  }
};
```

现在，每个状态转换（action.type决定）都返回一个基于之前 state 和 payload 的新状态。例如，在请求成功的情况下，payload 用于设置新 state 对象的 data 属性。

总之，reducer hook 确保使用自己的逻辑封装状态管理的这一部分。通过提供 action type 和可选 payload ，总是会得到可预测的状态更改。此外，永远不会遇到无效状态。例如，以前可能会意外地将 `isLoading` 和 `isError` 设置为true。在这种情况下，UI中应该显示什么？ 现在，由 reducer 函数定义的每个 state 转换都指向一个有效的 state 对象。

## 在 Hook 中中断数据请求

在React中的有一个常见问题，即使组件已经卸载（例如使用React Router切换了路由），仍会设置组件的state。我之前已经在这里写过关于这个问题的文章，它[描述了如何防止在已经Unmount的组件中调用setState](https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component/)。让我们看看我们如何阻止在数据提取的自定义钩子中设置状态：

```
const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  useEffect(() => {
    let didCancel = false;

    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });

      try {
        const result = await axios(url);

        if (!didCancel) {
          dispatch({ type: 'FETCH_SUCCESS', payload: result.data });
        }
      } catch (error) {
        if (!didCancel) {
          dispatch({ type: 'FETCH_FAILURE' });
        }
      }
    };

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [url]);

  const doFetch = url => {
    setUrl(url);
  };

  return { ...state, doFetch };
};
```

每个Effect Hook都带有一个clean up函数，它在组件卸载时运行。clean up 函数是 hook 返回的一个函数。在该案例中，我们使用 `didCancel` 变量来让 `fetchData` 知道组件的状态(挂载/卸载)。如果组件确实被卸载了，则应该将标志设置为 `true`，从而防止在最终异步解析数据获取之后设置组件状态。

*注意：实际上并没有中止数据获取（不过可以通过[Axios Cancellation](<https://github.com/axios/axios#cancellation>)来实现），但是不再为卸载的组件执行状态转换。由于 Axios 取消在我看来并不是最好的API，所以这个防止设置状态的布尔标志也可以完成这项工作。*