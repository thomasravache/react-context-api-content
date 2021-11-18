# Gabarito dos exercícios

## Exercícios de fixação

### Introduzindo o problema

1. Partindo do pressuposto de que cada pessoa da fila só consegue ouvir a que está atrás, teríamos que ter algum tipo de "ponte" que fizesse a ligação entre a primeira pessoa da fila e a última.

2. Realmente é difícil pensar em uma solução que não envolva prop drilling. Por isso, gerenciadores de estados globais são muito interessantes a medida que uma aplicação se torna maior e mais complexa.

### Criando um contexto

1. Código:
~~~javascript
import React from 'react';

const MyContext = React.createContext(defaultValue);
~~~

ou

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext(defaultValue);
~~~

### Provider & Consumer (Fazendo a coisa funcionar!)

1. Código:
`/src/context/MyContext.js`

~~~javascript
import React from 'react';

export const MyContext = React.createContext({});

export default MyContext;

~~~

`src/components/Component1.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContext';

class Component1 extends React.Component {
  render() {
    return (
      <MyContext.Consumer>
        {({ info }) => (
          <h1>Exibindo chave info do context: { info }</h1>
        )}
      </MyContext.Consumer>
    );
  }
}

export default Component1;

~~~

`src/components/App.js`

~~~javascript
import React from 'react';
import Component1 from './components/Component1';
import MyContext from './context/MyContext';

class App extends React.Component {
  render() {
    const contextValue = {
      info: 'valor do contexto'
    };

    return (
      <MyContext.Provider value={ contextValue }>
        <Component1 />
      </MyContext.Provider>
    );
  }
}

export default App;

~~~

### Controlando context via estado

1. Código:

`src/context/index.js`
~~~javascript
import React from 'react';

const MyContext = React.createContext({});

export default MyContext;

~~~

`src/context/MyContextProvider.js`
~~~javascript
import React from 'react';
import PropTypes from 'prop-types';
import MyContext from '.';

class MyContextProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      textInfo: 'valor inicial de estado',
    }
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({
      textInfo: 'valor alterado',
    });
  }

  render() {
    const { children } = this.props;
    const { textInfo } = this.state;

    const contextValue = {
      textInfo,
      handleClick: this.handleClick,
    };
    return (
      <MyContext.Provider value={ contextValue }>
        { children }
      </MyContext.Provider>
    );
  }
}

MyContextProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default MyContextProvider;

~~~

`src/components/Component1jsx`
~~~javascript
import React from 'react';
import MyContext from '../context';

class Component1 extends React.Component {
  render() {
    const { textInfo } = this.context;
    return (
      <>
        <h1>Estado atual: { textInfo }</h1>
      </>
    );
  }
}

Component1.contextType = MyContext;

export default Component1;

~~~

`src/components/Component2.jsx`
~~~javascript
import React from 'react';
import MyContext from '../context';

class Component2 extends React.Component {
  render() {
    const { handleClick } = this.context;
    return (
      <button
        type="button"
        onClick={ handleClick }
      >Clique para alterar o estado que o context recebe</button>
    );
  }
}

Component2.contextType = MyContext;

export default Component2;

~~~

`src/App.js`
~~~javascript
import React from 'react';
import MyContextProvider from './context/MyContextProvider';
import Component1 from './components/Component1';
import Component2 from './components/Component2';

class App extends React.Component {
  render() {
    return (
      <MyContextProvider>
        <Component1 />
        <Component2 />
      </MyContextProvider>
    );
  }
}

export default App;

~~~

## Exercícios

### Exercício 1 - (Mirror)

`src/context/MyContextProvider.js`

~~~javascript
import React from 'react';
import PropTypes from 'prop-types';

export const MyContext = React.createContext({});

class MyContextProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0,
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick({ target: { textContent } }) {
    if (textContent === '+') {
      this.setState((prevState) => ({
        counter: prevState.counter + 1,
      }));
    } else {
      this.setState((prevState) => ({
        counter: prevState.counter - 1,
      }));
    }
  }

  render() {
    const { children } = this.props;
    const { counter } = this.state;
    const contextValue = {
      counter,
      handleClick: this.handleClick,
    };
    return(
      <MyContext.Provider value={ contextValue }>
        { children }
      </MyContext.Provider>
    );
  }
}

MyContextProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default MyContextProvider;

~~~

`/src/components/Counter.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class Counter extends React.Component {
  render() {
    const { counter, handleClick } = this.context;
    return (
      <div>
        <h1>Contador: { counter }</h1>
        <button
          type="button"
          onClick={ handleClick }
        >
          +
        </button>
        <button
          type="button"
          onClick={ handleClick }
        >
          -
        </button>
      </div>
    );
  }
}

Counter.contextType = MyContext;

export default Counter;

~~~

`/src/components/Mirror.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class Mirror extends React.Component {
  render() {
    const { counter, handleClick } = this.context;
    return (
      <div>
        <h1>Contador: { counter }</h1>
        <button
          type="button"
          onClick={ handleClick }
        >
          +
        </button>
        <button
          type="button"
          onClick={ handleClick }
        >
          -
        </button>
      </div>
    );
  }
}

Mirror.contextType = MyContext;

export default Mirror;

~~~

`/src/App.js`

~~~javascript
import React from 'react';
import Mirror from './components/Mirror';
import Counter from './components/Counter';
import MyContextProvider from './context/MyContextProvider';

class App extends React.Component {
  render() {
    return (
      <MyContextProvider>
        <Counter />
        <hr />
        <Mirror />
      </MyContextProvider>
    );
  }
}

export default App;

~~~

### Exercício 2 - (Tela de Login / Profile)

`/src/context/MyContextProvider.js`

~~~javascript
import React from 'react';
import PropTypes from 'prop-types';

export const MyContext = React.createContext({});

class MyContextProvider extends React.Component {
  constructor(props) {
    super(props);
  
    this.state = {
      isLogged: false,
      inputLogin: '',
    };
    this.setIsLogged = this.setIsLogged.bind(this);
    this.handleChangeInputLogin = this.handleChangeInputLogin.bind(this);
  }

  setIsLogged({ target: { textContent } }) {
    this.setState((prevState) => ({
      isLogged: !prevState.isLogged,
    }));
    if (textContent === 'Logoff') {
      this.setState({
        inputLogin: '',
      });
    }
  }

  handleChangeInputLogin({ target: { value } }) {
    this.setState({
      inputLogin: value,
    });
  }

  render() {
    const { children } = this.props;
    const { isLogged, inputLogin } = this.state;

    const contextValue = {
      isLogged,
      setIsLogged: this.setIsLogged,
      handleChangeInputLogin: this.handleChangeInputLogin,
      inputLogin,
    };
    return (
      <MyContext.Provider value={ contextValue }>
        { children }
      </MyContext.Provider>
    );
  }
}

MyContextProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default MyContextProvider;
~~~

`/src/components/Login.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class Login extends React.Component {
  render() {
    const { handleChangeInputLogin, setIsLogged, inputLogin } = this.context;
    return (
      <div>
        <h1>Login</h1>
        <label htmlFor="">
          Nome:
          <input type="text" onChange={ handleChangeInputLogin } value={ inputLogin } />
        </label>
        <button
          type="button"
          onClick={ setIsLogged }
        >
          Login
        </button>
      </div>
    );
  }
}

Login.contextType = MyContext;

export default Login;

~~~

`/src/components/Profile.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class Profile extends React.Component {
  render() {
    const { setIsLogged, inputLogin } = this.context;
    return (
      <div>
        <h1>Bem vindo, { inputLogin }</h1>
        <button
          type="button"
          onClick={ setIsLogged }
        >
          Logoff
        </button>
      </div>
    );
  }
}

Profile.contextType = MyContext;

export default Profile;

~~~

`/src/App.js`

~~~javascript
import React from 'react';
import './App.css';
import MyContextProvider, { MyContext } from './context/MyContextProvider';
import Login from './components/Login';
import Profile from './components/Profile';

class App extends React.Component {
  render() {
    return (
      <MyContextProvider>
        <MyContext.Consumer>
          {({ isLogged }) => (
            isLogged ? <Profile /> : <Login />
          )}
        </MyContext.Consumer>
      </MyContextProvider>
    );
  }
}

export default App;

~~~

### Exercício 3 - (Tela de Login / Profile)

`/src/context/MyContextProvider.js`

~~~javascript
import React from 'react';
import PropTypes from 'prop-types';

export const MyContext = React.createContext({});

class MyContextProvider extends React.Component {
  constructor(props) {
    super(props);
  
    this.state = {
      theme: 'light',
    };
    this.handleClickTheme = this.handleClickTheme.bind(this);
  }

  handleClickTheme() {
    const { theme } = this.state;

    if (theme === 'light') {
      this.setState({
        theme: 'dark',
      });
      document.body.style.backgroundColor = 'black';
      document.body.style.color = 'white';
    } else {
      this.setState({
        theme: 'light',
      });
      document.body.style.backgroundColor = 'white';
      document.body.style.color = 'black';
    }
  }

  render() {
    const { children } = this.props;
    const { theme } = this.state;

    const contextValue = {
      theme,
      handleClickTheme: this.handleClickTheme,
    };
    return (
      <MyContext.Provider value={ contextValue }>
        { children }
      </MyContext.Provider>
    );
  }
}

MyContextProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default MyContextProvider;

~~~

`src/components/Page1.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';
import { Link } from 'react-router-dom';

class Page1 extends React.Component {
  render() {
    const { theme, handleClickTheme } = this.context;
    return (
      <div>
        <h1>Página 1</h1>
        <button
          type="button"
          onClick={ handleClickTheme }
        >
          {theme === 'light' ? 'Noite' : 'Dia'}
        </button>
        <Link
        to="/page2"
        style={{
          textDecoration: 'none',
          color: 'inherit',
        }}
        >
          Ir para página 2
        </Link>
      </div>
    );
  }
}

Page1.contextType = MyContext;

export default Page1;

~~~

`src/components/Page2.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';
import { Link } from 'react-router-dom';

class Page2 extends React.Component {
  render() {
    const { theme, handleClickTheme } = this.context;
    return (
      <div>
        <h1>Página 2</h1>
        <button
          type="button"
          onClick={ handleClickTheme }
        >
          {theme === 'light' ? 'Noite' : 'Dia'}
        </button>
        <Link
        to="/"
        style={{
          textDecoration: 'none',
          color: 'inherit',
        }}
        >
         Ir para página 1
        </Link>
      </div>
    );
  }
}

Page2.contextType = MyContext;

export default Page2;

~~~

`src/App.js`

~~~javascript
import React from 'react';
import './App.css';
import MyContextProvider from './context/MyContextProvider';
import { Switch, Route } from 'react-router-dom';
import Page1 from './components/Page1';
import Page2 from './components/Page2';

class App extends React.Component {
  render() {
    return (
      <MyContextProvider>
          <Switch>
            <Route exact path="/" component={ Page1 } />
            <Route exact path="/page2" component={ Page2 } />
          </Switch>
      </MyContextProvider>
    );
  }
}

export default App;

~~~

`src/index.js`

~~~javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { BrowserRouter } from 'react-router-dom'

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

~~~

### Exercício Bônus - (Todo List com context API)

`/src/context/MyContextProvider.js`

~~~javascript
import React from 'react';
import PropTypes from 'prop-types';

export const MyContext = React.createContext({});

class MyContextProvider extends React.Component {
  constructor(props) {
    super(props);
  
    this.state = {
      todoList: [],
      inputValue: '',
      actualId: 0,
    };
    this.handleAddTodo = this.handleAddTodo.bind(this);
    this.handleChange = this.handleChange.bind(this);
    this.handleRemoveTodo = this.handleRemoveTodo.bind(this);
  }

  handleAddTodo() {
    const { todoList, inputValue, actualId } = this.state;

    this.setState({
      todoList: [...todoList, { id: actualId, inputValue }],
    }, () => {
      this.setState((prevState) => ({
        actualId: prevState.actualId + 1
      }));
    });
  }

  handleRemoveTodo(id) {
    const { todoList } = this.state;

    this.setState({
      todoList: [...todoList]
        .filter((item) => item.id !== id),
    });
  }

  handleChange({ target: { value } }) {
    this.setState({
      inputValue: value,
    });
  }

  render() {
    const { children } = this.props;
    const { inputValue, todoList } = this.state;

    const contextValue = {
      todoList,
      inputValue,
      handleAddTodo: this.handleAddTodo,
      handleChange: this.handleChange,
      handleRemoveTodo: this.handleRemoveTodo,
    };
    return (
      <MyContext.Provider value={ contextValue }>
        { children }
      </MyContext.Provider>
    );
  }
}

MyContextProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default MyContextProvider;

~~~

`/src/components/TodoList.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class TodoList extends React.Component {
  render() {
    const {
      handleAddTodo, handleChange, todoList,
      inputValue, handleRemoveTodo,
    } = this.context;

    return (
      <div>
        <h1>Todo List</h1>
        <label htmlFor="">
          <input
            type="text"
            onChange={ handleChange }
            value={ inputValue }  
          />
        </label>
        <ol>
          {todoList.length !== 0 && todoList.map(({ id, inputValue }) => (
            <li key={ id }>
              { inputValue }
              <button
                type="button"
                onClick={ () => handleRemoveTodo(id) }
              >
                X
              </button>
            </li>
          ))}
        </ol>
        <button
          type="button"
          onClick={ handleAddTodo }
        >
          Adicionar
        </button>
      </div>
    );
  }
}

TodoList.contextType = MyContext;

export default TodoList;

~~~

`/src/components/Footer.jsx`

~~~javascript
import React from 'react';
import { MyContext } from '../context/MyContextProvider';

class Footer extends React.Component {
  render() {
    const { todoList } = this.context;
    return (
      <div>
        <h1>Quantidade de itens: {todoList.length}</h1>
      </div>
    );
  }
}

Footer.contextType = MyContext;

export default Footer;

~~~

`/src/App.js`

~~~javascript
import React from 'react';
import './App.css';
import MyContextProvider from './context/MyContextProvider';
import TodoList from './components/TodoList';
import Footer from './components/Footer';

class App extends React.Component {
  render() {
    return (
      <MyContextProvider>
        <TodoList />
        <Footer />
      </MyContextProvider>
    );
  }
}

export default App;

~~~