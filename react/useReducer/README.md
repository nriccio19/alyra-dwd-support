# `useReducer` hook

Le hook `useReducer` permet de gérer le _state_, il a le même rôle que `useState`.

`useState` sera le premier choix en ce qui concerne la gestion du state. Le hook `useReducer` est son alternative **plus configurable** mais aussi **plus complexe.** Nous pouvons dire que `useReducer` et une extension de `useState.`

Pour comprendre son comportement nous allons transformer quelques exemples où nous utilisons initialement `useState` vers `useReducer`.

## Anatomie

```javascript
const reducer = (state, action) => {
  /* ... */
  // retourne nouvelle valeur de state
}

const [state, dispatch] = React.useReducer(reducer, initialState, init)
```

Comme dans l'API de `useState`, `useReducer` retourne un array, le premier élément est notre state, le deuxième est une fonction (`dispatch`).

Nous appelons la fonction `dispatch` à chaque fois où nous allons mettre à jour `state`. Pourtant la fonction `dispatch` ne modifie pas `state` directement. Elle délègue ceci à la fonction `reducer`.

La fonction `reducer` retourne une nouvelle valeur de state et si la modification de state est détectée le re-render est déclenché.

```javascript
dispatch(action)
// --->
reducer(state, action)
```

![](https://wptemplates.pehaa.com/assets/alyra/usereducer-anatomie.png)

La dénomination `dispatch`, `action` et `reducer` est tout à fait facultative et suit une certaine convention qui a ces racines dans `Redux`.

Dans notre premier exemple, pour voir plus clairement le passage de `useState` vers `useReducer`, nous n'allons pas utiliser les noms `dispatch` ou `action`.

## Exemple 1

```javascript
// avec useState
const Counter = () => {
  const [count, setCount] = React.useState(0)
  const handleButtonClick = () => {
    setCount(count + 1)
  }
  return (
    <button type="button" onClick={handleButtonClick}>
      {count}
    </button>
  )
}
```

```javascript
// avec useReducer
const reducer = (count, newCount) => newCount
const Counter = () => {
  const [count, setCount] = React.useReducer(reducer, 0)
  const handleButtonClick = () => {
    setCount(count + 1)
    // setCount(count + 1) délègue et appelle reducer(count, count + 1)
  }
  return (
    <button type="button" onClick={handleButtonClick}>
      {count}
    </button>
  )
}
```

Nous pouvons observer que le code suivant :

```javascript
const [state, setState] = React.useState(initialState)
```

est équivalent à :

```javascript
const reducer = (state, newState) => newState
const [state, setState] = React.useReducer(reducer, initialState)
```

où nous ajoutons un étape supplémentaire - `reducer`.

---

**Digression (optionnelle) :** Pour être plus précis, il faudrait prendre en compte aussi qu'avec l'API de `useState`, `setCount` peut avoir une _fonction_ comme paramètre, par exemple `setCount(count => count + 1)`. Le code ci-dessous fonctionne correctement dans les 2 cas `setCount(count + 1)` et `setCount(count => count + 1)` :

```javascript
const reducer = (count, newCount) =>
  typeof newCount === "function" ? newCount(count) : newCount
const [count, setCount] = React.useReducer(reducer, 0)
```

---

Pour résumer, `useReducer` est comme `useState` avec une étape supplémentaire, la fonction `reducer`.

Voici le code complet :

https://codepen.io/alyra/pen/Pozeedw

Est-ce cet exemple plus simple (lisible, maintenable) avec `useReducer` qu'avec `useState` ? Non, et nous allons pas utiliser `useReducer` dans ce type des cas.

Nous allons maintenant passer aux exemples où l'utilisation de `useReducer` est justifiée (notre code deviendra plus lisible, plus court, plus maintanable). Nous allons voir dans les exemples suivants que nous pouvons tailler `reducer` selon nos besoins. Par conséquent, nous avons aussi plus de liberté au niveau de la fonction `dispatch`.

## Exemple 2

https://codepen.io/alyra/pen/vYKjmvg

```javascript
const [text, setText] = React.useState(
  "Portez ce vieux whisky au juge blond qui fume !? 0123456789"
)
const [size, setSize] = React.useState("20")
const [italic, setItalic] = React.useState(false)
```

et nous utilisons ensuite :

- `setText(event.target.value)`,
- `setSize(event.target.value)`
- `setItalic(event.target.checked)`

Voici la possible version avec `React.useReducer`

```javascript
const initialState = {
  text: "Portez ce vieux whisky au juge blond qui fume !? 0123456789",
  size: "20",
  italic: false,
}
const [state, setState] = React.useReducer(reducer, initialState)
```

Notre fonction `reducer` est définie comme ci-dessous :

```
const reducer = (state, newState) => {
  return {
    ...state,
    ...newState
  }
}
```

et les mises à jour de state appelées comme ceci :

- `setState({ text: event.target.value })` => `reducer` modifiera la clé `text`,
- `setState({ size: event.target.value })` => `reducer` modifiera la clé `size`
- `setState({ italic: event.target.checked })` => `reducer` modifiera la clé `italic`

Vous trouverez le code complet dans le pen suivant :

https://codepen.io/alyra/pen/OJXZgJQ

## Exemple 3

https://codepen.io/alyra/pen/zYBjade

```javascript
const ShoppingApp = () => {
  const initialShopping = ["cumin", "curry", "poivre"]
  const [shopping, setShopping] = React.useState(initialShopping)

  const handleDoneClick = (product) => {
    // action "REMOVE"
    setShopping(shopping.filter((el) => el !== product))
  }

  return (
    <section>
      <h2>My shopping List</h2>
      <ol>
        {shopping.map((product) => {
          return (
            <li key={product}>
              {product}
              <button onClick={() => handleDoneClick(product)}>done!</button>
            </li>
          )
        })}
      </ol>
      <AddProductForm shopping={shopping} setShopping={setShopping} />
    </section>
  )
}

const AddProductForm = (props) => {
  const { shopping, setShopping } = props

  const handleFormSubmit = (event) => {
    event.preventDefault()
    const newProduct = event.target.elements.product.value

    if (shopping.includes(newProduct)) {
      alert(`${newProduct} est déjà sur la liste`)
    } else {
      // action "ADD"
      setShopping([...shopping, newProduct])
    }
    event.target.reset()
  }

  return (
    <form onSubmit={handleFormSubmit}>
      <label htmlFor="product">Ajouter sur la liste</label>
      <input id="product" required />
      <button type="submit">J'ajoute !</button>
    </form>
  )
}
```

Dans l'exemple si-dessus nous appelons `setShopping` pour effectuer une des deux actions :

- ajouter un nouvel produit sur la liste (nous pouvons appeler cette action "ADD")
- retirer un produit de la liste ("REMOVE")

Par convention les noms des actions sont souvent en majuscules.

Voici comment nous allons remplacer `useState` :

```javascript
// const initialShopping = ["cumin", "curry", "poivre"]
// avant : const [shopping, setShopping] = React.useState(initialShopping);
const initialShopping = ["cumin", "curry", "poivre"]
const [shopping, dispatch] = React.useReducer(shoppingReducer, initialShopping)
```

Et voici la fonction `shoppingReducer` :

```javascript
const shoppingReducer = (shopping, action) => {
  switch (action.type) {
    case "ADD":
      return [...shopping, action.product]
    case "REMOVE":
      return shopping.filter((el) => el !== action.product)
    default:
      throw new Error(`Unsupported action type ${action.type}`)
  }
}
```

Quand la fonction `dispatch` est appelée, `shoppingReducer` est exécutée. Les arguments de `shoppingReducer` est state (`shopping`) dans notre cas, 2e paramètre (`action`) reprend l'argument passé à `dispatch`.

Nous devons alors passer `type` et `product` dans `dispatch`.

```javascript
// avant setShopping(shopping.filter((el) => el !== product))
dispatch({ type: "REMOVE", product })
// avant setShopping([...shopping, newProduct])
dispatch({ type: "ADD", product: newProduct })
```

Vous trouverez le code complet ici :

https://codepen.io/alyra/pen/jOrxZov

## Exemple 4

Analysez l'exemple ci-dessous qui fusionne l'approche d'exemple 2 et d'exemple 3

`useState` - code de départ est ici :

https://codepen.io/alyra/pen/VwaBvqZ

Essayez de faire le refactoring vous même, vous devez alors mettre en place une variable `state` avec la valeur initiale

```javascript
const initialState = {
  planets: [],
  loading: false,
  page: 1,
  hasNext: true,
}
```

Notre fonction `reducer` supportera 3 actions "LOADING" (`loading` devient `true`), "DATA" et "NEXT PAGE".

`useReducer` - vous trouverez le code complet ici :

https://codepen.io/alyra/pen/GRqdxrQ

---

## "Lazy initial state"

Vous vous rappelez de "lazy initial state" dans l'API de `useState` ?

```javascript
const [shopping, setShopping] = useState(expensiveOperationFunction()) // pas bien 👎
```

React a besoin du résultat de `expensiveOperationFunction` uniquement une fois, quand le component monte, pour initier `shopping`. Pourtant mis en place comme ci-dessus, `expensiveOperationFunction` sera exécutée à chaque re-render.

Par contre :

```javascript
const [shopping, setShopping] = useState(() => expensiveOperationFunction()) //  bien 👍
```

ou simplement

```javascript
const [shopping, setShopping] = useState(expensiveOperationFunction) //  bien 👍
```

L'API de `useReducer` nous permet également à initier state uniquement a premier render. Pour cela nous utilisons le 3 argument de `useReducer`, par exemple

```javascript
const init = (initialState) = {
  // initialState = le 2e param de useReducer
  if (localStorage.getItem("shopping")) {
    return JSON.parse(localStorage.getItem("shopping"))
  }
  return initialState
}
const [shopping, dispatch] = React.useReducer(reducer, [], init)
```