# O que vamos aprender?
![imagem-inicial](/imagens/0-intro.png)

> Lado esquerdo da imagem mostrando um prop drilling e o lado direito o uso do Context API para transporte de informações entre componentes

Hoje iremos aprender uma nova funcionalidade que é nativa do React, o **Context API**! O **Context API** serve para que possamos ter um gerenciador de estado global para a nossa aplicação (assim como o Redux), fazendo com que não tenhamos de utilizar os famosos "prop drilling" para resolução de alguns problemas de transporte de informações entre componentes ou quando precisamos "setar" alguma informação que muitos componentes de uma árvore precisam ter acesso.

## Você será capaz de:

 - Entender o problema que o Context API pode resolver;
 - Utilizar o Context API para gerenciar estados globais.

# Por que isso é importante?

Em uma aplicação React que é convencional, nós estamos acostumados a passar as props necessárias de um componente pai para filho, correto? Mas, e quando a complexidade da nossa aplicação e a quantidade de componentes aumenta? Essa prática de ir passando props de um componente para outro "infinitamente" pode acabar se tornando um martírio, principalmente em se tratando da necessidade de alguma manutenção. Com o **Context API** é possível que informações sejam compartilhadas entre todos os componentes da mesma árvore, sem termos de ficar passando props entre cada nível de componentes!

# Conteúdos

## Introduzindo o problema

Imagine um cenário hipotético, onde haja uma fila com 4 pessoas e que **cada uma só consiga ouvir a que está atrás**. Agora imagine, que a primeira pessoa da fila, quisesse passar uma mensagem para a última, conforme a ilustração abaixo.

![Representação de uma fila com 4 pessoas](/imagens/0-stickers-1.png)

Para resolver esta questão com o que temos em mãos até o momento, a solução seria que a primeira pessoa (P1) passasse a mensagem para a segunda (P2), a segunda para terceira (P3) e assim por diante, até que a mensagem chegasse a última da fila, no caso, a P4... (Como na ilustração abaixo).

![Representação de uma fila com 4 pessoas](/imagens/1-stickers-2.png)

Você percebeu que as pessoas P2 e P3 também tiveram que ter acesso a informação passada para que se concluísse o transporte? Mesmo que o destino da informação não fosse especificamente direcionada para elas? Pois é, se levarmos este cenário para uma árvore de componentes em React, este é um dos problemas que a falta de um gerenciador de estado global pode trazer...

Agora... Vamos levar este cenário para os códigos?

Se nós precisássemos passar uma informação de um componente que está no início da nossa árvore de componentes para o componente de nível mais baixo, veja como ficaria com o que aprendemos até o momento, utilizando a mesma síntese de nosso estudo de caso, onde em React, a nossa "fila" será representada justamente pela hierarquia de componentes, onde o P1 seria o componente de nível mais alto na hierarquia, o P2 seria filho de P1, P3 filho de P2 e assim por diante...

~~~javascript
import React from 'react';

class App extends React.Component {
  render() {
    return(
      <Person1 />
    );
  }
}

class Person1 extends React.Component {
  render() {
    const info = 'informação nunca é demais...';
    return (
      <>
        <h1>P1 - Eu tenho a info e preciso que o P4 a receba.</h1>
        <Person2 info={ info } />
      </>
    );
  }
}

class Person2 extends React.Component {
  render() {
    const { info } = this.props;
    return (
      <>
        <h1>P2 - Eu não preciso da info, mas preciso passá-la adiante...</h1>
        <Person3 info={ info } />
      </>
    );
  }
}

class Person3 extends React.Component {
  render() {
    const { info } = this.props;
    return (
      <>
        <h1>P3 - Eu não preciso da info, mas preciso passá-la adiante também...</h1>
        <Person4 info={ info } />
      </>
    );
  }
}

class Person4 extends React.Component {
  render() {
    const { info } = this.props;
    return (
      <h1>P4 - Eu preciso da info, aqui está ela: { info }</h1>
    );
  }
}

export default App;
~~~

![exibição do código](/imagens/2-introduzindo-o-problema.png)

Como pudemos ver, para que conseguíssemos que o componente `Person4` tivesse acesso a informação do componente `Person1`, tivemos de ir passando ela através de props até que chegasse ao destino desejado. Agora, reflita a respeito... E se ao invés de 4 componentes na árvore da aplicação (ou de 4 pessoas na fila, se quiser seguir a analogia citada no início), tivéssemos 10, 20 ou mais? Ficaria bem mais trabalhoso o transporte da informação desejada, não acha? Além disto, com certeza a informação poderia se perder no meio do emaranhado de componentes criados, dificultando ainda mais uma eventual manutenção na aplicação... Como resolver este problema, então? 🤔🤔🤔

### Exercícios de fixação

1. Pensando em nossa analogia feita no início, tente imaginar uma forma de passar a informação direto da primeira pessoa da fila, para a última, sem que a informação tenha de transitar pelos demais da fila!

2. Agora, tente imaginar o cenário do exercício anterior, só que no React. **OBS:** Lembre-se, a "fila de pessoas" representa a hierarquia de componentes em React, onde no caso, o componente de nível mais baixo da hierarquia, tem necessidade de uma informação que se encontra no componente de nível mais alto da hierarquia de componentes.

## Criando um contexto

Para resolvermos o problema citado anteriormente, podemos utilizar o **Context API**, que permite a criação de um estado global para nossa aplicação. OK! Mas como fazemos para usar o **Context API**, afinal? Veja abaixo!

Para utilizarmos o **Context API**, precisamos antes criar um contexto, usando o método `createContext`:
~~~javascript
import React from 'react';

const MyContext = React.createContext(defaultValue);
~~~

Outra forma de criar um contexto, seria desestruturando o `createContext` do `React`:

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext(defaultValue);
~~~

Como colocado acima, precisamos atribuir nosso contexto a uma constante, e atribuir a ela o método `createContext`. O `defaultValue` seria o valor padrão que o nosso contexto teria, podendo estar vazio "`createContext()`", ou sendo uma `string`, `objeto`, `array` (Por exemplo: `createContext("meu valor padrão de contexto")`) e etc...

### Exercícios de fixação

1. Para fixar a criação de um contexto, abra seu VSCode e repita o processo mencionado acima.

## Provider & Consumer (Fazendo a coisa funcionar!)

Já aprendemos como se cria um contexto, certo? Agora... Como podemos provê-lo e consumi-lo em nossa aplicação? Vamos lá!

Para que nossa aplicação consiga utilizar o contexto criado, precisamos primeiramente provê-lo para ela, utilizando o `Provider`!

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext();

class App extends React.Component {
  render() {
    const contextValue = "passando um valor para meu contexto";

    return (
      <MyContext.Provider value={ contextValue }>
        <Component1 />
        <Component2 />
      </MyContext.Provider>
    );
  }
}

export default App;

~~~
**OBS:** Lembrando que o código acima está sendo representado em um só arquivo apenas para efeitos didáticos. O ideal é que as lógicas de componentes e de context sejam separados em arquivos diferentes, para melhor organização.

Quando criamos um contexto, a varíavel que nós atribuirmos o método `createContext()` se torna um objeto Contexto! Este objeto Contexto possui um componente `Provider`, que aceita uma prop `value`, e é neste `value` que passamos o valor do contexto (estado global) que utilizaremos. No caso acima, estamos provendo o valor do contexto para os componentes `Component1` e `Component2` e todos os seus filhos, se houverem.

Para conseguirmos utilizar o valor do contexto, precisamos de outro componente que o objeto Contexto possui, o `Consumer`. Veja abaixo:

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext('valor padrão');

class App extends React.Component {
  render() {
    const contextValue = "valor do meu contexto aqui!";

    return (
      <MyContext.Provider value={ contextValue }>
        <Component1 />
      </MyContext.Provider>
    );
  }
}

class Component1 extends React.Component {
  render() {
    return (
      <MyContext.Consumer>
        {(value) => (
          <h1>
            Exibindo o valor atribuído ao meu contexto: {value}
          </h1>
        )}
      </MyContext.Consumer>
    );
  }
}
~~~

![exibindo o contexto](/imagens/3-exibindo-context.png)

Para que o contexto seja exibido, é necessário que uma função seja passada como filha de `MyContext.Consumer`. Esta função recebe como parâmetro o valor atual do contexto e tem de retornar um nó React (um componente, um `h1`, uma `div`, etc...).

O `value` do `Consumer` exibe sempre o valor de contexto do Provider mais próximo! Caso não haja um `Provider` para o contexto acima, o valor exibido será igual ao `defaultValue` passado no `createContext()` (neste caso, seria "valor padrão").

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext('valor padrão');

class App extends React.Component {
  render() {
    return(
        <Component1 />
    )
  }
}

class Component1 extends React.Component {
  render() {
    return(
      <MyContext.Consumer>
        {(value) => (
          <h1>
            Exibindo o valor atribuído ao meu contexto: {value}
          </h1>
        )}
      </MyContext.Consumer>
    )
  }
}
~~~

![mostrando-default-value](/imagens/4-mostrando-default-value.png)

### Exercícios de fixação

1. Agora que aprendemos a utilizar o **Context API**, crie um novo app React, crie um contexto e exiba-o no browser.

## contextType

Há um jeito de utilizarmos o nosso contexto sem utilizarmos o `Consumer` propriamente dito! E podemos fazer isto, utilizando o `contextType`.

O `contextType` é uma propriedade que pode ser atribuída ao nosso objeto Contexto! Veja seu modo de uso abaixo:

~~~javascript
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* faz um side-effect na montagem utilizando o valor de MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* renderiza algo com base no valor de MyContext */
  }
}
MyClass.contextType = MyContext;
~~~
> Bloco retirado da [documentação](https://pt-br.reactjs.org/docs/context.html) do React.

Para mais detalhes de como o `contextType` funciona, leia a [documentação](https://pt-br.reactjs.org/docs/context.html) do **contextAPI**.

## Resolvendo o estudo de caso inicial

Agora que já sabemos como usar o **Context API**, vamos resolver nosso estudo de caso inicial sobre a "fila de pessoas":

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext('valor padrão');

class App extends React.Component {
  render() {
    return (
      <Person1 />
    );
  }
}

class Person1 extends React.Component {
  render() {
    const info = 'informação nunca é demais...';
    return (
      <MyContext.Provider value={ info }>
        <h1>P1 - Eu tenho a info e preciso que o P4 a receba.</h1>
        <Person2 />
      </MyContext.Provider>
    );
  }
}

class Person2 extends React.Component {
  render() {
    return (
      <>
        <h1>P2 - Eu não preciso da info, não preciso mais passá-la adiante...</h1>
        <Person3 />
      </>
    );
  }
}

class Person3 extends React.Component {
  render() {
    return (
      <>
        <h1>P3 - Eu não preciso da info, não preciso mais passá-la adiante também...</h1>
        <Person4 />
      </>
    );
  }
}

class Person4 extends React.Component {
  render() {
    const contextValue = this.context;
    return (
      <h1>P4 - Eu preciso da info, aqui está ela: { contextValue }</h1>
    );
  }
}

Person4.contextType = MyContext;

export default App;
~~~

![resolvendo-estudo-de-caso](/imagens/5-resolvendo-estudo-caso.png)

Deixando a informação que precisa ser compartilhada globalmente em nosso **context**, não tivemos mais a necessidade de ir passando props de um componente para o outro apenas para que um componente de nível mais baixo em nossa árvore tivesse acesso!

Se fôssemos reproduzir a solução do estudo de caso do início com ilustrações, ficaria mais ou menos assim:

![fila-resolvida](/imagens/6-stickers-3.png)

![fila-resolvida-legenda](/imagens/7-stickers-4.png)


## Controlando context via estado

Podemos utilizar os estados para fazer mudanças em nosso contexto também! Veja o exemplo abaixo:

~~~javascript
import React, { createContext } from 'react';

const MyContext = createContext('valor padrão');

class App extends React.Component {
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
    const { textInfo } = this.state;

    const contextValue = {
      textInfo,
      handleClick: this.handleClick,
    }

    return (
      <MyContext.Provider value={ contextValue }>
        <Component1 />
        <Component2 />
      </MyContext.Provider>
    );
  }
}

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

export default App;

~~~
**OBS:** Lembrando que o código acima está sendo representado em um só arquivo apenas para efeitos didáticos. O ideal é que as lógicas de componentes e de context sejam separados em arquivos diferentes, para melhor organização.

Neste caso, criamos o valor do contexto como um `objeto` com duas chaves, `textInfo` que é justamente o estado criado no `App`, e a chave `handleClick` que seria a função que altera este estado. No `Component1` eu exibo o valor de `textInfo` que foi passado para o nosso context, e no `Component2` eu utilizo a função `handleClick` (que também passamos para o nosso context) que faz a alteração do estado. O legal, é que todos os componentes que tiverem acesso a esse contexto, podem ter acesso também a essas informações!

![gif mostrando context alterado](/imagens/8-mostrando-context-com-estado.gif)

![gif mostrando context alterado no devtools](/imagens/9-mostrando-context-provider.gif)

Outra coisa importante a se destacar, é que deixar a lógica do context junto ao `App` pode não fazer tanto sentido assim, do ponto de vista de organização de código. Por isso podemos deixar toda a lógica do context, separada em uma pasta a parte, e criar um **Provider** em um arquivo a parte, para que os estados do context sejam criados e gerenciados no próprio **Provider**! Veja abaixo uma forma de separação de lógica do context!

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
    const { children } = this.props; // Serve para que possamos englobar nosso MyContextProvider em outros componentes!
    const { textInfo } = this.state;

    const contextValue = {
      textInfo,
      handleClick: this.handleClick,
    }
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

`src/App.js`
~~~javascript
import React from 'react';
import MyContextProvider from './context/MyContextProvider';
import MyContext from './context';

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

export default App;

~~~

Veja que toda a lógica de gerenciamento de estado do context agora, ao invés de ser feita pelo componente `App`, é feita agora pelo Provider que nós criamos e envolvemos no `App`, o `MyContextProvider`. Isso facilita na organização do código, fazendo com que a lógica dos estados globais do context, seja responsabilidade dele próprio.

### Exercícios de fixação

1. Repita o cenário acima, separando as responsabilidades do context em uma pasta separada.


# Vamos praticar!

## Exercícios

### 1. Espelho (Mirror)

Você exibirá as informações em apenas 2 componentes neste exercício! O primeiro componente é o `Counter`, e o segundo, `Mirror`.

Antes de iniciar, copie e cole o código abaixo no seu `App.js`:

~~~javascript
import React from 'react';
import Mirror from './components/Mirror';
import Counter from './components/Counter';

class App extends React.Component {
  render() {
    return (
      <Counter />
      <hr />
      <Mirror />
    );
  }
}

export default App;

~~~

Utilizando o **Context API**, crie um estado global que armazene um contador iniciando em `0`, e uma função que altere esse estado. Tanto o componente `Counter` quanto o `Mirror` devem possuir um texto mostrando o valor atual do contador e dois botões, um que faz o **incremento** e outro que faz o **decremento**.

Quando qualquer botão for clicado, o valor do contador deve ser atualizado para os dois componentes, como se um fosse o espelho do outro. 

**Segue abaixo um exemplo:**

![imagem de como pode ficar o exercicio 01](/imagens/10-exercicio-1.gif)

### 2. Tela de Login / Profile

Antes de iniciar, copie e cole o código abaixo no seu `App.js`:

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

Primeiramente você deve criar um componente de `Profile` e um de `Login`;

- O componente de `Login` deve possuir um input e um botão, que ao ser clicado, troca o `isLogged` para `true` e exibe o componente `Profile`. Como pode perceber, o `isLogged` é um estado que foi criado dentro do `MyContextProvider`.

- O componente de `Profile` deve possuir um `h1` que exiba o que foi digitado no input no componente de `Login`, e também deve possuir um botão de logoff, que seta o `isLogged` de `true` para `false` novamente, voltando a exibir o `Login`;

**Segue abaixo um exemplo:**

![imagem de como pode ficar o exercicio 02](/imagens/11-exercicio-2.gif)

### 3. Dark Theme / Light Theme

Antes de iniciar, copie e cole o código abaixo no seu `App.js`:

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

Neste exercício você deve utilizar o **context API** para armazenar no estado global o tema ('light' ou 'dark') da aplicação. Além disso, a aplicação deve possuir **duas** páginas. As páginas devem conter um `h1` com algum título que identifique a página, um botão que vai conter o texto `Dia` ou `Noite` conforme o tema que estiver setado e um link para a página anterior/próxima. Ao trocar de uma página para outra o tema deve permanecer igual.

**Segue abaixo um exemplo:**

![imagem de como pode ficar o exercicio 03](/imagens/12-exercicio-3.gif)


## Bônus

### 1. Todo list com context API

Elabore uma Todo list simples, o `App` deve renderizar um componente `TodoList` e um `Footer`. O componente `TodoList` deve conter um input de texto, um botão de adicionar um novo item na lista e cada item adicionado também deve ter um botão de remover. O `Footer` deve conter a quantidade de itens atual da lista.

**Segue abaixo um exemplo:**

![imagem de como pode ficar o exercicio 04](/imagens/13-exercicio-4.gif)

# Recursos Adicionais

- [Documentação do context](https://pt-br.reactjs.org/docs/context.html)
- [React Context API Step By Step](https://levelup.gitconnected.com/react-context-api-step-by-step-f1ee25d90c55)
- [Redux vs. React Context](https://medium.com/@hnordt/redux-vs-react-context-87a7053c12df)
- [Entendendo a Context API do React: criando um componente de loading](https://medium.com/reactbrasil/entendendo-a-context-api-do-react-criando-um-componente-de-loading-a84f84007dc7)